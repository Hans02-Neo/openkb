# 06-07: Middleware dan Auth — Penjaga Pintu

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-06-blade-template-engine  
> **Waktu baca**: 70-85 menit  
> **Kata kunci**: middleware, auth, authentication, authorization, gate, policy, role, throttle

---

## 📋 Ringkasan

**Middleware** adalah filter yang memproses request sebelum mencapai controller. **Auth** adalah sistem autentikasi dan otorisasi. Keduanya bersama memastikan hanya user yang tepat yang bisa mengakses resource tertentu.

**Target pemahaman:**
- Kamu paham cara kerja middleware
- Kamu bisa membuat middleware kustom
- Kamu paham sistem auth Laravel
- Kamu paham Gate dan Policy

---

## 1. Middleware — Filter Sebelum Controller

### 1.1 Konsep

Middleware adalah **lapisan** yang request harus lewati sebelum mencapai controller.

```
Request → Middleware1 → Middleware2 → Controller → Response
              │             │
         (Cek auth)   (Cek CSRF)
```

### 1.2 Middleware di Codebase

```php
// app/Http/Kernel.php
class Kernel extends HttpKernel
{
    // Global middleware — SEMUA request lewat sini
    protected $middleware = [
        \App\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ];

    // Middleware groups — apply ke grup route
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

    // Route middleware — apply ke route spesifik
    protected $middlewareAliases = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'role' => \App\Http\Middleware\CheckRole::class,
    ];
}
```

### 1.3 Cara Kerja Middleware

```php
<?php
// Contoh: middleware auth

class Authenticate
{
    public function handle(Request $request, Closure $next): Response
    {
        if (!auth()->check()) {
            // User belum login → redirect ke login
            return redirect()->route('login');
        }

        // User sudah login → lanjut ke controller
        return $next($request);
    }
}
```

### 1.4 Middleware di Route

```php
// Apply di route:
Route::get('/orders', [OrderController::class, 'index'])
    ->middleware('auth'); // cuma user login

Route::get('/orders/{order}', [OrderController::class, 'show'])
    ->middleware(['auth', 'verified']); // login + email verified

// Apply ke grup route:
Route::middleware(['auth'])->group(function () {
    Route::get('/orders', [OrderController::class, 'index']);
    Route::get('/cart', [CartController::class, 'index']);
});

// Apply di controller constructor:
class OrderController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('verified')->only('index', 'show');
        $this->middleware('role:admin')->only('destroy');
    }
}
```

### 1.5 Middleware Parameter

```php
// CheckRole middleware — menerima parameter
class CheckRole
{
    public function handle(Request $request, Closure $next, string ...$roles): Response
    {
        if (!auth()->check() || !in_array(auth()->user()->role, $roles)) {
            abort(403);
        }
        return $next($request);
    }
}

// Di route:
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware('role:admin,superadmin');
```

---

## 2. Middleware Kustom

### 2.1 Membuat Middleware

```bash
php artisan make:middleware CheckAge
```

```php
<?php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAge
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && $request->user()->age < 17) {
            return redirect()->route('home')
                ->with('error', 'Minimal 17 tahun');
        }

        return $next($request);
    }
}
```

### 2.2 Daftarkan di Kernel

```php
// app/Http/Kernel.php
protected $middlewareAliases = [
    // ...
    'age' => \App\Http\Middleware\CheckAge::class,
];
```

### 2.3 Middleware dengan Response

```php
class EnsureTokenIsValid
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== config('app.api_token')) {
            return response()->json(['error' => 'Invalid token'], 401);
        }

        return $next($request);
    }
}
```

### 2.4 Middleware Before vs After

```php
// Before middleware — diproses SEBELUM controller
class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Lakukan sesuatu sebelum request ke controller
        Log::info('Request: ' . $request->path());

        return $next($request);
    }
}

// After middleware — diproses SETELAH controller
class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Lakukan sesuatu setelah response dari controller
        Log::info('Response status: ' . $response->status());

        return $response;
    }
}
```

### 2.5 Terminate Middleware

```php
// Middleware yang jalan SETELAH response dikirim ke browser
// Cocok untuk logging, cleanup, jobs

class RequestLogger
{
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    public function terminate(Request $request, Response $response): void
    {
        // Response sudah dikirim — cocok untuk heavy operation
        Log::info('Request completed', [
            'url' => $request->fullUrl(),
            'status' => $response->status(),
            'memory' => memory_get_peak_usage(),
        ]);
    }
}
```

---

## 3. Authentication — Siapa Kamu?

### 3.1 Sistem Auth Laravel

```
Login form → POST /login → Authenticate user → Session created
                │                                    │
                ▼                                    ▼
          Credentials valid?                   Auth::check() = true
          Yes → redirect                       Auth::user() = User
          No  → back with error
```

### 3.2 Auth di Codebase

```php
// Auth route — Laravel Breeze
Route::get('/login', [AuthenticatedSessionController::class, 'create'])
    ->middleware('guest')
    ->name('login');

Route::post('/login', [AuthenticatedSessionController::class, 'store'])
    ->middleware('guest');

Route::post('/logout', [AuthenticatedSessionController::class, 'destroy'])
    ->middleware('auth')
    ->name('logout');
```

### 3.3 Auth Helper

```php
// Cek login
auth()->check();       // true/false
auth()->guest();       // true jika belum login
Auth::check();

// Ambil user
auth()->user();        // User model or null
auth()->id();          // user_id or null
Auth::user();

// Login manual
auth()->login($user);
auth()->loginUsingId(1);

// Logout
auth()->logout();

// Attempt (login with credentials)
if (auth()->attempt(['email' => $email, 'password' => $password])) {
    // login berhasil
}
```

### 3.4 Blade Auth Directive

```blade
@auth
    <p>Selamat datang, {{ auth()->user()->name }}!</p>
@endauth

@guest
    <a href="{{ route('login') }}">Login</a>
@else
    <a href="{{ route('orders.index') }}">Pesanan Saya</a>
@endguest
```

---

## 4. Authorization — Kamu Boleh Apa?

### 4.1 Gate — Closure-based

```php
// app/Providers/AuthServiceProvider.php
public function boot(): void
{
    // Gate untuk update produk
    Gate::define('update-product', function (User $user, Product $product) {
        return $user->isAdmin() || $user->id === $product->user_id;
    });

    // Gate tanpa model
    Gate::define('view-orders', function (User $user) {
        return $user->role === 'admin' || $user->role === 'customer';
    });
}
```

```php
// Menggunakan Gate
if (Gate::allows('update-product', $product)) { ... }
if (Gate::denies('update-product', $product)) { ... }

// Atau di controller:
$this->authorize('update-product', $product);

// Atau di Blade:
@can('update-product', $product)
    <button>Edit</button>
@endcan

@cannot('update-product', $product)
    <p>Kamu tidak bisa edit produk ini.</p>
@endcannot
```

### 4.2 Policy — Class-based

```php
// Buat policy
php artisan make:policy OrderPolicy --model=Order
```

```php
<?php
// app/Policies/OrderPolicy.php
namespace App\Policies;

use App\Models\Order;
use App\Models\User;

class OrderPolicy
{
    // User bisa lihat order?
    public function view(User $user, Order $order): bool
    {
        return $user->id === $order->user_id || $user->isAdmin();
    }

    // User bisa update order?
    public function update(User $user, Order $order): bool
    {
        return $user->isAdmin();
    }

    // User bisa delete order?
    public function delete(User $user, Order $order): bool
    {
        return $user->isAdmin();
    }
}
```

```php
// Daftarkan policy
// app/Providers/AuthServiceProvider.php
protected $policies = [
    Order::class => OrderPolicy::class,
];
```

```php
// Gunakan policy di controller
public function show(Order $order)
{
    $this->authorize('view', $order); // panggil OrderPolicy@view
    return view('orders.show', compact('order'));
}

// Atau via User model
if ($user->can('view', $order)) { ... }
if ($user->cannot('update', $order)) { ... }
```

### 4.3 Policy vs Gate

| | Gate | Policy |
|---|------|--------|
| **Scope** | Global (action-based) | Per model |
| **Define** | Closure di Provider | Class terpisah |
| **Model** | Bisa tanpa model | Biasanya dengan model |
| **Contoh** | `view-orders` | `OrderPolicy@view` |
| **Gunakan** | Aksi umum | CRUD per resource |

---

## 5. Role-Based Access Control

### 5.1 Role di Codebase

```php
// User model atau field
// asumsi: User punya field/relationship 'role' atau 'roles'

// Middleware CheckRole — cek role
class CheckRole
{
    public function handle(Request $request, Closure $next, string ...$roles): Response
    {
        if (!$request->user() || !$request->user()->hasAnyRole($roles)) {
            abort(403);
        }
        return $next($request);
    }
}
```

```php
// Route admin
Route::middleware(['auth', 'role:admin'])->prefix('admin')->group(function () {
    Route::resource('products', Admin\ProductController::class);
    Route::resource('categories', Admin\CategoryController::class);
    Route::resource('orders', Admin\OrderController::class);
});
```

### 5.2 Throttle — Rate Limiting

```php
// Batasi request — misal 60 request per menit
Route::get('/api/products', [ProductController::class, 'index'])
    ->middleware('throttle:60,1'); // 60 request, 1 menit

// Dynamic throttle (berdasarkan user)
Route::middleware('throttle:api')->group(function () {
    // throttle:api didefinisikan di Kernel
    // 'api' => '60,1'
});
```

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ MIDDLEWARE                                                         │
│  Global:  Semua request (TrimStrings, ConvertEmptyStrings, dll)    │
│  Group:   web (session, CSRF), api (throttle)                      │
│  Route:   auth, guest, verified, role, throttle                    │
├─────────────────────────────────────────────────────────────────────┤
│ AUTHENTICATION (siapa kamu?)                                       │
│  auth()->check()   → login?                                       │
│  auth()->user()    → User model                                   │
│  auth()->id()      → user ID                                      │
│  @auth / @guest    → Blade directives                             │
├─────────────────────────────────────────────────────────────────────┤
│ AUTHORIZATION (kamu boleh apa?)                                    │
│  Gate:  Gate::define('action', fn($user, $model) => bool)        │
│  Policy: Class per model (view, create, update, delete)           │
│  Blade: @can / @cannot                                            │
│  Route: middleware('role:admin')                                   │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  Middleware: auth, guest, verified, role, throttle                 │
│  Route: orders → middleware auth                                  │
│  Admin: routes → middleware role:admin                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Cek Kernel.** Buka `app/Http/Kernel.php`. Identifikasi:
   - Global middleware
   - Web middleware group
   - Route middleware aliases

2. **Cek middleware di route.** Buka `routes/web.php`. Route mana yang pakai middleware?
   - Route mana yang pakai `auth`?
   - Route mana yang pakai `guest`?
   - Route mana yang pakai `verified`?

3. **Cek CheckRole.** Buka `app/Http/Middleware/CheckRole.php`. Bagaimana cara kerjanya?

4. **Test auth.** Buka `olshop-koneksi.test/orders` tanpa login. Apa yang terjadi? Ke mana di-redirect?

5. **Buat middleware.** Buat middleware `LogRequest` yang mencatat setiap request ke log file. Daftarkan di Kernel. Apply ke route `/shop`.

---

## 🔗 Referensi

- [Laravel Docs: Middleware](https://laravel.com/docs/11.x/middleware)
- [Laravel Docs: Authentication](https://laravel.com/docs/11.x/authentication)
- [Laravel Docs: Authorization](https://laravel.com/docs/11.x/authorization)
- [Laravel Docs: Gates](https://laravel.com/docs/11.x/authorization#gates)
- [Laravel Docs: Policies](https://laravel.com/docs/11.x/authorization#policies)
- Codebase: `app/Http/Kernel.php`
- Codebase: `app/Http/Middleware/CheckRole.php`
- Codebase: `app/Providers/AuthServiceProvider.php`
