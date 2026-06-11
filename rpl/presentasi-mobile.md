# Script Presentasi — EcoEats Mobile (Android)
# Durasi: 8 menit
# Format: per slide, narasi lengkap

---

## SLIDE 1 — Opening & Identitas Proyek
**Durasi: ~45 detik**

Selamat pagi/siang/sore. Pada kesempatan ini kami akan mempresentasikan sisi mobile dari platform EcoEats — sebuah aplikasi Android yang menjadi ujung tombak pengalaman pengguna dalam menemukan dan memesan makanan surplus di Kota Solo.

Perlu kami sampaikan di awal bahwa EcoEats adalah sistem yang terdiri dari dua sisi: sisi web yang digunakan oleh merchant dan admin — yang sudah dipresentasikan sebelumnya — dan sisi mobile Android yang kami bangun khusus untuk konsumen pembeli. Hari ini kami akan fokus pada sisi mobile ini.

---

## SLIDE 2 — Mengapa Mobile?
**Durasi: ~45 detik**

Keputusan untuk menempatkan pengalaman user di mobile bukan keputusan sembarangan. Ada alasan teknis dan UX yang kuat di baliknya.

Pertama, konsumen EcoEats adalah orang yang sedang bergerak — mereka browsing makanan sambil di perjalanan, membutuhkan akses lokasi untuk menemukan merchant terdekat, dan butuh notifikasi real-time saat pesanan mereka siap diambil. Semua kebutuhan ini hanya bisa dipenuhi secara optimal oleh aplikasi mobile native.

Kedua, fitur peta interaktif dengan geolocation, kalkulasi jarak terdekat, dan navigasi ke lokasi merchant adalah fitur yang jauh lebih powerful di mobile dibanding web. Inilah mengapa kami memisahkan domain user sepenuhnya ke Android.

---

## SLIDE 3 — Tech Stack Mobile
**Durasi: ~40 detik**

Aplikasi mobile EcoEats dibangun menggunakan Kotlin sebagai bahasa utama, yang merupakan bahasa resmi Android modern yang direkomendasikan Google. Untuk komunikasi dengan backend Laravel, kami menggunakan Retrofit — library HTTP client yang sudah menjadi standar industri untuk aplikasi Android.

Autentikasi menggunakan Laravel Sanctum dengan mekanisme token-based, sehingga setiap request dari mobile membawa token yang diverifikasi oleh server. Untuk dokumentasi API, backend sudah menyiapkan Scramble yang dapat diakses di endpoint docs/api, sehingga tim mobile dan tim backend bisa bekerja dengan kontrak API yang jelas.

---

## SLIDE 4 — Arsitektur Sistem: Mobile ↔ Backend
**Durasi: ~50 detik**

Sebelum masuk ke fitur, penting untuk memahami bagaimana mobile dan backend berkomunikasi.

Aplikasi Android berkomunikasi dengan Laravel melalui REST API di routes/api.php. Setiap request dilengkapi header Authorization Bearer token yang diperoleh saat proses login. Backend memvalidasi token via Sanctum, mengambil data dari database, dan mengembalikan response dalam format JSON yang kemudian di-parse oleh Retrofit di sisi Android.

Arsitektur ini memastikan pemisahan yang bersih antara presentation layer di Android dan business logic di Laravel. Tim mobile tidak perlu tahu detail implementasi backend, cukup mengikuti kontrak API yang sudah didokumentasikan. Demikian pula sebaliknya — tim backend tidak perlu tahu bagaimana Android merender data yang dikirimkan.

Ini adalah pola arsitektur yang scalable — ke depannya jika ada platform lain seperti iOS atau web user, mereka cukup mengonsumsi API yang sama tanpa perubahan pada backend.

---

## SLIDE 5 — Fitur Autentikasi (Sudah Implementasi)
**Durasi: ~60 detik**

Fondasi dari seluruh aplikasi mobile adalah sistem autentikasi, dan inilah yang sudah berhasil kami implementasikan dan verifikasi end-to-end.

Alurnya adalah sebagai berikut: user membuka aplikasi, memasukkan email dan password, lalu Retrofit mengirim request POST ke endpoint /api/auth/login di backend Laravel. Backend memvalidasi kredensial, memverifikasi bahwa akun aktif dan memiliki role user, kemudian mengembalikan Sanctum token dalam format JSON.

Token ini disimpan secara persisten di perangkat menggunakan SharedPreferences, sehingga user tidak perlu login ulang setiap membuka aplikasi. Setiap request berikutnya — baik untuk mengambil listing, membuat pesanan, maupun melihat riwayat — secara otomatis menyertakan token ini di header Authorization.

Proses logout menghapus token dari perangkat dan memanggil endpoint /api/auth/logout di backend untuk mencabut token dari sisi server, memastikan keamanan dari dua arah.

Yang menjadi poin penting: integrasi frontend Android dengan backend Laravel untuk autentikasi ini sudah berjalan sempurna. Request masuk, token dikeluarkan, token disimpan, request berikutnya authenticated — seluruh alur ini sudah terverifikasi berfungsi.

---

## SLIDE 6 — Fitur yang Sedang Dikembangkan
**Durasi: ~55 detik**

Di atas fondasi autentikasi yang sudah solid, kami sedang mengembangkan fitur-fitur berikut secara paralel.

Yang pertama adalah katalog makanan surplus — user dapat melihat daftar semua listing aktif dari merchant yang sudah terverifikasi, dilengkapi foto, nama, harga diskon, persentase diskon, dan sisa stok. User juga bisa filter berdasarkan kategori.

Yang kedua adalah fitur peta interaktif menggunakan Leaflet Android atau Google Maps SDK — menampilkan pin lokasi merchant di sekitar user, memanfaatkan GPS perangkat untuk menghitung jarak terdekat, dan memberikan navigasi ke lokasi pickup.

Yang ketiga adalah sistem pemesanan — user dapat menambahkan item ke keranjang, memilih metode pembayaran, melakukan checkout, dan menerima kode pickup unik yang digunakan saat pengambilan di merchant.

Keempat adalah riwayat pesanan — user dapat melihat seluruh riwayat transaksi beserta status terkini dari setiap pesanan secara real-time.

---

## SLIDE 7 — Integrasi dengan Web Platform
**Durasi: ~45 detik**

Salah satu kekuatan arsitektur EcoEats adalah bagaimana sisi mobile dan web bekerja sebagai satu ekosistem yang koheren, bukan dua sistem terpisah.

Ketika user membuat pesanan di Android, notifikasi langsung masuk ke dashboard merchant di web. Merchant mengkonfirmasi pesanan dari web, dan status pesanan di aplikasi Android user berubah secara real-time. Ketika makanan siap, merchant menandai "siap diambil" dari web, dan user di Android mendapat notifikasi untuk datang ke lokasi.

Saat user tiba, mereka menunjukkan kode pickup dari Android, merchant memasukkan kode itu ke web untuk menyelesaikan transaksi. Satu alur yang mulus antara dua platform, satu database, satu sumber kebenaran.

---

## SLIDE 8 — Penutup & Demo
**Durasi: ~40 detik**

Untuk merangkum: EcoEats Android dibangun dengan Kotlin dan Retrofit di atas fondasi API Laravel Sanctum yang sudah solid. Autentikasi end-to-end sudah berjalan, kontrak API sudah terdokumentasi, dan arsitektur yang kami pilih memastikan pengembangan fitur-fitur selanjutnya dapat berjalan dengan terstruktur dan terukur.

Kami percaya bahwa fondasi yang kuat lebih penting dari fitur yang banyak tapi rapuh. Dan fondasi itulah yang sudah kami bangun.

Demikian presentasi dari tim mobile EcoEats. Kami siap menerima pertanyaan.

---

## CATATAN WAKTU (estimasi total)

| Slide | Topik | Durasi |
|---|---|---|
| 1 | Opening | 45 detik |
| 2 | Mengapa Mobile | 45 detik |
| 3 | Tech Stack | 40 detik |
| 4 | Arsitektur | 50 detik |
| 5 | Autentikasi (demo) | 60 detik |
| 6 | Fitur dalam pengembangan | 55 detik |
| 7 | Integrasi web-mobile | 45 detik |
| 8 | Penutup | 40 detik |
| **Total** | | **~8 menit** |

---

## TIPS PENYAMPAIAN

- Slide 5 adalah inti presentasi — perlambat di sini, tunjukkan demo live jika memungkinkan (buka aplikasi, login, tampilkan token yang diterima di Logcat atau response).
- Slide 6 sampaikan dengan percaya diri sebagai "roadmap yang sedang berjalan", bukan "yang belum selesai".
- Jika ada pertanyaan tentang fitur yang belum jadi, jawab: "Fitur tersebut sedang dalam pengembangan aktif dan dijadwalkan selesai pada sprint berikutnya — fondasi API-nya sudah tersedia di backend."
- Hindari kata "belum" — ganti dengan "sedang dikembangkan" atau "dalam progress".
