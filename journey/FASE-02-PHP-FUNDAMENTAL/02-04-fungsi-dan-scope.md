# 02-04: Fungsi dan Variable Scope

> **Fase**: 2 — PHP Fundamental  
> **Prasyarat**: 02-03-array-dan-string  
> **Waktu baca**: 75-90 menit  
> **Kata kunci**: function, parameter, return type, type hint, variadic, named argument, scope, closure, arrow function, anonymous function, callback, callable, recursion

---

## 📋 Ringkasan

Fungsi adalah **blok kode yang bisa dipanggil berkali-kali** dengan input berbeda. Ini adalah fondasi dari **setiap** aplikasi — Laravel sendiri adalah kumpulan fungsi yang disebut "method".

Setelah dokumen ini, kamu akan:
- Menulis fungsi dengan parameter dan return type yang benar
- Memahami perbedaan scope global, lokal, dan static
- Menguasai closure dan arrow function
- Bisa membaca dan menulis kode dengan dependency injection
- Paham bagaimana fungsi dipakai di seluruh codebase ini

---

## 1. Anatomi Fungsi

### 1.1 Definisi Paling Dasar

```
function namaFungsi(parameter): returnType {
    // body
    return nilai;
}
```

```php
<?php

function sapa(string $nama): string
{
    return "Halo, {$nama}!";
}

echo sapa("Budi"); // "Halo, Budi!"
```

**Apa yang terjadi di memori saat fungsi dipanggil:**
1. PHP membuat **stack frame** (call frame) di memory stack
2. Parameter `$nama` dialokasikan di frame ini — nilainya `"Budi"`
3. Eksekusi lompat ke body fungsi
4. String `"Halo, Budi!"` dibuat di heap
5. Nilai dikembalikan, stack frame dihapus
6. Eksekusi kembali ke baris setelah `sapa("Budi")`

### 1.2 Fungsi di Codebase

Di `app/Models/Order.php:59-68`:

```php
public function getStatusLabelAttribute(): string
{
    return match ($this->status) {
        'pending'    => 'Menunggu Pembayaran',
        'processing' => 'Diproses',
        'shipped'    => 'Dikirim',
        'completed'  => 'Selesai',
        'cancelled'  => 'Dibatalkan',
        default      => ucfirst($this->status),
    };
}
```

Analisis:
- `public` — visibility (bisa dipanggil dari mana saja)
- `function` — kata kunci deklarasi
- `getStatusLabelAttribute` — nama fungsi (camelCase)
- `: string` — return type declaration
- `match (...) {...}` — body fungsi, mengembalikan string

### 1.3 `function` vs `method`

Dalam OOP, **method** adalah fungsi yang menjadi **anggota sebuah class**.

```
Fungsi biasa: function hitungTotal($items) { ... }
Method:       public function hitungTotal($items) { ... }
```

Di codebase ini, hampir semua fungsi adalah **method** — karena Laravel adalah framework OOP. Fungsi "biasa" (procedural) biasanya hanya di file helper atau `routes/web.php`.

---

## 2. Visibility: public, protected, private

### 2.1 Perbedaan

| Visibility | Class sendiri | Subclass (turunan) | Dunia luar |
|------------|:---:|:---:|:---:|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

### 2.2 Contoh dari Codebase

**`RajaOngkirService.php`** — contoh sempurna perbedaan visibility:

```php
class RajaOngkirService
{
    // PUBLIC — API yang bisa dipanggil dari controller mana pun
    public function getProvinces(): array       { ... }
    public function getCities($provinceId = null): array { ... }
    public function calculateCost($origin, $destination, int $weight, string $courier): array { ... }

    // PRIVATE — hanya untuk internal class ini, tidak untuk dipanggil dari luar
    private function getDummyProvinces(): array  { ... }
    private function getDummyCities(...): array  { ... }
    private function getDummyCost($courier): array { ... }
}
```

**Kenapa private?** Karena `getDummyProvinces()` adalah **fallback** — hanya dipakai kalau API RajaOngkir gagal. Tidak ada alasan bagi controller untuk memanggil dummy data secara langsung.

### 2.3 Aturan Praktis

- `public` — untuk method yang merupakan **kontrak publik** class
- `protected` — untuk method yang dipakai oleh subclass (jarang di codebase ini)
- `private` — untuk **detail implementasi internal**, helper kecil

---

## 3. Parameter

### 3.1 Type Declaration pada Parameter

PHP 7.0+ mendukung tipe pada parameter. PHP akan **melempar TypeError** jika tipe tidak cocok.

```php
<?php

// String parameter
public function getSnapToken(Order $order): string
// ↑ $order harus instance dari class Order

// Int + string parameter
public function calculateCost($origin, $destination, int $weight, string $courier): array
// ↑ $weight harus int, $courier harus string

// Object parameter (PHP 8.0+)
public function via(object $notifiable): array
// ↑ $notifiable bisa instance class apa pun
```

**Di codebase — MidtransService.php:20:**
```php
public function getSnapToken(Order $order): string
```

Laravel's service container membaca type hint ini untuk **auto-inject dependency**. Saat method dipanggil oleh Laravel (misal dari controller), framework otomatis mencari instance `Order` yang sesuai dan memasukkannya.

### 3.2 Default Parameter

Parameter dengan nilai default **tidak wajib** diisi saat pemanggilan.

**Di codebase — CartService.php:17:**
```php
public function add($productId, $quantity = 1)
```

Pemanggilan valid:
```php
$cartService->add(5);       // quantity = 1
$cartService->add(5, 3);    // quantity = 3
```

**Di codebase — RajaOngkirService.php:54:**
```php
public function getCities($provinceId = null): array
```

Pemanggilan valid:
```php
$service->getCities();          // $provinceId = null → semua kota
$service->getCities(5);         // $provinceId = 5 → filter provinsi
```

**Aturan:** Parameter dengan default harus **setelah** parameter tanpa default.

```php
// ✅ BENAR
function f($required, $optional = null) {}

// ❌ SALAH — parameter required setelah optional
function f($optional = null, $required) {}
```

### 3.3 Variadic Parameter (`...$args`)

`...$args` mengumpulkan **semua argumen tambahan** menjadi satu array.

**Di codebase — RoleMiddleware.php:16:**
```php
public function handle(Request $request, Closure $next, ...$roles): Response
{
    foreach ($roles as $role) {
        if ($request->user()->roles()->where('slug', $role)->exists()) {
            return $next($request);
        }
    }
    abort(403, 'Unauthorized action.');
}
```

**Cara kerja:**
- Middleware dipanggil dengan: `->middleware('role:admin,superadmin')`
- Laravel memecah string menjadi: `['admin', 'superadmin']`
- `...$roles` menangkap: `$roles = ['admin', 'superadmin']`

**Aturan variadic:**
- Hanya boleh **satu** parameter variadic per fungsi
- Harus **paling akhir** dalam daftar parameter
- Bisa dikombinasikan dengan type hint: `public function f(string ...$names)`

**Spread operator (kebalikan):**

```php
<?php
function tambah(int $a, int $b, int $c): int
{
    return $a + $b + $c;
}

$angka = [1, 2, 3];
echo tambah(...$angka); // 6 — array di-spread jadi argumen individual
```

### 3.4 Named Arguments (PHP 8.0+)

Named argument memungkinkan kamu **menyebut nama parameter** saat memanggil fungsi — tidak perlu mengikuti urutan.

**Di codebase — RegisteredUserController.php:56:**
```php
return redirect(route('dashboard', absolute: false));
```

Tanpa named argument: `route('dashboard', [], false, null)` — harus hafal posisi parameter.
Dengan named argument: `route('dashboard', absolute: false)` — hanya sebut yang perlu diubah.

**Keuntungan:**
1. **Skip parameter default** — tidak perlu passing `null` untuk parameter yang tidak dipakai
2. **Self-documenting** — jelas parameter mana yang diubah
3. **Tidak tergantung urutan** — parameter bisa ditulis dalam urutan apa pun

```php
<?php
function buatOrder(
    string $customer,
    array $items,
    string $courier = 'jne',
    bool $insurance = false,
    string $notes = ''
) {}

// Tanpa named argument:
buatOrder('Budi', $items, 'jne', false, 'Cepat');

// Dengan named argument — lebih jelas:
buatOrder('Budi', $items, notes: 'Cepat');
// Atau bahkan:
buatOrder(notes: 'Cepat', customer: 'Budi', items: $items);
```

**Peringatan:** Named argument tidak bisa dipakai jika parameter dideklarasikan dengan `...` (variadic).

---

## 4. Return Type

### 4.1 Return Type Declaration

Setiap fungsi bisa mendeklarasikan tipe kembalian dengan `: tipe`.

```php
<?php
public function getProvinces(): array    // harus return array
public function getSnapToken(Order $order): string  // harus return string
public function view(User $user, Order $order): bool  // harus return bool
public function updating(Order $order): void  // tidak return apa pun
public function create(): View  // return View object (Laravel)
```

**Void** — fungsi yang tidak mengembalikan nilai:
```php
public function register(): void
{
    // ... kode di sini, tanpa return
}
```

### 4.2 Union Type (PHP 8.0+)

Parameter/return bisa menerima **lebih dari satu tipe**.

**Di codebase — Setting.php:23:**
```php
public static function get(string $key, mixed $default = null): mixed
```

`mixed` adalah union type yang setara dengan `array|bool|callable|int|float|null|object|resource|string` — artinya **apa pun**.

Contoh union type eksplisit:
```php
<?php
function findProduct(int|string $id): Product|null
{
    // $id bisa int (ID) atau string (slug)
    // return bisa Product atau null
}
```

### 4.3 `void` dan `never`

- `void` — fungsi selesai tanpa return (secara implisit return null)
- `never` — fungsi **tidak pernah** selesai (selalu throw exception atau die)

```php
<?php
function logError(string $msg): void
{
    // tidak perlu return
}

function abortIfInvalid(bool $condition): never
{
    if (!$condition) {
        throw new \InvalidArgumentException('Invalid');
    }
}
```

---

## 5. Variable Scope

### 5.1 Global Scope

Variabel yang dideklarasikan **di luar fungsi** berada di global scope.

```php
<?php

$globalVar = "Saya global"; // ← global scope

function test(): void
{
    // ❌ $globalVar TIDAK bisa diakses di sini!
    // Setiap fungsi punya scope sendiri
}
```

### 5.2 Local Scope

Variabel yang dideklarasikan **di dalam fungsi** hanya hidup di fungsi itu.

```php
<?php
function hitungPajak(int $harga): int
{
    $pajak = $harga * 0.11; // ← local scope
    return $pajak;
    // setelah return, $pajak dihapus dari memori
}

// echo $pajak; // ❌ Error: undefined variable
```

### 5.3 `global` Keyword (Jangan Dipakai)

PHP punya `global` untuk mengimpor variabel global ke fungsi — tapi **jangan lakukan ini**.

```php
<?php
$diskon = 0.1; // global

function hitungHarga(int $harga): int
{
    global $diskon; // ← ambil dari global scope
    return $harga - ($harga * $diskon);
}
```

**Kenapa jangan pakai `global`?**
1. Fungsi jadi **tidak pure** — output tergantung state global
2. Sulit dites — kamu harus set global dulu sebelum test
3. Sulit dilacak — siapa yang mengubah `$diskon`?
4. **Di codebase ini tidak ada satu pun `global`** — dan itu benar

**Solusi yang benar:** Lewatkan sebagai parameter.

```php
<?php
function hitungHarga(int $harga, float $diskon = 0.1): int
{
    return $harga - ($harga * $diskon);
}
```

### 5.4 Static Variable

Variabel static **mempertahankan nilainya** antar pemanggilan fungsi — tidak dihapus setelah fungsi selesai.

```php
<?php
function counter(): int
{
    static $count = 0; // hanya di-set sekali, dipertahankan
    $count++;
    return $count;
}

echo counter(); // 1
echo counter(); // 2
echo counter(); // 3
```

**Di memori:** Variabel static disimpan di **segmen data** (bukan stack), jadi tidak hilang saat stack frame dihapus.

**Di codebase** — tidak ada static variable di fungsi biasa. Tapi class menggunakan properti static:

```php
// Setting.php
public static function get(string $key, mixed $default = null): mixed
{
    $setting = static::where('key', $key)->first();
    // ...
}
```

Properti/method `static` milik class, bukan instance. Tidak perlu `new Setting()` untuk memanggil `Setting::get('key')`.

### 5.5 Static Method vs Instance Method

```php
<?php
class Calculator
{
    // Static method — bisa dipanggil tanpa instance
    public static function tambah(int $a, int $b): int
    {
        return $a + $b;
    }

    // Instance method — butuh instance
    public function kurangi(int $a, int $b): int
    {
        return $a - $b;
    }
}

// Static call
echo Calculator::tambah(1, 2); // 3

// Instance call
$calc = new Calculator();
echo $calc->kurangi(5, 3); // 2
```

**`self::` vs `static::` (Late Static Binding):**

```php
<?php
class Parent_
{
    public static function who(): string
    {
        return 'Parent';
    }

    public static function testSelf(): string
    {
        return self::who(); // ← resolves to Parent_::who()
    }

    public static function testStatic(): string
    {
        return static::who(); // ← resolves at runtime
    }
}

class Child extends Parent_
{
    public static function who(): string
    {
        return 'Child';
    }
}

echo Child::testSelf();   // "Parent" — self:: mengunci ke class tempat kode ditulis
echo Child::testStatic(); // "Child" — static:: ditentukan saat runtime
```

**Di codebase — Setting.php menggunakan `static::`:**

```php
public static function get(string $key, mixed $default = null): mixed
{
    $setting = static::where('key', $key)->first();
    // ...
}
```

Kenapa bukan `self::where(...)`? Karena kalau ada subclass `ExtendedSetting extends Setting`, `static::where(...)` akan query di tabel `extended_settings`. **Late static binding** memastikan class yang benar dipakai.

---

## 6. Constructor dan Dependency Injection

### 6.1 Constructor

`__construct()` adalah method spesial yang dipanggil **otomatis** saat objek dibuat.

**Di codebase — CheckoutController.php:19-29:**
```php
public function __construct(
    CartService $cartService,
    AddressService $addressService,
    OrderService $orderService,
    \App\Services\RajaOngkirService $rajaOngkirService
) {
    $this->cartService = $cartService;
    $this->addressService = $addressService;
    $this->orderService = $orderService;
    $this->rajaOngkirService = $rajaOngkirService;
}
```

**Dependency Injection (DI):** Parameter constructor adalah "dependencies" yang dibutuhkan class. Laravel secara otomatis:
1. Melihat type hint di constructor
2. Membuat instance setiap service yang dibutuhkan
3. Memanggil `new CheckoutController($cartService, $addressService, ...)`

Tanpa DI, programmer harus manual:
```php
// ❌ TANPA DI — manual, kaku
$controller = new CheckoutController(
    new CartService(),
    new AddressService(),
    new OrderService(),
    new RajaOngkirService()
);
```

Dengan DI, programmer tinggal:
```php
// ✅ DENGAN DI — Laravel urus semuanya
// Route::get('/checkout', [CheckoutController::class, 'index']);
```

### 6.2 Properti (`$this->property`)

Method instance mengakses properti melalui `$this`:

```php
<?php

class CartService
{
    protected $sessionKey = 'shopping_cart'; // properti

    public function getCart(): array
    {
        return Session::get($this->sessionKey, []); // ← pakai $this->
    }

    public function clear(): void
    {
        Session::forget($this->sessionKey); // ← pakai $this->
    }
}
```

**Key concept:** `$this` selalu merujuk ke **instance saat ini** dari class. Bukan class-nya, bukan instance lain — tapi persis objek yang method-nya sedang dieksekusi.

---

## 7. Anonymous Function (Closure)

### 7.1 Closure Dasar

Closure adalah fungsi **tanpa nama** yang bisa disimpan di variabel atau dilewatkan sebagai argument.

```php
<?php

// Closure disimpan di variabel
$sapa = function (string $nama): string {
    return "Halo, {$nama}!";
};

echo $sapa("Budi"); // "Halo, Budi!"
```

### 7.2 Closure dengan `use`

Closure punya scope sendiri — tidak bisa mengakses variabel dari scope luar. Gunakan `use` untuk mengimpor.

```php
<?php
$diskon = 0.1; // scope luar

$hitungDiskon = function (int $harga) use ($diskon): int {
    // $diskon diimpor dari scope luar via use
    return $harga - ($harga * $diskon);
};
```

**Di codebase — CartService.php:89-91:**
```php
$cart = $this->getCart();
return array_reduce($cart, function ($total, $item) {
    return $total + ($item['price'] * $item['quantity']);
}, 0);
```

Tidak perlu `use` di sini karena `$total` dan `$item` adalah parameter dari `array_reduce`.

**Di codebase — RajaOngkirService.php:147-149:**
```php
return array_values(array_filter($allCities, function ($city) use ($provinceId) {
    return $city['province_id'] == $provinceId;
}));
```

**Kenapa perlu `use ($provinceId)`?** Karena `$provinceId` adalah parameter dari method `getCities()`, bukan parameter closure. Closure tidak bisa "melihat" ke luar tanpa `use`.

### 7.3 Closure untuk DB Transaction

**Di codebase — OrderService.php:31-77:**
```php
return DB::transaction(function () use ($cart, $total, $data) {
    // Validasi stok
    foreach ($cart as $item) {
        $product = Product::lockForUpdate()->findOrFail($item['id']);
        if ($product->stock_quantity < $item['quantity']) {
            throw new \Exception("...");
        }
    }

    // Create order
    $order = Order::create([...]);

    // Create order items
    foreach ($cart as $item) {
        OrderItem::create([...]);
    }

    $this->cartService->clear();

    return $order;
}); // ← Jika exception terjadi di dalam, rollback otomatis!
```

`DB::transaction(function () { ... })` menjalankan semua kode di dalam closure dalam satu transaksi database. Jika ada exception, semua perubahan di-rollback.

**Pola yang sama di PaymentService.php:29:**
```php
DB::transaction(function () use ($payment) {
    $payment->update([...]);
    $order->update([...]);
    // kurangi stok...
    $order->user->notify(...);
});
```

Ini adalah **pola transaksional** — beberapa operasi database yang harus semua berhasil atau semua gagal.

---

## 8. Arrow Function (`fn`) — PHP 7.4+

Arrow function adalah **closure yang lebih pendek** — hanya bisa berisi **satu ekspresi** (return implisit).

### 8.1 Syntax

```php
<?php
// Closure biasa
$kuadrat = function (int $n): int {
    return $n * $n;
};

// Arrow function
$kuadrat = fn(int $n): int => $n * $n;
```

### 8.2 Perbedaan Kunci

| Fitur | Closure | Arrow function |
|-------|---------|----------------|
| Keyword | `function` | `fn` |
| `use` | Wajib untuk akses scope luar | **Otomatis** — akses langsung |
| Body | Banyak statement | **Satu ekspresi** saja |
| Return | Eksplisit `return` | **Implisit** |
| Multi-baris | Bisa | Tidak bisa |

```php
<?php
$diskon = 0.1;

// Closure — perlu use
$harga = array_map(function ($h) use ($diskon) {
    return $h - ($h * $diskon);
}, $hargaList);

// Arrow function — akses langsung $diskon
$harga = array_map(fn($h) => $h - ($h * $diskon), $hargaList);
```

### 8.3 Di Codebase

**DashboardController.php:24:**
```php
$totalCustomers = User::whereHas('roles', fn($q) => $q->where('slug', 'customer'))->count();
```

Tanpa arrow function:
```php
$totalCustomers = User::whereHas('roles', function ($q) {
    $q->where('slug', 'customer');
})->count();
```

**CustomerController.php:14:**
```php
$query = User::whereHas('roles', fn($q) => $q->where('slug', 'customer'))
    ->withCount('orders')
    ->latest();
```

**Kapan pakai arrow function vs closure:**
- **Arrow function:** Untuk callback sederhana 1 baris (filter, map, whereHas sederhana)
- **Closure:** Untuk logika multi-baris, perlu `use` dengan variabel reference, atau perlu beberapa statement

---

## 9. Advanced Function Patterns

### 9.1 Method Chaining

Method chaining terjadi ketika setiap method mengembalikan objek sehingga method berikutnya bisa dipanggil langsung.

**Di codebase — ProductService.php:12:**
```php
return Product::with(['category', 'brand'])->latest()->paginate($perPage);
```

Visual alur:
```
Product::with(['category', 'brand'])  → return QueryBuilder
         → .latest()                  → return QueryBuilder (sama)
         → .paginate($perPage)        → return LengthAwarePaginator
```

**Di codebase — RajaOngkirService.php:32-34:**
```php
$response = Http::withHeaders(['key' => $this->apiKey])
    ->timeout(10)
    ->get("{$this->baseUrl}/province");
```

Agar chaining bisa bekerja, method harus mengembalikan `$this` atau objek lain:

```php
<?php
class QueryBuilder
{
    protected array $conditions = [];

    public function where(string $field, mixed $value): static
    {
        $this->conditions[] = [$field, '=', $value];
        return $this; // ← kunci chaining!
    }

    public function orderBy(string $field, string $dir = 'asc'): static
    {
        // ...
        return $this;
    }

    public function get(): array
    {
        // eksekusi query
        return $results;
    }
}

// Penggunaan:
$results = (new QueryBuilder())
    ->where('status', 'active')
    ->orderBy('name')
    ->get();
```

### 9.2 Callable Type

Parameter bertipe `callable` menerima fungsi, method, closure, atau arrow function.

```php
<?php
function prosesArray(array $items, callable $callback): array
{
    $result = [];
    foreach ($items as $item) {
        $result[] = $callback($item);
    }
    return $result;
}

// Berbagai cara passing callable:
$hasil = prosesArray([1, 2, 3], fn($n) => $n * 2); // arrow function
$hasil = prosesArray([1, 2, 3], function ($n) { return $n * 2; }); // closure
$hasil = prosesArray([1, 2, 3], 'strtoupper'); // string nama fungsi

// Method sebagai callable: [objek, method]
$hasil = prosesArray([1, 2, 3], [$this, 'methodKu']);

// Static method: [ClassName::class, 'methodStatic']
$hasil = prosesArray([1, 2, 3], [StringHelper::class, 'format']);
```

### 9.3 First-class Callable (PHP 8.1+)

PHP 8.1 memperkenalkan sintaks untuk membuat callable dari method **tanpa closure wrapper**:

```php
<?php
// Sebelum PHP 8.1 — perlu closure wrapper
$callback = function (string $s): string {
    return Str::slug($s);
};
$slugs = array_map($callback, $names);

// PHP 8.1+ — first-class callable
$slugs = array_map(Str::slug(...), $names);
//           ↑ "mengubah method Str::slug jadi callable"
```

Syntax `Str::slug(...)` membuat `\Closure` dari method `Str::slug` yang bisa dipanggil dengan parameter sisanya.

### 9.4 Named Argument dengan Variadic

Ini penting: `...$args` (variadic) tidak bisa dipakai dengan named argument. Tapi named argument sangat berguna untuk parameter opsional di method panjang.

**Di route definition** (file `routes/web.php`):

```php
Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
```

Saat dipanggil:
```php
route('dashboard', absolute: false) // named argument
```

Lebih baik daripada:
```php
route('dashboard', [], false, null) // positional — harus ingat urutan parameter
```

---

## 10. Studi Kasus: Analisis Fungsi di Codebase

### 10.1 RajaOngkirService — Struktur Method

`app/Services/RajaOngkirService.php` adalah contoh sempurna struktur service class:

```
┌─────────────────────────────────────────────────────────┐
│ RajaOngkirService                                       │
├─────────────────────────────────────────────────────────┤
│ Konstruktor                                             │
│   __construct() — baca konfigurasi API key              │
│                                                         │
│ Public API (3 method)                                   │
│   getProvinces(): array                                 │
│   getCities($provinceId = null): array                  │
│   calculateCost($origin, $destination, int $weight,     │
│                 string $courier): array                 │
│                                                         │
│ Private Fallback (3 method)                             │
│   getDummyProvinces(): array                            │
│   getDummyCities($provinceId): array                    │
│   getDummyCost($courier): array                         │
└─────────────────────────────────────────────────────────┘
```

Setiap public method mengikuti pola yang sama:
1. Coba panggil API
2. Jika gagal, log error
3. Kembali ke dummy data

```php
public function getProvinces(): array
{
    if (empty($this->apiKey)) {
        return $this->getDummyProvinces(); // ← fallback
    }

    try {
        $response = Http::withHeaders(['key' => $this->apiKey])
            ->timeout(10)
            ->get("{$this->baseUrl}/province");

        if ($response->failed()) {
            Log::error('...');
            return $this->getDummyProvinces(); // ← fallback
        }

        return $response->json('rajaongkir.results')
            ?? $this->getDummyProvinces(); // ← fallback
    } catch (\Exception $e) {
        Log::error('...');
        return $this->getDummyProvinces(); // ← fallback
    }
}
```

**Design pattern:** **Fallback** — 3 lapis pertahanan (API key kosong → HTTP gagal → exception). Berguna untuk development tanpa API key nyata.

### 10.2 Setting Model — Static Method Pattern

`app/Models/Setting.php` adalah contoh clean static method:

```php
class Setting extends Model
{
    public static function get(string $key, mixed $default = null): mixed
    {
        $setting = static::where('key', $key)->first();
        if (!$setting) return $default;

        return match ($setting->type) {
            'boolean'   => (bool) $setting->value,
            'integer'   => (int) $setting->value,
            'json'      => json_decode($setting->value, true),
            default     => $setting->value,
        };
    }

    public static function set(string $key, mixed $value, string $type = 'string'): static
    {
        return static::updateOrCreate(
            ['key' => $key],
            ['value' => is_array($value) ? json_encode($value) : $value, 'type' => $type]
        );
    }
}
```

**Poin penting:**
- `static::` bukan `self::` — late static binding
- Return type `: static` — PHP 8.0+, mengembalikan instance class tempat method dipanggil
- `mixed` — parameter bisa apa pun
- `match` expression + type casting — transformasi otomatis
- `updateOrCreate` — method Eloquent, cari record atau buat baru

### 10.3 CheckoutController — Constructor DI Pattern

```php
class CheckoutController extends Controller
{
    protected $cartService;
    protected $addressService;
    protected $orderService;
    protected $rajaOngkirService;

    public function __construct(
        CartService $cartService,
        AddressService $addressService,
        OrderService $orderService,
        \App\Services\RajaOngkirService $rajaOngkirService
    ) {
        $this->cartService = $cartService;
        $this->addressService = $addressService;
        $this->orderService = $orderService;
        $this->rajaOngkirService = $rajaOngkirService;
    }
```

**Mengapa pola ini ada di hampir semua controller?**
1. **Separation of concerns** — controller hanya routing; logika bisnis di service
2. **Testability** — service bisa dimock/diganti saat testing
3. **Reusability** — service bisa dipakai banyak controller
4. **Laravel convention** — semua dependency di-inject via constructor

### 10.4 PaymentService — Transaction Closure Pattern

```php
public function confirmPayment(Payment $payment)
{
    DB::transaction(function () use ($payment) {
        $payment->update(['status' => 'confirmed', 'confirmed_at' => now()]);
        $payment->order->update(['status' => 'processing', 'paid_at' => now()]);

        foreach ($payment->order->items as $item) {
            $product = Product::lockForUpdate()->find($item->product_id);
            $product->decrement('stock_quantity', $item->quantity);
        }

        $payment->order->user->notify(new OrderStatusChanged($payment->order));
    });

    return $payment->fresh();
}
```

**Alur:**
1. `DB::transaction(function () { ... })` — buka transaksi
2. Semua query di dalam closure: update payment, update order, kurangi stok, kirim notifikasi
3. Jika semua sukses → commit
4. Jika exception terjadi → rollback semua perubahan
5. `$payment->fresh()` — reload dari database untuk dapat data terbaru

---

## 11. Perangkap Umum

### 11.1 `use` dan Reference

```php
<?php
$items = [];

$tambah = function () use ($items) {
    $items[] = 'baru'; // ← $items DI COPY, original tidak berubah!
};

$tambah();
print_r($items); // [] — kosong!

// Solusi: &$items (pass by reference)
$tambah2 = function () use (&$items) {
    $items[] = 'baru';
};
$tambah2();
print_r($items); // ['baru']
```

**Aturan:** `use` membuat **copy** dari variabel scope luar, kecuali pakai `&` (reference).

### 11.2 Type Error di PHP

```php
<?php
function hitung(int $a, int $b): int
{
    return $a + $b;
}

echo hitung(1, 2);    // 3 — OK
echo hitung("1", 2);  // 3 — OK (string bisa di-cast ke int secara implisit)
echo hitung("abc", 2); // TypeError — "abc" tidak bisa di-cast ke int
```

Agar PHP benar-benar ketat, nyalakan **strict types**:

```php
<?php
declare(strict_types=1);

function hitung(int $a, int $b): int
{
    return $a + $b;
}

echo hitung("1", 2); // TypeError! String tidak diterima di strict mode
```

**Best practice:** Selalu deklarasi `declare(strict_types=1)` di file baru. Tapi di codebase ini, belum dipakai secara konsisten — jadi PHP masih dalam mode coercive (konversi otomatis).

### 11.3 Method Tidak Ada (Call to undefined method)

```php
<?php
$service = new RajaOngkirService();
$service->getDummyProvinces(); // ❌ Fatal Error!
// getDummyProvinces() adalah private — tidak bisa dari luar
```

### 11.4 Recursion Tanpa Base Case

```php
<?php
function hitungMundur(int $n): void
{
    echo $n;
    hitungMundur($n - 1); // ← tidak ada base case!
    // Stack overflow setelah ~100-1000 iterasi
}

// Harusnya:
function hitungMundur(int $n): void
{
    if ($n <= 0) return; // ← base case
    echo $n;
    hitungMundur($n - 1);
}
```

### 11.5 Memanggil Method Non-static Secara Static

```php
<?php
class CartService
{
    public function getCart(): array { ... }
}

CartService::getCart(); // ❌ Deprecated di PHP 8.0, Error di PHP 8.2+
// Method non-static tidak bisa dipanggil secara static

// Benar:
$cartService = new CartService();
$cartService->getCart();
```

---

## 12. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│                      FUNGSI                               │
├──────────────────────────────────────────────────────────┤
│  Deklarasi:                                               │
│    function nama( Type $param, ... ): ReturnType { ... }  │
│                                                           │
│  Parameter:                                               │
│    Required: function f($a)                               │
│    Default:  function f($a = null)                        │
│    Variadic: function f(...$args)                         │
│    Named:    f(argumentName: value)                       │
│    Type:     function f(int $x, string $y)                │
│                                                           │
│  Return:                                                  │
│    : void    → tidak return                               │
│    : ?Type   → bisa null                                  │
│    : Type|Type  → union (PHP 8)                           │
│    : mixed   → apa pun                                    │
│    : static  → return instance class (PHP 8)              │
├──────────────────────────────────────────────────────────┤
│                      SCOPE                                │
├──────────────────────────────────────────────────────────┤
│  Variabel di dalam fungsi → local (hidden dari luar)      │
│  Variabel global → tidak bisa diakses fungsi              │
│  static $var → dipertahankan antar pemanggilan            │
│  use ($var) → import scope luar ke closure                │
│                                                           │
│  $this     → instance method, akses properti/method       │
│  self::    → static, resolves ke class saat kompilasi     │
│  static::  → static, resolves ke class saat runtime (LSB) │
├──────────────────────────────────────────────────────────┤
│                      CLOSURE                               │
├──────────────────────────────────────────────────────────┤
│  Closure:  function ($p) use ($var) { ... }               │
│  Arrow:    fn($p) => expr                                 │
│                                                           │
│  Arrow function:                                          │
│    - Otomatis $var dari scope luar (no use)               │
│    - Hanya 1 ekspresi                                     │
│    - Tidak bisa modify scope luar                         │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

Kerjakan di `php -a` atau file `.php`:

1. **Deklarasi fungsi.** Buat fungsi `formatRupiah(int $jumlah): string` yang mengembalikan format `"Rp 1.000.000"`. Gunakan `number_format`.

2. **Default parameter.** Buat fungsi `buatProduk(string $nama, float $harga, int $stok = 0): array` yang mengembalikan array asosiatif.

3. **Variadic.** Buat fungsi `jumlahkan(int ...$angka): int` yang menjumlahkan semua argumen.

4. **Closure + array_filter.** Dari array `$produk = [['nama' => 'A', 'harga' => 1000], ['nama' => 'B', 'harga' => 500]]`, filter yang harga > 800.

5. **Arrow function.** Ubah closure di #4 jadi arrow function.

6. **Baca `app/Models/Setting.php`.** Jelaskan ke diri sendiri alur `Setting::get()` dan `Setting::set()` — apa yang terjadi step by step.

7. **Buat fungsi rekursif.** `faktorial(int $n): int` tanpa loop (pakai rekursi).

---

## 🔗 Referensi

- [PHP Manual: Functions](https://www.php.net/manual/en/language.functions.php)
- [PHP Manual: Variable scope](https://www.php.net/manual/en/language.variables.scope.php)
- [PHP Manual: Anonymous functions](https://www.php.net/manual/en/functions.anonymous.php)
- [PHP 8.0: Named Arguments](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments)
- [PHP 8.1: First-class Callable](https://www.php.net/manual/en/functions.first_class_callable_syntax.php)
- Codebase: `app/Services/RajaOngkirService.php`
- Codebase: `app/Services/CartService.php`
- Codebase: `app/Models/Setting.php`
- Codebase: `app/Http/Controllers/CheckoutController.php`
- Codebase: `app/Http/Middleware/RoleMiddleware.php`
- Codebase: `app/Services/PaymentService.php`
