# Panduan Lengkap API EcoEats
## Laravel Sanctum + Scramble + Integrasi Kotlin (Android)

---

## Daftar Isi

1. [Gambaran Besar Arsitektur](#1-gambaran-besar-arsitektur)
2. [Setup Laravel Sanctum](#2-setup-laravel-sanctum)
3. [Setup Scramble](#3-setup-scramble)
4. [Struktur Direktori Controller](#4-struktur-direktori-controller)
5. [Membuat API Controller Pertama (Auth)](#5-membuat-api-controller-pertama-auth)
6. [Cara Kerja Token & Request API](#6-cara-kerja-token--request-api)
7. [Integrasi di Android (Kotlin)](#7-integrasi-di-android-kotlin)
8. [Alur Kerja End-to-End](#8-alur-kerja-end-to-end)
9. [Checklist Setup](#9-checklist-setup)

---

## 1. Gambaran Besar Arsitektur

Sebelum masuk ke kode, pahami dulu **"siapa berkomunikasi dengan apa"** di EcoEats.

```
┌─────────────────────────────────────────────────────────────┐
│                     SERVER LARAVEL                           │
│                                                              │
│   routes/web.php          routes/api.php                     │
│   ┌──────────────┐        ┌──────────────────────────────┐   │
│   │ /login       │        │ /api/v1/auth/login           │   │
│   │ /merchant/.. │        │ /api/v1/food-listings        │   │
│   │ /admin/..    │        │ /api/v1/orders               │   │
│   │ /dashboard/..│        │ dst...                       │   │
│   └──────┬───────┘        └──────────────┬───────────────┘   │
│          │                               │                    │
│    Session Auth                    Token Auth                 │
│    (Cookie)                        (Sanctum)                  │
│          │                               │                    │
│          ▼                               ▼                    │
│   ┌─────────────┐              ┌──────────────────┐           │
│   │  Blade View │              │  JSON Response   │           │
│   │  (HTML)     │              │  (API)           │           │
│   └──────┬──────┘              └────────┬─────────┘           │
└──────────┼──────────────────────────────┼─────────────────────┘
           │                              │
           ▼                              ▼
   ┌───────────────┐             ┌─────────────────┐
   │ Browser       │             │ Android App     │
   │ (Admin &      │             │ (Kotlin)        │
   │  Merchant     │             │ — User role     │
   │  dashboard)   │             │ — Beli makanan  │
   └───────────────┘             └─────────────────┘
```

**Intinya:**
- **Browser** (admin & merchant) → pakai `routes/web.php` → auth via **session** (sudah ada, tidak diubah)
- **Android** (user pembeli) → pakai `routes/api.php` → auth via **token Sanctum**
- **Scramble** → otomatis membaca `routes/api.php` dan generate dokumentasi di `/docs/api`

---

## 2. Setup Laravel Sanctum

### 2.1 Apa itu Sanctum?

Sanctum adalah package resmi Laravel untuk **autentikasi berbasis token**. Cara kerjanya sederhana:

1. User login → Laravel buat token unik (string panjang)
2. Token disimpan di database (tabel `personal_access_tokens`)
3. Setiap request dari Android wajib menyertakan token ini di header
4. Laravel cek token → jika valid → request diizinkan

### 2.2 Install Sanctum

Di Laravel 11, cara yang **paling direkomendasikan** adalah lewat perintah artisan khusus, bukan manual lewat composer:

```bash
# Jalankan di folder root project (src/backend/)
# Perintah ini sekaligus: install Sanctum, buat routes/api.php, dan siapkan migrasi
php artisan install:api
```

Ketika diminta konfirmasi `One new database migration has been published. Would you like to run it now?`, ketik `yes`.

Perintah ini melakukan tiga hal sekaligus:
1. Install package `laravel/sanctum` via composer
2. Membuat file `routes/api.php` (yang di Laravel 11 tidak ada secara default)
3. Mempublikasikan dan menjalankan migrasi tabel `personal_access_tokens`

> Jika sudah terlanjur `composer require laravel/sanctum` secara manual, tetap jalankan `php artisan install:api` — tidak akan konflik, hanya melanjutkan langkah yang belum selesai.

### 2.3 Verifikasi Migrasi

Pastikan tabel `personal_access_tokens` sudah ada di database (cek via phpMyAdmin):

Setelah migrasi berhasil, akan ada tabel baru `personal_access_tokens` di database:

```
personal_access_tokens
├── id
├── tokenable_type   (biasanya App\Models\User)
├── tokenable_id     (id user yang punya token)
├── name             (nama token, misal "mobile")
├── token            (token yang di-hash, disimpan aman)
├── abilities
├── last_used_at
├── expires_at
└── created_at
```

### 2.4 Tambahkan Trait di Model User

Buka `app/Models/User.php`, tambahkan `HasApiTokens`:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens; // ← TAMBAHKAN INI

class User extends Authenticatable
{
    use HasApiTokens, Notifiable; // ← TAMBAHKAN HasApiTokens DI SINI

    // ✅ JANGAN HAPUS INI — penting untuk login karena kolom kita bukan 'password'
    public function getAuthPassword()
    {
        return $this->password_hash;
    }

    // ... sisa kode yang sudah ada tetap tidak diubah
}
```

> **Kenapa `getAuthPassword()` harus tetap ada?**
> Laravel secara default mencari kolom bernama `password`. Karena database EcoEats pakai `password_hash`, kita override method ini supaya Laravel tahu harus cek kolom yang mana. Jika dihapus, login akan selalu gagal.

### 2.5 Konfigurasi `config/sanctum.php`

Buka file `config/sanctum.php` yang baru saja di-publish. Cari bagian `expiration` dan sesuaikan:

```php
// Berapa menit token berlaku. null = tidak pernah expired.
// Untuk development, set null dulu. Untuk production bisa set 43200 (30 hari).
'expiration' => null,
```

### 2.6 Daftarkan Middleware Sanctum di Laravel 11

> ⚠️ **Laravel 11 tidak lagi punya `app/Http/Kernel.php`.** Semua konfigurasi middleware sekarang dilakukan di `bootstrap/app.php`.

Buka `bootstrap/app.php`, pastikan isinya seperti ini:

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',   // ← pastikan baris ini ada
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Daftarkan Sanctum sebagai middleware untuk guard 'api'
        $middleware->statefulApi();  // ← tambahkan ini
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

> **Apa itu `statefulApi()`?**
> Ini adalah shortcut di Laravel 11 yang secara otomatis menambahkan `EnsureFrontendRequestsAreStateful` dari Sanctum ke middleware group `api`. Efeknya sama dengan yang dulu dilakukan di `Kernel.php`, tapi caranya lebih ringkas.

> **Apakah `routes/api.php` otomatis ada?**
> Di Laravel 11, file `routes/api.php` **tidak dibuat secara default** — berbeda dengan Laravel 10. Jika belum ada, buat dulu filenya:
> ```bash
> php artisan install:api
> ```
> Perintah ini sekaligus membuat `routes/api.php` **dan** menginstall Sanctum sekaligus. Jika sudah menjalankan `composer require laravel/sanctum` manual, tetap jalankan perintah ini untuk memastikan file route dan konfigurasi terbuat dengan benar. Jika diminta konfirmasi migrasi, ketik `yes`.

---

## 3. Setup Scramble

### 3.1 Apa itu Scramble?

Scramble adalah tool yang **otomatis membaca kode Laravel** dan menghasilkan dokumentasi API dalam format OpenAPI (seperti Swagger). Tanpa menulis satu baris dokumentasi pun, Scramble bisa generate halaman interaktif di mana tim mobile bisa lihat semua endpoint, parameter, dan contoh response.

### 3.2 Install Scramble

```bash
composer require dedoc/scramble
```

### 3.3 Publish Config

```bash
php artisan vendor:publish --tag="scramble-config"
```

> Di Laravel 11, auto-discovery sudah berjalan otomatis sehingga tidak perlu menyebut nama provider secara manual.

### 3.4 Edit `config/scramble.php`

```php
<?php

return [
    // Scramble hanya akan baca route dengan prefix ini
    'api_path' => 'api',

    'api_domain' => null,

    // Info yang tampil di halaman dokumentasi
    'info' => [
        'title' => 'EcoEats API',
        'version' => '1.0.0',
        'description' => 'API untuk aplikasi mobile EcoEats. Platform makanan surplus Kota Solo.',
    ],

    'middleware' => [
        'web',
    ],

    'servers' => null,
];
```

### 3.5 Batasi Akses Docs (Opsional tapi Disarankan)

Di Laravel 11, `AppServiceProvider` masih ada di `app/Providers/AppServiceProvider.php`. Tambahkan konfigurasi Scramble di method `boot()`:

```php
<?php

namespace App\Providers;

use Dedoc\Scramble\Scramble;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Dokumentasi hanya bisa diakses di environment 'local' (development)
        Scramble::auth(function () {
            return app()->environment('local');
        });
    }
}
```

> Di Laravel 11, `AppServiceProvider` adalah satu-satunya provider yang tersisa secara default (provider lain seperti `RouteServiceProvider`, `AuthServiceProvider` sudah dihapus dan digabung ke `bootstrap/app.php`). Jadi pastikan kalian edit file yang benar ini.

### 3.6 Akses Dokumentasi

Setelah setup, buka browser dan akses:

| URL | Keterangan |
|---|---|
| `http://localhost:8000/docs/api` | Halaman UI interaktif (Swagger-like) |
| `http://localhost:8000/docs/api.json` | File JSON spec (untuk import ke Postman) |

---

## 4. Struktur Direktori Controller

Ini adalah struktur folder yang harus diikuti semua anggota tim:

```
app/Http/Controllers/
│
├── Auth/                          ← SUDAH ADA — jangan diubah
│   └── LoginController.php        ← Login web (session, Blade)
│
├── Api/                           ← BARU — khusus untuk mobile
│   ├── AuthController.php         ← Login/register/logout mobile
│   ├── FoodListingController.php  ← Katalog makanan (Alena)
│   ├── MerchantController.php     ← Data merchant & peta (Nabil)
│   ├── OrderController.php        ← Pemesanan (nanti)
│   └── CategoryController.php    ← Kategori makanan
│
└── Dashboard/                     ← SUDAH ADA — jangan diubah
    ├── UserDashboardController.php
    ├── MerchantDashboardController.php
    └── AdminDashboardController.php
```

**Aturan penting:**
- Semua controller di folder `Api/` hanya mengembalikan **JSON**, bukan view Blade
- Semua controller di folder `Auth/` dan `Dashboard/` tetap seperti sekarang, hanya mengembalikan **view Blade**
- Namespace controller Api: `App\Http\Controllers\Api`

---

## 5. Membuat API Controller Pertama (Auth)

### 5.1 Buat File Controller

```bash
# Buat folder Api jika belum ada
mkdir -p app/Http/Controllers/Api

# Atau buat via artisan
php artisan make:controller Api/AuthController
```

### 5.2 Isi `AuthController.php`

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    /**
     * Register user baru (khusus role 'user' — pembeli)
     */
    public function register(Request $request)
    {
        $request->validate([
            'name'     => 'required|string|max:100',
            'email'    => 'required|email|unique:users,email',
            'password' => 'required|string|min:8|confirmed', // butuh field password_confirmation
            'phone'    => 'nullable|string|max:20',
        ]);

        $user = User::create([
            'name'          => $request->name,
            'email'         => $request->email,
            'password_hash' => Hash::make($request->password), // simpan ke password_hash, bukan password
            'phone'         => $request->phone,
            'role'          => 'user', // selalu 'user' — merchant daftar lewat web
        ]);

        $token = $user->createToken('mobile')->plainTextToken;

        return response()->json([
            'message' => 'Registrasi berhasil.',
            'token'   => $token,
            'user'    => [
                'id'    => $user->id,
                'name'  => $user->name,
                'email' => $user->email,
                'phone' => $user->phone,
                'role'  => $user->role,
            ],
        ], 201);
    }

    /**
     * Login user mobile dan kembalikan token
     */
    public function login(Request $request)
    {
        $request->validate([
            'email'    => 'required|email',
            'password' => 'required|string',
        ]);

        // Cari user, pastikan tidak soft-deleted, dan hanya role 'user'
        $user = User::where('email', $request->email)
                    ->where('role', 'user')
                    ->whereNull('deleted_at')
                    ->first();

        // Cek apakah user ada dan password cocok
        // Ingat: kolom password kita adalah password_hash
        if (! $user || ! Hash::check($request->password, $user->password_hash)) {
            return response()->json([
                'message' => 'Email atau password salah.',
            ], 401);
        }

        // Hapus token lama (opsional — supaya tidak menumpuk)
        $user->tokens()->delete();

        // Buat token baru
        $token = $user->createToken('mobile')->plainTextToken;

        return response()->json([
            'message' => 'Login berhasil.',
            'token'   => $token,
            'user'    => [
                'id'    => $user->id,
                'name'  => $user->name,
                'email' => $user->email,
                'phone' => $user->phone,
                'role'  => $user->role,
            ],
        ]);
    }

    /**
     * Logout — hapus token aktif
     */
    public function logout(Request $request)
    {
        // Hapus hanya token yang dipakai sekarang
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logout berhasil.',
        ]);
    }

    /**
     * Ambil data profil user yang sedang login
     */
    public function me(Request $request)
    {
        return response()->json([
            'user' => $request->user(),
        ]);
    }
}
```

### 5.3 Setup `routes/api.php`

Buka (atau buat jika belum ada) `routes/api.php`:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\AuthController;
use App\Http\Controllers\Api\FoodListingController;
use App\Http\Controllers\Api\MerchantController;

/*
|--------------------------------------------------------------------------
| API Routes - EcoEats Mobile
|--------------------------------------------------------------------------
| Semua route di sini otomatis dapat prefix /api
| Tambahkan prefix v1 untuk versioning
*/

Route::prefix('v1')->group(function () {

    // ── Public Routes (tidak perlu login) ──────────────────────────────
    Route::post('/auth/register', [AuthController::class, 'register']);
    Route::post('/auth/login',    [AuthController::class, 'login']);

    // ── Protected Routes (wajib login, sertakan token) ─────────────────
    Route::middleware('auth:sanctum')->group(function () {

        // Auth
        Route::post('/auth/logout', [AuthController::class, 'logout']);
        Route::get('/auth/me',      [AuthController::class, 'me']);

        // Food Listings — katalog makanan (dikerjakan Alena)
        Route::get('/food-listings',      [FoodListingController::class, 'index']);
        Route::get('/food-listings/{id}', [FoodListingController::class, 'show']);

        // Merchants — data merchant & lokasi (dikerjakan Nabil)
        Route::get('/merchants',      [MerchantController::class, 'index']);
        Route::get('/merchants/{id}', [MerchantController::class, 'show']);

        // Orders — pemesanan (nanti)
        // Route::get('/orders',        [OrderController::class, 'index']);
        // Route::post('/orders',       [OrderController::class, 'store']);
        // Route::get('/orders/{id}',   [OrderController::class, 'show']);
    });
});
```

---

## 6. Cara Kerja Token & Request API

### 6.1 Flow Login

```
Android                          Laravel Server
  │                                    │
  │  POST /api/v1/auth/login           │
  │  Body: { email, password }         │
  │ ─────────────────────────────────► │
  │                                    │ 1. Cari user di DB
  │                                    │ 2. Hash::check(password, password_hash)
  │                                    │ 3. createToken('mobile')
  │                                    │ 4. Simpan token di personal_access_tokens
  │  Response: { token: "1|abc123..." }│
  │ ◄───────────────────────────────── │
  │                                    │
  │  Simpan token di SharedPreferences │
```

### 6.2 Flow Request dengan Token

```
Android                          Laravel Server
  │                                    │
  │  GET /api/v1/food-listings         │
  │  Header:                           │
  │    Authorization: Bearer 1|abc123  │
  │    Accept: application/json        │
  │ ─────────────────────────────────► │
  │                                    │ 1. Middleware auth:sanctum baca header
  │                                    │ 2. Cari token di DB
  │                                    │ 3. Token valid → lanjut ke controller
  │                                    │ 4. Controller query food_listings
  │  Response: { data: [...] }         │
  │ ◄───────────────────────────────── │
```

### 6.3 Format Response yang Konsisten

Biasakan semua API controller mengembalikan format ini:

```json
// Sukses (satu item)
{
    "message": "Berhasil.",
    "data": { ... }
}

// Sukses (banyak item)
{
    "message": "Berhasil.",
    "data": [ ... ]
}

// Error validasi (422)
{
    "message": "The given data was invalid.",
    "errors": {
        "email": ["Email sudah digunakan."],
        "password": ["Password minimal 8 karakter."]
    }
}

// Error auth (401)
{
    "message": "Unauthenticated."
}

// Error tidak ditemukan (404)
{
    "message": "Data tidak ditemukan."
}
```

---

## 7. Integrasi di Android (Kotlin)

### 7.1 Library yang Dibutuhkan

Tambahkan di `build.gradle` (module: app):

```groovy
dependencies {
    // Retrofit — untuk HTTP request ke API
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'

    // OkHttp — HTTP client yang dipakai Retrofit di balik layar
    implementation 'com.squareup.okhttp3:okhttp:4.12.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.12.0' // untuk debug log

    // Coroutines — supaya HTTP request tidak blocking UI
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3'

    // ViewModel & LiveData
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0'
    implementation 'androidx.lifecycle:lifecycle-luntimektx:2.7.0'
}
```

Sync Gradle setelah menambahkan dependencies.

### 7.2 Tambahkan Permission Internet

Di `AndroidManifest.xml`:

```xml
<manifest ...>
    <!-- Tambahkan ini sebelum tag <application> -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application ...>
        ...
    </application>
</manifest>
```

### 7.3 Buat Data Class (Model)

Buat file `data/model/AuthModels.kt`:

```kotlin
package com.ecoeats.app.data.model

import com.google.gson.annotations.SerializedName

// Request body untuk login
data class LoginRequest(
    val email: String,
    val password: String
)

// Request body untuk register
data class RegisterRequest(
    val name: String,
    val email: String,
    val password: String,
    @SerializedName("password_confirmation")
    val passwordConfirmation: String,
    val phone: String? = null
)

// Data user dari response
data class UserData(
    val id: Int,
    val name: String,
    val email: String,
    val phone: String?,
    val role: String
)

// Response dari endpoint login/register
data class AuthResponse(
    val message: String,
    val token: String,
    val user: UserData
)
```

Buat file `data/model/FoodListingModels.kt`:

```kotlin
package com.ecoeats.app.data.model

import com.google.gson.annotations.SerializedName

data class FoodListing(
    val id: Int,
    val name: String,
    val description: String?,
    @SerializedName("original_price")
    val originalPrice: Double,
    @SerializedName("discount_price")
    val discountPrice: Double,
    @SerializedName("stock_qty")
    val stockQty: Int,
    @SerializedName("photo_url")
    val photoUrl: String?,
    @SerializedName("pickup_start")
    val pickupStart: String,
    @SerializedName("pickup_end")
    val pickupEnd: String,
    val status: String,
    val category: Category?,
    val merchant: MerchantSummary?
)

data class Category(
    val id: Int,
    val name: String
)

data class MerchantSummary(
    val id: Int,
    @SerializedName("business_name")
    val businessName: String,
    @SerializedName("business_address")
    val businessAddress: String,
    val latitude: Double,
    val longitude: Double
)

// Response banyak item
data class FoodListingListResponse(
    val message: String,
    val data: List<FoodListing>
)
```

### 7.4 Buat API Interface

Buat file `data/remote/ApiService.kt`:

```kotlin
package com.ecoeats.app.data.remote

import com.ecoeats.app.data.model.*
import retrofit2.Response
import retrofit2.http.*

interface ApiService {

    // ── Auth ──────────────────────────────────────────────────────────

    @POST("v1/auth/login")
    suspend fun login(
        @Body request: LoginRequest
    ): Response<AuthResponse>

    @POST("v1/auth/register")
    suspend fun register(
        @Body request: RegisterRequest
    ): Response<AuthResponse>

    @POST("v1/auth/logout")
    suspend fun logout(
        @Header("Authorization") token: String
    ): Response<Map<String, String>>

    // ── Food Listings ─────────────────────────────────────────────────

    @GET("v1/food-listings")
    suspend fun getFoodListings(
        @Header("Authorization") token: String
    ): Response<FoodListingListResponse>

    @GET("v1/food-listings/{id}")
    suspend fun getFoodListingDetail(
        @Header("Authorization") token: String,
        @Path("id") id: Int
    ): Response<Map<String, Any>>

    // ── Merchants ─────────────────────────────────────────────────────

    @GET("v1/merchants")
    suspend fun getMerchants(
        @Header("Authorization") token: String
    ): Response<Map<String, Any>>
}
```

### 7.5 Buat Retrofit Instance

Buat file `data/remote/RetrofitClient.kt`:

```kotlin
package com.ecoeats.app.data.remote

import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit

object RetrofitClient {

    // ⚠️ GANTI INI sesuai environment:
    // Development (emulator Android): "http://10.0.2.2:8000/api/"
    // Development (device fisik, PC dan HP satu WiFi): "http://192.168.x.x:8000/api/"
    // Production: "https://api.ecoeats.com/api/"
    private const val BASE_URL = "http://10.0.2.2:8000/api/"

    // Logging untuk debug — tampilkan request dan response di Logcat
    private val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY // tampilkan full body
    }

    private val httpClient = OkHttpClient.Builder()
        .addInterceptor(loggingInterceptor)
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .build()

    val instance: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(httpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}
```

> **Kenapa `10.0.2.2`?**
> Emulator Android menganggap dirinya sebagai perangkat terpisah. `localhost` di dalam emulator = emulator itu sendiri, bukan PC kamu. Alamat `10.0.2.2` adalah cara emulator mengakses `localhost` di PC tempat emulator berjalan. Kalau pakai HP fisik, ganti dengan IP WiFi PC kamu (cek di `ipconfig` / `ifconfig`).

### 7.6 Simpan & Ambil Token (SharedPreferences)

Buat file `data/local/TokenManager.kt`:

```kotlin
package com.ecoeats.app.data.local

import android.content.Context
import android.content.SharedPreferences

class TokenManager(context: Context) {

    private val prefs: SharedPreferences =
        context.getSharedPreferences("ecoeats_prefs", Context.MODE_PRIVATE)

    companion object {
        private const val KEY_TOKEN = "auth_token"
        private const val KEY_USER_ID = "user_id"
        private const val KEY_USER_NAME = "user_name"
        private const val KEY_USER_EMAIL = "user_email"
    }

    // Simpan token setelah login berhasil
    fun saveToken(token: String) {
        prefs.edit().putString(KEY_TOKEN, token).apply()
    }

    // Ambil token — dipakai di setiap request API
    fun getToken(): String? {
        return prefs.getString(KEY_TOKEN, null)
    }

    // Format Bearer Token untuk header Authorization
    fun getBearerToken(): String {
        return "Bearer ${getToken()}"
    }

    // Simpan data user
    fun saveUserData(id: Int, name: String, email: String) {
        prefs.edit()
            .putInt(KEY_USER_ID, id)
            .putString(KEY_USER_NAME, name)
            .putString(KEY_USER_EMAIL, email)
            .apply()
    }

    // Cek apakah user sudah login
    fun isLoggedIn(): Boolean {
        return getToken() != null
    }

    // Hapus semua data saat logout
    fun clearAll() {
        prefs.edit().clear().apply()
    }
}
```

### 7.7 Contoh Penggunaan di ViewModel

Buat file `ui/auth/LoginViewModel.kt`:

```kotlin
package com.ecoeats.app.ui.auth

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.ecoeats.app.data.model.LoginRequest
import com.ecoeats.app.data.remote.RetrofitClient
import kotlinx.coroutines.launch

// Sealed class untuk representasi state UI
sealed class LoginState {
    object Loading : LoginState()
    data class Success(val name: String) : LoginState()
    data class Error(val message: String) : LoginState()
}

class LoginViewModel : ViewModel() {

    private val _loginState = MutableLiveData<LoginState>()
    val loginState: LiveData<LoginState> = _loginState

    fun login(email: String, password: String, onTokenSaved: (String, Int, String, String) -> Unit) {
        _loginState.value = LoginState.Loading

        viewModelScope.launch {
            try {
                val response = RetrofitClient.instance.login(
                    LoginRequest(email = email, password = password)
                )

                if (response.isSuccessful) {
                    val body = response.body()!!
                    // Panggil callback untuk simpan token
                    onTokenSaved(body.token, body.user.id, body.user.name, body.user.email)
                    _loginState.value = LoginState.Success(body.user.name)
                } else {
                    // Error dari server (401, 422, dst)
                    _loginState.value = LoginState.Error("Email atau password salah.")
                }
            } catch (e: Exception) {
                // Error koneksi (no internet, timeout, dst)
                _loginState.value = LoginState.Error("Tidak bisa terhubung ke server.")
            }
        }
    }
}
```

### 7.8 Contoh Penggunaan di Activity/Fragment

```kotlin
package com.ecoeats.app.ui.auth

import android.content.Intent
import android.os.Bundle
import android.view.View
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import com.ecoeats.app.data.local.TokenManager
import com.ecoeats.app.databinding.ActivityLoginBinding
import com.ecoeats.app.ui.home.HomeActivity

class LoginActivity : AppCompatActivity() {

    private lateinit var binding: ActivityLoginBinding
    private lateinit var tokenManager: TokenManager
    private val viewModel: LoginViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityLoginBinding.inflate(layoutInflater)
        setContentView(binding.root)

        tokenManager = TokenManager(this)

        // Jika sudah login, langsung ke HomeActivity
        if (tokenManager.isLoggedIn()) {
            goToHome()
            return
        }

        setupObservers()
        setupClickListeners()
    }

    private fun setupObservers() {
        viewModel.loginState.observe(this) { state ->
            when (state) {
                is LoginState.Loading -> {
                    binding.progressBar.visibility = View.VISIBLE
                    binding.btnLogin.isEnabled = false
                }
                is LoginState.Success -> {
                    binding.progressBar.visibility = View.GONE
                    goToHome()
                }
                is LoginState.Error -> {
                    binding.progressBar.visibility = View.GONE
                    binding.btnLogin.isEnabled = true
                    binding.tvError.text = state.message
                    binding.tvError.visibility = View.VISIBLE
                }
            }
        }
    }

    private fun setupClickListeners() {
        binding.btnLogin.setOnClickListener {
            val email = binding.etEmail.text.toString().trim()
            val password = binding.etPassword.text.toString()

            if (email.isEmpty() || password.isEmpty()) {
                binding.tvError.text = "Email dan password tidak boleh kosong."
                binding.tvError.visibility = View.VISIBLE
                return@setOnClickListener
            }

            viewModel.login(email, password) { token, id, name, userEmail ->
                // Simpan token dan data user ke SharedPreferences
                tokenManager.saveToken(token)
                tokenManager.saveUserData(id, name, userEmail)
            }
        }
    }

    private fun goToHome() {
        startActivity(Intent(this, HomeActivity::class.java))
        finish() // supaya tombol back tidak kembali ke halaman login
    }
}
```

---

## 8. Alur Kerja End-to-End

Berikut gambaran lengkap dari mulai user buka app sampai berhasil lihat daftar makanan:

```
1. USER BUKA APP
   └── LoginActivity cek tokenManager.isLoggedIn()
       ├── Ada token → langsung ke HomeActivity
       └── Tidak ada → tampilkan form login

2. USER ISI FORM & KLIK LOGIN
   └── LoginViewModel.login() dipanggil
       └── Retrofit POST ke /api/v1/auth/login
           ├── Server validasi email & password
           ├── Server cek deleted_at IS NULL
           ├── Server buat token di personal_access_tokens
           └── Server return { token, user }

3. SIMPAN TOKEN
   └── TokenManager.saveToken(token)
       └── Disimpan di SharedPreferences (tetap ada walaupun app ditutup)

4. PINDAH KE HOME
   └── HomeActivity load daftar makanan
       └── Retrofit GET /api/v1/food-listings
           Header: Authorization: Bearer 1|xyz...
           ├── Middleware auth:sanctum cek token
           ├── Token valid → lanjut ke FoodListingController
           └── Return { data: [ {id, name, price, ...}, ... ] }

5. TAMPILKAN DATA
   └── RecyclerView populate dengan data listing
```

---

## 9. Checklist Setup

Gunakan checklist ini untuk memastikan semua sudah dilakukan dengan benar:

### Laravel (Backend)
- [ ] `php artisan install:api` (install Sanctum + buat `routes/api.php` + migrasi sekaligus)
- [ ] Tabel `personal_access_tokens` sudah muncul di phpMyAdmin
- [ ] `bootstrap/app.php` sudah ada `->withRouting(api: ...)` dan `$middleware->statefulApi()`
- [ ] Model `User.php` sudah ada `use HasApiTokens` **dan** method `getAuthPassword()` tetap ada
- [ ] `composer require dedoc/scramble`
- [ ] `php artisan vendor:publish --tag="scramble-config"`
- [ ] `config/scramble.php` sudah diisi info EcoEats
- [ ] `AppServiceProvider.php` sudah ada `Scramble::auth(...)` untuk batasi akses docs
- [ ] Folder `app/Http/Controllers/Api/` sudah dibuat
- [ ] `AuthController.php` sudah ada di folder `Api/`
- [ ] `routes/api.php` sudah berisi route auth dan group middleware `auth:sanctum`
- [ ] Akses `http://localhost:8000/docs/api` dan halaman terbuka

### Android (Kotlin)
- [ ] Dependencies Retrofit, OkHttp, Coroutines sudah ditambah di `build.gradle`
- [ ] `INTERNET` permission sudah ada di `AndroidManifest.xml`
- [ ] `RetrofitClient.kt` sudah dibuat, BASE_URL sudah benar
- [ ] `ApiService.kt` sudah dibuat dengan endpoint login
- [ ] `TokenManager.kt` sudah dibuat
- [ ] Login berhasil dan token tersimpan
- [ ] Request dengan token berhasil (coba GET food-listings)

---

## Catatan Akhir

**Pembagian kerja dengan panduan ini:**

| Anggota | Yang perlu dilakukan |
|---|---|
| **Semua** | Setup Sanctum & Scramble (langkah 2 & 3) — lakukan sekali bersama |
| **Wiwid** | Buat `Api/FoodListingController` untuk CRUD listing (endpoint merchant) |
| **Alena** | Buat `Api/FoodListingController` untuk index & show (endpoint user/mobile) |
| **Maria** | Buat `Api/MerchantController` untuk data merchant |
| **Nabil** | Buat `Api/MerchantController` untuk data lokasi & koordinat |

**Komunikasi data antara Laravel dan Kotlin selalu dalam format JSON.** Laravel mengirim JSON, Kotlin menerimanya dan mengubah ke data class menggunakan Gson (bagian dari Retrofit).
