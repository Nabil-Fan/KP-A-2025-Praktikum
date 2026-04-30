# Data Dictionary — EcoEats

> Dokumen ini mendokumentasikan seluruh tabel dan kolom pada basis data platform EcoEats.
> Dibuat sebagai bagian dari artefak P4 — Praktikum Rekayasa Perangkat Lunak.

---

## Tabel: `users`

Menyimpan semua akun pengguna platform (user biasa, merchant, dan admin). Peran dibedakan menggunakan kolom `role`.

| Kolom | Tipe Data | Constraint | Keterangan |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID unik pengguna |
| `name` | VARCHAR(100) | NOT NULL | Nama lengkap pengguna |
| `email` | VARCHAR(255) | UNIQUE, NOT NULL | Email untuk login dan notifikasi |
| `password_hash` | VARCHAR(255) | NOT NULL | Password yang di-hash menggunakan bcrypt (cost factor ≥ 10) |
| `phone` | VARCHAR(20) | NULL | Nomor HP (opsional, untuk notifikasi) |
| `role` | ENUM('user', 'merchant', 'admin') | NOT NULL, DEFAULT 'user' | Peran pengguna dalam sistem |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu akun dibuat |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Waktu terakhir data diperbarui |

---

## Tabel: `merchant_profiles`

Menyimpan informasi detail dan status verifikasi mitra merchant. Setiap merchant memiliki tepat satu profil (relasi 1:1 dengan `users`).

| Kolom | Tipe Data | Constraint | Keterangan |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID unik profil merchant |
| `user_id` | INT | FK → users.id, UNIQUE, NOT NULL | Referensi ke akun pengguna |
| `business_name` | VARCHAR(150) | NOT NULL | Nama usaha kuliner |
| `business_address` | TEXT | NOT NULL | Alamat lengkap tempat usaha |
| `latitude` | DECIMAL(10,7) | NOT NULL | Koordinat lintang lokasi merchant |
| `longitude` | DECIMAL(10,7) | NOT NULL | Koordinat bujur lokasi merchant |
| `business_license_url` | VARCHAR(500) | NULL | URL file foto surat izin usaha yang diunggah |
| `halal_cert_url` | VARCHAR(500) | NULL | URL file sertifikat halal (jika ada) |
| `verification_status` | ENUM('pending', 'approved', 'rejected') | NOT NULL, DEFAULT 'pending' | Status verifikasi dokumen oleh admin |
| `verified_at` | TIMESTAMP | NULL | Waktu akun merchant disetujui admin |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu profil dibuat |

---

## Tabel: `food_listings`

Menyimpan daftar makanan surplus yang dipublikasikan oleh merchant.

| Kolom | Tipe Data | Constraint | Keterangan |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID unik listing makanan |
| `merchant_id` | INT | FK → merchant_profiles.id, NOT NULL | Referensi ke profil merchant pemilik listing |
| `name` | VARCHAR(150) | NOT NULL | Nama produk makanan surplus |
| `description` | TEXT | NULL | Deskripsi singkat produk |
| `category` | VARCHAR(50) | NOT NULL | Kategori makanan (misal: Nasi & Mie, Roti & Kue, Minuman) |
| `original_price` | DECIMAL(10,2) | NOT NULL | Harga normal produk sebelum diskon |
| `discount_price` | DECIMAL(10,2) | NOT NULL | Harga jual surplus (harga diskon) |
| `stock_qty` | INT | NOT NULL, DEFAULT 0 | Jumlah stok tersedia saat ini |
| `photo_url` | VARCHAR(500) | NULL | URL foto produk yang diunggah merchant |
| `pickup_start` | TIMESTAMP | NOT NULL | Waktu mulai pengambilan tersedia |
| `pickup_end` | TIMESTAMP | NOT NULL | Batas waktu pengambilan (deadline) |
| `status` | ENUM('available', 'unavailable', 'sold_out') | NOT NULL, DEFAULT 'available' | Status ketersediaan listing |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu listing dibuat |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Waktu terakhir listing diperbarui |

---

## Tabel: `orders`

Menyimpan transaksi pemesanan yang dilakukan user kepada merchant.

| Kolom | Tipe Data | Constraint | Keterangan |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID unik pesanan |
| `user_id` | INT | FK → users.id, NOT NULL | Referensi ke pengguna yang memesan |
| `merchant_id` | INT | FK → merchant_profiles.id, NOT NULL | Referensi ke merchant yang menerima pesanan |
| `pickup_code` | VARCHAR(10) | UNIQUE, NOT NULL | Kode unik pengambilan yang diberikan ke user setelah checkout |
| `status` | ENUM('pending', 'confirmed', 'ready', 'completed', 'rejected', 'expired') | NOT NULL, DEFAULT 'pending' | Status pesanan saat ini |
| `total_amount` | DECIMAL(10,2) | NOT NULL | Total harga yang dibayarkan user |
| `payment_method` | VARCHAR(50) | NOT NULL | Metode pembayaran yang dipilih (misal: transfer, e-wallet) |
| `payment_status` | ENUM('unpaid', 'paid', 'refunded') | NOT NULL, DEFAULT 'unpaid' | Status pembayaran |
| `ordered_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu pesanan dibuat |
| `confirmed_at` | TIMESTAMP | NULL | Waktu merchant mengkonfirmasi pesanan |
| `completed_at` | TIMESTAMP | NULL | Waktu pesanan selesai (pickup berhasil) |

---

## Tabel: `order_items`

Tabel penghubung (junction table) antara `orders` dan `food_listings`. Satu pesanan dapat berisi banyak item makanan.

| Kolom | Tipe Data | Constraint | Keterangan |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID unik record item pesanan |
| `order_id` | INT | FK → orders.id, NOT NULL | Referensi ke pesanan induk |
| `food_listing_id` | INT | FK → food_listings.id, NOT NULL | Referensi ke listing makanan yang dipesan |
| `quantity` | INT | NOT NULL, DEFAULT 1 | Jumlah porsi yang dipesan |
| `unit_price` | DECIMAL(10,2) | NOT NULL | Harga satuan pada saat transaksi (snapshot harga) |
| `subtotal` | DECIMAL(10,2) | NOT NULL | Hasil perkalian quantity × unit_price |

---

## Tabel: `merchant_verifications`

Menyimpan log riwayat keputusan verifikasi yang dilakukan admin terhadap pengajuan merchant. Mendukung audit trail.

| Kolom | Tipe Data | Constraint | Keterangan |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID unik record verifikasi |
| `merchant_id` | INT | FK → merchant_profiles.id, NOT NULL | Referensi ke profil merchant yang diajukan |
| `admin_id` | INT | FK → users.id, NOT NULL | Referensi ke akun admin yang mengambil keputusan |
| `action` | ENUM('approved', 'rejected') | NOT NULL | Keputusan admin: disetujui atau ditolak |
| `notes` | TEXT | NULL | Catatan atau alasan keputusan dari admin |
| `actioned_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu keputusan diambil |

---

## Tabel: `reviews`

Menyimpan rating dan ulasan yang diberikan user kepada merchant setelah pesanan selesai.
> **Catatan**: Fitur ini memiliki prioritas **Low** (US-14 — Won't Have pada sprint ini) dan dapat ditangguhkan ke rilis berikutnya.

| Kolom | Tipe Data | Constraint | Keterangan |
|---|---|---|---|
| `id` | INT | PK, AUTO_INCREMENT | ID unik ulasan |
| `order_id` | INT | FK → orders.id, UNIQUE, NOT NULL | Referensi ke pesanan yang diulas (satu pesanan hanya boleh satu ulasan) |
| `user_id` | INT | FK → users.id, NOT NULL | Referensi ke pengguna yang menulis ulasan |
| `merchant_id` | INT | FK → merchant_profiles.id, NOT NULL | Referensi ke merchant yang diulas |
| `rating` | TINYINT | NOT NULL, CHECK (rating BETWEEN 1 AND 5) | Nilai rating bintang 1–5 |
| `comment` | TEXT | NULL | Teks ulasan (opsional) |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Waktu ulasan dibuat |

---

## Ringkasan Relasi Antar Tabel

| Dari | Ke | Kardinalitas | Keterangan |
|---|---|---|---|
| `users` | `merchant_profiles` | 1 : 0..1 | Satu user (merchant) memiliki satu profil merchant |
| `merchant_profiles` | `food_listings` | 1 : N | Satu merchant dapat memposting banyak listing makanan |
| `users` | `orders` | 1 : N | Satu user dapat membuat banyak pesanan |
| `merchant_profiles` | `orders` | 1 : N | Satu merchant dapat menerima banyak pesanan |
| `orders` | `order_items` | 1 : N | Satu pesanan berisi satu atau lebih item |
| `food_listings` | `order_items` | 1 : N | Satu listing dapat muncul di banyak pesanan berbeda |
| `merchant_profiles` | `merchant_verifications` | 1 : N | Satu merchant bisa memiliki riwayat beberapa kali verifikasi |
| `users` (admin) | `merchant_verifications` | 1 : N | Satu admin dapat menangani banyak verifikasi |
| `orders` | `reviews` | 1 : 0..1 | Satu pesanan hanya menghasilkan satu ulasan |
| `users` | `reviews` | 1 : N | Satu user dapat menulis banyak ulasan |
| `merchant_profiles` | `reviews` | 1 : N | Satu merchant dapat menerima banyak ulasan |
