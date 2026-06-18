# 04-03: Map, Filter, Reduce — Tiga Raja Functional

> **Fase**: 4 — Functional Programming  
> **Prasyarat**: 04-02-first-class-citizen  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: array_map, array_filter, array_reduce, transform, filter, accumulate, pipeline, chaining, collection

---

## 📋 Ringkasan

Tiga fungsi ini adalah fondasi functional programming untuk **operasi array**. Hampir semua manipulasi data bisa diekspresikan dengan kombinasi map, filter, dan reduce.

**Target pemahaman:**
- Kamu paham kapan pakai map vs filter vs reduce
- Kamu bisa mengubah loop imperatif menjadi pipeline fungsional
- Kamu bisa membaca dan menulis pipeline di codebase

---

## 1. Map — Transformasi

### 1.1 Konsep

**Map** mengubah **setiap elemen** array menjadi bentuk lain. Jumlah elemen tetap.

```
Input:  [1,  2,  3,  4,  5]
Map:     fn($n) => $n * 2
Output: [2,  4,  6,  8,  10]
         ↑   ↑   ↑   ↑   ↑
         Setiap elemen di-transform — jumlah tetap 5
```

### 1.2 Imperatif vs Fungsional

```php
<?php
$prices = [10000, 25000, 5000, 75000];

// ❌ IMPERATIF — loop manual, mutasi array
$withTax = [];
for ($i = 0; $i < count($prices); $i++) {
    $withTax[] = $prices[$i] * 1.11;
}

// ✅ FUNCTIONAL — map, no mutation
$withTax = array_map(fn($price) => $price * 1.11, $prices);
```

### 3.1 Map dengan Key (array_keys)

`array_map` tidak mempertahankan key (kecuali objek). Untuk mempertahankan key:

```php
<?php
$products = [
    'laptop' => ['price' => 15000, 'stock' => 5],
    'mouse'  => ['price' => 250,   'stock' => 20],
];

// array_map — key LOST!
$prices = array_map(fn($p) => $p['price'], $products);
// [15000, 250] — key 'laptop' dan 'mouse' hilang!

// Pakai loop atau Collection kalau perlu key:
$prices = collect($products)->map(fn($p) => $p['price']);
// ['laptop' => 15000, 'mouse' => 250] — key dipertahankan
```

---

## 2. Filter — Penyaringan

### 2.1 Konsep

**Filter** menyaring elemen yang memenuhi kondisi. Jumlah elemen bisa **berkurang**.

```
Input:  [1,  2,  3,  4,  5,  6]
Filter:  fn($n) => $n % 2 == 0  (genap)
Output: [2,  4,  6]
         ↑   ↑   ↑
         Hanya yang memenuhi kondisi — jumlah bisa berubah
```

### 2.2 Imperatif vs Fungsional

```php
<?php
$products = [
    ['name' => 'Laptop', 'price' => 15000000, 'stock' => 5],
    ['name' => 'Mouse',  'price' => 250000,   'stock' => 0],
    ['name' => 'Keyboard', 'price' => 750000, 'stock' => 3],
];

// ❌ IMPERATIF
$available = [];
foreach ($products as $p) {
    if ($p['stock'] > 0) {
        $available[] = $p;
    }
}

// ✅ FUNCTIONAL
$available = array_filter($products, fn($p) => $p['stock'] > 0);
```

### 2.3 array_filter dengan Key

**Di codebase — RajaOngkirService.php:146-149:**

```php
return array_values(array_filter($allCities, function ($city) use ($provinceId) {
    return $city['province_id'] == $provinceId;
}));
```

**Kenapa `array_values`?** `array_filter` **mempertahankan key asli**. Jika elemen index 2 dan 5 cocok, hasilnya `[2 => [...], 5 => [...]]`. `array_values` mereset jadi `[0 => [...], 1 => [...]]`.

```
$allCities = [
    0 => ['city_id' => '39', 'province_id' => '5'],
    1 => ['city_id' => '418', 'province_id' => '5'],
    2 => ['city_id' => '152', 'province_id' => '6'],
    3 => ['city_id' => '153', 'province_id' => '6'],
];

array_filter → [0 => [...], 1 => [...]]  ← key 0,1 dipertahankan
array_values → [0 => [...], 1 => [...]]  ← key direset
```

### 2.4 Filter Null / Empty

```php
<?php
$items = ['Laptop', null, '', 'Mouse', false, 0];

// Hapus falsy values
$clean = array_filter($items); // ['Laptop', 'Mouse']
// (null, '', false, 0 dianggap falsy → dihapus)
```

---

## 3. Reduce — Akumulasi

### 3.1 Konsep

**Reduce** mereduksi array menjadi **satu nilai** — jumlah, rata-rata, produk, string gabungan, dll.

```
Input:  [10000, 25000, 5000, 15000]
         ↓       ↓      ↓      ↓
         +       +      +      +
         └───→ 35000 ←──┘      │
                 │             │
                 └─────→ 40000←┘
                         │
                         ▼
Output: 40000 (total)
```

Visual sebagai **akumulator** yang bergerak:
```
Start: total = 0 (initial value)
Step 1: total = 0     + 10000 = 10000
Step 2: total = 10000 + 25000 = 35000
Step 3: total = 35000 + 5000  = 40000
Step 4: total = 40000 + 15000 = 55000
Result: 55000
```

### 3.2 Imperatif vs Fungsional

```php
<?php
$items = [
    ['price' => 10000, 'qty' => 2],
    ['price' => 25000, 'qty' => 1],
    ['price' => 5000,  'qty' => 3],
];

// ❌ IMPERATIF
$total = 0;
foreach ($items as $item) {
    $total += $item['price'] * $item['qty'];
}

// ✅ FUNCTIONAL
$total = array_reduce($items, function ($carry, $item) {
    return $carry + ($item['price'] * $item['qty']);
}, 0);
```

### 3.3 array_reduce di Codebase

**CartService.php — tiga kali array_reduce:**

```php
// 1. TOTAL HARGA (line 89)
array_reduce($cart, function ($total, $item) {
    return $total + ($item['price'] * $item['quantity']);
}, 0);

// 2. JUMLAH ITEM (line 97)
array_reduce($cart, function ($count, $item) {
    return $count + $item['quantity'];
}, 0);

// 3. TOTAL BERAT (line 105)
array_reduce($cart, function ($weight, $item) {
    return $weight + ($item['weight_grams'] * $item['quantity']);
}, 0);
```

**Pola yang konsisten:**
- Parameter 1: array yang akan direduksi
- Parameter 2: callback `fn($carry, $item) => $carry + $item['field']`
- Parameter 3: nilai awal (`0`)

### 3.4 Reduce untuk Membangun Array

Reduce tidak hanya untuk angka — bisa untuk array juga:

```php
<?php
$items = [
    ['category' => 'laptop', 'name' => 'Gaming'],
    ['category' => 'laptop', 'name' => 'Office'],
    ['category' => 'mouse',  'name' => 'Wireless'],
];

// Group by category
$grouped = array_reduce($items, function ($groups, $item) {
    $groups[$item['category']][] = $item['name'];
    return $groups;
}, []);

// Hasil:
// ['laptop' => ['Gaming', 'Office'], 'mouse' => ['Wireless']]
```

---

## 4. Pipeline — Menggabungkan Map, Filter, Reduce

### 4.1 The Real Power

Kekuatan sesungguhnya adalah **menggabungkan** ketiganya dalam pipeline:

```php
<?php
$products = [
    ['name' => 'Laptop', 'price' => 15000000, 'stock' => 5,  'category' => 'elektronik'],
    ['name' => 'Mouse',  'price' => 250000,   'stock' => 0,  'category' => 'elektronik'],
    ['name' => 'Buku',   'price' => 75000,    'stock' => 10, 'category' => 'buku'],
    ['name' => 'Keyboard', 'price' => 750000, 'stock' => 3,  'category' => 'elektronik'],
];

// Pipeline: filter → filter → map → reduce
$totalElektronikTersedia = collect($products)
    ->filter(fn($p) => $p['category'] === 'elektronik')  // hanya elektronik
    ->filter(fn($p) => $p['stock'] > 0)                  // hanya yang tersedia
    ->map(fn($p) => $p['price'] * $p['stock'])            // hitung nilai stok
    ->sum();                                               // jumlahkan

echo $totalElektronikTersedia;
// (15000000*5) + (750000*3) = 75000000 + 2250000 = 77250000
```

**Bandingkan dengan imperatif:**

```php
<?php
$total = 0;
foreach ($products as $p) {
    if ($p['category'] === 'elektronik' && $p['stock'] > 0) {
        $total += $p['price'] * $p['stock'];
    }
}
```

Imperatif lebih pendek **di sini**, tapi pipeline lebih:
1. **Readable** — setiap langkah jelas apa fungsinya
2. **Composable** — bisa menambah/mengurangi langkah tanpa mengubah yang lain
3. **Testable** — setiap langkah bisa ditest terpisah

### 4.2 Pipeline di Codebase (Eloquent)

**ShopController.php — pipeline query builder:**

```php
$products = Product::with(['category', 'brand'])
    ->where('is_active', true)
    ->when($request->filled('category'), fn($q) => $q->whereHas('category', ...))
    ->when($request->filled('search'), fn($q) => $q->where(...))
    ->when($request->filled('in_stock'), fn($q) => $q->where('stock_quantity', '>', 0))
    ->orderBy(match($sort) { ... })
    ->paginate(12)
    ->appends($request->query());
```

Ini pipeline — setiap method mengembalikan object yang sama, method berikutnya dipanggil di atasnya.

---

## 5. Map, Filter, Reduce — Kapan Pakai yang Mana?

| Fungsi | Input → Output | Jumlah Elemen | Contoh |
|--------|---------------|---------------|--------|
| **Map** | `[A]` → `[B]` | **Sama** | Ubah semua harga jadi string Rupiah |
| **Filter** | `[A]` → `[A]` subset | **Berkurang** | Ambil produk dengan stok > 0 |
| **Reduce** | `[A]` → `B` | **Satu nilai** | Jumlah total harga semua item |

**Tanya 3 pertanyaan ini:**
1. Apakah saya mengubah bentuk setiap elemen? → **Map**
2. Apakah saya menyaring sebagian elemen? → **Filter**
3. Apakah saya ingin satu nilai dari keseluruhan? → **Reduce**

---

## 6. Laravel Collection — Map/Filter/Reduce yang Lebih Powerful

### 6.1 Collection vs Native Array

Laravel `Collection` membungkus array dengan method functional yang lebih lengkap:

```php
<?php
// Native PHP
$result = array_map(fn($p) => $p['name'],
    array_filter($products, fn($p) => $p['stock'] > 0)
);
// Urutan: filter dalam, map luar — tricky!

// Laravel Collection — urutan natural
$result = collect($products)
    ->filter(fn($p) => $p['stock'] > 0)
    ->map(fn($p) => $p['name'])
    ->values();
// Baca dari atas ke bawah: filter → map → values
```

### 6.2 Collection Method untuk Map/Filter/Reduce

| Collection | Native PHP | Guna |
|------------|-----------|------|
| `->map(fn)` | `array_map` | Transformasi |
| `->filter(fn)` | `array_filter` | Penyaringan |
| `->reduce(fn, init)` | `array_reduce` | Akumulasi |
| `->sum()` | `array_sum` | Jumlah |
| `->avg()` | — | Rata-rata |
| `->pluck('name')` | `array_column` | Ambil satu kolom |
| `->groupBy('cat')` | — | Grouping |
| `->sortBy('field')` | — | Sorting |
| `->firstWhere('k', 'v')` | — | Cari pertama |

### 6.3 Collection di Codebase

**ShopController.php:57 — Collection dipertahankan dari query:**
```php
$categories = Category::where('is_active', true)
    ->whereNull('parent_id')
    ->orderBy('sort_order')
    ->get();
// $categories adalah Collection
```

**ShopController.php:62 — Collection method:**
```php
$selectedCategory = $request->filled('category')
    ? $categories->firstWhere('slug', $request->category)
    : null;
```

---

## 7. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│ MAP                                                      │
│  [1, 2, 3] → fn *2 → [2, 4, 6]                         │
│  Jumlah tetap, bentuk berubah                            │
├──────────────────────────────────────────────────────────┤
│ FILTER                                                   │
│  [1, 2, 3, 4] → fn genap → [2, 4]                      │
│  Jumlah berkurang, isi yang cocok                        │
├──────────────────────────────────────────────────────────┤
│ REDUCE                                                   │
│  [1, 2, 3, 4] → fn + → 10                              │
│  Satu nilai dari keseluruhan                             │
├──────────────────────────────────────────────────────────┤
│ PIPELINE                                                 │
│  collect($data)                                          │
│    ->filter(fn($x) => $x > 0)    ← saring               │
│    ->map(fn($x) => $x * 2)       ← transform            │
│    ->reduce(fn($c, $x) => $c+$x, 0) ← akumulasi        │
├──────────────────────────────────────────────────────────┤
│ DI CODEBASE:                                             │
│  array_map       → (tidak langsung, via Collection)      │
│  array_filter    → RajaOngkirService::filter by province │
│  array_reduce    → CartService::getTotal/getCount/weight │
│  Collection      → ShopController::firstWhere            │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Map.** Dari array `[2, 4, 6, 8, 10]`, gunakan `array_map` untuk menghasilkan `[4, 16, 36, 64, 100]` (kuadrat).

2. **Filter.** Dari `$products`, filter yang harganya antara 10000 dan 50000:
```php
$products = [
    ['name' => 'A', 'price' => 5000],
    ['name' => 'B', 'price' => 25000],
    ['name' => 'C', 'price' => 75000],
];
```

3. **Reduce.** Gunakan `array_reduce` untuk menghitung total harga stok (`price * stock`):
```php
$items = [
    ['price' => 10000, 'stock' => 5],
    ['price' => 25000, 'stock' => 2],
    ['price' => 5000,  'stock' => 10],
];
```

4. **Pipeline.** Gabung filter + map + reduce: dari array produk, filter yang `is_active`, ambil nama, gabung dengan koma.

5. **Baca CartService.** Buka `app/Services/CartService.php`. Tiga method menggunakan `array_reduce`. Bisakah kamu tulis ulang menggunakan loop imperatif? Mana yang lebih jelas?

6. **Baca RajaOngkirService.** Buka `app/Services/RajaOngkirService.php` method `getDummyCities`. Jelaskan apa yang dilakukan `array_filter` + `array_values`.

---

## 🔗 Referensi

- [PHP Manual: array_map](https://www.php.net/manual/en/function.array-map.php)
- [PHP Manual: array_filter](https://www.php.net/manual/en/function.array-filter.php)
- [PHP Manual: array_reduce](https://www.php.net/manual/en/function.array-reduce.php)
- [Laravel Docs: Collections](https://laravel.com/docs/11.x/collections)
- Codebase: `app/Services/CartService.php` — array_reduce x3
- Codebase: `app/Services/RajaOngkirService.php` — array_filter + array_values
