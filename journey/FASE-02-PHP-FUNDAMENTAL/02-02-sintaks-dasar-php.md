# 02-02: Sintaks Dasar PHP

> **Fase**: 2 — PHP Fundamental  
> **Prasyarat**: 02-01-pengantar-php  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: sintaks PHP, variabel, konstanta, operator, type declaration, strict types, heredoc, nowdoc, nullsafe, match, enum, named argument, variadic  

---

## 📋 Ringkasan  

Dokumen sebelumnya memperkenalkan PHP secara konseptual.  
Sekarang kita akan **menulis kode PHP sesungguhnya** — sintaks, aturan,  
dan semua konstruksi bahasa yang akan kamu pakai setiap hari.  

**Target pemahaman:**  
- Kamu bisa menulis script PHP dari nol tanpa melihat referensi  
- Kamu paham perbedaan `==` vs `===`, `""` vs `''`, `include` vs `require`  
- Kamu tahu fitur-fitur PHP 8+ yang modern  
- Kamu bisa membaca dan memahami kode PHP di codebase ini  

---

## 1. Struktur File PHP  

### 1.1 Aturan Dasar  

```php
<?php
// 1. File PHP WAJIB diawali <?php (tidak boleh ada spasi/karakter sebelum ini)
// 2. Tag penutup ?> di akhir file adalah OPTIONAL
// 3. Jika file PURE PHP (hanya PHP, tanpa HTML), jangan pakai ?> di akhir
// 4. Setiap statement diakhiri titik koma (;)
// 5. PHP case-sensitive untuk variabel ($nama ≠ $Nama)

echo "Ini file PHP murni";
echo "Tidak perlu ?> di akhir";

// ❌ JANGAN LAKUKAN INI:
// File pure PHP dengan ?> di akhir — riskan: setelah ?> bisa terlanjur ada spasi
// baru output, menyebabkan "Headers already sent" error!
?>
```

```php
<?php
// File dengan HTML — WAJIB pakai ?> untuk tutup PHP, lanjut HTML
?>
<!DOCTYPE html>
<html>
<head><title>Halaman</title></head>
<body>
    <h1>Ini HTML murni</h1>
    <?php
    // Buka PHP lagi di tengah HTML
    echo "<p>Ini dari PHP</p>";
    ?>
</body>
</html>
<?php
// Bisa buka PHP lagi setelah HTML
// Tapi untuk file mixed: tutup ?> di akhir tidak masalah
?>
```

**Best practice — untuk file yang pure PHP (seperti class, config):**  

```php
<?php
// ❌ BURUK — tutup tag di akhir file pure PHP
namespace App\Services;
class CartService {
    // ...
}
?>

<?php
// ✅ BAIK — tanpa ?> di akhir
namespace App\Services;
class CartService {
    // ...
}
// EOF — tidak perlu ?>
```

Kenapa? Karena jika ada SATU spasi atau baris baru SETELAH `?>`,  
spasi itu akan dikirim sebagai output. Di Laravel, ini bisa menyebabkan  
"headers already sent" error — sangat sulit di-debug.  

### 1.2 Separator Instruksi — Titik Koma  

```php
<?php
echo "Hello";        // ✅ titik koma mengakhiri statement
echo "World";        // ✅

echo "Hello"         // ❌ ERROR: syntax error, unexpected end of file
                     //     (lupa titik koma)

// Satu-satunya pengecualian: sebelum tag penutup PHP
echo "Hello"         // ✅ valid — karena ?> bertindak seperti titik koma
?>
```

### 1.3 Nama Variabel — Aturan Detail  

```
$variabel
├── WAJIB diawali $
├── Setelah $: huruf (a-z, A-Z) atau underscore (_)
├── Setelah karakter pertama: huruf, angka, underscore
├── Case-sensitive: $nama ≠ $Nama ≠ $NAMA
└── Tidak boleh: karakter spesial ($nama-depan, $nama! )
```

```php
<?php
// ✅ VALID
$nama = "Fathan";
$_nama = "Fathan";          // underscore di awal — biasa untuk "internal"
$nama_lengkap = "Fathan M"; // snake_case
$namaLengkap = "Fathan M";  // camelCase
$nama1 = "Fathan";          // angka di tengah/akhir
$PANJANG_SEKALI_DENGAN_UNDERSCORE = "";

// ❌ INVALID
// $1nama = "Fathan";       // angka di awal → ERROR syntax
// $nama-lengkap = "Fathan"; // strip (-) → dianggap operator minus!
// $nama.lengkap = "Fathan"; // dot (.) → dianggap concatenation!
// $nama lengkap = "Fathan"; // spasi → ERROR
// $ = "Fathan";             // tidak ada nama → ERROR
?>
```

### 1.4 Variable Variables — Variabel Dinamis  

PHP punya fitur unik: nama variabel bisa berasal dari variabel lain.  

```php
<?php
$nama = "Fathan";
$$nama = "Pelajar";    // Buat variabel $Fathan dengan nilai "Pelajar"

echo $Fathan;          // "Pelajar"
echo ${$nama};         // "Pelajar" — sintaks eksplisit

// Praktis? Jarang. Tapi ada di codebase tertentu:
// $key = "price";
// $product->{$key} = 50000;  // Sama dengan $product->price = 50000
?>
```

### 1.5 By Reference (&) — Bukan Copy, Tapi Alamat  

Normalnya, saat kamu assign variabel ke variabel lain, nilai di- **copy**.  

```php
<?php
// PASS BY VALUE (copy) — default
$a = 10;
$b = $a;        // $b adalah COPY dari $a
$b = 20;        // $a tetap 10
echo $a;        // 10 — tidak berubah

// PASS BY REFERENCE (&) — alias/link
$x = 10;
$y = &$x;       // $y adalah REFERENCE ke $x (bukan copy)
$y = 20;        // $x IKUT berubah!
echo $x;        // 20 — berubah!

// Analogi:
// $b = $a  → fotokopi buku. $b diubah, $a tidak berubah
// $y = &$x → dua orang lihat buku yang SAMA. Satu corat-coret, yang lain lihat coretan itu
?>
```

**Di codebase — reference jarang dipakai, tapi penting untuk dipahami**  
karena beberapa fungsi PHP menggunakan reference:  

```php
<?php
// sort() mengubah array ASLI (pass by reference)
$buah = [3, 1, 2];
sort($buah);           // sort() menerima &$array
print_r($buah);        // [1, 2, 3] — array asli BERUBAH

// foreach dengan reference — HATI-HATI!
$angka = [1, 2, 3];
foreach ($angka as &$value) {
    $value *= 2;       // Ubah nilai asli via reference
}
print_r($angka);       // [2, 4, 6]

// ⚠️ Bahaya reference di foreach:
// $value masih mereferensi ke elemen terakhir array!
// Jika dipakai lagi tanpa unset(), bisa overwrite data!
unset($value);         // ✅ SELALU unset() setelah foreach with reference
?>
```

---

## 2. Konstanta — Nilai yang Tidak Berubah  

### 2.1 define() vs const  

```php
<?php
// Cara 1: define() — runtime constant
define("APP_NAME", "Koneksi Store");
define("PI", 3.14159);
define("DB_CONFIG", [
    'host' => 'localhost',
    'port' => 3306,
]);

// Cara 2: const keyword — compile-time constant (harus di top-level class)
const VERSION = "1.0.0";
const MAX_ITEMS = 100;
const STATUSES = ['pending', 'paid', 'shipped']; // Array constant sejak PHP 5.6

// Perbedaan:
// define()  — bisa di dalam blok if/else, loop, function
// const     — harus di TOP LEVEL class/file
?>
```

### 2.2 Magic Constants — Nilai Bawaan PHP  

PHP punya konstanta "ajaib" yang nilainya berubah tergantung konteks:  

```php
<?php
echo __LINE__;           // Baris kode saat ini — 5
echo __FILE__;           // Path lengkap file saat ini
echo __DIR__;            // Directory dari file saat ini
echo __FUNCTION__;       // Nama function saat ini
echo __CLASS__;          // Nama class saat ini
echo __METHOD__;         // Nama method (Class::method)
echo __NAMESPACE__;      // Nama namespace saat ini
echo __TRAIT__;          // Nama trait saat ini
?>
```

**Sangat berguna untuk logging dan debugging:**  

```php
<?php
// Di codebase — sering dipakai untuk log:
Log::error("Error di " . __FILE__ . " baris " . __LINE__);

// Atau di Laravel:
logger()->error("Gagal memproses order", [
    'file' => __FILE__,
    'line' => __LINE__,
    'order_id' => $orderId,
]);
?>
```

### 2.3 Predefined Constants — Konstanta Bawaan PHP  

```php
<?php
// PHP version
echo PHP_VERSION;         // "8.3.30"
echo PHP_MAJOR_VERSION;   // 8
echo PHP_MINOR_VERSION;   // 3

// OS
echo PHP_OS;              // "WINNT" (Windows)

// Integer bounds
echo PHP_INT_MAX;         // 9223372036854775807
echo PHP_INT_MIN;         // -9223372036854775808
echo PHP_INT_SIZE;        // 8 (byte — 64-bit)

// Directory separator
echo DIRECTORY_SEPARATOR;  // "\" (Windows) atau "/" (Linux/Mac)
echo PHP_EOL;              // "\r\n" (Windows) atau "\n" (Linux/Mac)

// Nilai khusus
echo INF;                  // Infinity (float)
echo NAN;                  // Not-a-Number (float)
?>
```

---

## 3. Operator — Manipulasi Data  

### 3.1 Operator Aritmetika  

```php
<?php
$a = 10;
$b = 3;

echo $a + $b;   // 13 — penjumlahan
echo $a - $b;   // 7  — pengurangan
echo $a * $b;   // 30 — perkalian
echo $a / $b;   // 3.3333333333333 — pembagian (float!)
echo $a % $b;   // 1  — modulus (sisa bagi)

// Operator pangkat (PHP 5.6+)
echo 2 ** 10;   // 1024 — 2 pangkat 10

// Pembagian integer (PHP 8+)
echo intdiv(10, 3);  // 3 — integer division (hasil dibulatkan ke bawah)
?>
```

### 3.2 Operator Assignment  

```php
<?php
$x = 10;          // assignment dasar

$x += 5;          // $x = $x + 5   → 15
$x -= 3;          // $x = $x - 3   → 12
$x *= 2;          // $x = $x * 2   → 24
$x /= 4;          // $x = $x / 4   → 6
$x %= 3;          // $x = $x % 3   → 0
$x **= 2;         // $x = $x ** 2  → 0 (0^2=0)

// String assignment
$text = "Hello";
$text .= " World";  // $text = $text . " World" → "Hello World"

// Null coalescing assignment (PHP 7.4+)
$data['key'] ??= 'default';
// Sama dengan: $data['key'] = $data['key'] ?? 'default';
?>
```

### 3.3 Operator Perbandingan  

```php
<?php
$a = 5;
$b = "5";
$c = 10;

// LOOSE equality — konversi tipe otomatis
var_dump($a == $b);     // true   — 5 == "5" → true (string diubah ke int)
var_dump($a != $c);     // true   — 5 != 10

// STRICT equality — cek tipe juga
var_dump($a === $b);    // false  — integer !== string
var_dump($a !== $c);    // true   — 5 !== 10

// Lebih besar / lebih kecil
var_dump($a > $c);      // false
var_dump($a < $c);      // true
var_dump($a >= 5);      // true
var_dump($a <= 5);      // true

// Spaceship operator (PHP 7+) — untuk sorting
echo 5 <=> 10;          // -1  (kiri < kanan)
echo 10 <=> 10;         // 0   (sama)
echo 15 <=> 10;         // 1   (kiri > kanan)
?>
```

**⚠️ Jebakan umum — perbandingan dengan tipe berbeda:**  

```php
<?php
// PHP type juggling — tabel perilaku:
// (sumber: php.net)

var_dump(0 == "a");         // false — 0 == 0 → true. TAPI:
var_dump("0" == "a");       // false — string vs string, "0" ≠ "a"
var_dump(0 == "0");         // TRUE  — 0 == 0
var_dump(0 == "");          // TRUE  — 0 == 0 (string kosong → 0)
var_dump(0 == null);        // TRUE  — 0 == 0 (null → 0)
var_dump(false == "false"); // false — "false" (string) → 0, false → 0, tapi...
var_dump(false == 0);       // TRUE  — false == 0

// ⚠️ INI BERBAHAYA:
$role = "admin";
if (in_array($role, ['admin', 0])) {
    echo "AKSES DIBERIKAN!"; // Ini akan EKSEKUSI! Karena "admin" == 0 → true
}
// ✅ Solusi: strict flag
if (in_array($role, ['admin', 0], true)) {
    // Tidak masuk — karena "admin" !== 0
}
?>
```

### 3.4 Operator Logika  

```php
<?php
$a = true;
$b = false;

// AND
var_dump($a && $b);     // false — dua-duanya harus true
var_dump($a and $b);    // false — sama, tapi precedence berbeda

// OR
var_dump($a || $b);     // true  — salah satu true cukup
var_dump($a or $b);     // true

// NOT
var_dump(!$a);          // false — membalikkan

// XOR (exclusive or)
var_dump($a xor $b);    // true — salah satu true, tapi tidak keduanya
var_dump(true xor true); // false

// Precedence: && > || > and > or
$x = true || false && false;
// Sebenarnya: true || (false && false) → true || false → true

$y = true or false && false;
// Sebenarnya: (true) or (false && false) — karena 'or' precedence rendah
//               ↑ $y = true (assignment dulu, baru or)

// ☝️ INI ALASANNYA: SELALU pakai && dan ||, JANGAN and/or
?>
```

### 3.5 Operator String  

```php
<?php
// Concatenation (.)
$firstName = "Fathan";
$lastName = "Mubina";
$fullName = $firstName . " " . $lastName;
echo $fullName;   // "Fathan Mubina"

// Concatenation assignment
$message = "Halo";
$message .= " ";
$message .= $firstName;
echo $message;    // "Halo Fathan"
?>
```

### 3.6 Operator Ternary & Null Coalescing  

```php
<?php
$umur = 20;

// Ternary: (kondisi) ? nilai_true : nilai_false
$status = ($umur >= 17) ? "Dewasa" : "Anak-anak";
echo $status; // "Dewasa"

// Ternary bertingkat — HATI-HATI, sulit dibaca!
$grade = ($nilai >= 90) ? "A" : (($nilai >= 80) ? "B" : "C");
// ❌ BURUK — ganti dengan if/else atau match()

// Null coalescing (??) — cek isset()
$nama = $_GET['nama'] ?? 'Guest';
// Sama dengan: $nama = isset($_GET['nama']) ? $_GET['nama'] : 'Guest';

// Null coalescing chain
$city = $user?->address?->city ?? 'Unknown';
// Jika $user null → null ?? 'Unknown' → 'Unknown'

// Elvis operator (?:) — cek truthy
$display = $input ?: 'default';
// Sama dengan: $display = $input ? $input : 'default';
// ⚠️ Beda dengan ?? — ?: cek truthy, ?? cek null/isset

$input = 0;
echo $input ?: 'default';  // 'default' — 0 falsy
echo $input ?? 'default';  // 0 — 0 isset (tidak null)
?>
```

### 3.7 Operator Array  

```php
<?php
$a = ['a', 'b', 'c'];
$b = ['d', 'e'];

// Union (+)
$c = $a + $b;
print_r($c); // ['a', 'b', 'c', 'd', 'e']
// ⚠️ Untuk key string: jika ada key yang sama, yang kiri menang

// Equality (==) — sama jika key/value sama
$x = ['a' => 1, 'b' => 2];
$y = ['b' => 2, 'a' => 1];
var_dump($x == $y);   // true — urutan tidak penting
var_dump($x === $y);  // false — urutan berbeda!

// Identity (===) — urutan dan tipe harus sama
$z = ['a' => 1, 'b' => 2];
var_dump($x === $z);  // true — urutan dan nilai sama

// Ship operator untuk sorting array asosiatif:
$users = [
    ['name' => 'Fathan', 'age' => 20],
    ['name' => 'Umar', 'age' => 25],
    ['name' => 'Ali', 'age' => 18],
];
usort($users, fn($a, $b) => $a['age'] <=> $b['age']);
// Urut: Ali(18), Fathan(20), Umar(25)
?>
```

### 3.8 Operator Bitwise (Jarang dipakai, tapi ada)  

```php
<?php
// Operasi binary pada level bit
$a = 5;   // 0101
$b = 3;   // 0011

echo $a & $b;   // 1  — AND:    0101 & 0011 = 0001 (1)
echo $a | $b;   // 7  — OR:     0101 | 0011 = 0111 (7)
echo $a ^ $b;   // 6  — XOR:    0101 ^ 0011 = 0110 (6)
echo ~$a;       // -6 — NOT:    0101 → ...11111010 (-6)
echo $a << 1;   // 10 — shift left:  0101 → 1010 (10)
echo $a >> 1;   // 2  — shift right: 0101 → 0010 (2)

// Di codebase — biasanya untuk permission flags:
$permissionRead = 1;    // 0001
$permissionWrite = 2;   // 0010
$permissionExecute = 4; // 0100

$userPerm = $permissionRead | $permissionWrite;  // 0011 (3)
if ($userPerm & $permissionRead) {   // cek 0011 & 0001 = 0001 (true)
    echo "Boleh baca";
}
?>
```

### 3.9 Operator Error Control (@)  

```php
<?php
// @ — suppress error (HINDARI — ini menyembunyikan masalah!)
$data = @file_get_contents('file_tidak_ada.txt');
// Tanpa @: Warning: file_get_contents(...): failed to open stream
// Dengan @: Warning di-supress — $data = false

// ❌ BURUK — jangan sembunyikan error
// ✅ BAIK — handle error dengan benar
if (file_exists('file_tidak_ada.txt')) {
    $data = file_get_contents('file_tidak_ada.txt');
} else {
    $data = 'default';
}
?>
```

### 3.10 Operator Execution (Backtick \` \`)  

```php
<?php
// `command` — eksekusi shell command
$output = `ls -la`;
echo "<pre>$output</pre>";

// Di Windows:
$dir = `dir`;
echo "<pre>$dir</pre>";

// ⚠️ Hati-hati: user input jangan langsung ke backtick!
// ❌ JANGAN: `ping {$userInput}` — bisa command injection!
?>
```

---

## 4. Type Declarations — PHP Modern  

Sejak PHP 7, kamu bisa (dan harus) mendeklarasikan tipe parameter dan return value.  

### 4.1 Scalar Type Declarations  

```php
<?php
// Tanpa type declaration — bisa diisi apa saja
function sapa($nama) {
    return "Halo, $nama!";
}
echo sapa(123);        // "Halo, 123!" — mungkin tidak diinginkan

// Dengan type declaration — PHP akan memvalidasi
function sapa(string $nama): string {
    return "Halo, $nama!";
}
// echo sapa(123);     // TypeError! — 123 bukan string

// Tipe yang bisa dideklarasikan:
function contoh(
    int $a,            // integer
    float $b,           // float (desimal)
    string $c,          // string (teks)
    bool $d,            // boolean (true/false)
    array $e,           // array
    callable $f,        // function/callback
    iterable $g,        // array atau Traversable
    object $h,          // object
    mixed $i,           // any type (PHP 8+)
    ?string $j,         // nullable (string atau null)
): string {             // return type
    return "OK";
}
?>
```

### 4.2 Strict Types — Mode Ketat  

Secara default, PHP akan **mencoba mengkonversi** tipe jika tidak cocok:  

```php
<?php
function kali(int $a, int $b): int {
    return $a * $b;
}

echo kali(5, "3");     // 15 — PHP konversi "3" ke 3
echo kali(5, "3.apel"); // 15 — "3.apel" → 3 (PHP ambil angka di awal)
echo kali(5, "apel");   // TypeError! — "apel" tidak bisa dikonversi ke int
?>
```

Dengan **strict_types**, PHP menolak konversi otomatis:  

```php
<?php
declare(strict_types=1);  // ← WAJIB di baris PERTAMA file (setelah <?php)

function kali(int $a, int $b): int {
    return $a * $b;
}

echo kali(5, 3);         // 15  — OK
echo kali(5, "3");       // TypeError! — string tidak diterima untuk int
echo kali(5, 3.0);       // TypeError! — float tidak diterima untuk int
?>
```

**Best practice di Laravel — codebase ini:**  

```php
// app/Services/CartService.php (konsep)
declare(strict_types=1);

namespace App\Services;

class CartService
{
    public function addItem(int $productId, int $quantity): void
    {
        // ...
    }

    public function getTotal(): int
    {
        // ...
    }

    public function isCartEmpty(): bool
    {
        // ...
    }
}
```

### 4.3 Union Types — Lebih dari Satu Tipe (PHP 8+)  

```php
<?php
declare(strict_types=1);

function formatPrice(int|float $price): string {
    return "Rp " . number_format($price, 0, ',', '.');
}

echo formatPrice(50000);      // "Rp 50.000"
echo formatPrice(50000.50);   // "Rp 50.001" — float, number_format bulatkan

function findUser(int|string $identifier): ?User {
    // Bisa cari berdasarkan ID (int) atau email (string)
    if (is_int($identifier)) {
        return User::find($identifier);
    }
    return User::where('email', $identifier)->first();
}
?>
```

### 4.4 Mixed Type — "Terserah" (PHP 8+)  

```php
<?php
// mixed = string|int|float|bool|array|object|null|callable|resource
function debug(mixed $data): void {
    var_dump($data);
}

// ⚠️ mixed adalah "jalan keluar" — usahakan selalu spesifik
// ❌ function process(mixed $input): mixed
// ✅ function process(string $input): array
?>
```

### 4.5 Void vs Never  

```php
<?php
// void — function tidak mengembalikan nilai apapun
function logActivity(string $action): void {
    Log::info("Action: {$action}");
    // Tidak ada return — atau bisa "return;"
}

// never — function TIDAK PERNAH selesai (selalu throw/exit)
function abort(string $message): never {
    throw new \Exception($message);
    // Kode setelah ini tidak akan pernah dieksekusi
}

function redirect(string $url): never {
    header("Location: $url");
    exit;
}
?>
```

---

## 5. Control Flow — Detail Lengkap  

### 5.1 if/else — Semua Variasi  

```php
<?php
// Bentuk 1: if saja
if ($kondisi) {
    // eksekusi jika true
}

// Bentuk 2: if-else
if ($kondisi) {
    // jika true
} else {
    // jika false
}

// Bentuk 3: if-elseif-else
if ($kondisi1) {
    //
} elseif ($kondisi2) {
    //
} else {
    //
}

// Bentuk 4: tanpa curly braces (hanya 1 statement)
if ($kondisi) echo "True";

// Bentuk 5: alternatif syntax (pakai colon — untuk template)
if ($kondisi):
    echo "True";
elseif ($kondisi2):
    echo "Kedua";
else:
    echo "False";
endif;
?>
```

**Bentuk alternatif (`:` dan `endif;`) — lazim di view/template:**  

```php
<?php // Di file template — seperti Blade nanti ?>
<?php if ($isLoggedIn): ?>
    <p>Selamat datang, <?= $name ?></p>
    <a href="/logout">Logout</a>
<?php elseif ($isGuest): ?>
    <p>Silakan login</p>
<?php else: ?>
    <p>Halo, pengunjung</p>
<?php endif; ?>
```

### 5.2 switch/match — Detail  

```php
<?php
// switch — klasik (hati-hati lupa break!)
switch ($role) {
    case 'admin':
        $permissions = ['all'];
        break;
    case 'merchant':
        $permissions = ['manage_products', 'view_orders'];
        break;
    case 'customer':
        $permissions = ['view_products', 'purchase'];
        break;
    default:
        $permissions = [];
}

// match (PHP 8+) — modern, return value, strict comparison (===)
// switch pakai == (loose), match pakai === (strict)
$permissions = match ($role) {
    'admin' => ['all'],
    'merchant' => ['manage_products', 'view_orders'],
    'customer' => ['view_products', 'purchase'],
    default => [],
};

// match bisa multiple condition:
$dayOfWeek = 3; // Rabu
$type = match (true) {
    $dayOfWeek <= 5 => 'weekday',
    $dayOfWeek >= 6 => 'weekend',
};
echo $type; // 'weekday'

// ⚠️ match harus exhaustive — semua kemungkinan harus tercakup
// Atau ada default — jika tidak, UnhandledMatchError
?>
```

### 5.3 Loop — Semua Variasi  

```php
<?php
// for — counter
for ($i = 0; $i < 5; $i++) {
    echo $i; // 01234
}

// for dengan multiple counter
for ($i = 0, $j = 10; $i < 5; $i++, $j--) {
    echo "($i, $j) "; // (0,10) (1,9) (2,8) (3,7) (4,6)
}

// foreach — array
$items = ['a', 'b', 'c'];
foreach ($items as $item) {
    echo $item;
}
foreach ($items as $key => $value) {
    echo "$key: $value";
}

// while
$i = 0;
while ($i < 5) {
    echo $i++;
}

// do-while — minimal sekali jalan
$i = 100;
do {
    echo $i; // 100 — dijalankan sekali
} while ($i < 5);
?>
```

---

## 6. Include & Require — Memecah File  

### 6.1 Empat Cara Memasukkan File  

```php
<?php
// include — coba include, jika gagal: warning (script lanjut)
include 'config.php';
include 'file_tidak_ada.php'; // Warning, script lanjut

// include_once — sama, tapi hanya sekali
include_once 'config.php'; // jika sudah di-include, skip

// require — WAJIB, jika gagal: fatal error (script berhenti)
require 'config.php';
require 'file_tidak_ada.php'; // Fatal Error! Script STOP

// require_once — sama, tapi hanya sekali
require_once 'vendor/autoload.php'; // Std di Laravel
?>
```

### 6.2 Perbedaan — Kapan Pakai Yang Mana  

| Situasi | Pakai |
|---------|-------|
| File konfigurasi WAJIB ADA | `require` |
| File library/class | `require_once` |
| File opsional (template, widget) | `include` |
| File opsional yang hanya sekali | `include_once` |
| Autoload Composer | `require_once __DIR__ . '/vendor/autoload.php'` |

### 6.3 Cara Kerja Include  

```php
<?php
// config.php
$dbHost = 'localhost';
$dbName = 'olshop';
echo "Config loaded!\n";
?>
```

```php
<?php
// index.php
echo "Start\n";
include 'config.php';  // → Semua kode di config.php dieksekusi DI SINI
                       // → Variabel $dbHost, $dbName jadi tersedia
                       // → Output: "Config loaded!"
echo $dbHost;          // "localhost"
echo "End\n";          // Output: "Start\nConfig loaded!\nlocalhost\nEnd\n"
?>
```

### 6.4 File Path — __DIR__  

```php
<?php
// ❌ BURUK — relative path tergantung mana file yang executing
require 'config.php';        // Cari di current working directory
require '../config.php';     // Naik satu folder

// ✅ BAIK — absolute path dari folder file ini
require __DIR__ . '/config.php';           // File di folder yang sama
require __DIR__ . '/../config.php';        // Naik satu folder
require __DIR__ . '/../../config.php';     // Naik dua folder

// Di Laravel, base_path() helper:
require base_path('vendor/autoload.php');
?>
```

---

## 7. Fitur PHP Modern (8+) — Wajib Tahu  

### 7.1 Named Arguments  

```php
<?php
// Tanpa named arguments — harus urut:
function createUser(string $name, string $email, bool $isAdmin, ?string $phone): User
{
    // ...
}

// ❌ Sulit dibaca — apa itu true? apa itu null?
createUser('Fathan', 'fathan@test.com', true, null);

// ✅ Named arguments — jelas maksudnya
createUser(
    name: 'Fathan',
    email: 'fathan@test.com',
    isAdmin: true,
    phone: null,
);

// Bisa skip parameter yang optional:
function config(string $host, int $port = 3306, string $dbname = 'default'): void {}
config(host: 'localhost', dbname: 'olshop'); // port skip — pakai default 3306

// ⚠️ Named args tidak bisa digabung dengan positional setelahnya:
// config('localhost', dbname: 'olshop'); // OK
// config(host: 'localhost', 3306);       // ERROR!
?>
```

### 7.2 Attributes (Annotations) — PHP 8+  

```php
<?php
// Attribute mirip annotation di Java — metadata untuk class/method/property
#[Route('/api/products', methods: ['GET'])]
#[Middleware('auth')]
class ProductController
{
    #[Validate('required|numeric')]
    private int $price;

    #[OnEvent('order.placed')]
    public function handleOrderPlaced(Order $order): void
    {
        // ...
    }
}

// Di Laravel (sebelum PHP 8, pakai PHPDoc):
// /**
//  * @param int $id
//  * @return Product
//  */
// Di Laravel modern, attribute makin banyak dipakai:
// #[AsCommand(name: 'app:process-orders')]
// class ProcessOrders extends Command { ... }
?>
```

### 7.3 Enums (Enumeration) — PHP 8.1+  

```php
<?php
// Sebelum enum — pakai konstanta class:
class OrderStatus {
    const PENDING = 'pending';
    const PAID = 'paid';
    const SHIPPED = 'shipped';
    const DELIVERED = 'delivered';
}
$status = OrderStatus::PAID;

// ✅ Dengan enum — type-safe, method, case sensitif:
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';
    case Shipped = 'shipped';
    case Delivered = 'delivered';

    public function label(): string
    {
        return match ($this) {
            self::Pending => 'Menunggu Pembayaran',
            self::Paid => 'Sudah Dibayar',
            self::Shipped => 'Dikirim',
            self::Delivered => 'Diterima',
        };
    }

    public function canCancel(): bool
    {
        return $this === self::Pending;
    }
}

// Penggunaan:
$status = OrderStatus::Paid;
echo $status->value;       // "paid"
echo $status->label();     // "Sudah Dibayar"
echo $status->canCancel();  // false

// Di codebase — cek app/Enums/ (jika ada):
function updateStatus(OrderStatus $status): void {
    // Hanya menerima OrderStatus yang valid
}
?>
```

### 7.4 First-class Callable — PHP 8.1+  

```php
<?php
// Sebelum: pakai string/Closure
$callback = 'strtoupper';
$callback = function($s) { return strtoupper($s); };

// PHP 8.1+: First-class callable syntax
$callback = strtoupper(...);  // Callable object

$names = ['fathan', 'umar', 'ali'];
$upper = array_map($callback, $names);
// ['FATHAN', 'UMAR', 'ALI']

// Di Laravel Collections:
$products->map(fn($p) => $p->price);
// atau
$products->map(fn(Product $p) => $p->price);
?>
```

### 7.5 Readonly Properties — PHP 8.1+  

```php
<?php
class UserDTO {
    // Property hanya bisa di-set sekali (di constructor)
    public readonly string $name;
    public readonly int $age;

    public function __construct(string $name, int $age) {
        $this->name = $name;  // OK — set di constructor
        $this->age = $age;
    }

    public function setName(string $name): void {
        // $this->name = $name;  // ERROR! — readonly, sudah di-set
    }
}

// PHP 8.2: readonly class — semua property jadi readonly
readonly class ConfigDTO {
    public function __construct(
        public string $host,
        public int $port,
        public string $dbname,
    ) {}
}
// Data Transfer Object — sekali buat, tidak bisa diubah
?>
```

### 7.6 Array Unpacking — Spread Operator  

```php
<?php
$parts1 = ['a', 'b', 'c'];
$parts2 = ['d', 'e'];
$combined = [...$parts1, ...$parts2];
// ['a', 'b', 'c', 'd', 'e']

// Untuk array asosiatif (PHP 8.1+):
$config1 = ['host' => 'localhost', 'port' => 3306];
$config2 = ['dbname' => 'olshop', 'charset' => 'utf8'];
$config = [...$config1, ...$config2];

// ⚠️ Jika ada key sama, yang belakang menimpa:
$default = ['host' => 'localhost', 'port' => 3306];
$custom = ['port' => 5432]; // override port
$config = [...$default, ...$custom]; // port: 5432
?>
```

### 7.7 Fiber — Concurrency (PHP 8.1+)  

```php
<?php
// Fiber adalah low-level concurrency — mirip coroutine
// Laravel 11+ menggunakan Fiber untuk concurrent tasks
$fiber = new Fiber(function (): void {
    $value = Fiber::suspend('first');
    echo "Resumed with: $value";
});

$value = $fiber->start();
echo $value; // "first"
$fiber->resume('second'); // "Resumed with: second"
?>
```

---

## 8. Strict Types & Type System — Praktik Terbaik  

### 8.1 Deklarasi di Codebase Ini  

```php
// app/Models/Product.php
declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Product extends Model
{
    protected $fillable = [
        'name', 'slug', 'sku', 'price', 'stock',
        'weight', 'description', 'is_active',
        'category_id', 'brand_id',
    ];

    protected $casts = [
        'price' => 'integer',
        'stock' => 'integer',
        'weight' => 'integer',
        'is_active' => 'boolean',
        'category_id' => 'integer',
        'brand_id' => 'integer',
    ];

    public function isLowStock(): bool
    {
        return $this->stock <= 5;
    }

    public function isAvailable(): bool
    {
        return $this->is_active && $this->stock > 0;
    }

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }
}
```

### 8.2 Type Casting di Model  

```php
<?php
// $casts memastikan tipe data yang benar saat diakses
$product = Product::find(1);

// Tanpa cast: $product->price adalah string "500000"
// Dengan cast (integer): $product->price adalah int(500000)

// Ini penting untuk perbandingan strict!
// $product->price === 500000  → true (dengan cast)
// $product->price === 500000  → false (tanpa cast — string !== int)
?>
```

---

## 9. Ringkasan — Sintaks PHP dalam Satu Halaman  

```
<?php                         // Tag buka
declare(strict_types=1);     // Mode ketat (baris pertama)

// VARIABEL & KONSTANTA
$nama = "Fathan";             // Variabel
const APP = "Koneksi";        // Konstanta compile-time
define("DB", "mysql");        // Konstanta runtime
echo __FILE__;                // Magic constant

// TIPE DATA
int $a = 10;                  // Integer
float $b = 3.14;              // Float
string $c = "teks";           // String
bool $d = true;               // Boolean
array $e = [1, 2];            // Array
?string $f = null;            // Nullable

// OPERATOR
+, -, *, /, %                 // Aritmetika
., .=                         // String
==, ===, !=, !==, <, >, <=>  // Perbandingan
&&, ||, !                     // Logika
??, ?:, ?->                   // Ternary & nullsafe
&                             // Reference

// CONTROL FLOW
if, else, elseif              // Conditional
switch, match                 // Multi-condition
for, foreach, while           // Loop
break, continue               // Loop control

// FILE
include, require, _once       // Memasukkan file
__DIR__                       // Path file saat ini

// MODERN (PHP 8+)
named arguments               // Argumen bernama
attributes                    // Metadata #[Attribute]
enum                          // Enumerasi type-safe
readonly                      // Immutable property
match                         // Switch modern
union types (int|string)      // Multi-type
mixed                         // Any type
never                         // No return (throw/exit)
first-class callable          // strtoupper(...)
spread (...$array)            // Array unpacking
```

### Yang Harus Kamu Ingat  

1. **`<?php`** di awal, **tanpa `?>`** di akhir (untuk file pure PHP)  
2. **`declare(strict_types=1)`** — biasakan di setiap file baru  
3. **Type declaration** — selalu tipe parameter dan return type  
4. **`===`** lebih aman dari `==` — strict comparison  
5. **`??`** untuk null default, **`?->`** untuk nullsafe  
6. **`match`** lebih baik dari `switch` (PHP 8+)  
7. **`readonly`** untuk data immutable  
8. **Named arguments** untuk fungsi dengan banyak parameter  
9. **`__DIR__`** untuk path absolut — jangan relative path  
10. **Jangan pakai `@`** — handle error dengan benar  

---

## 📌 PRAKTIK — Kerjakan Ini  

```
✍️  Buat file baru: latihan-sintaks.php di C:\laragon\www\:
    <?php
    declare(strict_types=1);
    
    // 1. Buat variabel dengan tipe data berbeda
    $nama = "Fathan";
    $umur = 20;
    $isPelajar = true;
    $hobi = ["Coding", "Membaca"];
    
    // 2. Tampilkan dengan echo
    echo "Nama: $nama, Umur: $umur\n";
    
    // 3. Ternary operator
    $status = $umur >= 17 ? "Dewasa" : "Anak-anak";
    echo "Status: $status\n";
    
    // 4. Null coalescing
    $kota = $_GET['city'] ?? 'Jakarta';
    echo "Kota: $kota\n";
    
    // 5. Match expression
    $grade = 'A';
    $nilai = match($grade) {
        'A' => 'Sangat Baik',
        'B' => 'Baik',
        default => 'Cukup',
    };
    echo "Nilai: $nilai\n";
    ?>
    Jalankan: php latihan-sintaks.php

✍️  Eksplorasi strict types:
    Buat file test-strict.php:
    <?php
    declare(strict_types=1);
    
    function tambah(int $a, int $b): int {
        return $a + $b;
    }
    
    echo tambah(5, 3);    // OK
    echo tambah(5, "3");  // Error — strict types!
    ?>

✍️  Eksplorasi match vs switch:
    Buat perbandingan match dan switch untuk role-based message

🔬  Buka app/Models/Product.php di codebase
    Lihat deklarasi strict_types, type hints, return types
    Buka app/Services/CartService.php
    Lihat bagaimana type declarations digunakan
```

---

## 🔗 Referensi & Lanjutan  

| Topik | Di Journey Ini |
|-------|----------------|
| String manipulation detail | FASE-02: 02-03-array-dan-string |
| Fungsi dan scope | FASE-02: 02-04-fungsi-dan-scope |
| OOP di PHP (class, object) | FASE-03: semua dokumen |
| Laravel Routing | FASE-06: 06-03-routing-web-php |
| PHP Type System | php.net/manual/en/language.types.php |

---

*"Sintaks hanyalah alat — yang penting adalah logika di baliknya.  
Tapi tanpa menguasai alat, logika terbaik pun tidak bisa diwujudkan."*

*Hafalkan sintaks dasar dengan menulis, bukan dengan membaca.*
