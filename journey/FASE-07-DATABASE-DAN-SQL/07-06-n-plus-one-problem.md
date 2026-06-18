# 07-06: N+1 Problem — Silent Performance Killer

> **Fase**: 7 — Database & SQL  
> **Prasyarat**: 07-05-index-dan-performance  
> **Waktu baca**: 55-70 menit  
> **Kata kunci**: N+1, lazy loading, eager loading, with, loadCount, Telescope, query optimization

---

## 📋 Ringkasan

N+1 adalah **masalah performa paling umum** di Laravel. Terjadi saat kamu mengambil data dari relasi tanpa eager loading — query tambahan dijalankan untuk setiap baris.

**Target pemahaman:**
- Kamu bisa mendeteksi N+1 problem
- Kamu bisa memperbaikinya dengan eager loading
- Kamu paham cara mencegah N+1
- Kamu bisa menggunakan alat deteksi (Telescope, Laravel Debugbar)

---

## 1. Apa Itu N+1?

### 1.1 Contoh Klasik

```php
// Controller:
$products = Product::all(); // 1 query: SELECT * FROM products

// View:
@foreach($products as $product)
    <p>{{ $product->category->name }}</p> {{-- N query! --}}
@endforeach
```

**Jika ada 100 produk:**
```
Query 1:  SELECT * FROM products                       → 1 query
Query 2:  SELECT * FROM categories WHERE id = 1        → untuk produk 1
Query 3:  SELECT * FROM categories WHERE id = 1        → untuk produk 2
Query 4:  SELECT * FROM categories WHERE id = 2        → untuk produk 3
...
Query 101: SELECT * FROM categories WHERE id = 5       → untuk produk 100
```

**Total: 1 + 100 = 101 query!** (makanya disebut N+1)

### 1.2 Visual

```
TANPA EAGER LOADING:
Product::all()                    [1 query]
  ├── $product->category->name   [1 query]  ← N query untuk N produk
  ├── $product->category->name   [1 query]
  ├── $product->category->name   [1 query]
  └── ...                        [N query]

DENGAN EAGER LOADING:
Product::with('category')->get()  [2 query]
  ├── $product->category->name   [0 query] ← Data sudah di-load
  ├── $product->category->name   [0 query]
  ├── $product->category->name   [0 query]
  └── ...
```

---

## 2. Mendeteksi N+1

### 2.1 Telescope

Buka `http://olshop-koneksi.test/telescope/queries` dan reload halaman shop:

```
Query list di Telescope:
1. SELECT * FROM `products` WHERE `is_active` = ?
2. SELECT * FROM `categories` WHERE `id` IN (?, ?, ?)  ← Ini eager loading (baik!)
   Bindings: [1, 2, 3]
   Hanya 2 query! ✅

Tapi jika N+1:
1. SELECT * FROM `products` WHERE `is_active` = ?
2. SELECT * FROM `categories` WHERE `id` = 1
3. SELECT * FROM `categories` WHERE `id` = 1
4. SELECT * FROM `categories` WHERE `id` = 2
5. SELECT * FROM `categories` WHERE `id` = 2
... (N query berulang)
   Banyak query! ❌
```

### 2.2 Laravel Debugbar

Install Laravel Debugbar untuk melihat query di bagian bawah halaman:

```bash
composer require barryvdh/laravel-debugbar --dev
```

Debugbar menampilkan:
- Jumlah query
- Query list dengan timing
- Duplikasi query (N+1)

### 2.3 Load Counting

```php
// Cek jumlah query di tinker:
DB::enableQueryLog();

$products = Product::all();
foreach ($products as $product) {
    echo $product->category->name;
}

dump(count(DB::getQueryLog()));
// 101 query untuk 100 produk — N+1!
```

---

## 3. Memperbaiki N+1

### 3.1 Eager Loading dengan `with()`

```php
// ✅ Perbaiki: eager loading
$products = Product::with('category')->get();
// 2 query: products + categories

foreach ($products as $product) {
    echo $product->category->name; // sudah di-cache, no extra query
}
```

### 3.2 Multiple Relationships

```php
// Multiple eager loading:
Product::with(['category', 'brand', 'tags'])->get();
// 4 query: products + categories + brands + tags

// Nested eager loading:
Order::with('items.product.category')->get();
// orders + order_items + products + categories
```

### 3.3 Conditional Eager Loading

```php
// Hanya load relasi jika diperlukan:
$products = Product::query();
if ($request->has('include')) {
    $products->with('category');
}
$products = $products->get();
```

### 3.4 Lazy Eager Loading

```php
// Jika terlanjur query tanpa with:
$products = Product::all(); // tanpa with

if (auth()->user()->isAdmin()) {
    $products->load('category'); // lazy load setelahnya
    $products->load(['category', 'brand']); // multiple
}
```

---

## 4. withCount — Tanpa Load Relasi

Kadang kamu hanya perlu **jumlah** relasi, bukan data relasi:

```php
// ❌ N+1 — load semua kategori hanya untuk count
$categories = Category::all();
foreach ($categories as $category) {
    echo $category->products->count(); // load semua produk!
}

// ✅ withCount — hitung tanpa load data
$categories = Category::withCount('products')->get();
// 2 query: categories + COUNT per category

foreach ($categories as $category) {
    echo $category->products_count; // langsung tersedia
}
```

```sql
-- SQL yang dihasilkan withCount:
SELECT * FROM categories;
SELECT category_id, COUNT(*) AS products_count
FROM products
WHERE category_id IN (1, 2, 3, ...)
GROUP BY category_id;
```

### withCount di Codebase

```php
// Contoh jika ada:
Category::withCount('products')->get();
```

---

## 5. Mencegah N+1

### 5.1 Strict Mode (Laravel 8+)

```php
// app/Providers/AppServiceProvider.php

use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    // Prevent lazy loading in development
    Model::preventLazyLoading(!app()->isProduction());

    // Atau selalu prevent:
    Model::preventLazyLoading();
}
```

Saat lazy loading terjadi, Laravel throw **exception**:

```
Illuminate\Database\LazyLoadingViolationException
Attempted to lazy load [category] on model [App\Models\Product]
```

### 5.2 Resource Controller Pattern

```php
// Selalu eager load di controller:
class ProductController extends Controller
{
    public function index(): View
    {
        $products = Product::with(['category', 'brand'])
                          ->where('is_active', true)
                          ->paginate(12);

        return view('shop.index', compact('products'));
    }
}
```

### 5.3 View Composer

```php
// app/Providers/AppServiceProvider.php
public function boot(): void
{
    view()->composer('partials.product-card', function ($view) {
        // Pastikan data sudah di-load sebelum view
        $products = $view->getData()['product'];
        // Jika perlu, bisa load relasi di sini
    });
}
```

---

## 6. N+1 di Codebase

### 6.1 Cek ShopController

```php
// app/Http/Controllers/ShopController.php
public function index(ShopRequest $request): View
{
    $products = Product::with(['category', 'brand'])
        ->where('is_active', true)
        ->when(...)
        ->paginate(12);

    return view('shop.index', compact('products'));
}
```

✅ **Sudah pakai eager loading.** Tidak ada N+1 di halaman shop.

### 6.2 Cek Halaman Lain

```php
// Periksa apakah ada N+1 di:
// - OrderController@index → Order::with('items')->get()
// - CategoryController@index → Category::withCount('products')
// - CartController@index → apakah ada relasi?
```

---

## 7. Other Common Performance Problems

### 7.1 Select * → Select Kolom Tertentu

```php
// ❌ SELECT * — ambil semua kolom
$products = Product::all();

// ✅ SELECT kolom yang diperlukan
$products = Product::select('id', 'name', 'price', 'slug')->get();
```

### 7.2 Lazy Collection for Large Dataset

```php
// ❌ Load 1 juta user ke memory
$users = User::all();

// ✅ Chunk — proses 100 per batch
User::chunk(100, function ($users) {
    foreach ($users as $user) {
        // proses
    }
});

// ✅ Cursor — lazy collection, 1 row per time
foreach (User::cursor() as $user) {
    // proses
}
```

### 7.3 Pagination with Count

```php
// paginate() already optimized — hanya 2 query:
// 1. SELECT COUNT(*) FROM products
// 2. SELECT * FROM products LIMIT 12 OFFSET 0

$products = Product::paginate(12); // ✅ efficient
```

---

## 8. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ N+1 PROBLEM                                                        │
│  Terjadi: Saat akses relasi tanpa eager loading                    │
│  Contoh: $product->category->name di loop                         │
│  Dampak: 1 + N query (100 produk = 101 query)                    │
│  Solusi: Product::with('category')->get() → 2 query               │
├─────────────────────────────────────────────────────────────────────┤
│ DETEKSI                                                            │
│  Telescope → http://olshop-koneksi.test/telescope/queries         │
│  Cari: query berulang dengan binding berbeda                      │
│  Laravel Debugbar → barryvdh/laravel-debugbar                     │
│  Prevent: Model::preventLazyLoading()                             │
├─────────────────────────────────────────────────────────────────────┤
│ SOLUSI                                                             │
│  with('relasi')        → eager loading                            │
│  withCount('relasi')   → hitung tanpa load data                   │
│  load('relasi')        → lazy eager loading (setelah query)       │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  ShopController → ✅ pakai with(['category', 'brand'])            │
│  Cek controller lain → Telescope verify                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Cek Telescope.** Buka `http://olshop-koneksi.test/telescope/queries`. Buka halaman shop — berapa query? Apakah ada tanda-tanda N+1?

2. **Simulasi N+1.** Di controller, hapus sementara `with()` dan lihat Telescope lagi. Berapa query sekarang?

3. **Cek dengan Debugbar.** Install Laravel Debugbar:
   ```bash
   composer require barryvdh/laravel-debugbar --dev
   ```
   Buka halaman shop — lihat jumlah query di bagian bawah.

4. **Prevent lazy loading.** Aktifkan `Model::preventLazyLoading()` di `AppServiceProvider`. Buka halaman yang mungkin melakukan lazy loading — apakah ada error?

5. **withCount.** Buka `Category` model. Apakah ada scope atau method yang menggunakan `withCount`? Jika tidak, coba buat query untuk menampilkan jumlah produk per kategori.

---

## 🔗 Referensi

- [Laravel Docs: Eager Loading](https://laravel.com/docs/11.x/eloquent-relationships#eager-loading)
- [Laravel Docs: Preventing Lazy Loading](https://laravel.com/docs/11.x/eloquent-relationships#preventing-lazy-loading)
- [Laravel News: N+1 Problem](https://laravel-news.com/n-plus-one)
- Codebase: `app/Http/Controllers/ShopController.php` — with()
- Codebase: Telescope → Queries
