# 07-05: Index dan Performance — Database Cepat

> **Fase**: 7 — Database & SQL  
> **Prasyarat**: 07-04-erd-dan-schema-design  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: index, EXPLAIN, query performance, slow query, composite index, fulltext search, query optimization

---

## 📋 Ringkasan

Database yang lambat membuat aplikasi lemot. **Index** adalah fitur database yang mempercepat pencarian data — seperti indeks di buku. Tanpa index, database harus membaca SEMUA baris (full table scan).

**Target pemahaman:**
- Kamu paham cara kerja index
- Kamu bisa membuat index yang tepat
- Kamu bisa membaca query plan dengan EXPLAIN
- Kamu paham cara mengoptimasi query lambat

---

## 1. Cara Kerja Index

### 1.1 Full Table Scan (Tanpa Index)

```sql
-- Tanpa index di kolom email:
SELECT * FROM users WHERE email = 'budi@example.com';
-- Database baca SEMUA baris dari baris 1 sampai ketemu
-- 1 juta user → baca 1 juta baris!
```

### 1.2 Dengan Index

```sql
-- Dengan index di kolom email:
SELECT * FROM users WHERE email = 'budi@example.com';
-- Database langsung ke baris yang tepat (via B-tree)
-- 1 juta user → baca ~3-4 level tree
```

### 1.3 B-Tree Index

Index menggunakan struktur B-Tree (Balanced Tree):

```
                        [500]
                       /     \
                 [250]       [750]
                /    \       /    \
            [125]  [375]  [625]  [875]
              ↓
            Cari email = 'budi@example.com'
            Level 1: 500 → 'budi' < 500 → kiri
            Level 2: 250 → 'budi' < 250 → kiri
            Level 3: 125 → ketemu!
            3 langkah, bukan 125 langkah!
```

### 1.4 Visual: Index vs No Index

```
TANPA INDEX (Full Scan):
□ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □ □
→ Baca semua kotak sampai ketemu yang dicari

DENGAN INDEX (B-Tree):
        ┌─────┐
        │ 500 │
     ┌──┴──┬──┴──┐
     │ 250 │ 750 │
   ┌─┴─┐ ┌─┴─┐ ┌─┴─┐
   │125│ │375│ │625│ │875│
   └───┘ └───┘ └───┘ └───┘
→ Langsung ke node yang tepat
```

---

## 2. Kapan Index Diperlukan?

### 2.1 Kolom yang Sering di WHERE

```php
// Kolom ini sering di-where → perlu index
Product::where('is_active', true)->get();
Product::where('slug', $slug)->first();
Product::where('category_id', 1)->get();

// Migration:
$table->index('is_active');
$table->unique('slug'); // UNIQUE = index + constraint
$table->index('category_id');
```

### 2.2 Kolom untuk ORDER BY

```sql
-- ORDER BY kolom yang di-index lebih cepat
SELECT * FROM products ORDER BY created_at DESC;
-- Index di created_at membantu

SELECT * FROM products ORDER BY price ASC;
-- Index di price membantu
```

### 2.3 Foreign Key

```php
// Foreign key OTOMATIS di-index Laravel!
$table->foreignId('category_id')->constrained();
// constrained() otomatis bikin index
```

### 2.4 Kapan TIDAK Perlu Index

```sql
-- ❌ Kolom dengan sedikit nilai unik (low cardinality)
-- Contoh: is_active (hanya 0/1) — index tidak membantu
SELECT * FROM products WHERE is_active = 1;
-- Database tetap baca 50% baris — index tidak berguna

-- ❌ Table kecil (< 1000 baris)
-- Full scan lebih cepat dari index lookup

-- ❌ Kolom yang jarang dipakai untuk WHERE
-- Contoh: updated_at — jarang dicari
```

---

## 3. Membuat Index

### 3.1 Di Migration

```php
// Single column index
$table->index('email');
$table->index('category_id');
$table->index('status');

// Unique index
$table->unique('slug');
$table->unique(['user_id', 'product_id']); // composite unique

// Fulltext index — untuk pencarian teks
$table->fullText('description');

// Index with name
$table->index(['category_id', 'is_active'], 'idx_cat_active');

// Drop index
$table->dropIndex(['email']);    // drop by columns
$table->dropIndex('idx_cat_active'); // drop by name
```

### 3.2 Menambah Index ke Table Existing

```bash
php artisan make:migration add_index_to_products_table --table=products
```

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->index('price');
        $table->index(['category_id', 'is_active']);
    });
}

public function down(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropIndex(['price']);
        $table->dropIndex(['category_id', 'is_active']);
    });
}
```

### 3.3 Composite Index

Index pada **lebih dari satu kolom**:

```php
$table->index(['category_id', 'is_active', 'price']);
```

```sql
-- Composite index berguna untuk query ini:
SELECT * FROM products
WHERE category_id = 1
  AND is_active = 1
  AND price > 10000;

-- TAPI TIDAK berguna untuk:
SELECT * FROM products
WHERE is_active = 1; -- kolom pertama bukan category_id
```

**Urutan kolom penting!** Kolom yang paling sering di-filter harus pertama.

---

## 4. EXPLAIN — Melihat Query Plan

### 4.1 Menggunakan EXPLAIN

```sql
EXPLAIN SELECT * FROM products WHERE category_id = 1;
```

Hasil:
```
┌───────┬──────────┬────────┬──────────┬────────┐
│ type  │ rows     │ Extra  │ key      │ key_len│
├───────┼──────────┼────────┼──────────┼────────┤
│ ref   │ 150      │ NULL   │ category │ 8      │
└───────┴──────────┴────────┴──────────┴────────┘
```

**Kolom penting:**
| Kolom | Arti | Bagus | Jelek |
|-------|------|-------|-------|
| `type` | Tipe akses | `const`, `ref`, `range` | `ALL` (full scan) |
| `rows` | Perkiraan baris dibaca | Kecil | Besar |
| `key` | Index yang dipakai | Nama index | `NULL` (tanpa index) |
| `Extra` | Info tambahan | `Using index` | `Using filesort` |

### 4.2 Yang Perlu Diwaspadai

```sql
-- ❌ ALL = full table scan (tanpa index)
EXPLAIN SELECT * FROM products WHERE name LIKE '%laptop%';
-- type: ALL, rows: 100000 → baca semua baris!

-- ✅ ref = menggunakan index non-unique
EXPLAIN SELECT * FROM products WHERE category_id = 1;
-- type: ref, rows: 150 → hanya 150 baris

-- ✅ const = menggunakan primary key / unique index
EXPLAIN SELECT * FROM products WHERE id = 1;
-- type: const, rows: 1 → langsung!
```

### 4.3 EXPLAIN via Laravel

```php
// Tidak bisa langsung, tapi bisa lihat di Telescope
// Buka: http://olshop-koneksi.test/telescope/queries
// Klik query → lihat timing dan penjelasan
```

---

## 5. Slow Query Optimization

### 5.1 Query Lambat Umum

```php
// ❌ LIKE dengan wildcard di AWAL — tidak bisa pakai index!
Product::where('name', 'like', '%laptop%')->get();
// Solution: Fulltext index → whereFullText()

// ❌ Function di kolom — index tidak berguna!
Product::whereRaw('YEAR(created_at) = 2024')->get();
// Solution: whereYear('created_at', 2024)

// ❌ NOT IN — biasanya lambat
Product::whereNotIn('category_id', [2, 3])->get();
// Solution: pakai whereIn dengan data yang dibutuhkan

// ❌ OR tanpa index
Product::where('name', 'like', '%laptop%')
    ->orWhere('description', 'like', '%laptop%')
    ->get();
// Solution: Fulltext index + whereFullText()

// ❌ N+1 — N query untuk relasi
$products = Product::all();
foreach ($products as $p) {
    echo $p->category->name; // N query!
}
// Solution: Eager loading
```

### 5.2 Query Optimization Checklist

```
☑ Apakah ada index untuk kolom di WHERE?
☑ Apakah query menggunakan LIKE dengan wildcard di awal?
☑ Apakah ada N+1 problem (lazy loading relasi)?
☑ Apakah SELECT * mengambil terlalu banyak kolom?
☑ Apakah pagination menggunakan OFFSET besar?
☑ Apakah ada subquery yang tidak perlu?
☑ Apakah ada function di kolom (WHERE YEAR(...))?
```

### 5.3 Pagination with OFFSET

```sql
-- ❌ OFFSET besar = lambat
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 100000;
-- Database tetap baca 100010 baris, buang 100000

-- ✅ Cursor pagination (untuk API)
SELECT * FROM products WHERE id > 100000 ORDER BY id LIMIT 10;
```

---

## 6. Fulltext Search

### 6.1 Migration

```php
Schema::table('products', function (Blueprint $table) {
    $table->fullText(['name', 'description']);
    // Membuat FULLTEXT index untuk pencarian teks
});
```

### 6.2 Query

```php
// Pencarian dengan fulltext
Product::whereFullText(['name', 'description'], 'laptop gaming')->get();

// Mode boolean — untuk kontrol lebih
Product::whereFullText(['name', 'description'], '+laptop -acer', ['mode' => 'boolean'])->get();
// +laptop = harus ada
// -acer = tidak boleh ada
```

### 6.3 Like vs Fulltext

```sql
-- ❌ LIKE — lambat, tidak bisa pakai index
SELECT * FROM products WHERE name LIKE '%laptop%';

-- ✅ FULLTEXT — cepat, pakai index
SELECT * FROM products WHERE MATCH(name, description) AGAINST('laptop');
```

---

## 7. Index di Codebase

### 7.1 Index yang Ada

```php
// Dari migration yang ada:
$table->unique('slug');                    // products, categories, brands
$table->unique('email');                   // users
$table->unique(['user_id', 'product_id']); // wishlist
$table->index('category_id');              // products (via FK)
$table->index('brand_id');                 // products (via FK)
$table->index('status');                   // orders
```

### 7.2 Yang Mungkin Kurang

```php
// Cek apakah ada index untuk:
// - products.is_active (sering di-where)
// - products.price (sering di-order/filter)
// - orders.user_id (sering di-join)
// - orders.created_at (sering di-order)
```

---

## 8. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ INDEX                                                              │
│  Bagai daftar isi buku → langsung ke halaman                      │
│  Primary Key → index otomatis                                     │
│  Foreign Key → index otomatis (via constrained())                 │
│  Unique → index + constraint                                      │
│  Fulltext → untuk LIKE / pencarian teks                           │
├─────────────────────────────────────────────────────────────────────┤
│ KAPAN INDEX?                                                       │
│  ✅ Kolom di WHERE (sering dipakai)                               │
│  ✅ Kolom di ORDER BY                                             │
│  ✅ Foreign Key                                                   │
│  ❌ Low cardinality (is_active — hanya 0/1)                       │
│  ❌ Table kecil (< 1000 baris)                                    │
│  ❌ Kolom jarang dipakai                                          │
├─────────────────────────────────────────────────────────────────────┤
│ EXPLAIN                                                            │
│  type: ALL → full scan (❌)                                       │
│  type: ref → index lookup (✅)                                    │
│  type: const → PK lookup (✅✅)                                   │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  Index: unique slug, FK index                                     │
│  Telescope → lihat timing query                                   │
│  Yang perlu dicek: index untuk is_active, price                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Cek index.** Buka phpMyAdmin → pilih table `products` → Structure → lihat Indexes. Index apa saja yang ada?

2. **EXPLAIN test.** Di phpMyAdmin → SQL, jalankan:
   ```sql
   EXPLAIN SELECT * FROM products WHERE slug = 'laptop-gaming';
   ```
   Lihat `type`, `rows`, `key`.

3. **EXPLAIN tanpa index:**
   ```sql
   EXPLAIN SELECT * FROM products WHERE price > 10000;
   ```
   Apakah pakai index? Jika tidak, coba tambah index untuk price.

4. **Telescope query.** Buka `http://olshop-koneksi.test/telescope/queries`. Cari query yang lambat (slow). Apa yang bisa dioptimasi?

5. **Tambah index.** Buat migration untuk tambah index `is_active` di products table. Jalankan `php artisan migrate`.

---

## 🔗 Referensi

- [MySQL Indexing](https://dev.mysql.com/doc/refman/8.4/en/optimization-indexes.html)
- [MySQL EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html)
- [Laravel: Migrations - Indexes](https://laravel.com/docs/11.x/migrations#indexes)
- [Laravel: Fulltext Search](https://laravel.com/docs/11.x/scout)
- Codebase: Telescope → Queries
- Codebase: `database/migrations/` — existing indexes
