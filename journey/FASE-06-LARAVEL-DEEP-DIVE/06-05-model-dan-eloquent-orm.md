# 06-05: Model dan Eloquent ORM — Data dengan Gaya

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-04-controller-dan-request  
> **Waktu baca**: 80-95 menit  
> **Kata kunci**: Eloquent, ORM, model, relationship, accessor, mutator, query scope, pagination, collection

---

## 📋 Ringkasan

Eloquent adalah **ORM** (Object-Relational Mapping) Laravel — cara berinteraksi dengan database menggunakan **object PHP** daripada SQL mentah. Setiap table punya **Model** yang mewakilinya.

**Target pemahaman:**
- Kamu paham konsep ORM dan Active Record
- Kamu bisa membuat model, relationship, query
- Kamu paham accessor, mutator, scope
- Kamu bisa menggunakan pagination dan collection

---

## 1. ORM — Object-Relational Mapping

### 1.1 Tanpa ORM (SQL Mentah)

```php
// ❌ SQL mentah — rentan error, typing, tidak ada autocomplete
$pdo = DB::connection()->getPdo();
$stmt = $pdo->prepare('SELECT * FROM products WHERE is_active = ?');
$stmt->execute([1]);
$products = $stmt->fetchAll();
// $products = array of arrays — tidak ada method, typing manual
foreach ($products as $p) {
    echo $p['name']; // string? null? harus diingat manual
}
```

### 1.2 Dengan Eloquent ORM

```php
// ✅ Eloquent — object-oriented, type-safe, method chaining
$products = Product::where('is_active', true)->get();
// $products = Collection of Product objects

foreach ($products as $product) {
    echo $product->name;          // property → column
    echo $product->category->name; // relationship!
}

// Query builder fluent
Product::where('is_active', true)
    ->where('price', '>', 10000)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

### 1.3 Active Record Pattern

Eloquent menggunakan **Active Record** — satu Model instance = satu baris di database.

```
products table (baris = 1 produk)
┌────┬──────────────────┬────────┬───────────┐
│ id │ name             │ price  │ is_active │
├────┼──────────────────┼────────┼───────────┤
│ 1  │ Laptop Gaming    │ 15000000 │ 1        │ ← $product = Product::find(1)
│ 2  │ Mouse Wireless   │ 250000 │ 1        │
└────┴──────────────────┴────────┴───────────┘

$product = Product::find(1);
$product->name;       // "Laptop Gaming"
$product->price;      // 15000000
$product->is_active;  // true

$product->price = 14000000; // ubah property
$product->save();            // simpan ke database!
```

---

## 2. Model — Representasi Table

### 2.1 Model Sederhana

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    // Convention → tanpa konfigurasi:
    // Table: "products" (plural dari class name)
    // Primary key: "id"
    // Timestamps: created_at, updated_at (true)
}
```

### 2.2 Model dengan Konfigurasi

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    // Custom table name (default: plural dari class)
    protected $table = 'my_products';

    // Custom primary key (default: 'id')
    protected $primaryKey = 'product_id';

    // Auto-increment? (default: true)
    public $incrementing = true;

    // Primary key type (default: 'int')
    protected $keyType = 'string'; // untuk UUID

    // Timestamps? (default: true)
    public $timestamps = true;

    // Custom timestamps field names
    const CREATED_AT = 'created_date';
    const UPDATED_AT = 'updated_date';

    // Mass assignment — fillable (yang boleh diisi massal)
    protected $fillable = [
        'name', 'slug', 'price', 'stock_quantity',
        'category_id', 'brand_id', 'is_active',
        'description', 'short_description', 'weight_grams',
    ];

    // Mass assignment — guarded (yang tidak boleh diisi)
    // guarded = ['*'] artinya SEMUA guarded (hanya via fillable)
    protected $guarded = ['id', 'created_at', 'updated_at'];

    // Hidden — tidak muncul di JSON
    protected $hidden = ['password', 'remember_token'];

    // Casts — konversi tipe otomatis
    protected $casts = [
        'is_active' => 'boolean',
        'price' => 'integer',
        'stock_quantity' => 'integer',
        'weight_grams' => 'integer',
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
        'metadata' => 'array',  // JSON → array PHP
    ];
}
```

### 2.3 Model di Codebase

```php
// app/Models/Product.php
class Product extends Model
{
    use HasFactory, InteractsWithMedia;

    protected $fillable = [
        'name', 'slug', 'short_description', 'description',
        'price', 'stock_quantity', 'weight_grams',
        'category_id', 'brand_id', 'is_active',
        'meta_title', 'meta_description',
    ];

    protected $casts = [
        'is_active' => 'boolean',
        'price' => 'integer',
        'stock_quantity' => 'integer',
    ];

    // Relationships
    public function category(): BelongsTo { ... }
    public function brand(): BelongsTo { ... }
    public function orderItems(): HasMany { ... }
}
```

---

## 3. CRUD dengan Eloquent

### 3.1 Create

```php
// Cara 1: create() — mass assignment
$product = Product::create([
    'name' => 'Laptop Gaming',
    'slug' => 'laptop-gaming',
    'price' => 15000000,
    'stock_quantity' => 10,
    'category_id' => 1,
    'is_active' => true,
]);

// Cara 2: make() + save()
$product = new Product();
$product->name = 'Laptop Gaming';
$product->slug = 'laptop-gaming';
$product->price = 15000000;
$product->save();

// Cara 3: firstOrCreate() — cari dulu, buat jika tidak ada
$product = Product::firstOrCreate(
    ['slug' => 'laptop-gaming'],    // kondisi cari
    ['name' => 'Laptop Gaming', 'price' => 15000000] // data tambahan
);

// Cara 4: updateOrCreate() — update jika ada, buat jika tidak
$product = Product::updateOrCreate(
    ['slug' => 'laptop-gaming'],
    ['price' => 14000000] // update jika ditemukan
);
```

### 3.2 Read

```php
// Semua
$products = Product::all();                          // Collection

// Find by ID
$product = Product::find(1);                         // Model or null
$product = Product::findOrFail(1);                   // Model or 404

// Find by multiple IDs
$products = Product::find([1, 2, 3]);                // Collection

// First by condition
$product = Product::where('slug', 'laptop')->first(); // Model or null
$product = Product::where('slug', 'laptop')->firstOrFail(); // Model or 404

// Where
$products = Product::where('is_active', true)->get();
$products = Product::where('price', '>', 10000)->get();
$products = Product::whereBetween('price', [10000, 50000])->get();
$products = Product::whereIn('category_id', [1, 3, 5])->get();
$products = Product::whereNull('deleted_at')->get();

// Chaining
$products = Product::where('is_active', true)
    ->where('price', '>', 10000)
    ->orderBy('price', 'desc')
    ->take(10)
    ->get();

// Count, sum, avg, min, max
$count = Product::where('is_active', true)->count();
$total = Order::where('status', 'paid')->sum('total_amount');
$avg = Product::avg('price');
```

### 3.3 Update

```php
// Cara 1: save()
$product = Product::find(1);
$product->price = 14000000;
$product->save();

// Cara 2: update() — mass assignment
Product::where('is_active', true)->update(['price' => 10000]);

// Cara 3: update on query
$product = Product::find(1);
$product->update(['price' => 14000000]);
```

### 3.4 Delete

```php
// Soft delete (jika model pakai SoftDeletes)
$product = Product::find(1);
$product->delete(); // set deleted_at, tidak hilang dari DB

// Force delete (benar-benar hilang)
$product->forceDelete();

// Restore soft delete
Product::withTrashed()->find(1)->restore();

// Delete by query
Product::where('is_active', false)->delete();
```

---

## 4. Relationships — Relasi Antar Model

### 4.1 Jenis Relationship

```php
// app/Models/Product.php
class Product extends Model
{
    // BelongsTo — milik kategori
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
        // product.category_id → categories.id
    }

    // BelongsTo — milik brand
    public function brand(): BelongsTo
    {
        return $this->belongsTo(Brand::class);
    }

    // HasMany — punya banyak order items
    public function orderItems(): HasMany
    {
        return $this->hasMany(OrderItem::class);
        // products.id → order_items.product_id
    }
}
```

```php
// app/Models/Category.php
class Category extends Model
{
    // HasMany — punya banyak produk
    public function products(): HasMany
    {
        return $this->hasMany(Product::class);
        // categories.id → products.category_id
    }

    // BelongsTo — parent category (self-referential)
    public function parent(): BelongsTo
    {
        return $this->belongsTo(Category::class, 'parent_id');
    }

    // HasMany — child categories
    public function children(): HasMany
    {
        return $this->hasMany(Category::class, 'parent_id');
    }
}
```

```php
// app/Models/Order.php
class Order extends Model
{
    // BelongsTo — milik user
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    // HasMany — punya banyak item
    public function items(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }

    // HasMany through Payment (polymorphic atau hasMany)
    public function payment(): HasOne
    {
        return $this->hasOne(Payment::class);
    }
}
```

```php
// app/Models/OrderItem.php
class OrderItem extends Model
{
    // BelongsTo — milik order
    public function order(): BelongsTo
    {
        return $this->belongsTo(Order::class);
    }

    // BelongsTo — milik product
    public function product(): BelongsTo
    {
        return $this->belongsTo(Product::class);
    }
}
```

### 4.2 Menggunakan Relationship

```php
// Eager loading — ambil relasi sekali query
$products = Product::with(['category', 'brand'])->get();
// SQL: SELECT * FROM products
// SQL: SELECT * FROM categories WHERE id IN (1, 2, 3, ...)
// SQL: SELECT * FROM brands WHERE id IN (1, 2, 3, ...)

foreach ($products as $product) {
    echo $product->category->name; // tidak perlu query lagi
    echo $product->brand->name;
}

// Nested eager loading
$orders = Order::with('items.product')->get();
// Ambil order + order_items + products sekaligus

// Conditional eager loading
$products = Product::with(['category' => function ($query) {
    $query->where('is_active', true);
}])->get();

// Lazy loading (N+1 problem — HINDARI!)
$products = Product::all(); // 1 query
foreach ($products as $product) {
    echo $product->category->name; // 1 query PER produk = N query!
}
// Total: 1 + N query — ini yang disebut N+1 problem
```

### 4.3 N+1 Problem

```php
// ❌ Buruk — N+1 query
$products = Product::all();
foreach ($products as $p) {
    echo $p->category->name; // query kategori untuk setiap produk!
}

// ✅ Baik — 2 query total
$products = Product::with('category')->get();
foreach ($products as $p) {
    echo $p->category->name; // sudah di-load, no extra query
}
```

---

## 5. Accessor, Mutator, Cast

### 5.1 Accessor — Getter

Accessor memodifikasi nilai saat **diambil** dari model.

```php
// app/Models/Product.php
public function getPriceAttribute(string $value): string
{
    return 'Rp ' . number_format($value, 0, ',', '.');
}

// Saat dipakai:
$product->price; // "Rp 15.000.000" — bukan 15000000
// Tapi $product->getRawOriginal('price') = 15000000 (asli)
```

### 5.2 Mutator — Setter

Mutator memodifikasi nilai saat **disimpan** ke model.

```php
// app/Models/Product.php
public function setNameAttribute(string $value): void
{
    $this->attributes['name'] = ucfirst(strtolower($value));
    // "LAPTOP GAMING" → "Laptop Gaming"
}

// Saat di-set:
$product->name = 'LAPTOP GAMING';
// Tersimpan: "Laptop Gaming"
```

### 5.3 Cast — Konversi Otomatis

```php
protected $casts = [
    'is_active' => 'boolean',      // integer 0/1 → boolean
    'price' => 'integer',           // string → integer
    'created_at' => 'datetime',    // string → Carbon
    'metadata' => 'array',          // JSON string → array
    'config' => 'object',           // JSON → stdClass
    'options' => 'collection',      // JSON → Collection
];

// Dengan cast:
$product->is_active; // true/false (boolean), bukan 1/0
$product->created_at; // Carbon object → bisa format()
$product->created_at->format('d/m/Y');
```

---

## 6. Query Scopes — Query yang Bisa Dipakai Ulang

### 6.1 Local Scope

```php
// app/Models/Product.php
public function scopeActive(Builder $query): Builder
{
    return $query->where('is_active', true);
}

public function scopeInStock(Builder $query): Builder
{
    return $query->where('stock_quantity', '>', 0);
}

public function scopePriceBetween(Builder $query, int $min, int $max): Builder
{
    return $query->whereBetween('price', [$min, $max]);
}

// Pemakaian:
$products = Product::active()->inStock()->get();
$products = Product::active()->priceBetween(10000, 50000)->get();
```

### 6.2 Global Scope

```php
// Global scope — otomatis terpakai di SEMUA query model
// app/Models/Product.php
protected static function booted(): void
{
    static::addGlobalScope('active', function (Builder $builder) {
        $builder->where('is_active', true);
    });
}

// Semua query Product otomatis punya WHERE is_active = 1
$products = Product::all(); // WHERE is_active = 1

// Menonaktifkan global scope:
$products = Product::withoutGlobalScope('active')->get();
$products = Product::withoutGlobalScopes()->get(); // semua scope
```

---

## 7. Pagination

### 7.1 Paginate

```php
// Controller
$products = Product::where('is_active', true)
    ->with(['category', 'brand'])
    ->orderBy('created_at', 'desc')
    ->paginate(12); // 12 item per halaman

return view('shop.index', compact('products'));
```

```blade
{{-- View — pagination links --}}
@foreach($products as $product)
    {{-- card produk --}}
@endforeach

{{-- Bootstrap pagination --}}
{{ $products->links() }}
{{-- atau: {{ $products->onEachSide(2)->links() }} --}}
```

### 7.2 Pagination Methods

```php
$products = Product::paginate(12);

$products->currentPage();  // halaman saat ini
$products->lastPage();     // halaman terakhir
$products->total();        // total item
$products->perPage();      // item per halaman
$products->hasPages();     // ada halaman lain?
$products->hasMorePages(); // ada halaman berikutnya?
$products->firstItem();    // nomor item pertama
$products->lastItem();     // nomor item terakhir
$products->onFirstPage();  // sedang di halaman 1?
```

---

## 8. Collection — Hasil Query Eloquent

### 8.1 Collection Methods

```php
$products = Product::where('is_active', true)->get();

// Filtering
$cheap = $products->filter(fn($p) => $p->price < 50000);
$inStock = $products->where('stock_quantity', '>', 0);

// Transformation
$names = $products->pluck('name');
$prices = $products->map(fn($p) => $p->price);

// Aggregation
$total = $products->sum('price');
$avg = $products->avg('price');
$max = $products->max('price');

// Sorting
$sorted = $products->sortBy('price');
$sortedDesc = $products->sortByDesc('price');

// Grouping
$byCategory = $products->groupBy('category_id');
```

---

## 9. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ ELOQUENT ORM — Object-Relational Mapping                           │
│  Model = Table                                                     │
│  Instance = Row                                                    │
│  Property = Column                                                 │
│  Method = Relationship / Query                                     │
├─────────────────────────────────────────────────────────────────────┤
│ CRUD                                                               │
│  Create:  Product::create([...])                                  │
│  Read:    Product::find(1), Product::where(...)->get()            │
│  Update:  $product->update([...])                                 │
│  Delete:  $product->delete()                                      │
├─────────────────────────────────────────────────────────────────────┤
│ RELATIONSHIPS                                                      │
│  BelongsTo:   $product->category  (FK di sini)                    │
│  HasMany:     $category->products (FK di sana)                    │
│  HasOne:      $order->payment                                     │
├─────────────────────────────────────────────────────────────────────┤
│ ACCESSOR / MUTATOR / CAST                                          │
│  getXAttribute → modifikasi saat GET                              │
│  setXAttribute → modifikasi saat SET                              │
│  $casts → konversi tipe otomatis                                  │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  Product → category(), brand(), orderItems()                      │
│  Order → user(), items(), payment()                               │
│  ShopController → paginate(12) + with(['category','brand'])       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Baca model.** Buka `app/Models/Product.php`. Identifikasi:
   - Fillable fields
   - Casts
   - Relationships
   - Scopes (jika ada)

2. **Eksplorasi tinker.** Jalankan `php artisan tinker`:
   ```php
   Product::count();                    // berapa produk?
   Product::where('is_active', true)->count();
   Product::first()->name;             // produk pertama
   Product::with('category')->first()->category->name; // relasi
   ```

3. **N+1 test.** Buka `app/Http/Controllers/ShopController.php`. Apakah pakai `with()` untuk eager loading? Cek query di Telescope.

4. **Cek pagination.** Di `ShopController@index`, berapa item per halaman? Coba ganti jadi 20.

5. **Buat scope.** Tambah `scopePriceAbove(int $min)` di Product model. Pakai di controller.

---

## 🔗 Referensi

- [Laravel Docs: Eloquent ORM](https://laravel.com/docs/11.x/eloquent)
- [Laravel Docs: Relationships](https://laravel.com/docs/11.x/eloquent-relationships)
- [Laravel Docs: Accessors & Mutators](https://laravel.com/docs/11.x/eloquent-mutators)
- [Laravel Docs: Scopes](https://laravel.com/docs/11.x/eloquent#query-scopes)
- [Laravel Docs: Pagination](https://laravel.com/docs/11.x/pagination)
- Codebase: `app/Models/` — semua model
