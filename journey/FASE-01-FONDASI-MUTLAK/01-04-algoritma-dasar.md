# 01-04: Algoritma Dasar

> **Fase**: 1 — Fondasi Mutlak  
> **Prasyarat**: 01-03-variabel-tipe-data-logika  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: algoritma, control flow, if, else, switch, loop, for, while, foreach, complexity, pseudocode, flowchart, time complexity, Big O  

---

## 📋 Ringkasan  

Dokumen sebelumnya membahas **data** — variabel, tipe data, dan logika boolean.  
Sekarang kita bahas **algoritma** — bagaimana data **diproses**, **diubah**, dan **dikontrol** untuk menyelesaikan masalah.  

Kamu akan memahami:  
- Apa itu algoritma dan cara berpikir algoritmis  
- **Control flow** — menentukan jalur eksekusi program  
- **Conditional** (`if`, `else`, `switch`) — membuat keputusan  
- **Loop** (`for`, `while`, `foreach`) — mengulang instruksi  
- **Time complexity** — seberapa efisien algoritmamu  
- **Pseudocode & flowchart** — merancang sebelum coding  

**Target pemahaman:**  
- Kamu bisa membaca dan menulis `if/else`, `for`, `foreach`  
- Kamu bisa memilih kapan pakai `for` vs `foreach` vs `while`  
- Kamu paham konsep "algoritma yang baik vs yang buruk"  

---

## 1. Apa Itu Algoritma?  

### 1.1 Definisi  

**Algoritma** adalah **urutan langkah-langkah logis** untuk menyelesaikan suatu masalah.  

Istilah ini berasal dari nama **Al-Khawarizmi** (780-850 M), matematikawan Persia yang menulis buku tentang sistem bilangan desimal dan metode perhitungan sistematis.  

### 1.2 Syarat Algoritma yang Baik  

| Syarat | Penjelasan | Contoh Buruk | Contoh Baik |
|--------|-----------|-------------|-------------|
| **Input** | Punya masukan (bisa nol) | — | Data produk, ID user, keyword |
| **Output** | Menghasilkan keluaran | `proses(){}` — tidak ngapa-ngapain | `totalHarga($items){...}` |
| **Definiteness** | Setiap langkah jelas, tidak ambigu | "Hitung diskon sesuai kebijakan" | `harga * 0.1` (10%) |
| **Finiteness** | Berhenti dalam waktu terbatas | `while(true){}` (loop tak berujung) | `for($i=0;$i<10;$i++){}` |
| **Effectiveness** | Setiap langkah bisa dikerjakan | "Jalankan di quantum computer" | `echo $nama` |

### 1.3 Analogi Sehari-hari  

**Algoritma Memasak Nasi Goreng:**  

```
1. Siapkan bahan: nasi, telur, bawang, kecap     ← Input
2. Panaskan minyak                                ← Step 1
3. Tumis bawang hingga harum                      ← Step 2
4. Masukkan telur, orak-arik                      ← Step 3
5. Masukkan nasi, aduk rata                       ← Step 4
6. Tambahkan kecap, garam, aduk                   ← Step 5
7. Sajikan di piring                              ← Step 6 → Output
```

**Algoritma Login di Koneksi Store:**  

```
1. Terima input email dan password                ← Input
2. Cari user dengan email tersebut di database    ← Step 1
3. Jika user tidak ditemukan → tampilkan "Email tidak terdaftar"
4. Jika user ditemukan:
   a. Cek apakah password cocok
   b. Jika tidak cocok → tampilkan "Password salah"
   c. Jika cocok → buat session login
5. Redirect ke dashboard                          ← Output
```

---

## 2. Control Flow — Alur Eksekusi Program  

Tanpa control flow, program dieksekusi **berurutan dari atas ke bawah**.  

```php
<?php
echo "Langkah 1";
echo "Langkah 2";
echo "Langkah 3";
// Output: Langkah 1Langkah 2Langkah 3 — selalu sama
?>
```

Control flow memungkinkan program **mengambil keputusan** dan **mengulang** instruksi.  

```
Tanpa Control Flow:      Dengan Control Flow:

Start                    Start
  │                        │
  ▼                        ▼
[Step 1]               [Input umur]
  │                        │
  ▼                     ┌──┴──┐
[Step 2]               Ya    Tidak
  │                     │     │
  ▼                     ▼     ▼
[Step 3]            [Boleh] [Dilarang]
  │                     │     │
  ▼                     └──┬──┘
End                        │
                          ▼
                       End
```

---

## 3. Conditional — Pengambilan Keputusan  

### 3.1 if — "Jika ... maka ..."

```php
<?php
$umur = 17;

if ($umur >= 17) {
    echo "Kamu sudah boleh buat KTP";
}
// Output: (tidak ada) — karena 17 >= 17 true, tapi wait...
// 17 >= 17 itu true! Coba cek ulang...

$umur = 20;
if ($umur >= 17) {
    echo "Kamu sudah boleh buat KTP";
}
// Output: "Kamu sudah boleh buat KTP"
?>
```

**Struktur if dalam memori:**  

```
CPU mengeksekusi:
1. $umur = 20                 → simpan 20 di RAM
2. if ($umur >= 17)           → ALU bandingkan: 20 >= 17?
                              → CU lihat hasil: TRUE
3. { echo "KTP"; }           → CU ambil jalur TRUE → eksekusi blok ini

Jika $umur = 15:
1. $umur = 15                 → simpan 15 di RAM
2. if ($umur >= 17)           → ALU bandingkan: 15 >= 17?
                              → CU lihat hasil: FALSE
3. { echo "KTP"; }           → CU SKIP blok ini → lanjut ke kode berikutnya
```

### 3.2 if-else — "Jika ... maka ... jika tidak ..."

```php
<?php
$umur = 15;

if ($umur >= 17) {
    echo "Boleh buat KTP";
} else {
    echo "Belum cukup umur";
}
// Output: "Belum cukup umur"
?>
```

**Flowchart if-else:**  

```
        ┌──────────┐
        │ $umur=15  │
        └────┬─────┘
             │
        ┌────┴────┐
        │ $umur   │
        │ >= 17?  │
        └────┬────┘
       ┌─────┴─────┐
      Ya           Tidak
       │            │
       ▼            ▼
  "Buat KTP"  "Belum cukup"
       │            │
       └─────┬──────┘
             ▼
         Lanjut...
```

### 3.3 if-elseif-else — Banyak Kondisi  

```php
<?php
$nilai = 85;

if ($nilai >= 90) {
    $grade = "A";
} elseif ($nilai >= 80) {
    $grade = "B";         // ← ini yang dieksekusi (85 >= 80)
} elseif ($nilai >= 70) {
    $grade = "C";
} elseif ($nilai >= 60) {
    $grade = "D";
} else {
    $grade = "E";
}

echo $grade; // "B"
?>
```

**⚠️ Urutan kondisi penting!**  

```php
<?php
$nilai = 95;

// ❌ SALAH — kondisi pertama sudah true, sisanya tidak dievaluasi
if ($nilai >= 60) {      // 95 >= 60 = TRUE → stop di sini!
    $grade = "D";
} elseif ($nilai >= 70) {
    $grade = "C";
} elseif ($nilai >= 80) {
    $grade = "B";
} elseif ($nilai >= 90) {
    $grade = "A";
}
echo $grade; // "D" — PADAHAL harusnya A!

// ✅ BENAR — dari spesifik ke umum
if ($nilai >= 90) {
    $grade = "A";
} elseif ($nilai >= 80) {
    $grade = "B";
} elseif ($nilai >= 70) {
    $grade = "C";
} elseif ($nilai >= 60) {
    $grade = "D";
} else {
    $grade = "E";
}
echo $grade; // "A" — benar
?>
```

### 3.4 Nested if — if di dalam if  

```php
<?php
$isLoggedIn = true;
$isAdmin = false;
$isVerified = true;

if ($isLoggedIn) {
    if ($isAdmin) {
        echo "Selamat datang Admin!";
    } elseif ($isVerified) {
        echo "Selamat datang, User terverifikasi!";
    } else {
        echo "Silakan verifikasi email Anda.";
    }
} else {
    echo "Silakan login terlebih dahulu.";
}
// Output: "Selamat datang, User terverifikasi!"
?>
```

**Gunakan operator logika untuk nested if yang lebih bersih:**  

```php
<?php
// ❌ Nested if berlebihan
if ($isLoggedIn) {
    if ($isAdmin) {
        if ($isVerified) {
            // ...
        }
    }
}

// ✅ Flat dengan &&
if ($isLoggedIn && $isAdmin && $isVerified) {
    // ...
}

// ❌ if tanpa braces — rawan error!
if ($isLoggedIn)
    echo "Halo";
    echo "Dunia";      // ← baris ini SELALU dijalankan, bukan bagian dari if!

// ✅ Selalu pakai braces {}
if ($isLoggedIn) {
    echo "Halo";
    echo "Dunia";
}
?>
```

### 3.5 Ternary Operator — if-else dalam Satu Baris  

```php
<?php
$umur = 20;

// if-else biasa:
$status = "";
if ($umur >= 17) {
    $status = "Dewasa";
} else {
    $status = "Anak-anak";
}

// Ternary — sama persis:
$status = ($umur >= 17) ? "Dewasa" : "Anak-anak";
//        └─kondisi──┘ └─jika true─┘ └─jika false──┘

echo $status; // "Dewasa"
?>
```

**Struktur ternary:**  

```
$variabel = (kondisi) ? nilai_jika_true : nilai_jika_false;
```

**Di codebase ini — ternary sering digunakan di Blade:**  

```php
{{-- resources/views/shop/index.blade.php --}}
<div class="{{ $product->is_active ? 'bg-white' : 'bg-gray-200' }}">
    {{ $product->name }}
</div>
```

**Null coalescing operator (`??`) — ternary khusus null:**  

```php
<?php
// Jika $nama null, pakai 'Guest'
$displayName = $nama ?? 'Guest';
// Sama dengan: $displayName = isset($nama) ? $nama : 'Guest';

// Di Laravel — sangat sering dipakai:
$search = $request->input('q') ?? '';
$sortBy = $request->input('sort', 'created_at'); // dengan default
?>
```

### 3.6 Nullsafe Operator (PHP 8+) — "?->"  

```php
<?php
// ❌ Tanpa nullsafe — butuh banyak pengecekan
$city = null;
if ($user !== null) {
    if ($user->address !== null) {
        $city = $user->address->city;
    }
}

// ✅ Nullsafe — jika salah satu null, berhenti dan return null
$city = $user?->address?->city;
// Jika $user null → langsung null (tidak error)
// Jika $address null → null

// Di codebase:
$couponDiscount = $order?->coupon?->calculate($total);
?>
```

### 3.7 switch — Banyak Nilai Tetap  

Gunakan `switch` ketika kamu membandingkan **satu variabel** dengan **banyak nilai tetap**.  

```php
<?php
$role = "admin";

// if-else yang panjang:
if ($role === "admin") {
    $permission = "full";
} elseif ($role === "merchant") {
    $permission = "product_management";
} elseif ($role === "customer") {
    $permission = "view_only";
} else {
    $permission = "none";
}

// switch — lebih bersih untuk kasus ini:
switch ($role) {
    case "admin":
        $permission = "full";
        break;              // ← PENTING! tanpa break, lanjut ke case berikutnya
    case "merchant":
        $permission = "product_management";
        break;
    case "customer":
        $permission = "view_only";
        break;
    default:                // ← else
        $permission = "none";
}

// PHP 8+ — match expression (lebih modern):
$permission = match ($role) {
    "admin" => "full",
    "merchant" => "product_management",
    "customer" => "view_only",
    default => "none",
};
?>
```

**⚠️ Fall-through — jika lupa `break`:**  

```php
<?php
$nilai = "A";

switch ($nilai) {
    case "A":
        echo "Sangat Baik!";
        // tanpa break → LANJUT ke case B!
    case "B":
        echo "Baik";
        break;        // baru berhenti di sini
    default:
        echo "Cukup";
}
// Output: "Sangat Baik!Baik" — mungkin bukan yang diinginkan!
?>
```

Fall-through kadang **sengaja** digunakan:  

```php
<?php
$hari = "Sabtu";

switch ($hari) {
    case "Sabtu":
    case "Minggu":
        echo "Akhir pekan!";   // Sabtu DAN Minggu masuk ke sini
        break;
    default:
        echo "Hari kerja";
}
?>
```

---

## 4. Loop — Pengulangan Instruksi  

### 4.1 for — "Ulangi sebanyak N kali"  

Gunakan ketika kamu **tahu persis berapa kali** ingin mengulang.  

```php
<?php
// Hitung dari 1 sampai 5
for ($i = 1; $i <= 5; $i++) {
    echo "Perulangan ke-{$i}\n";
}
// Output:
// Perulangan ke-1
// Perulangan ke-2
// Perulangan ke-3
// Perulangan ke-4
// Perulangan ke-5
?>
```

**Anatomi `for`:**  

```
for ( [inisialisasi]; [kondisi]; [post-eksekusi] ) {
    // blok kode
}

    1               2           4
for ($i = 1;   $i <= 5;   $i++) {
    echo $i;                         // 3
}
```

**Urutan eksekusi:**  

```
Iterasi 1:
  1. Inisialisasi: $i = 1   (hanya sekali di awal!)
  2. Kondisi: 1 <= 5 → TRUE → masuk loop
  3. Eksekusi blok: echo 1
  4. Post: $i++ → $i = 2

Iterasi 2:
  2. Kondisi: 2 <= 5 → TRUE → masuk loop
  3. Eksekusi blok: echo 2
  4. Post: $i++ → $i = 3

... (iterasi 3-5)

Iterasi 6:
  2. Kondisi: 6 <= 5 → FALSE → KELUAR dari loop
```

**Visualisasi memori selama loop:**  

```
RAM:
┌────────────────────────────────────────────────────┐
│ $i: 1 → 2 → 3 → 4 → 5 → 6 (stop karena 6 > 5)    │
│                                                     │
│ Setiap iterasi, $i diubah dan dicek kondisinya.    │
│ Saat kondisi false, CPU loncat ke baris setelah }  │
└────────────────────────────────────────────────────┘
```

### 4.2 foreach — "Untuk setiap elemen dalam array"  

**Ini adalah loop yang paling sering kamu gunakan di Laravel.**  

```php
<?php
$buah = ["apel", "pisang", "jeruk"];

foreach ($buah as $item) {
    echo "Buah: {$item}\n";
}
// Output:
// Buah: apel
// Buah: pisang
// Buah: jeruk
?>
```

**Dengan key dan value (untuk array asosiatif):**  

```php
<?php
$produk = [
    "nama" => "Sepatu Nike",
    "harga" => 500000,
    "stok" => 10
];

foreach ($produk as $key => $value) {
    echo "{$key}: {$value}\n";
}
// Output:
// nama: Sepatu Nike
// harga: 500000
// stok: 10
?>
```

**Di Blade (Laravel template):**  

```php
{{-- resources/views/shop/index.blade.php --}}
@foreach($products as $product)
    <div class="product-card">
        <h2>{{ $product->name }}</h2>
        <p>Rp {{ number_format($product->price) }}</p>
        @if($product->stock > 0)
            <span class="badge-success">Tersedia</span>
        @else
            <span class="badge-danger">Habis</span>
        @endif
    </div>
@endforeach

{{-- Blade juga punya $loop variable: --}}
@foreach($products as $product)
    @if($loop->first)
        <div class="row-first">Produk Pertama!</div>
    @endif
    {{ $product->name }}
    @if($loop->last)
        <div class="row-last">Produk Terakhir!</div>
    @endif
    {{-- $loop->iteration (nomor urut), $loop->count (total) --}}
@endforeach
```

### 4.3 while — "Ulangi SELAGI kondisi true"  

Gunakan ketika kamu **tidak tahu** berapa kali akan mengulang.  

```php
<?php
$stok = 100;
$terjual = 0;

while ($stok > 0) {
    // Ada pembeli — kurangi stok
    $stok--;
    $terjual++;

    echo "Terjual {$terjual}, sisa stok: {$stok}\n";

    // Simulasi: kadang stok habis di tengah jalan
    if ($terjual >= 5) {
        break;  // ← hentikan loop paksa
    }
}
// Output:
// Terjual 1, sisa stok: 99
// Terjual 2, sisa stok: 98
// ...
// Terjual 5, sisa stok: 95
?>
```

### 4.4 do-while — "Kerjakan DULU, baru cek kondisi"  

```php
<?php
$stok = 0;

do {
    // Minimal 1 kali dijalankan — meskipun kondisi false!
    echo "Proses stok: {$stok}\n";
    $stok--;
} while ($stok > 0);

// Output: "Proses stok: 0" — dijalankan sekali
?>
```

### 4.5 break vs continue  

| Perintah | Fungsi | Contoh |
|----------|--------|--------|
| **`break`** | Keluar dari loop sepenuhnya | `break;` atau `break 2;` (keluar 2 level) |
| **`continue`** | Lompat ke iterasi berikutnya (skip sisa blok) | `continue;` |

```php
<?php
$products = [
    ["name" => "Sepatu", "price" => 500000],
    ["name" => "Tas",    "price" => 0],       // gratis? skip!
    ["name" => "Topi",   "price" => 75000],
    ["name" => "Baju",   "price" => 150000],
];

$total = 0;
foreach ($products as $product) {
    if ($product['price'] <= 0) {
        continue;   // ← skip produk gratis
    }

    if ($total + $product['price'] > 500000) {
        break;      // ← berhenti jika sudah melebihi budget
    }

    $total += $product['price'];
    echo "Tambah: {$product['name']} — Total: {$total}\n";
}
echo "Final total: {$total}";
?>
```

---

## 5. Contoh Algoritma — Dari Masalah ke Kode  

### 5.1 Menghitung Total Belanja  

**Masalah:** Hitung total harga dari array produk di keranjang.  
**Input:** Array produk dengan `price` dan `quantity`.  
**Output:** Integer total harga.  

```php
<?php
$cartItems = [
    ['name' => 'Sepatu',    'price' => 500000, 'qty' => 1],
    ['name' => 'Tas',       'price' => 250000, 'qty' => 2],
    ['name' => 'Topi',      'price' => 75000,  'qty' => 3],
];

// Algoritma:
// 1. Buat variabel total = 0
// 2. Untuk setiap item di cartItems:
//    2a. Hitung subtotal = price × qty
//    2b. Tambahkan subtotal ke total
// 3. Kembalikan total

function hitungTotal(array $items): int
{
    $total = 0;                                    // Step 1

    foreach ($items as $item) {                    // Step 2
        $subtotal = $item['price'] * $item['qty']; // Step 2a
        $total += $subtotal;                       // Step 2b
    }

    return $total;                                 // Step 3
}

echo hitungTotal($cartItems); // 725000
// Sepatu: 500000 × 1 = 500000
// Tas:    250000 × 2 = 500000
// Topi:   75000  × 3 = 225000
// Total:  500000 + 500000 + 225000 = 1225000
?>
```

**Di codebase — CartService::getTotal() melakukan hal yang sama:**  

```php
// app/Services/CartService.php (konsep serupa)
public function getTotal(): int
{
    $total = 0;
    $items = session()->get('cart', []);

    foreach ($items as $item) {
        $total += $item['price'] * $item['quantity'];
    }

    return $total;
}
```

### 5.2 Mencari Produk Termahal  

**Masalah:** Dari array produk, cari yang harganya paling mahal.  

```php
<?php
$products = [
    ['name' => 'Sepatu', 'price' => 500000],
    ['name' => 'Tas',    'price' => 250000],
    ['name' => 'Jam',    'price' => 1500000], // termahal
    ['name' => 'Topi',   'price' => 75000],
];

function cariTermahal(array $products): array
{
    if (empty($products)) {
        return [];                     // tidak ada produk
    }

    $termahal = $products[0];          // asumsikan produk pertama termahal

    foreach ($products as $product) {
        if ($product['price'] > $termahal['price']) {
            $termahal = $product;      // update jika ditemukan yang lebih mahal
        }
    }

    return $termahal;
}

$hasil = cariTermahal($products);
echo $hasil['name']; // "Jam"
?>
```

### 5.3 Filter Produk Aktif  

**Masalah:** Hanya tampilkan produk yang aktif dan punya stok.  

```php
<?php
function filterTersedia(array $products): array
{
    $tersedia = [];

    foreach ($products as $product) {
        if ($product['is_active'] && $product['stock'] > 0) {
            $tersedia[] = $product;   // tambahkan ke array hasil
        }
    }

    return $tersedia;
}

// Di Laravel — ini cukup dengan query builder:
$products = Product::where('is_active', true)
    ->where('stock', '>', 0)
    ->get();
?>
```

---

## 6. Algoritma Sederhana — Searching dan Sorting  

### 6.1 Linear Search — Cari Satu per Satu  

```php
<?php
$products = [
    ['sku' => 'SPT-001', 'name' => 'Sepatu Nike'],
    ['sku' => 'TAS-002', 'name' => 'Tas Eiger'],
    ['sku' => 'JAM-003', 'name' => 'Jam Tangan'],
];

function cariBySku(array $items, string $targetSku): ?array
{
    foreach ($items as $item) {
        if ($item['sku'] === $targetSku) {
            return $item;                  // ditemukan!
        }
    }
    return null;                           // tidak ditemukan
}

$hasil = cariBySku($products, 'TAS-002');
echo $hasil['name']; // "Tas Eiger"

// Kompleksitas: O(n) — dalam worst case, cek semua item
// n = jumlah produk. Kalau 1000 produk → maksimal 1000 kali cek
?>
```

### 6.2 Binary Search — Cari di Data Terurut  

Hanya bekerja jika data sudah **terurut**. Jauh lebih cepat.  

```
Data terurut: [10, 20, 30, 40, 50, 60, 70, 80, 90]
Cari: 70

Iterasi 1: Ambil tengah (50) → 70 > 50 → cari di kanan
Iterasi 2: Ambil tengah kanan (70) → 70 == 70 → DITEMUKAN!

Hanya 2 langkah! (vs 7 langkah linear search)

Kompleksitas: O(log n) — eksponensial lebih cepat
n=1000 → linear: 1000 cek, binary: 10 cek
n=1.000.000 → linear: 1jt cek, binary: 20 cek
```

---

## 7. Time Complexity — Mengukur Efisiensi Algoritma  

### 7.1 Kenapa Ini Penting?  

Dua algoritma bisa menghasilkan output yang sama, tapi **kecepatannya bisa berbeda ribuan kali lipat** untuk data besar.  

```php
<?php
// Algoritma A — O(n) — linear
function jumlahArrayA(array $angka): int
{
    $total = 0;
    foreach ($angka as $n) {
        $total += $n;           // 1 operasi per elemen
    }
    return $total;
}
// n=1000 → 1000 operasi
// n=1.000.000 → 1.000.000 operasi

// Algoritma B — O(n²) — kuadratik (JANGAN DIPAKAI!)
function jumlahArrayB(array $angka): int
{
    $total = 0;
    foreach ($angka as $a) {
        foreach ($angka as $b) {
            if ($a === $b) {
                $total += $a;   // n × n operasi!
            }
        }
    }
    return $total;
}
// n=1000 → 1.000.000 operasi!
// n=1.000.000 → 1 TRILLIUN operasi!
?>
```

### 7.2 Big O Notation — Bahasa Mengukur Kompleksitas  

| Notasi | Nama | Arti | Contoh Algoritma |
|--------|------|------|-----------------|
| **O(1)** | Konstan | Waktu tetap, berapapun data | Akses array index (`$arr[5]`) |
| **O(log n)** | Logaritmik | Waktu naik lambat seiring data naik | Binary search |
| **O(n)** | Linear | Waktu naik proporsional dengan data | Foreach, linear search |
| **O(n log n)** | Linearithmic | Waktu naik sedikit lebih cepat | Sortir (Merge Sort, Quick Sort) |
| **O(n²)** | Kuadratik | Waktu naik kuadrat — **HATI-HATI** | Nested loop, bubble sort |
| **O(2ⁿ)** | Eksponensial | Waktu naik eksplosif — **HINDARI** | Rekursi fibonacci tanpa memoization |

**Visualisasi pertumbuhan:**  

```
Waktu ▲
      │                         O(2ⁿ) — naik vertikal
      │                   /
      │              O(n²) — parabola
      │         /
      │    O(n) — garis lurus
      │   /
      │  O(log n) — melandai
      │ /
      │ O(1) — garis datar
      └───────────────────────▶ Jumlah Data (n)
```

### 7.3 Contoh di Codebase  

**O(n) — foreach untuk total harga:**  

```php
// CartService.php — O(n) linear
foreach ($items as $item) {
    $total += $item['price'] * $item['quantity'];
}
// Jika 10 item → 10 iterasi
// Jika 1000 item → 1000 iterasi
```

**O(n) — query ke database:**  

```php
// ProductController.php — O(n) untuk menampilkan
$products = Product::where('is_active', true)->get();
// Database juga punya kompleksitas sendiri (O(log n) dengan index)
```

**O(n²) — yang harus dihindari:**  

```php
// ❌ N+1 problem — O(1 + n) tapi terlihat seperti O(n²)
$orders = Order::all();            // 1 query
foreach ($orders as $order) {
    $items = $order->items;        // N query tambahan!
}

// ✅ Eager loading — O(2) konstan
$orders = Order::with('items')->get(); // 2 query total
```

---

## 8. Pseudocode — Mendesain Sebelum Coding  

**Pseudocode** adalah cara menulis algoritma dengan bahasa manusia — bukan bahasa pemrograman.  
Gunakan ini untuk **merencanakan** sebelum menulis kode.  

### Contoh Pseudocode untuk Fitur Checkout  

```
ALGORITMA: Proses Checkout
INPUT: Data pembeli, data keranjang, kode kupon (opsional)
OUTPUT: Order ID atau pesan error

MULAI
1.  Validasi input pembeli (nama, alamat, telepon)
    JIKA tidak valid → KEMBALIKAN error

2.  Ambil data keranjang dari session
    JIKA keranjang kosong → KEMBALIKAN error "Keranjang kosong"

3.  Hitung total harga:
    total = 0
    UNTUK setiap item di keranjang:
        subtotal = item.harga × item.kuantitas
        total = total + subtotal

4.  Jika ada kode kupon:
    VALIDASI kupon (masih berlaku? minimal belanja?)
    JIKA valid → hitung diskon → kurangi total
    JIKA tidak valid → KEMBALIKAN error "Kupon tidak valid"

5.  Hitung ongkos kirim (via API RajaOngkir)
    JIKA API gagal → KEMBALIKAN error "Gagal hitung ongkir"

6.  Buat order di database:
    order.buat(tanggal, total, status='pending')
    UNTUK setiap item:
        order_item.buat(order_id, produk_id, harga, qty)

7.  Kosongkan keranjang

8.  Redirect ke halaman pembayaran

SELESAI
```

**Pseudocode ini bisa langsung diterjemahkan ke kode Laravel.**  

### Flowchart — Visualisasi Algoritma  

```
                    ┌─────────┐
                    │  START  │
                    └────┬────┘
                         │
                         ▼
                    ┌─────────┐
                    │ Input   │
                    │ cart +  │
                    │ coupon  │
                    └────┬────┘
                         │
                    ┌────┴────┐
                    │ Cart    │
               ────▶│ kosong? │◀────
              Ya    └────┬────┘    │
               │         │ Tidak   │
               ▼         ▼         │
         ┌────────┐ ┌──────────┐   │
         │Error:  │ │ Hitung   │   │
         │Cart    │ │ total    │   │
         │kosong  │ └────┬─────┘   │
         └────────┘      │         │
                         ▼         │
                    ┌────┴────┐    │
                    │ Ada     │    │
                    │ kupon?  │    │
                    └────┬────┘    │
                   Ya    │ Tidak   │
                    │    │         │
                    ▼    ▼         │
              ┌────────┐           │
              │ Valid? │           │
              └───┬────┘           │
             Ya   │   Tidak        │
              │   ▼                │
              │  ┌────────┐        │
              │  │Error:  │        │
              │  │Kupon   │        │
              │  │invalid │        │
              │  └────────┘        │
              ▼                    │
         ┌──────────┐              │
         │ Hitung   │              │
         │ diskon   │              │
         └────┬─────┘              │
              │                    │
              ▼                    │
         ┌──────────┐              │
         │ Buat     │              │
         │ Order    │              │
         └────┬─────┘              │
              │                    │
              ▼                    │
         ┌──────────┐              │
         │ Redirect │              │
         │ Payment  │              │
         └────┬─────┘              │
              │                    │
              ▼                    │
         ┌─────────┐               │
         │  END    │───────────────┘
         └─────────┘
```

---

## 9. Strategi Menulis Algoritma — Problem Solving  

### 9.1 Empat Langkah Problem Solving  

1. **Pahami masalahnya** — Apa inputnya? Apa outputnya? Batasannya?  
2. **Rencanakan** — Pseudocode dulu, jangan langsung coding  
3. **Kerjakan** — Tulis kode  mengikuti rencana  
4. **Evaluasi** — Apakah output benar? Apakah bisa lebih efisien?  

### 9.2 Teknik Berpikir Algoritmis  

| Teknik | Cara | Contoh Kasus |
|--------|------|-------------|
| **Decomposition** | Pecah masalah besar jadi masalah kecil-kecil | "Checkout" → validasi → hitung total → kupon → ongkir → order |
| **Pattern Recognition** | Cari pola yang mirip dengan yang sudah pernah dilihat | "Filter produk" sama dengan "filter pengguna" |
| **Abstraction** | Fokus pada yang penting, abaikan detail | "Proses pembayaran" — tidak perlu tahu cara kerja bank |
| **Algorithm Design** | Tentukan langkah-langkah solusi | Pilih for/while/foreach yang tepat |

### 9.3 Contoh — Menerapkan Empat Langkah  

**Masalah:** Hitung jumlah produk yang stoknya di bawah 5 (low stock alert).  

**Langkah 1 — Pahami:**  
- Input: array produk, setiap produk punya `stock`  
- Output: integer (jumlah produk low stock)  

**Langkah 2 — Rencanakan (Pseudocode):**  
```
lowStockCount = 0
UNTUK setiap produk:
    JIKA produk.stock <= 5:
        lowStockCount = lowStockCount + 1
KEMBALIKAN lowStockCount
```

**Langkah 3 — Kerjakan (PHP):**  
```php
function countLowStock(array $products): int
{
    $count = 0;

    foreach ($products as $product) {
        if ($product['stock'] <= 5) {
            $count++;
        }
    }

    return $count;
}
```

**Langkah 4 — Evaluasi:**  
- O(n) — harus cek semua produk, tidak bisa lebih cepat  
- Bisakah dioptimasi? Di database, pakai query:  
  ```sql
  SELECT COUNT(*) FROM products WHERE stock <= 5
  ```  
- Di Laravel: `Product::where('stock', '<=', 5)->count()`  

---

## 10. Ringkasan — Algoritma dalam Satu Halaman  

```
ALGORITMA = Langkah-langkah logis untuk menyelesaikan masalah

─────────────────────────────────────────────────────

CONTROL FLOW — Mengontrol alur eksekusi

├── CONDITIONAL (Percabangan)
│   ├── if / else               — "Jika ... maka ..."
│   ├── elseif                  — Banyak kondisi
│   ├── switch / match          — Satu variabel, banyak nilai
│   ├── ternary (? :)           — If satu baris
│   └── null coalescing (??)    — If null satu baris
│
├── LOOP (Pengulangan)
│   ├── for                     — Tahu jumlah pasti
│   ├── foreach                 — Array (PALING SERING)
│   ├── while                   — Tidak tahu jumlah pasti
│   ├── do-while                — Minimal 1x eksekusi
│   ├── break                   — Keluar paksa
│   └── continue                — Skip iterasi
│
└── COMPLEXITY (Efisiensi)
    ├── O(1)   — Konstan        — Tercepat
    ├── O(log n) — Logaritmik   — Sangat cepat
    ├── O(n)   — Linear         — Normal (foreach)
    ├── O(n log n) — Linearithm — Sortir
    ├── O(n²)  — Kuadratik      — Lambat (nested loop)
    └── O(2ⁿ)  — Eksponensial   — Hindari!
```

### Yang Harus Kamu Ingat  

1. **Algoritma** = langkah logis untuk solve masalah  
2. **Pseudocode** dulu, baru coding — biasakan dari sekarang!  
3. **`if/else`** untuk keputusan — urutkan dari kondisi paling spesifik  
4. **`foreach`** adalah loop paling penting di Laravel — kuasai ini  
5. **`for`** ketika perlu counter numerik  
6. **`break`** untuk keluar loop, **`continue`** untuk skip  
7. **Operator `??`** dan **`?->`** akan kamu pakai setiap hari  
8. **Big O** — pahami O(n) vs O(n²), hindari nested loop tidak perlu  
9. **N+1 problem** = O(n) jadi O(n²) di database — selalu `with()`  
10. **Flowchart & pseudocode** membantu sebelum coding — biasakan  

---

## 📌 PRAKTIK — Kerjakan Ini  

```
✍️  Latihan 1 — if/else:
    php -a
    php> $nilai = 78;
    php> if ($nilai >= 90) { echo "A"; }
    php> elseif ($nilai >= 80) { echo "B"; }
    // ... teruskan sampai grade E

✍️  Latihan 2 — foreach:
    php> $produk = ["Sepatu" => 500000, "Tas" => 250000];
    php> foreach ($produk as $nama => $harga) {
    php>     echo "{$nama}: Rp {$harga}\n";
    php> }

✍️  Latihan 3 — Ternary:
    php> $umur = 20;
    php> $status = ($umur >= 17) ? "Dewasa" : "Anak-anak";
    php> echo $status;

✍️  Eksplorasi codebase:
    Buka routes/web.php — lihat route definitions
    Buka app/Http/Controllers/ShopController.php
    Cari: if, foreach, return
    Pahami alur logika dari controller tersebut

🔬  Buka resources/views/shop/index.blade.php
    Cari @foreach, @if, @endif, @endforeach
    Pahami bagaimana Blade template menggunakan control flow
    
🔬  Buka app/Models/Product.php
    Cari fungsi isLowStock() dan isAvailable()
    Ini adalah contoh algoritma sederhana dalam method
```

---

## 🔗 Referensi & Lanjutan  

| Topik | Di Journey Ini |
|-------|----------------|
| PHP function & scope | FASE-02: 02-04-fungsi-dan-scope |
| Array & collection di Laravel | FASE-06: 06-05-model-dan-eloquent-orm |
| Blade directives (@if, @foreach) | FASE-06: 06-06-blade-template-engine |
| Service pattern (algoritma di service) | FASE-08: 08-03-service-pattern |
| N+1 problem (query optimization) | FASE-07: 07-06-n-plus-one-problem |

---

## 🏁 Akhir Fase 1 — Selamat!  

Kamu telah menyelesaikan **FASE 1: Fondasi Mutlak** 🎉  

**Yang sudah kamu kuasai:**  
- ✅ Apa itu pemrograman dan cara kerja komputer  
- ✅ Arsitektur komputer: CPU, RAM, Storage, Cache  
- ✅ Variabel, tipe data, dan logika boolean  
- ✅ Algoritma, control flow, if/else, loop, Big O notation  

**Lanjut ke FASE 2: PHP Fundamental** untuk mulai menulis kode PHP yang sesungguhnya.  

---

*"Algoritma adalah seni memecahkan masalah —  
bukan tentang bahasa pemrograman, tapi tentang cara berpikir.  
Bahasa hanya alat; algoritma adalah ilmunya."*

*Praktikkan setiap latihan. Jangan hanya baca.*
