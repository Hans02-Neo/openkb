# 07-04: ERD dan Schema Design — Merancang Database

> **Fase**: 7 — Database & SQL  
> **Prasyarat**: 07-03-relationship-dan-join  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: ERD, entity, attribute, normalization, denormalization, schema design, cardinality, migration design

---

## 📋 Ringkasan

Sebelum membuat migration, kamu perlu **merancang** database — entity apa saja, hubungannya bagaimana, kolom apa yang diperlukan. ERD (Entity Relationship Diagram) adalah alat untuk merancang database.

**Target pemahaman:**
- Kamu bisa membaca dan membuat ERD
- Kamu paham normalisasi database
- Kamu bisa mendesain schema dari requirement bisnis
- Kamu bisa membaca ERD dari migration yang ada

---

## 1. ERD — Entity Relationship Diagram

### 1.1 Komponen ERD

```
┌─────────────────┐
│    PRODUCT      │  ← ENTITY (table)
├─────────────────┤
│ id (PK)         │  ← ATTRIBUTE (column)
│ name            │
│ slug            │
│ price           │
│ category_id (FK)│
│ created_at      │
└────────┬────────┘
         │
         │  (N,1)
         │
┌────────▼────────┐
│   CATEGORY      │
├─────────────────┤
│ id (PK)         │
│ name            │
│ slug            │
└─────────────────┘
```

### 1.2 Entity

Entity = table di database. Setiap entity mewakili **objek di dunia nyata**:

| Entity | Contoh Row | 
|--------|-----------|
| Product | Laptop Gaming |
| Category | Elektronik |
| Order | ORD-001 |
| User | budi@example.com |
| Payment | TRX-001 |

### 1.3 Attribute (Kolom)

Attribute = kolom di table. Ada beberapa jenis:

| Jenis | Contoh | Keterangan |
|-------|--------|------------|
| **Primary Key** | `id` | Unik, identitas row |
| **Foreign Key** | `category_id` | Merujuk ke table lain |
| **Regular** | `name`, `price` | Data biasa |
| **Timestamp** | `created_at` | Waktu dibuat/diubah |
| **Soft Delete** | `deleted_at` | Untuk soft delete |

### 1.4 Cardinality — Jumlah Relasi

```
───||──  (one)    satu
───|──   (many)   banyak

[categories] ──|───────||── [products]
    1                 N
Satu kategori punya banyak produk
Satu produk milik satu kategori
```

**Simbol Cardinality:**
```
One ──||──
Many ──|──
One (and only one) ──||──
Zero or more ─────────
```

---

## 2. Normalization — Mengurangi Duplikasi

### 2.1 Masalah Tanpa Normalisasi

```sql
-- ❌ UNNORMALIZED — satu table untuk semua
CREATE TABLE orders_flat (
    id INT PRIMARY KEY,
    customer_name VARCHAR(255),
    customer_email VARCHAR(255),
    product_name VARCHAR(255),
    product_price INT,
    quantity INT,
    order_date DATETIME
);

INSERT INTO orders_flat VALUES
(1, 'Budi', 'budi@email.com', 'Laptop', 15000000, 1, '2024-01-01'),
(2, 'Budi', 'budi@email.com', 'Mouse', 250000, 2, '2024-01-01');
--        ↑ DUPLIKASI! Nama dan email Budi diulang
```

**Masalah:**
- **Redundansi** — data diulang
- **Anomali update** — ubah email Budi → harus ganti di semua baris
- **Anomali delete** — hapus order → kehilangan data customer

### 2.2 Normalized (3NF)

```sql
-- ✅ NORMALIZED — data dipisah per entity
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255) UNIQUE
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    order_date DATETIME
);

CREATE TABLE order_items (
    id INT PRIMARY KEY,
    order_id INT REFERENCES orders(id),
    product_id INT REFERENCES products(id),
    quantity INT,
    price INT
);

-- Data Budi cukup 1x di customers
-- Budi bisa punya banyak order
-- Setiap order bisa punya banyak item
```

### 2.3 Bentuk Normalisasi

| Bentuk | Aturan | Dilanggar Jika |
|--------|--------|---------------|
| **1NF** | Setiap kolom berisi satu nilai | Kolom `tags` = "elektronik,gaming,laptop" |
| **2NF** | 1NF + setiap kolom non-PK bergantung penuh pada PK | `product_name` di tabel order_items (bergantung pada product_id, bukan order_id) |
| **3NF** | 2NF + tidak ada transitive dependency | `customer_email` di tabel orders (bisa dari customers) |

### 2.4 Kapan Normalisasi Tidak Perlu?

```php
// JSON column — untuk data yang tidak perlu di-query terpisah
$table->json('metadata');
// Isi: {"color": "red", "size": "XL", "material": "cotton"}
// Tidak perlu table terpisah untuk metadata produk

// Denormalization — untuk performa
// Simpan total_amount di orders walau bisa dihitung dari items
// Karena query "total semua order" jadi lebih cepat
$table->integer('total_amount'); // denormalized — tapi untuk performa
```

---

## 3. Schema Design dari Use Case

### 3.1 Contoh: Fitur Reviews

**Requirement:**
- User bisa mereview produk
- Review punya rating (1-5) dan komentar
- Satu user hanya bisa review sekali per produk

**ERD:**
```
users ──||───────|── reviews ──|───────||── products
           1              N           1

review: user_id (FK) + product_id (FK) → UNIQUE pair
```

**Migration:**
```php
Schema::create('reviews', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->foreignId('product_id')->constrained()->onDelete('cascade');
    $table->tinyInteger('rating'); // 1-5
    $table->text('comment')->nullable();
    $table->timestamps();

    $table->unique(['user_id', 'product_id']); // satu review per user per produk
});
```

**Eloquent:**
```php
// User model
public function reviews(): HasMany

// Product model
public function reviews(): HasMany

// Review model
public function user(): BelongsTo
public function product(): BelongsTo
```

### 3.2 Contoh: Fitur Wishlist

**Requirement:**
- User bisa menyimpan produk favorit
- Bisa banyak produk
- Bisa dihapus kapan saja

**ERD:**
```
users ──||───────|── wishlists ──|───────||── products
           1              N           1
```

**Migration:**
```php
Schema::create('wishlists', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->foreignId('product_id')->constrained()->onDelete('cascade');
    $table->timestamps();

    $table->unique(['user_id', 'product_id']);
});
```

---

## 4. ERD Codebase

### 4.1 Entity dan Relasi

```
┌─────────────────────────────────────────────────────────────────┐
│                       KONEKSI STORE ERD                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐     ┌──────────┐     ┌──────────────┐           │
│  │  USERS   │────►│  ORDERS  │────►│  ORDER_ITEMS │────►PRODUCTS│
│  └──────────┘     └──────────┘     └──────────────┘           │
│       │                                                        │
│       │              ┌──────────────┐                          │
│       └─────────────►│  CUSTOMERS   │                          │
│                      └──────────────┘                          │
│                                                                 │
│  ┌──────────┐     ┌──────────┐                                  │
│  │CATEGORIES│────►│ PRODUCTS │──►┌────────┐                    │
│  └──────────┘     └──────────┘   │ BRANDS │                    │
│                                  └────────┘                    │
│                                                                 │
│  ┌──────────┐     ┌──────────┐                                  │
│  │  ORDERS  │────►│ PAYMENTS │                                  │
│  └──────────┘     └──────────┘                                  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ MEDIA (spatie/laravel-medialibrary) — polymorphic        │  │
│  │ model_type = Product, model_id = product_id             │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Tabel Lengkap

| Table | PK | FKs | Relasi |
|-------|----|-----|--------|
| `users` | id | — | — |
| `customers` | id | user_id → users |
| `categories` | id | parent_id → categories | Self-referential |
| `brands` | id | — | — |
| `products` | id | category_id, brand_id | BelongsTo both |
| `orders` | id | user_id → users | BelongsTo user |
| `order_items` | id | order_id, product_id | BelongsTo both |
| `payments` | id | order_id → orders | HasOne order |
| `settings` | id | — | Key-value |
| `media` | id | model_id, model_type | Polymorphic |
| `notifications` | id | notifiable_id, notifiable_type | Polymorphic |
| `jobs` | id | — | Queue |
| `failed_jobs` | id | — | Failed queue |
| `sessions` | id | — | Session |

---

## 5. Best Practices Schema Design

### 5.1 Naming Convention

```php
// Table: plural snake_case
products, order_items, product_category

// PK: selalu 'id'
$table->id();

// FK: singular + _id
$table->foreignId('category_id');

// Pivot table: singular + singular (alphabetical)
product_tag, order_product

// Timestamps
$table->timestamps();      // created_at, updated_at
$table->softDeletes();     // deleted_at
```

### 5.2 Migration Rules

```php
// 1. Setiap table harus punya id
$table->id();

// 2. Setiap relasi harus foreign key
$table->foreignId('category_id')->constrained();

// 3. Pilih onDelete yang tepat
->onDelete('cascade')   // parent hapus → child ikut hapus
->onDelete('set null')  // parent hapus → FK jadi null
->onDelete('restrict')  // cegah hapus jika masih ada child

// 4. Tambah index untuk kolom yang sering di-query
$table->index('category_id');
$table->index('is_active');
```

### 5.3 Kapan Perlu Table Baru?

```
❌ Jangan buat kolom untuk data yang bisa banyak:
  product_color_1, product_color_2, product_color_3
✅ Buat table terpisah:
  product_colors: product_id, color

❌ Jangan simpan data yang bisa dihitung:
  order_items_count di orders
✅ Hitung via query:
  Order::withCount('items')

❌ Jangan simpan data yang bisa diambil dari relasi:
  customer_name di orders
✅ Ambil via relasi:
  $order->customer->name
```

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ ERD COMPONENTS                                                     │
│  Entity  → Table (Product, Order, User)                           │
│  Attribute → Column (name, price, email)                          │
│  Relationship → FK (category_id → categories.id)                  │
│  Cardinality → 1:1, 1:N, N:M                                      │
├─────────────────────────────────────────────────────────────────────┤
│ NORMALIZATION                                                      │
│  1NF: Satu nilai per kolom                                        │
│  2NF: Bergantung penuh pada PK                                    │
│  3NF: Tidak ada transitive dependency                             │
├─────────────────────────────────────────────────────────────────────┤
│ SCHEMA DESIGN FLOW                                                 │
│  Requirements → Entities → Relationships → Attributes → Migration │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  ~10 entities (users, products, categories, orders, dll)          │
│  Relasi: one-to-many (category→products), belongsTo (product→cat) │
│  Polymorphic: media (spatie)                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Gambar ERD.** Dari migration yang ada, gambar ERD untuk:
   - Product, Category, Brand
   - Order, OrderItem, Product

2. **Identifikasi cardinality.** Dari codebase, tentukan cardinality:
   - User → Orders
   - Orders → OrderItems
   - Products → Category

3. **Desain schema.** Desain schema untuk fitur "Diskon":
   - Produk bisa punya diskon (percentase atau nominal)
   - Diskon punya periode (start_date, end_date)
   - Satu produk bisa punya banyak diskon (tapi hanya satu aktif)

4. **Cari denormalisasi.** Cari kolom di codebase yang sebenarnya bisa dihitung dari relasi (misal `total_amount` di orders). Apakah itu denormalisasi? Kenapa ada di sana?

5. **Migration review.** Buka `database/migrations/`:
   - Apakah semua table punya `id`?
   - Apakah semua FK pakai `constrained()`?
   - Apakah ada `onDelete` yang tepat?

---

## 🔗 Referensi

- [LucidChart: ERD Guide](https://www.lucidchart.com/pages/er-diagrams)
- [Database Normalization](https://www.guru99.com/database-normalization.html)
- [Laravel: Migrations](https://laravel.com/docs/11.x/migrations)
- Codebase: `database/migrations/` — semua migration
