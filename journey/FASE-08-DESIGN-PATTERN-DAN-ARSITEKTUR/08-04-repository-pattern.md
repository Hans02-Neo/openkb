# 08-04: Repository Pattern — Abstraksi Database

> **Fase**: 8 — Design Pattern & Arsitektur  
> **Prasyarat**: 08-03-service-pattern  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: repository, abstraction, data access, query logic, testability, Eloquent vs repository

---

## 📋 Ringkasan

Repository pattern memisahkan **logika query** dari Model. Alih-alih memanggil `Product::where(...)` langsung dari controller atau service, kamu buat class Repository yang membungkus query.

**Target pemahaman:**
- Kamu paham kapan repository diperlukan
- Kamu bisa membuat repository
- Kamu paham trade-off repository vs Eloquent langsung
- Kamu tahu kenapa codebase ini tidak pakai repository

---

## 1. Repository — Apa dan Kenapa

### 1.1 Masalah

```php
// Query tersebar di banyak tempat
class ShopController {
    public function index() {
        Product::where('is_active', true)
            ->where('price', '>', 10000)
            ->get();
    }
}
class OrderController {
    public function store() {
        // Query serupa tapi beda tempat
        Product::where('is_active', true)->get();
    }
}
```

**Masalah:**
- Query duplikasi
- Ganti database → ganti banyak file
- Test → butuh database beneran

### 1.2 Solusi: Repository

```php
// Semua query produk di satu tempat
class ProductRepository
{
    public function getActive(): Collection
    {
        return Product::where('is_active', true)->get();
    }

    public function findById(int $id): ?Product
    {
        return Product::find($id);
    }

    public function getByCategory(int $categoryId): Collection
    {
        return Product::where('category_id', $categoryId)
            ->where('is_active', true)
            ->get();
    }
}
```

---

## 2. Repository vs Eloquent Langsung

### 2.1 Tanpa Repository

```php
// Controller langsung panggil Eloquent
class ShopController extends Controller
{
    public function index(): View
    {
        $products = Product::with('category')
            ->where('is_active', true)
            ->paginate(12);

        return view('shop.index', compact('products'));
    }
}
```

**Keuntungan:** Sederhana, tidak perlu class tambahan.
**Kekurangan:** Query tidak reusable, susah test (butuh DB).

### 2.2 Dengan Repository

```php
class ProductRepository
{
    public function getActiveWithCategory(int $perPage = 12): LengthAwarePaginator
    {
        return Product::with('category')
            ->where('is_active', true)
            ->paginate($perPage);
    }
}

class ShopController extends Controller
{
    public function __construct(private ProductRepository $productRepo) {}

    public function index(): View
    {
        $products = $this->productRepo->getActiveWithCategory();
        return view('shop.index', compact('products'));
    }
}
```

**Keuntungan:** Query terpusat, ganti storage mudah, mock di test.
**Kekurangan:** Banyak boilerplate, abstraksi tambahan.

---

## 3. Repository dengan Interface

Untuk benar-benar **decouple** dari Eloquent:

```php
// Interface — contract
interface ProductRepositoryInterface
{
    public function findById(int $id): ?Product;
    public function getActive(): Collection;
    public function getByCategory(int $categoryId): Collection;
    public function search(string $query): Collection;
}

// Implementation — Eloquent
class EloquentProductRepository implements ProductRepositoryInterface
{
    public function findById(int $id): ?Product
    {
        return Product::find($id);
    }

    public function getActive(): Collection
    {
        return Product::where('is_active', true)->get();
    }

    // ... implementasi lainnya
}

// Alternative implementation — Cache
class CacheProductRepository implements ProductRepositoryInterface
{
    public function findById(int $id): ?Product
    {
        return Cache::remember("product.{$id}", 3600, function () use ($id) {
            return Product::find($id);
        });
    }
}

// Binding di ServiceProvider:
$this->app->bind(
    ProductRepositoryInterface::class,
    EloquentProductRepository::class
    // Ganti ke CacheProductRepository kapan saja
);
```

---

## 4. Kapan Pakai Repository?

### 4.1 Perlu Repository

```php
// ✅ PERLU:
// - Aplikasi besar (100+ model)
// - Multiple data source (Eloquent + API + file)
// - Butuh test tanpa database (unit test)
// - Tim besar (standarisasi akses data)
```

### 4.2 Tidak Perlu Repository

```php
// ✅ TIDAK PERLU:
// - Aplikasi kecil/sederhana
// - Hanya Eloquent (tidak ganti storage)
// - Codebase seperti ini (olshop-koneksi)
```

### 4.3 Kenapa Codebase Ini Tidak Pakai Repository?

```php
// 1. Ukuran: ~10 model — langsung Eloquent cukup
// 2. Tidak ada rencana ganti database
// 3. Eloquent sudah jadi abstraction layer sendiri
//    (ganti MySQL → PostgreSQL → SQLite → cuma ganti .env)
// 4. Service layer sudah pisahkan logika bisnis
// 5. Query sederhana — tidak perlu query complex yang reusable

// Laravel sendiri menyarankan: "Jangan buat repository
// sampai benar-benar diperlukan"
```

---

## 5. Repository vs Service vs Model

| Layer | Tugas | Contoh Method |
|-------|-------|-------------|
| **Model** | Data definition | `$fillable`, `$casts`, relationships |
| **Repository** | Data access | `findById()`, `getActive()`, `search()` |
| **Service** | Business logic | `createOrder()`, `applyDiscount()` |
| **Controller** | HTTP handling | Validasi, panggil service, return response |

Jika ketiganya dipakai:

```
Controller → Service → Repository → Model (Eloquent)
                │
                ▼
             View (Blade)
```

---

## 6. Implementasi Repository Sederhana

```php
<?php
namespace App\Repositories;

use App\Models\Product;
use Illuminate\Pagination\LengthAwarePaginator;
use Illuminate\Support\Collection;

class ProductRepository
{
    public function getActiveWithFilters(
        ?string $category = null,
        ?string $search = null,
        ?string $sort = 'terbaru',
        int $perPage = 12
    ): LengthAwarePaginator {
        $query = Product::with(['category', 'brand'])
            ->where('is_active', true);

        if ($category) {
            $query->whereHas('category', fn($q) =>
                $q->where('slug', $category)
            );
        }

        if ($search) {
            $query->where(function ($q) use ($search) {
                $q->where('name', 'like', "%{$search}%")
                  ->orWhere('short_description', 'like', "%{$search}%");
            });
        }

        $query->orderBy(match($sort) {
            'termurah' => 'price',
            'termahal' => 'price',
            default => 'created_at',
        }, in_array($sort, ['termahal']) ? 'desc' : 'asc');

        return $query->paginate($perPage);
    }

    public function findBySlug(string $slug): ?Product
    {
        return Product::with(['category', 'brand'])
            ->where('slug', $slug)
            ->where('is_active', true)
            ->first();
    }
}
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ REPOSITORY PATTERN                                                 │
├─────────────────────────────────────────────────────────────────────┤
│  Controller → Service → Repository → Eloquent Model → DB          │
│                            │                                        │
│                            ├── ProductRepository                   │
│                            ├── OrderRepository                     │
│                            └── UserRepository                      │
├─────────────────────────────────────────────────────────────────────┤
│ KAPAN PERLU?                                                        │
│  ✅ Multiple data source (DB + API + cache)                       │
│  ✅ Test tanpa database (unit test dengan mock)                    │
│  ✅ Query logic kompleks yang reusable                            │
│  ❌ Aplikasi sederhana (Eloquent cukup)                           │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE INI                                                     │
│  ❌ Tidak pakai Repository                                          │
│  ✅ Langsung Eloquent di Controller/Service                        │
│  ✅ Service Layer sudah pisahkan logika bisnis                     │
│  ✅ Ganti database cukup ganti .env                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Analisis.** Buka `app/Http/Controllers/ShopController.php`. Hitung berapa kali Eloquent dipanggil langsung. Apakah perlu repository?

2. **Buat repository.** Buat `ProductRepository` dengan method:
   - `findById(int $id)`
   - `getActive()`
   - `search(string $query)`
   - `getByCategory(int $categoryId)`

3. **Refaktor.** Ubah ShopController untuk pakai repository. Service provider binding.

4. **Mock test.** Tulis test untuk controller dengan mock repository:
   ```php
   $repoMock = $this->createMock(ProductRepository::class);
   $repoMock->method('getActive')->willReturn(collect([]));
   ```

5. **Keputusan.** Menurutmu, apakah codebase ini perlu repository? Tulis pro dan kontra.

---

## 🔗 Referensi

- [Martin Fowler: Repository](https://martinfowler.com/eaaCatalog/repository.html)
- [Laravel Docs: Architecture](https://laravel.com/docs/11.x/architecture)
- [Laravel News: Repository Pattern](https://laravel-news.com/repository-pattern-in-laravel)
- Codebase: `app/Models/` — Eloquent langsung (tanpa repository)
