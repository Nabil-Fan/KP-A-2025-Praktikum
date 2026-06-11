# Script Presentasi — EcoEats Mobile (Android)
# Pertemuan ke-15 | Durasi: maks. 8 menit

---

## SLIDE 1 — Pembukaan
**~25 detik**

Selamat [pagi/siang/sore]. Kami dari tim mobile EcoEats akan menyampaikan update progress pengembangan aplikasi Android EcoEats, yang merupakan platform sisi konsumen dari ekosistem EcoEats — melengkapi sisi web yang sudah dipresentasikan oleh tim backend dan frontend sebelumnya.

---

## SLIDE 2 — Peran Mobile dalam Ekosistem EcoEats
**~50 detik**

EcoEats sebagai platform terdiri dari dua layer: web untuk operasional merchant dan admin, dan Android untuk pengalaman konsumen pembeli. Keputusan untuk memisahkan domain user ke mobile bukan sekadar pembagian kerja — ada pertimbangan teknis di baliknya.

Fitur inti yang dibutuhkan konsumen EcoEats secara fundamental lebih cocok di mobile: geolocation untuk mendeteksi posisi user dan menghitung jarak ke merchant terdekat, push notification untuk memberi tahu user ketika status pesanan berubah, dan akses kamera untuk verifikasi kode pickup. Ketiganya memerlukan akses ke hardware perangkat yang jauh lebih mudah dan optimal diimplementasikan di native Android dibandingkan di web browser.

Aplikasi ini dibangun dengan Kotlin sebagai bahasa utama, Retrofit sebagai HTTP client untuk komunikasi ke backend Laravel, dan Laravel Sanctum sebagai mekanisme autentikasi berbasis token.

---

## SLIDE 3 — Arsitektur Komunikasi Android ↔ Laravel
**~60 detik**

Komunikasi antara aplikasi Android dan backend Laravel berjalan melalui REST API yang didefinisikan di routes/api.php. Setiap request dari Android dikirim oleh Retrofit dengan menyertakan token Sanctum di header Authorization dalam format Bearer token.

Di sisi Android, Retrofit dikonfigurasi dengan OkHttp interceptor yang secara otomatis menambahkan header Authorization ke setiap request — sehingga layer UI dan ViewModel tidak perlu menangani token secara manual di setiap pemanggilan API. Token itu sendiri disimpan secara persisten di SharedPreferences dan dibaca saat aplikasi pertama kali diinisialisasi.

Di sisi backend, middleware auth:sanctum pada setiap route API memvalidasi token sebelum request diproses. Jika token tidak valid atau sudah dicabut, backend mengembalikan response 401 dan aplikasi Android mengarahkan user kembali ke halaman login.

Untuk dokumentasi kontrak API antara tim mobile dan backend, kami menggunakan Scramble yang dapat diakses di /docs/api — sehingga kedua tim bisa bekerja paralel dengan referensi endpoint yang sama.

---

## SLIDE 4 — Progress: Done / On Going / Next Step
**~80 detik**

**Sudah selesai:**
Sistem autentikasi sudah selesai dan terverifikasi berjalan end-to-end. Ini mencakup halaman login dengan validasi input di sisi client, pengiriman kredensial ke endpoint POST /api/auth/login via Retrofit, penerimaan dan penyimpanan Sanctum token di SharedPreferences, injeksi token otomatis ke setiap request berikutnya melalui OkHttp interceptor, serta alur logout yang mencabut token dari server melalui POST /api/auth/logout sekaligus menghapusnya dari perangkat. Seluruh alur ini sudah diuji dan berjalan dengan benar.

**Sedang dikerjakan:**
Halaman katalog listing makanan surplus. Struktur Retrofit untuk endpoint GET /api/listings sudah dibuat, model data sudah didefinisikan, RecyclerView untuk menampilkan daftar listing sedang dalam proses implementasi. Saat ini UI sudah terbentuk namun koneksi ke data real dari API masih dalam proses integrasi.

**Target hingga pertemuan 15/16:**
Menyelesaikan katalog listing dengan data real, kemudian fitur peta lokasi merchant menggunakan Maps SDK dengan kalkulasi jarak berbasis koordinat GPS, dilanjutkan sistem pemesanan dan verifikasi kode pickup, dan terakhir halaman riwayat pesanan.

---

## SLIDE 5 — Kendala
**~60 detik**

Kendala pertama yang paling signifikan adalah kurva belajar ekosistem Android. Pola arsitektur Android — seperti pengelolaan lifecycle Activity dan Fragment, penggunaan ViewModel dan LiveData, serta asynchronous programming dengan Kotlin Coroutine — membutuhkan waktu adaptasi yang cukup panjang, terutama bagi anggota tim yang sebelumnya lebih banyak bekerja di lingkungan web.

Kendala kedua adalah sinkronisasi dengan tim backend. Beberapa endpoint API yang dibutuhkan mobile belum tersedia ketika tim Android sudah sampai di tahap implementasi, sehingga pengerjaan harus menggunakan mock data sementara dan baru diintegrasikan setelah endpoint siap.

Kendala ketiga adalah kecepatan iterasi. Development Android secara inheren lebih lambat dari web karena setiap perubahan memerlukan proses build Gradle yang memakan waktu, ditambah pengujian di emulator atau perangkat fisik yang tidak selalu konsisten hasilnya.

---

## SLIDE 6 — Strategi Penyelesaian
**~50 detik**

Strategi kami untuk mengejar target hingga pertemuan terakhir adalah mengerjakan fitur secara vertikal — satu fitur diselesaikan sampai benar-benar bisa dijalankan sebelum pindah ke fitur berikutnya, daripada mengerjakan banyak fitur secara horizontal tapi tidak ada yang sampai selesai.

Untuk mengatasi masalah sinkronisasi API, kami menetapkan kontrak endpoint lebih awal melalui dokumentasi Scramble dan menggunakan dummy response selama endpoint belum siap, sehingga UI dan logic di Android bisa dikerjakan paralel tanpa harus menunggu backend.

Target realistis yang kami tetapkan: katalog listing dan peta bisa berjalan dengan data real sebelum pertemuan terakhir. Untuk pemesanan dan riwayat pesanan, kami upayakan semaksimal mungkin dalam waktu yang tersisa.

---

## SLIDE 7 — Demo / Preview
**~50 detik**

Berikut kami demonstrasikan hasil yang sudah bisa berjalan saat ini.

[Buka aplikasi di perangkat/emulator]

Ini adalah tampilan login EcoEats Android. Ketika user memasukkan email dan password yang terdaftar, aplikasi mengirim request POST ke /api/auth/login di backend Laravel melalui Retrofit. Backend memvalidasi kredensial, memverifikasi role user, dan mengembalikan Sanctum token dalam response JSON. Token tersimpan di SharedPreferences dan user diarahkan ke halaman utama.

[Tunjukkan Logcat atau response JSON jika memungkinkan]

Ini membuktikan bahwa komunikasi antara Android dan Laravel sudah berjalan — Retrofit berhasil mengirim request dengan format yang benar, backend merespons dengan token yang valid, dan aplikasi berhasil memprosesnya. Fondasi inilah yang menjadi dasar untuk seluruh fitur yang sedang dan akan dikerjakan.

---

## SLIDE 8 — Penutup
**~20 detik**

Demikian update progress mobile EcoEats dari kami. Autentikasi sudah selesai end-to-end, dan pengerjaan fitur berikutnya sedang berjalan. Kami terbuka untuk pertanyaan.

---

## ESTIMASI WAKTU

| Slide | Topik | Durasi |
|---|---|---|
| 1 | Pembukaan | 25 detik |
| 2 | Peran mobile | 50 detik |
| 3 | Arsitektur | 60 detik |
| 4 | Progress done/ongoing/next | 80 detik |
| 5 | Kendala | 60 detik |
| 6 | Strategi penyelesaian | 50 detik |
| 7 | Demo/preview | 50 detik |
| 8 | Penutup | 20 detik |
| **Total** | | **~7 menit + buffer demo** |
