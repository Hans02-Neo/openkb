# 04-02: First-Class Citizen — Fungsi sebagai Warga Negara Kelas Satu

> **Fase**: 4 — Functional Programming  
> **Prasyarat**: 04-01-paradigma-functional  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: first-class function, higher-order function, closure, callback, anonymous function, callable, variable function, closure binding, use

---

## 📋 Ringkasan

Di dunia OOP, **objek** adalah warga negara kelas satu — bisa disimpan di variabel, dilewatkan ke fungsi, dikembalikan dari fungsi. Di dunia functional, **fungsi** juga melakukan hal yang sama.

PHP, sejak versi 5.3 (2009), memperlakukan fungsi sebagai **first-class citizen** — fungsi bisa:
1. Disimpan di **variabel**
2. Dilewatkan sebagai **argument**
3. Dikembalikan sebagai **return value**
4. Disimpan di **array**

**Target pemahaman:**
- Kamu bisa membuat dan menggunakan closure
- Kamu paham `use` untuk mengimpor variabel ke closure
- Kamu paham arrow function vs closure
- Kamu bisa melihat pola callback di codebase

---

## 1. Fungsi di Variabel

```php
<?php

// Fungsi "biasa" — dideklarasikan, dipanggil via nama
function tambah(int $a, int $b): int { return $a + $b; }
echo tambah(2, 3); // 5

// Closure — fungsi anonim disimpan di variabel
$kali = function (int $a, int $b): int {
    return $a * $b;
};
echo $kali(2, 3); // 6 — panggil variabel + ()

// Arrow function — closure satu ekspresi
$kurang = fn(int $a, int $b): int => $a - $b;
echo $kurang(5, 3); // 2
```

**Kenapa ini penting?** Karena closure bisa dilewatkan ke fungsi lain sebagai **callback**.

---

## 2. Callback — Fungsi sebagai Argument

### 2.1 Contoh Paling Sederhana

```php
<?php

function prosesArray(array $items, callable $callback): array
{
    $result = [];
    foreach ($items as $item) {
        $result[] = $callback($item); // panggil callback
    }
    return $result;
}

$numbers = [1, 2, 3, 4, 5];

// Callback via closure
$doubled = prosesArray($numbers, function ($n) {
    return $n * 2;
});

// Callback via arrow function
$tripled = prosesArray($numbers, fn($n) => $n * 3);

// Callback via nama fungsi
$asString = prosesArray($numbers, 'strval'); // ['1','2','3','4','5']
```

### 2.2 Callback di Codebase

**array_reduce — callback di CartService.php:89:**
```php
array_reduce($cart, function ($total, $item) {
    return $total + ($item['price'] * $item['quantity']);
}, 0);
```

Callback `function ($total, $item) { ... }` dilewatkan ke `array_reduce` yang memanggilnya untuk setiap elemen.

**DB::transaction — callback di OrderService.php:31:**
```php
return DB::transaction(function () use ($cart, $total, $data) {
    // Semua logika di sini — callback dijalankan di dalam transaksi
});
```

`DB::transaction` menerima **callable** — menjalankannya, dan rollback jika exception terjadi.

### 2.3 Variable Functions

```php
<?php
function sapa(string $nama): string { return "Halo {$nama}"; }
function pamit(string $nama): string { return "Dadah {$nama}"; }

$fungsi = 'sapa';
echo $fungsi('Budi'); // "Halo Budi" — PHP panggil fungsi sapa()

$fungsi = 'pamit';
echo $fungsi('Budi'); // "Dadah Budi"
```

**Tidak dipakai di codebase ini**, tapi berguna untuk dispatch dinamis.

### 2.4 Callable Type

PHP punya tipe `callable` untuk parameter yang menerima fungsi/closure:

```php
<?php
function measure(callable $fn): float
{
    $start = microtime(true);
    $fn();
    return microtime(true) - $start;
}

$time = measure(function () {
    sleep(1);
});
```

---

## 3. Closure dan Scope

### 3.1 Closure Punya Scope Sendiri

```php
<?php
$diskon = 0.1;

$hitungDiskon = function (float $harga): float {
    // ❌ $diskon TIDAK BISA diakses!
    // Closure punya scope terpisah
    return $harga - ($harga * $diskon); // Error: undefined variable
};
```

### 3.2 `use` — Mengimpor Variabel ke Closure

```php
<?php
$diskon = 0.1;

$hitungDiskon = function (float $harga) use ($diskon): float {
    // ✅ $diskon diimpor via use
    return $harga - ($harga * $diskon);
};

echo $hitungDiskon(10000); // 9000
```

**Di codebase — RajaOngkirService.php:147:**
```php
array_filter($allCities, function ($city) use ($provinceId) {
    return $city['province_id'] == $provinceId;
});
```

**Di codebase — OrderService.php:31:**
```php
DB::transaction(function () use ($cart, $total, $data) {
    // $cart, $total, $data dariimpor dari scope luar
});
```

### 3.3 Pass by Reference dengan `&`

Secara default, `use` membuat **copy** dari variabel. Untuk mengubah variabel asli, pakai `&`:

```php
<?php
$counter = 0;

$increment = function () use (&$counter): void {
    $counter++; // mengubah $counter asli
};

$increment();
$increment();
echo $counter; // 2
```

**Hati-hati dengan reference!** Ini melanggar immutability — gunakan hanya jika benar-benar perlu.

---

## 4. Arrow Function (`fn`) — PHP 7.4+

### 4.1 Syntax

```php
<?php
// Closure biasa
$doubled = array_map(function ($n) use ($multiplier) {
    return $n * $multiplier;
}, $numbers);

// Arrow function — lebih pendek, use OTOMATIS
$doubled = array_map(fn($n) => $n * $multiplier, $numbers);
```

### 4.2 Perbedaan Closure vs Arrow

| | Closure | Arrow Function |
|---|---------|---------------|
| Keyword | `function` | `fn` |
| `use` | **Wajib** untuk akses variabel luar | **Otomatis** (semua variabel luar bisa diakses) |
| Body | Banyak statement | **Satu ekspresi** (implisit return) |
| Return | Harus `return` | Otomatis return ekspresi |
| Mutate luar | Bisa dengan `&` | **Tidak bisa** mengubah variabel luar |

### 4.3 Arrow Function di Codebase

**CustomerController.php:14:**
```php
User::whereHas('roles', fn($q) => $q->where('slug', 'customer'))
```

Ini setara dengan:
```php
User::whereHas('roles', function ($q) {
    $q->where('slug', 'customer');
});
```

**DashboardController.php:24:**
```php
User::whereHas('roles', fn($q) => $q->where('slug', 'customer'))->count();
```

---

## 5. Higher-Order Functions — Fungsi yang Mengembalikan Fungsi

### 5.1 Factory Function

```php
<?php
function createDiscounter(float $discountRate): callable
{
    // Mengembalikan CLOSURE yang sudah "mengingat" $discountRate
    return fn(float $price): float => $price - ($price * $discountRate);
}

$discount10 = createDiscounter(0.10); // fungsi diskon 10%
$discount20 = createDiscounter(0.20); // fungsi diskon 20%

echo $discount10(100000); // 90000
echo $discount20(100000); // 80000
```

**Ini disebut closure binding** — closure "mengingat" variabel dari scope saat ia diciptakan.

### 5.2 Partial Application

```php
<?php
function add(int $a, int $b): int { return $a + $b; }

function partialAdd(int $a): callable
{
    return fn(int $b) => add($a, $b);
}

$add5 = partialAdd(5);
echo $add5(3);  // 8
echo $add5(10); // 15
```

### 5.3 Function Composition

```php
<?php
function compose(callable $f, callable $g): callable
{
    return fn(mixed $x): mixed => $f($g($x));
}

$addTax = fn(float $p): float => $p * 1.11;
$formatRupiah = fn(float $p): string => 'Rp ' . number_format($p, 0, ',', '.');

$hargaJual = compose($formatRupiah, $addTax);
echo $hargaJual(10000); // "Rp 11.100"
// Pertama: addTax(10000) → 11100
// Kedua: formatRupiah(11100) → "Rp 11.100"
```

---

## 6. First-Class Callable Syntax (PHP 8.1)

### 6.1 Masalah

```php
<?php
// Sebelum PHP 8.1 — butuh closure wrapper untuk convert method ke callable
$slugs = array_map(function (string $name): string {
    return Str::slug($name);
}, $names);
```

### 6.2 Solusi PHP 8.1

```php
<?php
// PHP 8.1+ — first-class callable: Str::slug(...)
$slugs = array_map(Str::slug(...), $names);
```

`Str::slug(...)` membuat callable dari method `slug` tanpa perlu closure wrapper. Tiga titik `...` bukan variadic — ini syntax khusus untuk first-class callable.

```php
<?php
// Contoh lain:
$ids = array_map(fn($p) => $p->id, $products);
// Bisa jadi:
$ids = array_map(fn(Product $p) => $p->id, ...); // property access tidak bisa pakai ...

$upper = array_map(strtoupper(...), $names);      // native function
$lower = array_map(Str::lower(...), $names);       // static method
$notNull = array_filter($items, fn($i) => $i !== null);
```

**Di codebase:** Belum menggunakan first-class callable syntax.

---

## 7. Closure Sebagai Callback di Eloquent

Eloquent menggunakan closure di banyak tempat untuk **query scoping**:

### 7.1 whereHas

```php
// ShopController.php:18
$query->whereHas('category', function ($q) use ($request) {
    $q->where('slug', $request->category);
});
```

### 7.2 where dengan grup

```php
// ShopController.php:24-27
$query->where(function ($q) use ($request) {
    $q->where('name', 'like', '%' . $request->search . '%')
      ->orWhere('short_description', 'like', '%' . $request->search . '%');
});
```

### 7.3 with (eager loading) — closure untuk constraint

```php
// Belum di codebase, tapi pattern umum:
Product::with(['category' => function ($q) {
    $q->where('is_active', true);
}]);
```

---

## 8. Closure di Array dan Callable di Parameter

### 8.1 Array of Callables

```php
<?php
$validations = [
    'required' => fn($v) => !empty($v),
    'email'    => fn($v) => filter_var($v, FILTER_VALIDATE_EMAIL),
    'min'      => fn($v) => strlen($v) >= 3,
];

function validate(mixed $value, array $rules): array
{
    $errors = [];
    foreach ($rules as $name => $rule) {
        if (!$rule($value)) {
            $errors[] = "Validation {$name} failed";
        }
    }
    return $errors;
}
```

### 8.2 Callable di Parameter (Laravel pattern)

```php
// Macro — menambah method ke class inti Laravel
Str::macro('customSlug', function (string $title) {
    return str_replace(' ', '-', strtolower($title));
});

// Menggunakan callable di config
config(['services.rajaongkir.key' => env('RAJAONGKIR_API_KEY')]);

// Menggunakan callback di route
Route::get('/dashboard', function () {
    return redirect()->route('orders.index');
});
```

---

## 9. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│ FIRST-CLASS FUNCTION                                     │
│  Fungsi bisa:                                            │
│  1. Disimpan di variabel: $fn = fn($x) => $x * 2;      │
│  2. Dilewatkan ke fungsi: array_map($fn, $arr);         │
│  3. Dikembalikan dari fungsi: return fn($x) => $x;      │
│  4. Disimpan di array: $callbacks['kali'] = $fn;        │
├──────────────────────────────────────────────────────────┤
│ CLOSURE            │ ARROW FUNCTION                      │
│ function ($p)      │ fn($p) => expr                     │
│ use ($var)         │ Otomatis akses $var                 │
│ Multi baris        │ Satu ekspresi                       │
│ Bisa &$var (ref)   │ Tidak bisa ubah luar                │
├──────────────────────────────────────────────────────────┤
│ DI CODEBASE:                                             │
│                                                          │
│  array_reduce($cart, function ($t, $i) { ... }, 0)       │
│  DB::transaction(function () use (...) { ... })          │
│  array_filter($arr, fn($x) => $x['id'] == $id)          │
│  $q->whereHas('rel', fn($q) => $q->where(...))          │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Buat closure.** Simpan fungsi yang menerima string dan mengembalikan panjangnya ke variabel `$panjangString`.

2. **Callback.** Buat fungsi `applyToEach(array $items, callable $fn): array` yang menerapkan `$fn` ke setiap item.

3. **Use.** Buat closure yang menggunakan variabel `$pajak = 0.11` dari scope luar untuk menghitung harga setelah pajak.

4. **Higher-order.** Buat fungsi `createGreeter(string $greeting): callable` yang mengembalikan fungsi yang menyapa dengan `$greeting` tertentu.

5. **Identifikasi.** Cari semua penggunaan closure dan arrow function di `app/Http/Controllers/ShopController.php`. Sebutkan baris dan fungsinya.

---

## 🔗 Referensi

- [PHP Manual: Anonymous Functions](https://www.php.net/manual/en/functions.anonymous.php)
- [PHP Manual: Arrow Functions](https://www.php.net/manual/en/functions.arrow.php)
- [PHP 8.1: First-class Callable](https://www.php.net/manual/en/functions.first_class_callable_syntax.php)
- Codebase: `app/Services/CartService.php` — closures
- Codebase: `app/Services/OrderService.php` — DB::transaction closure
- Codebase: `app/Http/Controllers/ShopController.php` — whereHas closure
