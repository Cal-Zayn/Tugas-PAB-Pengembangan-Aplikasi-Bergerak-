# Untuk menanggapi pertanyaan - pertanyaan pada Latihan 9, maka izinkan saya untuk menjawabnya
# melalui dokumen Readme.md ini, dibawah ini adalah pertanyaan sekaligus jawaban dari saya sendiri.



# Latihan 9

1. **Tambah satu permintaan lagi.** Tambahkan `suspend fun getCourseCount(): Int` di `TaskRepository` dengan `delay(1500L)`. Simpan hasilnya di field baru pada `TasksUiState`, lalu muat dengan `async` ketiga di `loadData()`. Apakah total waktu pemuatan masih sekitar 2 detik? Jelaskan alasannya.

   > **Jawaban:**
   > Karena ketiga fungsi dipanggil secara paralel (konkuren) menggunakan `async` di dalam thread pool `Dispatchers.IO`. `coroutineScope` akan menunggu hingga tugas yang paling lama selesai.
   > * `getTasks()` = **2000 ms** *(paling lama)*
   > * `getAnnouncement()` = **1000 ms**
   > * `getCourseCount()` = **1500 ms**
   >
   > Karena berjalan secara bersamaan, **total waktu eksekusi dibatasi oleh tugas terlama yaitu ~2000 ms (2 detik)**, bukan dijumlahkan.

---

2. **Tombol Retry.** Tampilkan tombol Retry di sebelah pesan error. Tombol ini memanggil `loadData()` lagi dan hanya muncul kalau `errorMessage` tidak null.

---

3. **Menangkap terlalu banyak.** Ganti `catch (e: IOException)` menjadi `catch (e: Exception)`. Tekan Load, lalu segera tekan Cancel. Pesan apa yang sekarang muncul, dan kenapa berbeda dari sebelumnya?

   > **Jawaban:**
   > * **Sebelumnya (`catch (e: IOException)`):** membatalkan coroutine dengan cara melempar `CancellationException`. Karena `CancellationException` bukan turunan dari `IOException`, exception tersebut melewati blok `catch` dan membatalkan coroutine secara bersih. Fungsi `cancelLoading()` kemudian meng-update pesan menjadi `"Loading cancelled"`.
   > * **Sekarang (`catch (e: Exception)`):** `CancellationException` adalah turunan dari `java.lang.Exception`. Blok `catch` menangkap `CancellationException` tersebut dan menganggapnya sebagai error biasa, lalu menampilkan pesan error dari exception pembatalan coroutine.

---

4. **Di mana macetnya?** Ganti `delay(2000L)` dengan `Thread.sleep(2000L)` di dalam `withContext(Dispatchers.IO)` pada `getTasks()`. Apakah layar jadi macet? Setelah itu, hapus `withContext` dari `getTasks()` dan coba lagi. Jelaskan kenapa hasilnya berbeda.

   > **Jawaban :**
   > Mengganti `delay(2000L)` dengan `Thread.sleep(2000L)` di dalam `withContext(Dispatchers.IO)`:
   > Hasilnya **Layar TIDAK macet/freeze**. Karena walaupun `Thread.sleep` memblokir thread tempat ia berjalan, pemblokiran terjadi pada thread background (`Dispatchers.IO`). Thread Utama (Main/UI Thread) tetap bebas untuk merender UI dan merespons sentuhan pengguna.
   >
   > Berbeda dengan menghapus `withContext(Dispatchers.IO)` dan menjalankan `Thread.sleep(2000L)`:
   > Maka, hasilnya akan menjadi **Layar macet /freeze dengan total selama 2 detik**. Alasannya karena Coroutine dipanggil di `viewModelScope.launch` yang secara default berjalan di Main Thread (`Dispatchers.Main`). Tanpa `withContext(Dispatchers.IO)`, `Thread.sleep(2000L)` dieksekusi langsung di Main Thread sehingga memblokir pemrosesan UI dan animasi.

---

5. **Pertanyaan konsep.** Jelaskan dengan bahasa Anda sendiri: (a) apa beda `launch` dan `async`; (b) kenapa `GlobalScope` sebaiknya dihindari; (c) apa yang terjadi pada `getAnnouncement()` kalau `getTasks()` gagal.

   > **Jawaban:**
   >
   > **(a) Beda `launch` dan `async`:**
   > * **`launch`**: Digunakan untuk menjalankan coroutine dengan pola *fire-and-forget* (tidak mengembalikan nilai hasil). Mengembalikan objek `Job`.
   > * **`async`**: Digunakan untuk menjalankan coroutine yang mengembalikan nilai hasil (result). Mengembalikan objek `Deferred<T>` yang nilainya dapat diambil menggunakan `.await()`.
   >
   > **(b) Kenapa `GlobalScope` sebaiknya dihindari:**
   > `GlobalScope` terikat pada siklus hidup seluruh aplikasi (Application Lifecycle), bukan komponen UI. Coroutine yang dijalankan di `GlobalScope` tidak akan otomatis dibatalkan saat layar atau ViewModel dihancurkan. Hal ini menyebabkan **Memory Leak** dan pemborosan resource CPU.
   >
   > **(c) Apa yang terjadi pada `getAnnouncement()` kalau `getTasks()` gagal:**
   > Berdasarkan konsep **Structured Concurrency** dalam `coroutineScope { ... }`, jika salah satu child coroutine (`getTasks()`) mengalami crash/melempar `Exception`, maka `coroutineScope` akan otomatis membatalkan seluruh child coroutine lainnya di dalam scope tersebut (termasuk `getAnnouncement()`) dan mengarahkan exception ke blok `catch`.

---

6. **Tantangan.** Tambahkan tombol Sort untuk mengurutkan daftar tugas berdasarkan judul. Jalankan proses pengurutannya di `Dispatchers.Default` dengan `withContext`, lalu perbarui state dengan hasilnya.
