# 06-03: Routing — Jantung dari Navigasi Laravel

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-02-arsitektur-laravel  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: route, route parameter, named route, resource route, route group, route model binding, fallback

---

## 📋 Ringkasan

Routing adalah cara Laravel **mencocokkan URL** yang diminta user dengan **controller** yang akan memprosesnya. Tanpa routing, Laravel tidak tahu harus menjalankan kode mana untuk URL tertentu.

**Target pemahaman:**
- Kamu bisa membaca dan menulis route di `web.php`
- Kamu paham route parameter, named route, resource route
- Kamu paham Route Model Binding
- Kamu bisa route grouping dengan middleware

---

## 1. Route Basics

### 1.1 Anatomi Route

```php
// routes/web.php

Route::method('/path', [Controller::class, 'method'])->name('name');
//  ↑       ↑         ↑                              ↑
//  |       |         |                              └── Nama route (untuk link)
//  |       |         └── Controller@method (array callable)
//  |       └── URL path
//  └── HTTP method: get, post, put, patch, delete
```

### 1.2 Semua Route di Codebase

```php
// routes/web.php — lihat file asli:
Route::get('/', function () {                          // Home
    return redirect()->route('shop.index');
});

Route::get('/shop', ...)                               // List produk
Route::get('/shop/{product:slug}', ...)                 // Detail produk

Route::get('/cart', ...)                               // Lihat cart
Route::post('/cart/add', ...)                          // Tambah ke cart
Route::put('/cart/{cart}', ...)                        // Update cart item
Route::delete('/cart/{cart}', ...)                     // Hapus cart item

Route::get('/checkout', ...)                           // Form checkout
Route::post('/checkout', ...)                          // Proses checkout

Route::get('/orders', ...)                             // Daftar order
Route::get('/orders/{order}', ...)                     // Detail order

Route::post('/payment/callback', ...)                  // Midtrans callback

// Auth routes (breeze/jetstream)
Route::get('/login', ...)
Route::post('/login', ...)
Route::post('/logout', ...)
Route::get('/register', ...)
Route::post('/register', ...)
```

### 1.3 Route dengan Closure (Simple)

```php
// Kadang route langsung return tanpa controller:
Route::get('/', function () {
    return redirect()->route('shop.index');
});

Route::get('/dashboard', function () {
    return redirect()->route('orders.index');
})->middleware(['auth', 'verified']);
```

---

## 2. Route Parameter

### 2.1 Required Parameter

```php
// URL: /shop/laptop-gaming-123
Route::get('/shop/{slug}', [ShopController::class, 'show']);

// Controller:
public function show(string $slug)
{
    $product = Product::where('slug', $slug)->firstOrFail();
    return view('shop.show', compact('product'));
}
```

### 2.2 Optional Parameter

```php
// URL: /orders (tanpa filter) atau /orders/pending (dengan filter)
Route::get('/orders/{status?}', [OrderController::class, 'index']);
// {status?} → optional, default null

// Controller:
public function index(?string $status = null)
{
    if ($status) {
        $orders = Order::where('status', $status)->get();
    } else {
        $orders = Order::all();
    }
    return view('orders.index', compact('orders'));
}
```

### 2.3 Regular Expression Constraint

```php
// Hanya terima angka
Route::get('/orders/{id}', [OrderController::class, 'show'])
    ->where('id', '[0-9]+');

// Hanya terima slug
Route::get('/shop/{slug}', [ShopController::class, 'show'])
    ->where('slug', '[a-z0-9-]+');

// Global constraint (di RouteServiceProvider):
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
    Route::pattern('slug', '[a-z0-9-]+');
}
```

---

## 3. Named Route — Memberi Nama Route

### 3.1 Kenapa Named Route?

Named route memungkinkan kamu **merujuk route via nama**, bukan hardcode URL.

```php
// Tanpa named route — hardcode URL di view:
<a href="/shop">Shop</a>
// Kalau path berubah → cari ganti semua!

// Dengan named route — refer by name:
<a href="{{ route('shop.index') }}">Shop</a>
// Path berubah → cukup di route file
```

### 3.2 Named Route dengan Parameter

```php
Route::get('/shop/{product:slug}', [ShopController::class, 'show'])
    ->name('shop.show');

// Di view:
<a href="{{ route('shop.show', $product->slug) }}">
    {{ $product->name }}
</a>
// Hasil: /shop/laptop-gaming-123

// Multiple parameter:
Route::get('/orders/{order}/items/{item}', ...)->name('orders.items.edit');
// route('orders.items.edit', ['order' => 5, 'item' => 2])
// → /orders/5/items/2
```

### 3.3 Named Route di Codebase

```php
// web.php:
Route::get('/shop', [ShopController::class, 'index'])->name('shop.index');
Route::get('/shop/{product:slug}', [ShopController::class, 'show'])->name('shop.show');
Route::post('/cart/add', [CartController::class, 'addItem'])->name('cart.add');
Route::put('/cart/{cart}', [CartController::class, 'update'])->name('cart.update');
Route::delete('/cart/{cart}', [CartController::class, 'destroy'])->name('cart.destroy');

// Di Blade:
<a href="{{ route('shop.index') }}" class="nav-link">Shop</a>
<a href="{{ route('shop.show', $product->slug) }}" class="btn btn-primary">Detail</a>
<form action="{{ route('cart.destroy', $item['product_id']) }}" method="POST">...</form>
```

### 3.4 route() Helper

```php
// Di controller:
return redirect()->route('orders.index');

// Di blade:
{{ route('shop.index') }}         // /shop
{{ route('shop.show', 'laptop') }} // /shop/laptop

// Dengan query string:
route('shop.index', ['category' => 'laptop', 'sort' => 'price'])
// → /shop?category=laptop&sort=price
```

---

## 4. Resource Route — CRUD Otomatis

### 4.1 Masalah

Untuk CRUD biasa, kamu perlu 7 route:

```php
// Manual — 7 route untuk CRUD products:
Route::get('/products', ...)->name('products.index');
Route::get('/products/create', ...)->name('products.create');
Route::post('/products', ...)->name('products.store');
Route::get('/products/{product}', ...)->name('products.show');
Route::get('/products/{product}/edit', ...)->name('products.edit');
Route::put('/products/{product}', ...)->name('products.update');
Route::delete('/products/{product}', ...)->name('products.destroy');
```

### 4.2 Resource Route

```php
// Satu baris = 7 route:
Route::resource('products', ProductController::class);
```

### 4.3 Tabel Resource Route

| Method | URL | Controller Method | Route Name |
|--------|-----|-------------------|------------|
| GET | `/products` | `index()` | `products.index` |
| GET | `/products/create` | `create()` | `products.create` |
| POST | `/products` | `store()` | `products.store` |
| GET | `/products/{product}` | `show()` | `products.show` |
| GET | `/products/{product}/edit` | `edit()` | `products.edit` |
| PUT | `/products/{product}` | `update()` | `products.update` |
| DELETE | `/products/{product}` | `destroy()` | `products.destroy` |

### 4.4 API Resource

```php
// Untuk API — hanya index, store, show, update, destroy
Route::apiResource('products', ProductController::class);
// Tanpa create, edit (karena API tidak butuh form)
```

### 4.5 Partial Resource

```php
// Hanya route tertentu:
Route::resource('products', ProductController::class)->only([
    'index', 'show'
]);

// Kecualikan route tertentu:
Route::resource('products', ProductController::class)->except([
    'create', 'edit'
]);
```

---

## 5. Route Grouping

### 5.1 Group dengan Middleware

```php
// Semua route dalam grup ini pakai middleware 'auth'
Route::middleware(['auth'])->group(function () {
    Route::get('/orders', [OrderController::class, 'index'])->name('orders.index');
    Route::get('/orders/{order}', [OrderController::class, 'show'])->name('orders.show');
    Route::get('/cart', [CartController::class, 'index'])->name('cart.index');
    Route::post('/cart/add', [CartController::class, 'addItem'])->name('cart.add');
});
```

### 5.2 Group dengan Prefix

```php
// Semua route di grup punya prefix /admin
Route::prefix('admin')->middleware(['auth', 'role:admin'])->group(function () {
    Route::resource('products', Admin\ProductController::class);
    Route::resource('categories', Admin\CategoryController::class);
    Route::resource('orders', Admin\OrderController::class);
    // → /admin/products, /admin/categories, /admin/orders
});
```

### 5.3 Group dengan Name

```php
Route::name('admin.')->prefix('admin')->group(function () {
    Route::resource('products', Admin\ProductController::class);
    // → route name: admin.products.index, admin.products.show, dll.
});
```

### 5.4 Group dengan Where

```php
Route::where(['id' => '[0-9]+'])->group(function () {
    Route::get('/orders/{id}', [OrderController::class, 'show']);
    Route::get('/users/{id}', [UserController::class, 'show']);
});
```

### 5.5 Group di Codebase

```php
// Di web.php kira-kira seperti:
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/orders', [OrderController::class, 'index'])->name('orders.index');
    Route::get('/orders/{order}', [OrderController::class, 'show'])->name('orders.show');
});
```

---

## 6. Route Model Binding

### 6.1 Tanpa Binding

```php
// Tanpa Route Model Binding — manual query
Route::get('/orders/{id}', function ($id) {
    $order = Order::findOrFail($id); // Query manual!
    return view('orders.show', compact('order'));
});
```

### 6.2 Dengan Binding (Implicit)

```php
// Dengan Route Model Binding — otomatis query!
Route::get('/orders/{order}', function (Order $order) {
    // Laravel otomatis: Order::findOrFail($id)
    return view('orders.show', compact('order'));
});
```

**Cara kerja:**
1. Route parameter `{order}` cocok dengan type-hint `Order $order`
2. Laravel lihat `{order}` → cari `id` di URL
3. Jalankan `Order::findOrFail($id)`
4. Inject hasilnya ke parameter

### 6.3 Custom Key Binding

```php
// Default: cari berdasarkan 'id'
// /shop/5 → Product::findOrFail(5)

// Custom key: cari berdasarkan 'slug'
// /shop/laptop-gaming → Product::where('slug', 'laptop-gaming')->firstOrFail()
Route::get('/shop/{product:slug}', [ShopController::class, 'show']);
// Ditulis: {nama_model:nama_kolom}
```

### 6.4 Soft Deleted Models

```php
// Default: hanya yang tidak dihapus (deleted_at IS NULL)
// Untuk include soft deleted:
Route::get('/orders/{order}', function (Order $order) {
    // ...
})->withTrashed();
```

---

## 7. Fallback Route

```php
// Route ini dijalankan jika tidak ada route lain yang cocok
Route::fallback(function () {
    return view('errors.404');
    // Atau redirect:
    // return redirect()->route('shop.index');
});
```

---

## 8. Route List — Melihat Semua Route

```bash
php artisan route:list
# Lihat semua route yang terdaftar

php artisan route:list --path=api
# Hanya route dengan prefix api

php artisan route:list --method=GET
# Hanya route GET
```

---

## 9. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ ROUTE ANATOMY                                                      │
│  Route::get('/shop/{product:slug}', [ShopController::class, 'show')│
│  → method: GET                                                     │
│  → path: /shop/{product:slug}                                     │
│  → controller: ShopController@show                                 │
│  → name: (optional, pakai ->name('shop.show'))                    │
├─────────────────────────────────────────────────────────────────────┤
│ ROUTE TYPES                                                        │
│  Basic:  Route::get('/path', [C::class, 'method'])                │
│  With param: Route::get('/path/{param?}', [C::class, 'method'])   │
│  Named:   ->name('route.name') — untuk route() helper             │
│  Resource: Route::resource('photos', PhotoController)              │
│  Group:   Route::middleware([])->group(function() { ... })        │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  web.php: Shop, Cart, Checkout, Order, Auth routes                │
│  Named:   shop.index, cart.index, orders.show, dll                │
│  Binding: {product:slug} — find by slug, bukan id                 │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Baca web.php.** Buka `routes/web.php` dan identifikasi:
   - Route GET
   - Route POST
   - Route PUT
   - Route DELETE
   - Route dengan parameter
   - Named route

2. **Route list.** Jalankan `php artisan route:list`. Berapa total route? Filter hanya method GET.

3. **Cek named route.** Di Blade, cari `route('shop.show', ...)`. Parameter apa yang dikirim? Apa URL yang dihasilkan?

4. **Route model binding.** Cari route dengan `{product:slug}`. Kenapa pakai `slug` bukan `id`? Apa bedanya?

5. **Buat route baru.** Tambah route GET `/promo` yang return view `promo.index`. Beri name `promo.index`.

---

## 🔗 Referensi

- [Laravel Docs: Routing](https://laravel.com/docs/11.x/routing)
- [Laravel Docs: Route Model Binding](https://laravel.com/docs/11.x/routing#route-model-binding)
- [Laravel Docs: Resource Controllers](https://laravel.com/docs/11.x/controllers#resource-controllers)
- Codebase: `routes/web.php` — semua route web
- Command: `php artisan route:list`
