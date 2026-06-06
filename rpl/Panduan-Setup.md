# 🌿 Panduan Setup & Pengembangan EcoEats

Panduan ini untuk semua anggota tim yang ingin menjalankan proyek EcoEats di laptop masing-masing. Baca dari atas ke bawah, jangan skip langkah.

---

## Daftar Isi

- [Bagian 1 — Install Tools (sekali saja)](#bagian-1--install-tools-sekali-saja)
- [Bagian 2 — Clone & Setup Proyek (sekali saja)](#bagian-2--clone--setup-proyek-sekali-saja)
- [Bagian 3 — Menjalankan Proyek](#bagian-3--menjalankan-proyek)
- [Bagian 4 — Update Kode Terbaru (rutin)](#bagian-4--update-kode-terbaru-rutin)
- [Bagian 5 — Alur Kerja Harian](#bagian-5--alur-kerja-harian)
- [Bagian 6 — Roadmap Fitur Berikutnya](#bagian-6--roadmap-fitur-berikutnya)
- [Bagian 7 — Troubleshooting](#bagian-7--troubleshooting)

---

## Bagian 1 — Install Tools (sekali saja)

### 1.1 XAMPP

XAMPP dipakai untuk menjalankan MySQL (database). **Kalau sudah punya XAMPP, skip ke 1.2.**

1. Download di [https://www.apachefriends.org](https://www.apachefriends.org) — pilih versi PHP 8.x
2. Install, semua opsi default saja
3. Buka **XAMPP Control Panel**, klik **Start** di baris **MySQL**
4. Pastikan status MySQL berubah jadi hijau

> Apache tidak perlu dijalankan — Laravel punya server sendiri.

---

### 1.2 PHP

1. Buka XAMPP Control Panel → klik **Shell** (pojok kanan atas)
2. Ketik perintah ini untuk cek apakah PHP sudah bisa diakses:
   ```
   php -v
   ```
3. Kalau muncul versi PHP → **lanjut ke 1.3**
4. Kalau muncul error `'php' is not recognized` → lakukan langkah berikut:

**Tambahkan PHP ke PATH Windows:**
1. Buka **File Explorer** → pergi ke `C:\xampp\php\`
2. Salin path tersebut
3. Buka **Start Menu** → cari **"Edit the system environment variables"** → buka
4. Klik **Environment Variables** → di bagian **System variables**, cari `Path` → klik **Edit**
5. Klik **New** → paste `C:\xampp\php\` → klik OK sampai semua jendela tertutup
6. **Restart Command Prompt / PowerShell**, lalu cek ulang `php -v`

---

### 1.3 Composer

Composer adalah package manager PHP — dipakai untuk install semua library Laravel.

1. Download installer di [https://getcomposer.org/download](https://getcomposer.org/download) — klik **Composer-Setup.exe**
2. Jalankan installer
3. Saat installer tanya lokasi PHP, arahkan ke `C:\xampp\php\php.exe`
4. Selesai install, buka terminal baru dan cek:
   ```
   composer --version
   ```
5. Kalau muncul versi → Composer berhasil diinstall ✓

---

### 1.4 Git

1. Download di [https://git-scm.com/download/win](https://git-scm.com/download/win)
2. Install, semua opsi default saja
3. Cek di terminal:
   ```
   git --version
   ```

---

## Bagian 2 — Clone & Setup Proyek (sekali saja)

### 2.1 Clone Repository

Buka **Command Prompt** atau **PowerShell**, lalu jalankan:

```bash
git clone https://github.com/Nabil-Fan/praktikum-rpl-a-5.git
cd praktikum-rpl-a-5
```

---

### 2.2 Masuk ke Folder Backend

Semua perintah Laravel dijalankan dari dalam folder ini:

```bash
cd src\backend
```

> ⚠️ **Penting:** Semua perintah `php artisan` dan `composer` di panduan ini dijalankan dari `src\backend\`, bukan dari root repo.

---

### 2.3 Install Dependencies Laravel

```bash
composer install
```

Proses ini mengunduh semua library yang dibutuhkan. Tunggu sampai selesai (bisa 2–5 menit tergantung koneksi).

---

### 2.4 Buat File `.env`

```bash
copy .env.example .env
```

Lalu buka file `.env` dengan Notepad atau VS Code, cari bagian ini dan sesuaikan:

```env
APP_NAME=EcoEats
APP_URL=http://127.0.0.1:8000

APP_TIMEZONE=Asia/Jakarta

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ecoeats
DB_USERNAME=root
DB_PASSWORD=

SESSION_DRIVER=file
SESSION_LIFETIME=120
SESSION_DOMAIN=
```

> `DB_PASSWORD` dikosongkan karena XAMPP defaultnya tidak pakai password.

---

### 2.5 Generate App Key

```bash
php artisan key:generate
```

---

### 2.6 Buat Database

1. Pastikan MySQL di XAMPP sudah **Start** (statusnya hijau)
2. Buka browser → pergi ke [http://localhost/phpmyadmin](http://localhost/phpmyadmin)
3. Klik **New** di sidebar kiri
4. Isi nama database: `ecoeats`
5. Pilih collation: `utf8mb4_unicode_ci`
6. Klik **Create**

---

### 2.7 Jalankan Migrasi & Seed Data

```bash
php artisan migrate
php artisan db:seed --class=AlenaBakerySeeder
```

Migrasi akan membuat semua tabel di database. Seeder akan mengisi data awal untuk testing.

**Akun testing yang tersedia setelah seed:**

| Email | Password | Role |
|---|---|---|
| `admin@test.com` | `password` | Admin |
| `merchant@test.com` | `password` | Merchant |
| `alena.bakery@ecoeats.id` | `alena1234` | Merchant (approved + punya listing) |
| `user@test.com` | `password` | User |

---

### 2.8 Setup Storage untuk Upload File

```bash
php artisan storage:link
```

Perintah ini membuat symlink agar foto yang diupload bisa diakses dari browser. **Jalankan sekali saja.**

---

## Bagian 3 — Menjalankan Proyek

Setiap kali ingin mengerjakan atau mencoba proyek:

**Step 1** — Pastikan MySQL di XAMPP sudah **Start**

**Step 2** — Buka terminal, masuk ke folder backend:
```bash
cd praktikum-rpl-a-5\src\backend
```

**Step 3** — Jalankan server Laravel:
```bash
php artisan serve
```

**Step 4** — Buka browser di:
```
http://127.0.0.1:8000
```

> Selalu pakai `127.0.0.1:8000`, **bukan** `localhost:8000` — keduanya dianggap domain berbeda oleh browser dan akan menyebabkan masalah session/login.

Untuk menghentikan server: tekan `Ctrl + C` di terminal.

---

## Bagian 4 — Update Kode Terbaru (rutin)

Lakukan ini **setiap kali sebelum mulai kerja** untuk memastikan kode kamu selalu sinkron dengan tim.

```bash
# 1. Masuk ke folder backend
cd praktikum-rpl-a-5\src\backend

# 2. Pindah ke branch dev (branch utama pengembangan)
git checkout dev

# 3. Ambil perubahan terbaru dari GitHub
git pull origin dev

# 4. Install dependency baru jika ada (aman dijalankan berulang)
composer install

# 5. Jalankan migrasi jika ada tabel baru
php artisan migrate

# 6. Clear cache agar perubahan config terbaca
php artisan config:clear
php artisan cache:clear
```

> Kalau ada pesan seperti `CONFLICT` saat `git pull`, jangan panik — lihat [Bagian 7 — Troubleshooting](#bagian-7--troubleshooting).

---

## Bagian 5 — Alur Kerja Harian

### Cara kerja branch di proyek ini

```
main          ← kode yang sudah stabil (jangan langsung push ke sini)
└── dev       ← branch utama pengembangan, selalu sync dengan ini
    └── feature/nama-fitur   ← kerjakan fitur baru di sini
    └── fix/nama-bug         ← perbaikan bug di sini
```

### Alur membuat fitur baru

```bash
# 1. Pastikan dev sudah update
git checkout dev
git pull origin dev

# 2. Buat branch baru dari dev
git checkout -b feature/nama-fitur-kamu

# 3. Kerjakan fitur di branch ini...

# 4. Setelah selesai, add dan commit perubahan
git add .
git commit -m "feat: tambah halaman daftar pesanan merchant"

# 5. Push ke GitHub
git push origin feature/nama-fitur-kamu

# 6. Buka GitHub → buat Pull Request dari branch kamu ke dev
```

### Format commit message

Pakai format ini supaya riwayat Git mudah dibaca:

| Prefix | Kapan dipakai | Contoh |
|---|---|---|
| `feat:` | Fitur baru | `feat: tambah form registrasi merchant` |
| `fix:` | Perbaikan bug | `fix: session hilang setelah login` |
| `style:` | Perubahan tampilan saja | `style: rapikan sidebar merchant` |
| `refactor:` | Ubah kode tanpa ganti fungsi | `refactor: pindah logika ke helper` |
| `docs:` | Update dokumentasi | `docs: tambah panduan setup` |

---

## Bagian 6 — Roadmap Fitur Berikutnya

Ini urutan fitur yang akan dikerjakan sampai proyek selesai, beserta panduan teknisnya.

---

### ✅ Sudah Selesai

- Multi-portal login (user / merchant / admin)
- Registrasi user dan merchant
- Verifikasi merchant oleh admin (approve / reject / reset)
- Dashboard admin dan merchant dengan data real
- CRUD food listing oleh merchant (+ upload foto)
- Manajemen user oleh admin
- Manajemen kategori oleh admin
- Model Order, OrderItem, Payment
- Controller dan view pesanan masuk merchant (konfirmasi / tolak / siap / selesai)
- Monitoring pesanan oleh admin

---

### 🔲 P8 — Order Flow Lengkap + API Mobile Dasar

**Tujuan:** Merchant bisa menerima dan mengelola pesanan dari mobile. Tim mobile (Kotlin) bisa mulai integrasi.

**Yang perlu dibuat:**

**Backend API (untuk Kotlin):**
- `POST /api/auth/login` — login user, return Sanctum token
- `GET /api/listings` — daftar food listing aktif semua merchant
- `GET /api/listings/{id}` — detail satu listing
- `POST /api/orders` — buat pesanan baru
- `GET /api/orders` — riwayat pesanan user yang login
- `GET /api/orders/{id}` — detail pesanan + status terkini

Semua controller API ada di `app/Http/Controllers/Api/`, hanya return JSON, tidak return view Blade.

**Cara membuat API controller:**
```bash
php artisan make:controller Api/ListingController
php artisan make:controller Api/OrderController
```

**Web — update Dashboard Merchant:**
- Aktifkan stats `pending_orders` dan `completed_today` di `MerchantDashboardController` (sudah ada kode-nya, tinggal uncomment)

---

### 🔲 P9 — Peta Interaktif (Leaflet.js + OpenStreetMap)

**Tujuan:** User mobile bisa melihat lokasi merchant di peta dan estimasi jarak.

**Teknologi yang dipakai:** Leaflet.js (gratis, tanpa API key) + OpenStreetMap.

**Cara kerja:**
1. Controller ambil semua merchant dengan status `approved` yang punya listing aktif
2. Encode koordinat ke JSON, kirim ke view
3. Leaflet render peta, loop JSON untuk taruh pin di tiap merchant
4. Saat user klik "Lokasi Saya" → browser minta izin GPS → hitung jarak dengan formula Haversine
5. Urutkan merchant dari yang terdekat

**Untuk tim mobile (Kotlin):**

Tambahkan endpoint API:
```
GET /api/merchants/map
```
Return: array merchant approved dengan koordinat, nama usaha, listing aktif, dan jarak dari koordinat user (dikirim sebagai query parameter `?lat=...&lng=...`).

**Untuk web (jika diperlukan preview peta admin):**

Load Leaflet dari CDN di Blade — tidak perlu install package apapun:
```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

Koordinat merchant sudah tersimpan di kolom `latitude` dan `longitude` di tabel `merchant_profiles`. Pastikan merchant mengisi koordinat saat registrasi (sudah ada di form register merchant).

---

### 🔲 P10 — Riwayat & Pendapatan Merchant

**Tujuan:** Merchant bisa melihat ringkasan penjualan dan total pendapatan.

**Yang perlu dibuat:**
- Halaman riwayat pesanan merchant dengan filter tanggal
- Ringkasan: total pesanan selesai, total pendapatan, rata-rata per hari
- Saldo yang bisa dicairkan (dihitung dari: total `payments` status `paid` dikurangi total `withdrawals` status `completed`)

**Rumus saldo:**
```php
$saldo = Payment::whereHas('order', fn($q) => $q->where('merchant_id', $profile->id))
    ->where('status', 'paid')
    ->sum('amount')
    - Withdrawal::where('merchant_id', $profile->id)
        ->whereIn('status', ['completed', 'processing'])
        ->sum('amount');
```

---

### 🔲 P11 — Sistem Withdrawal (Pencairan Dana)

**Tujuan:** Merchant bisa ajukan pencairan saldo, admin proses dan upload bukti transfer.

**Tabel `withdrawals` sudah ada di database** — tinggal buat model dan fiturnya.

**Yang perlu dibuat:**
- Model `Withdrawal`
- Halaman ajukan withdrawal (merchant) — isi nominal, nama bank, nomor rekening
- Halaman daftar withdrawal (admin) — approve/reject, upload bukti transfer
- Validasi: nominal tidak boleh melebihi saldo tersedia

**Model Withdrawal:**
```bash
php artisan make:model Withdrawal
```

---

### 🔲 P12 — Review & Rating (Prioritas Low)

**Tujuan:** User bisa beri rating dan ulasan ke merchant setelah pesanan selesai.

**Catatan SRS:** FR-14 prioritas Low — bisa ditunda ke rilis terakhir jika waktu terbatas.

**Yang perlu dibuat:**
- Tabel `reviews` (belum ada di DB — perlu migration baru)
- Endpoint API `POST /api/orders/{id}/review`
- Tampilan rating di profil merchant (web)

**Migration:**
```bash
php artisan make:migration create_reviews_table
```

---

### 🔲 Final — Polish & Deployment

**Sebelum demo/submit:**

1. **Cek semua konvensi kode:**
   - Semua query tabel `users` dan `food_listings` ada filter `whereNull('deleted_at')` ✓
   - Tidak ada `withTrashed()` di model `User` ✓
   - `order_items` pakai snapshot `listing_name` dan `unit_price`, bukan relasi ✓

2. **Cek responsivitas** — buka Chrome DevTools, simulasi layar 375px, pastikan tidak ada elemen terpotong (NFR-03 di SRS)

3. **Cek performa** — buka Lighthouse di Chrome DevTools, pastikan halaman utama load < 3 detik (NFR-01)

4. **Environment production:**
   ```env
   APP_ENV=production
   APP_DEBUG=false
   ```

5. **Jalankan seeder ulang dengan data yang lebih lengkap** untuk demo

---

## Bagian 7 — Troubleshooting

### ❌ `'php' is not recognized`
PHP belum masuk PATH Windows. Ikuti langkah 1.2 di atas untuk menambahkan `C:\xampp\php\` ke PATH.

---

### ❌ `'composer' is not recognized`
Composer belum terinstall. Ikuti langkah 1.3.

---

### ❌ `SQLSTATE: Access denied` atau `Connection refused`
MySQL di XAMPP belum jalan. Buka XAMPP Control Panel → klik **Start** di baris MySQL.

---

### ❌ `Class not found` setelah pull
Ada file baru yang ditambahkan tim. Jalankan:
```bash
composer dump-autoload
```

---

### ❌ Error setelah pull tapi tidak ada file baru
Mungkin ada perubahan config atau migration baru. Jalankan:
```bash
php artisan config:clear
php artisan cache:clear
php artisan migrate
```

---

### ❌ Login berhasil tapi langsung kembali ke halaman login
Masalah session. Pastikan:
1. Browser buka di `http://127.0.0.1:8000` bukan `http://localhost:8000`
2. File `.env` punya `SESSION_DOMAIN=` (kosong, bukan `null`)
3. Jalankan `php artisan config:clear`

---

### ❌ Foto/gambar yang diupload tidak muncul
Symlink storage belum dibuat. Jalankan:
```bash
php artisan storage:link
```

---

### ❌ `CONFLICT` saat `git pull`

Ini terjadi kalau kamu dan teman mengedit file yang sama. Jangan panik:

1. Buka file yang konflik (Git akan menandai dengan `<<<<<<<`, `=======`, `>>>>>>>`)
2. Pilih kode mana yang benar (atau gabungkan keduanya)
3. Hapus marker `<<<<<<<`, `=======`, `>>>>>>>`
4. Save file
5. Jalankan:
   ```bash
   git add .
   git commit -m "fix: resolve merge conflict"
   ```

Kalau bingung, hubungi anggota yang ngerti Git sebelum asal-asalan.

---

### ❌ `Integrity constraint violation` saat migrate
Ada data yang sudah ada di DB yang melanggar constraint baru. Solusi paling aman untuk lokal:
```bash
php artisan migrate:fresh --seed
```
> ⚠️ Perintah ini menghapus SEMUA data dan buat ulang dari awal. Hanya lakukan di lokal, tidak pernah di server produksi.

---

## Catatan Penting

> **Branch `dev` adalah branch utama pengembangan.** Jangan push langsung ke `main`. Semua fitur baru dikerjakan di branch `feature/...` lalu di-merge ke `dev` lewat Pull Request setelah di-review.

> **Jangan commit file `.env`** — file ini sudah ada di `.gitignore` dan berisi informasi sensitif (database password, app key).

> **Selalu `git pull origin dev` sebelum mulai kerja** — ini mencegah konflik besar di akhir.
