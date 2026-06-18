# 07-07: Database di Codebase Ini — Mapping Lengkap

> **Fase**: 7 — Database & SQL  
> **Prasyarat**: 07-06-n-plus-one-problem  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: codebase mapping, migration analysis, query analysis, schema, optimization, telescope

---

## 📋 Ringkasan

Dokumen terakhir FASE-07 ini **merangkum semua** konsep database yang sudah dipelajari dan **memetakannya ke codebase nyata** — migration, model, controller, query, dan optimasi.

**Target pemahaman:**
- Kamu bisa membaca dan memahami seluruh schema database
- Kamu bisa melacak alur data dari request ke database
- Kamu bisa mengidentifikasi area optimasi
- Kamu punya gambaran lengkap database codebase

---

## 1. Schema Lengkap

### 1.1 Daftar Semua Table

```bash
# Dari migration:
Database: koneksi_store
Tables:
├── users              # User authentication (Laravel Breeze)
├── password_reset_tokens
├── sessions
├── personal_access_tokens  # Sanctum token
├── customers          # Data pelanggan
├── categories         # Kategori produk (hierarki)
├── brands             # Merek produk
├── products           # Produk
├── orders             # Pesanan
├── order_items        # Item dalam pesanan
├── payments           # Pembayaran (Midtrans)
├── settings           # Konfigurasi aplikasi
├── media              # Media library (spatie)
├── notifications      # Database notifications
├── jobs               # Queue jobs
├── failed_jobs        # Failed queue jobs
└── migrations         # Riwayat migration (built-in)
```

### 1.2 Struktur Detail

```sql
-- products
┌────────────────┬──────────────┬──────────────────────────────┐
│ Column         │ Type         │ Notes                        │
├────────────────┼──────────────┼──────────────────────────────┤
│ id             │ BIGINT PK    │ Auto increment               │
│ name           │ VARCHAR(255) │ Nama produk                  │
│ slug           │ VARCHAR(255) │ UNIQUE — untuk URL           │
│ short_desc     │ TEXT?        │ Deskripsi singkat            │
│ description    │ TEXT?        │ Deskripsi lengkap            │
│ price          │ INT          │ Harga (dalam rupiah)         │
│ stock_quantity │ INT          │ Stok                         │
│ weight_grams   │ INT          │ Berat (gram)                 │
│ is_active      │ BOOLEAN      │ Aktif/tidak                  │
│ category_id    │ BIGINT FK    │ → categories.id              │
│ brand_id       │ BIGINT FK?   │ → brands.id                 │
│ meta_title     │ VARCHAR?     │ SEO                          │
│ meta_desc      │ TEXT?        │ SEO                          │
│ created_at     │ TIMESTAMP    │                              │
│ updated_at     │ TIMESTAMP    │                              │
│ deleted_at     │ TIMESTAMP?   │ Soft delete                  │
└────────────────┴──────────────┴──────────────────────────────┘

-- orders
┌──────────────────┬──────────────┬──────────────────────────────┐
│ Column           │ Type         │ Notes                        │
├──────────────────┼──────────────┼──────────────────────────────┤
│ id               │ BIGINT PK    │                              │
│ user_id          │ BIGINT FK    │ → users.id                  │
│ order_number     │ VARCHAR(255) │ UNIQUE — ORD-xxx            │
│ total_amount     │ INT          │ Total + ongkir              │
│ shipping_cost    │ INT          │ Biaya kirim                 │
│ status           │ VARCHAR(50)  │ pending/paid/processing/... │
│ shipping_addr    │ TEXT         │ Alamat kirim                │
│ shipping_city    │ VARCHAR      │ Kota                        │
│ shipping_postal  │ VARCHAR      │ Kode pos                    │
│ shipping_phone   │ VARCHAR      │ No. HP                      │
│ created_at       │ TIMESTAMP    │                              │
│ updated_at       │ TIMESTAMP    │                              │
└──────────────────┴──────────────┴──────────────────────────────┘

-- order_items
┌──────────────┬──────────┬──────────────────────────────────┐
│ Column       │ Type     │ Notes                            │
├──────────────┼──────────┼──────────────────────────────────┤
│ id           │ BIGINT PK│                                  │
│ order_id     │ BIGINT FK│ → orders.id                     │
│ product_id   │ BIGINT FK│ → products.id                   │
│ product_name │ VARCHAR  │ Denormalized — snapshot nama     │
│ quantity     │ INT      │ Jumlah                           │
│ price        │ INT      │ Harga per item (snapshot)        │
│ subtotal     │ INT      │ price * quantity                 │
│ created_at   │ TIMESTAMP│                                  │
│ updated_at   │ TIMESTAMP│                                  │
└──────────────┴──────────┴──────────────────────────────────┘
```

---

## 2. Alur Data dari Request ke Database

### 2.1 Contoh: Halaman Shop

```
Browser: GET /shop?category=laptop
                    │
Route: GET /shop → ShopController@index
                    │
Controller:
  Product::with(['category', 'brand'])
    ->where('is_active', true)
    ->when($request->category, fn($q) =>
        $q->whereHas('category', fn($q) =>
            $q->where('slug', $request->category)
        )
    )
    ->paginate(12)
                    │
SQL yang dihasilkan:
  SELECT COUNT(*) FROM products
    INNER JOIN categories ON products.category_id = categories.id
    WHERE products.is_active = 1
    AND categories.slug = 'laptop'

  SELECT * FROM products
    INNER JOIN categories ON products.category_id = categories.id
    WHERE products.is_active = 1
    AND categories.slug = 'laptop'
    LIMIT 12 OFFSET 0

  SELECT * FROM categories WHERE id IN (1, 2, ...)
  SELECT * FROM brands WHERE id IN (1, 2, ...)
                    │
View: shop/index.blade.php
  @foreach($products as $product)
    {{ $product->name }}
    {{ $product->category->name }}  ← sudah eager-loaded
  @endforeach
```

### 2.2 Contoh: Checkout

```
Browser: POST /checkout (form data)
                    │
Controller: CheckoutController@process
  $request->validated()  ← FormRequest
                    │
OrderService::createOrder()
                    │
DB::transaction(function () {
  1. CartService::getCart()         ← session
  2. Validasi stok                   ← SELECT products
  3. Order::create(...)             ← INSERT orders
  4. foreach cart:
       OrderItem::create(...)       ← INSERT order_items
       Product::decrement(...)      ← UPDATE products
  5. MidtransService::process(...)  ← INSERT/UPDATE payments
  6. CartService::clear()           ← session
  7. dispatch job                    ← INSERT jobs
})
                    │
Return: redirect()->route('orders.show', $order)
```

---

## 3. Query Analysis via Telescope

### 3.1 Membuka Telescope

```
http://olshop-koneksi.test/telescope/queries
```

### 3.2 Yang Perlu Diperhatikan

| Indikator | Arti | Tindakan |
|-----------|------|----------|
| Query > 100ms | Lambat | Tambah index |
| Query berulang | N+1? | Tambah eager loading |
| `SELECT *` | Ambil semua kolom | Pilih kolom tertentu |
| `type: ALL` | Full table scan | Tambah index |
| `Using filesort` | Sorting tanpa index | Tambah index untuk kolom ORDER BY |

### 3.3 Contoh Analisis

```sql
-- Query dari Telescope:
SELECT * FROM `products`
WHERE `is_active` = ?
ORDER BY `created_at` DESC
LIMIT 12 OFFSET 0
-- Time: 15ms ✅ OK
-- Rows: 100000 (masih full scan?)

SELECT * FROM `products`
INNER JOIN `categories` ON `products`.`category_id` = `categories`.`id`
WHERE `categories`.`slug` = ?
-- Time: 3ms ✅ Cepat (pakai index)
```

---

## 4. Performance Audit

### 4.1 Current State

| Area | Status | Catatan |
|------|--------|---------|
| Eager loading `with()` | ✅ Done | ShopController pakai with() |
| Pagination | ✅ | paginate(12) |
| Index | ⚠️ Mungkin kurang | Cek index untuk is_active, price |
| N+1 | ✅ Tidak ada | Verified via Telescope |
| Query optimization | ⚠️ | Bisa dioptimasi lebih lanjut |

### 4.2 Rekomendasi Index

```php
// Migration baru untuk optimasi:
Schema::table('products', function (Blueprint $table) {
    $table->index('is_active');        // Sering di-where
    $table->index('price');            // Sering di-sort/filter
    $table->index(['is_active', 'category_id']); // Composite — filter umum
});

Schema::table('orders', function (Blueprint $table) {
    $table->index('status');           // Sering di-filter
    $table->index('user_id');          // FK — sudah ada
    $table->index('created_at');       // Sering di-order
});
```

### 4.3 Rekomendasi Query

```php
// 1. Select kolom tertentu — jangan SELECT *
$products = Product::select('id', 'name', 'price', 'slug')
    ->with('category:id,name')
    ->paginate(12);

// 2. whereHas → use join for performance
// Instead of:
Product::whereHas('category', fn($q) => $q->where('slug', $slug));

// Consider:
Product::join('categories', 'products.category_id', '=', 'categories.id')
    ->where('categories.slug', $slug)
    ->select('products.*')
    ->paginate(12);

// 3. Cursor for large exports (not web)
// Instead of: Order::all()
// Use: Order::cursor()
```

---

## 5. Database Backup dan Maintenance

### 5.1 Backup via Laragon

Laragon → Database → Backup → Pilih database → Backup Now
File `.sql` disimpan di folder backup.

### 5.2 Export via phpMyAdmin

```
http://localhost/phpmyadmin
→ Pilih database koneksi_store
→ Export
→ Custom: pilih table
→ Go → download .sql
```

### 5.3 Command Line

```bash
# Export database
"C:\laragon\bin\mysql\mysql-8.4.0\bin\mysqldump.exe" -u root koneksi_store > backup.sql

# Import database
"C:\laragon\bin\mysql\mysql-8.4.0\bin\mysql.exe" -u root koneksi_store < backup.sql
```

---

## 6. Schema Diagram Lengkap

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       KONEKSI STORE — DATABASE SCHEMA                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌────────────────┐        ┌──────────────────┐        ┌────────────────┐  │
│  │     users      │        │    customers      │        │  categories    │  │
│  ├────────────────┤        ├──────────────────┤        ├────────────────┤  │
│  │ id             │        │ id               │        │ id             │  │
│  │ name           │        │ user_id (FK)─────┼──┐     │ name           │  │
│  │ email (U)      │        │ phone            │  │     │ slug (U)       │  │
│  │ password       │        │ address          │  │     │ parent_id (FK)─┼──┐
│  │ role           │        │ city             │  │     │ is_active      │  │
│  │ created_at     │        │ created_at       │  │     │ sort_order     │  │
│  └───────┬────────┘        └──────────────────┘  │     │ created_at     │  │
│          │                                        │     └───────┬────────┘  │
│          │                                        │             │           │
│          ▼                                        │             │           │
│  ┌──────────────────┐      ┌──────────────────┐   │             │           │
│  │     orders       │      │    payments      │   │             │           │
│  ├──────────────────┤      ├──────────────────┤   │             │           │
│  │ id               │      │ id               │   │             │           │
│  │ user_id (FK)─────┼──┐   │ order_id (FK)────┼───┼──────┐      │           │
│  │ order_number (U) │  │   │ transaction_id   │   │      │      │           │
│  │ total_amount     │  │   │ status           │   │      │      │           │
│  │ shipping_cost    │  │   │ payment_method   │   │      │      │           │
│  │ status           │  │   │ raw_response     │   │      │      │           │
│  │ shipping_address │  │   │ created_at       │   │      │      │           │
│  │ created_at       │  │   └──────────────────┘   │      │      │           │
│  └───────┬──────────┘  │                          │      │      │           │
│          │             │                          │      │      │           │
│          ▼             │                          │      │      │           │
│  ┌──────────────────┐  │                          │      │      │           │
│  │   order_items    │  │                          │      │      │           │
│  ├──────────────────┤  │                          │      │      │           │
│  │ id               │  │                          │      │      │           │
│  │ order_id (FK)────┼──┘                          │      │      │           │
│  │ product_id (FK)──┼──┐                          │      │      │           │
│  │ product_name     │  │                          │      │      │           │
│  │ quantity         │  │                          │      │      │           │
│  │ price            │  │     ┌────────────────┐   │      │      │           │
│  │ subtotal         │  │     │    brands      │   │      │      │           │
│  │ created_at       │  │     ├────────────────┤   │      │      │           │
│  └──────────────────┘  │     │ id             │   │      │      │           │
│                        │     │ name           │   │      │      │           │
│                        │     │ slug (U)       │   │      │      │           │
│                        ▼     │ is_active      │   │      │      │           │
│  ┌──────────────────┐        │ created_at     │   │      │      │           │
│  │    products      │        └───────┬────────┘   │      │      │           │
│  ├──────────────────┤               │            │      │      │           │
│  │ id               │               │            │      │      │           │
│  │ name             │               │            │      │      │           │
│  │ slug (U)         │               │            │      │      │           │
│  │ price            │               │            │      │      │           │
│  │ stock_quantity   │               │            │      │      │           │
│  │ is_active        │               │            │      │      │           │
│  │ category_id (FK)─┼───────────────┼────────────┼──────┘      │           │
│  │ brand_id (FK)────┼───────────────┘            │             │           │
│  │ weight_grams     │                            │             │           │
│  │ created_at       │                            │             │           │
│  │ deleted_at       │                            │             │           │
│  └──────────────────┘                            │             │           │
│                                                  │             │           │
│  ┌──────────────────┐      ┌──────────────────┐  │             │           │
│  │     media        │      │  notifications   │  │             │           │
│  ├──────────────────┤      ├──────────────────┤  │             │           │
│  │ id               │      │ id               │  │             │           │
│  │ model_type       │      │ type             │  │             │           │
│  │ model_id         │      │ notifiable_type  │  │             │           │
│  │ collection_name  │      │ notifiable_id    │  │             │           │
│  │ file_name        │      │ data (JSON)      │  │             │           │
│  │ mime_type        │      │ read_at          │  │             │           │
│  │ disk             │      │ created_at       │  │             │           │
│  │ created_at       │      └──────────────────┘  │             │           │
│  └──────────────────┘                            │             │           │
│                                                  │             │           │
│  ┌──────────────────┐      ┌──────────────────┐  │             │           │
│  │     settings     │      │      jobs        │  │             │           │
│  ├──────────────────┤      ├──────────────────┤  │             │           │
│  │ id               │      │ id               │  │             │           │
│  │ key (U)          │      │ queue            │  │             │           │
│  │ value            │      │ payload (JSON)   │  │             │           │
│  │ created_at       │      │ attempts         │  │             │           │
│  └──────────────────┘      │ reserved_at      │  │             │           │
│                            │ available_at     │  │             │           │
│                            │ created_at       │  │             │           │
│                            └──────────────────┘  │             │           │
│                                                  ▼             ▼           │
│                                    ┌──────────────────────────────────┐  │
│                                    │        failed_jobs               │  │
│                                    ├──────────────────────────────────┤  │
│                                    │ id, uuid, connection, queue,    │  │
│                                    │ payload, exception, failed_at   │  │
│                                    └──────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Ringkasan — Apa yang Sudah Dipelajari

| Dokumen | Materi | Praktik di Codebase |
|---------|--------|---------------------|
| 07-01 | Konsep database | MySQL via Laragon, phpMyAdmin |
| 07-02 | SQL CRUD | SELECT/INSERT/UPDATE/DELETE |
| 07-03 | Relationship & JOIN | Eloquent relationships |
| 07-04 | ERD & Schema Design | Migration analysis |
| 07-05 | Index & Performance | EXPLAIN, index optimization |
| 07-06 | N+1 Problem | Eager loading with() |
| 07-07 | Database di Codebase | Full schema mapping |

---

## 🧪 Latihan

1. **Full mapping.** Buka setiap migration. Cocokkan dengan diagram di atas. Apakah ada table/kolom yang kurang?

2. **Telescope audit.** Buka Telescope → Queries. Catat 5 query paling lambat. Apa yang bisa dioptimasi?

3. **Index audit.** Cek index di setiap table via phpMyAdmin atau `SHOW INDEX FROM table`. Apakah ada index yang kurang?

4. **Write query.** Tulis SQL untuk:
   - Top 5 produk terlaris (berdasarkan quantity di order_items)
   - Total revenue per bulan
   - Kategori dengan produk terbanyak

5. **Optimasi saran.** Dari audit di atas, buat daftar 3 optimasi database yang paling berdampak untuk codebase ini.

---

## 🔗 Referensi

- Seluruh dokumen FASE-07 (07-01 sampai 07-06)
- Codebase: `database/migrations/`
- Codebase: `app/Models/`
- Codebase: Telescope → `http://olshop-koneksi.test/telescope`
- phpMyAdmin: `http://localhost/phpmyadmin`
