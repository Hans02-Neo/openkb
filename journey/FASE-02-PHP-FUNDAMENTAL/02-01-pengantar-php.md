# 02-01: Pengantar PHP

> **Fase**: 2 — PHP Fundamental  
> **Prasyarat**: FASE 1 (selesai)  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: PHP, Zend Engine, OPcache, PHP lifecycle, SAPI, CLI, FPM, mod_php, php.ini, server API  

---

## 📋 Ringkasan  

Fase 1 memberi mu fondasi tentang komputer, data, dan algoritma.  
Sekarang kita akan mempelajari **PHP** — bahasa yang digunakan oleh Laravel  
dan oleh codebase `olshop-koneksi`.  

Dokumen ini adalah **pengantar menyeluruh** tentang PHP:  
- Sejarah — kenapa PHP ada dan kenapa masih relevan  
- Bagaimana PHP bekerja di dalam (Zend Engine, OPcache)  
- Berbagai cara PHP berjalan (CLI, FPM, mod_php)  
- Tools yang kamu butuhkan  
- Script PHP pertamamu  

**Target pemahaman:**  
- Kamu paham posisi PHP dalam ekosistem web  
- Kamu bisa menjelaskan alur file `.php` dari ditulis sampai menghasilkan HTML  
- Kamu bisa menjalankan script PHP dari terminal dan dari browser  

---

## 1. Apa Itu PHP?  

### 1.1 Definisi  

**PHP** (PHP: Hypertext Preprocessor) adalah **bahasa pemrograman server-side**  
yang dirancang khusus untuk pengembangan web.  

Artinya:  
- **Server-side** — kode PHP dijalankan di **server** (bukan di browser)  
- **Web-oriented** — fitur-fiturnya dirancang untuk membuat halaman web dinamis  
- **Embedded** — kode PHP bisa disisipkan (embedded) langsung di dalam HTML  

### 1.2 Sejarah Singkat  

| Tahun | Kejadian |
|-------|----------|
| **1994** | Rasmus Lerdorf membuat "Personal Home Page Tools" — sekumpulan skrip Perl untuk tracking pengunjung websitenya |
| **1995** | Rilis PHP 1.0 — masih berupa tool sederhana |
| **1997** | PHP 3.0 — ditulis ulang oleh Andi Gutmans dan Zeev Suraski. Nama berubah menjadi "PHP: Hypertext Preprocessor" |
| **2000** | PHP 4.0 — memperkenalkan **Zend Engine 1**, mesin eksekusi pertama |
| **2004** | PHP 5.0 — **Zend Engine 2** dengan dukungan OOP yang serius |
| **2015** | PHP 7.0 — lompatan performa besar (2x lebih cepat dari PHP 5) |
| **2020** | PHP 8.0 — **JIT Compiler**, named arguments, attributes, match expression |
| **2022** | PHP 8.2 — readonly classes, random extension |
| **2024** | PHP 8.3 — json_validate, override attribute |
| **2025** | PHP 8.4 — property hooks, asymmetric visibility, lazy objects |

**PHP hari ini:**  

```
PHP 8.x digunakan oleh:
- 77% dari seluruh website yang menggunakan bahasa server-side
- WordPress (43% dari semua website di internet)
- Laravel (framework #1 untuk PHP)
- Facebook (HHVM — varian PHP buatan Meta)

Sumber: W3Techs, 2026
```

### 1.3 Kenapa PHP Bertahan 30 Tahun?  

| Alasan | Penjelasan |
|--------|-----------|
| **Mudah dipelajari** | Sintaks sederhana, dokumentasi luar biasa (php.net) |
| **Zero configuration** | File `.php` langsung jalan — tidak perlu compile |
| **Hosting friendly** | Semua hosting support PHP — cPanel, shared hosting |
| **Ekosistem kaya** | Laravel, WordPress, Composer, Symfony, dll. |
| **Performa** | PHP 8.x sangat cepat — JIT compiler, OPcache |
| **Komunitas besar** | Stack Overflow, GitHub, forum — apapun masalahnya, sudah ada jawabannya |

---

## 2. Arsitektur PHP — Bagaimana PHP Bekerja  

### 2.1 Zend Engine — Jantung PHP  

**Zend Engine** adalah mesin virtual yang mengeksekusi kode PHP.  
Dia adalah inti dari PHP — semua kode PHP pada akhirnya dijalankan oleh Zend Engine.  

```
┌─────────────────────────────────────────────────────────────┐
│                      PHP INTERPRETER                         │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   ZEND ENGINE                          │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │   │
│  │  │   LEXER      │  │   PARSER     │  │ COMPILER   │  │   │
│  │  │ (Tokenizer)  │─▶│  (Syntax)    │─▶│ (OPcode)   │  │   │
│  │  └──────────────┘  └──────────────┘  └──────┬─────┘  │   │
│  │                                              ▼        │   │
│  │  ┌──────────────────────────────────────────────────┐  │   │
│  │  │            EXECUTOR (OPcode Executor)            │  │   │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐ │  │   │
│  │  │  │ Memory   │ │ Variable │ │ Function/Class   │ │  │   │
│  │  │  │ Manager  │ │ Handler  │ │ Handler          │ │  │   │
│  │  │  └──────────┘ └──────────┘ └──────────────────┘ │  │   │
│  │  └──────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌───────────────────┐    ┌──────────────────┐               │
│  │   EXTENSIONS       │    │   OPcache         │               │
│  │   (PDO, JSON,     │    │   (Shared Memory)  │               │
│  │    mbstring, curl) │    │   ┌────────────┐ │               │
│  └───────────────────┘    │   │OPcode Cache│ │               │
│                            │   └────────────┘ │               │
│                            └──────────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

**Alur eksekusi kode PHP langkah demi langkah:**  

```
File: script.php
<?php
$nama = "Fathan";
echo "Halo, $nama!";
?>

STEP 1 — LEXING (Tokenization)
   │  Zend Engine membaca file .php karakter per karakter
   │  Mengelompokkan menjadi TOKEN
   │
   │  "<?php"  → T_OPEN_TAG
   │  "$nama"  → T_VARIABLE
   │  "="      → T_ASSIGN
   │  ""Fathan"" → T_CONSTANT_ENCAPSED_STRING
   │  ";"      → T_SEMICOLON
   │  "echo"   → T_ECHO
   │  "$nama"  → T_VARIABLE
   │  ";"      → T_SEMICOLON
   │  "?>"     → T_CLOSE_TAG
   │
   ▼
STEP 2 — PARSING
   │  Parser memeriksa apakah urutan token VALID secara sintaks
   │  Membangun AST (Abstract Syntax Tree)
   │
   │  AST:
   │  ┌──────────────────────────────────────┐
   │  │ Script                              │
   │  │   ├── Assign: $nama = "Fathan"      │
   │  │   └── Echo: $nama                   │
   │  └──────────────────────────────────────┘
   │
   ▼
STEP 3 — COMPILATION (ke OPcode)
   │  AST diterjemahkan ke OPcode — instruksi biner intermediate
   │  Mirip "assembly" tapi untuk Zend Engine
   │
   │  OPcode: (disederhanakan)
   │  ASSIGN $nama, "Fathan"
   │  ECHO $nama
   │  RETURN 1
   │
   ▼
STEP 4 — EXECUTION
   │  Zend Engine menjalankan OPcode baris per baris
   │  1. Buat variabel $nama di memori, isi "Fathan"
   │  2. Ambil nilai $nama, kirim ke output buffer
   │  3. Return 1 (sukses)
   │
   ▼
STEP 5 — OUTPUT
   │  Output buffer dikirim ke web server (Apache/Nginx)
   │  Web server kirim ke browser sebagai HTTP response
   │
   │  Browser menerima: "Halo, Fathan!"
```

### 2.2 OPcache — Mempercepat Eksekusi  

Setiap kali file PHP di-request, secara default PHP akan:  

```
Request 1:  Lexing → Parsing → Compilation → Execution  (lambat)
Request 2:  Lexing → Parsing → Compilation → Execution  (lambat lagi!)
Request 3:  Lexing → Parsing → Compilation → Execution  (sama lambatnya!)
```

Ini boros! Langkah 1-3 (lexing → parsing → compilation) selalu menghasilkan  
OPcode yang **sama** untuk file yang sama — kecuali file-nya berubah.  

**OPcache** menyimpan OPcode hasil kompilasi di **shared memory** (RAM).  

```
Request 1:  Lexing → Parsing → Compilation → Execution
                  ↓ OPcode disimpan di shared memory (OPcache)

Request 2:  (skip lexing, skip parsing, skip compilation)
                  ↓ OPcode diambil dari memory
            Execution (JAUH LEBIH CEPAT!)

Request 3:  Sama — langsung execution
```

**Perbedaan kecepatan:**  

| Tanpa OPcache | Dengan OPcache |
|---------------|----------------|
| ~10-20 ms per request | ~1-3 ms per request |
| CPU 100% untuk kompilasi | CPU hanya untuk eksekusi |
| Setiap request lambat | Hanya request pertama lambat |

**Cek status OPcache di Laragon:**  

```bash
# Buat file phpinfo di C:\laragon\www\:
# isi: <?php phpinfo();
# Buka: http://localhost/phpinfo.php
# Cari: OPcache → harus "enabled"

# Atau via terminal:
php -i | grep opcache
```

### 2.3 Lifecycle Satu Request PHP  

Inilah yang terjadi setiap kali browser meminta halaman `.php`:  

```
BROWSER                          SERVER (Laragon)
   │                                  │
   │  GET /shop.php                   │
   │─────────────────────────────────▶│
                                     │
                   1. Apache menerima request
                   → Cocokkan URL dengan file
                   → File: /www/shop.php (akhiran .php)
                   → Panggil PHP (via mod_php atau FPM)
                                     │
                   2. PHP INITIALIZATION
                   │  - Baca php.ini (konfigurasi PHP)
                   │  - Load extension (PDO, JSON, curl, dll)
                   │  - Init global variables ($_GET, $_POST, $_SERVER)
                   │  - Init memory manager
                   │
                   3. PHP EXECUTION
                   │  - Cek OPcache: ada OPcode untuk shop.php?
                   │    → Ya: ambil dari cache
                   │    → Tidak: lexing → parsing → compilation
                   │  - Eksekusi OPcode:
                   │    • Kode PHP dijalankan baris per baris
                   │    • Koneksi ke database (jika perlu)
                   │    • Query database
                   │    • Render output (echo, HTML)
                   │
                   4. PHP CLEANUP
                   │  - Kirim output buffer
                   │  - Destroy semua variabel
                   │  - Tutup koneksi database
                   │  - Free memory
                   │
   │                                  │
   │  HTTP Response (HTML)            │
   │◀─────────────────────────────────│
   │                                  │
   Browser render HTML
```

**Total waktu:**  
- **PHP init + cleanup**: ~1-5 ms  
- **PHP execution** (kode Laravel): ~50-300 ms  
- **Total**: ~51-305 ms per request (di localhost)  

### 2.4 SAPI — Server API  

PHP bisa berjalan dalam mode yang berbeda, disebut **SAPI** (Server API).  

```
PHP SAPI (Server Application Programming Interface)
   │
   ├── mod_php (Apache Module)
   │     PHP tertanam di dalam Apache
   │     Satu proses Apache bisa handle banyak request PHP
   │     → Digunakan Laragon secara default
   │
   ├── FPM (FastCGI Process Manager) ★ MODERN
   │     PHP jalan sebagai proses terpisah
   │     Apache/Nginx kirim request ke PHP-FPM via FastCGI
   │     → Lebih stabil, lebih aman, lebih cepat
   │     → Recommended untuk production
   │
   ├── CLI (Command Line Interface)
   │     PHP jalan dari terminal
   │     Tidak ada web server — langsung output ke console
   │     → php artisan, composer, script otomatis
   │
   └── Embedded
         PHP tertanam di aplikasi lain (jarang)
```

**Di Laragon:**  
- Default: **mod_php** (Apache) — satu paket, mudah  
- Bisa diubah ke **nginx + PHP-FPM** melalui menu Laragon  

---

## 3. Toolchain PHP — Yang Ada di Komputermu  

### 3.1 PHP Binary  

```bash
# Cek versi PHP
C:\laragon\www\olshop-koneksi> php --version
PHP 8.3.30 (cli) (built: ... )
Copyright (c) The PHP Group
Zend Engine v4.3.30, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.30, Copyright (c), by Zend Technologies
    with JIT v0.1.0-dev, Copyright (c), by Zend Technologies

# Informasi detail
php -i                        # phpinfo di terminal (ribuan baris)
php -i | grep "php.ini"      # Cari lokasi file konfigurasi
php -i | grep "Loaded Configuration File"
# Output: C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.ini

# Mode yang tersedia
php -v                        # versi
php -m                        # module/extension yang terload
php -r "echo 'hello';"        # jalanin kode langsung (tanpa file)
php -a                        # interactive shell (REPL)
php -l file.php               # lint: cek syntax error saja
```

### 3.2 php.ini — Konfigurasi PHP  

File `php.ini` adalah **pusat konfigurasi PHP**. Semua setting PHP ada di sini.  

```ini
; C:\laragon\bin\php\php-8.3.30-Win32-vs16-x64\php.ini

; IMPORTANT — setting yang sering diubah:

; BERAPA BANYAK MEMORI YANG BOLEH DIPAKAI SATU SCRIPT?
memory_limit = 128M        ; Maximum per script (M = Megabyte)
                           ; Laravel butuh minimal 128M, ideal 256M

; BERAPA BESAR FILE YANG BOLEH DI-UPLOAD?
upload_max_filesize = 64M  ; Maximum file upload
post_max_size = 64M        ; Maximum POST data (harus >= upload_max_filesize)

; APAKAH ERROR DITAMPILKAN?
display_errors = On        ; Tampilkan error di layar (ON = development)
                           ; OFF = production (untuk keamanan)
display_startup_errors = On
error_reporting = E_ALL    ; Laporkan SEMUA error (termasuk deprecation)

; TIMEZONE
date.timezone = "Asia/Jakarta"

; EKSTENSI YANG DI-LOAD
extension=curl             ; HTTP requests
extension=gd               ; Image processing
extension=mbstring         ; Multi-byte string (UTF-8)
extension=mysqli           ; MySQL (old)
extension=pdo_mysql        ; MySQL via PDO (modern — dipakai Laravel)
extension=openssl          ; Enkripsi, HTTPS
extension=zip              ; ZIP file handling
extension=fileinfo         ; Deteksi tipe file

; OPCACHE
opcache.enable=1
opcache.memory_consumption=128   ; Berapa RAM untuk OPcache
opcache.max_accelerated_files=10000
opcache.revalidate_freq=2        ; Cek perubahan file tiap 2 detik
; opcache.validate_timestamps=0  ; 0 = production (tidak perlu cek)
```

### 3.3 Composer — Dependency Manager PHP  

Composer adalah **npm-nya PHP**. Dia mengelola library/packages yang  
digunakan proyek.  

```bash
# Cek composer
composer --version

# Inisialisasi proyek baru
composer init

# Install package
composer require laravel/framework

# Install semua dependency (dari composer.json)
composer install

# Update dependency
composer update

# Hapus package
composer remove laravel/telescope

# Autoload — regenerate class loader
composer dump-autoload
```

**Bagaimana Composer Bekerja:**  

```
composer.json (kamu tulis)
     │
     ▼
Composer membaca: "saya butuh laravel/framework ^10.0"
     │
     ▼
Composer cek di packagist.org (registry packages PHP)
     │
     ▼
Composer download package + semua dependensinya
     │
     ▼
Composer generate:
  - vendor/              ← semua library
  - composer.lock       ← versi exact yang terinstall
  - vendor/autoload.php ← auto-loading (tidak perlu require manual)

Di kode kamu:
require __DIR__ . '/vendor/autoload.php';
// ← Semua class dari semua package langsung bisa dipakai!
```

**Di codebase ini (olshop-koneksi):**  

```bash
# Lihat semua package yang terinstall
composer show

# Output (sebagian):
# laravel/framework       v13.8.0   Framework utama
# laravel/telescope       v5.20.0   Debugging assistant
# midtrans/midtrans-php   v2.6.0    Payment gateway
# spatie/laravel-medialibrary v11.0.0  Upload & manage images
```

---

## 4. Script PHP Pertama — "Hello, World!"  

### 4.1 Mode Terminal (CLI)  

Buat file `hello.php` di folder mana pun:  

```php
<?php
echo "Hello, World!\n";
echo "Selamat belajar PHP!\n";
?>
```

Jalankan:  

```bash
php hello.php
# Output:
# Hello, World!
# Selamat belajar PHP!
```

### 4.2 Mode Web (Browser)  

Buat file di `C:\laragon\www\test.php`:  

```php
<?php
$nama = "Fathan";
$waktu = date("H:i");
?>
<!DOCTYPE html>
<html>
<head>
    <title>Belajar PHP</title>
</head>
<body>
    <h1>Halo, <?php echo $nama; ?>!</h1>
    <p>Sekarang jam <?= $waktu; ?></p>
    
    <?php if (date("H") < 12): ?>
        <p>Selamat pagi!</p>
    <?php elseif (date("H") < 18): ?>
        <p>Selamat sore!</p>
    <?php else: ?>
        <p>Selamat malam!</p>
    <?php endif; ?>
</body>
</html>
```

Akses: `http://localhost/test.php`  

**Lihat source code di browser:**  
Klik kanan → View Page Source:  

```html
<!DOCTYPE html>
<html>
<head>
    <title>Belajar PHP</title>
</head>
<body>
    <h1>Halo, Fathan!</h1>         <!-- PHP sudah diproses -->
    <p>Sekarang jam 14:30</p>       <!-- Hasil dari date() -->
    <p>Selamat sore!</p>            <!-- Hasil dari if -->
</body>
</html>
```

**Tidak ada kode PHP di source browser!** — karena PHP sudah diproses  
di server sebelum HTML dikirim ke browser.  

### 4.3 Perbedaan Echo vs Print  

```php
<?php
// echo — yang paling sering dipakai
echo "Halo";                    // string
echo "Halo", " ", "Dunia";      // multiple arguments
echo 12345;                     // angka
echo $variabel;                 // variabel
echo "<b>Teks tebal</b>";       // HTML (akan di-render browser)

// print — sama, tapi return value (1) dan hanya 1 argument
print "Halo";                   // valid
print "Halo", "Dunia";        // ERROR — hanya 1 argument

// printf — format string (seperti C)
printf("Harga: Rp %s", number_format(50000, 0, ',', '.'));
// Output: Harga: Rp 50.000

// Short echo tag (<?=)
// Ini hanyalah shortcut untuk <?php echo ...
?>
<p><?= $nama ?></p>            <!-- sama dengan <?php echo $nama; ?> -->
```

### 4.4 PHP Tags — Cara Menyisipkan PHP di HTML  

Ada beberapa cara menulis kode PHP:  

```php
// 1. Standard tag (paling umum — SELALU PAKAI INI)
<?php
    // kode PHP di sini
?>

// 2. Short echo tag (untuk echo saja — sering di Blade)
<?= $variabel ?>
// Sama dengan: <?php echo $variabel; ?>

// 3. Short tag (tidak direkomendasikan — butuh konfigurasi)
<? echo "Hello"; ?>
// ❌ Tidak portabel — beberapa server tidak support

// 4. ASP tag (tidak direkomendasikan — legacy)
<% echo "Hello"; %>
// ❌ Legacy — dihapus di PHP 7+
```

**Best practice:**  
- **Selalu pakai `<?php`** untuk blok kode  
- **Pakai `<?=`** hanya untuk echo cepat di template (Blade atau langsung)  

### 4.5 Komentar — Menulis Catatan di Kode  

```php
<?php
// Ini komentar satu baris (paling sering dipakai)

# Ini juga komentar satu baris (gaya shell)

/*
 * Ini komentar multi-baris
 * Bisa untuk beberapa baris
 * Biasanya untuk dokumentasi fungsi/class
 */

/**
 * Ini DocBlock — komentar untuk dokumentasi formal
 * @param string $nama Nama pengguna
 * @return string Pesan sapaan
 */
function sapa(string $nama): string
{
    return "Halo, $nama!";
}
?>
```

**Kenapa komentar penting:**  

```php
// ❌ KOMENTAR BURUK — hanya bilang APA yang dilakukan
$i++; // increment variable i

// ✅ KOMENTAR BAIK — bilang MENGAPA dilakukan
// Harga dalam sen — larangan pakai float untuk uang!
// Konversi dari rupiah ke sen: 25000 → 2500000
$priceInCents = $priceRupiah * 100;
```

---

## 5. Variabel Superglobal — Data yang Selalu Ada  

PHP memiliki **variabel global bawaan** yang bisa kamu akses dari mana saja.  
Ini adalah jembatan antara PHP dan dunia luar (browser, server, dll).  

```php
<?php
// $_GET — data dari URL (query string)
// URL: http://example.com/shop?category=sepatu&sort=price
$category = $_GET['category']; // "sepatu"
$sort = $_GET['sort'];         // "price"

// $_POST — data dari form submission (method POST)
$nama = $_POST['nama'];
$email = $_POST['email'];

// $_REQUEST — gabungan GET + POST + COOKIE
$data = $_REQUEST['field'];

// $_SERVER — informasi server dan request
$_SERVER['REQUEST_METHOD'];    // "GET" / "POST"
$_SERVER['HTTP_HOST'];         // "olshop-koneksi.test"
$_SERVER['REQUEST_URI'];       // "/shop?category=sepatu"
$_SERVER['HTTP_USER_AGENT'];   // "Mozilla/5.0 ..."
$_SERVER['REMOTE_ADDR'];       // IP pengunjung

// $_SESSION — data session (harus start_session() dulu)
session_start();
$_SESSION['user_id'] = 5;
$_SESSION['cart'] = ['items' => [...]];

// $_COOKIE — data cookie
$theme = $_COOKIE['theme'];    // "dark"

// $_FILES — file upload
$file = $_FILES['gambar'];
$file['name'];                 // nama file asli
$file['tmp_name'];             // lokasi temporary
$file['size'];                 // ukuran dalam byte

// $_ENV — environment variables (dari .env)
$dbHost = $_ENV['DB_HOST'];

// $_SERVER['argv'] dan $_SERVER['argc'] — CLI arguments
// $argv = array of arguments
// $argc = count of arguments
?>
```

⚠️ **Di Laravel, kamu JARANG pakai variabel superglobal langsung.**  
Laravel membungkus semuanya dalam object yang lebih aman dan terstruktur:  

```php
// ❌ PHP mentah — langsung akses superglobal
$name = $_POST['name'];

// ✅ Laravel — via Request object
$name = $request->input('name');
$name = $request->validated()['name'];

// ❌ Langsung session
$_SESSION['user_id'] = 5;

// ✅ Laravel session
session(['user_id' => 5]);
Session::put('user_id', 5);
```

Tapi **kamu WAJIB tahu superglobal** karena:  
1. Mereka adalah fondasi — Laravel membangun abstraksi di atasnya  
2. Saat debug, kamu perlu paham apa yang terjadi di balik layar  
3. Kadang ada kasus edge yang perlu akses langsung  

---

## 6. PHP vs Template Lain — Blade nanti  

Di dokumen ini kita pakai PHP langsung di HTML. Di Laravel, kamu akan  
menggunakan **Blade**, template engine yang membuat HTML + PHP  
jauh lebih bersih.  

```php
// PHP langsung — campur aduk
<?php if ($isLoggedIn): ?>
    <p>Selamat datang, <?= $name ?></p>
<?php endif; ?>

// Blade — lebih bersih
@if($isLoggedIn)
    <p>Selamat datang, {{ $name }}</p>
@endif
```

Tapi untuk sekarang, pahami dulu PHP murni. Blade hanyalah **syntactic sugar**  
di atas PHP — semua directive Blade pada akhirnya dikompilasi ke PHP biasa.  

---

## 7. Ringkasan — Peta Mental PHP  

```
PHP (PHP: Hypertext Preprocessor)
   │
   ├── SIFAT
   │   ├── Server-side       — jalan di server
   │   ├── Dynamic typing    — tipe data otomatis
   │   ├── Interpreted       — via Zend Engine
   │   └── Embedded          — bisa di dalam HTML
   │
   ├── EKSEKUSI
   │   ├── Lexing    → Token
   │   ├── Parsing   → AST
   │   ├── Compile   → OPcode  ──▶ OPcache
   │   └── Execute   → Output  ←── JIT (PHP 8+)
   │
   ├── SAPI (Mode Jalan)
   │   ├── mod_php       — Apache module
   │   ├── FPM           — FastCGI (production)
   │   ├── CLI           — Terminal
   │   └── php -r / -a   — One-off / interactive
   │
   ├── TOOLS
   │   ├── php.ini       — Konfigurasi
   │   ├── Composer      — Dependency manager
   │   └── OPcache       — Caching OPcode
   │
   └── SUPER GLOBAL
       ├── $_GET, $_POST, $_REQUEST
       ├── $_SERVER, $_SESSION
       ├── $_COOKIE, $_FILES
       └── $_ENV
```

### Yang Harus Kamu Ingat  

1. **PHP = server-side** — tidak bisa dilihat dari browser  
2. **File .php** bisa campur HTML dan PHP dalam satu file  
3. **`<?php ... ?>`** untuk blok kode, **`<?= ... ?>** untuk echo  
4. **Semua kode PHP** melalui Zend Engine: Lexing → Parsing → Compile → Execute  
5. **OPcache** menyimpan OPcode di RAM — request kedua jauh lebih cepat  
6. **Composer** mengelola library — seperti npm untuk JavaScript  
7. **Superglobal** adalah data bawaan dari browser/server — fondasi web PHP  
8. **Laravel** membungkus superglobal — tapi kamu harus tahu aslinya  

---

## 📌 PRAKTIK — Kerjakan Ini  

```
✍️  Cek versi PHP di komputermu:
    php --version
    php -i | grep "Loaded Configuration"
    php -i | grep "opcache.enable"

✍️  Buat file test.php di C:\laragon\www\:
    Isi:
    <?php
    phpinfo();
    ?>
    Buka: http://localhost/test.php
    Cari: PHP Version, Loaded Configuration File, OPcache

✍️  Buat file biodata.php di C:\laragon\www\:
    <?php
    $nama = "Nama Kamu";
    $umur = 20;
    $hobi = ["Coding", "Membaca", "Olahraga"];
    ?>
    <h1>Biodata</h1>
    <p>Nama: <?= $nama ?></p>
    <p>Umur: <?= $umur ?></p>
    <h2>Hobi:</h2>
    <ul>
    <?php foreach ($hobi as $h): ?>
        <li><?= $h ?></li>
    <?php endforeach; ?>
    </ul>
    Akses: http://localhost/biodata.php

✍️  Coba superglobal $_SERVER:
    Buat file info.php:
    <?php
    echo "<pre>";
    print_r($_SERVER);
    echo "</pre>";
    ?>
    Lihat: HTTP_HOST, REQUEST_URI, REMOTE_ADDR, REQUEST_METHOD

✍️  Cek Composer di proyek:
    cd C:\laragon\www\olshop-koneksi
    composer show | Select-String "laravel"
```

---

## 🔗 Referensi & Lanjutan  

| Topik | Di Journey Ini |
|-------|----------------|
| Sintaks PHP lebih detail (variabel, operator, tipe data) | FASE-02: 02-02-sintaks-dasar-php |
| String manipulation | FASE-02: 02-03-array-dan-string |
| Fungsi dan scope | FASE-02: 02-04-fungsi-dan-scope |
| PHP di web server (Apache/Nginx) | FASE-02: 02-05-php-dan-web-server |
| Blade template (Laravel) | FASE-06: 06-06-blade-template-engine |
| Composer + autoloading detail | FASE-11: 11-01-apa-itu-composer |

---

*"PHP bukan bahasa yang sempurna — tapi dia adalah bahasa yang  
menggerakkan sebagian besar web. Untuk belajar web development  
di Indonesia, PHP adalah fondasi yang paling praktis."*

*Praktikkan setiap contoh. Jangan hanya baca — tulis sendiri.*
