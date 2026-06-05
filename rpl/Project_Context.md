# EcoEats — Project Context untuk Sesi AI Baru

Dokumen ini dibaca bersama file-file yang dilampirkan (SRS, SQL schema/data dictionary, dan web.php). Jangan duplikasi informasi dari file tersebut — dokumen ini hanya mencakup hal yang tidak ada di sana: stack teknis, konvensi kode, progress, dan struktur folder lengkap.

---

## 1. Gambaran Proyek

EcoEats adalah platform web berbasis **Laravel 11** yang menghubungkan pelaku usaha kuliner di **Kota Solo** dengan konsumen yang ingin membeli makanan surplus dengan harga terjangkau. Konsepnya mirip Too Good To Go tapi skala lokal. Sistem **tidak punya fitur delivery** — semua transaksi self-pickup di lokasi merchant.

Proyek ini adalah tugas **Praktikum Rekayasa Perangkat Lunak**. Ada dokumen SRS, ERD, user stories di folder `docs/`.

Penting: **sisi user (pembeli) hanya diakses lewat aplikasi mobile (Kotlin/Android)**. Web hanya untuk merchant dan admin. Dashboard user di web cukup halaman placeholder bertuliskan "gunakan aplikasi mobile".

---

## 2. Tech Stack

| Komponen | Detail |
|---|---|
| Framework | Laravel 11 (PHP) |
| Database | MySQL, server MariaDB 10.4.32, dikelola via phpMyAdmin |
| Frontend | Blade template engine murni, tanpa Vue/React |
| Auth web | Session-based (bawaan Laravel) |
| Auth mobile | Token-based Laravel Sanctum (fondasi sudah ada, belum diimplementasi) |
| Storage | Laravel Storage, local disk (`storage/app/public`) |
| Version control | Git + GitHub, branch `main` dan `dev` |
| Mobile (nanti) | Kotlin/Android, dikerjakan tim lain |

---

## 3. Struktur Repository

```
Praktikum-rpl-a-5/
├── docs/                   ← SRS, ERD, user stories, dll.
├── src/
│   ├── backend/            ← PROJECT LARAVEL ADA DI SINI
│   └── mobile/             ← Kotlin, dikerjakan tim lain
└── tests/
```

Semua path Laravel di bawah ini relatif terhadap `src/backend/`.

---

## 4. Palet Warna & Font Resmi

Wajib konsisten di semua Blade view. Sudah didefinisikan sebagai CSS variables.

| Nama | Hex | CSS Variable |
|---|---|---|
| Dark Olive Green | `#5F6F52` | `--olive` |
| Laurel Green | `#A9B388` | `--laurel` |
| Cornsilk | `#FEFAE0` | `--cornsilk` |
| Camel | `#B99470` | `--camel` |
| Charcoal Brown | `#382C23` | `--charcoal` |

Font: **Fraunces** (heading/display, serif) dan **Plus Jakarta Sans** (body), keduanya dari Google Fonts.

---

## 5. Catatan Teknis Kritis Laravel 11

Hal-hal ini sering jadi sumber bug — baca sebelum menulis kode apapun.

**a. Tidak ada `app/Http/Kernel.php`**
Middleware dikonfigurasi di `bootstrap/app.php`:
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->statefulApi();
})
```

**b. `routes/api.php` dibuat manual**
Tidak dibuat otomatis di Laravel 11. Sudah ada di project, isinya kosong untuk mobile nanti.

**c. Hanya satu provider: `AppServiceProvider`**
Provider lain sudah dihapus di L11.

**d. Kolom password bernama `password_hash`, bukan `password`**
Model `User` punya override wajib yang tidak boleh dihapus:
```php
public function getAuthPassword(): string
{
    return $this->password_hash;
}
```

**e. Model `User` tidak pakai trait `SoftDeletes`**
Kolom `deleted_at` dikelola manual. Jangan panggil `withTrashed()` atau `restore()` di model ini — gunakan `whereNull('deleted_at')` / `whereNotNull('deleted_at')` secara eksplisit di semua query.

**f. Sanctum sudah diinstall**
Config sudah dipublish, tabel `personal_access_tokens` sudah ada. Trait `HasApiTokens` sudah ada di model `User`. Belum dipakai, untuk mobile nanti.

**g. Scramble (API docs) sudah diinstall**
`dedoc/scramble` v0.13.23. Dokumentasi API bisa diakses di `/docs/api`. `Scramble::auth()` tidak tersedia di versi ini, biarkan `AppServiceProvider` kosong.

---

## 6. Struktur Folder Laravel Lengkap

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Auth/
│   │   │   ├── LoginController.php          ← multi-portal login (sudah ada sejak awal)
│   │   │   └── RegisterController.php       ← registrasi user & merchant
│   │   ├── Admin/
│   │   │   ├── DashboardController.php      ← summary stats real dari DB
│   │   │   ├── MerchantVerificationController.php  ← approve/reject/list merchant
│   │   │   ├── UserController.php           ← daftar user, soft delete toggle
│   │   │   ├── FoodListingController.php    ← read-only + force delete
│   │   │   ├── AccountController.php        ← tambah akun manual, reset verifikasi
│   │   │   ├── OrderController.php          ← BARU — monitoring pesanan, read-only
│   │   │   └── CategoryController.php       ← CRUD kategori
│   │   ├── Merchant/
│   │   │   ├── DashboardController.php      ← stats merchant, recent listings
│   │   │   ├── FoodListingController.php    ← CRUD listing, upload foto
│   │   │   ├── OrderController.php          ← BARU — confirm, reject, markReady, complete
│   │   │   └── ProfileController.php        ← edit profil usaha, upload dokumen
│   │   └── Api/                             ← untuk mobile, belum diisi
│   └── Middleware/
│       └── RoleMiddleware.php               ← proteksi route berdasarkan role + cek deleted_at
├── Models/
│   ├── User.php                             ← sudah ada, ada getAuthPassword(), HasApiTokens
│   ├── MerchantProfile.php                  ← relasi, isApproved(), isPending(), isRejected()
│   ├── Category.php                         ← relasi ke FoodListing
│   ├── FoodListing.php                      ← SoftDeletes, discountPercent(), scope active()
│   └── MerchantVerification.php             ← audit log verifikasi

resources/views/
├── layouts/
│   └── auth.blade.php                       ← layout halaman login/register
├── auth/
│   ├── login-user.blade.php
│   ├── login-merchant.blade.php
│   ├── login-admin.blade.php
│   ├── register-user.blade.php
│   └── register-merchant.blade.php          ← form 3 step: akun + profil usaha + dokumen
├── dashboard/
│   ├── admin.blade.php                      ← data real, sidenav lengkap
│   ├── merchant.blade.php                   ← status verifikasi, listing terbaru
│   └── user.blade.php                       ← placeholder "gunakan aplikasi mobile"
├── admin/
│   ├── merchant-verification/
│   │   ├── index.blade.php                  ← filter tab per status
│   │   └── show.blade.php                   ← detail dokumen, approve/reject/reset
│   ├── users/
│   │   └── index.blade.php                  ← search, filter aktif/nonaktif, toggle
│   ├── food-listings/
│   │   └── index.blade.php                  ← filter status+kategori, force delete
│   ├── accounts/
│   │   └── create.blade.php                 ← form dinamis, field merchant muncul jika role merchant
│   ├── categories/
│   │   └── index.blade.php                  ← list + inline edit + form tambah
│   └── orders/
│       ├── index.blade.php   ← BARU — monitoring semua pesanan, filter status+search
│       └── show.blade.php    ← BARU — detail pesanan read-only, timeline status
└── merchant/
    ├── food-listings/
    │   ├── index.blade.php                  ← grid card, discount %, action buttons
    │   ├── create.blade.php                 ← form lengkap + photo preview
    │   └── edit.blade.php                   ← pre-filled, bisa ganti foto
    └── profile/
        └── edit.blade.php                   ← edit profil usaha + upload ulang dokumen
   └── orders/
        ├── index.blade.php   ← BARU — daftar pesanan masuk, card layout, reject modal
        └── show.blade.php    ← BARU — detail + aksi (confirm/reject/ready/complete + verifik
```

---

## 7. Konvensi Kode — Wajib Diikuti

1. **Soft delete filter wajib** — query aktif pada tabel `users` dan `food_listings` selalu filter `WHERE deleted_at IS NULL`. Jangan lupa.

2. **Merchant belum approved = tidak bisa publish listing** — selalu cek `$profile->isApproved()` sebelum izinkan create/store food listing.

3. **Snapshot di `order_items`** — saat membuat order, salin `name` listing ke kolom `listing_name` dan harga ke `unit_price`. Jangan ambil dari relasi listing saat tampilkan histori.

4. **Tidak ada delivery** — hanya self-pickup. Jangan tambah field atau logika pengiriman.

5. **Controller web** ada di `app/Http/Controllers/` dengan subfolder sesuai role. **Controller API** ada di `app/Http/Controllers/Api/` dan hanya boleh return JSON.

6. **Jangan hapus `getAuthPassword()`** di model `User`.

7. **Jangan panggil `withTrashed()` pada model `User`** — tidak pakai trait SoftDeletes.

8. **Upload file** disimpan ke Laravel Storage local disk (`public`). Jalankan `php artisan storage:link` sekali jika belum pernah dijalankan.

---

## 8. Arsitektur Auth

Dua sistem auth berjalan paralel, tidak saling mengganggu.

**Web (session-based):**
- `/login` → user
- `/merchant/login` → merchant
- `/admin/login` → admin
- `/register` → registrasi user baru (langsung aktif, tanpa verifikasi email)
- `/merchant/register` → registrasi merchant (satu form: akun + profil usaha + dokumen, langsung pending)

**API (token Sanctum, untuk mobile):**
- Hanya role user (pembeli)
- Route ada di `routes/api.php`, belum diisi

---

## 9. Alur Verifikasi Merchant

1. Merchant daftar lewat `/merchant/register` → status otomatis `pending`
2. Admin review di `/admin/merchants` → bisa approve atau reject (catatan wajib diisi saat reject)
3. Setiap keputusan admin dicatat di tabel `merchant_verifications` sebagai audit log
4. Jika merchant upload ulang dokumen lewat edit profil, status kembali ke `pending` otomatis
5. Admin bisa reset status approved/rejected merchant kembali ke `pending` dari halaman detail

---

## 10. Progress Terakhir — Sudah Selesai

**Models:** `MerchantProfile`, `Category`, `FoodListing`, `MerchantVerification`

**Controllers Admin:** Dashboard, MerchantVerification, User, FoodListing, Account, Category

**Controllers Merchant:** Dashboard, FoodListing (CRUD penuh), Profile

**Controllers Auth:** Login (sudah dari awal), Register (baru)

**Views Admin:** dashboard, merchant verification (index+show), users, food listings, accounts/create, categories

**Views Merchant:** dashboard, food listings (index+create+edit), profile/edit

**Views Auth:** login (3 portal), register user, register merchant

- **Order model** dengan pickup code auto-generator di `boot()` — 8 karakter alphanumeric, tanpa 0/O/1/I, loop sampai unik
- **Merchant order flow** — konfirmasi, tolak (dengan alasan wajib + kembalikan stok), tandai ready, complete dengan verifikasi kode pickup
- **Admin order monitoring** — lihat semua pesanan platform, filter status, search kode/user/merchant, detail pesanan
- **Merchant dashboard stats** — `pending_orders` dan `completed_today` sudah aktif dari DB
- **Sidenav merchant** — semua halaman merchant sudah punya link Pesanan Masuk dengan badge merah jika ada pending
- **Sidenav admin** — semua halaman admin sudah punya link Semua Pesanan

**Fitur yang sudah jalan:**
- Multi-portal login (user/merchant/admin) dengan validasi role dan cek soft delete
- Registrasi user (langsung aktif) dan merchant (langsung pending)
- Dashboard admin dengan stats real dari DB
- Verifikasi merchant: approve, reject, reset ke pending, audit log
- Manajemen user: daftar, toggle aktif/nonaktif, tambah manual
- Tambah akun manual oleh admin (user/merchant/admin), merchant langsung approved
- CRUD food listing oleh merchant dengan upload foto
- Edit profil usaha merchant dengan upload ulang dokumen
- CRUD kategori oleh admin dengan proteksi hapus (tidak bisa hapus jika masih dipakai listing)
- Semua sidenav admin sudah terhubung ke semua halaman

---

## 11. Yang Belum Dikerjakan (Prioritas Berikutnya)

Urutan yang disarankan:

1. **API mobile** — endpoint Sanctum: login user, daftar listing, create order (pakai `Order::generatePickupCode()` yang sudah ada). Ini prioritas utama karena order flow web sudah selesai.
2. **Withdrawal merchant** — tabel `withdrawals` sudah ada di DB, belum ada controller/view sama sekali. Alur: merchant ajukan pencairan → admin approve/reject + upload bukti transfer.
3. **Riwayat & pendapatan merchant** — halaman ringkasan pesanan selesai, total pendapatan bulan ini, saldo yang bisa dicairkan (dihitung dari payments paid - withdrawals completed).
4. **Peta lokasi merchant** — FR-07, belum diputuskan Google Maps atau Leaflet/OpenStreetMap.
5. **Review & rating** — FR-14, prioritas Low di SRS.

---

## 12. Seed Data untuk Testing

Tiga akun sudah ada di DB:

| Email | Password | Role |
|---|---|---|
| `user@test.com` | `password` | user |
| `SPPG@test.com` | `password` | merchant |
| `admin@test.com` | `password` | admin |

## Catatan Teknis Tambahan
 
**Order model — hal penting:**
- `CREATED_AT = 'ordered_at'` (bukan `created_at` standar)
- Pickup code: 8 karakter dari charset `ABCDEFGHJKLMNPQRSTUVWXYZ23456789`
- Stok dikembalikan saat merchant reject pesanan (loop `$order->items`, `increment('stock_qty', $item->quantity)`)
- Admin hanya bisa monitor — tidak bisa ubah status pesanan
**Merchant order controller — alur status:**
```
pending → confirm()  → confirmed
pending → reject()   → rejected  (stok dikembalikan)
confirmed → markReady() → ready
ready → complete()   → completed (butuh input pickup_code dari user, dicocokkan case-insensitive)
```
 
**Route order yang sudah ada:**
```
merchant.orders.index / show / confirm / reject / ready / complete
admin.orders.index / show
