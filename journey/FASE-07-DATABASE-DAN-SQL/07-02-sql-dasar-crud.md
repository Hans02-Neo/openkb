# 07-02: SQL Dasar — CRUD dengan Query

> **Fase**: 7 — Database & SQL  
> **Prasyarat**: 07-01-apa-itu-database  
> **Waktu baca**: 75-90 menit  
> **Kata kunci**: SELECT, INSERT, UPDATE, DELETE, WHERE, ORDER BY, LIMIT, LIKE, JOIN, GROUP BY, Eloquent vs SQL

---

## 📋 Ringkasan

SQL (Structured Query Language) adalah bahasa untuk berkomunikasi dengan database. Semua operasi CRUD bisa dilakukan dengan SQL. Eloquent ORM Laravel pada akhirnya menghasilkan SQL.

**Target pemahaman:**
- Kamu bisa menulis SELECT, INSERT, UPDATE, DELETE
- Kamu paham WHERE, JOIN, GROUP BY
- Kamu bisa melihat SQL yang dihasilkan Eloquent
- Kamu paham Eloquent vs SQL mentah

---

## 1. SELECT — Membaca Data

### 1.1 SELECT Dasar

```sql
-- Ambil semua kolom
SELECT * FROM products;

-- Ambil kolom tertentu
SELECT id, name, price FROM products;

-- Ambil dengan alias
SELECT name AS product_name, price FROM products;

-- Batasi jumlah
SELECT * FROM products LIMIT 10;

-- Urutkan
SELECT * FROM products ORDER BY price DESC;
SELECT * FROM products ORDER BY name ASC;
```

### 1.2 WHERE — Filter

```sql
-- Sama dengan
SELECT * FROM products WHERE id = 1;
SELECT * FROM products WHERE is_active = 1;

-- Perbandingan
SELECT * FROM products WHERE price > 10000;
SELECT * FROM products WHERE price >= 10000;
SELECT * FROM products WHERE price < 5000;
SELECT * FROM products WHERE price BETWEEN 10000 AND 50000;

-- LIKE (pencarian teks)
SELECT * FROM products WHERE name LIKE '%laptop%';
SELECT * FROM products WHERE name LIKE 'laptop%';    -- mulai dengan 'laptop'
SELECT * FROM products WHERE name LIKE '%gaming';    -- akhir dengan 'gaming'

-- IN (salah satu dari)
SELECT * FROM products WHERE category_id IN (1, 3, 5);

-- NULL
SELECT * FROM products WHERE deleted_at IS NULL;
SELECT * FROM products WHERE description IS NOT NULL;

-- AND / OR
SELECT * FROM products
WHERE is_active = 1
  AND price > 10000
  AND (category_id = 1 OR category_id = 2);
```

### 1.3 Eloquent vs SQL

```php
// Eloquent:
Product::where('is_active', true)
    ->where('price', '>', 10000)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

```sql
-- SQL yang dihasilkan:
SELECT * FROM products
WHERE is_active = 1
  AND price > 10000
ORDER BY created_at DESC
LIMIT 10;
```

---

## 2. INSERT — Menambah Data

```sql
-- Insert satu baris
INSERT INTO products (name, slug, price, category_id, is_active)
VALUES ('Laptop Gaming', 'laptop-gaming', 15000000, 1, 1);

-- Insert multiple baris
INSERT INTO products (name, slug, price, category_id, is_active)
VALUES
    ('Mouse', 'mouse-wireless', 250000, 1, 1),
    ('Keyboard', 'keyboard-mechanical', 750000, 1, 1);
```

```php
// Eloquent:
Product::create([
    'name' => 'Laptop Gaming',
    'slug' => 'laptop-gaming',
    'price' => 15000000,
    'category_id' => 1,
]);
```

```sql
-- SQL yang dihasilkan:
INSERT INTO products (name, slug, price, category_id, is_active, created_at, updated_at)
VALUES ('Laptop Gaming', 'laptop-gaming', 15000000, 1, 1, NOW(), NOW());
```

---

## 3. UPDATE — Mengubah Data

```sql
-- Update satu baris
UPDATE products
SET price = 14000000, updated_at = NOW()
WHERE id = 1;

-- Update banyak baris
UPDATE products
SET is_active = 0
WHERE category_id = 3;
```

```php
// Eloquent:
$product = Product::find(1);
$product->price = 14000000;
$product->save();

// Atau:
Product::where('category_id', 3)->update(['is_active' => false]);
```

```sql
-- SQL yang dihasilkan:
UPDATE products SET price = 14000000, updated_at = NOW() WHERE id = 1;
UPDATE products SET is_active = 0, updated_at = NOW() WHERE category_id = 3;
```

---

## 4. DELETE — Menghapus Data

```sql
-- Hapus satu baris
DELETE FROM products WHERE id = 1;

-- Hapus banyak
DELETE FROM products WHERE is_active = 0;

-- Hapus semua (HATI-HATI!)
DELETE FROM products;
-- atau TRUNCATE products; (lebih cepat, reset auto-increment)
```

```php
// Eloquent (soft delete — set deleted_at, tidak hilang):
$product->delete();

// Eloquent (force delete — hilang dari DB):
$product->forceDelete();

// Eloquent (delete by query):
Product::where('is_active', false)->delete();
```

```sql
-- Soft delete:
UPDATE products SET deleted_at = NOW() WHERE id = 1;

-- Force delete:
DELETE FROM products WHERE id = 1;
```

---

## 5. JOIN — Menggabungkan Table

### 5.1 INNER JOIN

Ambil data yang **cocok** di kedua table:

```sql
SELECT products.name, categories.name AS category
FROM products
INNER JOIN categories ON products.category_id = categories.id;
```

```
Hasil:
┌──────────────────┬──────────────┐
│ name             │ category     │
├──────────────────┼──────────────┤
│ Laptop Gaming    │ Elektronik   │
│ Mouse Wireless   │ Elektronik   │
│ Buku PHP         │ Buku         │
└──────────────────┴──────────────┘
```

### 5.2 LEFT JOIN

Ambil SEMUA data dari table kiri, cocokkan dengan kanan:

```sql
SELECT products.name, categories.name AS category
FROM products
LEFT JOIN categories ON products.category_id = categories.id;
-- Produk tanpa category_id → category = NULL
```

### 5.3 Multiple JOIN

```sql
SELECT
    products.name,
    categories.name AS category,
    brands.name AS brand
FROM products
INNER JOIN categories ON products.category_id = categories.id
LEFT JOIN brands ON products.brand_id = brands.id;
```

### 5.4 Eloquent vs SQL JOIN

```php
// Eloquent — eager loading
$products = Product::with(['category', 'brand'])->get();
```

```sql
-- SQL yang dihasilkan:
SELECT * FROM products;
SELECT * FROM categories WHERE id IN (1, 2, 3, ...);
SELECT * FROM brands WHERE id IN (1, 2, 3, ...);
```

Atau dengan join eksplisit:
```sql
SELECT p.*, c.name AS category_name, b.name AS brand_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
LEFT JOIN brands b ON p.brand_id = b.id;
```

---

## 6. GROUP BY dan Aggregate

### 6.1 Fungsi Aggregate

```sql
SELECT COUNT(*) FROM products;                    -- jumlah
SELECT SUM(price) FROM products;                  -- total
SELECT AVG(price) FROM products;                  -- rata-rata
SELECT MIN(price) FROM products;                  -- termurah
SELECT MAX(price) FROM products;                  -- termahal
```

### 6.2 GROUP BY

```sql
-- Jumlah produk per kategori
SELECT
    categories.name,
    COUNT(products.id) AS total_products,
    AVG(products.price) AS avg_price
FROM products
INNER JOIN categories ON products.category_id = categories.id
GROUP BY categories.id, categories.name
ORDER BY total_products DESC;
```

```
Hasil:
┌──────────────┬────────────────┬───────────┐
│ name         │ total_products │ avg_price │
├──────────────┼────────────────┼───────────┤
│ Elektronik   │ 15             │ 500000    │
│ Buku         │ 8              │ 75000     │
│ Fashion      │ 3              │ 150000    │
└──────────────┴────────────────┴───────────┘
```

### 6.3 HAVING — Filter setelah GROUP BY

```sql
SELECT
    category_id,
    COUNT(*) AS total
FROM products
GROUP BY category_id
HAVING total > 5;  -- hanya kategori dengan >5 produk
```

### 6.4 Eloquent vs SQL Aggregate

```php
// Eloquent:
Product::where('is_active', true)->count();
Product::where('category_id', 1)->sum('price');
Product::avg('price');
```

---

## 7. Subquery

```sql
-- Produk dengan harga di atas rata-rata
SELECT name, price FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- Kategori yang memiliki produk aktif
SELECT * FROM categories
WHERE id IN (
    SELECT DISTINCT category_id
    FROM products
    WHERE is_active = 1
);
```

---

## 8. Melihat SQL dari Eloquent

### 8.1 toSql() — Lihat Query tanpa Eksekusi

```php
$query = Product::where('is_active', true)
    ->where('price', '>', 10000);

dump($query->toSql());
// "SELECT * FROM products WHERE is_active = ? AND price > ?"

dump($query->getBindings());
// [1, 10000]
```

### 8.2 Telescope — Lihat Query Real

Buka `http://olshop-koneksi.test/telescope/queries` untuk melihat semua query yang dijalankan:

```sql
-- Di Telescope:
SELECT * FROM `products`
WHERE `is_active` = ?
AND `price` > ?
ORDER BY `created_at` DESC
LIMIT 12 OFFSET 0
-- Bindings: [1, 10000]
-- Time: 2.5ms
```

### 8.3 DB::enableQueryLog()

```php
// Di controller atau tinker:
\Illuminate\Support\Facades\DB::enableQueryLog();

Product::where('is_active', true)->get();

dump(\Illuminate\Support\Facades\DB::getQueryLog());
// [
//   'query' => 'SELECT * FROM products WHERE is_active = ?',
//   'bindings' => [1],
//   'time' => 2.5,
// ]
```

---

## 9. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ SQL CRUD                                                           │
│  SELECT  → SELECT * FROM products WHERE id = 1                    │
│  INSERT  → INSERT INTO products (...) VALUES (...)                 │
│  UPDATE  → UPDATE products SET price = 1000 WHERE id = 1          │
│  DELETE  → DELETE FROM products WHERE id = 1                      │
├─────────────────────────────────────────────────────────────────────┤
│ CLAUSES                                                            │
│  WHERE   → filter (AND, OR, IN, LIKE, BETWEEN)                    │
│  ORDER BY → sorting (ASC, DESC)                                   │
│  LIMIT   → batasi jumlah                                          │
│  JOIN    → gabung table (INNER, LEFT)                             │
│  GROUP BY → group + aggregate (COUNT, SUM, AVG)                   │
│  HAVING  → filter setelah GROUP BY                                │
├─────────────────────────────────────────────────────────────────────┤
│ ELOQUENT → SQL                                                     │
│  Product::find(1)     → SELECT * FROM products WHERE id = 1       │
│  Product::where(...)  → SELECT * FROM products WHERE ...          │
│  $product->save()     → UPDATE products SET ... WHERE id = ?      │
│  Product::create(...) → INSERT INTO products (...) VALUES (...)   │
│  $product->delete()   → UPDATE products SET deleted_at = NOW()    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **SELECT via phpMyAdmin.** Buka phpMyAdmin → koneksi_store → SQL. Jalankan:
   ```sql
   SELECT * FROM products;
   SELECT id, name, price FROM products WHERE price > 100000;
   SELECT COUNT(*) FROM products WHERE is_active = 1;
   ```

2. **Lihat SQL Eloquent.** Buka telescope di `http://olshop-koneksi.test/telescope/queries`. Buka halaman shop — SQL apa yang dijalankan?

3. **Join query.** Tulis SQL untuk mengambil produk dengan nama kategori:
   ```sql
   SELECT p.name, c.name AS category
   FROM products p
   INNER JOIN categories c ON p.category_id = c.id;
   ```

4. **Eloquent toSql.** Di `php artisan tinker`:
   ```php
   echo Product::where('is_active', true)->where('price', '>', 10000)->toSql();
   ```

5. **GROUP BY.** Tulis SQL untuk menghitung jumlah produk per kategori (tampilkan nama kategori dan jumlahnya).

---

## 🔗 Referensi

- [MySQL SELECT](https://dev.mysql.com/doc/refman/8.4/en/select.html)
- [Laravel: Query Builder](https://laravel.com/docs/11.x/queries)
- [Laravel: Eloquent SQL](https://laravel.com/docs/11.x/eloquent#eloquent-sql)
- Codebase: Telescope → Queries → lihat SQL real
- Codebase: `php artisan tinker` → test Eloquent → toSql()
