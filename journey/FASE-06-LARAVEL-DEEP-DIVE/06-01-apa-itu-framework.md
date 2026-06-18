# 06-01: Apa Itu Framework — Mengapa Laravel?

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: FASE 3 (OOP) + FASE 5 (Web Fundamental)  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: framework, library, convention over configuration, DRY, MVC, Laravel vs PHP polos

---

## 📋 Ringkasan

Kamu sudah menulis PHP, OOP, dan paham cara kerja web. Tapi kenapa kita pakai Laravel? Kenapa tidak PHP polos saja? Dokumen ini menjawab: **apa itu framework** dan **mengapa Laravel** menjadi pilihan utama untuk web development modern.

**Target pemahaman:**
- Kamu paham perbedaan framework vs library
- Kamu tahu masalah apa yang framework selesaikan
- Kamu paham konsep Convention over Configuration
- Kamu bisa menjelaskan keuntungan Laravel spesifik untuk codebase ini

---

## 1. Masalah: PHP Polos (Tanpa Framework)

### 1.1 Kode Berantakan

```php
<?php
// index.php — semuanya campur aduk!

// Koneksi database
$pdo = new PDO('mysql:host=localhost;dbname=shop', 'root', '');

// Cek URL
$path = $_SERVER['REQUEST_URI'];

// Routing manual
if ($path === '/shop' && $_SERVER['REQUEST_METHOD'] === 'GET') {
    // Query database
    $stmt = $pdo->query('SELECT * FROM products WHERE is_active = 1');
    $products = $stmt->fetchAll();

    // Render HTML
    ?>
    <!DOCTYPE html>
    <html>
    <head><title>Shop</title></head>
    <body>
        <?php foreach ($products as $p): ?>
            <h2><?= $p['name'] ?></h2>
            <p><?= $p['price'] ?></p>
        <?php endforeach; ?>
    </body>
    </html>
    <?php
} elseif ($path === '/cart' && $_SERVER['REQUEST_METHOD'] === 'POST') {
    // Logic cart...
}
?>
```

**Masalah:**
1. **Routing manual** — setiap tambah halaman harus edit if-else
2. **Koneksi DB manual** — harus buat ulang di setiap halaman
3. **Query tanpa proteksi** — SQL injection mengintai
4. **View campur logic** — HTML di dalam PHP
5. **No reusability** — Copy-paste koneksi DB, header, footer
6. **Security** — CSRF, XSS, session handling manual

### 1.2 Masalah di Proyek Nyata

| Aspek | PHP Polos | Laravel |
|-------|-----------|---------|
| Routing | `if/else` manual | `Route::get()` |
| Database | `PDO` manual | Eloquent ORM |
| Template | `<?php ?>` campur | Blade (clean) |
| Form validation | `if(empty($_POST['x']))` | FormRequest class |
| Security | Manual CSRF token | Built-in CSRF |
| Session | `$_SESSION` | `session()` helper |
| Auth | Manual hash + session | `make:auth` |
| File structure | Terserah (bebas) | `app/`, `resources/`, dll |
| Testing | Tidak ada | PHPUnit + Pest |

---

## 2. Framework vs Library

### 2.1 Perbedaan Fundamental

| | Library | Framework |
|---|---------|-----------|
| **Kontrol** | Kamu panggil library | Framework panggil kode kamu |
| **Inversion of Control** | ❌ Kamu kontrol | ✅ Framework kontrol |
| **Arsitektur** | Tidak menentukan | Menentukan (ikuti aturannya) |
| **Contoh** | `Carbon::now()` | Laravel, Symfony, Rails |

**Library:** "Kamu yang pegang kendali, library bantu tugas tertentu."
```php
// Kamu panggil Carbon — kamu kontrol kapan pakainya
echo Carbon::now()->format('Y-m-d');
```

**Framework:** "Framework pegang kendali, kamu isi bagian tertentu."
```php
// Framework panggil controller kamu
class ShopController extends Controller {
    public function index() {
        // Framework sudah handle routing, request, session, dll.
        // Kamu tinggal isi logika di sini
    }
}
```

### 2.2 Inversion of Control (IoC)

Prinsip ini disebut **Hollywood Principle**: "Don't call us, we'll call you."

```
Library:       Kode Kamu ──panggil──► Library
Framework:     Framework ──panggil──► Kode Kamu
```

Laravel memanggil kode kamu di **controller**, **middleware**, **event listener**, dll. Kamu tidak perlu memanggil framework — framework yang mengatur alur.

---

## 3. Convention over Configuration

### 3.1 Konsep

**Convention over Configuration** berarti: ikuti aturan yang sudah ada → tidak perlu konfigurasi. Hanya perlu konfigurasi jika ingin berbeda dari default.

### 3.2 Contoh Convention

| Convention | Tanpa Framework | Laravel |
|------------|----------------|---------|
| Database table | Harus tulis nama tabel | `Product` model → `products` table (otomatis) |
| Controller | Terserah | `ProductController` di `app/Http/Controllers/` |
| View | Terserah | `shop.index` → `resources/views/shop/index.blade.php` |
| Route file | Terserah | `routes/web.php` untuk web, `routes/api.php` untuk API |
| Auth | Manual | `php artisan make:auth` + `auth()` helper |

### 3.3 Contoh di Codebase

```php
// Tanpa convention, kamu harus tulis:
protected $table = 'products';
protected $primaryKey = 'id';
public $timestamps = true;
const CREATED_AT = 'created_at';
const UPDATED_AT = 'updated_at';

// Dengan convention, cukup:
class Product extends Model {}
// Laravel otomatis tahu:
// - Table: "products" (plural dari Product)
// - Primary key: "id"
// - Timestamps: "created_at", "updated_at"
```

---

## 4. Kenapa Laravel?

### 4.1 Ekosistem Lengkap

```
LARAVEL ECOSYSTEM
├── Routing        → web.php, api.php
├── ORM            → Eloquent (Active Record)
├── Template       → Blade
├── Auth           → Built-in authentication
├── Queue          → Redis, database, SQS
├── Mail           → SMTP, Mailgun, Postmark
├── Notifications  → Database, mail, SMS
├── Cache          → Redis, file, database
├── Filesystem     → Local, S3, FTP
├── Testing        → PHPUnit, Pest
├── Task scheduling → Cron (Artisan schedule)
├── Broadcasting   → WebSocket (Pusher, Reverb)
├── Scout          → Full-text search (Algolia, Meilisearch)
└── Telescope      → Debugging (digunakan di codebase ini!)
```

### 4.2 Keuntungan untuk Codebase Ini

| Fitur | Dipakai di Codebase? | File |
|-------|---------------------|------|
| Eloquent ORM | ✅ Semua model | `app/Models/` |
| Blade | ✅ Semua view | `resources/views/` |
| Routing | ✅ `web.php` | `routes/web.php` |
| Controller | ✅ Semua controller | `app/Http/Controllers/` |
| Middleware | ✅ `auth`, `guest`, `verified`, `role` | `app/Http/Kernel.php` |
| FormRequest | ✅ CheckoutRequest | `app/Http/Requests/` |
| Service Layer | ✅ CartService, RajaOngkirService, dll | `app/Services/` |
| Migration | ✅ 25 migration | `database/migrations/` |
| Seeder | ✅ DatabaseSeeder | `database/seeders/` |
| Mail/Notif | ✅ Invoice email | `app/Mail/` |
| Queue | ✅ Midtrans status check | `app/Jobs/` |
| Telescope | ✅ Debugging | `config/telescope.php` |
| Media Library | ✅ Gambar produk | spatie package |
| Auth | ✅ Login/register | `app/Http/Controllers/Auth/` |

---

## 5. MVC — Model View Controller

### 5.1 Konsep MVC

Laravel mengikuti pola **MVC** (Model-View-Controller):

```
User ──request──► Route ──► Controller ──► Model ──► Database
                          │                   │
                          │                   └── Query data
                          ▼
                       View (Blade)
                          │
                          └── HTML ke browser
```

| Layer | Tugas | Di Laravel |
|-------|-------|-----------|
| **Model** | Data & bisnis logic | `app/Models/` (Eloquent) |
| **View** | Tampilan (HTML) | `resources/views/` (Blade) |
| **Controller** | Jembatan Model ↔ View | `app/Http/Controllers/` |

### 5.2 Alur MVC di Codebase

```php
// 1. Route → URL /shop → ShopController@index
Route::get('/shop', [ShopController::class, 'index'])->name('shop.index');

// 2. Controller → query data dari Model
class ShopController extends Controller
{
    public function index(ShopRequest $request): View
    {
        $products = Product::with(['category', 'brand'])  // Model
            ->where('is_active', true)
            ->get();

        return view('shop.index', compact('products'));   // View
    }
}

// 3. Model → representasi table products
class Product extends Model
{
    public function category(): BelongsTo { ... }
    public function brand(): BelongsTo { ... }
}

// 4. View → tampilkan data
// resources/views/shop/index.blade.php
@extends('layouts.app')
@section('content')
    @foreach($products as $product)
        <div class="card">{{ $product->name }}</div>
    @endforeach
@endsection
```

### 5.3 Kenapa MVC?

| Tanpa MVC (Campur aduk) | Dengan MVC (Pisah) |
|-------------------------|-------------------|
| Satu file `index.php` berisi semua | Model, View, Controller terpisah |
| Ganti tampilan → otak-atik query | Ganti view saja |
| Ganti database → otak-atik HTML | Ganti model saja |
| Testing susah | Testing per layer |
| 2 programmer susah kerja bareng | 2 programmer bisa kerja paralel |

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ FRAMEWORK VS LIBRARY                                               │
│  Library:   Kamu panggil library                                   │
│  Framework: Framework panggil kode kamu (IoC)                     │
├─────────────────────────────────────────────────────────────────────┤
│ MASALAH YANG FRAMEWORK SELESAIKAN:                                 │
│  ✅ Routing — tidak perlu if/else manual                          │
│  ✅ Database — ORM, migrasi, seeder                                │
│  ✅ Security — CSRF, XSS, SQL injection                           │
│  ✅ Template — Blade, component, layout                            │
│  ✅ Validation — FormRequest, rules                                │
│  ✅ Auth — Login, register, password reset                        │
│  ✅ Testing — PHPUnit, Pest                                       │
├─────────────────────────────────────────────────────────────────────┤
│ LARAVEL UNTUK CODEBASE INI:                                        │
│  Eloquent → 20+ model (Product, Order, User, dll)                 │
│  Blade    → Layout, components, partials                          │
│  Services → Cart, Order, Payment, RajaOngkir, Midtrans            │
│  Queue    → Midtrans status check                                 │
│  Telescope → Debugging & monitoring                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Identifikasi framework.** Buka `composer.json`. Cari `require` — package apa saja yang merupakan framework/laravel package?

2. **Cari convention.** Buka `app/Models/Order.php`. Tanpa lihat property `$table`, apa nama tabel yang Laravel asumsikan? Cek di database via Laragon → MySQL untuk verifikasi.

3. **IoC di codebase.** Buka `app/Services/CartService.php`. Constructor menerima `CustomerService` — siapa yang memanggil constructor ini? (Laravel Service Container)

4. **Tanpa framework.** Bayangkan fitur filter produk di ShopController — berapa baris kode jika pakai PHP polos dengan if/else routing? Bandingkan dengan implementasi Laravel saat ini.

5. **Eksplorasi.** Buka `config/app.php`. Cari `providers` — ini adalah service providers yang Laravel load. Berapa jumlahnya?

---

## 🔗 Referensi

- [Laravel Docs: Architecture Concepts](https://laravel.com/docs/11.x/architecture)
- [Wikipedia: MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)
- [Wikipedia: Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control)
- Codebase: `composer.json` — dependencies
- Codebase: `config/app.php` — service providers
