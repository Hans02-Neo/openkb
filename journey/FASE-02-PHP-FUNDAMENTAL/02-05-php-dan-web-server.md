# 02-05: PHP dan Web Server

> **Fase**: 2 — PHP Fundamental  
> **Prasyarat**: 02-04-fungsi-dan-scope.md  
> **Waktu baca**: 80-100 menit  
> **Kata kunci**: HTTP, web server, Apache, PHP-FPM, superglobals, $_GET, $_POST, $_SERVER, $_SESSION, Request, Response, middleware, CSRF, routing, session

---

## 📋 Ringkasan

Dokumen sebelumnya membahas fungsi dan scope di dalam PHP. Sekarang kita naik satu level: **bagaimana PHP berkomunikasi dengan web server** untuk menghasilkan halaman web yang kamu lihat di browser.

Ini adalah jembatan antara "PHP sebagai bahasa" dan "PHP sebagai aplikasi web". Setelah ini kamu akan paham:

- Apa yang terjadi dari URL diketik sampai halaman tampil
- Bagaimana data dari browser sampai ke kode PHP
- Bagaimana session dan cookies bekerja
- Apa itu middleware dan bagaimana melindungi route
- Bagaimana request di-handle di codebase ini

---

## 1. Arsitektur Web Server + PHP

### 1.1 SAPI — Server API

PHP bisa berjalan di berbagai mode, tergantung **SAPI** (Server API):

| SAPI | Cara Kerja | Digunakan di |
|------|------------|--------------|
| **mod_php** | PHP adalah modul Apache (`.so`/`.dll`) | **Laragon**, XAMPP, MAMP |
| **PHP-FPM** | PHP proses terpisah, Nginx komunikasi via FastCGI | Produksi, server bersama |
| **CLI** | Command line, tanpa web server | `php artisan`, `php script.php` |
| **Built-in** | Server web mini untuk development | `php -S localhost:8000` |

**Laragon menggunakan mod_php:** Apache memuat PHP sebagai modul internal. Tidak ada proses PHP terpisah — semuanya di dalam proses Apache.

### 1.2 Flow Request-Response

```
Browser                          Apache + mod_php              Laravel App
   │                                  │                            │
   │ 1. GET /shop?category=laptop     │                            │
   │ ─────────────────────────────►   │                            │
   │                                  │ 2. Apache terima request    │
   │                                  │    Parse .htaccess (jika ada)│
   │                                  │ 3. Route ke index.php       │
   │                                  │ ───────────────────────►   │
   │                                  │                            │
   │                                  │ 4. Laravel bootstrap:      │
   │                                  │    - Load autoload         │
   │                                  │    - Load .env             │
   │                                  │    - Init service container│
   │                                  │    - Parse route           │
   │                                  │    - Run middleware         │
   │                                  │    - Call controller        │
   │                                  │    - Generate response      │
   │                                  │ ◄───────────────────────│
   │                                  │ 5. Apache kirim response    │
   │ ◄─────────────────────────────   │                            │
   │                                  │                            │
   │ 6. Browser render HTML          │                            │
```

### 1.3 PHP Built-in Server (untuk Development)

Laragon punya Apache, tapi Laravel juga bisa jalan dengan `php artisan serve`:

```bash
php artisan serve
# Server running on http://127.0.0.1:8000
```

Ini menggunakan PHP **built-in web server** (SAPI CLI dengan server mode). Cocok untuk testing cepat, tapi **jangan untuk produksi** — single-threaded, tidak support HTTPS.

---

## 2. HTTP Protocol — Bahasa Komunikasi Web

### 2.1 HTTP Request

Setiap request dari browser ke server punya struktur:

```
METHOD /path?query=string HTTP/1.1
Host: olshop-koneksi.test
User-Agent: Mozilla/5.0 ...
Content-Type: application/x-www-form-urlencoded
Cookie: XSRF-TOKEN=abc123; session=xyz

body=content&foo=bar
```

**Komponen utama:**
- **Method:** GET, POST, PATCH, DELETE, PUT
- **Path:** `/shop`, `/cart/add`, `/checkout`
- **Query string:** `?category=laptop&sort=price_asc` (hanya GET)
- **Headers:** metadata (Content-Type, User-Agent, Cookie, Authorization)
- **Body:** data dikirim (POST, PATCH, PUT)

### 2.2 HTTP Response

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Set-Cookie: session=xyz123; path=/; httponly

<!DOCTYPE html>
<html>
<head><title>...</title></head>
<body>...</body>
</html>
```

**Komponen utama:**
- **Status code:** 200 (OK), 302 (Redirect), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 500 (Server Error)
- **Headers:** Content-Type, Set-Cookie, Cache-Control
- **Body:** HTML, JSON, atau konten lain

### 2.3 Status Code Paling Penting

| Code | Nama | Arti | Contoh di Codebase |
|------|------|------|-------------------|
| 200 | OK | Sukses | `return view(...)` |
| 302 | Redirect | Pindah halaman | `redirect()->route(...)` |
| 401 | Unauthorized | Belum login | `abort(401)` di RoleMiddleware |
| 403 | Forbidden | Tidak punya akses | `abort(403)` di RoleMiddleware |
| 404 | Not Found | Halaman tidak ada | `findOrFail()` |
| 419 | Session Expired | CSRF token timeout | Auto dari Laravel |
| 422 | Validation Error | Validasi gagal | `$request->validate()` |
| 500 | Server Error | Error internal | Exception tanpa handler |

---

## 3. PHP Superglobals

### 3.1 Apa Itu Superglobal?

PHP punya **variabel global bawaan** yang otomatis tersedia di **semua scope** — fungsi, method, closure. Mereka adalah jembatan antara web server dan kode PHP.

```
$_GET     → query string dari URL
$_POST    → body dari form POST
$_SERVER  → informasi server dan request
$_SESSION → data session
$_COOKIE  → cookie dari browser
$_FILES   → file upload
$_REQUEST → gabungan GET + POST + COOKIE
```

### 3.2 `$_GET` — Query String

Ketika browser request `http://olshop-koneksi.test/shop?category=laptop&sort=price_asc`:

```php
// PHP mengurai query string secara otomatis
echo $_GET['category']; // "laptop"
echo $_GET['sort'];     // "price_asc"
```

**Di codebase:** Tidak ada penggunaan `$_GET` langsung. Semua melalui `$request->query()` atau `$request->get()`.

### 3.3 `$_POST` — Form Data

Ketika form dikirim dengan method POST:

```php
// Form: <input name="email" value="budi@email.com">
echo $_POST['email']; // "budi@email.com"
```

**Di codebase:** Tidak ada penggunaan `$_POST` langsung.

### 3.4 `$_SERVER` — Metadata Request

`$_SERVER` berisi informasi dari web server:

```php
$_SERVER['REQUEST_METHOD']    // "GET", "POST"
$_SERVER['REQUEST_URI']       // "/shop?category=laptop"
$_SERVER['HTTP_HOST']         // "olshop-koneksi.test"
$_SERVER['HTTP_USER_AGENT']   // "Mozilla/5.0 ..."
$_SERVER['REMOTE_ADDR']       // IP client
$_SERVER['SERVER_NAME']       // "olshop-koneksi.test"
$_SERVER['QUERY_STRING']      // "category=laptop"
```

### 3.5 `$_SESSION` — Session Data

Session menyimpan data yang **tetap ada antar request**, unik per user.

```php
session_start(); // mulai session (otomatis di Laravel)
$_SESSION['user_id'] = 5;
echo $_SESSION['user_id']; // 5
```

**Di codebase:** Tidak ada penggunaan `$_SESSION` langsung. Laravel menyediakan facade `Session::get/put` dan `$request->session()`.

### 3.6 `$_FILES` — File Upload

```php
// Form: <input type="file" name="avatar">
$_FILES['avatar']['name']     // "foto.jpg"
$_FILES['avatar']['tmp_name'] // "/tmp/phpABC123"
$_FILES['avatar']['size']     // 102400
$_FILES['avatar']['error']    // 0 (UPLOAD_ERR_OK)
```

**Di codebase:** Lewat `$request->file('images')`.

### 3.7 Kenapa Laravel Tidak Pakai Superglobals Langsung?

**Di seluruh codebase ini, tidak ada satupun penggunaan `$_GET`, `$_POST`, `$_SESSION`, `$_COOKIE`, atau `$_FILES`.** Semua melalui fasilitas Laravel.

**Alasannya:**

1. **Testability** — Superglobal tidak bisa di-mock di unit test. `$request` adalah objek yang bisa diganti dengan mock.
2. **Security** — `$_GET['key'] ?? null` vs `$request->input('key')` — Laravel otomatis trim dan sanitize.
3. **Consistency** — Request data bisa datang dari query string, form, atau JSON body. Laravel menyatukan semuanya.
4. **Convenience** — `$request->validate()`, `$request->user()`, `$request->session()` — semua dalam satu objek.

---

## 4. Laravel Request — Abstraksi HTTP

### 4.1 Dari `$_GET` ke `$request->query()`

**Di ShopController.php:**

```php
public function index(Request $request)
{
    // Daripada: $_GET['category'] ?? null
    if ($request->filled('category')) {
        // ...
    }

    // Daripada: $_GET['sort'] ?? 'latest'
    $sort = $request->get('sort', 'latest');
    
    // Daripada: $_GET (semua query string)
    $products = $query->paginate(12)->appends($request->query());
}
```

**Perbandingan method Request:**

| Method | Setara Superglobal | Kegunaan |
|--------|-------------------|----------|
| `$request->all()` | — | Semua input (query + body + JSON) |
| `$request->input('key')` | `$_POST['key'] ?? $_GET['key']` | Input dari mana pun |
| `$request->query('key')` | `$_GET['key']` | Hanya query string |
| `$request->post('key')` | `$_POST['key']` | Hanya POST body |
| `$request->filled('key')` | `isset($_GET['key']) && $_GET['key'] !== ''` | Ada dan tidak kosong |
| `$request->has('key')` | `isset($_GET['key'])` | Ada (meski string kosong) |
| `$request->get('key', default)` | `$_GET['key'] ?? default` | Dengan default |
| `$request->only(['a', 'b'])` | — | Ambil subset |
| `$request->except(['a'])` | — | Ambil semua kecuali |

### 4.2 Request Validation

**Di CheckoutController.php:83-94:**

```php
$request->validate([
    'address_id' => 'required|exists:addresses,id',
    'shipping_courier' => 'required|string',
    'shipping_service' => 'required|string',
    'shipping_cost' => 'required|numeric|min:0',
    'notes' => 'nullable|string|max:500',
], [
    'address_id.required' => 'Silakan pilih alamat pengiriman.',
    // custom messages...
]);
```

**Apa yang terjadi saat validasi gagal:**
1. Laravel mengumpulkan semua error
2. Melempar `ValidationException`
3. Exception di-catch oleh framework
4. Redirect back ke halaman sebelumnya
5. Error disimpan di flash session → ditampilkan di Blade via `@error`

**Validasi menggunakan FormRequest class:**

Di `LoginRequest.php`:

```php
public function rules(): array
{
    return [
        'email' => ['required', 'string', 'email'],
        'password' => ['required', 'string'],
    ];
}
```

Dengan `LoginRequest`, controller jadi lebih bersih:

```php
public function store(LoginRequest $request): RedirectResponse
{
    $request->authenticate();     // validasi sudah otomatis
    $request->session()->regenerate();
    return redirect()->intended(route('dashboard', absolute: false));
}
```

### 4.3 File Upload via Request

**Di Admin/ProductController.php:41-66:**

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'images.*' => 'nullable|image|mimes:jpeg,png,jpg,webp|max:2048',
        // ...
    ]);

    $images = $request->file('images') ?? [];
    $this->productService->createProduct($validated, $images);
}
```

**Proses upload file:**
1. Browser kirim file (form dengan `enctype="multipart/form-data"`)
2. PHP terima file, simpan di `php://tmp` (sementara)
3. Laravel bungkus jadi `UploadedFile` instance
4. `$request->file('images')` mengembalikan array `UploadedFile`
5. `addMedia($image)->toMediaCollection('products')` — pindahkan ke storage permanen

### 4.4 CSRF Token via Request

CSRF (Cross-Site Request Forgery) mencegah serangan di mana situs jahat mengirim request atas nama user yang sudah login.

**Di Blade:**
```blade
<form action="{{ route('checkout.process') }}" method="POST">
    @csrf
    <!-- ... -->
</form>
```

`@csrf` menghasilkan:
```html
<input type="hidden" name="_token" value="abc123...">
```

Laravel secara otomatis memverifikasi token ini di setiap POST/PATCH/DELETE request. Jika token tidak cocok → 419 Session Expired.

**Pengecualian untuk webhook:**

Di `bootstrap/app.php:18-20`:
```php
$middleware->validateCsrfTokens(except: [
    'midtrans/webhook',  // Midtrans tidak tahu CSRF token kita
]);
```

Webhook dari Midtrans tidak bisa membawa CSRF token, jadi endpoint webhook dikecualikan.

---

## 5. Session Management

### 5.1 Bagaimana Session Bekerja

```
Browser                              Server
  │                                    │
  │ 1. GET / (first visit, no cookie)  │
  │ ─────────────────────────────►    │
  │                                    │ 2. Generate session ID: abc123
  │                                    │ 3. Simpan data session di storage
  │ 4. Set-Cookie: session=abc123      │
  │ ◄─────────────────────────────    │
  │                                    │
  │ 5. GET /shop (dengan cookie)       │
  │ Cookie: session=abc123            │
  │ ─────────────────────────────►    │
  │                                    │ 6. Baca session ID dari cookie
  │                                    │ 7. Load data session dari storage
  │                                    │ 8. $cart tersedia lagi!
```

### 5.2 Session di CartService

**app/Services/CartService.php — contoh penggunaan session paling jelas:**

```php
class CartService
{
    protected $sessionKey = 'shopping_cart';

    public function getCart(): array
    {
        // Session::get(key, default) — baca dari session
        return Session::get($this->sessionKey, []);
    }

    public function add($productId, $quantity = 1): array
    {
        $cart = $this->getCart();
        // ... modify $cart ...
        Session::put($this->sessionKey, $cart); // simpan kembali
        return $cart;
    }

    public function clear(): void
    {
        Session::forget($this->sessionKey); // hapus dari session
    }
}
```

**Kenapa cart disimpan di session, bukan database?**
- Pengguna **belum login** bisa menambahkan ke cart
- Cart bersifat **sementara** — tidak perlu disimpan permanen
- Performa — session (file/redis) lebih cepat daripada query database

### 5.3 Session Invalidation — Login/Logout

**Di AuthenticatedSessionController.php:37-46:**

```php
public function destroy(Request $request): RedirectResponse
{
    Auth::guard('web')->logout();

    $request->session()->invalidate();     // ← hapus semua data session
    $request->session()->regenerateToken(); // ← buat CSRF token baru

    return redirect('/');
}
```

**Kenapa session di-invalidate saat logout?** Mencegah **session fixation attack** — orang lain memakai session ID lama.

### 5.4 Session Drivers

| Driver | Cara Kerja | Cocok Untuk |
|--------|-----------|-------------|
| `file` | Simpan di `storage/framework/sessions/` | **Development** (Laragon default) |
| `database` | Simpan di tabel sessions | Produksi skala kecil |
| `redis` | Simpan di Redis (in-memory) | **Produksi** — sangat cepat |
| `cookie` | Simpan di cookie (terenkripsi) | Aplikasi stateless |
| `array` | Simpan di array (hilang tiap request) | **Testing** |

**Di `.env` Laragon:**
```
SESSION_DRIVER=file
```

Session file ada di `storage/framework/sessions/` — setiap session adalah satu file dengan nama = session ID.

---

## 6. Response Types

### 6.1 View Response

**Yang paling sering dipakai — mengembalikan halaman HTML:**

```php
return view('shop.checkout', compact('cart', 'total', 'totalWeight', 'addresses', 'defaultAddress'));
```

Ini setara dengan:
```php
return response()->view('shop.checkout', $data);
```

Laravel akan:
1. Cari file `resources/views/shop/checkout.blade.php`
2. Render Blade → HTML
3. Set header `Content-Type: text/html`
4. Return status 200

### 6.2 Redirect Response

```php
// Redirect ke named route
return redirect()->route('payments.confirmation', $order->id)
    ->with('success', 'Pesanan berhasil dibuat!');

// Redirect back (ke halaman sebelumnya)
return redirect()->back()->with('error', 'Terjadi kesalahan: ' . $e->getMessage());

// Redirect intended (setelah login)
return redirect()->intended(route('dashboard', absolute: false));
```

`->with('key', 'value')` menyimpan **flash data** ke session — data yang hanya bertahan untuk satu request berikutnya (untuk menampilkan success/error message).

### 6.3 JSON Response

```php
// Sukses
return response()->json($costs);

// Error dengan status code
return response()->json(['error' => $e->getMessage()], 500);

// Webhook response
return response()->json(['message' => 'Webhook handled successfully']);
```

Laravel otomatis:
1. Set header `Content-Type: application/json`
2. Encoding array ke JSON

### 6.4 Status Code

```php
abort(401);       // Unauthorized
abort(403);       // Forbidden
abort(404);       // Not Found
abort(500);       // Server Error
```

---

## 7. Middleware — Filter Sebelum Controller

### 7.1 Konsep Middleware

Middleware adalah **lapisan** yang membungkus request sebelum sampai ke controller.

```
Request → Middleware 1 → Middleware 2 → Controller → Response
                                          ↓
                             Middleware  boleh memb lok akses
                             (401/403)    sebelum controller
```

### 7.2 Middleware Bawaan Laravel

**auth** — hanya user yang sudah login bisa akses:
```php
Route::middleware('auth')->group(function () {
    Route::get('/checkout', ...);
});
```

**guest** — hanya user yang **belum** login bisa akses:
```php
Route::middleware('guest')->group(function () {
    Route::get('login', [AuthenticatedSessionController::class, 'create']);
});
```

**verified** — email harus sudah diverifikasi:
```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['auth', 'verified']);
```

**throttle** — batasi jumlah request:
```php
Route::get('verify-email/{id}/{hash}', ...)
    ->middleware(['signed', 'throttle:6,1']);
// Maksimal 6 kali dalam 1 menit
```

**signed** — URL harus ditandatangani (untuk link verifikasi email):
```php
Route::get('verify-email/{id}/{hash}', ...)
    ->middleware('signed');
```

### 7.3 Custom Middleware: RoleMiddleware

**app/Http/Middleware/RoleMiddleware.php:**

```php
public function handle(Request $request, Closure $next, ...$roles): Response
{
    if (!$request->user()) {
        abort(401); // belum login
    }

    foreach ($roles as $role) {
        if ($request->user()->roles()->where('slug', $role)->exists()) {
            return $next($request); // punya role → lanjut
        }
    }

    abort(403, 'Unauthorized action.'); // tidak punya role → blok
}
```

**Registrasi di bootstrap/app.php:14-15:**
```php
$middleware->alias([
    'role' => \App\Http\Middleware\RoleMiddleware::class,
]);
```

**Penggunaan di route:**
```php
Route::middleware(['auth', 'role:admin,merchant,super-admin'])->prefix('admin')->name('admin.')->group(function () {
    Route::resource('products', \App\Http\Controllers\Admin\ProductController::class);
});
```

### 7.4 Middleware Chain

Saat request ke `admin/products`:

```
Request
  → auth middleware (apakah login?)
    → role middleware (apakah role admin/merchant/super-admin?)
      → ProductController@index
        → Response kembali
```

Jika auth gagal → redirect ke login (302).
Jika role tidak cocok → abort 403.
Jika semua lolos → controller dipanggil.

---

## 8. Routing — Peta Aplikasi

### 8.1 Anatomi Route

```php
Route::get('/shop/{slug}', [ShopController::class, 'show'])->name('shop.show');
```

| Bagian | Arti |
|--------|------|
| `Route::get` | Method HTTP |
| `/shop/{slug}` | URL pattern (slug adalah parameter) |
| `[ShopController::class, 'show']` | Controller + method |
| `->name('shop.show')` | Nama route (untuk dipanggil di view) |

### 8.2 Route Parameter

```php
// Route: /shop/{slug}
// URL:   /shop/laptop-gaming-2024
public function show($slug)
{
    $product = Product::with(['category', 'brand'])
        ->where('slug', $slug)
        ->firstOrFail();
    // $slug = "laptop-gaming-2024"
}
```

### 8.3 Named Route

Route dengan nama bisa dipanggil dari mana pun tanpa hardcode URL:

```php
// Definisi
Route::get('/shop', ...)->name('shop.index');

// Di controller
return redirect()->route('shop.index');

// Di Blade
<a href="{{ route('shop.index') }}">Shop</a>
<a href="{{ route('shop.show', $product->slug) }}">{{ $product->name }}</a>
```

### 8.4 Route Group

Group untuk mengelompokkan route dengan middleware, prefix, atau nama yang sama:

```php
Route::middleware(['auth', 'role:admin,merchant,super-admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {

    // URL: /admin/products
    // Nama: admin.products.index
    Route::resource('products', \App\Http\Controllers\Admin\ProductController::class);

    // URL: /admin/payments/{payment}/confirm
    // Nama: admin.payments.confirm
    Route::patch('payments/{payment}/confirm', ...)->name('payments.confirm');
});
```

### 8.5 Route Resource

`Route::resource` membuat 7 route CRUD otomatis:

```php
Route::resource('products', ProductController::class);
```

Setara dengan:

| Method | URL | Controller Method | Nama Route |
|--------|-----|------------------|------------|
| GET | `/products` | `index()` | products.index |
| GET | `/products/create` | `create()` | products.create |
| POST | `/products` | `store()` | products.store |
| GET | `/products/{product}` | `show()` | products.show |
| GET | `/products/{product}/edit` | `edit()` | products.edit |
| PATCH/PUT | `/products/{product}` | `update()` | products.update |
| DELETE | `/products/{product}` | `destroy()` | products.destroy |

### 8.6 Route Registration di Codebase

**Webhook tanpa middleware auth:**
```php
Route::post('/midtrans/webhook', [MidtransWebhookController::class, 'handle'])
    ->name('midtrans.webhook');
```

**Route dengan method berbeda:**
```php
Route::post('/cart/add', [CartController::class, 'add'])->name('cart.add');
Route::patch('/cart/update/{productId}', [CartController::class, 'update'])->name('cart.update');
Route::delete('/cart/remove/{productId}', [CartController::class, 'remove'])->name('cart.remove');
```

**Closure route (langsung, tanpa controller):**
```php
Route::get('/dashboard', function () {
    return redirect()->route('orders.index');
})->middleware(['auth', 'verified'])->name('dashboard');
```

---

## 9. Lifecycle Lengkap: Satu Request

Mari trace satu request dari awal sampai akhir:

### Request: POST `/checkout/process`

**1. Browser kirim form** (dengan CSRF token)

```
POST /checkout/process HTTP/1.1
Host: olshop-koneksi.test
Content-Type: application/x-www-form-urlencoded
Cookie: XSRF-TOKEN=abc; session=xyz

_token=abc&address_id=1&shipping_courier=jne&shipping_service=REG&shipping_cost=15000&notes=
```

**2. Apache terima** → cari `index.php` → PHP dieksekusi

**3. Laravel bootstrap:**
- `public/index.php` dipanggil
- Autoload (Composer) — load semua class
- `.env` dibaca
- Service container diinisialisasi
- Kernel HTTP dijalankan

**4. Middleware chain berjalan:**
- `EncryptCookies` — decrypt cookie
- `AddQueuedCookiesToResponse`
- `StartSession` — mulai session, load data
- `ShareErrorsFromSession` — siapkan error flash
- `VerifyCsrfToken` — cocokkan `_token` dengan session
- `SubstituteBindings` — resolve route parameter

**5. Route matching:**

```
POST /checkout/process → CheckoutController@process
```

**6. Controller method dipanggil:**

```php
public function process(Request $request)
{
    // Validasi input
    $request->validate([...]);

    // Baca data cart dari session (via CartService)
    $cart = $this->cartService->getCart();

    // Proses order (dalam DB transaction)
    $order = $this->orderService->placeOrder([...]);

    // Response: redirect ke halaman pembayaran
    return redirect()->route('payments.confirmation', $order->id)
        ->with('success', 'Pesanan berhasil dibuat!');
}
```

**7. Laravel mengirim response:**

```
HTTP/1.1 302 Found
Location: http://olshop-koneksi.test/orders/5/payment
Set-Cookie: session=xyz; ...
```

**8. Browser mengikuti redirect (GET `/orders/5/payment`)**

---

## 10. Webhook — Request dari Sistem Lain

**MidtransWebhookController** menerima request **dari server Midtrans**, bukan dari browser:

```php
public function handle(Request $request)
{
    // $request->all() membaca JSON body dari Midtrans
    $payload = $request->all();

    // Verifikasi signature
    $signatureKey = hash('sha512',
        $payload['order_id'] . $payload['status_code']
        . $payload['gross_amount'] . $serverKey
    );

    if ($signatureKey !== $payload['signature_key']) {
        return response()->json(['message' => 'Invalid'], 403);
    }

    // Proses payment status
    // ...

    return response()->json(['message' => 'Handled']);
}
```

**Perbedaan webhook vs request biasa:**
- Tidak ada CSRF token (dikecualikan di `bootstrap/app.php`)
- Tidak ada session (server Midtrans tidak punya cookie)
- Body dalam format JSON (`$request->all()` membaca JSON otomatis)
- Response dalam JSON, bukan redirect
- CSRF dikecualikan di `bootstrap/app.php`

---

## 11. Cookie

### 11.1 Cookie vs Session

| | Cookie | Session |
|---|--------|---------|
| **Disimpan di** | Browser | Server |
| **Ukuran** | Maks 4KB | Tidak terbatas (tapi makan memory) |
| **Keamanan** | Bisa dibaca/diubah user | Aman (di server) |
| **Kadaluarsa** | Bisa atur tanggal | Hilang saat browser ditutup (biasanya) |
| **Gunakan untuk** | Preferensi, tracking | Data login, cart |

### 11.2 Session Cookie

Session ID dikirim via cookie bernama biasanya `PHPSESSID` atau (di Laravel) nama dari `config/session.php`:

```
Set-Cookie: session=xyz123; expires=...; path=/; httponly; samesite=lax
```

**Flag penting:**
- `httponly` — JavaScript tidak bisa membaca cookie ini (cegah XSS)
- `samesite=lax` — cookie hanya dikirim untuk request same-site (cegah CSRF)

---

## 12. Perangkap Umum

### 12.1 `Headers Already Sent` Error

Error klasik yang muncul kalau ada output sebelum `session_start()` atau `setcookie()`.

```php
<?php
echo "Hello";        // ← output sudah dikirim!
session_start();     // ❌ Error: headers already sent!
```

**Penyebab:**
- Spasi atau BOM sebelum `<?php`
- `echo` atau `print` sebelum fungsi yang butuh header
- File PHP dengan `?>` diikuti spasi/baris baru

**Di codebase ini:** Tidak ada error ini karena Laravel menggunakan **Output Buffer** — semua output ditahan sampai response siap.

### 12.2 CSRF Token Mismatch (419)

**Penyebab:**
- Session expired (terlalu lama di halaman form)
- Cookie tidak dikirim (browser block third-party cookie)
- Form tidak pakai `@csrf`

**Solusi:**
- Refresh halaman
- Pastikan form punya `@csrf`
- Untuk webhook, exclude dari CSRF di `bootstrap/app.php`

### 12.3 Mass Assignment

```php
// ❌ BERBAHAYA — user bisa set field apa pun
Product::create($request->all());

// ✅ AMAN — hanya field yang diizinkan
Product::create($request->only(['name', 'price', 'stock']));
```

Eloquent model punya `$fillable` untuk membatasi field yang bisa diisi massal.

### 12.4 Method Spoofing

HTML form hanya mendukung GET dan POST. Untuk PATCH/DELETE, Laravel pakai method spoofing:

```blade
<form action="{{ route('cart.update', $productId) }}" method="POST">
    @method('PATCH')
    @csrf
    <!-- ... -->
</form>
```

`@method('PATCH')` menghasilkan:
```html
<input type="hidden" name="_method" value="PATCH">
```

Laravel membaca `_method` dan memperlakukan request seolah-olah method-nya PATCH.

---

## 13. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────┐
│                    HTTP REQUEST                           │
├─────────────────────────────────────────────────────────┤
│  Browser → Apache → index.php → Laravel Kernel           │
│                                   ↓                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │              MIDDLEWARE STACK                     │    │
│  │  EncryptCookies → StartSession → VerifyCsrfToken │    │
│  │  → auth → role:admin → ...                       │    │
│  └─────────────────────────────────────────────────┘    │
│                                   ↓                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │              ROUTER                              │    │
│  │  POST /checkout/process → CheckoutController    │    │
│  └─────────────────────────────────────────────────┘    │
│                                   ↓                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │              CONTROLLER                          │    │
│  │  Validasi → Service → Logic                      │    │
│  │  ↓                                               │    │
│  │  Session (baca/tulis)                            │    │
│  │  Database (Eloquent)                             │    │
│  └─────────────────────────────────────────────────┘    │
│                                   ↓                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │              RESPONSE                            │    │
│  │  View → HTML (200)                              │    │
│  │  Redirect → Location header (302)               │    │
│  │  JSON → application/json (200)                  │    │
│  │  Abort → Error page (401/403/404/500)           │    │
│  └─────────────────────────────────────────────────┘    │
│                                   ↓                       │
│  Apache → Browser → Render                              │
└─────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Trace request.** Buka browser, akses `http://olshop-koneksi.test/shop`. Buka DevTools (F12) → Network tab. Lihat:
   - Request method, headers, query string
   - Response status code, headers
   - Cookie yang dikirim

2. **Baca routes/web.php.** Cari route dengan method POST, PATCH, DELETE. Catat middleware masing-masing.

3. **Coba CSRF.** Hapus `@csrf` dari form checkout blade. Coba submit — apa yang terjadi?

4. **Session cart.** Di `CartService.php`, tambah method `getItemCount()` yang sudah ada. Baca kodenya dan jelaskan bagaimana session menyimpan data cart.

5. **Buat route baru.** Di `routes/web.php`, tambah route GET `/ping` yang return `response()->json(['status' => 'ok'])`. Test di browser.

6. **Middleware.** Baca `app/Http/Middleware/RoleMiddleware.php`. Jelaskan apa yang terjadi jika user login tapi tidak punya role yang cocok.

7. **Coba PHP built-in server.** Jalankan `php -S localhost:8000 -t public/` dari terminal di folder project. Akses `http://localhost:8000`. Bandingkan dengan akses lewat Laragon.

---

## 🔗 Referensi

- [PHP Manual: Superglobals](https://www.php.net/manual/en/language.variables.superglobals.php)
- [PHP Manual: Sessions](https://www.php.net/manual/en/book.session.php)
- [Laravel Docs: Request](https://laravel.com/docs/11.x/requests)
- [Laravel Docs: Responses](https://laravel.com/docs/11.x/responses)
- [Laravel Docs: Middleware](https://laravel.com/docs/11.x/middleware)
- [Laravel Docs: Routing](https://laravel.com/docs/11.x/routing)
- [Laravel Docs: CSRF](https://laravel.com/docs/11.x/csrf)
- [MDN: HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- Codebase: `routes/web.php`
- Codebase: `bootstrap/app.php`
- Codebase: `app/Http/Middleware/RoleMiddleware.php`
- Codebase: `app/Services/CartService.php`
- Codebase: `app/Http/Controllers/CheckoutController.php`
- Codebase: `app/Http/Controllers/MidtransWebhookController.php`
