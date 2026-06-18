# 06-09: Migration dan Seeder — Schema dan Data Awal

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-08-service-layer  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: migration, schema, table, column, seeder, factory, artisan migrate, foreign key, rollback

---

## 📋 Ringkasan

**Migration** adalah **version control untuk database** — cara mendefinisikan dan memodifikasi schema database dengan kode PHP. **Seeder** adalah data awal untuk testing dan development.

**Target pemahaman:**
- Kamu paham cara membuat dan menjalankan migration
- Kamu bisa menulis schema untuk table dan relasi
- Kamu bisa membuat seeder dan factory
- Kamu paham workflow migration di tim

---

## 1. Migration — Version Control untuk Database

### 1.1 Masalah Tanpa Migration

```
Tim development:
  Developer A: buat table 'products' di local
  Developer B: buat table 'products' di local — tapi kolom beda!
  Developer C: tambah kolom 'discount' — ngomong ke A dan B?

Tanpa migration → sinkronisasi database manual → error!
```

### 1.2 Dengan Migration

```
Setiap perubahan schema adalah FILE PHP:
2024_01_01_000001_create_products_table.php  → CREATE TABLE products
2024_01_15_000002_add_discount_to_products.php → ALTER TABLE ADD discount
2024_02_01_000003_create_orders_table.php    → CREATE TABLE orders

Semua file di-version control (git).
Developer tinggal run: php artisan migrate
```

### 1.3 Anatomi Migration

```php
<?php
// database/migrations/2024_01_01_000001_create_products_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    // Jalankan migration — buat table
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();                          // BIGINT AUTO_INCREMENT PRIMARY KEY
            $table->string('name', 255);           // VARCHAR(255)
            $table->string('slug', 255)->unique(); // UNIQUE INDEX
            $table->text('description')->nullable();  // TEXT, nullable
            $table->integer('price')->default(0);  // INT, default 0
            $table->integer('stock_quantity')->default(0);
            $table->boolean('is_active')->default(true);
            $table->foreignId('category_id')
                  ->constrained()                  // FOREIGN KEY → categories.id
                  ->onDelete('cascade');           // CASCADE ON DELETE
            $table->timestamps();                  // created_at, updated_at
        });
    }

    // Batalkan migration — hapus table
    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

### 1.4 Migration di Codebase

```bash
# 25 migration sudah dijalankan
# Lihat di database/migrations/
```

Codebase memiliki migration untuk:
- `create_users_table` — user auth
- `create_customers_table` — data pelanggan
- `create_categories_table` — kategori produk
- `create_brands_table` — merek produk
- `create_products_table` — produk
- `create_orders_table` — pesanan
- `create_order_items_table` — item pesanan
- `create_payments_table` — pembayaran
- Dan lainnya...

---

## 2. Column Types

### 2.1 Column Types Umum

```php
// String & Text
$table->string('name', 100);       // VARCHAR(100)
$table->text('description');       // TEXT
$table->longText('content');       // LONGTEXT
$table->char('phone', 15);        // CHAR(15)
$table->json('metadata');          // JSON

// Numeric
$table->integer('qty');            // INT
$table->tinyInteger('age');        // TINYINT
$table->bigInteger('views');       // BIGINT
$table->decimal('price', 10, 2);   // DECIMAL(10,2)
$table->float('rating');           // FLOAT
$table->boolean('is_active');      // TINYINT(1)

// Date & Time
$table->date('birth_date');        // DATE
$table->dateTime('published_at');  // DATETIME
$table->time('open_time');         // TIME
$table->timestamp('verified_at');  // TIMESTAMP
$table->timestamps();              // created_at, updated_at
$table->softDeletes();             // deleted_at (soft delete)

// Other
$table->uuid('id')->primary();     // UUID primary key
$table->foreignId('user_id');      // UNSIGNED BIGINT
$table->enum('status', ['a','b']); // ENUM
$table->year('tahun');             // YEAR
```

### 2.2 Column Modifiers

```php
$table->string('email')->unique();           // UNIQUE INDEX
$table->string('name')->nullable();          // NULL allowed
$table->integer('views')->default(0);        // DEFAULT 0
$table->string('slug')->after('name');       // After column 'name'
$table->string('slug')->first();             // Column pertama
$table->string('note')->charset('utf8mb4'); // Custom charset
```

---

## 3. Foreign Key dan Relasi

### 3.1 Foreign Key

```php
// Cara 1: foreignId() + constrained() — recommended
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->foreignId('category_id')
          ->constrained()           // → categories.id
          ->onDelete('cascade');    // hapus produk jika kategori dihapus
    $table->foreignId('brand_id')
          ->constrained()
          ->onDelete('set null');   // set null jika brand dihapus
});

// Cara 2: Manual
$table->unsignedBigInteger('user_id');
$table->foreign('user_id')
      ->references('id')
      ->on('users')
      ->onDelete('cascade');
```

### 3.2 On Delete Behavior

| Action | Efek |
|--------|------|
| `cascade` | Hapus child jika parent dihapus |
| `restrict` | Cegah hapus parent jika masih ada child |
| `set null` | Set FK jadi NULL jika parent dihapus |
| `no action` | Tidak lakukan apa-apa |

### 3.3 Index

```php
$table->index('email');           // INDEX
$table->unique('slug');           // UNIQUE INDEX
$table->primary('id');            // PRIMARY KEY
$table->fullText('description');  // FULLTEXT INDEX

// Composite index
$table->index(['category_id', 'is_active']);
$table->unique(['user_id', 'product_id']); // unique pair
```

---

## 4. Membuat Migration

### 4.1 Command

```bash
# Buat migration
php artisan make:migration create_products_table
# File: database/migrations/2024_xx_xx_xxxxxx_create_products_table.php

# Migration untuk existing table
php artisan make:migration add_discount_to_products_table
# --table=products untuk isi Schema::table()

# Migration spesifik
php artisan make:migration create_orders_table --create=orders
php artisan make:migration add_discount_to_products --table=products
```

### 4.2 Menjalankan Migration

```bash
# Jalankan semua migration yang belum dijalankan
php artisan migrate

# Lihat status migration
php artisan migrate:status

# Rollback migration terakhir
php artisan migrate:rollback

# Rollback beberapa step
php artisan migrate:rollback --step=3

# Rollback semua
php artisan migrate:reset

# Rollback + migrate ulang
php artisan migrate:refresh

# Hapus semua table + migrate ulang
php artisan migrate:fresh

# migrate:fresh + seed
php artisan migrate:fresh --seed
```

### 4.3 Migration di Production

```bash
# ✅ AMAN
php artisan migrate

# ❌ JANGAN di production
php artisan migrate:fresh     # HAPUS SEMUA DATA!
php artisan migrate:refresh   # HAPUS SEMUA DATA!
```

---

## 5. Seeder — Data Awal

### 5.1 DatabaseSeeder

```php
<?php
// database/seeders/DatabaseSeeder.php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            CategorySeeder::class,
            BrandSeeder::class,
            ProductSeeder::class,
            AdminUserSeeder::class,
        ]);
    }
}
```

### 5.2 Seeder Sederhana

```php
<?php
// database/seeders/CategorySeeder.php

namespace Database\Seeders;

use App\Models\Category;
use Illuminate\Database\Seeder;

class CategorySeeder extends Seeder
{
    public function run(): void
    {
        Category::create([
            'name' => 'Elektronik',
            'slug' => 'elektronik',
            'is_active' => true,
            'sort_order' => 1,
        ]);

        Category::create([
            'name' => 'Fashion',
            'slug' => 'fashion',
            'is_active' => true,
            'sort_order' => 2,
        ]);

        Category::create([
            'name' => 'Makanan & Minuman',
            'slug' => 'makanan-minuman',
            'is_active' => true,
            'sort_order' => 3,
        ]);
    }
}
```

### 5.3 Factory — untuk Banyak Data

```bash
php artisan make:factory ProductFactory --model=Product
```

```php
<?php
// database/factories/ProductFactory.php

namespace Database\Factories;

use App\Models\Product;
use Illuminate\Database\Eloquent\Factories\Factory;

class ProductFactory extends Factory
{
    protected $model = Product::class;

    public function definition(): array
    {
        return [
            'name' => fake()->words(3, true),
            'slug' => fake()->unique()->slug(3),
            'short_description' => fake()->sentence(),
            'description' => fake()->paragraphs(3, true),
            'price' => fake()->numberBetween(10000, 5000000),
            'stock_quantity' => fake()->numberBetween(0, 100),
            'weight_grams' => fake()->numberBetween(100, 5000),
            'is_active' => true,
            'category_id' => Category::factory(),
            'brand_id' => Brand::factory(),
        ];
    }

    // State — variasi
    public function inactive(): static
    {
        return $this->state(fn(array $attrs) => [
            'is_active' => false,
        ]);
    }

    public function cheap(): static
    {
        return $this->state(fn(array $attrs) => [
            'price' => fake()->numberBetween(1000, 50000),
        ]);
    }
}
```

### 5.4 Menggunakan Factory

```php
// Di seeder
class ProductSeeder extends Seeder
{
    public function run(): void
    {
        // Buat 50 produk
        Product::factory(50)->create();

        // Buat 10 produk inactive
        Product::factory(10)->inactive()->create();

        // Buat 20 produk murah
        Product::factory(20)->cheap()->create();

        // Buat 1 produk spesifik + relasi
        Product::factory()
            ->has(OrderItem::factory()->count(5))
            ->create([
                'name' => 'Produk Unggulan',
                'price' => 15000000,
            ]);
    }
}

// Di tinker:
Product::factory()->count(10)->create();
```

### 5.5 Menjalankan Seeder

```bash
# Jalankan semua seeder
php artisan db:seed

# Jalankan seeder spesifik
php artisan db:seed --class=CategorySeeder

# Saat migrate:fresh
php artisan migrate:fresh --seed
```

---

## 6. Migration Workflow Tim

### 6.1 Workflow

```
1. Developer A membuat migration baru
2. Commit + push ke git
3. Developer B pull → php artisan migrate

Tidak ada lagi: "Tambahin kolom ini ke database ya!"
Semua otomatis via migration!
```

### 6.2 Aturan Emas Migration

1. **Jangan edit migration yang sudah di-commit** — buat migration baru
2. **`up()` dan `down()` harus balanced** — rollback harus bisa
3. **Setiap perubahan schema = migration baru**
4. **Jalankan migration sebelum commit** — pastikan jalan

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ MIGRATION                                                          │
│  File: database/migrations/2024_xx_xx_xxxxxx_create_products_table │
│  up():    Schema::create('products', fn($table) => { ... })       │
│  down():  Schema::dropIfExists('products')                         │
│  Run:     php artisan migrate                                      │
│  Rollback: php artisan migrate:rollback                            │
├─────────────────────────────────────────────────────────────────────┤
│ COLUMN TYPES                                                       │
│  id(), string(), text(), integer(), boolean(), decimal()           │
│  timestamps(), softDeletes(), foreignId(), json()                  │
├─────────────────────────────────────────────────────────────────────┤
│ SEEDER                                                             │
│  DatabaseSeeder:: → panggil seeder lain via $this->call()          │
│  Seeder: Category::create(['name' => '...'])                      │
│  Factory: Product::factory(50)->create()                          │
│  Run:     php artisan db:seed                                      │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  25 migration — semua table sudah termigrasi                       │
│  Seeder → data awal (admin, kategori, brand, produk dummy)         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Lihat migration.** Buka `database/migrations/`. Berapa file migration? Buka migration terakhir — apa yang dilakukan?

2. **Cek status.** Jalankan `php artisan migrate:status`. Migration mana yang sudah dijalankan (Y) dan yang belum (N)?

3. **Buat migration.** Buat migration `add_discount_to_products` yang menambah:
   - `discount_percent` integer, default 0, nullable
   - `discount_start` datetime, nullable
   - `discount_end` datetime, nullable

4. **Buat seeder.** Buat `ProductSeeder` yang membuat 20 produk menggunakan factory.

5. **Run seeder.** Jalankan `php artisan db:seed --class=ProductSeeder`. Cek database — apakah produk baru muncul?

---

## 🔗 Referensi

- [Laravel Docs: Migrations](https://laravel.com/docs/11.x/migrations)
- [Laravel Docs: Seeding](https://laravel.com/docs/11.x/seeding)
- [Laravel Docs: Factories](https://laravel.com/docs/11.x/eloquent-factories)
- Codebase: `database/migrations/` — 25 migration
- Codebase: `database/seeders/` — semua seeder
- Codebase: `database/factories/` — factory (jika ada)
