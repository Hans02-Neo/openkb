# 01-03: Variabel, Tipe Data, dan Logika

> **Fase**: 1 — Fondasi Mutlak  
> **Prasyarat**: 01-02-cara-kerja-komputer  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: variabel, memori, tipe data, integer, string, boolean, array, logika boolean, operator, truthy, falsy, representasi data, bit, byte, ASCII, Unicode  

---

## 📋 Ringkasan  

Dokumen sebelumnya menjelaskan **bagaimana komputer bekerja secara fisik**. Sekarang kita akan bahas **bagaimana data disimpan, direpresentasikan, dan dimanipulasi** di dalam komputer — ini adalah fondasi dari **semua** kode yang akan kamu tulis.  

Kamu akan memahami:  
- Apa sebenarnya **variabel** itu (dari sisi memori)  
- Semua **tipe data** yang ada di pemrograman  
- Bagaimana angka, huruf, true/false disimpan di dalam RAM  
- **Logika boolean** — dasar dari semua keputusan dalam kode  
- Operator perbandingan dan logika  

**Target pemahaman:**  
- Kamu bisa menjelaskan apa yang terjadi di RAM saat `$umur = 25` dieksekusi  
- Kamu paham perbedaan integer, float, string, boolean, array  
- Kamu mengerti bagaimana komputer membandingkan dua nilai  

---

## 1. Variabel — "Kotak" di Dalam Memori  

### 1.1 Definisi Paling Dasar  

**Variabel** adalah "tempat" di memori (RAM) yang menyimpan data dan bisa diubah-ubah isinya.  

```
Bayangkan RAM sebagai kompleks gudang dengan ribuan unit.

Setiap unit punya:
- Alamat (contoh: 0x7FFD4C8A)
- Kapasitas (8 byte, 4 byte, dll)
- Isi (data yang disimpan)

Variabel adalah LABEL yang kamu tempel di satu unit gudang itu.
Dengan label, kamu tidak perlu hafal alamat — cukup panggil labelnya.
```

### 1.2 Analogi Kotak  

```
RAM (Memori) :
┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ 0  │ 1  │ 2  │ 3  │ 4  │ 5  │ 6  │ 7  │ 8  │ 9  │ ...
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘

Saat kamu tulis: $umur = 25;

Yang terjadi:
1. PHP minta 1 blok memori (4 byte) dari OS → dapat alamat 0x7FFD4C8A
2. PHP catat: "variabel 'umur' ada di alamat 0x7FFD4C8A"
3. Angka 25 (dalam binary: 00000000 00000000 00000000 00011001) disimpan di alamat itu

RAM:
┌──────────────────────────────────────────────┐
│ ... | 00011001 | 00000000 | 00000000 | ...   │
│      └─ 4 byte untuk $umur = 25 ──┘          │
│ Alamat: 0x7FFD4C8A                            │
└──────────────────────────────────────────────┘

Saat kamu panggil echo $umur;
1. PHP cari alamat untuk label "umur" → 0x7FFD4C8A
2. Baca 4 byte dari alamat itu → 00000000 00000000 00000000 00011001
3. Terjemahkan ke desimal → 25
4. Kirim ke output: "25"
```

### 1.3 PHP — Variabel Tidak Perlu Dideklarasikan Tipenya  

PHP adalah **dynamically typed language** — kamu tidak perlu bilang "ini integer" atau "ini string". PHP akan menentukan tipe data secara otomatis saat kamu mengisi nilai.  

```php
<?php
$umur = 25;           // PHP tahu: ini integer (angka)
$nama = "Fathan";     // PHP tahu: ini string (teks)
$isActive = true;     // PHP tahu: ini boolean (true/false)
$harga = 25000.50;    // PHP tahu: ini float (desimal)
?>
```

**Yang terjadi di belakang layar:**  

```
Saat PHP membaca: $umur = 25;

1. Parser PHP menemukan: "$umur"
   → "Ini variabel baru, beri nama 'umur'"

2. Parser PHP menemukan: "= 25"
   → "Nilainya adalah angka 25 (tanpa kutip = angka/bilangan)"

3. PHP alokasikan 8 byte di memori (karena integer 64-bit)
   → Simpan angka 25 dalam binary

4. PHP catat di symbol table:
   [umur] → { type: "integer", value: 25, address: 0x7FFD4C8A }
```

### 1.4 Symbol Table — Buku Alamat PHP  

PHP menyimpan semua variabel dalam **symbol table** — seperti buku alamat raksasa.  

```
Symbol Table PHP (ada di RAM, selama script jalan):
┌────────────────────────────────────────────────────────────┐
│ Nama Variabel │ Tipe     │ Alamat Memori │ Nilai           │
├────────────────────────────────────────────────────────────┤
│ umur          │ integer  │ 0x7FFD4C8A    │ 25              │
│ nama          │ string   │ 0x7FFD5B2F    │ "Fathan"        │
│ isActive      │ boolean  │ 0x7FFD6C10    │ true            │
│ harga         │ float    │ 0x7FFD7D11    │ 25000.50        │
└────────────────────────────────────────────────────────────┘
```

Setiap kali kamu menyebut `$umur`, PHP:  
1. Cek symbol table → cari "umur"  
2. Dapat alamat memori → 0x7FFD4C8A  
3. Baca nilai dari alamat tersebut  
4. Kembalikan nilainya ke kode kamu  

### 1.5 Assignment (=) — "Isi Kotak"  

Tanda `=` dalam pemrograman **BUKAN** sama dengan "sama dengan" di matematika.  
Dia berarti **"isi/assign nilai ke variabel"** (assignment).  

```php
<?php
$x = 10;  // "Isi variabel x dengan angka 10"
$x = $x + 5;  // "Ambil x (10), tambah 5, simpan hasilnya (15) ke x"

echo $x;  // Output: 15
?>
```

Di matematika: `x = x + 5` adalah tidak masuk akal (x tidak mungkin sama dengan x+5).  
Di pemrograman: artinya **"ambil nilai x yang lama, tambah 5, simpan sebagai nilai x yang baru"**.

Visualisasi memori:
```
Step 1: $x = 10
  RAM: [x: 10]

Step 2: $x = $x + 5
  RAM: [x: 10]  → baca x (10) → hitung 10 + 5 = 15 → simpan ke x
  RAM: [x: 15]  ← nilai lama 10 TIMPA dengan 15
```

### 1.6 Naming Convention — Aturan Penamaan Variabel  

Di PHP (dan banyak bahasa lain):  

| Aturan | Contoh Valid | Contoh Invalid |
|--------|-------------|----------------|
| Mulai dengan `$` | `$nama` | `nama` (tanpa $) |
| Setelah $: huruf atau underscore | `$_nama`, `$nama` | `$1nama` |
| Case-sensitive | `$Nama` ≠ `$nama` | — |
| Hanya huruf, angka, underscore | `$nama_lengkap` | `$nama-lengkap` (strip) |

**Best practice penamaan variabel (kamu harus biasakan dari sekarang):**

```php
<?php
// ❌ BURUK — tidak bermakna, tidak konsisten
$a = 25;
$b = "Fathan";
$c = true;
$harga_setelah_diskon_dengan_kode = 9000;

// ✅ BAIK — deskriptif, konsisten
$umur = 25;
$namaLengkap = "Fathan";       // camelCase
$pelangganAktif = true;        // camelCase
$hargaSetelahDiskon = 9000;    // jelas maksudnya
?>
```

**Ada dua gaya utama penamaan:**

| Gaya | Contoh | Digunakan di |
|------|--------|-------------|
| **camelCase** | `$namaLengkap`, `$hargaProduk` | PHP (variabel fungsi), JavaScript |
| **snake_case** | `$nama_lengkap`, `$harga_produk` | PHP (variabel saja — banyak legacy code) |
| **PascalCase** | `$NamaLengkap`, `$HargaProduk` | Nama Class (di OOP) |
| **kebab-case** | `nama-lengkap` | URL, CSS class — **tidak valid untuk PHP variabel** |

**Di codebase ini (olshop-koneksi):**  
- Variabel biasa: `camelCase` — lihat di controller: `$request`, `$products`, `$totalAmount`  
- Nama class: `PascalCase` — `ProductController`, `CartService`, `User`  
- Nama tabel database: `snake_case` — `order_items`, `product_images`  

---

## 2. Tipe Data Fundamental  

### 2.1 Integer (Bilangan Bulat)  

Menyimpan bilangan bulat (positif, negatif, atau nol) tanpa desimal.  

```php
<?php
$umur = 25;              // desimal (basis 10)
$suhu = -5;              // negatif
$angkasa = 0;            // nol
$hex = 0x1A;             // 26 dalam heksadesimal (basis 16)
$bin = 0b11001;          // 25 dalam binary (basis 2)
$oktal = 031;            // 25 dalam oktal (basis 8)

// Batasan integer di PHP 64-bit:
// -2147483648  sampai  2147483647  (32-bit)
// atau
// -9223372036854775808  sampai  9223372036854775807  (64-bit)

// Kalau melebihi batas → otomatis jadi float:
$besar = 9223372036854775808;  // overflow → jadi float
var_dump($besar); // float(9.2233720368548E+18)
?>
```

**Representasi dalam memori (integer 64-bit — 8 byte):**  

```
Angka 25 dalam binary (64-bit / 8 byte):
┌──────────────────────────────────────────────────────────┐
│ 00000000 00000000 00000000 00000000 00000000 00000000   │
│ 00000000 00011001                                        │
└──────────────────────────────────────────────────────────┘
  ↑ byte ke-1 (MSB)                    ↑ byte ke-8 (LSB)

MSB = Most Significant Byte (byte paling "berat")
LSB = Least Significant Byte (byte paling "ringan")

Angka negatif: menggunakan two's complement
-25 = ~25 + 1 = bit 25 dibalik semua + 1
     = 11111111 11111111 11111111 11111111 11111111 11111111
       11111111 11100111
```

### 2.2 Float / Double (Bilangan Desimal)  

Menyimpan bilangan pecahan. Juga disebut **floating point** — karena titik desimalnya bisa "mengapung" (berpindah).  

```php
<?php
$harga = 25000.50;
$pi = 3.14159265359;
$jarak = -12.75;
$notasiIlmiah = 1.5e3;  // 1.5 × 10³ = 1500
$notasiIlmiah2 = 2e-3;  // 2 × 10⁻³ = 0.002

// ⚠️ PERINGATAN: Float tidak presisi untuk pecahan tertentu!
echo 0.1 + 0.2;         // 0.30000000000000004 ← BUKAN 0.3!
var_dump(0.1 + 0.2 == 0.3); // bool(false) ← INI BERBAHAYA!

// Mengapa? Karena 0.1 dalam binary adalah pecahan berulang:
// 0.1 (desimal) = 0.00011001100110011... (binary) → tidak pernah tepat!
?>
```

**Representasi float di memori (IEEE 754 — 8 byte / 64-bit):**  

```
┌──────────────────────────────────────────────────────┐
│ [1 bit] [11 bit]        [52 bit]                     │
│ Sign    Exponent        Mantissa/Fraction            │
│ (+/−)   (pangkat 2)     (angka signifikan)           │
│                                                      │
│ Contoh 25.5:                                         │
│ 25 = 11001 binary                                    │
│ 0.5 = 0.1 binary                                     │
│ 25.5 = 11001.1 binary = 1.10011 × 2⁴                │
│                                                      │
│ Sign: 0 (+)                                          │
│ Exponent: 4 + 1023 = 1027 → 10000000011              │
│ Mantissa: 100110000000... (52 bit)                  │
└──────────────────────────────────────────────────────┘
```

**Aturan praktis untuk float:**  

Jangan pernah bandingkan float dengan `==` langsung.  
Gunakan toleransi (epsilon):  

```php
<?php
$hasil = 0.1 + 0.2;  // 0.30000000000000004

// ❌ SALAH
if ($hasil == 0.3) { ... } // false!

// ✅ BENAR — pakai toleransi
if (abs($hasil - 0.3) < 0.0001) { ... } // true
?>
```

Di Laravel, untuk uang **jangan pakai float**.  
Selalu gunakan **integer** (dalam sen) atau library khusus:  

```php
<?php
// ❌ BURUK — float untuk uang
$total = 25000.50;    // bisa error presisi!

// ✅ BAIK — integer dalam satuan terkecil (sen)
$total = 2500050;     // Rp 25.000,50 dalam sen

// Di Laravel, ada kolom tipe decimal di migration:
// $table->decimal('price', 10, 2); // 10 digit, 2 desimal
?>
```

### 2.3 String (Teks)  

Menyimpan rangkaian karakter (huruf, angka, simbol).  

```php
<?php
$nama = "Fathan";                 // double quote
$email = 'fathan@example.com';    // single quote
$kosong = "";                     // string kosong
$kalimat = "Halo, $nama!";        // string interpolation (hanya "")
$kalimat2 = 'Halo, $nama!';       // output: Halo, $nama! (tidak di-interpolasi)

// Escape characters (hanya di "")
$teks = "Baris 1\nBaris 2";       // \n = newline
$path = "C:\\xampp\\htdocs";      // \\ = backslash literal
$quote = "Dia bilang \"Halo\"";    // \" = double quote literal
?>
```

**Perbedaan `""` dan `''` di PHP:**  

| Aspek | Double Quote (`""`) | Single Quote (`''`) |
|-------|-------------------|-------------------|
| **Interpolasi variabel** | ✅ `"Halo $nama"` → "Halo Fathan" | ❌ `'Halo $nama'` → "Halo $nama" |
| **Escape character** | ✅ `\n`, `\t`, `\\`, `\"` | ✅ hanya `\\` dan `\'` |
| **Kecepatan** | Lebih lambat (harus parsing) | Lebih cepat (tidak perlu parsing) |
| **Kapan pakai?** | Ada variabel/escape | Teks statis |

**Representasi string di memori:**  

```
$nama = "Fathan";

RAM:
┌────┬────┬────┬────┬────┬────┬────┐
│ 70 │ 97 │116 │104 │97  │110 │ 0  │
└────┴────┴────┴────┴────┴────┴────┘
  F    a    t    h    a    n   \0 (null terminator)

Setiap karakter = 1 byte (ASCII) atau lebih (UTF-8)

PHP juga menyimpan metadata string:
{
  value: "Fathan",
  length: 6,           // jumlah karakter
  type: "string"
}
```

#### ASCII vs Unicode  

**ASCII (American Standard Code for Information Interchange):**  
- 7-bit → 128 karakter (A-Z, a-z, 0-9, simbol dasar)  
- Hanya mencakup karakter Bahasa Inggris  
- 1 karakter = 1 byte  

**Unicode (UTF-8):**  
- Mencakup semua karakter dunia (150.000+ karakter)  
- 1 karakter = 1-4 byte (variable-length)  
- **Ini yang digunakan PHP modern dan Laravel**  

```php
<?php
// ASCII — 1 byte per karakter
$ascii = "Hello";      // 5 byte

// UTF-8 — 2-4 byte per karakter untuk non-ASCII
$indonesia = "Saya makan nasi";    // karakter latin → 1 byte
$emoji = "🔥";                       // 4 byte!
$jepang = "こんにちは";               // 3 byte per karakter
$arab = "مرحبا";                    // 2 byte per karakter

// Fungsi PHP untuk string multi-byte:
echo strlen($emoji);        // 4 (salah — hitung byte)
echo mb_strlen($emoji);     // 1 (benar — hitung karakter)

// Di Laravel, selalu gunakan helper Str:
use Illuminate\Support\Str;
echo Str::length($emoji);   // 1
?>
```

*Penting untuk web development Indonesia:* data pengguna bisa berisi  
karakter UTF-8 (nama dengan aksen, emoji di alamat, dll).  
Selalu gunakan `mb_*` functions atau Laravel `Str` helper.  

### 2.4 Boolean (Logika)  

Hanya punya dua nilai: `true` atau `false`. Ini adalah fondasi dari semua **keputusan** dalam pemrograman.  

```php
<?php
$isActive = true;
$isAdmin = false;
$isLoggedIn = (5 > 3);  // true — hasil perbandingan
$isEqual = (10 == 10);  // true
?>
```

**Representasi boolean di memori:**  

```
PHP: bool(true)  disimpan sebagai integer 1 di memori (1 byte)
PHP: bool(false) disimpan sebagai integer 0 di memori (1 byte)

RAM:
┌────┐
│ 01 │  ← true
└────┘

┌────┐
│ 00 │  ← false
└────┘
```

### 2.5 Null — "Tidak Ada Nilai"  

**Null** berarti variabel ada tapi **tidak punya nilai**.  

```php
<?php
$data = null;          // sengaja dikosongkan
$belumDiisi;           // null — belum di-assign (akan warning)
$hasil = 5 + null;     // 5 — null dianggap 0 oleh PHP

var_dump($data);       // NULL
var_dump($belumDiisi); // NULL + Warning: Undefined variable

// Cara cek null:
if (is_null($data)) { ... }
if ($data === null) { ... }
if (!$data) { ... }          // null = falsy
?>
```

**Penting:** Null **bukan** 0, bukan string kosong, bukan false.  

```php
<?php
0       !== null;   // true — 0 adalah angka
""      !== null;   // true — string kosong bukan null
false   !== null;   // true — false adalah boolean
null    === null;   // true — hanya null yang sama dengan null
?>
```

### 2.6 Array (Kumpulan Data)  

Array menyimpan **banyak nilai dalam satu variabel**.  

```php
<?php
// Array index (dimulai dari 0)
$buah = ["apel", "pisang", "jeruk"];
echo $buah[0];  // "apel"
echo $buah[1];  // "pisang"
echo $buah[2];  // "jeruk"

// Array asosiatif (key => value)
$produk = [
    "nama" => "Sepatu Nike",
    "harga" => 500000,
    "stok" => 10
];
echo $produk["nama"];   // "Sepatu Nike"
echo $produk["harga"];  // 500000

// Array multidimensi (array di dalam array)
$produkList = [
    ["nama" => "Sepatu", "harga" => 500000],
    ["nama" => "Tas",    "harga" => 250000],
    ["nama" => "Topi",   "harga" => 75000]
];
echo $produkList[0]["nama"];  // "Sepatu"
echo $produkList[1]["harga"]; // 250000
?>
```

**Representasi array di memori:**  

```
$buah = ["apel", "pisang", "jeruk"];

Di RAM, PHP menyimpan array sebagai hash table (ordered map):

{
  "buah": {
    type: "array",
    data: {
      0: { type: "string", value: "apel",   address: 0xA1 },
      1: { type: "string", value: "pisang", address: 0xA2 },
      2: { type: "string", value: "jeruk",  address: 0xA3 }
    },
    length: 3,
    address: 0xB0
  }
}
```

Array di PHP adalah **ordered map** — menghubungkan key dengan value.  
Key bisa integer (index array) atau string (associative array).  

Ini akan menjadi struktur data yang **paling sering** kamu gunakan di Laravel.  
Data dari database selalu dalam bentuk array (collection):  

```php
<?php
// Di Laravel — hasil query adalah array of objects
$products = Product::where('active', true)->get();
// [ { id: 1, name: "Sepatu", ... }, { id: 2, name: "Tas", ... } ]

foreach ($products as $product) {
    echo $product->name;  // akses property object
    echo $product['name']; // atau akses seperti array
}
?>
```

---

## 3. Logika Boolean — Dasar Semua Keputusan  

### 3.1 Operator Perbandingan  

Membandingkan dua nilai → menghasilkan **boolean** (true/false).  

```php
<?php
$a = 10;
$b = 5;
$c = "10";

// Sama dengan
$a == $b;        // false (10 ≠ 5)
$a == $c;        // TRUE! (loose equality — PHP konversi otomatis tipe)

// Sama dengan + cek tipe (IDENTIK)
$a === $c;       // false! (10 integer ≠ "10" string)

// Tidak sama
$a != $b;        // true
$a != $c;        // false (loose)
$a !== $c;       // true (strict — tipe berbeda)

// Lebih besar / lebih kecil
$a > $b;         // true (10 > 5)
$a < $b;         // false (10 < 5)
$a >= 10;        // true
$a <= 5;         // false
?>
```

**⚠️ Penting: `==` vs `===`**  

Ini adalah sumber bug paling umum di PHP.  

```php
<?php
// LOOSE EQUALITY (==) — konversi tipe otomatis (type juggling)
0 == false;      // true!   0 dianggap sama dengan false
0 == "0";        // true!   string "0" dianggap sama dengan 0
"" == false;     // true!   string kosong dianggap false
"123" == 123;    // true!   string dikonversi ke angka
null == false;   // true!   null dianggap false
null == 0;       // TRUE!!  (ini paling berbahaya)

// STRICT EQUALITY (===) — cek nilai DAN tipe
0 === false;     // false — integer ≠ boolean
0 === "0";       // false — integer ≠ string
"" === false;    // false — string ≠ boolean
"123" === 123;   // false — string ≠ integer
null === false;  // false — null ≠ boolean
null === 0;      // false — null ≠ integer

// ✅ DI LARAVEL: selalu gunakan === kecuali ada alasan spesifik
if ($user->role === 'admin') { ... }  // benar
if ($user->role == 'admin') { ... }   // bisa error type juggling
?>
```

### 3.2 Operator Logika  

Menggabungkan beberapa kondisi boolean.  

```php
<?php
$isLoggedIn = true;
$isAdmin = false;
$umur = 17;

// AND (&&) — SEMUA harus true
$bisaAksesAdmin = $isLoggedIn && $isAdmin;    // false
// $bisaAksesAdmin = true && false → false

// OR (||) — SALAH SATU true saja cukup
$bisaMasuk = $isLoggedIn || $isAdmin;          // true
// $bisaMasuk = true || false → true

// NOT (!) — Membalikkan nilai
$isGuest = !$isLoggedIn;                        // false
// $isGuest = !true → false

// Kombinasi:
$bisaBeliMinumanKeras = ($umur >= 21) && ($isLoggedIn);
// (17 >= 21) && true = false && true = false
?>
```

**Tabel kebenaran (truth table):**  

| A | B | A && B | A \|\| B | !A |
|---|---|--------|----------|----|
| true | true | true | true | false |
| true | false | false | true | false |
| false | true | false | true | true |
| false | false | false | false | true |

### 3.3 Short-Circuit Evaluation  

PHP mengevaluasi ekspresi logika secara **short-circuit** — jika hasil sudah bisa ditentukan dari bagian pertama, bagian kedua tidak dievaluasi.  

```php
<?php
function ambilDataDariDatabase() {
    echo "Fungsi dipanggil!";
    return true;
}

// AND short-circuit:
$hasil = false && ambilDataDariDatabase();
// Output: (kosong) — fungsi TIDAK dipanggil!
// Karena false && apapun = false, PHP tidak perlu evaluasi sisanya

// OR short-circuit:
$hasil = true || ambilDataDariDatabase();
// Output: (kosong) — fungsi TIDAK dipanggil!
// Karena true || apapun = true

// Ini berguna untuk:
$user = getUser() ?? redirect('/login');
// Jika getUser() return null → redirect dijalankan
// Jika getUser() return user → redirect TIDAK dijalankan
?>
```

**Di Laravel, short-circuit digunakan di banyak tempat:**  

```php
<?php
// Contoh di codebase:
$products = $request->has('category')
    ? Product::where('category_id', $request->category)
    : Product::all();

// Atau dengan coalescing operator (??):
$nama = $request->input('nama') ?? 'Guest';
// Jika input('nama') null → pakai 'Guest'
?>
```

### 3.4 Truthy dan Falsy — Nilai yang "Dianggap" Boolean  

Dalam PHP (dan banyak bahasa), nilai non-boolean bisa **dianggap** true atau false dalam konteks boolean (seperti di `if`).  

**Nilai yang dianggap FALSE (falsy):**  

```php
<?php
false;          // boolean false
0;              // integer nol
0.0;            // float nol
"";             // string kosong
"0";            // string "0"
[];             // array kosong
null;           // null
// (semua nilai lain dianggap TRUE)
?>
```

**Nilai yang dianggap TRUE (truthy):**  

```php
<?php
true;           // boolean true
1;              // integer bukan nol
-1;             // negatif juga truthy!
"string";       // string tidak kosong
"false";        // string "false" adalah truthy!
[1, 2, 3];      // array tidak kosong
new stdClass(); // object
?>
```

**Ini penting untuk kode yang bersih:**  

```php
<?php
$nama = "";

// ❌ BURUK — redundan
if (!empty($nama)) { ... }

// ✅ BAIK — $nama falsy, jadi langsung false
if ($nama) { ... }

// Tapi hati-hati dengan "0":
$status = "0";  // dianggap falsy!
if ($status) { ... } // false — padahal mungkin maksudnya "status 0 = pending"
// Gunakan === untuk kasus seperti ini
?>
```

---

## 4. Type Juggling — Konversi Otomatis PHP  

PHP secara otomatis mengubah tipe data saat diperlukan. Ini fitur sekaligus sumber bug.  

```php
<?php
// String → Integer (untuk operasi matematika)
echo "10" + 5;           // 15 (string "10" diubah ke integer)
echo "10 apel" + 5;      // 15 (bagian " apel" diabaikan — PHP ambil angka di awal)
echo "apel 10" + 5;      // 5 ("apel 10" → 0, karena tidak diawali angka)

// Integer → String (untuk concatenation)
echo "Harga: " . 50000;  // "Harga: 50000"

// Boolean → Integer
echo true + true;         // 2 (true = 1, 1+1=2)
echo false + 5;           // 5 (false = 0, 0+5=5)

// String → Boolean
if ("error") { ... }      // true! string "error" truthy
if ("") { ... }           // false — string kosong falsy
?>
```

**⚠️ Kasus berbahaya — in_array dengan loose comparison:**  

```php
<?php
$role = "admin";

// ❌ BERBAHAYA
if (in_array($role, ['admin', 'super-admin'])) {
    // Benar untuk "admin"
}
if (in_array($role, ['admin', 0])) {
    // TRUE!! Karena "admin" == 0 adalah true (type juggling)
    // PHP: "admin" dikonversi ke integer → 0, lalu 0 == 0 → true
}

// ✅ AMAN — strict mode
if (in_array($role, ['admin', 0], true)) {
    // FALSE! Sekarang juga cek tipe
}
?>
```

**Golden rule:**  
- Operasi matematika → PHP konversi ke angka  
- Operasi string (concatenation `.`) → PHP konversi ke string  
- Operasi logika (`if`, `&&`, `||`) → PHP konversi ke boolean  
- Perbandingan `==` → PHP konversi tipe (HATI-HATI!)  
- Perbandingan `===` → PHP **tidak** konversi tipe (AMAN)  

---

## 5. Operator Aritmetika  

```php
<?php
$a = 10;
$b = 3;

$tambah = $a + $b;       // 13
$kurang = $a - $b;       // 7
$kali = $a * $b;         // 30
$bagi = $a / $b;         // 3.3333... (float!)
$modulus = $a % $b;      // 1 (sisa bagi)
$pangkat = $a ** $b;     // 1000 (10³)

// Operator increment/decrement:
$counter = 0;
$counter++;    // $counter = $counter + 1  → 1
$counter--;    // $counter = $counter - 1  → 0

// Pre vs Post increment:
$x = 5;
echo ++$x;     // 6 (pre: tambah dulu, lalu echo)
echo $x++;     // 6 (post: echo dulu, lalu tambah → x jadi 7)
echo $x;       // 7
?>
```

### Operator Assignment Gabungan  

```php
<?php
$harga = 100000;

$harga += 5000;     // $harga = $harga + 5000  → 105000
$harga -= 10000;    // $harga = $harga - 10000 → 95000
$harga *= 2;        // $harga = $harga * 2     → 190000
$harga /= 5;        // $harga = $harga / 5     → 38000
$harga %= 3;        // $harga = $harga % 3     → 2
?>
```

---

## 6. Konstanta — Variabel yang Tidak Berubah  

**Konstanta** seperti variabel, tapi nilainya **tidak bisa diubah** setelah didefinisikan.  

```php
<?php
// Const — saat compile time
const APP_NAME = "Koneksi Store";
const TAX_RATE = 0.11;        // 11% PPN

// define() — saat runtime
define("APP_VERSION", "1.0.0");
define("MAX_LOGIN_ATTEMPTS", 5);

// Penggunaan:
echo APP_NAME;       // tanpa $ — "Koneksi Store"
echo TAX_RATE;       // 0.11

// ❌ Tidak bisa diubah
// APP_NAME = "Toko Baru"; // Error! konstanta tidak bisa re-assign
?>
```

**Di Laravel, konstanta biasanya di file config:**  

```php
<?php
// config/app.php
return [
    'name' => env('APP_NAME', 'Laravel'),
    'tax_rate' => 0.11,
];

// Dipanggil dengan:
config('app.name');      // "Koneksi Store"
config('app.tax_rate');  // 0.11
?>
```

---

## 7. Type Casting — Konversi Manual  

Kadang kamu ingin **memaksa** suatu nilai ke tipe tertentu.  

```php
<?php
$angkaString = "123";
$angka = (int) $angkaString;        // 123 (integer)
$desimal = (float) "45.67";         // 45.67 (float)
$bool = (bool) "true";              // true
$bool2 = (bool) 1;                  // true
$string = (string) 50000;           // "50000"
$array = (array) "tunggal";         // ["tunggal"]

// Fungsi konversi spesifik:
intval("123");       // 123
floatval("45.67");   // 45.67
strval(50000);       // "50000"
boolval(1);          // true
?>
```

---

## 8. Contoh Konkret dari Codebase Ini  

Mari kita lihat bagaimana variabel dan tipe data digunakan di `olshop-koneksi`:  

```php
<?php
// Dari app/Models/Product.php

class Product extends Model
{
    // Property — variabel milik class
    protected $fillable = [
        'name',           // string — nama produk
        'price',          // integer — harga dalam rupiah (bukan float!)
        'stock',          // integer — jumlah stok
        'is_active',      // boolean — produk aktif/tidak
        'description',    // string — deskripsi panjang
        'weight',         // integer — berat dalam gram
        'sku',            // string — kode produk unik
        'category_id',    // integer — ID kategori (foreign key)
        'brand_id',       // integer — ID brand (bisa null)
    ];

    // Contoh method dengan logika boolean dan perbandingan
    public function isLowStock(): bool
    {
        // Membandingkan stock dengan threshold
        // return boolean: true jika stok <= 5, false jika > 5
        return $this->stock <= 5;
    }

    public function isAvailable(): bool
    {
        // AND — dua kondisi harus terpenuhi
        return $this->is_active && $this->stock > 0;
    }
}
?>
```

```php
<?php
// Dari app/Http/Controllers/CheckoutController.php

public function process(CheckoutRequest $request)
{
    // $request berisi data dari form — array asosiatif
    $validated = $request->validated();

    // String — nama pembeli
    $customerName = $validated['name'];

    // Array — items di keranjang
    $cartItems = CartService::getItems();

    // Integer — total belanja dalam Rupiah
    $totalAmount = 0;

    foreach ($cartItems as $item) {
        // Aritmetika — hitung total
        $totalAmount += $item['price'] * $item['quantity'];
    }

    // String interpolation
    Log::info("Checkout processed for {$customerName}, total: {$totalAmount}");

    // Boolean — cek apakah gratis ongkir
    $isFreeShipping = $totalAmount >= 100000;  // boolean hasil perbandingan

    // Null — bisa null jika belum ada diskon
    $couponDiscount = $request->input('coupon_code')
        ? CouponService::calculate($request->input('coupon_code'), $totalAmount)
        : null;
}
?>
```

---

## 9. Ringkasan — Mind Map  

```
                    DATA DI KOMPUTER
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                  ▼
   VARIABEL           TIPE DATA          OPERATOR
  (tempat data)    (jenis data)         (pemroses)
        │                 │                  │
        │           ┌─────┼─────┐     ┌──────┴──────┐
        │           ▼     ▼     ▼     ▼             ▼
    Label alamat  Skalar  Compound  Special  Aritmetika  Logika
    di RAM        ─────── ───────  ───────  (+ - * /)  (&& || !)
                  int     array    null
                  float   object   resource  Perbandingan
                  string           callable  (== === > <)
                  bool

    TYPE JUGGLING — PHP konversi otomatis (HATI-HATI!)
    TRUTHY / FALSY — Setiap nilai punya "kebenaran" boolean
```

### Yang Harus Kamu Ingat  

1. **Variabel** = label untuk alamat di RAM — isinya bisa berubah  
2. **Tipe data** = cara komputer menginterpretasikan binary di memori  
3. **Integer** = bilangan bulat — gunakan untuk uang (dalam sen)  
4. **Float** = desimal — **jangan untuk uang** (error presisi)  
5. **String** = teks — perhatikan UTF-8 vs ASCII  
6. **Boolean** = true/false — fondasi semua logika  
7. **Array** = kumpulan data — paling sering dipakai  
8. **Null** = tidak ada nilai — beda dengan 0 atau ""  
9. **`===`** selalu lebih aman dari `==` (strict comparison)  
10. **Truthy/falsy** — pahami nilai apa yang dianggap true/false  
11. **Type juggling** — fitur PHP yang berguna sekaligus berbahaya  

---

## 📌 PRAKTIK — Kerjakan Ini  

```
✍️  Buka terminal, jalankan PHP interaktif:
    php -a

    Ketik satu per satu dan lihat hasilnya:
    php> $a = 10;
    php> $b = 3;
    php> echo $a + $b;
    php> echo $a / $b;
    php> var_dump($a === $b);
    php> var_dump($a == "10");
    php> var_dump($a === "10");

    Keluar: Ctrl+C atau exit

✍️  Cek truthy/falsy:
    php -r "var_dump((bool)'false');"       // true? false?
    php -r "var_dump((bool)'0');"           // true? false?
    php -r "var_dump((bool)[]);"            // true? false?
    php -r "var_dump((bool)[0]);"           // true? false?
    Tebak dulu, lalu jalankan!

✍️  Eksplorasi codebase:
    Buka app/Models/Product.php
    Cari property $fillable — lihat semua tipe data yang digunakan
    Cari method yang mengembalikan boolean (return type :bool)

    Buka app/Http/Controllers/CheckoutController.php
    Cari variabel, lihat bagaimana string, integer, boolean digunakan
```

---

## 🔗 Referensi & Lanjutan  

| Topik | Di Journey Ini |
|-------|----------------|
| Algoritma — logika if/else, loop | FASE-01: 01-04-algoritma-dasar |
| OOP — class dengan property & method | FASE-03: semua dokumen |
| Collection — array super di Laravel | FASE-06: 06-05-model-dan-eloquent-orm |
| Database — tipe data kolom | FASE-07: semua dokumen |

---

*"Data adalah bahan mentah, variabel adalah wadahnya,  
tipe data adalah aturan bagaimana wadah itu digunakan,  
dan logika adalah cara kita memproses semuanya."*

*Kamu baru saja memahami 50% dari semua programming —  
sisanya hanyalah variasi dan aplikasi dari konsep ini.*
