# Script Presentasi — EcoEats Mobile (Android)
# Pertemuan ke-15 | Durasi: maks. 8 menit

---

## SLIDE 1 — Pembukaan
**~20 detik**

Selamat [pagi/siang/sore]. Pada kesempatan ini kami akan menyampaikan update progress pengembangan EcoEats, khususnya pada sisi mobile Android.

---

## SLIDE 2 — Overview Mobile EcoEats
**~45 detik**

EcoEats Mobile adalah aplikasi Android yang ditujukan untuk konsumen — pengguna yang mencari dan memesan makanan surplus dari merchant di Kota Solo. Aplikasi ini dibangun menggunakan Kotlin dengan Retrofit sebagai HTTP client untuk berkomunikasi ke backend Laravel, dan menggunakan Laravel Sanctum untuk autentikasi berbasis token.

Secara arsitektur, aplikasi Android berkomunikasi ke backend melalui REST API. Setiap request menyertakan token autentikasi di header, yang divalidasi oleh backend sebelum data dikembalikan ke aplikasi.

---

## SLIDE 3 — Progress: Done
**~60 detik**

Yang sudah selesai sampai pertemuan ini adalah sistem autentikasi, yang sudah terverifikasi berjalan secara end-to-end antara Android dan backend Laravel.

Cakupannya meliputi: halaman login dengan validasi input, pengiriman kredensial ke endpoint login via Retrofit, penerimaan Sanctum token dari backend, penyimpanan token secara persisten di perangkat, injeksi token otomatis ke setiap request berikutnya, serta alur logout yang mencabut token dari server sekaligus menghapusnya dari perangkat.

Seluruh alur ini sudah diuji dan berjalan dengan benar.

---

## SLIDE 4 — Progress: On Going & Next Step
**~50 detik**

Yang sedang dikerjakan saat ini adalah halaman katalog listing makanan surplus — menampilkan data dari API ke tampilan daftar di aplikasi. Struktur komunikasi ke endpoint sudah disiapkan, dan saat ini sedang dalam proses integrasi antara UI dan data dari backend.

Target pengerjaan hingga pertemuan berikutnya adalah menyelesaikan katalog listing, kemudian dilanjutkan dengan fitur peta lokasi merchant, sistem pemesanan, dan riwayat pesanan secara berurutan.

---

## SLIDE 5 — Kendala
**~50 detik**

Beberapa kendala yang dihadapi selama pengembangan:

Pertama, adaptasi terhadap ekosistem Android membutuhkan waktu — pola arsitektur, pengelolaan state, dan asynchronous programming di Android berbeda cukup signifikan dari pengembangan web.

Kedua, sinkronisasi dengan tim backend. Beberapa endpoint yang dibutuhkan mobile belum tersedia ketika sudah sampai di tahap implementasi, sehingga pengerjaan harus menggunakan data sementara sambil menunggu endpoint siap.

Ketiga, kecepatan iterasi Android yang lebih lambat dibanding web karena setiap perubahan memerlukan proses build ulang dan pengujian di emulator atau perangkat fisik.

---

## SLIDE 6 — Strategi Penyelesaian
**~45 detik**

Untuk mengejar target hingga akhir, strategi yang kami terapkan adalah menyelesaikan satu fitur secara penuh sebelum pindah ke fitur berikutnya — memastikan setiap fitur benar-benar bisa berjalan dengan data real sebelum melanjutkan, bukan mengerjakan banyak fitur setengah jadi.

Untuk masalah sinkronisasi API, tim mobile dan backend menggunakan dokumentasi API bersama sebagai kontrak, sehingga implementasi di kedua sisi bisa berjalan paralel dengan referensi yang sama.

---

## SLIDE 7 — Hasil / Preview
**~50 detik**

Berikut kami tunjukkan hasil yang sudah bisa berjalan saat ini.

[Tunjukkan demo login di perangkat atau emulator]

Ini adalah alur login di aplikasi EcoEats Android. User memasukkan email dan password, aplikasi mengirim request ke backend, backend memvalidasi dan mengembalikan token, token tersimpan dan user berhasil masuk ke halaman utama.

[Sesuaikan dengan apa yang bisa ditunjukkan tim]

---

## SLIDE 8 — Penutup
**~20 detik**

Demikian update progress mobile EcoEats dari kami. Autentikasi sudah selesai, dan pengerjaan fitur selanjutnya sedang berjalan. Terima kasih, kami terbuka untuk pertanyaan.

---

## ESTIMASI WAKTU

| Slide | Topik | Durasi |
|---|---|---|
| 1 | Pembukaan | 20 detik |
| 2 | Overview | 45 detik |
| 3 | Done | 60 detik |
| 4 | On Going & Next Step | 50 detik |
| 5 | Kendala | 50 detik |
| 6 | Strategi | 45 detik |
| 7 | Preview/Demo | 50 detik |
| 8 | Penutup | 20 detik |
| **Total** | | **~6 menit + buffer demo** |
