# 07-03: Relationship dan Join — Menghubungkan Data

> **Fase**: 7 — Database & SQL  
> **Prasyarat**: 07-02-sql-dasar-crud  
> **Waktu baca**: 70-85 menit  
> **Kata kunci**: relationship, one-to-one, one-to-many, many-to-many, polymorphic, JOIN, foreign key, eager loading

---

## 📋 Ringkasan

Data di dunia nyata saling berhubungan — user punya order, order punya item, item merujuk ke produk. Relasi database memungkinkan data dihubungkan secara efisien tanpa duplikasi.

**Target pemahaman:**
- Kamu paham jenis-jenis relasi database
- Kamu bisa mengimplementasikan relasi di migration dan Eloquent
- Kamu paham eager loading untuk menghindari N+1
- Kamu bisa mapping relasi codebase

---

## 1. Relasi Satu-ke-Satu (One-to-One)

### 1.1 Konsep

Satu record di table A berhubungan dengan **satu** record di table B.

```
users ──────► profiles
1 : 1
Satu user punya SATU profile
```

### 1.2 Database Schema

```php
// create_users_table.php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('email')->unique();
    $table->string('password');
    $table->timestamps();
});

// create_profiles_table.php
Schema::create('profiles', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')
          ->unique()           // one-to-one: UNIQUE!
          ->constrained()
          ->onDelete('cascade');
    $table->string('full_name');
    $table->string('phone')->nullable();
    $table->date('birth_date')->nullable();
    $table->timestamps();
});
```

### 1.3 Eloquent

```php
// User model
public function profile(): HasOne
{
    return $this->hasOne(Profile::class);
}

// Profile model
public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}

// Pemakaian:
$user = User::with('profile')->find(1);
$user->profile->full_name; // "Budi Santoso"

$profile = Profile::with('user')->find(1);
$profile->user->email; // "budi@example.com"
```

### 1.4 Di Codebase

```php
// Tidak ada one-to-one eksplisit di codebase
// Tapi pattern seperti User → Profile bisa ditambahkan
```

---

## 2. Relasi Satu-ke-Banyak (One-to-Many)

### 2.1 Konsep

Satu record di table A berhubungan dengan **banyak** record di table B.

```
categories ──────► products
1 : N
Satu kategori punya BANYAK produk
```

### 2.2 Database Schema

```php
// create_categories_table.php
Schema::create('categories', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('slug')->unique();
    $table->boolean('is_active')->default(true);
    $table->timestamps();
});

// create_products_table.php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->foreignId('category_id')  // FK di sini!
          ->constrained()
          ->onDelete('cascade');
    $table->integer('price');
    // ...
});
```

### 2.3 Eloquent

```php
// Category model — satu kategori punya banyak produk
public function products(): HasMany
{
    return $this->hasMany(Product::class);
}

// Product model — produk milik satu kategori
public function category(): BelongsTo
{
    return $this->belongsTo(Category::class);
}

// Pemakaian:
$category = Category::with('products')->find(1);
foreach ($category->products as $product) {
    echo $product->name;
}

$product = Product::with('category')->find(1);
echo $product->category->name; // "Elektronik"
```

### 2.4 SQL JOIN

```sql
-- Ambil produk dengan nama kategori
SELECT p.*, c.name AS category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.id;

-- Atau ambil kategori dengan jumlah produk
SELECT c.*, COUNT(p.id) AS product_count
FROM categories c
LEFT JOIN products p ON c.id = p.category_id
GROUP BY c.id, c.name;
```

### 2.5 Di Codebase

```php
// Product → category
public function category(): BelongsTo
{
    return $this->belongsTo(Category::class);
}

// Category → products (HasMany)
public function products(): HasMany
{
    return $this->hasMany(Product::class);
}

// Order → items
public function items(): HasMany
{
    return $this->hasMany(OrderItem::class);
}

// OrderItem → order
public function order(): BelongsTo
{
    return $this->belongsTo(Order::class);
}
```

---

## 3. Relasi Banyak-ke-Banyak (Many-to-Many)

### 3.1 Konsep

Satu record di table A berhubungan dengan **banyak** record di table B, dan sebaliknya.

```
products ────► product_tag ◄──── tags
N : M
Satu produk punya BANYAK tag
Satu tag dipakai di BANYAK produk
```

Butuh **pivot table** (`product_tag`) untuk menyimpan relasi.

### 3.2 Database Schema

```php
// create_tags_table.php
Schema::create('tags', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('slug')->unique();
    $table->timestamps();
});

// Pivot table: product_tag
Schema::create('product_tag', function (Blueprint $table) {
    $table->id();
    $table->foreignId('product_id')->constrained()->onDelete('cascade');
    $table->foreignId('tag_id')->constrained()->onDelete('cascade');
    $table->timestamps();

    // Unique pair — tidak boleh duplikat
    $table->unique(['product_id', 'tag_id']);
});
```

### 3.3 Eloquent

```php
// Product model
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class)
                ->withTimestamps(); // pivot pakai timestamps
}

// Tag model
public function products(): BelongsToMany
{
    return $this->belongsToMany(Product::class);
}

// Pemakaian:
$product = Product::with('tags')->find(1);
foreach ($product->tags as $tag) {
    echo $tag->name; // "elektronik", "gaming", "laptop"
}

// Attach / detach:
$product->tags()->attach([1, 3, 5]);        // tambah
$product->tags()->sync([1, 2, 3]);          // sinkronisasi (hapus yang tidak ada)
$product->tags()->detach(3);                // hapus satu
$product->tags()->detach();                 // hapus semua
```

### 3.4 SQL JOIN Many-to-Many

```sql
-- Ambil semua tag untuk produk id=1
SELECT t.*
FROM tags t
INNER JOIN product_tag pt ON t.id = pt.tag_id
WHERE pt.product_id = 1;
```

### 3.5 Di Codebase

```php
// Contoh (jika ada di codebase):
// Product → tags (many-to-many)
// User → roles (many-to-many via role_user)
public function roles(): BelongsToMany
{
    return $this->belongsToMany(Role::class);
}
```

---

## 4. Polymorphic Relationship

### 4.1 Konsep

Satu table berelasi dengan **banyak table lain**.

```
comments
├── commentable_type = 'App\Models\Product'
│   └── commentable_id = 5  → produk id 5
└── commentable_type = 'App\Models\Post'
    └── commentable_id = 2  → post id 2
```

### 4.2 Database Schema

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->morphs('commentable'); // commentable_type (string) + commentable_id (BIGINT)
    $table->text('body');
    $table->timestamps();
});
```

### 4.3 Eloquent

```php
// Comment model
public function commentable(): MorphTo
{
    return $this->morphTo();
}

// Product model
public function comments(): MorphMany
{
    return $this->morphMany(Comment::class, 'commentable');
}

// Post model
public function comments(): MorphMany
{
    return $this->morphMany(Comment::class, 'commentable');
}

// Pemakaian:
$product = Product::find(1);
foreach ($product->comments as $comment) {
    echo $comment->body;
}

$comment = Comment::find(1);
$commentable = $comment->commentable; // Product atau Post (otomatis)
```

### 4.4 Polymorphic di Codebase

```php
// Laravel medialibrary — gambar produk
// media table: model_type = 'App\Models\Product', model_id = 5

// Atau jika ada notifications table:
// notifiable_type = 'App\Models\User', notifiable_id = 1
```

---

## 5. HasManyThrough — Relasi Bertingkat

### 5.1 Konsep

Akses data dari table yang "jauh" melalui table perantara.

```
countries ────► users ────► orders
  1 : N          1 : N

$country->orders → ambil semua order dari user di country itu
```

### 5.2 Eloquent

```php
// Country model
public function orders(): HasManyThrough
{
    return $this->hasManyThrough(
        Order::class,
        User::class,
        'country_id', // FK di users
        'user_id',    // FK di orders
        'id',         // PK di countries
        'id'          // PK di users
    );
}

// Pemakaian:
$country = Country::find(1);
foreach ($country->orders as $order) {
    echo $order->total_amount;
}
```

---

## 6. Eager Loading vs Lazy Loading

### 6.1 N+1 Problem (Lazy Loading)

```php
// ❌ N+1 — lazy loading
$products = Product::all();          // 1 query
foreach ($products as $product) {
    echo $product->category->name;   // N query!
}
// Total: 1 + N query
```

### 6.2 Eager Loading

```php
// ✅ Eager loading — dengan()
$products = Product::with('category')->all(); // 2 query
foreach ($products as $product) {
    echo $product->category->name;   // sudah di-load!
}
// Total: 2 query

// Multiple relationships:
Product::with(['category', 'brand', 'tags'])->get();

// Nested:
Order::with('items.product')->get();
```

### 6.3 Lazy Eager Loading

```php
// Kalau terlanjur query tanpa with:
$products = Product::all();
if (auth()->user()->isAdmin()) {
    $products->load('category', 'brand'); // lazy load
}
```

---

## 7. Mapping Relasi Codebase

```php
// Product
public function category(): BelongsTo
public function brand(): BelongsTo
public function orderItems(): HasMany

// Category
public function products(): HasMany
public function parent(): BelongsTo
public function children(): HasMany

// Order
public function user(): BelongsTo
public function items(): HasMany
public function payment(): HasOne

// OrderItem
public function order(): BelongsTo
public function product(): BelongsTo

// User
public function orders(): HasMany
public function roles(): BelongsToMany
```

---

## 8. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ RELATIONSHIP TYPES                                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  One-to-One:    user ────── profile                                │
│                 hasOne / belongsTo                                  │
│                                                                     │
│  One-to-Many:   category ──► products                              │
│                 hasMany / belongsTo                                 │
│                                                                     │
│  Many-to-Many:  products ──► product_tag ◄── tags                │
│                 belongsToMany / belongsToMany                       │
│                 (pivot table)                                       │
│                                                                     │
│  Polymorphic:   comments ──► (products OR posts)                   │
│                 morphMany / morphTo                                 │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  Product → category (BelongsTo)                                    │
│  Product → brand (BelongsTo)                                       │
│  Product → orderItems (HasMany)                                    │
│  Order → items (HasMany)                                           │
│  Order → payment (HasOne)                                          │
│  User → orders (HasMany)                                           │
│  User → roles (BelongsToMany)                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Cek relasi Product.** Buka `app/Models/Product.php`. Semua method relationship — apa saja? Tulis tipe relasinya.

2. **Cek relasi Order.** Buka `app/Models/Order.php`. Relationship apa yang ada?

3. **Eager loading.** Cek `ShopController@index`. Apakah pakai `with()`? Jika tidak, coba tambahkan `with(['category', 'brand'])`.

4. **SQL JOIN test.** Di phpMyAdmin, jalankan:
   ```sql
   SELECT p.name, c.name AS category, b.name AS brand
   FROM products p
   LEFT JOIN categories c ON p.category_id = c.id
   LEFT JOIN brands b ON p.brand_id = b.id;
   ```

5. **N+1 test.** Buka Telescope → Queries. Buka halaman shop. Apakah terlihat N+1 problem? (query berulang untuk relasi)

---

## 🔗 Referensi

- [Laravel Docs: Eloquent Relationships](https://laravel.com/docs/11.x/eloquent-relationships)
- [Laravel Docs: Eager Loading](https://laravel.com/docs/11.x/eloquent-relationships#eager-loading)
- [MySQL JOIN Tutorial](https://www.mysqltutorial.org/mysql-join/)
- Codebase: `app/Models/` — semua relationship
- Codebase: Telescope → Queries → cek N+1
