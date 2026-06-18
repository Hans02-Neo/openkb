# 05-01: Cara Kerja Web — Dari Browser ke Server dan Kembali

> **Fase**: 5 — Web Fundamental  
> **Prasyarat**: FASE 2 (PHP Fundamental)  
> **Waktu baca**: 70-85 menit  
> **Kata kunci**: browser, server, HTTP, request, response, DNS, TCP/IP, URL, client-server, hosting, Laragon

---

## 📋 Ringkasan

Kamu sudah menulis kode PHP dan Laravel. Tapi pernahkah kamu bertanya: **apa yang terjadi persis** saat kamu mengetik `http://olshop-koneksi.test` di browser dan menekan Enter?

Dokumen ini menjawab pertanyaan itu — dari tombol Enter ditekan sampai halaman tampil. Ini adalah fondasi yang membuat semua web development masuk akal.

**Target pemahaman:**
- Kamu paham arsitektur client-server
- Kamu bisa menjelaskan perjalanan request HTTP
- Kamu paham peran DNS, TCP/IP, port, hosting
- Kamu paham bagaimana Laragon menyediakan server lokal

---

## 1. Arsitektur Client-Server

### 1.1 Dua Pihak

Web bekerja dengan **dua pihak**:

```
┌─────────────────────┐          ┌─────────────────────┐
│     CLIENT          │          │       SERVER        │
│  (Browser)          │  HTTP    │  (Apache/Nginx)     │
│                     │ ◄──────► │                     │
│  Meminta data       │  REQUEST │  Menyediakan data   │
│  Menampilkan hasil  │  ──────► │  Memproses logika   │
│                     │  RESPONSE│  Akses database     │
│  Chrome, Firefox    │  ◄────── │  Laragon, Live      │
└─────────────────────┘          └─────────────────────┘
```

**Client** (browser): Meminta halaman. Menampilkan HTML, CSS, JavaScript.
**Server** (Apache/Nginx): Menyimpan file, menjalankan kode PHP, mengakses database, mengirim response.

### 1.2 Analogi Restoran

| Web | Restoran |
|-----|---------|
| Browser (client) | Pelanggan |
| Server (Apache) | Koki |
| Database | Kulkas/penyimpanan |
| URL (alamat web) | Alamat restoran |
| HTTP Request | Pesanan ("Nasi goreng!") |
| HTTP Response | Hidangan |

Kamu (browser) datang ke restoran (server), memesan (request), koki memasak (PHP memproses), dan mengirimkan (response) hidangan (halaman HTML).

### 1.3 Laragon sebagai Server Lokal

Laragon menyediakan **server lengkap di laptop kamu**:

```
Laragon
├── Apache (Web Server)     → port 80 (HTTP)
├── PHP (Bahasa Pemrograman)→ versi 8.3
├── MySQL (Database)        → port 3306
└── Node.js / npm           → untuk asset frontend
```

Tanpa Laragon, kamu perlu install Apache, PHP, MySQL satu per satu. Laragon menggabungkan semuanya dan menyediakan:
- **Virtual Host** → `olshop-koneksi.test` otomatis diarahkan ke folder project
- **Auto SSL** → HTTPS dengan `olshop-koneksi.test`
- **Service Management** → Start/Stop Apache, MySQL dari tray icon

---

## 2. Perjalanan Request — Dari Enter ke Halaman Tampil

Inilah urutan lengkap saat kamu mengetik `http://olshop-koneksi.test`:

### Step 1: Kamu mengetik URL

```
URL: http://olshop-koneksi.test/shop?category=laptop
      │      │                    │     │
      │      │                    │     └── Query string (parameter)
      │      │                    └── Path (halaman)
      │      └── Domain (nama server)
      └── Protocol (HTTP)
```

### Step 2: Browser mencari IP (DNS Lookup)

Browser tidak paham `olshop-koneksi.test`. Browser paham **IP Address** (misal `127.0.0.1`).

DNS (Domain Name System) adalah **buku telepon internet** — menerjemahkan domain ke IP.

```

Browser: "Siapa punya domain olshop-koneksi.test?"
                                                      ┌─────────────┐
DNS: "Itu ada di 127.0.0.1 (localhost)" ─────────────►│127.0.0.1:80 │
                                                      │ (Apache)    │
                                                      └─────────────┘
```

Di Laragon, domain `.test` langsung diarahkan ke `127.0.0.1` (localhost) via **hosts file**.

### Step 2b: Kenapa Ada Port?

Server bisa menjalankan banyak service. Port adalah **nomor pintu**:

```
127.0.0.1:80   → Apache (HTTP)
127.0.0.1:3306 → MySQL
127.0.0.1:443  → Apache (HTTPS)
```

URL `http://olshop-koneksi.test` sebenarnya `http://olshop-koneksi.test:80` — port 80 adalah default HTTP.

### Step 3: Browser membuat koneksi TCP

Browser membuka koneksi ke server (`127.0.0.1:80`) menggunakan protokol **TCP** — protokol yang memastikan data sampai dengan utuh.

Ini disebut **three-way handshake**:
```
Browser ───SYN───► Server
Browser ◄──SYN-ACK── Server
Browser ───ACK───► Server  ← Koneksi terbentuk!
```

### Step 4: Browser mengirim HTTP Request

Setelah koneksi terbentuk, browser mengirim pesan HTTP:

```
GET /shop?category=laptop HTTP/1.1
Host: olshop-koneksi.test
User-Agent: Mozilla/5.0 (Windows NT 10.0)
Accept: text/html,application/xhtml+xml
Cookie: cart_session=abc123; XSRF-TOKEN=xyz
```

Baris pertama (request line):
- **GET** → metode HTTP (minta data)
- `/shop?category=laptop` → path + query
- `HTTP/1.1` → versi protokol

Header:
- `Host` → domain (server bisa host banyak website)
- `User-Agent` → identitas browser
- `Accept` → tipe response yang diterima
- `Cookie` → data session yang disimpan browser

### Step 5: Apache menerima request

Apache (web server) melihat request dan menentukan:
1. Domain mana? → `olshop-koneksi.test` → folder `C:\laragon\www\olshop-koneksi`
2. File mana? → `/shop` → route Laravel

### Step 5b: Apache vs PHP

Apache tidak bisa langsung menjalankan PHP. Apache menggunakan **mod_php** atau **FPM**:

```
                  ┌─────────────────────┐
Apache ───request──►   PHP Interpreter   │
        ◄──response─│  (mod_php / FPM)   │
                  └─────────────────────┘
```

PHP membaca file `index.php` → mem-bootstrap Laravel → menjalankan route → controller → database → menghasilkan HTML.

### Step 6: Laravel memproses

```
Request masuk → bootstrap → middleware → route → controller → response
```

1. **Bootstrap** → Load semua file Laravel, koneksi database
2. **Middleware** → Cek auth, CSRF, throttle
3. **Route** → Cocokkan `/shop?category=laptop` → `ShopController::index`
4. **Controller** → Query produk dari database
5. **Response** → Render view `shop/index.blade.php` → HTML

### Step 7: Server mengirim HTTP Response

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Set-Cookie: cart_session=abc123; path=/

<!DOCTYPE html>
<html>
<head>
    <title>Koneksi Store</title>
</head>
<body>
    <h1>Shop</h1>
    ...
</body>
</html>
```

### Step 8: Browser menerima dan merender

1. **Parse HTML** → Membangun DOM (Document Object Model)
2. **CSS** → Render Tree (styling)
3. **JavaScript** → Interaktivitas
4. **Layout** → Posisi setiap elemen
5. **Paint** → Tampilkan ke layar

### Visualisasi Lengkap

```
┌──────┐   DNS    ┌──────────┐  TCP   ┌──────────┐  PHP   ┌──────────┐
│USER  │──────►   │BROWSER   │──────► │APACHE    │──────►│LARAVEL   │
│Enter │          │Parse URL │        │Web Server│       │index.php │
└──────┘          │DNS lookup│        │80/443    │       │Route     │
                  │TCP conn  │        │Send file │       │Controller│
                  │Send req  │        │or PHP    │       │View      │
                  │          │◄───────│          │◄───────│Database  │
                  │Parse HTML│  HTTP  │Response  │  HTML │          │
                  │Render    │  200   │          │       └──────────┘
                  │Display   │        └──────────┘
                  └──────────┘
```

---

## 3. URL Anatomy

```php
// Setiap URL punya struktur yang sama:

$url = 'https://budi:pass123@olshop-koneksi.test:443/shop?category=laptop#reviews';
//       │      │    │       │                  │   │     │              │
//       │      │    │       │                  │   │     │              └── Fragment
//       │      │    │       │                  │   │     └── Query String
//       │      │    │       │                  │   └── Path
//       │      │    │       │                  └── Port
//       │      │    │       └── Host (Domain)
//       │      │    └── Userinfo (Auth)
//       │      └── Scheme (Protocol)
//       └── (https = aman, http = biasa)
```

| Bagian | Contoh | Guna |
|--------|--------|------|
| **Scheme** | `https` | Protokol — `http` (tanpa enkripsi), `https` (dengan enkripsi SSL/TLS) |
| **Userinfo** | `budi:pass123` | Jarang dipakai — kredensial di URL |
| **Host** | `olshop-koneksi.test` | Domain server |
| **Port** | `:443` | Nomor port (80 = HTTP, 443 = HTTPS) |
| **Path** | `/shop` | Resource di server |
| **Query** | `?category=laptop` | Parameter — key=value dipisah `&` |
| **Fragment** | `#reviews` | Tidak dikirim ke server — untuk navigasi halaman |

### URL di Codebase

Di Laravel, URL dihasilkan oleh **route**:

```php
// web.php — URL /shop dengan query string
Route::get('/shop', [ShopController::class, 'index'])->name('shop.index');

// Di blade:
// <a href="{{ route('shop.index', ['category' => 'laptop']) }}">
// Hasil: http://olshop-koneksi.test/shop?category=laptop
```

---

## 4. HTTP — Bahasa Client dan Server

### 4.1 Request — Apa yang Diminta

Setiap HTTP Request punya:

```
METHOD   PATH          PROTOCOL
 │        │              │
GET      /shop          HTTP/1.1
POST     /checkout      HTTP/1.1
PUT      /cart/1        HTTP/1.1
DELETE   /cart/1        HTTP/1.1
```

| Method | Guna | Di Codebase |
|--------|------|-------------|
| **GET** | Minta data (baca) | `ShopController@index`, `CartController@index` |
| **POST** | Kirim data baru | `CheckoutController@process` |
| **PUT/PATCH** | Update data | `CartController@update` |
| **DELETE** | Hapus data | `CartController@destroy` |

### 4.2 Response — Apa yang Diterima

```
PROTOCOL  STATUS CODE  STATUS MESSAGE
 │           │             │
HTTP/1.1    200           OK
HTTP/1.1    404           Not Found
HTTP/1.1    500           Internal Server Error
```

### 4.3 Status Code — Kelompok

| Kode | Kelompok | Arti | Contoh |
|------|---------|------|--------|
| `1xx` | Informational | Sedang diproses | 101 Switching Protocols |
| `2xx` | Success | Berhasil | 200 OK, 201 Created |
| `3xx` | Redirection | Pindah alamat | 301 Moved, 302 Found |
| `4xx` | Client Error | Salah dari client | 404 Not Found, 403 Forbidden |
| `5xx` | Server Error | Error di server | 500 Internal Server Error |

### 4.4 Status Code di Codebase

```php
// 200 OK — default Laravel
return response()->json($data); // 200

// 201 Created — setelah sukses buat data
return response()->json($order, 201);

// 302 Found — redirect
return redirect()->route('orders.index');

// 404 Not Found — otomatis dari Model::findOrFail
$order = Order::findOrFail($id); // 404 jika tidak ketemu

// 403 Forbidden — otorisasi gagal
abort(403, 'Unauthorized action.');

// 419 Session Expired — CSRF token mismatch

// 500 Server Error — unexpected exception
```

---

## 5. HTTPS — Lapisan Enkripsi

### 5.1 HTTP vs HTTPS

```
HTTP:  Browser ──plain text──► Server
       "password=rahasia123"  → Bisa dibaca siapa pun di jaringan!

HTTPS: Browser ──encrypted──► Server
       "8a3f7b2e..."          → Tidak bisa dibaca tanpa kunci
```

HTTPS = HTTP + **SSL/TLS** — data dienkripsi sebelum dikirim.

### 5.2 Cara Kerja Sederhana

1. Browser minta koneksi HTTPS ke server
2. Server kirim **SSL Certificate** (kunci publik)
3. Browser verifikasi certificate (dari Certificate Authority)
4. Browser buat **session key** acak, enkripsi dengan kunci publik
5. Server buka dengan **kunci privat** — mulai sesi aman

### 5.3 Laragon Auto SSL

Laragon menyediakan HTTPS otomatis via **ngrok** atau **mkcert**:

```
http://olshop-koneksi.test   →  Apache port 80
https://olshop-koneksi.test  →  Apache port 443 (dengan sertifikat lokal)
```

---

## 6. Cookies dan Session

### 6.1 HTTP Adalah Stateless

HTTP tidak punya memori. Dua request berturut-turut tidak saling kenal.

```
Request 1: GET /product/1
Server: "Ini produk A" (tidak tahu siapa kamu)

Request 2: GET /cart
Server: "Kamu belum login" (lupa request 1!)
```

### 6.2 Solusi: Cookie + Session

**Cookie**: Data kecil (max 4KB) disimpan di browser, dikirim setiap request.

**Session**: Data disimpan di server, direferensi oleh ID di cookie.

```
Request 1: POST /login
Server: 
  1. Cek username/password → OK
  2. Buat session: session_id = abc123, data = {user_id: 1}
  3. Kirim cookie: Set-Cookie: session_id=abc123

Request 2: GET /cart
Browser kirim: Cookie: session_id=abc123
Server:
  1. Baca cookie → session_id = abc123
  2. Ambil data session dari storage (file/database/redis)
  3. Tahu user_id = 1 → ambil cart user 1
```

### 6.3 Session di Laravel

```php
// Session disimpan di file (storage/framework/sessions)
// Untuk produksi, biasanya pindah ke database atau Redis

// Menyimpan ke session
session(['cart' => $items]);
request()->session()->put('cart', $items);

// Membaca dari session
$cart = session('cart', []); // default [] jika tidak ada
$cart = request()->session()->get('cart', []);
```

### 6.4 Session di CartService

```php
// CartService menggunakan session untuk cart:
$cart = session('cart', []); // ambil cart dari session
session(['cart' => $updatedCart]); // simpan cart ke session
```

---

## 7. Hosting — Dari Lokal ke Publik

### 7.1 Development vs Production

| Aspek | Development (Lokal) | Production (Live) |
|-------|-------------------|-------------------|
| URL | `http://olshop-koneksi.test` | `https://koneksi-store.com` |
| Server | Laragon (Apache) | Server sungguhan (VPS, shared hosting) |
| Database | MySQL lokal | Managed database (RDS, DigitalOcean) |
| HTTPS | Self-signed (Laragon) | Let's Encrypt / Cloudflare |
| File storage | `storage/app/public` | S3, Cloud Storage |
| Email | Log/trap | SMTP sungguhan (Mailgun, SendGrid) |

### 7.2 Jenis Hosting untuk Laravel

1. **Shared Hosting** — Murah, satu server banyak website. PHP + MySQL. Harus kompatibel Laravel.
2. **VPS** — Lebih mahal, kendali penuh. Kamu install sendiri Apache/Nginx + PHP + MySQL. Contoh: DigitalOcean, Linode, Vultr.
3. **Platform-as-a-Service** — Pasang kode, server diurus provider. Contoh: Laravel Forge + DigitalOcean, Vapor (serverless), Railway.

### 7.3 Deployment Sederhana

```
Kode di lokal
    │
    ├── git push origin main
    │
    ▼
GitHub / GitLab
    │
    ├── Pull / Deploy hook
    │
    ▼
Server (VPS / Forge)
    │
    ├── composer install --optimize
    ├── php artisan migrate
    ├── php artisan config:cache
    └── php artisan optimize
```

---

## 8. Web di Codebase

### 8.1 Seluruh Perjalanan Request

Mari trace satu request lengkap di `olshop-koneksi.test`:

```
Browser: http://olshop-koneksi.test/shop?category=laptop

1. DNS → 127.0.0.1:80
2. Apache terima → folder C:\laragon\www\olshop-koneksi
3. public/index.php — bootstrap Laravel
4. public/.htaccess — rewrite rule ke index.php
5. Laravel bootstrap — load config, service providers
6. Middleware — EncryptCookies, StartSession, ShareErrorsFromSession
7. Route — GET /shop → ShopController@index
8. Controller — query produk, filter, pagination
9. View — shop/index.blade.php → HTML
10. Response — HTTP 200, Content-Type: text/html
11. Browser — parse HTML, load CSS/JS, render
```

### 8.2 Alat untuk Melihat Request

**Laravel Telescope** (tersedia di codebase):

```
http://olshop-koneksi.test/telescope/requests
```

Telescope mencatat:
- Semua request masuk (method, path, status)
- Query database yang dijalankan
- Exception yang terjadi
- Mail yang dikirim
- Job queue

```php
// Config: config/telescope.php
// Diaktifkan untuk development — jangan di production!
```

### 8.3 Browser DevTools

Tekan **F12** atau **Ctrl+Shift+I** di browser:

| Tab | Guna |
|-----|------|
| **Network** | Lihat semua request, status code, waktu loading |
| **Elements** | Lihat HTML hasil render |
| **Console** | Log JavaScript, error |
| **Application** | Cookies, session storage, cache |

**Latihan:** Buka `olshop-koneksi.test`, F12 → Network, reload halaman. Lihat request pertama (document) — status 200, size, waktu.

---

## 9. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ PERJALANAN REQUEST WEB                                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  User: mengetik URL                                                 │
│    │                                                                │
│    ▼                                                                │
│  DNS: domain → IP address                                          │
│    │                                                                │
│    ▼                                                                │
│  TCP: three-way handshake (SYN, SYN-ACK, ACK)                      │
│    │                                                                │
│    ▼                                                                │
│  HTTP REQUEST: GET /shop HTTP/1.1                                   │
│    │                                                                │
│    ▼                                                                │
│  WEB SERVER: Apache (Laragon)                                       │
│    │                                                                │
│    ▼                                                                │
│  PHP: public/index.php → Laravel bootstrap                          │
│    │                                                                │
│    ▼                                                                │
│  ROUTING: /shop → ShopController@index                              │
│    │                                                                │
│    ▼                                                                │
│  CONTROLLER: query DB, filter, paginate                             │
│    │                                                                │
│    ▼                                                                │
│  VIEW: shop/index.blade.php → HTML                                  │
│    │                                                                │
│    ▼                                                                │
│  HTTP RESPONSE: 200 OK, Content-Type: text/html                     │
│    │                                                                │
│    ▼                                                                │
│  BROWSER: parse HTML → CSS → JS → render                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Trace request manual.** Buka `olshop-koneksi.test`, F12 → Network. Reload. Klik request pertama. Lihat:
   - Request URL
   - Request Method
   - Status Code
   - Response Headers

2. **Cek headers.** Buka ProductController atau ShopController. Di browser Network tab, cari response header `Content-Type` dan `Set-Cookie`.

3. **Cek cookies.** Buka F12 → Application → Cookies → `olshop-koneksi.test`. Cookie apa saja yang ada?

4. **404 test.** Buka `http://olshop-koneksi.test/halaman-tidak-ada`. Status code berapa? Halaman error apa yang tampil?

5. **Identifikasi port.** Buka Laragon tray icon → Menu → Apache → httpd.conf. Cari `Listen 80` dan `Listen 443`. Port apa yang digunakan?

---

## 🔗 Referensi

- [MDN: How the Web Works](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/How_the_Web_works)
- [MDN: HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [Laravel Docs: Request Lifecycle](https://laravel.com/docs/11.x/lifecycle)
- [Laragon Docs](https://laragon.org/docs/)
- Codebase: `public/index.php` — entry point
- Codebase: `public/.htaccess` — URL rewrite
- Codebase: `app/Http/Kernel.php` — middleware stack
