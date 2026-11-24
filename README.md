# Kotlin Coroutines – Pemrograman Asinkron

## Deskripsi Project

Project ini merupakan implementasi latihan Kotlin Coroutines pada aplikasi Android. Tujuannya adalah memahami konsep pemrograman asinkron, menjalankan beberapa proses secara bersamaan, menangani error dan pembatalan coroutine, serta mengelola proses komputasi agar tidak mengganggu responsivitas antarmuka pengguna (UI).

## Fitur dan Implementasi

### 1. Menambahkan Permintaan `getCourseCount()`

Menambahkan proses pengambilan jumlah course menggunakan Kotlin Coroutines.

**Implementasi:**
- `TaskRepository.kt`: Menambahkan fungsi `suspend fun getCourseCount(): Int` dengan simulasi waktu proses menggunakan `delay(1500L)`.
- `TasksUiState.kt`: Menambahkan properti `courseCount: Int = 0`.
- `TasksViewModel.kt`: Menjalankan `getCourseCount()` menggunakan `async` dan mengambil hasilnya dengan `await()`.
- `TasksScreen.kt`: Menampilkan informasi jumlah course jika nilainya lebih dari 0.

**Hasil dan Analisis:**

Total waktu pemuatan tetap sekitar 2 detik karena ketiga proses dijalankan secara bersamaan menggunakan `async` di dalam `coroutineScope`.

| Proses | Waktu |
|---|---:|
| `getTasks()` | 2000 ms |
| `getAnnouncement()` | 1000 ms |
| `getCourseCount()` | 1500 ms |

Total waktu ditentukan oleh proses yang paling lama, bukan penjumlahan seluruh durasi.

### 2. Menambahkan Tombol Retry

Menambahkan tombol Retry untuk mengulangi proses pemuatan data ketika terjadi kesalahan.

**Implementasi:**
- Pesan error ditampilkan ketika `state.errorMessage` tidak bernilai `null`.
- Pesan error dan tombol Retry ditempatkan dalam sebuah `Row`.
- Tombol Retry memanggil `onLoad`, yang menjalankan kembali `viewModel.loadData()`.

**Hasil:**

Pengguna dapat mencoba memuat data kembali tanpa harus menjalankan ulang aplikasi.

### 3. Menangkap Terlalu Banyak Exception

Menguji perubahan penanganan error dari `catch (e: IOException)` menjadi `catch (e: Exception)`.

**Hasil Pengujian:**

Ketika tombol Load ditekan lalu segera diikuti Cancel, pesan error dapat muncul seperti:

`Failed to load: StandaloneCoroutine was cancelled`

**Analisis:**

Pembatalan coroutine pada Kotlin menggunakan `CancellationException`. Karena exception tersebut merupakan turunan dari `Exception`, penggunaan `catch (e: Exception)` dapat menangkap pembatalan coroutine.

Akibatnya, pembatalan yang seharusnya berlangsung secara normal justru diperlakukan sebagai kesalahan biasa dan dapat menampilkan pesan error pada UI.

### 4. Perbandingan `Thread.sleep()` dan `withContext()`

Melakukan pengujian untuk memahami pengaruh pemilihan thread terhadap responsivitas aplikasi.

**Skenario A: `Thread.sleep()` di dalam `Dispatchers.IO`**

`Thread.sleep(2000L)` dijalankan di dalam `withContext(Dispatchers.IO)`.

Hasil: UI tetap responsif karena thread yang diblokir adalah thread latar belakang, bukan Main Thread.

**Skenario B: `Thread.sleep()` tanpa `withContext(Dispatchers.IO)`**

`withContext(Dispatchers.IO)` dihapus sehingga `Thread.sleep(2000L)` dijalankan pada Main Thread.

Hasil: UI mengalami freeze selama sekitar 2 detik karena Main Thread tidak dapat memproses interaksi pengguna maupun memperbarui tampilan.

**Kesimpulan:**

Operasi yang berpotensi memblokir thread harus dijalankan pada dispatcher yang sesuai agar tidak mengganggu responsivitas aplikasi.

### 5. Pertanyaan Konsep Kotlin Coroutines

#### A. Perbedaan `launch` dan `async`

| `launch` | `async` |
|---|---|
| Menjalankan coroutine tanpa hasil return yang perlu diambil. | Menjalankan coroutine yang menghasilkan nilai. |
| Mengembalikan `Job`. | Mengembalikan `Deferred`. |
| Tidak menggunakan `await()`. | Menggunakan `await()` untuk mengambil hasil. |

#### B. Mengapa `GlobalScope` Sebaiknya Dihindari?

`GlobalScope` tidak terikat pada lifecycle Activity atau ViewModel. Coroutine yang dijalankan dapat terus berlangsung meskipun layar sudah ditutup.

Hal tersebut dapat menyebabkan pemborosan sumber daya dan pekerjaan latar belakang yang tidak diperlukan.

Karena itu, lebih disarankan menggunakan scope yang terikat pada lifecycle, seperti `viewModelScope`.

#### C. Apa yang Terjadi Jika `getTasks()` Gagal?

Berdasarkan prinsip Structured Concurrency, ketika salah satu child coroutine mengalami exception, coroutine lain yang berada dalam scope yang sama dapat ikut dibatalkan.

Dengan demikian, jika `getTasks()` gagal, proses `getAnnouncement()` juga dapat dibatalkan dan exception diteruskan ke parent `coroutineScope`.

### 6. Tantangan: Menambahkan Tombol Sort

Menambahkan fitur untuk mengurutkan daftar tugas berdasarkan judul (*title*) secara alfabetis.

**Implementasi:**
- `TasksViewModel.kt`: Menambahkan fungsi `sortTasks()`.
- Menggunakan `withContext(Dispatchers.Default)` untuk menjalankan proses pengurutan.
- Menggunakan `sortedBy { it.title }` untuk mengurutkan daftar tugas.
- Memperbarui UI State menggunakan `_uiState.update`.
- `TasksScreen.kt`: Menambahkan tombol Sort yang aktif ketika daftar tugas sudah dimuat.

**Contoh Implementasi:**

```kotlin
fun sortTasks() {
    viewModelScope.launch {
        val currentTasks = _uiState.value.tasks

        if (currentTasks.isNotEmpty()) {
            val sorted = withContext(Dispatchers.Default) {
                currentTasks.sortedBy { it.title }
            }

            _uiState.update {
                it.copy(tasks = sorted)
            }
        }
    }
}
```

**Hasil:**

Ketika tombol Sort ditekan, daftar tugas akan diurutkan berdasarkan judul secara alfabetis tanpa membebani Main Thread.

## Teknologi yang Digunakan

- **Kotlin** – Bahasa pemrograman utama.
- **Android Studio** – IDE pengembangan aplikasi Android.
- **Kotlin Coroutines** – Menangani proses asinkron.
- **ViewModel** – Mengelola data dan state UI.
- **StateFlow** – Mengelola serta mengamati perubahan UI State.
- **Jetpack Compose** – Membangun antarmuka pengguna.

## Kesimpulan

Melalui latihan ini, konsep Kotlin Coroutines diterapkan untuk menjalankan proses asinkron secara efisien, menangani exception dan cancellation, serta menjaga responsivitas aplikasi Android.

Penggunaan `async`, `launch`, `withContext`, dan `viewModelScope` membantu mengelola pekerjaan berdasarkan kebutuhan dan lifecycle aplikasi. Selain itu, penggunaan dispatcher yang tepat dapat mencegah proses berat menghambat Main Thread.
