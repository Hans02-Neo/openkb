# 02-03: Array dan String — Dua Tipe Data Paling Penting

> **Fase**: 2 — PHP Fundamental  
> **Prasyarat**: 02-02-sintaks-dasar-php  
> **Waktu baca**: 75-90 menit  
> **Kata kunci**: array, indexed array, associative array, multidimensional, array_push, array_map, array_filter, array_reduce, compact, string, heredoc, nowdoc, explode, implode, substr, Str::slug, JSON, serialisasi

---

## 📋 Ringkasan

Dokumen sebelumnya memperkenalkan semua sintaks dasar PHP. Sekarang kita fokus ke **dua tipe data yang paling sering kamu pakai**: array dan string.

Hampir semua data di aplikasi web adalah array dan string:
- Produk datang sebagai **array** (koleksi)
- JSON dari API adalah **string** yang perlu diubah ke **array**
- Nama, email, slug, alamat — semua **string**
- Keranjang belanja adalah **array asosiatif**

**Target pemahaman:**
- Kamu bisa membuat, membaca, mengubah, dan menghapus data di array
- Kamu paham perbedaan indexed vs associative array
- Kamu bisa memanipulasi string dengan fungsi bawaan PHP
- Kamu bisa mengubah array ↔ string sesuai kebutuhan
- Kamu paham dan bisa membaca kode array/string di codebase ini

---

## 1. Array — Koleksi Data dalam Satu Wadah

### 1.1 Apa Itu Array?

**Array** adalah struktur data yang menyimpan **banyak nilai** dalam **satu variabel**.

Bayangkan array seperti **loker**:

```
Loker (Array):
┌──────┬──────┬──────┬──────┬──────┐
│ Baju │ Celana│ Topi │ Jaket│ Sepatu│
│ [0]  │ [1]  │ [2]  │ [3]  │ [4]  │
└──────┴──────┴──────┴──────┴──────┘
```

Setiap "loker" punya **nomor indeks** (kunci) untuk mengakses isinya.

### 1.2 Indexed Array (Array Numerik)

Indexed array menggunakan **angka** sebagai kunci, dimulai dari **0**.

```php
<?php
// Cara 1: fungsi array()
$produk = array("Laptop", "Mouse", "Keyboard");

// Cara 2: syntax pendek [] — STANDAR modern, selalu pakai ini
$produk = ["Laptop", "Mouse", "Keyboard"];

// Mengakses elemen
echo $produk[0]; // "Laptop"
echo $produk[1]; // "Mouse"
echo $produk[2]; // "Keyboard"

// Di memory, array ini disimpan seperti:
// [0] => "Laptop"
// [1] => "Mouse"
// [2] => "Keyboard"
```

**Mengapa mulai dari 0?** Karena di level memori, indeks adalah **offset** (jarak) dari alamat awal array. Elemen pertama ada di offset 0, kedua di offset 1, dst.

### 1.3 Associative Array (Array Asosiatif)

Associative array menggunakan **string** sebagai kunci (key → value).

```php
<?php
$produk = [
    'id'    => 1,
    'nama'  => 'Laptop Gaming',
    'harga' => 15000000,
    'stok'  => 10,
];

echo $produk['nama'];  // "Laptop Gaming"
echo $produk['harga']; // 15000000
```

**Kenapa pakai string sebagai kunci?** Karena lebih deskriptif. `$produk['harga']` lebih jelas daripada `$produk[3]`.

**Di memory**, associative array diimplementasikan sebagai **hash table**:
- Key (`'nama'`) dilewatkan ke fungsi hash → menghasilkan alamat memori
- Value disimpan di alamat itu
- Pencarian O(1) — sangat cepat, tidak peduli seberapa besar array-nya

### 1.4 Multidimensional Array (Array di Dalam Array)

Array bisa berisi array lain — ini yang paling sering kamu temui di aplikasi nyata.

```php
<?php
$produk = [
    [
        'id'    => 1,
        'nama'  => 'Laptop Gaming',
        'harga' => 15000000,
    ],
    [
        'id'    => 2,
        'nama'  => 'Mouse Wireless',
        'harga' => 250000,
    ],
    [
        'id'    => 3,
        'nama'  => 'Keyboard Mechanical',
        'harga' => 750000,
    ],
];

echo $produk[0]['nama'];  // "Laptop Gaming"
echo $produk[1]['harga']; // 250000
```

### 1.5 Array di Codebase: Dummy Data RajaOngkir

Di `app/Services/RajaOngkirService.php:136-144`, ada contoh multidimensional array untuk data kota:

```php
$allCities = [
    ['city_id' => '39', 'province_id' => '5', 'province' => 'DI Yogyakarta', 'type' => 'Kabupaten', 'city_name' => 'Bantul', 'postal_code' => '55715'],
    ['city_id' => '418', 'province_id' => '5', 'province' => 'DI Yogyakarta', 'type' => 'Kota', 'city_name' => 'Yogyakarta', 'postal_code' => '55111'],
    ['city_id' => '152', 'province_id' => '6', 'province' => 'DKI Jakarta', 'type' => 'Kota', 'city_name' => 'Jakarta Pusat', 'postal_code' => '10540'],
    // ...
];
```

Setiap elemen adalah array asosiatif dengan key `city_id`, `province_id`, `province`, `type`, `city_name`, `postal_code`.

---

## 2. Operasi Dasar Array (CRUD)

### 2.1 Create — Menambah Elemen

```php
<?php
$cart = [];

// Menambah ke INDEXED array — otomatis dapat indeks berikutnya
$cart[] = ['id' => 1, 'qty' => 2]; // [0] => [...]

// Menambah ke ASSOCIATIVE array — pakai key
$cart[1] = ['id' => 2, 'qty' => 1]; // key 1

// array_push — menambah satu atau lebih elemen di akhir
array_push($cart, ['id' => 3, 'qty' => 5]);
```

**Di codebase — CartService.php:33-43:**

```php
$cart[$productId] = [
    'id'           => $product->id,
    'name'         => $product->name,
    'price'        => $product->price,
    'quantity'     => $quantity,
    'weight_grams' => $product->weight_grams ?? 0,
    'image'        => $product->getFirstMediaUrl('products'),
    'slug'         => $product->slug,
];
```

Perhatikan: `$productId` (dari database) dipakai sebagai **key**. Ini membuat update dan hapus lebih mudah — tinggal `$cart[$productId]`.

### 2.2 Read — Membaca Elemen

```php
<?php
$cart = [
    1 => ['name' => 'Laptop', 'price' => 15000000, 'quantity' => 1],
    2 => ['name' => 'Mouse',  'price' => 250000,   'quantity' => 2],
];

// Cek apakah key ada
if (isset($cart[1])) {
    echo $cart[1]['name']; // "Laptop"
}

// Cek key dengan array_key_exists (beda: null tetap dianggap ada)
if (array_key_exists(1, $cart)) {
    echo "Ada";
}
```

**Perbedaan `isset` vs `array_key_exists`:**
- `isset($arr[$key])` → false jika nilai `null` atau key tidak ada
- `array_key_exists($key, $arr)` → true jika key ada, meski nilainya `null`

### 2.3 Update — Mengubah Elemen

```php
<?php
$cart = [1 => ['name' => 'Laptop', 'quantity' => 1]];
$cart[1]['quantity'] = 3; // Ubah quantity jadi 3
```

**Di codebase — CartService.php:27-32:**

```php
if (isset($cart[$productId])) {
    $newQuantity = $cart[$productId]['quantity'] + $quantity;
    // validasi stok...
    $cart[$productId]['quantity'] = $newQuantity;
}
```

### 2.4 Delete — Menghapus Elemen

```php
<?php
$cart = [1 => ['name' => 'Laptop'], 2 => ['name' => 'Mouse']];

// Hapus satu elemen
unset($cart[1]);

// Hapus seluruh array
unset($cart);

// Hapus session cart
Session::forget('shopping_cart');
```

**Di codebase — CartService.php:69-78:**

```php
public function remove($productId)
{
    $cart = $this->getCart();
    if (isset($cart[$productId])) {
        unset($cart[$productId]); // ← hapus produk dari array
        Session::put($this->sessionKey, $cart);
    }
    return $cart;
}
```

---

## 3. Fungsi Array Esensial

### 3.1 `array_reduce` — Mereduksi Array Jadi Satu Nilai

`array_reduce($array, $callback, $initial)` menjalankan callback pada setiap elemen, membawa **nilai akumulator** dari iterasi ke iterasi.

```
Visual:

Array: [10000, 25000, 5000]
        │
        ▼
Step 1: total = 0 + 10000 = 10000
Step 2: total = 10000 + 25000 = 35000
Step 3: total = 35000 + 5000 = 40000
        │
        ▼
Hasil: 40000
```

**Di codebase — CartService.php:86-91:**

```php
public function getTotal()
{
    $cart = $this->getCart();
    return array_reduce($cart, function ($total, $item) {
        return $total + ($item['price'] * $item['quantity']);
    }, 0);
}
```

Analisis:
- `$cart` adalah array of items (multidimensional)
- `function ($total, $item)` — `$total` adalah akumulator (mulai dari 0), `$item` adalah setiap elemen array
- `$total + ($item['price'] * $item['quantity'])` — tambahkan subtotal item ke total
- `0` adalah nilai awal `$total`

Ada tiga fungsi `array_reduce` identik di CartService: `getTotal` (sum harga), `getItemCount` (sum quantity), `getTotalWeight` (sum weight × quantity).

### 3.2 `array_filter` — Menyaring Array

`array_filter($array, $callback)` mengembalikan hanya elemen yang memenuhi kondisi callback.

**Di codebase — RajaOngkirService.php:146-150:**

```php
return array_values(array_filter($allCities, function ($city) use ($provinceId) {
    return $city['province_id'] == $provinceId;
}));
```

Analisis:
- `$allCities` adalah array kota (multidimensional)
- Callback mengembalikan `true` hanya jika `province_id` cocok
- **Kenapa `array_values`?** Karena `array_filter` **mempertahankan kunci asli**. Jika kota indeks 3 dan 5 cocok, hasilnya `[3 => [...], 5 => [...]]`. `array_values` mereset ulang jadi `[0 => [...], 1 => [...]]`.

### 3.3 `array_map` — Mentransformasi Setiap Elemen

`array_map($callback, $array)` menjalankan callback pada setiap elemen dan mengembalikan array baru dengan hasilnya.

```php
<?php
$harga = [10000, 25000, 5000];

// Tambah PPN 11%
$denganPpn = array_map(function ($h) {
    return $h * 1.11;
}, $harga);

// Hasil: [11100, 27750, 5550]
```

### 3.4 `compact` — Membuat Array dari Variabel

`compact('var1', 'var2')` membuat array asosiatif dari variabel yang namanya disebutkan.

**Di codebase — ShopController.php:65:**

```php
return view('shop.index', compact('products', 'categories', 'brands', 'selectedCategory'));
```

Ini setara dengan:
```php
return view('shop.index', [
    'products'        => $products,
    'categories'      => $categories,
    'brands'          => $brands,
    'selectedCategory' => $selectedCategory,
]);
```

Alasan `compact` begitu populer di Laravel: **lebih pendek dan rapi** daripada menulis array secara manual.

### 3.5 `in_array` — Cek Keberadaan Nilai

```php
<?php
$status = 'processing';
if (in_array($status, ['pending', 'processing', 'shipped', 'completed'])) {
    // status valid
}
```

**Di codebase — OrderController.php:53:**

```php
if ($request->status === 'cancelled' && in_array($oldStatus, ['processing', 'shipped', 'completed'])) {
    foreach ($order->items as $item) {
        $product = Product::find($item->product_id);
        if ($product) {
            $product->increment('stock_quantity', $item->quantity);
        }
    }
}
```

Logika: Jika order dibatalkan dan status sebelumnya adalah salah satu yang sudah mengurangi stok (`processing`, `shipped`, `completed`), maka stok harus dikembalikan.

### 3.6 Fungsi Array Lain yang Wajib Dikuasai

| Fungsi | Kegunaan | Contoh |
|--------|----------|--------|
| `count($arr)` | Hitung jumlah elemen | `count($products)` |
| `array_keys($arr)` | Ambil semua key | `array_keys(['a' => 1, 'b' => 2])` → `['a', 'b']` |
| `array_values($arr)` | Ambil semua value (reset indeks) | `array_values($arr)` |
| `array_merge($a, $b)` | Gabung dua array | `array_merge($defaults, $input)` |
| `array_shift($arr)` | Ambil & hapus elemen pertama | `$first = array_shift($queue)` |
| `array_pop($arr)` | Ambil & hapus elemen terakhir | `$last = array_pop($queue)` |
| `sort($arr)` | Urutkan (ASC, modifikasi original) | `sort($names)` |
| `rsort($arr)` | Urutkan DESC | `rsort($prices)` |
| `asort($arr)` | Urutkan ASC, pertahankan key → value | `asort($products)` |
| `ksort($arr)` | Urutkan berdasarkan key | `ksort($data)` |
| `array_search($needle, $haystack)` | Cari nilai, return key pertama | `array_search(25000, $harga)` |
| `array_unique($arr)` | Hapus duplikat | `array_unique($names)` |
| `array_reverse($arr)` | Balik urutan | `array_reverse($sorted)` |

---

## 4. String — Teks dan Manipulasinya

### 4.1 String di Level Memori

String adalah **urutan byte** yang direpresentasikan sebagai teks.

```php
<?php
$nama = "Budi";

// Di memory (ASCII):
// B = 0x42 (1 byte)
// u = 0x75 (1 byte)
// d = 0x64 (1 byte)
// i = 0x69 (1 byte)
// Total: 4 byte
```

Di PHP, string bisa mencapai **2GB** (tergantung memory_limit).

### 4.2 Single Quotes (`'`) vs Double Quotes (`"`)

Ini adalah **perbedaan paling penting** yang wajib kamu hafal.

```php
<?php
$nama = "Budi";

// Single quotes ('') — literal, tidak memproses variabel
echo 'Halo, $nama';    // Output: Halo, $nama
echo 'Halo, ' . $nama; // Output: Halo, Budi (pakai concatenation)

// Double quotes ("") — memproses variabel dan escape sequences
echo "Halo, $nama";    // Output: Halo, Budi
echo "Halo, {$nama}";  // Output: Halo, Budi (dengan kurung kurawal — lebih aman)
```

**Escape sequences** yang hanya bekerja di double quotes:
- `\n` — baris baru (newline)
- `\t` — tab
- `\\` — backslash literal
- `\$` — dollar sign literal
- `\"` — double quote literal

**Aturan praktis:**
- Gunakan **single quotes** untuk string literal yang tidak perlu variabel
- Gunakan **double quotes** hanya saat perlu interpolasi variabel
- Di codebase ini, double quotes dipakai untuk: concatenation dengan variabel, string yang berisi kutipan tunggal, dan string multi-baris

### 4.3 Heredoc dan Nowdoc

Untuk string multi-baris yang panjang:

```php
<?php
// Heredoc — seperti double quotes (proses variabel)
$email = <<<EMAIL
Halo {$nama},

Terima kasih telah berbelanja di toko kami.
Pesanan Anda sedang diproses.

Salam,
Tim Koneksi Store
EMAIL;

// Nowdoc — seperti single quotes (tidak proses variabel)
$template = <<<'TEMPLATE'
<html>
<body>
    <h1>Selamat Datang</h1>
    <p>Ini template statis.</p>
</body>
</html>
TEMPLATE;
```

### 4.4 String di Codebase: Concatenation

**CartService.php:22** — double quotes untuk interpolasi di exception message:

```php
throw new \Exception("Stok produk '{$product->name}' tidak mencukupi. Tersisa: {$product->stock_quantity}");
```

**OrderService.php:45** — concatenation untuk membuat order number:

```php
'order_number' => 'ORD-' . strtoupper(Str::random(10)),
```

**MidtransService.php:23** — concatenation dengan timestamp:

```php
$midtransOrderId = $order->order_number . '-' . time();
```

---

## 5. Fungsi String Esensial

### 5.1 `substr` — Memotong String

```php
<?php
$teks = "Laptop Gaming Premium 2024";
echo substr($teks, 0, 6); // "Laptop" (mulai index 0, ambil 6 karakter)
echo substr($teks, 7);    // "Gaming Premium 2024" (index 7 sampai akhir)
```

**Di codebase — MidtransService.php:42:**

```php
'name' => substr($item->product_name, 0, 50),
```

Midtrans membatasi panjang nama item maksimal 50 karakter. `substr` memotong nama produk jika melebihi 50 karakter.

### 5.2 `strlen` dan `str_word_count`

```php
<?php
$teks = "Halo Dunia";
echo strlen($teks);        // 10 (termasuk spasi)
echo str_word_count($teks); // 2
```

### 5.3 `str_replace` dan `str_ireplace`

```php
<?php
$slug = "laptop-gaming-2024";
echo str_replace('-', ' ', $slug); // "laptop gaming 2024"
echo str_replace(['-', '2024'], [' ', ''], $slug); // "laptop gaming " (array pattern, array replacement)

// str_ireplace — case-insensitive version
echo str_ireplace('LAPTOP', 'KOMPUTER', $slug); // "komputer-gaming-2024"
```

### 5.4 `strpos` dan `str_contains`

```php
<?php
$teks = "Laptop Gaming Premium";

// strpos — cari posisi (return int atau false)
if (strpos($teks, "Gaming") !== false) {
    echo "Ketemu di posisi: " . strpos($teks, "Gaming"); // 7
}

// PHP 8: str_contains — return bool (LEBIH MUDAH)
if (str_contains($teks, "Gaming")) {
    echo "Ketemu";
}
```

### 5.5 `strtolower` dan `strtoupper`

```php
<?php
echo strtolower("BUDI"); // "budi"
echo strtoupper("budi"); // "BUDI"
echo ucfirst("budi");    // "Budi" (kapital huruf pertama)
echo lcfirst("Budi");    // "budi" (kecilkan huruf pertama)
```

**Di codebase — OrderService.php:45:**

```php
'order_number' => 'ORD-' . strtoupper(Str::random(10)),
// Hasil: "ORD-A7F3K2M9X1"
```

### 5.6 `trim`, `ltrim`, `rtrim`

```php
<?php
$input = "  budi@email.com  ";
echo trim($input);   // "budi@email.com" (hapus spasi kiri & kanan)
echo ltrim($input);  // "budi@email.com  " (hapus spasi kiri saja)
echo rtrim($input);  // "  budi@email.com" (hapus spasi kanan saja)
```

### 5.7 `nl2br` — Newline ke `<br>`

```php
<?php
$alamat = "Jl. Merdeka No. 1\nJakarta";
echo nl2br($alamat);
// Output: "Jl. Merdeka No. 1<br />Jakarta"
```

### 5.8 Fungsi String Modern: Laravel `Str` Helper

Laravel menyediakan class `Illuminate\Support\Str` yang lebih powerful daripada fungsi native PHP.

**Di codebase — ProductService.php:17-18:**

```php
if (empty($data['slug'])) {
    $data['slug'] = Str::slug($data['name']);
}
```

`Str::slug('Laptop Gaming 2024!')` → `laptop-gaming-2024`

Fungsi `Str` yang paling sering dipakai:

```php
<?php
use Illuminate\Support\Str;

Str::slug('Laptop Gaming!');            // "laptop-gaming"
Str::random(10);                        // "a7F3k2M9x1"
Str::limit($text, 100);                 // Potong 100 karakter + "..."
Str::of('hello world')->title();        // "Hello World" (fluent)
Str::of('hello')->upper();              // "HELLO" (fluent)
Str::of('  hello  ')->trim();           // "hello" (fluent)
Str::contains('hello world', 'world');  // true
Str::replace('world', 'php', 'hello world'); // "hello php"
```

**Fluent string (PHP 8 + Str::of):** Ini gaya baru — method chaining. Kamu bisa rangkai operasi string seperti pipeline:

```php
<?php
$result = Str::of('Laptop Gaming 2024!')
    ->slug()
    ->upper();
// Hasil: "LAPTOP-GAMING-2024"
```

---

## 6. Array ↔ String: Jembatan Penting

### 6.1 `implode` — Array → String

Gabung semua elemen array jadi satu string dengan separator.

```php
<?php
$items = ['Laptop', 'Mouse', 'Keyboard'];
echo implode(', ', $items); // "Laptop, Mouse, Keyboard"
echo implode(' | ', $items); // "Laptop | Mouse | Keyboard"
```

### 6.2 `explode` — String → Array

Pecah string berdasarkan separator.

```php
<?php
$teks = "Laptop, Mouse, Keyboard";
$items = explode(', ', $teks);
// ['Laptop', 'Mouse', 'Keyboard']

// Batasi jumlah pecahan
$parts = explode(', ', $teks, 2);
// ['Laptop', 'Mouse, Keyboard']
```

### 6.3 `json_encode` dan `json_decode`

JSON adalah format universal untuk pertukaran data. PHP punya fungsi bawaan untuk ini.

```php
<?php
$produk = [
    'id' => 1,
    'nama' => 'Laptop',
    'harga' => 15000000,
];

// Array → JSON string
$json = json_encode($produk);
// {"id":1,"nama":"Laptop","harga":15000000}

// JSON string → Array (parameter true = assoc array)
$kembali = json_decode($json, true);
// ['id' => 1, 'nama' => 'Laptop', 'harga' => 15000000]
```

**Di codebase — Setting.php:31 dan 43:**

```php
// Get: JSON → Array
'json' => json_decode($setting->value, true),

// Set: Array → JSON (hanya jika value adalah array)
['value' => is_array($value) ? json_encode($value) : $value, 'type' => $type]
```

Model `Setting` menyimpan nilai fleksibel — jika nilainya array, di-serialize ke JSON string sebelum disimpan ke database. Saat dibaca, di-decode kembali ke array.

### 6.4 `serialize` dan `unserialize`

Format serialisasi PHP (jangan dipakai untuk data yang perlu dibaca sistem lain — pakai JSON).

```php
<?php
$data = ['a' => 1, 'b' => 2];
$s = serialize($data);  // "a:2:{s:1:"a";i:1;s:1:"b";i:2;}"
$normal = unserialize($s); // ['a' => 1, 'b' => 2]
```

---

## 7. Kode Lengkap dari Codebase: Analisis Mendalam

### 7.1 CartService — Array Operations End-to-End

File `app/Services/CartService.php` adalah **studi kasus sempurna** untuk operasi array.

```php
<?php
namespace App\Services;

use App\Models\Product;
use Illuminate\Support\Facades\Session;

class CartService
{
    protected $sessionKey = 'shopping_cart';

    // READ — ambil array dari session (default: [] jika kosong)
    public function getCart()
    {
        return Session::get($this->sessionKey, []);
    }

    // CREATE / UPDATE — tambah atau update item
    public function add($productId, $quantity = 1)
    {
        $product = Product::findOrFail($productId);
        $cart = $this->getCart();

        if (isset($cart[$productId])) {
            // UPDATE: tambah quantity ke yang sudah ada
            $cart[$productId]['quantity'] += $quantity;
        } else {
            // CREATE: tambah item baru ke array
            $cart[$productId] = [
                'id'           => $product->id,
                'name'         => $product->name,
                'price'        => $product->price,
                'quantity'     => $quantity,
                'weight_grams' => $product->weight_grams ?? 0,
                'image'        => $product->getFirstMediaUrl('products'),
                'slug'         => $product->slug,
            ];
        }

        Session::put($this->sessionKey, $cart);
        return $cart;
    }

    // DELETE — hapus item dari array
    public function remove($productId)
    {
        $cart = $this->getCart();
        if (isset($cart[$productId])) {
            unset($cart[$productId]);
            Session::put($this->sessionKey, $cart);
        }
        return $cart;
    }

    // AGREGASI — array_reduce untuk kalkulasi
    public function getTotal()
    {
        $cart = $this->getCart();
        return array_reduce($cart, function ($total, $item) {
            return $total + ($item['price'] * $item['quantity']);
        }, 0);
    }
}
```

**Pola yang perlu diperhatikan:**
1. Cart disimpan sebagai **array asosiatif** dengan key = `$productId`
2. `isset($cart[$productId])` — O(1) lookup, sangat cepat
3. Setiap operasi (add, remove, update) selalu: get → modifikasi → put
4. Fungsi `getTotal` menggunakan `array_reduce` — pola fungsional, bukan loop manual

### 7.2 ShopController — Array Filtering dan Compact

```php
// app/Http/Controllers/ShopController.php

public function index(Request $request)
{
    $query = Product::with(['category', 'brand'])->where('is_active', true);

    // Filter berdasarkan kategori (jika ada di query string)
    if ($request->filled('category')) {
        $query->whereHas('category', function ($q) use ($request) {
            $q->where('slug', $request->category);
        });
    }

    // Search dengan LIKE (string operation)
    if ($request->filled('search')) {
        $query->where(function ($q) use ($request) {
            $q->where('name', 'like', '%' . $request->search . '%')
              ->orWhere('short_description', 'like', '%' . $request->search . '%');
        });
    }

    // Sorting dengan match expression
    $sort = $request->get('sort', 'latest');
    match ($sort) {
        'price_asc'  => $query->orderBy('price', 'asc'),
        'price_desc' => $query->orderBy('price', 'desc'),
        'name_asc'   => $query->orderBy('name', 'asc'),
        default      => $query->latest(),
    };

    // Pagination + compact
    $products   = $query->paginate(12)->appends($request->query());
    $categories = Category::where('is_active', true)->whereNull('parent_id')->orderBy('sort_order')->get();
    $brands     = Brand::where('is_active', true)->orderBy('name')->get();

    return view('shop.index', compact('products', 'categories', 'brands', 'selectedCategory'));
}
```

**Poin penting:**
- `$request->query()` mengembalikan **array asosiatif** dari semua parameter URL
- `->appends($request->query())` mempertahankan query string di link pagination
- `compact('products', 'categories', ...)` membuat array dari nama variabel

### 7.3 Order Model — String Match Expression

```php
// app/Models/Order.php

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

`match` adalah **expression** (bukan statement seperti `switch`), artinya selalu mengembalikan nilai. Ini lebih aman — tidak perlu `break`, dan throw `UnhandledMatchError` jika tidak ada case yang cocok (kecuali ada `default`).

### 7.4 MidtransService — String Truncation

```php
$itemDetails = [];
foreach ($order->items as $item) {
    $itemDetails[] = [
        'id'       => $item->product_id,
        'price'    => (int) $item->price,
        'quantity' => $item->quantity,
        'name'     => substr($item->product_name, 0, 50),
    ];
}
$params['item_details'] = $itemDetails;
```

Pola `foreach` → `$array[]` adalah cara paling umum di PHP untuk **membangun array dari query**.

---

## 8. Array Spreading dan Destructuring (PHP 7.4+ / 8+)

### 8.1 Spread Operator (`...`)

Spread operator "membongkar" array menjadi elemen-elemen individual.

```php
<?php
$default = ['status' => 'pending', 'currency' => 'IDR'];
$input = ['status' => 'completed', 'notes' => 'Express'];

$merge = [...$default, ...$input];
// ['status' => 'completed', 'currency' => 'IDR', 'notes' => 'Express']
// Input override default untuk key yang sama!
```

### 8.2 Array Destructuring

```php
<?php
$product = ['Laptop', 15000000, 10];

// Cara lama
$name = $product[0];
$price = $product[1];

// Cara modern — destructuring
[$name, $price, $stock] = $product;
echo $name;  // "Laptop"
echo $price; // 15000000

// Dengan array asosiatif
$data = ['id' => 1, 'name' => 'Laptop'];
['id' => $id, 'name' => $name] = $data;
echo $id;   // 1
echo $name; // "Laptop"
```

---

## 9. Perangkap Umum (Common Pitfalls)

### 9.1 String Interpolasi dengan Array

```php
<?php
$product = ['name' => 'Laptop'];

// ❌ SALAH — PHP bingung
echo "$product['name']";

// ✅ BENAR — pakai kurung kurawal
echo "{$product['name']}";
echo "{$product[name]}"; // juga OK, PHP cari constant 'name', fallback ke string 'name'
// Tapi JANGAN biasakan — selalu pakai kutip di dalam kurung: {$product['name']}
```

### 9.2 `foreach` dan Reference (&)

```php
<?php
$items = [
    ['qty' => 1, 'price' => 10000],
    ['qty' => 2, 'price' => 25000],
];

// ❌ BERBAHAYA — $item adalah reference ke elemen asli!
foreach ($items as &$item) {
    $item['total'] = $item['qty'] * $item['price'];
}
unset($item); // ← WAJIB! Hapus reference setelah loop

// ✅ AMAN — tanpa reference
foreach ($items as $key => $item) {
    $items[$key]['total'] = $item['qty'] * $item['price'];
}
```

**Kenapa `unset($item)` penting?** Setelah loop, `$item` masih jadi reference ke elemen terakhir array. Jika kamu tidak `unset`, dan kemudian `$item` dipakai lagi di kode lain secara tidak sengaja, elemen array asli bisa berubah.

### 9.3 `empty` vs `isset` untuk Array

```php
<?php
$cart = ['id' => 1, 'name' => null];

isset($cart['id']);     // true (ada, tidak null)
isset($cart['name']);   // false (ada tapi null!)
empty($cart['name']);   // true (null, '', 0, [], false dianggap empty)
array_key_exists('name', $cart); // true (ada meski null)
```

### 9.4 Concatenation vs Interpolasi — Mana Lebih Cepat?

```php
<?php
$name = "Budi";

// Concatenation
echo 'Halo, ' . $name . '!';

// Interpolasi
echo "Halo, {$name}!";
```

**Perbedaan performa:** Interpolasi sedikit lebih cepat karena PHP tidak perlu membuat string baru untuk setiap bagian. Tapi perbedaannya **tidak signifikan** untuk aplikasi nyata. Pilih berdasarkan **readability**.

### 9.5 Heresy Alert: Jangan Campur Array dan String

```php
<?php
// ❌ JANGAN — array jadi string otomatis (PHP Warning)
echo [1, 2, 3]; // "Array" — not useful!

// ✅ Always convert explicitly
echo implode(', ', [1, 2, 3]); // "1, 2, 3"
echo json_encode([1, 2, 3]);   // "[1,2,3]"
```

---

## 10. Ringkasan Visual

```
┌────────────────────────────────────────────────────┐
│                   ARRAY                            │
├────────────────────────────────────────────────────┤
│  Indexed: [0] => 'Laptop', [1] => 'Mouse'          │
│  Associative: ['nama' => 'Laptop', 'harga' => 2jt] │
│  Multidimensional: [['id'=>1], ['id'=>2]]          │
│                                                     │
│  Operasi: isset, unset, array_push, array_shift     │
│  Fungsi: array_map, array_filter, array_reduce      │
│          compact, in_array, count, array_merge      │
├────────────────────────────────────────────────────┤
│                   STRING                            │
├────────────────────────────────────────────────────┤
│  '' literal vs "" interpolasi                       │
│  Heredoc (proses var) vs Nowdoc (literal)           │
│                                                     │
│  Fungsi: substr, strlen, str_replace, str_contains  │
│          strtolower, strtoupper, trim, explode       │
│  Laravel: Str::slug, Str::random, Str::of (fluent)  │
├────────────────────────────────────────────────────┤
│          ARRAY ↔ STRING                             │
├────────────────────────────────────────────────────┤
│  implode  : array → string                          │
│  explode  : string → array                          │
│  json_encode : array → JSON string                  │
│  json_decode : JSON string → array                  │
└────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

Kerjakan di `php -a` (interactive shell) atau file `.php`:

1. Buat array asosiatif produk dengan `id`, `name`, `price`, `stock`. Tambah 3 produk.

2. Dari array di atas, gunakan `array_reduce` untuk menghitung total nilai stok (`price * stock`).

3. Buat fungsi yang menerima string nama dan mengembalikan slug (pakai `strtolower` + `str_replace` spasi jadi strip).

4. Dari array `['Laptop', 'Mouse', 'Keyboard']`, buat jadi string `"Laptop, Mouse, Keyboard"` lalu pecah lagi jadi array.

5. Baca `app/Services/CartService.php` dan jelaskan ke teman (atau ke diri sendiri) alur setiap fungsi.

---

## 🔗 Referensi

- [PHP Manual: Arrays](https://www.php.net/manual/en/language.types.array.php)
- [PHP Manual: String Functions](https://www.php.net/manual/en/ref.strings.php)
- [Laravel Str Helper](https://laravel.com/docs/11.x/helpers#fluent-strings)
- Codebase: `app/Services/CartService.php`
- Codebase: `app/Services/RajaOngkirService.php`
- Codebase: `app/Models/Order.php` (status label match)
- Codebase: `app/Services/MidtransService.php` (substr)
- Codebase: `app/Services/ProductService.php` (Str::slug)
- Codebase: `app/Http/Controllers/ShopController.php` (compact)
- Codebase: `app/Http/Controllers/Admin/OrderController.php` (in_array)
