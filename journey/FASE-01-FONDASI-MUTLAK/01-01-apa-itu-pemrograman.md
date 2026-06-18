# 01-01: Apa Itu Pemrograman?

> **Fase**: 1 — Fondasi Mutlak
> **Prasyarat**: Tidak ada
> **Waktu baca**: 30-45 menit
> **Kata kunci**: program, code, bahasa pemrograman, compiler, interpreter, binary, source code

---

## 📋 Ringkasan

Dokumen ini adalah fondasi paling dasar dari seluruh perjalanan belajarmu.
Kamu akan memahami apa sebenarnya yang dimaksud dengan "pemrograman"
(programming), bagaimana komputer menjalankan perintah, dan mengapa kita
perlu "belajar coding" untuk membuat komputer melakukan apa yang kita mau.

**Target pemahaman** setelah membaca:
- Kamu bisa menjelaskan apa itu program dengan analogi sederhana
- Kamu paham perbedaan antara kode yang kamu tulis dan yang dijalankan komputer
- Kamu tahu mengapa ada banyak bahasa pemrograman
- Kamu mengerti alur: **ide → kode → eksekusi → hasil**

---

## 1.1 Definisi Paling Dasar

### Apa Itu Program?

**Program** adalah sekumpulan instruksi yang memberitahu komputer apa yang harus dilakukan.

Analogikan dengan **resep masakan**:

```
Resep Masakan (Program)
┌──────────────────────────────────────┐
│ 1. Potong bawang                     │ ← Instruksi 1
│ 2. Panaskan minyak                   │ ← Instruksi 2
│ 3. Tumis bawang hingga harum         │ ← Instruksi 3
│ 4. Masukkan telur                    │ ← Instruksi 4
│ 5. Aduk hingga matang                │ ← Instruksi 5
│ 6. Sajikan                           │ ← Instruksi 6
└──────────────────────────────────────┘
     │
     ▼
[Kamu sebagai "komputer"] → mengikuti instruksi → menghasilkan telur dadar
```

Sama persis dengan program komputer:

```
Program Komputer
┌─────────────────────────────────────────┐
│ 1. Ambil angka dari keyboard            │ ← Input
│ 2. Kalikan angka dengan 2               │ ← Proses
│ 3. Tampilkan hasil di layar             │ ← Output
└─────────────────────────────────────────┘
     │
     ▼
Komputer menjalankan instruksi → menghasilkan output
```

### Apa Itu Pemrograman (Programming)?

**Pemrograman** adalah aktivitas **menulis instruksi** (dalam bahasa khusus
yang bisa dipahami komputer) untuk menyelesaikan suatu masalah.

Seorang **programmer** (atau developer) adalah orang yang:
1. Memahami **masalah** yang perlu diselesaikan
2. Merancang **solusi** dalam bentuk langkah-langkah logis
3. Menulis **kode** (instruksi) dalam bahasa pemrograman
4. Menguji apakah solusi berjalan dengan benar
5. Memperbaiki jika ada kesalahan

### Analogi Kehidupan Nyata

| Aktivitas Manusia | Analogi Programming |
|------------------|---------------------|
| Resep masakan | Source code (kode sumber) |
| Kamu yang masak | Komputer (processor) |
| Bahan makanan | Data input |
| Masakan jadi | Output / hasil |
| Rasa tidak enak | Bug (error) |
| Goreng lagi | Debugging (perbaiki) |
| Buku resep | Aplikasi / software |

---

## 1.2 Bagaimana Komputer Bekerja?

### Ilusi "Kecerdasan" Komputer

Komputer sebenarnya **BODOH**. Dia hanya bisa melakukan satu hal:
**mengeksekusi instruksi** dengan sangat cepat dan tepat.

Komputer TIDAK bisa:
- Berpikir kreatif
- Membuat keputusan sendiri
- Mengerti konteks atau maksud tersirat
- Belajar tanpa diprogram

Komputer hanya bisa:
- Membaca angka 0 dan 1 (binary)
- Melakukan operasi aritmetika (tambah, kurang, kali, bagi)
- Membandingkan dua angka (sama, lebih besar, lebih kecil)
- Memindahkan data dari satu tempat ke tempat lain
- Mengulang instruksi yang sama ribuan kali

**Kekuatan komputer bukanlah kecerdasan, melainkan kecepatan dan ketepatan.**

### Binary: Bahasa Mesin yang Sesungguhnya

Di level paling bawah, komputer hanya mengerti **dua keadaan**:

```
Listrik mengalir  = 1 (ON)
Listrik mati      = 0 (OFF)
```

Ini disebut **binary** (biner). Semua data di komputer — angka, huruf,
gambar, video, musik, game — pada akhirnya hanyalah rangkaian angka 0 dan 1.

**Contoh: Huruf 'A' dalam binary**

```
Huruf A
   │
   ▼
01000001
   │
   ▼
Dalam memori komputer: [OFF][ON][OFF][OFF][OFF][OFF][OFF][ON]
```

**Tabel konversi sederhana:**

| Huruf | Binary |
|-------|--------|
| A | 01000001 |
| B | 01000010 |
| C | 01000011 |
| ... | ... |
| Z | 01011010 |

### Tiga Komponen Utama Komputer

```
┌─────────────────────────────────────────────────────────────┐
│                       KOMPUTER                              │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │    CPU        │    │    RAM       │    │    Storage  │   │
│  │  (Processor)  │◄──▶│  (Memory)    │◄──▶│  (Harddisk)  │   │
│  │               │    │              │    │              │   │
│  │  Mengeksekusi │    │  Menyimpan   │    │  Menyimpan   │   │
│  │  instruksi    │    │  data sementara│  │  data permanen│  │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  I/O (Input/Output) — Keyboard, Mouse, Monitor, dll  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

| Komponen | Fungsi | Analogi |
|----------|--------|---------|
| **CPU** | "Koki" — mengeksekusi instruksi | Otak komputasi |
| **RAM** | "Meja dapur" — tempat kerja sementara | Cepat tapi tidak permanen |
| **Storage** | "Lemari resep" — menyimpan program dan data | Lambat tapi permanen |
| **I/O** | "Tangan & mata" — interaksi dengan dunia luar | Input/output |

### Alur Eksekusi Program

```
1. Program disimpan di Storage (harddisk/SSD)
         │
         ▼
2. Program dipindahkan ke RAM (dibuka/dijalankan)
         │
         ▼
3. CPU membaca instruksi dari RAM satu per satu
         │
         ▼
4. CPU mengeksekusi instruksi (hitungan, perbandingan, dll)
         │
         ▼
5. Hasil ditulis kembali ke RAM
         │
         ▼
6. Hasil ditampilkan ke layar (output)
```

Ini terjadi **milyaran kali per detik** (diukur dalam GHz — Gigahertz).

---

## 1.3 Bahasa Pemrograman

### Masalah: Manusia vs Mesin

**Manusia** berpikir dalam bahasa manusia (Indonesia, Inggris) — abstrak, kontekstual.

**Komputer** hanya mengerti binary (0 dan 1) — sangat literal, sangat detail.

**Bahasa pemrograman** adalah **jembatan** antara cara berpikir manusia
dan cara kerja mesin.

### Tingkatan Bahasa Pemrograman

```
Lebih dekat ke Manusia (Mudah dibaca)
        ▲
        │
        │   PHP, Python, JavaScript, Ruby ─── High-Level Language
        │                                    (Bahasa Tingkat Tinggi)
        │
        │   C, C++, Rust ──────────────────── Mid-Level Language
        │                                    (Bahasa Tingkat Menengah)
        │
        │   Assembly ──────────────────────── Low-Level Language
        │                                    (Bahasa Tingkat Rendah)
        │
        ▼   Binary (00101010) ────────────── Machine Code
Lebih dekat ke Mesin (Sulit dibaca)
```

### High-Level Language (contoh PHP)

```php
<?php
// Ini bahasa PHP — manusia bisa baca
$harga = 10000;
$diskon = 0.1;
$total = $harga - ($harga * $diskon);
echo "Total: Rp " . $total;
?>
```

Output: `Total: Rp 9000`

**Yang terjadi di belakang layar** (sangat disederhanakan):

```
Step 1: Simpan angka 10000 di kotak bernama "harga"
         RAM → [harga: 10000]

Step 2: Simpan angka 0.1 di kotak bernama "diskon"
         RAM → [harga: 10000, diskon: 0.1]

Step 3: Ambil harga (10000), kalikan dengan diskon (0.1) = 1000
Step 4: Kurangi harga (10000) dengan hasil (1000) = 9000
Step 5: Simpan hasil (9000) di kotak bernama "total"
         RAM → [harga: 10000, diskon: 0.1, total: 9000]

Step 6: Ambil teks "Total: Rp " dan gabung dengan total (9000)
Step 7: Kirim hasil ke layar
```

### Kenapa Tidak Langsung Pakai Binary?

Bayangkan kamu harus menulis program di binary:

```
// Instruksi: Cetak "Hello, World!" di layar
// Dalam binary (untuk prosesor x86_64):

10110000 00000001   ; mov al, 1
10110001 00001100   ; mov cl, 12
10001001 11001011   ; mov bl, cl
11001101 10000000   ; int 0x80
...

// Mana mungkin kamu mau nulis kayak gini untuk aplikasi toko online?
```

Dengan high-level language (PHP):

```php
<?php
echo "Hello, World!";
?>
```

Jauh lebih mudah dibaca, ditulis, dan dipelihara. **Inilah gunanya bahasa pemrograman.**

### Mengapa Ada Banyak Bahasa Pemrograman?

Setiap bahasa diciptakan untuk tujuan tertentu, seperti alat di kotak perkakas:

| Bahasa | Kegunaan Utama | Analogi |
|--------|---------------|---------|
| **PHP** | Web development (server-side) | Pisau dapur — khusus masak |
| **JavaScript** | Web development (client-side) | Pisau lipat — serbaguna di browser |
| **Python** | Data science, AI, backend | Pisau chef — serbaguna semua bidang |
| **C/C++** | Sistem operasi, game engine | Gergaji mesin — kuat, berat |
| **Java** | Enterprise, Android | Palu — kokoh, struktural |
| **Go** | Network service, microservice | Obeng — cepat, ringan |
| **SQL** | Database query | Alat ukur — khusus query data

**Tidak ada bahasa yang "terbaik"** — yang ada adalah bahasa yang **paling cocok** untuk pekerjaan tertentu.

### Compiler vs Interpreter

Ketika kamu menulis kode, komputer tidak bisa langsung menjalankannya.
Dibutuhkan **penerjemah** untuk mengubah kode tingkat tinggi ke binary (machine code).

Ada dua cara penerjemahan:

#### Interpreter (PHP, JavaScript, Python)

```
Source Code ──▶ Interpreter ──▶ Eksekusi (baris per baris)
                   ▲
                   │
             Diterjemahkan
             saat dijalankan
             (runtime)
```

- Menerjemahkan **sambil jalan** (baris per baris)
- Lebih lambat saat eksekusi (karena terjemahan terjadi real-time)
- Lebih mudah untuk development (langsung lihat hasil)
- **PHP menggunakan model ini** (dengan optimasi tambahan)

#### Compiler (C, C++, Go, Rust)

```
Source Code ──▶ Compiler ──▶ Machine Code (file executable) ──▶ Eksekusi
                   │
                   ▼
           Diterjemahkan SEKALIGUS
           SEBELUM dijalankan
           (build/compile time)
```

- Menerjemahkan **sekaligus** sebelum dijalankan
- Hasilnya file binary (`.exe`, `.out`) yang bisa langsung jalan
- Eksekusi lebih cepat (karena sudah dalam bentuk machine code)
- Proses kompilasi butuh waktu

### PHP: Model Hybrid (Khusus)

PHP menggunakan model **interpreter dengan optimasi compiler**:
- PHP memiliki **Just-In-Time (JIT) Compiler** sejak PHP 8.0
- Kode PHP dikompilasi ke **OPcode** (semacam setengah jalan menuju machine code)
- OPcode di-cache (disimpan) sehingga tidak perlu diterjemahkan ulang
- Ini disebut **OPcache**

```
Source Code (.php)
      │
      ▼
Parser ──▶ AST (Abstract Syntax Tree)
      │
      ▼
Compiler ──▶ OPcode (binary intermediate)
      │
      ▼
OPcache ◀──── Cache OPcode (agar tidak perlu compile ulang)
      │
      ▼
JIT Compiler (PHP 8+) ──▶ Machine Code (untuk bagian yang sering dijalankan)
      │
      ▼
Eksekusi oleh Zend Engine (mesin PHP)
```

**Yang penting kamu pahami sekarang**:
- Kamu menulis `.php` (teks biasa)
- PHP mengubahnya jadi OPcode (mirip kompilasi)
- Zend Engine menjalankan OPcode tersebut (mirip interpreter)
- Hasilnya dikirim ke web server → ke browser

---

## 1.4 Source Code, Program, dan Aplikasi

### Istilah Penting

| Istilah | Definisi | Analogi |
|---------|----------|---------|
| **Source Code** | Teks yang ditulis programmer (file `.php`, `.js`, `.py`) | Resep masakan |
| **Program** | Source code yang sudah siap dijalankan | Masakan jadi |
| **Aplikasi** | Kumpulan program yang punya antarmuka pengguna | Saj lengkap (makanan + piring + sendok) |
| **Software** | Istilah umum untuk program/aplikasi | Perangkat lunak |
| **System Software** | OS, driver, utility | Kompor, oven (infrastruktur) |
| **Application Software** | Browser, game, editor | Makanan (yang langsung dipakai) |

### Alur: Dari Ide ke Aplikasi

```
IDE (Gagasan)
   │
   ▼
── ANALISIS ──▶ Apa yang perlu dibuat?
   │              Contoh: "Saya ingin toko online"
   │
   ▼
── DESAIN ──▶ Bagaimana bentuknya?
   │              Contoh: "Halaman produk, keranjang, checkout"
   │
   ▼
── CODING ──▶ Menulis source code
   │              Contoh: file .php, .js, .css
   │
   ▼
── COMPILE/BUILD ──▶ Menyiapkan program (jika perlu)
   │                    Contoh: npm run build (Vite)
   │
   ▼
── TESTING ──▶ Menguji apakah berfungsi
   │              Contoh: buka di browser, cek error
   │
   ▼
── DEPLOY ──▶ Meletakkan di server agar bisa diakses
   │              Contoh: upload ke hosting / VPS
   │
   ▼
── MAINTENANCE ──▶ Perbaiki bug, tambah fitur
                     Contoh: update kode, deploy ulang
```

---

## 1.5 Programming Paradigm — Cara Berpikir Programmer

Ada beberapa **paradigma** (cara pandang) dalam menulis program:

### 1. Imperative (Prosedural)

"Lakukan ini, lalu itu, lalu itu."

```php
<?php
// Step by step — seperti resep
$buah = ["apel", "pisang", "jeruk"];
for ($i = 0; $i < count($buah); $i++) {
    echo $buah[$i] . "\n";
}
?>
```

**Dokumen ini belum perlu kamu kuasai** — hanya tahu bahwa paradigma ini ada.
Kita akan bahas satu per satu di fase berikutnya.

### 2. Object-Oriented Programming (OOP)

"Buat object yang punya data dan perilaku."

```php
<?php
class Mobil {
    public $warna;
    public function jalan() {
        echo "Mobil $this->warna berjalan...";
    }
}

$mobilku = new Mobil();
$mobilku->warna = "Merah";
$mobilku->jalan(); // Output: "Mobil Merah berjalan..."
?>
```

Ini adalah **paradigma utama Laravel** — akan dipelajari di Fase 3.

### 3. Functional Programming

"Program adalah komposisi fungsi."

```php
<?php
$angka = [1, 2, 3, 4, 5];
$genap = array_filter($angka, fn($n) => $n % 2 == 0);
// Hasil: [2, 4]
?>
```

Akan dipelajari di Fase 4.

---

## 1.6 Programming di Dunia Web (Konkret untuk Codebase Ini)

### Bagaimana PHP Masuk ke Web?

```
Browser (Chrome/Edge)
      │
      │  Request: GET /shop
      │
      ▼
Web Server (Apache/Nginx)
      │
      │ Apache membaca URL → cari file yang cocok
      │
      ▼
PHP Interpreter (Zend Engine)
      │
      │ PHP menjalankan file .php
      │ Ambil data dari database
      │ Render HTML
      │
      ▼
HTML + CSS + JavaScript (dikembalikan ke browser)
      │
      ▼
Browser menampilkan halaman web
```

### Perbedaan File "Biasa" vs File PHP

```
File HTML biasa (.html):
┌────────────────────────┐
│ <h1>Hello!</h1>        │ ← Browser langsung render ini
└────────────────────────┘
File ini dikirim ke browser apa adanya.

File PHP (.php):
┌────────────────────────┐
│ <?php                  │
│ $nama = "Fathan";      │ ← PHP di-SERVER dulu
│ echo "<h1>Halo, ";     │
│ echo $nama;            │
│ echo "</h1>";          │
│ ?>                     │
└────────────────────────┘
Hasil setelah PHP proses (yang dikirim ke browser):
┌────────────────────────┐
│ <h1>Halo, Fathan</h1>  │ ← Baru sampai ke browser
└────────────────────────┘
```

Ini adalah konsep **Server-Side Rendering** — HTML dibuat di server,
dikirim ke browser dalam bentuk jadi. Kamu tidak bisa melihat kode PHP
dari browser (coba saja: view source di browser — yang terlihat hanya HTML).

---

## 1.7 Debugging: Apa Itu Bug?

### Definisi Bug

**Bug** adalah kesalahan dalam program yang menyebabkan hasil tidak sesuai harapan.

Istilah "bug" berasal dari tahun 1947 ketika seorang programmer (Grace Hopper)
menemukan ngengat (bug) tersangkut di relay komputer, menyebabkan komputer
tidak berfungsi. Sejak saat itu, kesalahan program disebut "bug".

### Jenis-jenis Bug

| Jenis | Contoh | Deteksi |
|-------|--------|---------|
| **Syntax Error** | Lupa titik koma `;` | Terdeteksi otomatis saat dijalankan |
| **Logic Error** | Pakai `*` (kali) padahal harus `+` (tambah) | Hasil salah tapi tidak error |
| **Runtime Error** | Bagi angka dengan nol | Error muncul saat dijalankan |
| **Semantic Error** | Program jalan tapi tidak melakukan yang dimaksud | Sulit dideteksi |

### Debugging

**Debugging** adalah proses mencari dan memperbaiki bug.

Cara debugging termudah (yang akan sering kamu lakukan):

```php
<?php
$harga = 10000;
$diskon = 0.1;

// Coba lihat isi variabel
var_dump($harga);    // int(10000)
var_dump($diskon);   // float(0.1)

$total = $harga - ($harga * $diskon);
echo $total;         // 9000 — cek apakah benar?
?>
```

Atau dengan `dd()` di Laravel (akan dipelajari nanti):
```php
dd($harga, $diskon, $total); // Tampilkan dan berhenti
```

---

## 1.8 Ringkasan — Mind Map

```
                          ┌──────────────┐
                          │  PEMROGRAMAN  │
                          │  (Programming)│
                          └──────┬───────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                   ▼
      ┌────────────┐    ┌──────────────┐    ┌──────────────┐
      │   INPUT    │    │    PROSES    │    │   OUTPUT     │
      │  (Data)    │    │ (Instruksi)  │    │   (Hasil)    │
      └────────────┘    └──────────────┘    └──────────────┘
                              │
              ┌───────────────┴──────────────┐
              ▼                              ▼
      ┌──────────────┐             ┌──────────────────┐
      │   MANUSIA     │             │    KOMPUTER       │
      │ Nulis kode    │             │ Jalanin instruksi │
      │ (High-Level)  │             │ (Binary/Machine)  │
      └──────┬───────┘             └────────┬─────────┘
             │                              │
             │    ┌──────────────────┐      │
             └───▶│ BAHASA PEMROGRAMAN│◄────┘
                  │ (Jembatan)       │
                  └──────────────────┘
```

### Yang Harus Kamu Ingat dari Dokumen Ini

1. **Program** = instruksi untuk komputer
2. **Pemrograman** = aktivitas menulis instruksi
3. **Komputer** bodoh tapi cepat — dia hanya menjalankan perintah
4. **Bahasa pemrograman** menjembatani manusia dan mesin
5. **High-level language** (seperti PHP) lebih mudah dibaca manusia
6. **Interpreter** (PHP) menerjemahkan sambil jalan
7. **Compiler** menerjemahkan sebelum jalan
8. **Binary (0 dan 1)** adalah bahasa asli komputer
9. **Web server** menangani request browser → PHP proses → kirim HTML
10. **Bug** adalah kesalahan — debugging adalah cara memperbaikinya

---

## 📌 PRAKTIK — Kerjakan Ini

```
✍️  Buka terminal/cmd di proyek ini.
    Ketik: php -r "echo 'Hello, World!';"
    Lihat hasilnya.

✍️  Buka file .env di proyek.
    Cari baris APP_ENV=local.
    Ini adalah konfigurasi — data yang mempengaruhi cara program berjalan.

🔬  Buka browser, akses http://olshop-koneksi.test
    Klik kanan → View Page Source.
    Apa yang kamu lihat? Apakah ada kode PHP di sana?
    (Jawaban: Tidak ada — PHP sudah diproses di server)

🔬  Coba buat file baru test.php di folder public/
    Isi: <?php echo "Halo dari PHP!";
    Akses: http://olshop-koneksi.test/test.php
    Lihat hasilnya di browser.
```

---

## 🔗 Referensi & Bacaan Lanjutan

| Topik | Di Journey Ini | 
|-------|----------------|
| Cara kerja komputer lebih detail | FASE-01: 01-02-cara-kerja-komputer |
| Variabel dan tipe data | FASE-01: 01-03-variabel-tipe-data-logika |
| Algoritma dasar | FASE-01: 01-04-algoritma-dasar |
| PHP fundamental | FASE-02: semua dokumen |

---

*Dokumen ini adalah bagian dari Learning Journey Koneksi Store.
Pahami betul konsep ini sebelum lanjut ke dokumen berikutnya.
Jika ada istilah yang tidak dimengerti, baca ulang atau catat
untuk ditanyakan.*

*"Programming isn't about what you know; it's about what you can figure out."*
— Chris Pine
