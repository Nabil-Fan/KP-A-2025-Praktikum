# Script Presentasi — EcoEats Mobile (Android)
# Pertemuan ke-15 | Durasi: maks. 8 menit

---

## SLIDE 1 — Pembukaan
**~20 detik**

Selamat [pagi/siang/sore]. Kami akan menyampaikan update progress pengembangan sisi mobile EcoEats — aplikasi Android untuk konsumen pembeli makanan surplus di platform EcoEats.

---

## SLIDE 2 — Gambaran Singkat Peran Mobile dalam EcoEats
**~40 detik**

Sebelum masuk ke progress, sedikit konteks. EcoEats terdiri dari dua platform: web untuk merchant dan admin, dan Android untuk konsumen pembeli. Pembagian ini disengaja karena kebutuhan konsumen — GPS, notifikasi, browsing saat bergerak — lebih optimal di aplikasi mobile native.

Aplikasi Android ini dibangun dengan Kotlin, berkomunikasi ke backend Laravel menggunakan Retrofit, dan autentikasi menggunakan Sanctum token yang sama dengan yang dipakai sisi web.

---

## SLIDE 3 — Status Progress (Done / On Going / Next Step)
**~90 detik**

Berikut status progress pengembangan mobile hingga pertemuan ke-15 ini.

**Yang sudah selesai:**
Sistem autentikasi sudah selesai dan terverifikasi berjalan end-to-end. User bisa login dengan email dan password, token Sanctum dikembalikan backend dan disimpan di SharedPreferences, setiap request selanjutnya sudah ter-autentikasi secara otomatis. Logout juga sudah berjalan — token dihapus dari perangkat dan dicabut dari sisi server. Integrasi Android ke Laravel untuk autentikasi ini sudah solid.

**Yang sedang dikerjakan:**
Saat ini kami sedang mengerjakan halaman katalog listing — menampilkan daftar makanan surplus aktif dari seluruh merchant dengan data dari API. Struktur Retrofit untuk endpoint listing sudah dibuat, UI sedang dalam proses.

**Next step hingga pertemuan 15/16:**
Target kami hingga akhir adalah menyelesaikan flow utama secara end-to-end: katalog listing bisa dibuka dan menampilkan data real dari backend, dilanjutkan fitur peta untuk melihat lokasi merchant, kemudian sistem pemesanan beserta kode pickup, dan riwayat pesanan. Prioritas utama adalah katalog dan peta karena keduanya adalah entry point utama pengalaman user.

---

## SLIDE 4 — Kendala yang Dihadapi
**~70 detik**

Ada beberapa kendala yang kami hadapi selama pengembangan mobile ini.

Pertama, kurva belajar Kotlin dan ekosistem Android cukup curam, terutama untuk anggota tim yang sebelumnya lebih familiar dengan pengembangan web. Setup environment Android Studio, pengelolaan dependency Gradle, dan pola arsitektur Android membutuhkan waktu adaptasi yang tidak sedikit.

Kedua, sinkronisasi dengan tim backend. Karena mobile dan web dikerjakan secara paralel, ada beberapa titik di mana endpoint API yang dibutuhkan mobile belum siap saat tim Android sudah butuh. Ini menyebabkan beberapa bagian harus dikerjakan dengan data dummy dulu sambil menunggu endpoint tersedia.

Ketiga, pengembangan aplikasi mobile secara natural lebih lambat dibanding web karena setiap perubahan butuh build ulang dan testing di emulator atau perangkat fisik. Ini berdampak pada kecepatan iterasi dibanding tim web.

Keempat, keterbatasan waktu pengerjaan karena bersamaan dengan mata kuliah dan tugas lain di semester ini.

---

## SLIDE 5 — Target dan Strategi Penyelesaian
**~60 detik**

Untuk mengejar target hingga pertemuan 15 atau 16, strategi kami adalah fokus pada flow utama user secara berurutan — selesaikan satu fitur sampai bisa diDemo sebelum lanjut ke berikutnya, daripada mengerjakan banyak fitur setengah-setengah.

Urutan prioritas: autentikasi sudah selesai, berikutnya katalog listing, lalu peta lokasi merchant, kemudian pemesanan dan kode pickup, terakhir riwayat pesanan.

Untuk mengatasi kendala sinkronisasi API, tim mobile dan backend sudah menyepakati kontrak endpoint melalui dokumentasi Scramble yang tersedia di /docs/api, sehingga tim mobile bisa mulai implementasi bahkan sebelum endpoint benar-benar siap dengan menggunakan mock response terlebih dahulu.

Realistically, target yang paling mungkin tercapai hingga pertemuan terakhir adalah katalog listing dan peta sudah bisa berjalan dengan data real. Untuk pemesanan dan riwayat, kami upayakan semaksimal mungkin dalam waktu yang tersisa.

---

## SLIDE 6 — Demo / Preview Hasil
**~50 detik**

Berikut kami tampilkan hasil yang sudah bisa dilihat saat ini.

[Tunjukkan demo login di perangkat/emulator]

Ini adalah tampilan login aplikasi EcoEats Android. Saat user memasukkan email dan password lalu menekan tombol masuk, aplikasi mengirim request ke backend Laravel. Backend memvalidasi dan mengembalikan token. Token tersimpan dan user masuk ke halaman utama aplikasi.

[Tunjukkan response di Logcat atau tampilan setelah login berhasil]

Ini membuktikan komunikasi Android ke Laravel lewat Retrofit dan Sanctum sudah berjalan dengan benar. Fondasi ini yang akan digunakan oleh seluruh fitur berikutnya.

---

## SLIDE 7 — Penutup
**~20 detik**

Demikian update progress mobile EcoEats dari kami. Autentikasi sudah selesai, katalog dan fitur lainnya dalam proses pengerjaan aktif. Kami terbuka untuk pertanyaan atau masukan.

---

## ESTIMASI WAKTU

| Slide | Topik | Durasi |
|---|---|---|
| 1 | Pembukaan | 20 detik |
| 2 | Gambaran peran mobile | 40 detik |
| 3 | Status progress | 90 detik |
| 4 | Kendala | 70 detik |
| 5 | Target dan strategi | 60 detik |
| 6 | Demo/preview | 50 detik |
| 7 | Penutup | 20 detik |
| **Total** | | **~5,5 menit + buffer demo** |

> Slide 6 bisa lebih panjang tergantung demo yang ditunjukkan. Total dengan demo aktif sekitar 7–8 menit.
