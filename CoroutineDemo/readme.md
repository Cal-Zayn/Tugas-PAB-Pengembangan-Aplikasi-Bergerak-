# Latihan 9 — Coroutines & State Management

---

## 1. Tambah Satu Permintaan Lagi (`getCourseCount`)

> **Kesimpulan:** Total waktu pemuatan tetap **~2 detik (2000 ms)**, bukan dijumlahkan (`2000 + 1000 + 1500 = 4500 ms`).

Karena ketiga fungsi dipanggil secara **paralel (konkuren)** menggunakan `async` di dalam `Dispatchers.IO`, `coroutineScope` akan menunggu hingga **tugas yang paling lama** selesai:

* `getTasks()` — **2000 ms** *(paling lama)*
* `getAnnouncement()` — **1000 ms**
* `getCourseCount()` — **1500 ms**

---

## 2. Tombol Retry

Tampilkan tombol **Retry** di sebelah pesan error. Tombol ini memanggil `loadData()` lagi dan **hanya muncul jika `errorMessage != null`**.

---

## 3. Menangkap Terlalu Banyak (`IOException` vs `Exception`)

### Sebelumnya: `catch (e: IOException)`
* Pembatalan coroutine melempar `CancellationException`.
* Karena `CancellationException` **bukan** turunan dari `IOException`, exception melewati blok `catch` dan membatalkan coroutine secara bersih.
* `cancelLoading()` kemudian memperbarui pesan menjadi `"Loading cancelled"`.

### Sekarang: `catch (e: Exception)`
* `CancellationException` **merupakan** turunan dari `java.lang.Exception`.
* Blok `catch` ikut menangkap `CancellationException` dan menganggapnya sebagai **error biasa**, lalu menampilkan pesan error dari exception pembatalan coroutine.

---

## 4. Di Mana Macetnya? (`delay` vs `Thread.sleep` & `withContext`)

| Skenario | Kondisi UI / Layar | Penjelasan Teknis |
| :--- | :--- | :--- |
| **`Thread.sleep(2000L)` di dalam `withContext(Dispatchers.IO)`** | **Tidak macet / freeze** | Pemblokiran thread hanya terjadi di thread pool *background* (`Dispatchers.IO`). **Main/UI Thread** tetap bebas merender UI dan merespons input pengguna. |
| **`Thread.sleep(2000L)` tanpa `withContext(Dispatchers.IO)`** | **Macet / freeze selama ~2 detik** | `viewModelScope.launch` berjalan di **Main Thread (`Dispatchers.Main`)** secara default, sehingga `Thread.sleep` memblokir pemrosesan UI dan animasi secara langsung. |

---

## 5. Pertanyaan Konsep

### (a) Perbedaan `launch` dan `async`
* **`launch`**: Menjalankan coroutine dengan pola *fire-and-forget* (tidak mengembalikan nilai hasil) dan mengembalikan objek **`Job`**.
* **`async`**: Menjalankan coroutine yang mengembalikan nilai hasil dan mengembalikan objek **`Deferred<T>`** yang nilainya diambil menggunakan **`.await()`**.

### (b) Kenapa `GlobalScope` Sebaiknya Dihindari?
* Terikat pada **Application Lifecycle**, bukan siklus hidup layar atau `ViewModel`.
* Coroutine tidak otomatis dibatalkan saat komponen UI dihancurkan, sehingga berisiko memicu **memory leak** dan pemborosan resource CPU.

### (c) Apa yang Terjadi pada `getAnnouncement()` Kalau `getTasks()` Gagal?
* Berdasarkan **Structured Concurrency** di dalam `coroutineScope { ... }`, kegagalan (`Exception`) pada salah satu *child coroutine* (`getTasks()`) akan membuat `coroutineScope` **otomatis membatalkan seluruh *child coroutine* lainnya** (termasuk `getAnnouncement()`) dan meneruskan exception ke blok `catch`.

---

## 6. Tantangan — Tombol Sort

Tambahkan tombol **Sort** untuk mengurutkan daftar tugas berdasarkan judul. Jalankan proses pengurutannya di **`Dispatchers.Default`** menggunakan **`withContext`**, lalu perbarui UI state dengan hasilnya.
