# Panduan Setup Backend EcoEats (Laravel 11)

> Dokumen ini adalah panduan resmi setup backend untuk proyek EcoEats.
> Semua anggota tim **wajib** mengikuti langkah ini agar environment konsisten.
> Letakkan file ini di `docs/setup-backend.md` pada repo.

---

## Prasyarat

Pastikan semua anggota tim menggunakan versi yang **sama persis**:

| Tools | Versi yang Digunakan |
|---|---|
| PHP | 8.2.x |
| Composer | 2.x |
| Laravel | 11.x |
| MySQL | 8.0.x (via XAMPP) |
| XAMPP | 8.2.x |
| Node.js | 20.x LTS (untuk Vite asset) |

> Cek versi PHP kamu: `php -v`
> Cek versi Composer: `composer -V`
> Jika belum install, download XAMPP di https://www.apachefriends.org

---

## Langkah 1 — Clone Repository

```bash
git clone <url-repo-tim>
cd PrakRpl
```

---

## Langkah 2 — Inisialisasi Project Laravel

Buat project Laravel baru di dalam folder `src/backend/`:

```bash
composer create-project laravel/laravel src/backend "^11.0"
cd src/backend
```

> **Catatan:** Jika repo sudah ada folder `src/backend/`, skip langkah ini dan langsung ke Langkah 3.

---

## Langkah 3 — Konfigurasi File `.env`

Salin file `.env.example` menjadi `.env`:

```bash
cp .env.example .env
```

Kemudian generate application key:

```bash
php artisan key:generate
```

Edit file `.env` dan sesuaikan nilai berikut dengan environment lokal kamu:

```
DB_DATABASE=ecoeats
DB_USERNAME=root
DB_PASSWORD=
```

> **XAMPP default:** username `root`, password kosong.
> Jangan pernah commit file `.env` ke repo!

---

## Langkah 4 — Buat Database di XAMPP

1. Buka XAMPP Control Panel, start **Apache** dan **MySQL**
2. Buka browser, akses `http://localhost/phpmyadmin`
3. Klik **New** di sidebar kiri
4. Buat database baru dengan nama: `ecoeats`
5. Collation: `utf8mb4_unicode_ci`
6. Klik **Create**

---

## Langkah 5 — Install Dependencies

```bash
cd src/backend
composer install
npm install
```

---

## Langkah 6 — Jalankan Migrasi

Pastikan XAMPP MySQL sudah running, lalu jalankan:

```bash
php artisan migrate
```

Output yang diharapkan:

```
INFO  Running migrations.
  2024_01_01_000001_create_users_table ............. DONE
  2024_01_01_000002_create_categories_table ........ DONE
  2024_01_01_000003_create_merchant_profiles_table . DONE
  2024_01_01_000004_create_food_listings_table ..... DONE
  2024_01_01_000005_create_orders_table ............ DONE
  2024_01_01_000006_create_payments_table .......... DONE
  2024_01_01_000007_create_order_items_table ........ DONE
  2024_01_01_000008_create_merchant_verifications_table . DONE
```

Jika ada error koneksi, pastikan:
- MySQL di XAMPP sudah running (lampu hijau)
- Nilai `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME` di `.env` sudah benar

---

## Langkah 7 — Jalankan Development Server

```bash
php artisan serve
```

Akses di browser: `http://localhost:8000`

---

## Langkah 8 — Verifikasi Skema Database

Setelah migrasi berhasil, buka phpMyAdmin dan pastikan tabel berikut sudah terbentuk:

- `users`
- `categories`
- `merchant_profiles`
- `food_listings`
- `orders`
- `payments`
- `order_items`
- `merchant_verifications`
- `migrations` (tabel internal Laravel)

---

## Struktur Folder src/backend yang Relevan

```
src/backend/
├── app/
│   └── Models/          ← Model Eloquent akan dibuat di sini
├── database/
│   └── migrations/      ← Semua file migrasi ada di sini
├── routes/
│   ├── api.php          ← Endpoint API didefinisikan di sini
│   └── web.php
├── .env                 ← JANGAN dicommit! (ada di .gitignore)
├── .env.example         ← Template env — wajib dicommit
└── .gitignore
```

---

## Alur Git untuk Task Ini (PR Setup)

```bash
# 1. Buat branch baru dari dev
git checkout dev
git pull origin dev
git checkout -b setup/project-init

# 2. Tambahkan file yang dibutuhkan
#    - src/backend/ (seluruh project Laravel, kecuali .env & vendor)
#    - .gitignore (pastikan .env dan vendor/ sudah masuk)
#    - docs/setup-backend.md (file ini)

# 3. Commit
git add .
git commit -m "feat: initialize Laravel 11 backend with database migrations"

# 4. Push dan buka PR ke branch dev
git push origin setup/project-init
```

Buka PR di GitHub ke branch `dev` (bukan `main`), lalu minta review dari anggota tim.

---

## Troubleshooting Umum

| Masalah | Solusi |
|---|---|
| `php: command not found` | Tambahkan path PHP XAMPP ke environment variable sistem |
| `SQLSTATE: Connection refused` | Pastikan MySQL di XAMPP sudah running |
| `Target class [xxxController] does not exist` | Jalankan `composer dump-autoload` |
| `No application encryption key has been specified` | Jalankan `php artisan key:generate` |
| Migrasi error kolom sudah ada | Jalankan `php artisan migrate:fresh` (hati-hati: hapus semua data!) |
