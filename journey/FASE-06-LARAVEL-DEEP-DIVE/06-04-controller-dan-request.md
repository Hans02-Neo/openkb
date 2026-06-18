# 06-04: Controller dan Request — Otak dari Aplikasi

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-03-routing-web-php  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: controller, request, response, dependency injection, FormRequest, validation, redirect

---

## 📋 Ringkasan

Controller adalah **jembatan** antara route (URL) dan logika aplikasi. Controller menerima request, memproses data (via Model/Service), dan mengembalikan response.

**Target pemahaman:**
- Kamu paham cara membuat dan menggunakan controller
- Kamu bisa inject dependency via constructor dan method
- Kamu paham FormRequest untuk validasi
- Kamu bisa mengembalikan berbagai tipe response

---

## 1. Controller — Middleman

### 1.1 Anatomi Controller

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\View\View;

class ShopController extends Controller
{
    // Method dipanggil oleh route
    public function index(ShopRequest $request): View
    {
        $products = Product::where('is_active', true)->get();

        return view('shop.index', compact('products'));
    }
}
```

### 1.2 Cara Route Memanggil Controller

```php
// Cara 1: Array callable (Laravel 8+)
Route::get('/shop', [ShopController::class, 'index']);
// Laravel buat instance ShopController → panggil index()

// Cara 2: String (Laravel 7-) — deprecated
Route::get('/shop', 'ShopController@index');

// Cara 3: Closure langsung (untuk yang simpel)
Route::get('/shop', function () {
    return view('shop.index');
});
```

### 1.3 Controller dengan Dependency Injection

```php
<?php

namespace App\Http\Controllers;

use App\Services\CartService;
use App\Services\OrderService;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

class CheckoutController extends Controller
{
    // Constructor Injection — service tersedia di semua method
    public function __construct(
        private CartService $cartService,
        private OrderService $orderService,
    ) {}

    public function index(): View
    {
        $cart = $this->cartService->getCart();
        return view('checkout.index', compact('cart'));
    }

    // Method Injection — khusus method ini
    public function process(CheckoutRequest $request): RedirectResponse
    {
        $order = $this->orderService->createOrder(
            $request->validated(),
            $this->cartService->getCart()
        );

        return redirect()->route('orders.show', $order)
            ->with('success', 'Pesanan berhasil dibuat!');
    }
}
```

---

## 2. Request — Data dari User

### 2.1 Mengakses Data Request

```php
<?php

use Illuminate\Http\Request;

public function store(Request $request)
{
    // Semua data (GET + POST + JSON)
    $allData = $request->all();

    // Input tertentu (dengan default)
    $name = $request->input('name', 'Guest');
    $email = $request->input('email');

    // Query string saja (GET parameter)
    $category = $request->query('category');
    $sort = $request->query('sort', 'terbaru');

    // POST data saja
    $password = $request->post('password');

    // Boolean (checkbox)
    $agree = $request->boolean('agree');

    // Hanya field tertentu
    $validated = $request->only(['name', 'email', 'phone']);

    // Kecualikan field tertentu
    $withoutPassword = $request->except(['_token', 'password_confirmation']);

    // Cek apakah field ada
    if ($request->has('email')) { ... }
    if ($request->filled('name')) { ... } // ada dan tidak kosong

    // Old input (setelah validasi gagal)
    $oldName = $request->old('name');
    // Atau di Blade: old('name')
}
```

### 2.2 Request di Codebase

```php
// ShopController.php
public function index(ShopRequest $request): View
{
    // Filter parameter dari query string
    $category = $request->query('category');
    $search = $request->query('search');
    $sort = $request->input('sort', 'terbaru');

    // Gunakan $request->filled() untuk conditional query
    $products = Product::query()
        ->when($request->filled('category'), fn($q) => $q->whereHas('category', ...))
        ->when($request->filled('search'), fn($q) => $q->where(...))
        ->when($request->filled('in_stock'), fn($q) => $q->where('stock', '>', 0))
        ->orderBy(...)
        ->paginate(12);

    return view('shop.index', compact('products'));
}
```

### 2.3 Request Information

```php
public function show(Request $request)
{
    // URL & Path
    $url = $request->url();           // http://olshop-koneksi.test/shop
    $fullUrl = $request->fullUrl();   // http://olshop-koneksi.test/shop?category=laptop
    $path = $request->path();         // shop
    $method = $request->method();     // GET

    // IP & User Agent
    $ip = $request->ip();             // 127.0.0.1
    $ua = $request->userAgent();      // Mozilla/5.0...

    // Ajax?
    $isAjax = $request->ajax();       // true jika header X-Requested-With: XMLHttpRequest
    $wantsJson = $request->wantsJson(); // true jika Accept: application/json

    // Session
    $request->session()->get('cart');
}
```

---

## 3. FormRequest — Validasi Terpisah

### 3.1 Kenapa FormRequest?

Daripada validasi di controller:

```php
// ❌ Validasi di controller — controller jadi gemuk
public function store(Request $request)
{
    $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8',
    ], [
        'name.required' => 'Nama wajib diisi',
    ]);

    User::create($request->all());
}
```

Pisahkan ke FormRequest:

```php
// ✅ Validasi di FormRequest — controller tetap ramping
public function store(StoreUserRequest $request)
{
    User::create($request->validated());
}
```

### 3.2 Membuat FormRequest

```bash
php artisan make:request StoreUserRequest
```

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    // Otorisasi — siapa yang boleh request ini
    public function authorize(): bool
    {
        return true; // semua user boleh register
        // atau: return auth()->user()->isAdmin();
    }

    // Aturan validasi
    public function rules(): array
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|string|min:8|confirmed',
            'phone' => 'nullable|string|max:20',
        ];
    }

    // Custom error messages
    public function messages(): array
    {
        return [
            'name.required' => 'Nama lengkap wajib diisi',
            'email.required' => 'Alamat email wajib diisi',
            'email.email' => 'Format email tidak valid',
            'email.unique' => 'Email sudah terdaftar',
            'password.min' => 'Password minimal 8 karakter',
            'password.confirmed' => 'Konfirmasi password tidak cocok',
        ];
    }

    // Custom attribute names (untuk error :attribute)
    public function attributes(): array
    {
        return [
            'name' => 'Nama Lengkap',
            'email' => 'Alamat Email',
        ];
    }

    // Prepare for validation — modify data sebelum validasi
    protected function prepareForValidation(): void
    {
        $this->merge([
            'email' => strtolower($this->email),
        ]);
    }
}
```

### 3.3 FormRequest di Codebase

```php
// app/Http/Requests/CheckoutRequest.php
class CheckoutRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'customer_id' => 'required|exists:customers,id',
            'items' => 'required|array|min:1',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
            'shipping_address' => 'required|string|min:10',
            'shipping_city' => 'required|string',
            'shipping_postal_code' => 'required|string',
            'shipping_phone' => 'required|string',
        ];
    }
}

// Di Controller — otomatis validasi!
public function process(CheckoutRequest $request): RedirectResponse
{
    // Jika sampai sini, data sudah VALID
    $validated = $request->validated();
    // ...
}
```

### 3.4 Validasi Rules Umum

| Rule | Guna | Contoh |
|------|------|--------|
| `required` | Wajib | `'name' => 'required'` |
| `string` | Harus string | `'name' => 'string'` |
| `email` | Format email | `'email' => 'email'` |
| `numeric` | Harus angka | `'price' => 'numeric'` |
| `integer` | Harus integer | `'qty' => 'integer'` |
| `min:n` | Minimal n | `'password' => 'min:8'` |
| `max:n` | Maksimal n | `'name' => 'max:255'` |
| `unique:table,column` | Unik di DB | `'email' => 'unique:users,email'` |
| `exists:table,column` | Ada di DB | `'product_id' => 'exists:products,id'` |
| `confirmed` | Cocok dengan `_confirmation` | `'password' => 'confirmed'` |
| `image` | File gambar | `'photo' => 'image'` |
| `mimes:ext` | Ekstensi file | `'photo' => 'mimes:jpeg,png'` |
| `max:n_kb` | Max file size (KB) | `'photo' => 'max:2048'` |
| `array` | Harus array | `'items' => 'array'` |
| `boolean` | Harus boolean | `'agree' => 'boolean'` |
| `date` | Format tanggal | `'birth' => 'date'` |
| `after:date` | Setelah tanggal | `'end' => 'after:start'` |
| `in:a,b,c` | Salah satu dari | `'status' => 'in:pending,paid,cancelled'` |

---

## 4. Response — Mengirim Balik ke Browser

### 4.1 View Response

```php
// Paling umum — render Blade
return view('shop.index', ['products' => $products]);
return view('shop.index', compact('products')); // compact() sama

// View dengan layout spesifik
return view('shop.index')
    ->layout('layouts.custom');
```

### 4.2 JSON Response

```php
// JSON response — untuk API
return response()->json([
    'success' => true,
    'data' => $product,
    'message' => 'Product retrieved',
], 200); // status code

// JSON dengan header kustom
return response()->json($data, 201, [
    'X-Custom-Header' => 'Value',
]);
```

### 4.3 Redirect Response

```php
// Redirect ke named route
return redirect()->route('orders.index');

// Redirect ke route dengan parameter
return redirect()->route('orders.show', $order);

// Redirect ke URL
return redirect('/shop');

// Redirect back (ke halaman sebelumnya)
return redirect()->back();
return back(); // helper

// Redirect dengan flash message
return redirect()->route('orders.index')
    ->with('success', 'Order berhasil dibuat!');

// Redirect dengan input (old input)
return redirect()->back()
    ->withInput()
    ->withErrors(['email' => 'Email tidak valid']);
```

### 4.4 Other Response Types

```php
// Plain text
return response('Halo dunia', 200)
    ->header('Content-Type', 'text/plain');

// File download
return response()->download('path/to/file.pdf');

// File display (inline)
return response()->file('path/to/image.jpg');

// No content (204)
return response()->noContent();

// Abort with error page
abort(403, 'Unauthorized');
abort(404);
abort(500);
```

### 4.5 Response di Codebase

```php
// ShopController — View
public function index(ShopRequest $request): View
{
    return view('shop.index', compact('products'));
}

// CartController — Redirect
public function addItem(CartRequest $request): RedirectResponse
{
    $this->cartService->addItem(
        $request->input('product_id'),
        $request->input('quantity', 1)
    );
    return redirect()->route('cart.index')
        ->with('success', 'Produk ditambahkan ke keranjang');
}

// RajaOngkirController — JSON
public function getCities(RajaOngkirService $rajaOngkir): JsonResponse
{
    return response()->json(
        $rajaOngkir->getDummyCities()
    );
}
```

---

## 5. Controller Organization

### 5.1 Controller per Resource

```
app/Http/Controllers/
├── ShopController.php        → Product listing & detail
├── CartController.php        → Cart CRUD
├── CheckoutController.php    → Checkout flow
├── OrderController.php       → Order listing & detail
├── Admin/
│   ├── ProductController.php → Admin product management
│   ├── CategoryController.php
│   └── OrderController.php
├── Auth/
│   ├── LoginController.php
│   ├── RegisterController.php
│   └── ...
└── RajaOngkirController.php  → Shipping API
```

### 5.2 Invokable Controller

Untuk controller dengan satu method:

```php
// Controller dengan __invoke
php artisan make:controller ShowProduct --invokable

class ShowProduct extends Controller
{
    public function __invoke(string $slug): View
    {
        $product = Product::where('slug', $slug)->firstOrFail();
        return view('shop.show', compact('product'));
    }
}

// Route:
Route::get('/shop/{product:slug}', ShowProduct::class);
// Tanpa [Controller::class, 'method'] — langsung class
```

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ CONTROLLER LIFECYCLE                                               │
│  Route match → Controller → Dependency Injection → Method call     │
│    ├── Request object (data dari user)                            │
│    ├── Service/Model (logika)                                     │
│    └── Response (view/json/redirect)                              │
├─────────────────────────────────────────────────────────────────────┤
│ REQUEST                                                            │
│  $request->input('name')     → GET/POST data                      │
│  $request->query('category') → Query string                       │
│  $request->validated()       → Hanya field yang lolos validasi    │
│  $request->file('photo')     → Uploaded file                      │
├─────────────────────────────────────────────────────────────────────┤
│ RESPONSE                                                           │
│  view('shop.index', $data)   → HTML (Blade)                       │
│  response()->json($data)     → JSON (API)                         │
│  redirect()->route('x')      → Redirect (302)                     │
│  abort(404)                  → Error page                          │
├─────────────────────────────────────────────────────────────────────┤
│ FORMREQUEST                                                        │
│  authorize() → siapa yang boleh                                   │
│  rules()     → aturan validasi                                    │
│  messages()  → custom error                                       │
│  validated() → data yang sudah valid                              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Baca controller.** Buka `app/Http/Controllers/ShopController.php`. Identifikasi:
   - Constructor injection (jika ada)
   - Method injection di parameter
   - Tipe response yang dikembalikan
   - Bagaimana data dari request diakses

2. **Buat FormRequest.** Buat `StoreProductRequest` dengan validasi:
   - `name` required, string, max:255
   - `price` required, numeric, min:0
   - `stock` required, integer, min:0
   - `description` nullable, string

3. **Cek response types.** Di `CartController`, apa tipe return setiap method? (index, addItem, update, destroy)

4. **Request data.** Di `CheckoutController@process`, data apa saja yang diambil dari request?

5. **Error handling.** Submit form checkout dengan data kosong. Apa response-nya? Status code berapa?

---

## 🔗 Referensi

- [Laravel Docs: Controllers](https://laravel.com/docs/11.x/controllers)
- [Laravel Docs: HTTP Request](https://laravel.com/docs/11.x/requests)
- [Laravel Docs: FormRequest Validation](https://laravel.com/docs/11.x/validation#form-request-validation)
- [Laravel Docs: HTTP Responses](https://laravel.com/docs/11.x/responses)
- Codebase: `app/Http/Controllers/`
- Codebase: `app/Http/Requests/`
