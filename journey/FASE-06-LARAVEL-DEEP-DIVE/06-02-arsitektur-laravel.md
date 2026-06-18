# 06-02: Arsitektur Laravel — Perjalanan Request

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-01-apa-itu-framework  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: lifecycle, bootstrap, service container, service provider, facade, artisan, kernel, request lifecycle

---

## 📋 Ringkasan

Setelah paham apa itu framework, sekarang kita bedah **arsitektur Laravel** — bagaimana request masuk sampai response keluar, bagaimana service container bekerja, dan apa peran service provider.

**Target pemahaman:**
- Kamu bisa menjelaskan lifecycle request Laravel
- Kamu paham peran Service Container
- Kamu paham Service Provider
- Kamu tahu perbedaan facade vs helper

---

## 1. Request Lifecycle — Perjalanan Lengkap

### 1.1 Diagram

```
Request ──► public/index.php
                │
                ▼
           Bootstrap:     autoload, app instance, kernel
                │
                ▼
           Service Providers: register → boot
                │
                ▼
           Middleware (global)
                │
                ▼
           Route Matching
                │
                ▼
           Middleware (route)
                │
                ▼
           Controller
                │
                ▼
           Response ──► Browser
```

### 1.2 Step-by-Step

**Step 0: Apache menerima request**
```
Client GET /shop → Apache → public/index.php
```

**Step 1: Autoload**
```php
// public/index.php — baris pertama
require __DIR__.'/../vendor/autoload.php';
// Composer autoload — load semua class Laravel
```

**Step 2: App Instance**
```php
$app = require_once __DIR__.'/../bootstrap/app.php';
// Buat instance Application (IoC Container)
// Binding inti: app, config, kernel, exception handler
```

**Step 3: HTTP Kernel**
```php
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
// Ambil App\Http\Kernel dari container
```

**Step 4: Handle Request**
```php
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
// Ini yang BUTUH WAKTU:
// 1. Bootstrap semua Service Provider
// 2. Jalankan global middleware
// 3. Route request → controller
// 4. Jalankan route middleware
```

**Step 5: Send Response**
```php
$response->send();
// Kirim HTTP response ke browser (header + body)
```

**Step 6: Terminate**
```php
$kernel->terminate($request, $response);
// Call middleware terminate (cleanup)
```

### 1.3 Visual Full Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│ LARAVEL REQUEST LIFECYCLE                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  public/index.php                                                   │
│    │                                                                │
│    ▼                                                                │
│  bootstrap/app.php → Application instance                          │
│    │                                                                │
│    ▼                                                                │
│  Kernel::handle()                                                   │
│    │                                                                │
│    ├── 1. Register Service Providers (config/app.php)              │
│    │    ├── register() → binding ke container                      │
│    │    └── boot() → event, route, view composer                  │
│    │                                                                │
│    ├── 2. Global Middleware                                         │
│    │    ├── EncryptCookies                                         │
│    │    ├── StartSession                                            │
│    │    ├── ShareErrorsFromSession                                  │
│    │    ├── SubstituteBindings                                      │
│    │    └── ...                                                     │
│    │                                                                │
│    ├── 3. Route Matching                                            │
│    │    ├── RouteServiceProvider → routes/web.php                  │
│    │    └── Match: GET /shop → ShopController@index               │
│    │                                                                │
│    ├── 4. Route Middleware                                          │
│    │    ├── web (group middleware)                                 │
│    │    ├── auth, guest, verified, role (if specified)             │
│    │    └── throttle, signed (if specified)                        │
│    │                                                                │
│    ├── 5. Controller                                                │
│    │    ├── Dependency Injection (via container)                   │
│    │    ├── FormRequest (validation auto)                          │
│    │    └── Return response (view/json/redirect)                   │
│    │                                                                │
│    └── 6. Response sent to browser                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Service Container — Mesin Dependency Injection

### 2.1 Apa Itu Service Container?

Service Container adalah **wadah** yang mengelola pembuatan object dan dependency injection.

```php
// Tanpa container — Kamu buat manual
$service = new RajaOngkirService(new HttpRequest(), 'api-key');

// Dengan container — Laravel buatkan
$service = app(RajaOngkirService::class);
// Container lihat constructor → resolusi dependency → buat → inject
```

### 2.2 Binding — Mendaftarkan ke Container

```php
// app/Providers/AppServiceProvider.php

// Simple binding — buat instance baru setiap kali
$this->app->bind(RajaOngkirService::class, function ($app) {
    return new RajaOngkirService(
        $app->make(Client::class),
        config('services.rajaongkir.key')
    );
});

// Singleton — instance yang sama setiap kali
$this->app->singleton(CartService::class, function ($app) {
    return new CartService(
        $app->make(CustomerService::class)
    );
});
```

### 2.3 Resolusi — Mengambil dari Container

```php
// Cara 1: app() helper
$service = app(RajaOngkirService::class);

// Cara 2: resolve() helper
$service = resolve(RajaOngkirService::class);

// Cara 3: Type-hint di constructor (auto-resolve)
class OrderService
{
    public function __construct(
        private CartService $cartService,  // Auto-resolve!
    ) {}
}

// Cara 4: Type-hint di controller method
class CheckoutController extends Controller
{
    public function process(
        CheckoutRequest $request,        // Auto-resolve!
        OrderService $orderService       // Auto-resolve!
    ) {
        // ...
    }
}
```

### 2.4 Container di Codebase

```php
// Di controller — parameter auto-resolve:
class RajaOngkirController extends Controller
{
    public function getCities(
        RajaOngkirService $rajaOngkir  // Auto-resolve dari container!
    ) {
        return response()->json(
            $rajaOngkir->getDummyCities()
        );
    }
}
```

---

## 3. Service Provider — Otak dari Bootstrap

### 3.1 Apa Itu Service Provider?

Service Provider adalah class yang **mendaftarkan sesuatu ke container** saat Laravel bootstrap. Setiap package Laravel punya provider.

```php
// config/app.php — semua provider didaftarkan di sini
'providers' => [
    // Framework providers
    Illuminate\Auth\AuthServiceProvider::class,
    Illuminate\Broadcasting\BroadcastServiceProvider::class,
    Illuminate\Bus\BusServiceProvider::class,
    // ...
    App\Providers\AppServiceProvider::class,
    App\Providers\RouteServiceProvider::class,
],
```

### 3.2 Dua Method Penting

```php
class AppServiceProvider extends ServiceProvider
{
    // register() → binding ke container (sebelum boot)
    public function register(): void
    {
        $this->app->bind(RajaOngkirService::class, function ($app) {
            return new RajaOngkirService(
                $app->make(Client::class),
                config('services.rajaongkir.key')
            );
        });
    }

    // boot() → setelah semua provider register (event, route, view)
    public function boot(): void
    {
        // View composer
        view()->composer('layouts.app', function ($view) {
            $view->with('cartCount', session('cart_count', 0));
        });

        // Macro
        Str::macro('phone', fn($value) => '+62' . substr($value, 1));
    }
}
```

### 3.3 Urutan Eksekusi

```
1. Register semua provider (konfigurasi)
   ├── register() dipanggil UNTUK SEMUA provider
   └── Baru setelah semua register selesai...

2. Boot semua provider (eksekusi)
   └── boot() dipanggil UNTUK SEMUA provider

Kenapa register dulu semua?
→ Provider A mungkin butuh binding dari Provider B
→ Jadi semua register dulu, baru boot
```

### 3.4 Provider di Codebase

```php
// app/Providers/
├── AppServiceProvider.php    → Binding service, view composer, macro
├── RouteServiceProvider.php  → Route files (web.php, api.php)
└── AuthServiceProvider.php   → Policy registration
```

---

## 4. Facades — Static Proxy

### 4.1 Apa Itu Facade?

Facade memberikan antarmuka **static** ke class di container.

```php
// Tanpa Facade — dari container
$user = app('db')->table('users')->get();

// Dengan Facade — static
$user = DB::table('users')->get();
// DB:: → sebenarnya调用 Illuminate\Database\DatabaseManager di container
```

### 4.2 Cara Kerja Facade

```php
// Contoh: Cache::get('key')
// 1. Panggil static method Cache::get()
// 2. Cache facade extends Facade
// 3. Facade::__callStatic → getFacadeRoot()
// 4. getFacadeAccessor() → return 'cache' (string)
// 5. Container → resolve 'cache' → Illuminate\Cache\CacheManager
// 6. Panggil ->get('key') di CacheManager
```

### 4.3 Facades Umum

| Facade | Class Asli | Helper |
|--------|-----------|--------|
| `Route` | Router | `route()` |
| `DB` | DatabaseManager | — |
| `Cache` | CacheManager | `cache()` |
| `Config` | Repository | `config()` |
| `Storage` | FilesystemManager | `storage_path()` |
| `Mail` | Mailer | `mail()` |
| `Log` | LogManager | `logger()` |
| `Auth` | AuthManager | `auth()` |
| `Hash` | Hasher | `bcrypt()` |
| `Validator` | ValidationFactory | `validator()` |

### 4.4 Real-Time Facade

```php
// Mengubah class jadi facade tanpa bikin facade class
use Facades\App\Services\CartService;

CartService::getTotal(); // Sama dengan app(CartService::class)->getTotal()
```

### 4.5 Facade vs Helper

```php
// Facade: Cache::get('key')
// Helper: cache('key')

// Facade: Config::get('app.name')
// Helper: config('app.name')

// Facade: DB::table('users')
// Helper: (tidak ada)

// Helper lebih pendek, Facade lebih eksplisit
// Di codebase: helper lebih sering dipakai
```

---

## 5. Artisan — Command Line Laravel

### 5.1 Artisan di Codebase

Artisan adalah CLI Laravel:

```bash
# Lihat semua command
php artisan list

# Command sering dipakai:
php artisan make:controller    # Buat controller
php artisan make:model         # Buat model
php artisan make:migration     # Buat migration
php artisan make:request       # Buat FormRequest
php artisan make:mail          # Buat mail class
php artisan make:job           # Buat job
php artisan make:middleware    # Buat middleware

# Database
php artisan migrate            # Jalankan migration
php artisan migrate:fresh      # Hapus semua table + migrate ulang
php artisan db:seed            # Jalankan seeder
php artisan migrate:fresh --seed  # Reset + seed

# Cache
php artisan cache:clear        # Hapus cache
php artisan config:cache       # Cache konfigurasi
php artisan route:cache        # Cache routing
php artisan view:cache         # Cache Blade

# Development
php artisan serve              # Built-in server
php artisan tinker             # Interactive shell
php artisan make:cast          # Buat custom cast
php artisan storage:link       # Buat symbolic link

# Queue
php artisan queue:work         # Jalankan queue worker
php artisan queue:table        # Buat migration queue table
```

### 5.2 Artisan di Codebase Ini

```bash
# Ini yang sudah dijalankan:
php artisan migrate                  # 25 migrations (schema database)
php artisan db:seed                  # Seeder (data awal)
php artisan storage:link             # public/storage → storage/app/public
php artisan key:generate             # APP_KEY
php artisan optimize                 # Cache config, route, view

# Ini tersedia:
php artisan telescope:install        # Telescope (sudah)
php artisan queue:table              # Queue table (sudah)
php artisan schedule:run             # Cron (kalau ada)
```

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ REQUEST LIFECYCLE                                                  │
│  public/index.php                                                  │
│    → bootstrap/app.php (Application)                              │
│    → Kernel::handle()                                              │
│       → Register Service Providers (config/app.php)               │
│       → Boot Service Providers                                     │
│       → Global Middleware (Kernel.php $middleware)                 │
│       → Route Match (RouteServiceProvider)                        │
│       → Route Middleware (web, auth, role, throttle)               │
│       → Controller (Dependency Injection)                         │
│       → Response                                                   │
├─────────────────────────────────────────────────────────────────────┤
│ SERVICE CONTAINER                                                  │
│  app()->bind()  → daftarkan class                                 │
│  app()->make()  → ambil instance (auto-resolve dependency)        │
│  app()->singleton() → instance yang sama setiap kali              │
├─────────────────────────────────────────────────────────────────────┤
│ SERVICE PROVIDER                                                   │
│  register() → binding (sebelum provider lain boot)                │
│  boot()     → setelah semua register (event, view, route)         │
├─────────────────────────────────────────────────────────────────────┤
│ FACADES                                                            │
│  Cache::get()  → app('cache')->get()                              │
│  DB::select()  → app('db')->select()                              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Trace index.php.** Buka `public/index.php`. Identifikasi setiap baris:
   - Autoload (baris berapa?)
   - Bootstrap app
   - Kernel handle
   - Response send
   - Terminate

2. **Cek Service Provider.** Buka `config/app.php`. Hitung provider jumlah. Buka `app/Providers/AppServiceProvider.php` — apa yang di-register?

3. **Test Facade.** Buka `tinker`:
   ```bash
   php artisan tinker
   ```
   Ketik:
   ```php
   Config::get('app.name');
   config('app.name');
   // Sama? Beda?
   ```

4. **Container resolve.** Di `tinker`:
   ```php
   app()->make(App\Services\CartService::class);
   // Berhasil? Lihat constructor CartService
   ```

5. **Artisan list.** Jalankan `php artisan list | findstr make:` — lihat semua command yang bisa membuat file.

---

## 🔗 Referensi

- [Laravel Docs: Request Lifecycle](https://laravel.com/docs/11.x/lifecycle)
- [Laravel Docs: Service Container](https://laravel.com/docs/11.x/container)
- [Laravel Docs: Service Providers](https://laravel.com/docs/11.x/providers)
- [Laravel Docs: Facades](https://laravel.com/docs/11.x/facades)
- [Laravel Docs: Artisan Console](https://laravel.com/docs/11.x/artisan)
- Codebase: `public/index.php` — entry point
- Codebase: `bootstrap/app.php` — app bootstrap
- Codebase: `app/Providers/AppServiceProvider.php`
- Codebase: `config/app.php` — providers list
