# Panduan Kontribusi — EcoEats

Dokumen ini ditujukan untuk seluruh anggota tim yang ingin berkontribusi pada proyek EcoEats. Tidak perlu pengalaman Laravel sebelumnya — baca dari atas ke bawah sebelum mulai mengerjakan bagian kalian.

---

## Daftar Isi

1. [Cara Kerja Proyek Ini (Gambaran Besar)](#1-cara-kerja-proyek-ini-gambaran-besar)
2. [Cara Backend Laravel Bekerja](#2-cara-backend-laravel-bekerja)
3. [Struktur Folder yang Perlu Kamu Tahu](#3-struktur-folder-yang-perlu-kamu-tahu)
4. [Setup Awal di Komputer Kamu](#4-setup-awal-di-komputer-kamu)
5. [Alur Kerja Git (Wajib Diikuti)](#5-alur-kerja-git-wajib-diikuti)
6. [Cara Membuat Fitur Baru](#6-cara-membuat-fitur-baru)
7. [Pembagian Tugas Per Anggota](#7-pembagian-tugas-per-anggota)
8. [Aturan & Konvensi Kode](#8-aturan--konvensi-kode)
9. [Pertanyaan Umum (FAQ)](#9-pertanyaan-umum-faq)

---

## 1. Cara Kerja Proyek Ini (Gambaran Besar)

EcoEats adalah platform web yang menghubungkan **merchant** (penjual makanan surplus) dengan **user** (pembeli). Ada juga **admin** yang mengawasi platform.

```
Browser User/Merchant/Admin
         │
         │  Buka URL (misal: /admin/verifikasi)
         ▼
    Laravel (Backend)
         │
         ├── Cek: siapa yang login? Role apa?
         ├── Ambil data dari Database (MySQL)
         └── Kirim halaman HTML ke browser
                  │
                  ▼
           Tampil di Browser
           (halaman yang dibuat dengan Blade)
```

Sistem ini dibagi tiga peran:

| Peran | Akses | Contoh Halaman |
|---|---|---|
| **User** | Lihat makanan, pesan, lihat riwayat | `/dashboard`, `/food-listings` |
| **Merchant** | Kelola listing makanan, lihat pesanan | `/merchant/dashboard`, `/merchant/food-listings` |
| **Admin** | Verifikasi merchant, pantau platform | `/admin/dashboard`, `/admin/verifikasi` |

Masing-masing peran punya halaman login terpisah:
- User → `/login`
- Merchant → `/merchant/login`
- Admin → `/admin/login`

---

## 2. Cara Backend Laravel Bekerja

### Konsep Dasar: Request → Controller → View

Ketika seseorang membuka URL di browser, Laravel mengikuti alur ini:

```
1. Browser buka URL
   contoh: GET /merchant/food-listings

2. Laravel cek routes/web.php
   "URL ini ditangani oleh controller mana?"

3. Controller dijalankan
   Ambil data dari database, proses logika bisnis

4. Controller kirim data ke View (Blade)
   return view('merchant.food-listings.index', ['listings' => $data]);

5. Blade render HTML
   Data dari controller ditampilkan di halaman HTML

6. HTML dikirim ke browser
   User melihat hasilnya
```

### Route

File `routes/web.php` adalah "peta" semua URL yang ada di aplikasi. Contoh:

```php
// Ketika user buka /merchant/food-listings (GET),
// jalankan method index() di MerchantFoodListingController
Route::get('/merchant/food-listings', [FoodListingController::class, 'index']);

// Ketika user submit form tambah listing (POST),
// jalankan method store()
Route::post('/merchant/food-listings', [FoodListingController::class, 'store']);
```

### Controller

Controller adalah "otak" yang menangani logika. Tempatnya di `app/Http/Controllers/`. Contoh sederhana:

```php
class FoodListingController extends Controller
{
    // Dipanggil saat buka /merchant/food-listings
    public function index()
    {
        // Ambil data dari database
        $listings = FoodListing::where('merchant_id', auth()->user()->merchantProfile->id)
                                ->whereNull('deleted_at')
                                ->get();

        // Kirim ke file Blade resources/views/merchant/food-listings/index.blade.php
        return view('merchant.food-listings.index', [
            'listings' => $listings
        ]);
    }
}
```

### Model

Model adalah "penghubung" antara kode PHP dan tabel database. Setiap tabel punya Model sendiri:

| Tabel Database | Model |
|---|---|
| `users` | `app/Models/User.php` |
| `food_listings` | `app/Models/FoodListing.php` |
| `merchant_profiles` | `app/Models/MerchantProfile.php` |
| `orders` | `app/Models/Order.php` |
| `categories` | `app/Models/Category.php` |

Contoh cara pakai Model untuk ambil data:

```php
// Ambil semua listing yang masih aktif
$listings = FoodListing::whereNull('deleted_at')->get();

// Ambil listing tertentu berdasarkan ID
$listing = FoodListing::find(1);

// Simpan data baru
$listing = new FoodListing();
$listing->name = 'Nasi Goreng Surplus';
$listing->price = 10000;
$listing->save();
```

### View (Blade)

View adalah file HTML yang ditampilkan ke user. Tempatnya di `resources/views/`. Laravel pakai templating engine bernama **Blade** yang memungkinkan kita menulis PHP di dalam HTML:

```html
{{-- Ini adalah komentar Blade --}}

{{-- Tampilkan variabel --}}
<h1>{{ $listing->name }}</h1>

{{-- Perulangan --}}
@foreach ($listings as $listing)
    <div>{{ $listing->name }} - Rp{{ $listing->discount_price }}</div>
@endforeach

{{-- Kondisi --}}
@if ($listing->status === 'available')
    <span>Tersedia</span>
@else
    <span>Habis</span>
@endif

{{-- Extend layout --}}
@extends('layouts.app')
@section('content')
    <p>Konten halaman di sini</p>
@endsection
```

---

## 3. Struktur Folder yang Perlu Kamu Tahu

Project Laravel ada di `src/backend/`. Ini folder-folder yang paling sering kamu sentuh:

```
src/backend/
│
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/
│   │   │   │   ├── LoginController.php       ← login (jangan diubah)
│   │   │   │   └── RegisterController.php    ← registrasi
│   │   │   ├── Admin/
│   │   │   │   ├── DashboardController.php
│   │   │   │   ├── MerchantVerificationController.php
│   │   │   │   ├── FoodListingController.php
│   │   │   │   └── UserController.php
│   │   │   └── Merchant/
│   │   │       ├── DashboardController.php
│   │   │       ├── FoodListingController.php
│   │   │       └── ProfileController.php
│   │   └── Middleware/
│   │       └── RoleMiddleware.php            ← proteksi role (jangan diubah)
│   └── Models/
│       ├── User.php                          ← model user (jangan diubah sembarangan)
│       ├── FoodListing.php
│       ├── MerchantProfile.php
│       ├── Category.php
│       └── MerchantVerification.php
│
├── resources/
│   └── views/                               ← semua file HTML (Blade) ada di sini
│       ├── layouts/
│       │   └── app.blade.php                ← layout utama (header, sidebar, dll)
│       ├── auth/
│       │   ├── login-user.blade.php
│       │   ├── login-merchant.blade.php
│       │   └── login-admin.blade.php
│       ├── admin/
│       │   ├── dashboard/
│       │   ├── merchant-verification/
│       │   ├── food-listings/
│       │   └── users/
│       └── merchant/
│           ├── dashboard/
│           ├── food-listings/
│           └── profile/
│
├── routes/
│   ├── web.php                              ← semua URL/route web
│   └── api.php                             ← URL untuk mobile (belum dipakai)
│
├── database/
│   └── migrations/                         ← struktur tabel database
│
└── .env                                    ← konfigurasi (DB, dll) — JANGAN di-commit
```

---

## 4. Setup Awal di Komputer Kamu

Lakukan ini **sekali saja** saat pertama kali clone repo.

### Prasyarat

Pastikan sudah terinstall:
- **PHP 8.2+** — cek dengan `php -v`
- **Composer** — cek dengan `composer -v`
- **Git** — cek dengan `git -v`
- **XAMPP** atau **Laragon** (untuk MySQL dan phpMyAdmin)

### Langkah Setup

**1. Clone repo**
```bash
git clone https://github.com/[nama-org]/Praktikum-rpl-a-5.git
cd Praktikum-rpl-a-5/src/backend
```

**2. Install dependencies PHP**
```bash
composer install
```

**3. Salin file konfigurasi**
```bash
cp .env.example .env
```

**4. Generate app key**
```bash
php artisan key:generate
```

**5. Isi konfigurasi database di `.env`**

Buka file `.env`, cari bagian ini dan sesuaikan:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ecoeats        ← nama database yang sudah dibuat di phpMyAdmin
DB_USERNAME=root           ← username MySQL kamu (biasanya root)
DB_PASSWORD=               ← password MySQL kamu (biasanya kosong di XAMPP)
```

**6. Import database**

Buka phpMyAdmin → buat database baru bernama `ecoeats` → Import file SQL yang ada di repo (`docs/` atau tanya ketua kelompok).

**7. Jalankan server**
```bash
php artisan serve
```

Buka browser → `http://localhost:8000` → seharusnya sudah jalan.

---

## 5. Alur Kerja Git (Wajib Diikuti)

### Aturan Branch

| Branch | Fungsi |
|---|---|
| `main` | Kode final yang sudah stabil. **Jangan push langsung ke sini.** |
| `dev` | Branch pengembangan utama. Semua fitur di-merge ke sini dulu. |
| `feature/nama-fitur` | Branch untuk mengerjakan fitur spesifik. Dibuat dari `dev`. |

### Alur Setiap Kali Mengerjakan Fitur

```bash
# 1. Pastikan branch dev kamu up-to-date
git checkout dev
git pull origin dev

# 2. Buat branch baru dari dev
git checkout -b feature/nama-fitur-kamu
# contoh: git checkout -b feature/merchant-food-listing

# 3. Kerjakan fiturnya...
# edit file, tambah file, dll

# 4. Simpan perubahan
git add .
git commit -m "feat: deskripsi singkat apa yang kamu buat"
# contoh: git commit -m "feat: tambah halaman CRUD food listing merchant"

# 5. Push ke GitHub
git push origin feature/nama-fitur-kamu

# 6. Buat Pull Request di GitHub
# Dari branch feature/... → ke branch dev
# Minta review ke ketua kelompok sebelum di-merge
```

### Format Commit Message

Gunakan format ini supaya riwayat commit mudah dibaca:

| Prefix | Kapan dipakai | Contoh |
|---|---|---|
| `feat:` | Tambah fitur baru | `feat: tambah form tambah listing` |
| `fix:` | Perbaiki bug | `fix: perbaiki validasi stok negatif` |
| `style:` | Perubahan tampilan saja | `style: rapikan tampilan tabel listing` |
| `refactor:` | Rapikan kode tanpa ubah fungsi | `refactor: ekstrak method validasi` |
| `docs:` | Update dokumentasi | `docs: tambah komentar di controller` |

### File yang TIDAK Boleh Di-commit

Pastikan `.gitignore` sudah mengecualikan file-file ini (biasanya sudah otomatis):
- `.env` — berisi password database
- `vendor/` — dependencies composer
- `node_modules/` — dependencies npm
- `storage/app/public/` — file upload user

---

## 6. Cara Membuat Fitur Baru

Ini urutan langkah yang benar setiap kali membuat fitur baru di Laravel.

### Langkah 1 — Tambahkan Route di `routes/web.php`

```php
// Contoh: route untuk halaman kelola listing merchant
Route::middleware(['auth', 'role:merchant'])->prefix('merchant')->group(function () {
    Route::get('/food-listings', [MerchantFoodListingController::class, 'index'])->name('merchant.food-listings.index');
    Route::get('/food-listings/create', [MerchantFoodListingController::class, 'create'])->name('merchant.food-listings.create');
    Route::post('/food-listings', [MerchantFoodListingController::class, 'store'])->name('merchant.food-listings.store');
    Route::get('/food-listings/{id}/edit', [MerchantFoodListingController::class, 'edit'])->name('merchant.food-listings.edit');
    Route::put('/food-listings/{id}', [MerchantFoodListingController::class, 'update'])->name('merchant.food-listings.update');
    Route::delete('/food-listings/{id}', [MerchantFoodListingController::class, 'destroy'])->name('merchant.food-listings.destroy');
});
```

### Langkah 2 — Buat Controller

```bash
php artisan make:controller Merchant/FoodListingController
```

Isi method yang diperlukan di file yang baru dibuat.

### Langkah 3 — Buat View (Blade)

Buat file di `resources/views/merchant/food-listings/`:
- `index.blade.php` — halaman daftar listing
- `create.blade.php` — form tambah listing
- `edit.blade.php` — form edit listing

### Langkah 4 — Test di Browser

Jalankan `php artisan serve`, login sebagai merchant, buka URL yang baru dibuat.

---

## 7. Pembagian Tugas Per Anggota

### Wiwid — Manajemen Listing Makanan (Merchant Side)
**FR-04 & FR-05**

File yang dikerjakan:
- `app/Http/Controllers/Merchant/FoodListingController.php`
- `resources/views/merchant/food-listings/index.blade.php`
- `resources/views/merchant/food-listings/create.blade.php`
- `resources/views/merchant/food-listings/edit.blade.php`

Yang perlu diimplementasi:
- Form tambah listing makanan surplus (nama, deskripsi, harga normal, harga diskon, stok, kategori, foto, waktu pickup)
- Validasi input (harga diskon tidak boleh lebih dari harga normal, stok tidak boleh negatif)
- Edit listing yang sudah ada
- Hapus listing (soft delete — isi kolom `deleted_at`, jangan hapus dari DB)
- Merchant hanya bisa mengelola listing miliknya sendiri

Aturan penting:
- Merchant hanya bisa tambah listing jika `verification_status = approved`
- Query wajib filter `whereNull('deleted_at')`
- Simpan foto ke `storage/app/public/listings/`, simpan path-nya di kolom `photo_url`

---

### Alena — Katalog & Pencarian (User Side)
**FR-06**

File yang dikerjakan:
- `app/Http/Controllers/User/FoodListingController.php` (buat baru)
- `resources/views/user/food-listings/index.blade.php` (buat baru)
- `resources/views/user/food-listings/show.blade.php` (buat baru)

Yang perlu diimplementasi:
- Halaman katalog semua listing aktif dari merchant yang sudah terverifikasi
- Filter berdasarkan kategori
- Pencarian berdasarkan nama produk atau nama merchant
- Halaman detail listing (nama, deskripsi, harga, stok, lokasi merchant, jam pickup)
- Tampilkan hanya listing dengan `status = available` dan `deleted_at IS NULL` dan `stock_qty > 0`
- Merchant-nya juga harus `verification_status = approved`

---

### Maria — Verifikasi Merchant & Dashboard Admin
**FR-12 & FR-13**

File yang dikerjakan:
- `app/Http/Controllers/Admin/MerchantVerificationController.php`
- `app/Http/Controllers/Admin/DashboardController.php`
- `resources/views/admin/merchant-verification/index.blade.php`
- `resources/views/admin/merchant-verification/show.blade.php`
- `resources/views/admin/dashboard/index.blade.php`

Yang perlu diimplementasi:
- Dashboard admin: statistik jumlah user, merchant, listing aktif, pesanan hari ini
- Halaman antrian verifikasi merchant (status `pending`)
- Halaman detail merchant: tampilkan dokumen yang diupload (foto KTP, SIUP, sertifikat halal)
- Tombol Approve → ubah `verification_status` jadi `approved`, isi `verified_at`
- Tombol Reject → ubah `verification_status` jadi `rejected`, wajib isi alasan
- Setiap keputusan disimpan ke tabel `merchant_verifications` sebagai log audit

---

### Nabil — Peta Interaktif Lokasi Merchant
**FR-07 & FR-08**

File yang dikerjakan:
- `app/Http/Controllers/User/MerchantController.php` (buat baru)
- `resources/views/user/map.blade.php` (buat baru)

Yang perlu diimplementasi:
- Halaman peta yang menampilkan pin lokasi merchant yang punya listing aktif
- Data koordinat diambil dari kolom `latitude` dan `longitude` di tabel `merchant_profiles`
- Integrasi Leaflet.js (OpenStreetMap, gratis) atau Google Maps API
- Klik pin → tampilkan popup: nama merchant, listing tersedia, tombol lihat detail
- Filter: hanya tampilkan merchant dengan `verification_status = approved` dan punya listing `available`

Cara integrasi Leaflet.js (disarankan karena gratis):
```html
{{-- Di blade view, tambahkan di bagian head --}}
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

{{-- Di body --}}
<div id="map" style="height: 500px;"></div>
<script>
    var map = L.map('map').setView([-7.5653, 110.8215], 13); // koordinat Solo
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

    // Data merchant dari controller (di-pass lewat Blade)
    var merchants = @json($merchants);
    merchants.forEach(function(merchant) {
        L.marker([merchant.latitude, merchant.longitude])
         .addTo(map)
         .bindPopup('<b>' + merchant.business_name + '</b>');
    });
</script>
```

---

## 8. Aturan & Konvensi Kode

### Aturan Database (Wajib)

```php
// ✅ BENAR — selalu filter soft delete
$listings = FoodListing::whereNull('deleted_at')->get();

// ❌ SALAH — bisa ambil data yang sudah "dihapus"
$listings = FoodListing::all();
```

```php
// ✅ BENAR — hapus listing dengan soft delete
$listing->deleted_at = now();
$listing->save();

// ❌ SALAH — ini benar-benar hapus dari database, histori pesanan rusak
$listing->delete();
```

```php
// ✅ BENAR — merchant hanya bisa akses listing miliknya
$listing = FoodListing::where('merchant_id', $merchantProfile->id)
                      ->whereNull('deleted_at')
                      ->findOrFail($id);

// ❌ SALAH — merchant bisa akses listing milik merchant lain
$listing = FoodListing::findOrFail($id);
```

### Aturan Akses & Keamanan

- Setiap route yang butuh login wajib pakai middleware `auth`
- Setiap route yang spesifik untuk role tertentu wajib pakai middleware `role:merchant` atau `role:admin`
- Merchant tidak bisa publish listing jika `verification_status != approved` — cek ini di controller
- Jangan pernah simpan password dalam bentuk plaintext — selalu gunakan `Hash::make()`

### Konvensi Penamaan

| Hal | Konvensi | Contoh |
|---|---|---|
| Nama Controller | PascalCase + Controller | `FoodListingController` |
| Nama Method | camelCase | `index`, `store`, `update` |
| Nama View | kebab-case | `food-listings/index.blade.php` |
| Nama Route | kebab-case dengan titik | `merchant.food-listings.index` |
| Nama Variabel | camelCase | `$foodListings`, `$merchantProfile` |

### Palet Warna UI

Gunakan warna ini secara konsisten di semua halaman:

| Nama | Hex | Digunakan untuk |
|---|---|---|
| Dark Olive Green | `#5F6F52` | Warna utama, header, tombol primary |
| Laurel Green | `#A9B388` | Aksen, highlight |
| Cornsilk | `#FEFAE0` | Background halaman |
| Camel | `#B99470` | Tombol secondary, border |
| Charcoal Brown | `#382C23` | Teks utama, judul |

---

## 9. Pertanyaan Umum (FAQ)

**Q: Saya ubah kode tapi di browser tidak ada perubahan?**
A: Coba hard refresh browser dengan `Ctrl + Shift + R`. Kalau masih sama, cek apakah `php artisan serve` masih berjalan di terminal.

**Q: Muncul error "Class not found"?**
A: Jalankan `composer dump-autoload` di terminal, lalu refresh browser.

**Q: Bagaimana cara lihat error yang lebih detail?**
A: Pastikan di file `.env` ada `APP_DEBUG=true`. Error lengkap akan tampil di browser.

**Q: Saya tambah route tapi muncul 404?**
A: Cek apakah sudah simpan file `web.php`, dan pastikan tidak ada typo di URL yang kamu buka.

**Q: Bagaimana cara login sebagai admin untuk testing?**
A: Tanya ketua kelompok untuk kredensial akun testing, atau buat via tinker:
```bash
php artisan tinker
# Lalu ketik:
App\Models\User::create(['name'=>'Admin', 'email'=>'admin@ecoeats.com', 'password_hash'=>bcrypt('password'), 'role'=>'admin']);
```

**Q: Saya tidak sengaja push ke branch `main`/`dev` langsung, bagaimana?**
A: Segera beritahu ketua kelompok. Jangan panik dan jangan coba "fix" sendiri karena bisa memperparah.

**Q: File `.env` saya tidak ada, bagaimana?**
A: Jalankan `cp .env.example .env` lalu `php artisan key:generate`, kemudian isi konfigurasi database.

**Q: Bagaimana cara cek apakah relasi antar model sudah benar?**
A: Pakai tinker untuk test relasi:
```bash
php artisan tinker
$user = App\Models\User::find(1);
$user->merchantProfile; // seharusnya return data MerchantProfile
```

---

*Dokumen ini diperbarui sesuai progress proyek. Jika ada yang kurang jelas, tanyakan ke ketua kelompok atau buat issue di GitHub.*
