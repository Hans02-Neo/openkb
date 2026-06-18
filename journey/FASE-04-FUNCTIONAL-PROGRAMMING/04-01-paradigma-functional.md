# 04-01: Paradigma Functional Programming

> **Fase**: 4 — Functional Programming  
> **Prasyarat**: FASE 3 (OOP)  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: functional programming, pure function, immutability, side effect, declarative, imperative, referential transparency, higher-order function

---

## 📋 Ringkasan

Selama Fase 3, kamu belajar OOP — memandang program sebagai kumpulan **objek** yang saling berkomunikasi. Sekarang kita bahas paradigma lain: **Functional Programming (FP)** — memandang program sebagai **evaluasi fungsi** tanpa mengubah state.

**Target pemahaman:**
- Kamu paham perbedaan imperative vs declarative vs functional
- Kamu bisa menjelaskan pure function dan kenapa penting
- Kamu paham immutability dan side effects
- Kamu bisa melihat pola FP di codebase ini

---

## 1. Tiga Paradigma Besar

### 1.1 Imperative Programming

**"Bagaimana"** cara melakukan sesuatu — langkah demi langkah.

```php
<?php
// IMPERATIF: Beri tahu komputer LANGKAH DEMI LANGKAH
$numbers = [1, 2, 3, 4, 5];
$doubled = [];

for ($i = 0; $i < count($numbers); $i++) {
    $doubled[] = $numbers[$i] * 2;  // mutasi $doubled
}

// Kamu bilang: "Buat array kosong, lalu loop index 0 sampai 4,
// ambil setiap angka, kali 2, masukkan ke array baru."
```

### 1.2 Declarative Programming

**"Apa"** yang ingin dicapai — tanpa menjelaskan langkah detail.

```php
<?php
// DECLARATIVE: Bilang APA yang kamu mau
$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(fn($n) => $n * 2, $numbers);

// Kamu bilang: "Setiap angka di-doubel." — tidak perlu bilang cara loop-nya.
```

SQL adalah contoh bahasa deklaratif murni:
```sql
SELECT name FROM products WHERE price > 10000;
-- Kamu bilang APA yang mau — tidak perlu bilang cara query-nya.
```

### 1.3 Functional Programming

**Subset dari declarative** — program dibangun dari **fungsi murni** yang:
1. Selalu return nilai yang sama untuk input yang sama
2. Tidak mengubah state di luar fungsi

```php
<?php
// FUNCTIONAL: Fungsi murni, tanpa side effect
function double(array $numbers): array
{
    return array_map(fn($n) => $n * 2, $numbers);
    // Tidak mengubah $numbers asli — return array baru
}

$original = [1, 2, 3];
$result = double($original);
// $original masih [1, 2, 3] — tidak berubah!
```

---

## 2. Pure Function — Fungsi Murni

### 2.1 Definisi

**Pure function** adalah fungsi yang:
1. **Return yang sama untuk input yang sama** (tidak ada pengaruh dari luar)
2. **Tidak punya side effects** (tidak mengubah apa pun di luar fungsi)

```php
<?php
// ✅ PURE — hanya pakai parameter, return hasil kalkulasi
function add(int $a, int $b): int
{
    return $a + $b;
}

// add(2, 3) → 5. SELALU 5. Tidak peduli apa pun di luar.
```

```php
<?php
// ❌ IMPURE — tergantung variabel global
$discount = 0.1;
function calculatePrice(float $price): float
{
    global $discount;  // ← dari luar! Bisa berubah!
    return $price - ($price * $discount);
}

// calculatePrice(10000) → 9000 (kalau $discount = 0.1)
// calculatePrice(10000) → 9500 (kalau $discount = 0.05)
// Input SAMA, output BERBEDA — IMPURE!
```

### 2.2 Pure Functions di Codebase

**CartService.php:86-91:**

```php
public function getTotal(): int
{
    $cart = $this->getCart();
    return array_reduce($cart, function ($total, $item) {
        return $total + ($item['price'] * $item['quantity']);
    }, 0);
}
```

**Apakah ini pure?** Hampir — ketergantungannya hanya pada `$this->getCart()`. Jika session (penyimpanan cart) tidak berubah, output-nya selalu sama. Tidak ada side effect: tidak mengubah database, tidak mengubah session, tidak mengubah file.

### 2.3 Kenapa Pure Function Penting?

1. **Mudah dites** — tidak perlu setup database, session, atau mock
2. **Mudah dipahami** — semua input dari parameter, output dari return
3. **Parallel-safe** — tidak ada race condition
4. **Cacheable** — hasil bisa di-cache karena input → output deterministic
5. **Refactorable** — bisa dipindah, digabung, dipecah tanpa efek samping

---

## 3. Immutability — Jangan Ubah Data, Buat Baru

### 3.1 Mutasi (Yang Harus Dihindari)

```php
<?php
// ❌ MUTASI — mengubah data asli
$cart = [
    ['name' => 'Laptop', 'price' => 15000, 'qty' => 1],
    ['name' => 'Mouse',  'price' => 250,   'qty' => 2],
];

function addTax(array &$cart, float $taxRate): void
{
    foreach ($cart as &$item) {
        $item['price'] = $item['price'] * (1 + $taxRate); // ← MUTASI!
    }
}

addTax($cart, 0.11);
// $cart SEKARANG BERUBAH — data asli hilang!
```

### 3.2 Immutability (Yang Benar)

```php
<?php
// ✅ IMMUTABLE — buat array baru, jangan ubah asli
$cart = [
    ['name' => 'Laptop', 'price' => 15000, 'qty' => 1],
    ['name' => 'Mouse',  'price' => 250,   'qty' => 2],
];

function addTax(array $cart, float $taxRate): array
{
    return array_map(function ($item) use ($taxRate) {
        return array_merge($item, [
            'price' => $item['price'] * (1 + $taxRate)
        ]);
    }, $cart);
}

$newCart = addTax($cart, 0.11);
// $cart ASLI TETAP SAMA — $newCart yang baru
// $cart[0]['price'] masih 15000
// $newCart[0]['price'] jadi 16650
```

### 3.3 Immutability di Codebase

**CartService — method `getTotal()` tidak mengubah cart:**

```php
public function getTotal(): int
{
    $cart = $this->getCart();
    // array_reduce TIDAK mengubah $cart
    return array_reduce($cart, function ($total, $item) {
        return $total + ($item['price'] * $item['quantity']);
    }, 0);
}
```

**RajaOngkirService — filter mengembalikan array baru:**

```php
return array_values(array_filter($allCities, function ($city) use ($provinceId) {
    return $city['province_id'] == $provinceId;
}));
// $allCities TIDAK BERUBAH — array baru dikembalikan
```

**Eloquent Query Builder — chaining tidak mengubah query asli:**

```php
$query = Product::where('is_active', true);

$query->where('price', '>', 10000); // Ini memodifikasi $query asli (Eloquent tidak immutable)
// Tapi Collection method seperti ->map(), ->filter() — mengembalikan Collection baru
```

Catatan: Eloquent **Query Builder** bersifat mutable (method mengubah objek yang sama). Tapi **Collection** bersifat immutable (method mengembalikan Collection baru).

---

## 4. Side Effects — Efek Samping

### 4.1 Apa Itu Side Effect?

Side effect adalah perubahan state di luar fungsi:

```php
<?php
function saveOrder(array $data): Order
{
    Log::info('Creating order...');     // ← side effect: menulis file log
    Mail::send(...);                     // ← side effect: kirim email
    $order = Order::create($data);      // ← side effect: mengubah database
    Session::forget('cart');             // ← side effect: mengubah session
    return $order;
}
```

**FP ideal:** fungsi tanpa side effect. **Di dunia nyata:** aplikasi web penuh side effect — database, email, file, session. Tujuannya bukan **nol** side effect, tapi **mengisolasi** dan **mengontrol** side effect.

### 4.2 Memisahkan Pure dan Impure

```php
<?php
// ✅ PISAHKAN: pure logic + impure execution

// PURE — tidak ada side effect
function calculateTotal(array $cart): float
{
    return array_reduce($cart, fn($t, $i) => $t + $i['price'] * $i['qty'], 0);
}

function calculateDiscount(float $total, float $rate): float
{
    return $total * $rate;
}

// IMPURE — side effect terpusat di sini
function placeOrder(array $cart, array $customer): Order
{
    $total = calculateTotal($cart);       // pure
    $discount = calculateDiscount($total, 0.1); // pure
    $grandTotal = $total - $discount;     // pure

    return Order::create([                // impure (DB write)
        'total' => $grandTotal,
        'customer' => $customer,
    ]);
}
```

---

## 5. Higher-Order Functions

### 5.1 Definisi

**Higher-order function** adalah fungsi yang:
1. Menerima fungsi lain sebagai **argument**, atau
2. Mengembalikan fungsi lain sebagai **return value**

```php
<?php
// array_map adalah higher-order function — menerima closure
array_map(fn($n) => $n * 2, [1, 2, 3]);

// DB::transaction adalah higher-order function
DB::transaction(function () {
    // closure dijalankan di dalam transaksi
});

// Fungsi yang mengembalikan fungsi
function createMultiplier(int $factor): callable
{
    return fn(int $n) => $n * $factor;
}

$double = createMultiplier(2);
echo $double(5); // 10
```

### 5.2 Di Codebase

**DB::transaction() — higher-order function di OrderService.php:31:**
```php
return DB::transaction(function () use ($cart, $total, $data) {
    // closure diterima sebagai argument
});
```

**array_reduce — higher-order function di CartService.php:89:**
```php
array_reduce($cart, function ($total, $item) {
    return $total + ($item['price'] * $item['quantity']);
}, 0);
```

**array_filter — higher-order function di RajaOngkirService.php:147:**
```php
array_filter($allCities, function ($city) use ($provinceId) {
    return $city['province_id'] == $provinceId;
});
```

---

## 6. FP vs OOP — Bukan Lawan, Tapi Pelengkap

### 6.1 Perbandingan

| Aspek | OOP | FP |
|-------|-----|-----|
| **Unit dasar** | Objek (data + method) | Fungsi |
| **State** | Data diubah via method | Data tidak diubah (immutable) |
| **Organisasi** | Class hierarchy | Function composition |
| **Control flow** | Polymorphism, inheritance | Map, filter, reduce, recursion |
| **Laravel** | Model, Controller, Service | Collection, closures, pipelines |

### 6.2 Laravel Menggabungkan Keduanya

```php
<?php
// OOP: Model adalah object dengan data + behavior
class Product extends Model { /* ... */ }

// FP: Collection method — operasi data tanpa mutasi
Product::where('is_active', true)
    ->get()
    ->filter(fn($p) => $p->price > 10000)  // FP: filter
    ->map(fn($p) => $p->name)               // FP: map
    ->take(5);                               // FP: take
```

**Praktik terbaik:** Gunakan OOP untuk **struktur** (class, object, inheritance) dan FP untuk **operasi data** (collection pipeline, pure functions).

---

## 7. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│ IMPERATIF                                                │
│  "Bagaimana" — Langkah demi langkah                      │
│  $result = []                                            │
│  for (...) { $result[] = ... }                           │
├──────────────────────────────────────────────────────────┤
│ DECLARATIVE                                              │
│  "Apa" — Hasil akhir, bukan cara                         │
│  SELECT name FROM users WHERE active = 1                 │
├──────────────────────────────────────────────────────────┤
│ FUNCTIONAL                                               │
│  Subset declarative — fungsi murni + immutability         │
│  array_map(fn($n) => $n * 2, $numbers)                   │
│                                                          │
│  ✅ Pure function: same input → same output              │
│  ✅ Immutable: tidak mengubah data asli                  │
│  ✅ No side effects: tidak sentuh dunia luar             │
│  ✅ Higher-order: fungsi terima/kembalikan fungsi lain    │
├──────────────────────────────────────────────────────────┤
│ DI CODEBASE:                                             │
│                                                          │
│  array_reduce  → CartService::getTotal()                 │
│  array_filter  → RajaOngkirService::filter by province   │
│  Arrow fn      → CustomerController, DashboardController │
│  DB::transaction → OrderService, PaymentService, etc.    │
│  Collection chain → ShopController                       │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Pure vs Impure.** Mana yang pure function?
```php
function a(int $x): int { return $x + 1; }
function b(float $x): float { return $x * rand(1, 10); }
function c(string $s): string { return strtoupper($s); }
function d(): string { return date('Y-m-d'); }
function e(array $arr): array { return array_reverse($arr); }
```

2. **Refactor ke immutable.** Ubah fungsi berikut jadi immutable (tidak mengubah array asli):
```php
function applyDiscount(array $products, float $discount): array
{
    foreach ($products as &$p) {
        $p['price'] = $p['price'] * (1 - $discount);
    }
    return $products;
}
```

3. **Identifikasi side effect.** Buka `app/Services/PaymentService.php` dan identifikasi semua side effect yang terjadi di `confirmPayment()`.

4. **Pure function.** Buka `app/Services/CartService.php`. Method mana yang pure? Method mana yang impure? Kenapa?

5. **Tulis higher-order function.** Buat fungsi `repeat(int $times, callable $fn)` yang memanggil `$fn` sebanyak `$times` kali.

---

## 🔗 Referensi

- [Wikipedia: Functional Programming](https://en.wikipedia.org/wiki/Functional_programming)
- [PHP Manual: Anonymous Functions](https://www.php.net/manual/en/functions.anonymous.php)
- [PHP Manual: Arrow Functions](https://www.php.net/manual/en/functions.arrow.php)
- Codebase: `app/Services/CartService.php` — array_reduce pure functions
- Codebase: `app/Services/RajaOngkirService.php` — array_filter
- Codebase: `app/Services/OrderService.php` — DB::transaction higher-order
