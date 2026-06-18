# 04-04: Functional di Laravel — Collection, Pipeline, Fluent

> **Fase**: 4 — Functional Programming  
> **Prasyarat**: 04-03-array-map-filter-reduce  
> **Waktu baca**: 75-90 menit  
> **Kata kunci**: Collection, pipeline, pipe, tap, fluent string, higher-order message, lazy collection, macroable

---

## 📋 Ringkasan

Tiga dokumen sebelumnya membangun fondasi functional programming di PHP. Sekarang kita lihat **bagaimana Laravel mengadopsi FP** sebagai inti framework-nya.

Laravel bukan framework functional murni — tapi ia **meminjam ide-ide functional** di tempat yang tepat:
- **Collection** → map/filter/reduce/pipeline untuk array
- **Fluent Strings** → method chaining immutable untuk string
- **Pipeline** → middleware sebagai function composition
- **Higher-Order Messages** → method call via collection proxy
- **Macroable** → menambah method ke inti framework di runtime

**Target pemahaman:**
- Kamu bisa menggunakan Collection untuk operasi data sehari-hari
- Kamu paham kapan Collection lebih cocok daripada array biasa
- Kamu bisa membuat pipeline dengan `pipe()` dan `tap()`
- Kamu paham Fluent Strings dan Higher-Order Messages

---

## 1. Laravel Collection — Functional Toolkit Utama

### 1.1 Apa Itu Collection?

Collection adalah **wrapper** OOP di atas array PHP yang menyediakan method functional.

```php
<?php
// Daripada array PHP
$items = [
    ['name' => 'Laptop', 'price' => 15000],
    ['name' => 'Mouse',  'price' => 250],
];

// Manual — filter + map via native functions (baca dari dalam ke luar)
$result = array_map(
    fn($i) => $i['name'],
    array_filter($items, fn($i) => $i['price'] > 1000)
);

// Collection — baca dari atas ke bawah
$result = collect($items)
    ->filter(fn($i) => $i['price'] > 1000)
    ->map(fn($i) => $i['name'])
    ->values();
```

**Kenapa Collection lebih baik?**
1. **Method chaining** — baca kiri-ke-kanan, atas-ke-bawah
2. **Immutability** — method tidak mengubah array asli
3. **300+ method** — `sum()`, `avg()`, `groupBy()`, `pluck()`, `firstWhere()`, dll
4. **Lazy** — `LazyCollection` untuk dataset besar

### 1.2 Membuat Collection

```php
<?php
// Dari array
$collection = collect([1, 2, 3]);

// Dari query Eloquent (SUDAH Collection!)
$users = User::all(); // Collection of User models

// Dari query Builder (bukan Collection — dapat di-loop)
$users = User::where('active', true)->get(); // Collection

// Dengan range
$collection = collect(range(1, 100));

// Dengan times (factory helper)
$collection = Collection::times(5, fn($n) => $n * 2);
// [2, 4, 6, 8, 10]
```

### 1.3 Method Collection Paling Penting

| Method | Guna | Mirip Native |
|--------|------|-------------|
| `->map(fn)` | Transformasi setiap elemen | `array_map` |
| `->filter(fn)` | Saring elemen | `array_filter` |
| `->reduce(fn, init)` | Akumulasi ke satu nilai | `array_reduce` |
| `->sum('field')` | Jumlah nilai | `array_sum` |
| `->avg('field')` | Rata-rata | — |
| `->pluck('name')` | Ambil satu kolom | `array_column` |
| `->groupBy('cat')` | Grouping | — |
| `->sortBy('field')` | Sortir | `sort` / `usort` |
| `->firstWhere('k', 'v')` | Cari pertama | `array_filter` + `reset` |
| `->where('k', 'v')` | Filter key-value | `array_filter` |
| `->keys()` | Ambil semua key | `array_keys` |
| `->values()` | Reset index | `array_values` |
| `->unique()` | Hapus duplikat | `array_unique` |
| `->collapse()` | Gabung nested array | `array_merge(...$arr)` |
| `->chunk(n)` | Potong-potong array | `array_chunk` |
| `->each(fn)` | Loop tanpa return | `foreach` |
| `->pipe(fn)` | Kirim ke fungsi lain | — |
| `->tap(fn)` | Side-effect tanpa ubah | — |
| `->dd()` | Dump & die | `dd()` |
| `->dump()` | Dump & lanjut | `dump()` |

### 1.4 Collection di Codebase

**ShopController.php:57 — `get()` return Collection:**

```php
$categories = Category::where('is_active', true)
    ->whereNull('parent_id')
    ->orderBy('sort_order')
    ->get();
// $categories adalah Collection of Category models
```

**ShopController.php:62 — `firstWhere` Collection method:**

```php
$selectedCategory = $request->filled('category')
    ? $categories->firstWhere('slug', $request->category)
    : null;
```

**CartService.php — Collection dari array session:**

```php
// CartService.getCart() return collect($cart)
// Jadi method-method Collection bisa dipanggil setelahnya
```

---

## 2. Method Chaining — Pipeline Functional

### 2.1 Chaining di Query Builder

Ini adalah contoh pipeline functional paling jelas di Laravel:

```php
<?php
// ShopController.php
$products = Product::with(['category', 'brand'])
    ->where('is_active', true)                          // filter aktif
    ->when($request->filled('category'), function ($q) { // filter kategori
        $q->whereHas('category', fn($q) => $q->where('slug', $request->category));
    })
    ->when($request->filled('search'), function ($q) {    // filter search
        $q->where(function ($q) use ($request) {
            $q->where('name', 'like', "%{$request->search}%")
              ->orWhere('short_description', 'like', "%{$request->search}%");
        });
    })
    ->when($request->filled('in_stock'), function ($q) {  // filter stok
        $q->where('stock_quantity', '>', 0);
    })
    ->orderBy(...)                                        // sorting
    ->paginate(12);                                       // eksekusi
```

Setiap method **mengembalikan object yang sama** (`$q`), jadi method berikutnya dipanggil di atasnya. Ini mirip dengan pipeline functional:

```
input → where → when → when → orderBy → paginate → output
```

### 2.2 Chaining di Collection

```php
<?php
$result = collect($products)
    ->filter(fn($p) => $p['stock'] > 0)        // saring
    ->sortByDesc('price')                        // urutkan
    ->take(5)                                    // ambil 5 teratas
    ->map(fn($p) => [                            // transform
        'name'  => $p['name'],
        'price' => number_format($p['price']),
    ])
    ->values();                                  // reset key
```

### 2.3 Kenapa Ini Functional?

**Immutability:** Setiap method mengembalikan **Collection baru**, tidak mengubah yang asli:

```php
<?php
$original = collect([1, 2, 3, 4, 5]);
$filtered = $original->filter(fn($n) => $n > 2);
$mapped = $filtered->map(fn($n) => $n * 10);

dump($original); // [1, 2, 3, 4, 5] — TIDAK BERUBAH!
dump($filtered); // [3, 4, 5]
dump($mapped);   // [30, 40, 50]
```

---

## 3. Pipe dan Tap — Functional Composition

### 3.1 `pipe()` — Kirim Collection ke Fungsi Lain

`pipe()` mengirim collection ke **fungsi eksternal** dan mengembalikan hasil fungsi itu.

```php
<?php
$collection = collect([1, 2, 3, 4, 5]);

$result = $collection
    ->filter(fn($n) => $n > 2)
    ->pipe(fn($coll) => $coll->sum());
// Hasil: 12 (3 + 4 + 5)

// pipe() bisa untuk validasi atau transformasi kompleks
$validated = collect($items)
    ->pipe(function ($items) {
        // Validasi: throw jika ada item tanpa nama
        throw_if($items->contains(fn($i) => empty($i['name'])),
            ValidationException::class);
        return $items;
    })
    ->map(fn($i) => $i['name']);
```

### 3.2 `tap()` — Side-Effect Tanpa Mengubah Collection

`tap()` mirip `pipe()` tapi **mengembalikan collection asli**, bukan hasil fungsi.

```php
<?php
$collection = collect([1, 2, 3]);

$result = $collection
    ->filter(fn($n) => $n > 1)
    ->tap(fn($coll) => logger('Filtered: ' . $coll->toJson())) // logging
    ->map(fn($n) => $n * 10);
// tap() hanya untuk side-effect — log, cache, dump
// $result tetap hasil map, bukan hasil tap
```

**Guna `tap()`:**
- **Logging** — log state di tengah pipeline
- **Caching** — simpan ke cache tanpa menghentikan pipeline
- **Debugging** — `->tap(fn($c) => dump($c->toArray()))`
- **Side-effect** — kirim email, update statistik

### 3.3 `pipe()` vs `tap()`

| | `pipe()` | `tap()` |
|---|---------|--------|
| Return | Hasil fungsi | Collection asli |
| Guna | Transformasi keluar | Side-effect |
| Contoh | `->pipe(fn($c) => $c->sum())` | `->tap(fn($c) => Log::info(...))` |

---

## 4. Higher-Order Messages — PHP 7.4+ / Laravel 6+

### 4.1 Masalah

Sebelum Higher-Order Messages:

```php
<?php
$names = collect($users)->map(function ($user) {
    return $user->name;
});
```

### 4.2 Solusi — Higher-Order Proxy

Laravel Collection memiliki **HigherOrderCollectionProxy** yang membiarkan method dipanggil langsung di koleksi tanpa closure.

```php
<?php
// Daripada:
$names = $users->map(fn($user) => $user->name);
$emails = $users->map(fn($user) => $user->email);
$active = $users->filter(fn($user) => $user->isActive());

// Pakai Higher-Order Messages:
$names  = $users->map->name;       // panggil ->name di setiap user
$emails = $users->map->email;      // panggil ->email
$active = $users->filter->isActive(); // panggil ->isActive()
```

Ini bekerja karena Laravel menggunakan **__get** dan **__call** magic methods. Saat kamu akses `$users->map->name`:
1. `map` mengembalikan **HigherOrderCollectionProxy**
2. `->name` memicu `__get` — proxy tahu artinya "panggil `name` di setiap item"
3. Proxy menjalankan `$users->map(fn($item) => $item->name)`

### 4.3 Higher-Order Array Access

```php
<?php
// Dengan array, bukan object:
$items = collect([
    ['name' => 'Laptop', 'price' => 15000],
    ['name' => 'Mouse',  'price' => 250],
]);

$names = $items->map->name;          // ❌ Error — name bukan method
$names = $items->pluck('name');      // ✅ pluck untuk array key
```

Higher-Order Messages bekerja untuk **method dan property**, bukan array key. Untuk array key, pakai `pluck()`.

### 4.4 Method yang Support Higher-Order Messages

| Method | Higher-Order | Contoh |
|--------|-------------|--------|
| `map` | `$c->map->method()` | `$c->map->toArray()` |
| `filter` | `$c->filter->isActive()` | `$c->filter->isNotEmpty()` |
| `each` | `$c->each->notify()` | `$c->each->delete()` |
| `reject` | `$c->reject->isAdmin()` | `$c->reject->deleted()` |
| `first` | `$c->first->name` | — |

---

## 5. Fluent Strings — Functional untuk String

### 5.1 Masalah

Operasi string di PHP imperatif — setiap fungsi menerima string, mengembalikan string baru:

```php
<?php
$slug = 'Hello World from Laravel';
$slug = strtolower($slug);        // "hello world from laravel"
$slug = str_replace(' ', '-', $slug); // "hello-world-from-laravel"
$slug = substr($slug, 0, 20);     // "hello-world-from-la"

// Baca dari bawah ke atas — merepotkan!
```

### 5.2 Solusi — `Str::of()`

Laravel `Str::of()` membungkus string dalam object **Fluent String** yang memiliki method-method functional:

```php
<?php
$slug = Str::of('Hello World from Laravel')
    ->lower()
    ->replace(' ', '-')
    ->substr(0, 20)
    ->toString();
// Baca dari atas ke bawah — alami!

// Atau biarkan sebagai object — toString() otomatis di echo
echo Str::of('HELLO')->lower()->ucfirst(); // "Hello"
```

### 5.3 Method Fluent String

```php
<?php
$result = Str::of('  Laravel Framework  ')
    ->trim()                          // "Laravel Framework"
    ->lower()                         // "laravel framework"
    ->replace('framework', 'PHP')     // "laravel PHP"
    ->ucfirst()                       // "Laravel PHP"
    ->slug()                          // "laravel-php"
    ->limit(10)                       // "laravel ph"
    ->toString();                     // kembalikan string

// Method umum:
Str::of('hello')->upper();           // "HELLO"
Str::of('hello')->ucfirst();         // "Hello"
Str::of('hello world')->title();     // "Hello World"
Str::of('hello')->append(' world');  // "hello world"
Str::of('hello')->prepend('say ');   // "say hello"
Str::of('HELLO')->lower();           // "hello"
Str::of('hello')->repeat(3);         // "hellohellohello"
Str::of('hello')->limit(3);          // "hel"
Str::of('Hello World')->slug();     // "hello-world"
Str::of('admin')->is('admin');      // true
Str::of('user@example.com')->isEmail(); // true
Str::of('hello')->contains('ell');  // true
```

### 5.4 Fluent String di Codebase

**Setting.php — transform path:**

```php
// settings('app.discount_id') di baca sebagai string lalu di-parse
// detail ada di Setting model atau helper
```

**Str::slug() digunakan di Route model binding:**

```php
// Routes otomatis slug → parameter
// Product::where('slug', $slug)->first()
```

### 5.5 Conditional Method Chaining

```php
<?php
$name = Str::of($rawName)
    ->when(
        Str::isUuid($rawName),
        fn($str) => $str->after('-')->headline(),
        fn($str) => $str->headline(),
    )
    ->toString();
```

---

## 6. Pipeline — Middleware sebagai Function Composition

### 6.1 Konsep Pipeline

Pipeline Laravel mengambil input, melewatkannya melalui serangkaian **pipe** (callable), dan mengembalikan hasil akhir.

```
Input → [Pipe1] → [Pipe2] → [Pipe3] → Output
              ↓         ↓         ↓
         (modify)   (modify)   (modify)
```

### 6.2 Pipeline di Middleware

Setiap request HTTP melewati pipeline middleware:

```php
// Kernel.php — pipeline middleware global
protected $middleware = [
    \App\Http\Middleware\TrustProxies::class,
    \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
];
```

Setiap middleware adalah **pipe** — menerima request, melakukan sesuatu, lalu melewatkan ke pipe berikutnya:

```php
<?php
class TrimStrings
{
    public function handle($request, Closure $next)
    {
        // Before — modifikasi request
        $request->merge(...);

        // Lanjut ke middleware berikutnya
        $response = $next($request);

        // After — modifikasi response
        return $response;
    }
}
```

### 6.3 Pipeline Manual

```php
<?php
use Illuminate\Pipeline\Pipeline;

$result = app(Pipeline::class)
    ->send('initial data')              // input
    ->through([                          // pipes
        function ($data, $next) {
            $data .= ' → pipe 1';
            return $next($data);
        },
        function ($data, $next) {
            $data .= ' → pipe 2';
            return $next($data);
        },
    ])
    ->then(fn($data) => $data . ' → final'); // output

echo $result; // "initial data → pipe 1 → pipe 2 → final"
```

### 6.4 Pipeline di Codebase

Laravel sendiri menggunakan pipeline untuk:
1. **Global middleware stack** — semua request lewat sini
2. **Route middleware** — middleware spesifik per route
3. **Controller middleware** — `$this->middleware('auth')`
4. **Validation** — validated data melalui pipeline

Kamu bisa lihat middleware di `app/Http/Kernel.php`:

```php
protected $routeMiddleware = [
    'auth'          => \App\Http\Middleware\Authenticate::class,
    'verified'      => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    'role'          => \App\Http\Middleware\CheckRole::class,
    'throttle'      => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

---

## 7. LazyCollection — Functional untuk Data Besar

### 7.1 Masalah

Collection biasa memuat **semua data ke memory**:

```php
<?php
// Semua 1 juta user dimuat ke memory!
$users = User::all()->filter(fn($u) => $u->is_active);
```

### 7.2 Solusi — LazyCollection

`LazyCollection` memproses data **satu per satu** (lazy evaluation) — hanya memuat elemen yang diperlukan:

```php
<?php
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('large-file.csv', 'r');
    while (($line = fgets($handle)) !== false) {
        yield $line; // yield — PHP generator
    }
})
->map(fn($line) => str_getcsv($line))
->filter(fn($row) => $row[2] === 'active')
->each(fn($row) => ProcessUser::dispatch($row));
```

**Generator (yield)** di PHP memungkinkan fungsi "mengingat" state-nya dan melanjutkan dari tempat terakhir — ini lazy evaluation.

### 7.3 Lazy Chunk di Eloquent

Eloquent `cursor()` mengembalikan LazyCollection:

```php
<?php
foreach (User::where('is_active', true)->cursor() as $user) {
    // Hanya SATU user di memory setiap iterasi
    ProcessUser::dispatch($user);
}
```

### 7.4 Chunking dengan `chunk()`

```php
<?php
// chunk — proses 100 data per query
User::where('is_active', true)->chunk(100, function ($users) {
    foreach ($users as $user) {
        ProcessUser::dispatch($user);
    }
});
```

---

## 8. Macroable — Extending Inti Framework

### 8.1 Konsep

Laravel class seperti `Collection`, `Str`, `Request`, `Response` menggunakan **Macroable** trait — memungkinkan method baru ditambahkan di runtime.

```php
<?php
// Menambah method ke Collection
Collection::macro('toUpperCase', function () {
    return $this->map(fn($item) => strtoupper($item));
});

collect(['a', 'b', 'c'])->toUpperCase();
// ['A', 'B', 'C']
```

### 8.2 Macroable di Codebase

**AppServiceProvider.php — register macros di sini:**

```php
<?php
// Biasanya di boot() method AppServiceProvider
public function boot(): void
{
    Str::macro('phone', function (string $value): string {
        // Format nomor telepon
        return preg_replace('/^0/', '+62', $value);
    });

    // Contoh: Response macro
    Response::macro('api', function (mixed $data, int $code = 200) {
        return response()->json([
            'success' => $code < 400,
            'data'    => $data,
        ], $code);
    });
}
```

### 8.3 Macroable di Framework

Kelas yang menggunakan **Macroable**:
- `Illuminate\Support\Collection`
- `Illuminate\Support\Str`
- `Illuminate\Http\Request`
- `Illuminate\Routing\ResponseFactory`
- `Illuminate\Routing\Route`
- `Illuminate\Routing\UrlGenerator`

---

## 9. Functional Helpers Laravel

### 9.1 `data_get()` — Get Nested Data

```php
<?php
$data = [
    'user' => [
        'profile' => [
            'name' => 'Budi',
        ],
    ],
];

// Daripada: $data['user']['profile']['name'] ?? null
$name = data_get($data, 'user.profile.name'); // "Budi"
$age  = data_get($data, 'user.profile.age', 0); // 0 (default)

// Wildcard — semua nama dari semua user
$users = [
    ['name' => 'Budi', 'posts' => [['title' => 'A'], ['title' => 'B']]],
    ['name' => 'Siti', 'posts' => [['title' => 'C']]],
];
$titles = data_get($users, '*.posts.*.title');
// [['A', 'B'], ['C']]
```

### 9.2 `data_set()` — Set Nested Data

```php
<?php
$data = ['user' => ['name' => 'Budi']];
data_set($data, 'user.age', 25);
// ['user' => ['name' => 'Budi', 'age' => 25]]

data_set($data, 'user.addr.city', 'Jakarta');
// ['user' => ['name' => 'Budi', 'age' => 25, 'addr' => ['city' => 'Jakarta']]]
```

### 9.3 `data_fill()` — Fill if Not Set

```php
<?php
$data = ['user' => ['name' => 'Budi']];
data_fill($data, 'user.name', 'Siti');     // tetap 'Budi' — sudah ada
data_fill($data, 'user.age', 25);           // 'age' => 25 — baru diisi
```

### 9.4 `head()` dan `last()`

```php
<?php
$array = [10, 20, 30, 40];
head($array);  // 10
last($array);  // 40
```

### 9.5 `value()` — Callable atau Nilai

```php
<?php
value('hello');                    // 'hello'
value(fn() => 'hello');           // 'hello' — dipanggil jika callable
value(fn() => User::count(), 0);  // jumlah user atau 0
```

### 9.6 `tap()` (global helper)

```php
<?php
// Helper global tap() — berguna untuk side-effect
$user = tap(User::find(1), function ($user) {
    $user->update(['last_login' => now()]);
});
// tap() mengembalikan $user, bukan hasil update
```

---

## 10. Ringkasan: Functional Laravel Map

```
┌─────────────────────────────────────────────────────────────────────┐
│ LARAVEL FUNCTIONAL FEATURES                                        │
├─────────────────────────────────────────────────────────────────────┤
│ COLLECTION          │ STR OF           │ PIPELINE                   │
│ collect([...])      │ Str::of('...')   │ Pipeline::                 │
│   ->filter(fn)      │   ->trim()        │   send($req)              │
│   ->map(fn)         │   ->lower()       │   ->through($pipes)       │
│   ->pipe(fn)        │   ->slug()        │   ->then(fn)              │
│   ->sum()           │   ->toString()    │                           │
├─────────────────────────────────────────────────────────────────────┤
│ HIGHER-ORDER        │ LAZY COLLECTION   │ MACROABLE                 │
│ $c->map->name       │ cursor()          │ Collection::macro()       │
│ $c->filter->active  │ yield             │ Str::macro()              │
│ $c->each->delete    │ chunk(100)        │ Response::macro()         │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE:                                                        │
│  collect() → CartService, ShopController                            │
│  get()     → ShopController::categories Collection                  │
│  firstWhere → ShopController::selectedCategory                      │
│  when()    → ShopController::query builder                          │
│  Str::slug() → route model binding                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Buat pipeline Collection.** Ambil array produk, filter yang `is_active`, sort by `price` descending, ambil 5 termahal, ambil hanya `name` dan `price`.

2. **Gunakan `pipe()`.** Dari hasil nomor 1, gunakan `pipe()` untuk mengubah array menjadi JSON string.

3. **Fluent String.** Gunakan `Str::of()` untuk mengubah "   laravel PHP framework   " menjadi "Laravel-PHP-Framework".

4. **Higher-Order Message.** Jika `$users` adalah Collection of User model dengan method `isAdmin()`, filter admin menggunakan higher-order messages.

5. **Eksplorasi.** Buka `app/Providers/AppServiceProvider.php`. Apakah ada macro yang didaftarkan? Jika ya, apa fungsinya?

6. **Pipeline.** Buka `app/Http/Kernel.php`. Identifikasi middleware global dan route middleware. Cari middleware `CheckRole` dan jelaskan apa yang dilakukannya.

---

## 🔗 Referensi

- [Laravel Docs: Collections](https://laravel.com/docs/collections)
- [Laravel Docs: Fluent Strings](https://laravel.com/docs/strings)
- [Laravel Docs: HTTP Middleware](https://laravel.com/docs/middleware)
- [Laravel Docs: Pipeline](https://laravel.com/docs/helpers#pipeline)
- [PHP Manual: Generators](https://www.php.net/manual/en/language.generators.overview.php)
- Codebase: `app/Http/Kernel.php` — middleware pipeline
- Codebase: `app/Http/Controllers/ShopController.php` — Collection + query builder pipeline
- Codebase: `app/Providers/AppServiceProvider.php` — macro registration
