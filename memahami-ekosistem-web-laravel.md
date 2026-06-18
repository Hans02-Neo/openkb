# Memahami Ekosistem Web Laravel — Panduan Lengkap untuk Pemula

## Daftar Isi

1. [Web Development: Cara Kerja Sebenarnya](#1-web-development-cara-kerja-sebenarnya)
2. [Laravel vs Next.js/React — Paradigma Berbeda](#2-laravel-vs-nextjsreact--paradigma-berbeda)
3. [Alur Request-Response di Laravel](#3-alur-request-response-di-laravel)
4. [Server-Side Debugging (Error di Backend)](#4-server-side-debugging-error-di-backend)
5. [Client-Side Debugging (Error di Browser)](#5-client-side-debugging-error-di-browser)
6. [Koneksi Server ↔ Client: Data Flow](#6-koneksi-server--client-data-flow)
7. [Praktik Debugging Sehari-hari](#7-praktik-debugging-sehari-hari)
8. [Glosarium](#8-glosarium)

---

## 1. Web Development: Cara Kerja Sebenarnya

Sebelum paham debugging, kamu harus paham **siapa melakukan apa**.

### Analogi Restoran

```
Browser (Kamu)         Server (Dapur)            Database (Gudang)
    │                        │                         │
    │── Request (Pesan) ──▶ │                          │
    │                        │── Query (Cek stok) ──▶  │
    │                        │◀─ Data (Bahan) ─────────│
    │                        │                         │
    │                        │ Masak (Render)          │
    │                        │                         │
    │◀─ Response (Hidangan)─│                          │
```

### Dua Dunia yang Berbeda

| Aspek | Server-Side (PHP/Laravel) | Client-Side (Browser) |
|-------|--------------------------|----------------------|
| **Tempat eksekusi** | Di komputer server (atau localhost) | Di komputer user (browser) |
| **Bahasa** | PHP | JavaScript (JS) |
| **Tugas** | Ambil data dari DB, logic bisnis, render HTML | Interaksi UI, animasi, validasi form ringan |
| **Yang bisa lihat** | Developer (via log) | User (via console/browser) |
| **Error muncul di** | File `storage/logs/laravel.log` | Browser Console (F12) |

### Kenapa Terasa "Campur Aduk"?

Inilah realitas **traditional server-side rendering**:

```
File .blade.php (Laravel)
┌─────────────────────────────────────────────┐
│ <?php                                       │ ← PHP (SERVER)
│   $products = Product::where('active', 1)   │   Ambil data dari DB
│   ->get();                                  │
│                                             │
│   $total = count($products);                │   Logic server
│ ?>                                          │
│                                             │
│ <div class="grid">                          │ ← HTML
│   @foreach($products as $product)           │ ← Blade (template Laravel)
│     <div x-data="{ qty: 1 }">               │ ← Alpine.js (CLIENT)
│       <h2>{{ $product->name }}</h2>         │ ← Blade echo data PHP
│       <button @click="qty++">+</button>     │ ← Alpine.js event (JS)
│       <span x-text="qty"></span>            │ ← Alpine.js reactivity (JS)
│     </div>                                  │
│   @endforeach                               │
│ </div>                                      │
└─────────────────────────────────────────────┘
```

Dalam SATU file yang sama, kamu punya:
- **PHP** — logic server, query DB
- **Blade** — template engine Laravel (campuran PHP + HTML)
- **HTML** — struktur halaman
- **Alpine.js** — interaktivitas client-side
- **CSS/Tailwind** — styling

Ini BUKAN kekacauan — ini **sifat alami traditional SSR**. Berbeda dengan Next.js yang memisahkan secara eksplisit mana `'use client'` dan mana server component.

---

## 2. Laravel vs Next.js/React — Paradigma Berbeda

### Next.js / React (Full JavaScript)

```
Komponen React (JSX)
┌──────────────────────────────────┐
│ 'use client'                     │ ← Eksplisit: ini client
│ function Button() {              │
│   const [count, setCount] = useState(0)
│   return <button onClick={...}   │
│     {count}                      │
│   </button>                      │
│ }                                │
└──────────────────────────────────┘

Batasan JELAS:
- 'use client' = browser (CSR)
- Tanpa directive = server (SSR)
- Data flow via props/context
```

### Laravel (Traditional SSR + Enhancement)

```
File .blade.php
┌──────────────────────────────────────────┐
│ PHP (SERVER)                             │
│ HTML + Blade (server → render)           │
│ Alpine.js / vanilla JS (CLIENT)          │
│                                          │
│ SEMUA dalam satu file —                  │
│ TIDAK ada pemisahan eksplisit            │
└──────────────────────────────────────────┘

Batasan KABUR:
- PHP dan JS bisa saling bersebelahan
- Data dari PHP langsung disisipkan ke JS
- Server mengirim HTML+CSS sekaligus
```

### Perbandingan Langsung

| Aspek | Next.js/React | Laravel (Blade) |
|-------|---------------|-----------------|
| **Bahasa dominan** | JavaScript (satu bahasa) | PHP + JS + HTML |
| **Pemisahan server/client** | Eksplisit (`'use client'`) | Implisit (campur di blade) |
| **Rendering** | JSX (JavaScript XML) | Blade (PHP template) |
| **State management** | useState, Redux, dll. | Session, Database, Alpine.js |
| **Data dari server** | Fetch API / Server Actions | Langsung di-echo ke blade |
| **Error boundary** | React Error Boundary | Try-catch PHP / Whoops |
| **Transisi halaman** | SPA (router client) | Full page reload (traditional) |

### Kenapa Laravel Terasa "Campur Aduk"?

Karena **Laravel menggunakan pendekatan Traditional Server-Side Rendering (SSR)**:
- Server menyiapkan SEMUANYA (data + HTML)
- Browser tinggal nampilin
- JavaScript hanya untuk "perbaikan" (enhancement), bukan fondasi

Sedangkan **Next.js/React lahir dari dunia Client-Side Rendering (CSR)**:
- JavaScript sebagai fondasi
- Server opsional (untuk SSR/SEO)
- Pemisahan client/server lebih terdefinisi karena JavaScript adalah satu-satunya bahasa

> **Intinya**: Laravel adalah "server-first", Next.js adalah "JavaScript-first". Keduanya valid — hanya pendekatan berbeda.

---

## 3. Alur Request-Response di Laravel

Ini penting untuk debugging — kamu harus tahu DI MANA error terjadi.

```
Browser                                 Server (Laravel)
┌──────────┐                          ┌──────────────────────────┐
│          │  GET /shop               │                          │
│  User    │ ──────────────────────▶  │  1. public/index.php    │
│  types   │                          │     (entry point)        │
│  URL     │                          │       ↓                  │
│          │                          │  2. bootstrap/app.php    │
│          │                          │     (load framework)     │
│          │                          │       ↓                  │
│          │                          │  3. Kernel (HTTP)        │
│          │                          │     - Middleware global  │
│          │                          │     - CSRF, Session, dll │
│          │                          │       ↓                  │
│          │                          │  4. Router (web.php)     │
│          │                          │     Cocokkan URL         │
│          │                          │       ↓                  │
│          │                          │  5. Controller           │
│          │                          │     - Ambil data dari DB │
│          │                          │     - Panggil Service    │
│          │                          │       ↓                  │
│          │                          │  6. View (Blade)         │
│          │                          │     - Render HTML        │
│          │                          │       ↓                  │
│          │◀─────────────────────────│  7. Response (HTML)     │
│          │                          │                          │
│   Browser render HTML               │                          │
│   + jalankan JS (Alpine)            │                          │
│   + minta CSS via Vite              │                          │
│                                      │                         │
└──────────┘                          └──────────────────────────┘
```

### Titik-titik Error potensial:

| Langkah | Dimana | Jenis Error | Cara Debug |
|---------|--------|-------------|------------|
| 1-3 | Framework | 500 Internal Server Error | `storage/logs/laravel.log` |
| 4 | Router | 404 Not Found | `php artisan route:list` |
| 5 | Controller | Logic error, DB error | Laravel log / dd() / Telescope |
| 6 | Blade | Syntax error, variable undefined | Laravel log |
| 7 | Response | HTML/CSS broken | Browser DevTools (Elements) |
| JS | Browser | Console error, fetch failed | Browser Console (F12) |

---

## 4. Server-Side Debugging (Error di Backend)

### 4.1 Laravel Log File — Tempat Utama

```
File: storage/logs/laravel.log
```

Semua error PHP/Laravel tercatat di sini. Cara bacanya:

```bash
# Di Laragon: buka terminal di folder proyek
tail -f storage/logs/laravel.log
```

Atau pakai **Laragon APM Monitor** (ada di tray icon Laragon → APM Monitor).

Contoh isi log:

```
[2026-06-13 20:30:15] local.ERROR: SQLSTATE[42S02]: Base table or view not found
# ^ Waktu        ^ Level      ^ Pesan error

# stack trace:
# /var/www/app/Http/Controllers/ShopController.php:25
# /var/www/vendor/laravel/framework/src/...
```

**Level log** (dari rendah ke tinggi):
- `DEBUG` — Informasi detail untuk developer
- `INFO` — Informasi umum
- `NOTICE` — Normal tapi signifikan
- `WARNING` — Ada yang tidak biasa
- `ERROR` — Error, perlu tindakan
- `CRITICAL` — Komponen tidak berfungsi
- `ALERT` — Perlu tindakan segera
- `EMERGENCY` — Sistem tidak bisa jalan

### 4.2 Cara Manual: dd() dan dump()

**`dd()` = Dump and Die** — Debug favorit semua developer Laravel.

```php
// Di controller mana pun:
$products = Product::where('active', true)->get();

dd($products); // ← Berhenti di sini, tampilkan isi $products

// Hasil: halaman putih dengan dump data (Whoops style)
```

**`dump()`** — Sama seperti dd() tapi TIDAK berhenti.

```php
dump($products); // ← Tampilkan data, tapi lanjutkan eksekusi
```

### 4.3 Laravel Telescope — Dashboard Debugging

Proyek ini sudah punya Telescope. Akses:

```
http://olshop-koneksi.test/telescope
```

Telescope mencatot:
- **Requests** — Setiap request HTTP (method, URL, status, durasi)
- **Queries** — Semua query SQL yang dijalankan (lengkap dengan binding)
- **Exceptions** — Error yang terjadi (stack trace lengkap)
- **Logs** — Semua log yang direkam
- **Dumps** — Hasil `dump()` yang dipanggil
- **Mail** — Email yang dikirim
- **Notifications** — Notifikasi
- **Jobs** — Queue jobs
- **Cache** — Hit/miss cache
- **Schedule** — Cron job terjadwal

Cara pakai:
1. Buka `http://olshop-koneksi.test/telescope`
2. Pilih tab (Exceptions, Queries, dll.)
3. Klik entri untuk lihat detail

### 4.4 Whoops! — Error Page Visual

Laravel 13 menggunakan Whoops untuk halaman error. Saat `APP_DEBUG=true` di `.env`, error akan tampil sebagai halaman interaktif dengan:

- **Pesan error** (jelas)
- **File & line number** (bisa diklik)
- **Stack trace** (lengkap)
- **Kode sumber** (konteks sekitar error)
- **Request data** (input, session, headers)

### 4.5 Debug Query SQL

Mau lihat SQL yang dijalankan Laravel?

**Cara 1 — Telescope** (recommended):
Buka tab Queries di Telescope.

**Cara 2 — Log queries**:
```php
// Di AppServiceProvider.php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

public function boot(): void
{
    DB::listen(function ($query) {
        Log::info($query->sql, $query->bindings, $query->time);
    });
}
```

**Cara 3 — toSql()**:
```php
$query = Product::where('active', true);
dd($query->toSql()); // Lihat SQL mentah tanpa execute
```

### 4.6 Error Handling di Laragon (Khusus)

Laragon punya beberapa alat:

| Tools | Cara Akses | Fungsi |
|-------|-----------|--------|
| **APM Monitor** | Klik kanan Laragon → APM Monitor | Monitor CPU, memory, request |
| **Log Viewer** | Klik kanan Laragon → Apache/Nginx → Logs | Error log Apache/Nginx |
| **PHP Error Log** | `C:\laragon\bin\php\php-8.3\php_error_log` | Error PHP level rendah |

---

## 5. Client-Side Debugging (Error di Browser)

### 5.1 Browser DevTools (F12)

Ini adalah senjata utama untuk debugging client-side. Buka dengan **F12** atau **Ctrl+Shift+I** (di Chrome/Edge).

### Tab-tab Penting:

#### Console (Paling Penting)
```
│ ✅ Console │ Elements │ Sources │ Network   │
│─────────────────────────────────────────────│
│ Uncaught TypeError: Cannot read property     │
│ 'length' of undefined                        │
│   at shop.js:25                              │
│                                              │
│ X GET http://olshop-koneksi.test/api/data    │
│   404 (Not Found)                            │
│                                              │
│ ⬤ [HMR] Waiting for update signal from Vite │
└─────────────────────────────────────────────┘
```

Apa yang muncul di Console:
- **Error JavaScript** (merah) — syntax error, undefined variable, dll.
- **Warning** (kuning) — praktik tidak baik, deprecation
- **Log manual** (`console.log()`, `console.error()`)
- **Network error** — fetch/ajax gagal
- **Vite HMR messages** — hot reload info

#### Network (Kedua Terpenting)
```
│ Console │ ✅ Network │ Elements │ Sources  │
│─────────────────────────────────────────────│
│ Name                        Status │ Type   │
│ shop                         200   │ document│
│ app.css                      200   │ stylesheet
│ app.js                       200   │ script  │
│ /api/products                500   │ xhr     │ ← ERROR!
│ logo.png                    304    │ image   │
└─────────────────────────────────────────────┘
```

Fungsi:
- Lihat **semua request** yang dibuat halaman
- Status code (200 = OK, 404 = Not Found, 500 = Server Error)
- Response body (klik → Preview/Response)
- Request headers
- Timing (berapa lama tiap request)

> **Tips**: Centang "Preserve log" agar log tidak hilang saat reload halaman.

#### Elements
```
│ ✅ Elements │ Console │ Network │ Sources │
│─────────────────────────────────────────────│
│ <div class="product-card">                  │
│   <h2>Sepatu Nike</h2>                      │
│   <span x-data="{ qty: 1 }">                │
│     <button @click="qty++">+</button>       │
│   </span>                                   │
│ </div>                                      │
└─────────────────────────────────────────────┘
```

Fungsi:
- Lihat HTML **yang sudah dirender**
- Inspect CSS (computed styles)
- Edit HTML/CSS langsung (hanya visual, tidak permanen)
- Debug Alpine.js (klik elemen → konsol: `$el.__x.$data`)

### 5.2 Alpine.js Debugging

Karena proyek ini pakai Alpine.js:

```html
<!-- Di elemen mana pun -->
<div x-data="{ count: 0 }" x-init="console.log('Alpine siap!', $data)">
  <span x-text="count"></span>
</div>
```

**Cara debug Alpine:**
1. Buka Console (F12)
2. Klik elemen di tab Elements
3. Ketik di Console:
   ```js
   // Lihat data Alpine dari elemen yang dipilih
   $0.__x.$data

   // Lihat semua $data
   Alpine.$data(document.querySelector('[x-data]'))
   ```

### 5.3 Vite Dev Server

Saat menjalankan `npm run dev` (atau `composer run dev`), Vite menyediakan:
- **Hot Module Replacement (HMR)** — CSS/JS berubah langsung terlihat tanpa reload
- **Error overlay** — Error JS muncul sebagai overlay di browser
- **Source maps** — Error JS tetap bisa dilacak ke file asli (bukan file minified)

**Cek status Vite:**
```
Browser console akan muncul:
⬤ [Vite] connecting...
⬤ [Vite] connected.
```

Kalau tidak muncul:
- Pastikan `npm run dev` berjalan di terminal
- Cek apakah port Vite (default 5173) tidak bentrok

### 5.4 Debugging JavaScript Manual

Cara paling sederhana — tanam `console.log()` di JavaScript:

```javascript
// Di file JS atau di <script> di blade
console.log('Data produk:', products);
console.log('User klik tombol:', event.target);
console.table(arrayData); // Tampilkan array sebagai tabel
console.time('proses');   // Ukur waktu eksekusi
  // ... kode ...
console.timeEnd('proses');
```

---

## 6. Koneksi Server ↔ Client: Data Flow

### 6.1 Cara Data Berpindah dari Server ke Client

Ada 4 cara utama data server sampai ke browser:

#### Cara 1: Langsung di Blade (Inline)

```php
{{-- products.blade.php --}}
<script>
    // PHP di-render dulu di server → jadi teks JS di browser
    const products = @json($products);
    // Hasil: const products = [{"id":1,"name":"Sepatu",...}];
</script>
```

**Keuntungan**: Cepat, langsung tersedia saat JS jalan.
**Kerugian**: Data ada di source HTML (bisa dilihat user).

#### Cara 2: Fetch/Ajax (Request JS ke Server)

```javascript
// Di browser, JS minta data ke server
fetch('/api/products')
  .then(res => res.json())
  .then(data => {
    console.log('Data dari server:', data);
  });
```

**Keuntungan**: Data diminta saat dibutuhkan, tidak memberatkan initial load.
**Kerugian**: Ada delay, perlu handle loading/error state.

#### Cara 3: Form Submit (Traditional)

```html
<!-- Form dikirim → server proses → redirect ke halaman baru -->
<form action="/checkout" method="POST">
  @csrf
  <input name="address" value="...">
  <button type="submit">Checkout</button>
</form>
```

**Keuntungan**: Sederhana, reliable.
**Kerugian**: Reload halaman penuh.

#### Cara 4: Livewire/Alpine + Fetch (Modern)

```html
<div x-data="{
    search: '',
    results: [],
    async searchProducts() {
        let res = await fetch(`/search?q=${this.search}`);
        this.results = await res.json();
    }
}">
    <input x-model="search" @input.debounce="searchProducts">
    <template x-for="item in results" :key="item.id">
        <div x-text="item.name"></div>
    </template>
</div>
```

**Keuntungan**: Interaktif seperti SPA, tanpa framework JS berat.
**Kerugian**: Butuh pemahaman async JS.

### 6.2 Diagram Alur Data End-to-End

```
┌─────────────────────────────────────────────────────────────┐
│                   BROWSER (Client)                          │
│                                                             │
│  ┌────────────────┐    ┌─────────────────────┐              │
│  │ HTML (struktur)│──▶ │                    │              │
│  │ CSS (tampilan) │    │   Browser Engine    │              │
│  │ JS (interaksi) │──▶│   (Render + Execute)│              │
│  └────────────────┘    └────────┬────────────┘              │
│                                 │                           │
│                                 ▼                           │
│                    ┌──────────────────────┐                 │
│                    │  Alpine.js Reactivity│                 │
│                    │  - x-data, x-text    │                 │
│                    │  - @click, x-model   │                 │
│                    │  - fetch() ke server │                 │
│                    └──────────────────────┘                 │
└─────────────────────────────────────────────────────────────┘
          ▲                           │
          │ Request                   │ HTML/CSS/JS
          │ (GET/POST)                │ + Data
          │                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   SERVER (Laravel)                          │
│                                                             │
│  ┌──────────┐     ┌───────────┐    ┌──────────────────┐     │
│  │ Router   │───▶│ Controller│───▶│      View        │     │
│  │ web.php  │     │           │    │   (Blade Render) │     │
│  └──────────┘     └──┬────────┘    └──────────────────┘     │
│                      │                                      │
│                      ▼                                      │
│              ┌────────────────┐                             │
│              │    Service      │                            │
│              │    + Model      │                            │
│              └───────┬────────┘                             │
│                      │                                      │
│                      ▼                                      │
│              ┌─────────────────┐                            │
│              │   Database      │                            │
│              │   (MySQL/SQLite)│                            │
│              └─────────────────┘                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Praktik Debugging Sehari-hari

### Skenario 1: Halaman Blank / 500 Error

```
Langkah-langkah:
1. Buka Browser Console (F12) → lihat Network tab
   - Status code? 500 = error server
   - Response apa yang dikirim? (klik tab Response)

2. Cek Laravel log:
   tail -f storage/logs/laravel.log

3. Buka Telescope:
   http://olshop-koneksi.test/telescope → Exceptions

4. Kalau APP_DEBUG=true: halaman Whoops sudah kasih tahu
   - File mana error
   - Baris berapa
   - Stack trace lengkap
```

### Skenario 2: Data Tidak Muncul di Halaman

```
Langkah-langkah:
1. Cek Elements tab → apakah HTML-nya ada?
   - Ada HTML tapi kosong → masalah data dari server
   - Tidak ada HTML → Blade conditional (if/foreach) mungkin bermasalah

2. Cek Network tab:
   - Apakah request ke server sukses?
   - Response body apa?

3. Debug langsung di Controller:
   dd($products); // Lihat isi data dari DB

4. Debug di Blade:
   @dd($products) // Sama seperti dd() tapi di template
```

### Skenario 3: JavaScript Tidak Jalan

```
Langkah-langkah:
1. Buka Console tab (F12):
   - Ada error merah? Baca pesannya
   - "Uncaught ReferenceError: xx is not defined"
     → Variabel belum didefinisikan
   - "Cannot read property of undefined"
     → Data dari server tidak sesuai ekspektasi

2. Cek Network tab:
   - Apakah file JS ter-load? (Status 200?)
   - Apakah data dari fetch() sukses?

3. Cek Elements tab:
   - Apakah elemen dengan x-data ada?
   - Klik elemen, ketik di Console: $0.__x.$data

4. Tambahkan console.log() sementara:
   console.log('Step 1: mulai', data);
   console.log('Step 2: setelah fetch', result);
```

### Skenario 4: CSS Rusak / Layout Berantakan

```
Langkah-langkah:
1. Buka Elements tab → inspect elemen yang rusak
2. Lihat computed styles → apakah CSS yang diharapkan teraplikasi?
3. Cek apakah Tailwind class benar:
   - "grid grid-cols-3" → benar?
   - Class tidak dikenal? Ejaan salah?
4. Cek Network tab → apakah file CSS ter-load?
5. Refresh dengan cache cleared (Ctrl+F5)
```

### Skenario 5: Form Submit Tidak Bekerja

```
Langkah-langkah:
1. Cek Network tab → lihat request form submit
   - Status code? 302 (redirect) = sukses
   - Status code? 419 = CSRF token expired
   - Status code? 422 = Validasi gagal

2. Kalau 419:
   - Pastikan @csrf ada di form
   - Session expired? Refresh halaman

3. Kalau 422:
   - Lihat response body → ada pesan validasi
   - Cek apakah field nama sesuai dengan controller request

4. Kalau 500:
   - Cek Laravel log
```

### Skenario 6: Midtrans Payment Error

```
Langkah-langkah:
1. Cek Browser Console (F12):
   - Apakah Midtrans Snap.js ter-load?
   - Apakah ada error JavaScript?

2. Cek Network tab:
   - Apakah request ke endpoint checkout sukses?
   - Apakah response berisi snap_token?

3. Cek Laravel log:
   - MidtransService error?
   - Koneksi ke Midtrans gagal?

4. Cek Telescope → tab Exceptions dan Logs

5. Pastikan mode sandbox:
   MIDTRANS_IS_PRODUCTION=false di .env
```

### Command-Line Debugging Toolkit

```bash
# Lihat log Laravel secara real-time
tail -f storage/logs/laravel.log

# Lihat semua route
php artisan route:list

# Lihat status migrasi
php artisan migrate:status

# Clear cache (sering bikin masalah aneh)
php artisan optimize:clear

# Jalankan queue worker untuk job
php artisan queue:listen

# Cek environment
php artisan about

# Telescope (dashboard visual)
# Buka: http://olshop-koneksi.test/telescope
```

### Debugging Flowchart — Kesimpulan

```
ADA ERROR?
    │
    ├── Halaman error Whoops (APP_DEBUG=true)
    │   │
    │   └── Baca pesan error + file + line number → Fix
    │
    ├── Halaman kosong / 500
    │   │
    │   ├── Cek storage/logs/laravel.log
    │   ├── Cek Telescope (Exceptions tab)
    │   └── atau: dd() di controller
    │
    ├── Data tidak muncul / salah
    │   │
    │   ├── Cek Network tab → lihat response
    │   ├── dd($data) di controller
    │   └── @dd($data) di blade
    │
    ├── JavaScript error
    │   │
    │   ├── Buka Console (F12)
    │   ├── Cek Network → file JS ter-load?
    │   └── console.log() untuk tracing
    │
    └── CSS / layout rusak
        │
        ├── Cek Elements tab → computed styles
        ├── Cek class Tailwind (ejaan benar?)
        └── Clear cache browser (Ctrl+F5)
```

---

## 8. Glosarium

| Istilah | Arti |
|---------|------|
| **SSR** | Server-Side Rendering — HTML dibuat di server |
| **CSR** | Client-Side Rendering — HTML dibuat di browser oleh JS |
| **SPA** | Single Page Application — aplikasi di satu halaman (React, Vue) |
| **Blade** | Template engine Laravel (file `.blade.php`) |
| **Alpine.js** | Library JavaScript ringan untuk interaktivitas (seperti Vue mini) |
| **Middleware** | Filter yang dijalankan SEBELUM request sampai ke controller |
| **Eloquent** | ORM Laravel — cara interaksi dengan database via PHP |
| **Telescope** | Debugging assistant Laravel (dashboard real-time) |
| **Whoops** | Halaman error visual (dipakai Laravel saat debug mode) |
| **Artisan** | Command line tool Laravel (`php artisan ...`) |
| **Vite** | Build tool frontend (compiler CSS/JS, hot reload) |
| **HMR** | Hot Module Replacement — ganti CSS/JS tanpa reload halaman |
| **CSRF** | Cross-Site Request Forgery — token keamanan form Laravel |
| **dd()** | `dump()` + `die()` — debug termudah di Laravel |
| **Middleware** | Filter/penjaga sebelum request masuk ke controller |
| **Service** | Class khusus untuk logic bisnis (supaya controller tidak gemuk) |

---

## Penutup

Web development dengan Laravel memang terasa "campur aduk" karena kamu harus mengelola **PHP** (server), **Blade** (template), **HTML/CSS** (struktur), dan **JavaScript/Alpine.js** (client) secara bersamaan. Ini bukan kekurangan — ini adalah **pendekatan traditional SSR** yang sudah ada puluhan tahun.

Kunci untuk tidak bingung:
1. **Selalu tanya: "Ini jalan di server atau di browser?"**
2. Server error → cek `storage/logs/laravel.log` atau Telescope
3. Client error → buka F12 → Console / Network
4. Data flow dari server ke client → via blade `@json()` atau fetch API
5. Jangan takut pakai `dd()` — itu cara termudah debug di Laravel

Seiring waktu, kamu akan hafal pola ini dan tidak perlu berpikir keras untuk menentukan mana yang terjadi di mana.

---

*Dokumen ini dibuat untuk keperluan belajar tim Koneksi Store. Disimpan di .openkb sebagai referensi pengetahuan terbuka.*
