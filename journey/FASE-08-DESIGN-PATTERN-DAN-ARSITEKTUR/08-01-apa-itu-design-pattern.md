# 08-01: Apa Itu Design Pattern — Blueprint Solusi

> **Fase**: 8 — Design Pattern & Arsitektur  
> **Prasyarat**: FASE 3 (OOP)  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: design pattern, GoF, creational, structural, behavioral, anti-pattern, pattern di Laravel

---

## 📋 Ringkasan

Design pattern adalah **solusi umum untuk masalah umum** dalam pengembangan software. Bukan kode siap pakai, tapi **template** yang bisa diadaptasi. Laravel sendiri menggunakan banyak design pattern.

**Target pemahaman:**
- Kamu paham apa itu design pattern dan kenapa penting
- Kamu kenal kategori pattern (creational, structural, behavioral)
- Kamu bisa mengidentifikasi pattern yang dipakai Laravel
- Kamu paham anti-pattern yang harus dihindari

---

## 1. Design Pattern — Bukan Resep, Tapi Template

### 1.1 Analogi Arsitektur

```
Masalah:     Kamu mau bangun rumah.
             Setiap rumah punya pintu, jendela, atap.
             Tapi detailnya bisa beda.

Pattern:     "Rumah punya ruang tamu di depan, kamar tidur di belakang"
             → Ini pattern — bukan cetak biru spesifik

Implementasi:
             Rumah A: ruang tamu 5×5, kamar 4×4
             Rumah B: ruang tamu 3×6, kamar 3×3
             → Detailnya beda, tapi pattern-nya sama
```

### 1.2 Design Pattern di Software

```
Masalah:     Kamu perlu memastikan hanya SATU instance database
             yang dibuat di seluruh aplikasi.

Pattern:     Singleton → "buat object sekali saja, pakai lagi"

Solusi:
class Database {
    private static $instance = null;
    public static function getInstance(): self {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}
```

### 1.3 Kenapa Design Pattern?

| Tanpa Pattern | Dengan Pattern |
|--------------|---------------|
| "Saya bikin sendiri" | "Saya pakai Singleton" |
| Sulit dijelaskan ke tim | Semua programmer paham |
| Bisa salah desain | Terbukti berhasil |
| Sulit di-maintain | Mudah di-extend |

---

## 2. Kategori Design Pattern (GoF)

Gang of Four (GoF) — buku klasik yang mengkategorikan 23 pattern:

### 2.1 Creational — Cara Membuat Object

| Pattern | Guna | Di Laravel |
|---------|------|-----------|
| **Singleton** | Satu instance global | `app()->singleton()` |
| **Factory** | Buat object tanpa specify class | `Database::factory()` |
| **Builder** | Bangun object kompleks step-by-step | Query Builder |
| **Prototype** | Clone object | `clone` keyword |
| **DI Container** | Kelola pembuatan + dependency | Service Container |

### 2.2 Structural — Komposisi Class/Object

| Pattern | Guna | Di Laravel |
|---------|------|-----------|
| **Adapter** | Interface berbeda → bisa协作 | Filesystem (local/S3) |
| **Decorator** | Tambah behavior tanpa ubah class | Middleware |
| **Facade** | Interface sederhana ke subsystem | `Cache::get()` |
| **Proxy** | Pengganti object (lazy, cache) | Lazy loading Eloquent |
| **Composite** | Tree structure | Blade component |

### 2.3 Behavioral — Komunikasi Antar Object

| Pattern | Guna | Di Laravel |
|---------|------|-----------|
| **Observer** | Notify perubahan ke banyak object | Events + Listeners |
| **Strategy** | Pilih algoritma di runtime | Mail drivers |
| **Template Method** | Kerangka method, detail di subclass | Blade layout |
| **Chain of Responsibility** | Request lewat rantai handler | Middleware pipeline |
| **Command** | Enkapsulasi request sebagai object | Queue jobs |

---

## 3. Pattern di Laravel

### 3.1 Service Container — Dependency Injection (DI)

```php
// Pattern: DI Container + Singleton + Factory
$this->app->singleton(CartService::class, function ($app) {
    return new CartService($app->make(CustomerService::class));
});

// Pemakaian:
$cartService = app(CartService::class); // selalu instance yang sama
```

### 3.2 Facade — Structural

```php
// Pattern: Facade — menyembunyikan kompleksitas
// Daripada:
$cache = app('cache')->get('key');

// Jadi:
Cache::get('key');
```

### 3.3 Pipeline — Chain of Responsibility

```php
// Pattern: Chain of Responsibility
// Middleware dipanggil berantai
// Setiap middleware bisa:
// - Meneruskan request (return $next($request))
// - Menghentikan (return response(...))

class AuthMiddleware {
    public function handle($request, $next) {
        if (!auth()->check()) {
            return redirect('login'); // STOP
        }
        return $next($request); // LANJUT
    }
}
```

### 3.4 Observer — Event & Listener

```php
// Pattern: Observer
// Event → banyak Listener bisa merespon

class OrderCreated {
    public function __construct(public Order $order) {}
}

// Listeners:
class SendOrderConfirmation  // kirim email
class UpdateInventory        // update stok
class LogOrderActivity       // log aktivitas
```

### 3.5 Strategy — Mail Driver

```php
// Pattern: Strategy — pilih algoritma di runtime
// Mail bisa dikirim via SMTP, Mailgun, Log, SES
// Tinggal ganti MAIL_MAILER di .env

config(['mail.default' => 'mailgun']); // ganti strategy
```

---

## 4. Anti-Pattern — yang Harus Dihindari

### 4.1 God Object

```php
// ❌ God Object — satu class melakukan SEMUA
class OrderManager {
    public function createOrder() { ... }
    public function sendEmail() { ... }
    public function generateInvoice() { ... }
    public function updateStock() { ... }
    public function processPayment() { ... }
    public function sendNotification() { ... }
}

// ✅ Pisah per tanggung jawab:
OrderService, MailService, InvoiceService, dll.
```

### 4.2 Spaghetti Code

```php
// ❌ Semua campur di controller
public function index() {
    $products = DB::select('SELECT * FROM products');
    $html = '<html>';
    foreach ($products as $p) {
        $html .= '<h1>' . $p->name . '</h1>';
    }
    $html .= '</html>';
    return $html;
}

// ✅ Pisah: Controller → Service → View
```

### 4.3 Copy-Paste Programming

```php
// ❌ Copy-paste query yang sama
public function activeProducts() {
    return Product::where('is_active', true)->get();
}
public function featuredProducts() {
    return Product::where('is_active', true)
        ->where('is_featured', true)->get();
}

// ✅ Pakai scope
public function scopeActive($q) {
    return $q->where('is_active', true);
}
public function scopeFeatured($q) {
    return $q->where('is_featured', true);
}
```

---

## 5. Pattern yang Paling Terlihat di Codebase

| Pattern | Lokasi | Contoh |
|---------|--------|--------|
| DI Container | Seluruh app | Constructor injection |
| Facade | Helper | `Cache::`, `Storage::` |
| Observer | Events | Order event → send email |
| Strategy | Config | Mail driver, cache driver |
| Chain of Resp. | Kernel | Middleware stack |
| Template Method | Blade | Layout → section yield |
| Factory | Database | Model factories |
| Singleton | Providers | `$this->app->singleton()` |

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ DESIGN PATTERN — 3 KATEGORI                                       │
├─────────────────────────────────────────────────────────────────────┤
│ CREATIONAL (bikin object)                                          │
│  Singleton → 1 instance global                                     │
│  Factory   → bikin object tanpa specify class                     │
│  Builder   → bangun object step-by-step                           │
├─────────────────────────────────────────────────────────────────────┤
│ STRUCTURAL (komposisi)                                             │
│  Facade    → interface sederhana                                   │
│  Adapter   → cocokkan interface berbeda                            │
│  Decorator → tambah fungsi tanpa ubah class                       │
├─────────────────────────────────────────────────────────────────────┤
│ BEHAVIORAL (komunikasi)                                            │
│  Observer  → beri tahu banyak object                              │
│  Strategy  → pilih algoritma di runtime                           │
│  Chain of Resp. → request lewat rantai handler                   │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  DI Container → semua constructor injection                       │
│  Middleware   → Chain of Responsibility                           │
│  Facade       → Cache, Storage, DB                               │
│  Observer     → Event & Listener (jika ada)                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Identifikasi pattern.** Di codebase, cari contoh:
   - DI Container (constructor injection di controller/service)
   - Facade (panggil static method seperti `Cache::`, `Storage::`)
   - Singleton (cari `->singleton()` di ServiceProvider)

2. **Middleware chain.** Buka `app/Http/Kernel.php`. Identifikasi Chain of Responsibility — middleware apa yang dilewati request?

3. **Strategy pattern.** Buka `config/mail.php`. Driver apa yang tersedia? Ganti driver → behavior berubah tanpa ubah kode.

4. **Cari anti-pattern.** Apakah ada controller yang terlalu gemuk (fat controller)? Jika ya, bagaimana memperbaikinya?

5. **Factory pattern.** Buka `database/factories/`. Factory apa yang ada? Bagaimana cara kerjanya?

---

## 🔗 Referensi

- [GoF Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns)
- [Laravel: Design Patterns](https://laravel.com/docs/11.x/architecture)
- [Refactoring Guru: Design Patterns](https://refactoring.guru/design-patterns)
- Codebase: `app/Providers/` — Service Container patterns
- Codebase: `app/Http/Kernel.php` — Chain of Responsibility
