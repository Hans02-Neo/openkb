# 07-01: Apa Itu Database — Tempat Penyimpanan Data

> **Fase**: 7 — Database & SQL  
> **Prasyarat**: FASE 6 (Laravel Deep Dive)  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: database, relational, table, row, column, schema, DBMS, MySQL, primary key, foreign key

---

## 📋 Ringkasan

Database adalah tempat menyimpan data secara **terstruktur**. Di aplikasi web, database menyimpan user, produk, order, dan semua data yang perlu bertahan lama (tidak hilang saat server mati).

**Target pemahaman:**
- Kamu paham konsep relational database
- Kamu bisa membedakan table, row, column
- Kamu paham primary key dan foreign key
- Kamu paham bagaimana Laravel terhubung ke database

---

## 1. Database vs File Biasa

### 1.1 File Biasa (Teks/JSON)

```json
// data.json — Simpan data di file
[
    {"id": 1, "name": "Laptop", "price": 15000000},
    {"id": 2, "name": "Mouse", "price": 250000}
]
```

**Masalah file biasa:**
- **Concurrent access** — 2 user tulis bersamaan → data korup
- **Query** — cari produk harga > 10000 → baca semua file → filter manual
- **Relasi** — hubungkan produk ke kategori → manual
- **Scalability** — 1 juta produk → muat di memory?
- **Security** — siapa yang bisa akses?

### 1.2 Database Management System (DBMS)

DBMS adalah software khusus untuk mengelola data:

| DBMS | Jenis | Guna |
|------|-------|------|
| **MySQL** | Relational | Web app (paling populer) |
| **PostgreSQL** | Relational | Advanced features |
| **SQLite** | Relational | File-based, development |
| **MariaDB** | Relational | Fork MySQL |
| **MongoDB** | NoSQL | Dokumen JSON |

**Di codebase: MySQL** via Laragon.

---

## 2. Relational Database — Data Berelasi

### 2.1 Konsep

Data disimpan di **table** yang saling berelasi:

```
┌─────────────────────────────────────────────────────────┐
│ DATABASE: koneksi_store                                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  products table                                          │
│  ┌────┬──────────────────┬────────┬─────────────┐      │
│  │ id │ name             │ price  │ category_id │      │
│  ├────┼──────────────────┼────────┼─────────────┤      │
│  │ 1  │ Laptop Gaming    │ 15.000 │ 1           │──────┼──┐
│  │ 2  │ Mouse Wireless   │ 250    │ 1           │──┐   │  │
│  │ 3  │ Buku PHP         │ 75     │ 2           │  │   │  │
│  └────┴──────────────────┴────────┴─────────────┘  │   │  │
│                                                      │   │  │
│  categories table                                    │   │  │
│  ┌────┬──────────────┐                               │   │  │
│  │ id │ name         │                               │   │  │
│  ├────┼──────────────┤                               │   │  │
│  │ 1  │ Elektronik   │◄──────────────────────────────┘───┘  │
│  │ 2  │ Buku         │◄──────────────────────────────┘     │
│  └────┴──────────────┘                                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 Table, Row, Column

| Istilah | Arti | Analogi Excel |
|---------|------|---------------|
| **Table** | Kumpulan data sejenis | Sheet |
| **Row** | Satu record/data | Baris |
| **Column** | Satu field/attribute | Kolom |
| **Schema** | Struktur table (kolom apa saja) | Header |

```
TABLE: products
COLUMNS: id, name, price, category_id, created_at

ROW 1: 1, 'Laptop Gaming', 15000000, 1, '2024-01-01'
ROW 2: 2, 'Mouse Wireless', 250000, 1, '2024-01-02'
ROW 3: 3, 'Buku PHP', 75000, 2, '2024-01-03'
```

### 2.3 Diagram Relasi Codebase

```
users ────< orders ────< order_items >──── products
                           │
customers >───< addresses  │
                           │
categories >─── products   │
brands >─────── products   │
                           │
payments >─── orders       │
                           │
settings (single table)    │
                           ▼
                    media (spatie/laravel-medialibrary)
```

---

## 3. Primary Key dan Foreign Key

### 3.1 Primary Key — Identitas Unik

Setiap table harus punya **primary key** — kolom yang unik untuk setiap row.

```php
// Di migration:
$table->id(); // BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
// Sama dengan:
// $table->bigIncrements('id');
```

| id | name | — setiap id UNIK, tidak boleh sama |
|----|------|-------------------------------------|
| 1 | Laptop | ✅ |
| 2 | Mouse | ✅ |
| 1 | Keyboard | ❌ DUPLIKAT! |

### 3.2 Foreign Key — Relasi Antar Table

```php
// Di migration products:
$table->foreignId('category_id')
      ->constrained()     // references categories.id
      ->onDelete('cascade');
```

```
products.category_id ──► categories.id
      5                        1 (Elektronik)
      5                        2 (Buku)
      6                        3 (Fashion)

FK memastikan:
- Tidak bisa masukkan category_id = 999 jika tidak ada di categories ✅
- Tidak bisa hapus category jika masih ada produk di dalamnya ✅
```

### 3.3 Index — Mempercepat Pencarian

```php
$table->index('category_id');   // INDEX — cepat filter by category
$table->unique('slug');          // UNIQUE — tidak boleh duplikat
```

**Analogi:**
- Tanpa index → cari data seperti cari halaman di buku tanpa daftar isi
- Dengan index → seperti daftar isi — langsung ke halaman yang dicari

---

## 4. MySQL di Laragon

### 4.1 Mengakses MySQL

```bash
# Via Laragon → Database → Open MySQL
# Atau command line:
"C:\laragon\bin\mysql\mysql-8.4.0\bin\mysql.exe" -u root
```

```sql
-- Lihat database
SHOW DATABASES;

-- Pilih database
USE koneksi_store;

-- Lihat table
SHOW TABLES;

-- Lihat struktur table
DESCRIBE products;
```

### 4.2 phpMyAdmin

Laragon menyediakan phpMyAdmin di:
```
http://localhost/phpmyadmin
```
- Login: root (tanpa password)
- Pilih database: `koneksi_store`
- Lihat table, browse data, jalankan SQL

### 4.3 Config Database Laravel

```php
// config/database.php
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', '127.0.0.1'),
        'port' => env('DB_PORT', '3306'),
        'database' => env('DB_DATABASE', 'koneksi_store'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
    ],
];
```

```ini
// .env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=koneksi_store
DB_USERNAME=root
DB_PASSWORD=
```

---

## 5. Data Types Penting

| Tipe MySQL | Guna | Laravel Migration |
|-----------|------|-------------------|
| `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key | `$table->id()` |
| `VARCHAR(255)` | String pendek | `$table->string('name')` |
| `TEXT` | String panjang | `$table->text('desc')` |
| `INT` | Angka | `$table->integer('price')` |
| `DECIMAL(10,2)` | Uang | `$table->decimal('amount', 10, 2)` |
| `BOOLEAN` / `TINYINT(1)` | Yes/No | `$table->boolean('active')` |
| `DATETIME` | Tanggal + waktu | `$table->dateTime('published')` |
| `TIMESTAMP` | Otomatis | `$table->timestamps()` |
| `JSON` | Data JSON | `$table->json('metadata')` |
| `ENUM('a','b')` | Pilihan terbatas | `$table->enum('status', ['a','b'])` |

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ RELATIONAL DATABASE                                                │
│  Database = kumpulan table                                         │
│  Table    = kumpulan baris data sejenis                            │
│  Row      = satu record                                            │
│  Column   = satu field/attribute                                   │
│  PK       = Primary Key (unique ID)                               │
│  FK       = Foreign Key (relasi antar table)                      │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  DBMS: MySQL (via Laragon)                                         │
│  DB Name: koneksi_store                                            │
│  Tables: ~10 (products, categories, orders, users, dll)            │
│  Akses: phpMyAdmin di localhost/phpmyadmin                        │
│  Config: config/database.php + .env                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Akses MySQL.** Buka Laragon → Database → Open MySQL. Ketik:
   ```sql
   SHOW DATABASES;
   USE koneksi_store;
   SHOW TABLES;
   ```

2. **Lihat struktur.** Pilih satu table (misal `products`):
   ```sql
   DESCRIBE products;
   ```
   Catat semua kolom, tipe data, dan key.

3. **phpMyAdmin.** Buka `http://localhost/phpmyadmin`. Pilih `koneksi_store`. Browse `products` table. Berapa row?

4. **Cari foreign key.** Lihat migration `create_products_table`. Foreign key apa yang ada? Ke table mana?

5. **Cek .env.** Buka `.env` di root project. Cari `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`. Apa nilainya?

---

## 🔗 Referensi

- [MySQL Tutorial](https://www.mysqltutorial.org/)
- [Laravel Docs: Database](https://laravel.com/docs/11.x/database)
- [Laravel Docs: Migrations](https://laravel.com/docs/11.x/migrations)
- Codebase: `config/database.php`
- Codebase: `.env` — DB config
- Codebase: `database/migrations/` — schema definitions
