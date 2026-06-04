# Panduan Kontribusi — EcoEats

Dokumen ini untuk seluruh anggota tim EcoEats. Baca dari atas ke bawah sebelum mulai mengerjakan bagian apapun. Tidak perlu pengalaman Laravel sebelumnya — semua dijelaskan dari nol.

---

## Daftar Isi

1. [Kenapa Pakai Framework?](#1-kenapa-pakai-framework)
2. [Bagaimana Web Bekerja](#2-bagaimana-web-bekerja)
3. [Apa itu Laravel dan MVC](#3-apa-itu-laravel-dan-mvc)
4. [Komponen Laravel yang Perlu Kamu Tahu](#4-komponen-laravel-yang-perlu-kamu-tahu)
5. [Blade — HTML-nya Laravel](#5-blade--html-nya-laravel)
6. [Database di Laravel](#6-database-di-laravel)
7. [Struktur Folder Proyek EcoEats](#7-struktur-folder-proyek-ecoeats)
8. [Setup Awal di Komputer Kamu](#8-setup-awal-di-komputer-kamu)
9. [Alur Kerja Git](#9-alur-kerja-git)
10. [Pembagian Tugas Per Anggota](#10-pembagian-tugas-per-anggota)
11. [Aturan Kode yang Wajib Diikuti](#11-aturan-kode-yang-wajib-diikuti)
12. [FAQ](#12-faq)

---

## 1. Kenapa Pakai Framework?

Bayangkan kamu diminta membangun rumah. Kamu bisa saja membuat satu per satu dari batu bata — tapi itu lambat dan rentan salah. Framework itu seperti **blueprint + alat yang sudah disiapkan**. Fondasinya sudah ada, kamu tinggal fokus membangun bagian yang unik dari rumahmu.

Tanpa framework, website PHP biasa terlihat seperti ini:

```php
<?php
// Koneksi database — tulis sendiri setiap file
$conn = mysqli_connect("localhost", "root", "", "ecoeats");

// Ambil data — tulis query mentah
$result = mysqli_query($conn, "SELECT * FROM food_listings WHERE deleted_at IS NULL");

// Tampilkan — campur PHP dan HTML dalam satu file
while ($row = mysqli_fetch_assoc($result)) {
    echo "<div>" . $row['name'] . "</div>";
}

// Keamanan — harus ingat sendiri
// Validasi — harus buat sendiri
// Routing URL — harus buat sendiri
// Session login — harus buat sendiri
```

Masalahnya:
- Kode cepat berantakan karena logika dan tampilan campur jadi satu
- Harus tulis ulang hal yang sama di setiap file (koneksi DB, validasi, dll)
- Keamanan mudah jebol kalau lupa sanitasi input
- Susah dikerjakan banyak orang sekaligus karena tidak ada struktur yang jelas

**Laravel menyelesaikan semua masalah itu.** Kamu fokus ke fitur, Laravel yang urus sisanya.

---

## 2. Bagaimana Web Bekerja

Sebelum masuk ke Laravel, pahami dulu siklus dasar yang terjadi setiap kali kamu buka website.

```
Kamu buka browser → ketik URL → tekan Enter

Browser                         Server (Laravel)
  │                                   │
  │  "Hei, aku mau lihat             │
  │   halaman /merchant/login"        │
  │  ─────── HTTP Request ──────────► │
  │                                   │ Laravel terima request
  │                                   │ Cek: URL ini minta apa?
  │                                   │ Proses: ambil data, render HTML
  │                                   │
  │  ◄──────── HTTP Response ──────── │
  │  "Ini halaman login-nya,          │
  │   dalam bentuk HTML"              │
  │                                   │
Browser render HTML → kamu lihat halaman login
```

Ada dua jenis request HTTP yang paling sering dipakai:

| Jenis | Kapan terjadi | Contoh |
|---|---|---|
| **GET** | Saat kamu buka URL / klik link | Buka halaman daftar makanan |
| **POST** | Saat kamu submit form | Klik tombol "Login", "Simpan", "Tambah" |

Ini penting karena di Laravel, setiap URL punya jenis request yang berbeda dan ditangani secara berbeda.

---

## 3. Apa itu Laravel dan MVC

Laravel menggunakan pola desain bernama **MVC** — singkatan dari **Model, View, Controller**. Ini adalah cara memisahkan kode menjadi tiga bagian dengan tanggung jawab masing-masing.

```
┌─────────────────────────────────────────────────────────┐
│                        MVC                              │
│                                                         │
│   MODEL              VIEW               CONTROLLER      │
│  "Data"            "Tampilan"            "Otak"         │
│                                                         │
│  Berhubungan       File HTML yang       Menerima        │
│  langsung          ditampilkan          request,        │
│  dengan            ke user.             ambil data      │
│  database.         Dibuat dengan        dari Model,     │
│                    Blade.               kirim ke View.  │
│                                                         │
│  FoodListing.php   index.blade.php    FoodListingCtrl   │
│  User.php          show.blade.php     AuthController    │
│  Order.php         create.blade.php   OrderController   │
└─────────────────────────────────────────────────────────┘
```

### Analogi Restoran

Bayangkan sebuah restoran:

- **Controller** = Pelayan. Menerima pesanan dari tamu (request), pergi ke dapur, ambil makanan, dan antar ke meja tamu.
- **Model** = Dapur + bahan makanan. Tahu cara masak (query database) dan menyimpan stok (data).
- **View** = Presentasi makanan di piring. Tampilan akhir yang dilihat tamu.

Tamu tidak perlu tahu cara masak. Dapur tidak perlu tahu siapa tamunya. Pelayan yang menghubungkan keduanya.

### Alur Lengkap di Laravel

```
1. User buka browser → ketik http://localhost:8000/merchant/food-listings

2. Laravel buka routes/web.php
   → Cari: "ada tidak route GET /merchant/food-listings?"
   → Ketemu: "Panggil method index() di FoodListingController"

3. FoodListingController@index() dijalankan
   → Ambil data dari database lewat Model FoodListing
   → Siapkan data untuk dikirim ke View

4. Controller kirim data ke View
   → return view('merchant.food-listings.index', ['listings' => $data])

5. Blade render file resources/views/merchant/food-listings/index.blade.php
   → Data dari controller dimasukkan ke template HTML

6. HTML jadi dikirim ke browser
   → User melihat daftar food listing
```

---

## 4. Komponen Laravel yang Perlu Kamu Tahu

### 4.1 Route — Peta URL Aplikasi

File: `routes/web.php`

Route adalah daftar semua URL yang dikenali aplikasi dan "siapa" yang menanganinya. Tanpa route, semua URL akan menghasilkan error 404.

```php
// Sintaks dasar:
// Route::METHOD('URL', [Controller::class, 'method']);

// Contoh route GET — ketika user buka halaman
Route::get('/merchant/food-listings', [FoodListingController::class, 'index']);

// Contoh route POST — ketika user submit form
Route::post('/merchant/food-listings', [FoodListingController::class, 'store']);

// Contoh route dengan parameter — {id} bisa diisi angka apapun
Route::get('/merchant/food-listings/{id}/edit', [FoodListingController::class, 'edit']);

// Route dengan middleware — hanya bisa diakses kalau sudah login DAN role-nya merchant
Route::middleware(['auth', 'role:merchant'])->group(function () {
    Route::get('/merchant/dashboard', [DashboardController::class, 'index']);
    Route::get('/merchant/food-listings', [FoodListingController::class, 'index']);
    // ... route lain untuk merchant
});
```

> **Apa itu middleware?**
> Middleware adalah "penjaga pintu". Sebelum request sampai ke controller, middleware diperiksa dulu. Di EcoEats, ada middleware `auth` (cek apakah sudah login) dan `role:merchant` (cek apakah role-nya merchant). Kalau tidak lolos, langsung redirect ke halaman login.

### 4.2 Controller — Logika Bisnis

Lokasi: `app/Http/Controllers/`

Controller adalah kelas PHP yang berisi method-method untuk menangani request. Setiap method biasanya sesuai dengan satu aksi.

Nama method yang lazim dipakai (konvensi Laravel):

| Method | Fungsi | Jenis Route |
|---|---|---|
| `index()` | Tampilkan daftar semua data | GET `/food-listings` |
| `show($id)` | Tampilkan detail satu data | GET `/food-listings/5` |
| `create()` | Tampilkan form tambah data | GET `/food-listings/create` |
| `store()` | Simpan data baru dari form | POST `/food-listings` |
| `edit($id)` | Tampilkan form edit data | GET `/food-listings/5/edit` |
| `update($id)` | Simpan perubahan dari form edit | PUT `/food-listings/5` |
| `destroy($id)` | Hapus data | DELETE `/food-listings/5` |

Contoh controller lengkap yang bisa kamu baca:

```php
<?php

namespace App\Http\Controllers\Merchant;

use App\Http\Controllers\Controller;
use App\Models\FoodListing;
use Illuminate\Http\Request;

class FoodListingController extends Controller
{
    // Tampilkan daftar listing milik merchant yang sedang login
    public function index()
    {
        // auth()->user() = ambil data user yang sedang login
        // ->merchantProfile = akses relasi ke tabel merchant_profiles
        $merchantId = auth()->user()->merchantProfile->id;

        // Ambil listing milik merchant ini, yang belum dihapus
        $listings = FoodListing::where('merchant_id', $merchantId)
                               ->whereNull('deleted_at')  // ← WAJIB selalu ada
                               ->latest()                 // urutkan terbaru dulu
                               ->get();

        // Kirim ke view, dengan variabel bernama $listings
        return view('merchant.food-listings.index', [
            'listings' => $listings
        ]);
    }

    // Tampilkan form tambah listing
    public function create()
    {
        return view('merchant.food-listings.create');
    }

    // Simpan listing baru ke database
    public function store(Request $request)
    {
        // Validasi input dari form
        $request->validate([
            'name'           => 'required|string|max:150',
            'discount_price' => 'required|numeric|min:0',
            'stock_qty'      => 'required|integer|min:1',
        ]);

        // Simpan ke database
        FoodListing::create([
            'merchant_id'    => auth()->user()->merchantProfile->id,
            'name'           => $request->name,
            'discount_price' => $request->discount_price,
            'stock_qty'      => $request->stock_qty,
            // ... kolom lainnya
        ]);

        // Redirect ke halaman daftar, dengan pesan sukses
        return redirect()->route('merchant.food-listings.index')
                         ->with('success', 'Listing berhasil ditambahkan!');
    }
}
```

### 4.3 Model — Jembatan ke Database

Lokasi: `app/Models/`

Model adalah kelas PHP yang merepresentasikan satu tabel di database. Dengan Model, kamu tidak perlu menulis SQL mentah — cukup pakai method yang sudah disediakan Laravel.

```php
// Ambil semua data (JANGAN pakai ini kalau ada soft delete)
$listings = FoodListing::all();

// Ambil semua data yang belum dihapus
$listings = FoodListing::whereNull('deleted_at')->get();

// Ambil satu data berdasarkan ID (error 404 otomatis kalau tidak ketemu)
$listing = FoodListing::findOrFail($id);

// Ambil data dengan kondisi tertentu
$available = FoodListing::where('status', 'available')
                        ->where('stock_qty', '>', 0)
                        ->whereNull('deleted_at')
                        ->get();

// Simpan data baru
FoodListing::create([
    'name'  => 'Nasi Goreng',
    'price' => 15000,
]);

// Update data
$listing->update(['stock_qty' => 5]);

// Soft delete (isi deleted_at, data tidak benar-benar hilang dari DB)
$listing->deleted_at = now();
$listing->save();
```

Model juga mendefinisikan **relasi** antar tabel:

```php
// Di dalam app/Models/User.php
public function merchantProfile()
{
    // Satu user punya satu merchant profile
    return $this->hasOne(MerchantProfile::class);
}

// Di dalam app/Models/MerchantProfile.php
public function foodListings()
{
    // Satu merchant profile punya banyak food listing
    return $this->hasMany(FoodListing::class, 'merchant_id');
}
```

Cara pakainya di controller:

```php
// Akses profil merchant dari user yang login
$profile = auth()->user()->merchantProfile;

// Akses semua listing dari profil merchant tertentu
$listings = $merchantProfile->foodListings;
```

---

## 5. Blade — HTML-nya Laravel

Blade adalah template engine Laravel. Intinya: kamu nulis HTML biasa, tapi bisa ditambahkan logika PHP menggunakan sintaks khusus yang lebih bersih.

### Menampilkan Variabel

```html
{{-- Ini komentar Blade, tidak akan muncul di HTML --}}

{{-- Tampilkan variabel (aman dari XSS/injection) --}}
<h1>{{ $listing->name }}</h1>
<p>Harga: Rp{{ number_format($listing->discount_price) }}</p>

{{-- Tampilkan variabel tanpa escape (hati-hati, hanya untuk konten yang aman) --}}
{!! $htmlContent !!}
```

### Logika Kondisi

```html
@if ($listing->status === 'available')
    <span class="badge-green">Tersedia</span>
@elseif ($listing->status === 'sold_out')
    <span class="badge-red">Habis</span>
@else
    <span class="badge-gray">Tidak Aktif</span>
@endif

{{-- Cek apakah variabel ada isinya --}}
@if ($listing->photo_url)
    <img src="{{ $listing->photo_url }}" alt="Foto makanan">
@endif
```

### Perulangan

```html
@foreach ($listings as $listing)
    <div class="card">
        <h3>{{ $listing->name }}</h3>
        <p>Stok: {{ $listing->stock_qty }}</p>
    </div>
@endforeach

{{-- Tampilkan pesan kalau data kosong --}}
@forelse ($listings as $listing)
    <div>{{ $listing->name }}</div>
@empty
    <p>Belum ada listing makanan.</p>
@endforelse
```

### Layout — Supaya Tidak Tulis Header Ulang di Setiap Halaman

Laravel Blade punya sistem layout. Bayangkan seperti template yang bisa dipakai ulang.

File layout utama `resources/views/layouts/app.blade.php`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>EcoEats - @yield('title')</title>
    {{-- CSS, meta tags, dll --}}
</head>
<body>
    {{-- Sidebar atau navbar yang sama di semua halaman --}}
    <nav>...</nav>

    <main>
        {{-- Konten tiap halaman masuk di sini --}}
        @yield('content')
    </main>

    <footer>...</footer>
</body>
</html>
```

File halaman spesifik `resources/views/merchant/food-listings/index.blade.php`:
```html
{{-- Pakai layout app.blade.php --}}
@extends('layouts.app')

{{-- Isi bagian title --}}
@section('title', 'Daftar Listing Saya')

{{-- Isi bagian content --}}
@section('content')
    <h1>Daftar Makanan Surplus Saya</h1>

    @forelse ($listings as $listing)
        <div>{{ $listing->name }}</div>
    @empty
        <p>Belum ada listing.</p>
    @endforelse
@endsection
```

### Form di Blade

```html
{{-- Form untuk tambah data --}}
<form action="{{ route('merchant.food-listings.store') }}" method="POST">
    {{-- WAJIB ada di setiap form POST — untuk keamanan --}}
    @csrf

    <input type="text" name="name" placeholder="Nama makanan">

    {{-- Tampilkan error validasi --}}
    @error('name')
        <span style="color:red">{{ $message }}</span>
    @enderror

    <button type="submit">Simpan</button>
</form>

{{-- Form untuk edit (method PUT karena Laravel butuh ini) --}}
<form action="{{ route('merchant.food-listings.update', $listing->id) }}" method="POST">
    @csrf
    @method('PUT')  {{-- ← ini memberitahu Laravel bahwa ini PUT, bukan POST --}}

    <input type="text" name="name" value="{{ $listing->name }}">
    <button type="submit">Update</button>
</form>

{{-- Form untuk hapus --}}
<form action="{{ route('merchant.food-listings.destroy', $listing->id) }}" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus</button>
</form>
```

> **Kenapa harus ada `@csrf`?**
> CSRF (Cross-Site Request Forgery) adalah jenis serangan di mana website jahat bisa mengirim request atas nama user. `@csrf` menambahkan token rahasia ke setiap form sehingga Laravel bisa memverifikasi bahwa request memang berasal dari form di aplikasi kita sendiri. Kalau lupa `@csrf`, form tidak akan berfungsi dan Laravel akan error.

---

## 6. Database di Laravel

### Cara Laravel Berhubungan dengan Database

Laravel tidak mewajibkan kamu menulis SQL. Sebagai gantinya, ada dua cara:

**Eloquent ORM** — cara yang paling sering dipakai, seperti yang sudah dicontohkan di bagian Model. Kamu pakai method PHP, Laravel yang terjemahkan ke SQL.

**Query Builder** — kalau butuh query yang lebih kompleks:
```php
// Ini setara dengan: SELECT * FROM food_listings WHERE merchant_id = 5 AND deleted_at IS NULL
$listings = DB::table('food_listings')
              ->where('merchant_id', 5)
              ->whereNull('deleted_at')
              ->get();
```

### Soft Delete — Konsep Penting di EcoEats

Di EcoEats, beberapa tabel punya kolom `deleted_at`. Ini namanya **soft delete** — data tidak benar-benar dihapus dari database, hanya ditandai sebagai "sudah dihapus" dengan mengisi timestamp di kolom itu.

Kenapa? Karena tabel `order_items` menyimpan referensi ke `food_listings`. Kalau listing dihapus beneran dari DB, riwayat pesanan akan rusak.

```
Tabel food_listings:
┌────┬─────────────────┬────────────────────────┐
│ id │ name            │ deleted_at             │
├────┼─────────────────┼────────────────────────┤
│  1 │ Nasi Goreng     │ NULL                   │  ← aktif
│  2 │ Mie Ayam        │ 2024-06-01 10:00:00   │  ← sudah "dihapus"
│  3 │ Roti Bakar      │ NULL                   │  ← aktif
└────┴─────────────────┴────────────────────────┘
```

Aturannya: **setiap query yang ambil data aktif wajib tambahkan `->whereNull('deleted_at')`**.

---

## 7. Struktur Folder Proyek EcoEats

Project Laravel ada di `src/backend/`. Ini folder-folder yang akan sering kamu sentuh:

```
src/backend/
│
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/              ← login & register — sudah ada, hati-hati mengubah
│   │   │   ├── Admin/             ← semua fitur halaman admin
│   │   │   ├── Merchant/          ← semua fitur halaman merchant
│   │   │   └── User/              ← semua fitur halaman user biasa
│   │   └── Middleware/
│   │       └── RoleMiddleware.php ← penjaga route per role — jangan diubah
│   └── Models/
│       ├── User.php               ← jangan diubah sembarangan
│       ├── FoodListing.php
│       ├── MerchantProfile.php
│       ├── Category.php
│       └── MerchantVerification.php
│
├── resources/
│   └── views/                     ← semua file Blade (tampilan) ada di sini
│       ├── layouts/
│       │   └── app.blade.php      ← template induk semua halaman
│       ├── auth/                  ← halaman login & register
│       ├── admin/                 ← halaman-halaman admin
│       ├── merchant/              ← halaman-halaman merchant
│       └── user/                  ← halaman-halaman user
│
├── routes/
│   └── web.php                    ← daftar semua URL aplikasi
│
└── .env                           ← konfigurasi DB dll — JANGAN di-commit ke Git
```

---

## 8. Setup Awal di Komputer Kamu

Lakukan ini **sekali saja** saat pertama kali mau mulai ngerjain.

### Prasyarat — Install Dulu Ini

- **XAMPP** (sudah include PHP dan MySQL) — download di [apachefriends.org](https://www.apachefriends.org)
- **Composer** — download di [getcomposer.org](https://getcomposer.org)
- **Git** — download di [git-scm.com](https://git-scm.com)
- **VS Code** — editor kode, download di [code.visualstudio.com](https://code.visualstudio.com)

Setelah install, verifikasi di terminal/command prompt:
```bash
php -v        # harus muncul versi PHP 8.2+
composer -v   # harus muncul versi Composer
git -v        # harus muncul versi Git
```

### Langkah Setup Project

**1. Clone repo dari GitHub**
```bash
git clone https://github.com/[nama-org]/Praktikum-rpl-a-5.git
cd Praktikum-rpl-a-5/src/backend
```

**2. Install semua package Laravel**
```bash
composer install
```
> Ini akan mendownload semua library yang dibutuhkan ke folder `vendor/`. Bisa memakan waktu 1-3 menit.

**3. Buat file konfigurasi**
```bash
cp .env.example .env
php artisan key:generate
```
> File `.env` berisi konfigurasi sensitif (password DB, dll). File ini tidak masuk Git dan harus dibuat di setiap komputer.

**4. Buka phpMyAdmin, buat database baru**
- Jalankan XAMPP, aktifkan Apache dan MySQL
- Buka `http://localhost/phpmyadmin`
- Buat database baru dengan nama: `ecoeats`

**5. Isi konfigurasi database di file `.env`**

Buka file `.env` dengan VS Code, cari bagian ini dan sesuaikan:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ecoeats
DB_USERNAME=root
DB_PASSWORD=          ← kosongkan jika XAMPP default, isi jika ada password
```

**6. Import skema database**

Import file SQL yang ada di repo (tanya ketua kelompok untuk lokasi filenya) via phpMyAdmin:
- Buka phpMyAdmin → pilih database `ecoeats` → tab Import → pilih file SQL → klik Go

**7. Jalankan server**
```bash
php artisan serve
```
Buka browser → `http://localhost:8000` → kalau muncul halaman Laravel, setup berhasil.

---

## 9. Alur Kerja Git

### Aturan Branch

| Branch | Fungsi | Boleh push langsung? |
|---|---|---|
| `main` | Kode final yang sudah stabil | ❌ Tidak boleh |
| `dev` | Branch pengembangan utama | ❌ Tidak boleh |
| `feature/nama-fitur` | Branch untuk fitur yang sedang dikerjakan | ✅ Boleh |

### Cara Mengerjakan Fitur

```bash
# Langkah 1: Pastikan branch dev kamu paling update
git checkout dev
git pull origin dev

# Langkah 2: Buat branch baru dari dev
# Ganti "nama-fitur" dengan nama fitur yang kamu kerjakan
git checkout -b feature/nama-fitur
# Contoh: git checkout -b feature/merchant-food-listing

# Langkah 3: Kerjakan fiturnya di VS Code...

# Langkah 4: Setelah selesai, simpan perubahan ke Git
git add .
git commit -m "feat: deskripsi singkat perubahan yang kamu buat"

# Langkah 5: Upload ke GitHub
git push origin feature/nama-fitur

# Langkah 6: Buka GitHub, buat Pull Request
# Dari branch feature/... → ke branch dev
# Minta review ke ketua kelompok sebelum di-merge
```

### Format Pesan Commit

Gunakan format ini supaya riwayat mudah dibaca:

```
feat: tambah form input food listing merchant
fix: perbaiki validasi harga diskon tidak boleh nol
style: rapikan tampilan tabel daftar listing
docs: update README cara install
```

---

## 10. Pembagian Tugas Per Anggota

### Wiwid — Manajemen Listing Makanan Merchant (FR-04, FR-05)

**Tugasmu:** Merchant harus bisa tambah, edit, dan hapus listing makanan surplus mereka.

**File yang dikerjakan:**
```
app/Http/Controllers/Merchant/FoodListingController.php
resources/views/merchant/food-listings/index.blade.php
resources/views/merchant/food-listings/create.blade.php
resources/views/merchant/food-listings/edit.blade.php
```

**Yang perlu diimplementasi:**
- Halaman daftar listing milik merchant yang sedang login
- Form tambah listing: nama, deskripsi, harga normal, harga diskon, stok, kategori, foto, waktu pickup mulai & selesai
- Validasi: harga diskon tidak boleh lebih dari harga normal, stok minimal 1
- Form edit listing yang sudah ada
- Hapus listing (soft delete — **jangan hapus dari DB**, cukup isi `deleted_at = now()`)
- Merchant hanya bisa lihat dan kelola listing **miliknya sendiri**
- Merchant hanya bisa tambah listing jika `verification_status = approved`

**Aturan kode:**
```php
// ✅ Selalu filter merchant_id dan deleted_at
$listings = FoodListing::where('merchant_id', $merchantId)
                       ->whereNull('deleted_at')
                       ->get();

// ✅ Hapus dengan soft delete
$listing->deleted_at = now();
$listing->save();

// ❌ Jangan hapus langsung dari DB
$listing->delete(); // JANGAN
```

---

### Alena — Katalog & Pencarian untuk User (FR-06)

**Tugasmu:** User (pembeli) harus bisa lihat dan cari makanan surplus dari semua merchant.

**File yang dikerjakan:**
```
app/Http/Controllers/User/FoodListingController.php  (buat baru)
resources/views/user/food-listings/index.blade.php   (buat baru)
resources/views/user/food-listings/show.blade.php    (buat baru)
```

**Yang perlu diimplementasi:**
- Halaman katalog semua listing dari merchant yang sudah terverifikasi
- Filter berdasarkan kategori makanan
- Pencarian berdasarkan nama produk atau nama merchant
- Halaman detail listing: foto, nama, deskripsi, harga coret dan harga diskon, stok, nama merchant, lokasi, jam pickup
- Hanya tampilkan listing yang memenuhi semua syarat ini:

```php
$listings = FoodListing::where('status', 'available')
                       ->where('stock_qty', '>', 0)
                       ->whereNull('deleted_at')
                       ->whereHas('merchant', function ($query) {
                           // Hanya dari merchant yang sudah diverifikasi
                           $query->where('verification_status', 'approved');
                       })
                       ->get();
```

---

### Maria — Verifikasi Merchant & Dashboard Admin (FR-12, FR-13)

**Tugasmu:** Admin harus bisa review dokumen merchant dan approve atau reject mereka.

**File yang dikerjakan:**
```
app/Http/Controllers/Admin/MerchantVerificationController.php
app/Http/Controllers/Admin/DashboardController.php
resources/views/admin/merchant-verification/index.blade.php
resources/views/admin/merchant-verification/show.blade.php
resources/views/admin/dashboard/index.blade.php
```

**Yang perlu diimplementasi:**
- Dashboard admin: kartu statistik (total user, total merchant, listing aktif hari ini, pesanan pending)
- Halaman antrian verifikasi: daftar merchant dengan status `pending`
- Halaman detail merchant: tampilkan foto KTP, SIUP, sertifikat halal yang sudah diupload
- Tombol **Approve**: ubah `verification_status` → `approved`, isi `verified_at = now()`
- Tombol **Reject**: ubah `verification_status` → `rejected`, wajib minta alasan dari admin
- Setiap keputusan admin **wajib disimpan** ke tabel `merchant_verifications` sebagai log audit:

```php
// Setiap kali admin approve atau reject, simpan ke log
MerchantVerification::create([
    'merchant_id' => $merchantProfile->id,
    'admin_id'    => auth()->user()->id,
    'action'      => 'approved',  // atau 'rejected'
    'notes'       => $request->notes,
]);
```

---

### Nabil — Peta Interaktif Lokasi Merchant (FR-07, FR-08)

**Tugasmu:** User harus bisa lihat lokasi semua merchant di peta interaktif.

**File yang dikerjakan:**
```
app/Http/Controllers/User/MerchantController.php  (buat baru)
resources/views/user/map.blade.php                (buat baru)
```

**Yang perlu diimplementasi:**
- Halaman peta yang menampilkan pin lokasi semua merchant aktif yang punya listing tersedia
- Klik pin → muncul popup: nama merchant, alamat, jumlah listing tersedia, tombol "Lihat Listing"
- Hanya tampilkan merchant dengan `verification_status = approved` dan punya minimal 1 listing `available`

**Cara integrasi peta dengan Leaflet.js (gratis, tidak butuh API key):**

Di controller, siapkan data merchant dalam format yang bisa dibaca JavaScript:
```php
public function map()
{
    $merchants = MerchantProfile::where('verification_status', 'approved')
                                ->whereHas('foodListings', function ($q) {
                                    $q->where('status', 'available')
                                      ->where('stock_qty', '>', 0)
                                      ->whereNull('deleted_at');
                                })
                                ->with('foodListings')
                                ->get();

    return view('user.map', ['merchants' => $merchants]);
}
```

Di file Blade `user/map.blade.php`:
```html
{{-- Tambahkan di bagian head --}}
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />

{{-- Di body --}}
<div id="map" style="height: 500px; width: 100%;"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
    // Inisialisasi peta, pusatkan ke Kota Solo
    var map = L.map('map').setView([-7.5653, 110.8215], 14);

    // Tambahkan tile (gambar peta) dari OpenStreetMap
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
    }).addTo(map);

    // Data merchant dari Laravel — @json() otomatis convert ke format JavaScript
    var merchants = @json($merchants);

    // Tambahkan pin untuk setiap merchant
    merchants.forEach(function(merchant) {
        L.marker([merchant.latitude, merchant.longitude])
         .addTo(map)
         .bindPopup(
             '<b>' + merchant.business_name + '</b><br>' +
             merchant.business_address
         );
    });
</script>
```

---

## 11. Aturan Kode yang Wajib Diikuti

### Soft Delete — Selalu Filter `deleted_at`

```php
// ✅ BENAR
$listings = FoodListing::whereNull('deleted_at')->get();

// ❌ SALAH — bisa munculkan data yang sudah "dihapus"
$listings = FoodListing::all();
```

### Keamanan Akses Data — Merchant Hanya Boleh Akses Miliknya

```php
// ✅ BENAR — merchant tidak bisa akses listing milik merchant lain
$merchantId = auth()->user()->merchantProfile->id;
$listing = FoodListing::where('merchant_id', $merchantId)
                      ->findOrFail($id);

// ❌ SALAH — merchant bisa akses listing siapapun dengan tebak ID
$listing = FoodListing::findOrFail($id);
```

### Warna UI — Gunakan Palet Ini Secara Konsisten

| Nama | Kode Warna | Digunakan untuk |
|---|---|---|
| Dark Olive Green | `#5F6F52` | Tombol utama, header, warna primer |
| Laurel Green | `#A9B388` | Warna aksen, elemen sekunder |
| Cornsilk | `#FEFAE0` | Background halaman |
| Camel | `#B99470` | Tombol sekunder, border |
| Charcoal Brown | `#382C23` | Teks, judul |

### Jangan Commit File Ini

- `.env` — berisi password database
- `vendor/` — library composer, bisa di-install ulang
- `storage/app/` — file upload user

---

## 12. FAQ

**Q: Saya ubah kode tapi di browser tidak ada perubahan?**
Coba hard refresh: `Ctrl + Shift + R`. Pastikan juga `php artisan serve` masih berjalan di terminal.

**Q: Muncul error "Class not found"?**
Jalankan `composer dump-autoload` di terminal, lalu refresh browser.

**Q: Bagaimana cara lihat pesan error yang lengkap?**
Pastikan di file `.env` ada `APP_DEBUG=true`. Error detail akan muncul di browser.

**Q: Bagaimana cara login sebagai admin untuk testing?**
Tanya ketua kelompok untuk akun testing. Atau buat via tinker:
```bash
php artisan tinker
>>> App\Models\User::create(['name' => 'Admin Test', 'email' => 'admin@test.com', 'password_hash' => bcrypt('password123'), 'role' => 'admin']);
```

**Q: Saya tidak sengaja push langsung ke `main` atau `dev`?**
Segera beritahu ketua kelompok. Jangan coba perbaiki sendiri.

**Q: File `.env` saya tidak ada?**
Jalankan: `cp .env.example .env` lalu `php artisan key:generate`, kemudian isi konfigurasi database.

**Q: Muncul error "419 Page Expired" saat submit form?**
Kamu lupa `@csrf` di dalam tag `<form>`. Tambahkan setelah tag `<form>` pembuka.

**Q: Bagaimana cara cek apakah kode saya sudah benar sebelum di-push?**
Buka URL yang kamu kerjakan di browser. Kalau tampilannya sesuai ekspektasi dan tidak ada error merah, sudah aman.

---

*Dokumen ini diperbarui sesuai progress proyek. Ada yang kurang jelas? Tanyakan langsung ke ketua kelompok atau buat isu di GitHub.*
