# Laravel Telescope — Panduan Lengkap

## Daftar Isi

1. [Apa Itu Telescope?](#1-apa-itu-telescope)
2. [Instalasi & Konfigurasi](#2-instalasi--konfigurasi)
3. [Cara Mengakses Telescope](#3-cara-mengakses-telescope)
4. [Dashboard — Penjelasan Setiap Tab](#4-dashboard--penjelasan-setiap-tab)
5. [Tab Requests — Melacak Semua HTTP Request](#5-tab-requests)
6. [Tab Commands — Artisan Commands](#6-tab-commands)
7. [Tab Schedule — Cron Job Terjadwal](#7-tab-schedule)
8. [Tab Jobs — Queue Jobs](#8-tab-jobs)
9. [Tab Exceptions — Semua Error & Exception](#9-tab-exceptions)
10. [Tab Logs — Semua Log](#10-tab-logs)
11. [Tab Dumps — Output dd() & dump()](#11-tab-dumps)
12. [Tab Queries — Semua Query SQL](#12-tab-queries)
13. [Tab Mail — Email yang Dikirim](#13-tab-mail)
14. [Tab Notifications — Notifikasi](#14-tab-notifications)
15. [Tab Cache — Cache Hit & Miss](#15-tab-cache)
16. [Tab Events — Event yang Dipicu](#16-tab-events)
17. [Tab Gates — Policy & Gate Checks](#17-tab-gates)
18. [Filtering & Pencarian Data](#18-filtering--pencarian-data)
19. [Watching (Watches) — Memonitor Tertentu](#19-watching-watches)
20. [Konfigurasi Lanjutan](#20-konfigurasi-lanjutan)
21. [Telescope di Production](#21-telescope-di-production)
22. [Studi Kasus — Debugging Real World](#22-studi-kasus--debugging-real-world)
23. [Tips & Trik](#23-tips--trik)
24. [Perintah Artisan Telescope](#24-perintah-artisan-telescope)

---

## 1. Apa Itu Telescope?

**Laravel Telescope** adalah debugging assistant yang berjalan di dashboard web. Telescope mencatat **semua yang terjadi** di aplikasi Laravel kamu secara real-time — request, query SQL, error, log, email, notifikasi, queue jobs, dan banyak lagi.

Bayangkan Telescope sebagai **"black box" pesawat** untuk aplikasi web kamu. Setiap ada masalah, kamu bisa "memutar ulang" apa yang terjadi persis sebelum error muncul.

### Apa yang Bisa Telescope Lakukan?

| Fitur | Manfaat |
|-------|---------|
| Merekam semua HTTP request | Lihat method, URL, status code, durasi, headers, input |
| Merekam semua query SQL | Lihat SQL mentah, binding, durasi — temukan query lambat |
| Menangkap semua exception | Lihat stack trace lengkap tanpa perlu buka log file |
| Merekam output dd()/dump() | Lihat dump di dashboard, bukan cuma di terminal |
| Merekam email & notifikasi | Lihat isi email yang "dikirim" tanpa benar-benar kirim |
| Merekam queue jobs | Lihat job yang berhasil, gagal, atau sedang antre |
| Merekam cache operation | Lihat hit/miss cache |
| Merekam event & gate checks | Lihat policy authorization |

### Telescope vs Alternatif

| Alat | Kelebihan | Kekurangan |
|------|-----------|------------|
| **Telescope** | Dashboard visual, real-time, semua tercatat | Boros storage untuk production |
| **Laravel Log** | Sederhana, universal | Harus `tail -f`, format mentah teks |
| **Debugbar** (barryvdh) | Toolbar di browser, ringan | Tidak rekam history, hanya request saat ini |
| **Clockwork** | Mirip debugbar, di Chrome DevTools | Extension browser diperlukan |
| **dd()/dump()** | Sederhana dan langsung | Hentiin eksekusi, berantakan |

---

## 2. Instalasi & Konfigurasi

### Cek Apakah Telescope Sudah Terinstall

Proyek ini sudah memiliki Telescope. Cek:

```bash
composer show laravel/telescope
```

Atau lihat di `composer.json`:

```json
"require-dev": {
    "laravel/telescope": "^5.20"
}
```

### Jika Belum Terinstall (Referensi)

```bash
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

### File Penting Telescope

| File | Fungsi |
|------|--------|
| `config/telescope.php` | Konfigurasi utama Telescope |
| `app/Providers/TelescopeServiceProvider.php` | Service provider — atur apa yang dicatat |
| `database/migrations/*_create_telescope_entries_table.php` | Migrasi tabel penyimpanan |

### Konfigurasi Dasar

Lihat `config/telescope.php`:

```php
return [
    'domain' => env('TELESCOPE_DOMAIN'),       // Domain khusus (opsional)
    'path' => env('TELESCOPE_PATH', 'telescope'), // URL path
    'driver' => env('TELESCOPE_DRIVER', 'database'), // Penyimpanan

    'storage' => [
        'database' => [
            'connection' => env('DB_CONNECTION', 'mysql'),
            'chunk' => 100, // Batch insert
        ],
    ],

    // Filter — apa yang TIDAK dicatat
    'filter' => [
        App\Models\User::class => [
            'password',
            'remember_token',
        ],
    ],

    // Watchers — apa yang DICATAT
    'watchers' => [
        ...
    ],
];
```

---

## 3. Cara Mengakses Telescope

### Akses via Browser

```
http://olshop-koneksi.test/telescope
```

Jika pertama kali akses, kamu akan melihat dashboard kosong seperti ini:

```
┌─────────────────────────────────────────────────────┐
│  🔭 Telescope                                        │
│                                                      │
│  [Requests] [Commands] [Schedule] [Jobs] ...         │
│                                                      │
│  No entries yet.                                     │
│  Make a request to see it here.                      │
└─────────────────────────────────────────────────────┘
```

### Navigasi Dasar

| Elemen | Fungsi |
|--------|--------|
| **Navbar tabs** | Pindah antar kategori (Requests, Exceptions, dll) |
| **Search box** | Cari berdasarkan keyword |
| **Filter tag** | Filter berdasarkan tag (watch) |
| **Tombol refresh** | Muat ulang data (Telescope tidak auto-refresh) |

---

## 4. Dashboard — Penjelasan Setiap Tab

Telescope punya banyak tab. Ini ringkasan **semua tab dan fungsinya**:

| No | Tab | Fungsi | Paling Berguna Untuk |
|----|-----|--------|---------------------|
| 1 | **Requests** | Semua HTTP request masuk | Debugging API, form submit, AJAX |
| 2 | **Commands** | Semua artisan command | Debugging custom command |
| 3 | **Schedule** | Cron job terjadwal | Debugging scheduled tasks |
| 4 | **Jobs** | Queue jobs | Debugging job gagal |
| 5 | **Exceptions** | Semua error/exception | **Mencari error** |
| 6 | **Logs** | Semua log | **Mencari error + informasi** |
| 7 | **Dumps** | Output dd()/dump() | Tracing data di request |
| 8 | **Queries** | Semua query SQL | **Optimasi database** |
| 9 | **Mail** | Email yang dikirim | Debugging email |
| 10 | **Notifications** | Notifikasi | Debugging notifikasi |
| 11 | **Cache** | Cache hit/miss | Debugging cache |
| 12 | **Events** | Event yang dipicu | Debugging event/listener |
| 13 | **Gates** | Policy & Gate authorization | Debugging otorisasi |

---

## 5. Tab Requests

### Apa yang Direkam

Setiap request HTTP — baik halaman web, AJAX, form submit, atau API — dicatat detailnya.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Requests                                [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ GET    /shop              200  120ms  │ 2 queries │ 13:45:12 │
│ POST   /cart/add          302   45ms  │ 1 query   │ 13:45:10 │
│ GET    /admin/products    200  350ms  │ 8 queries │ 13:44:50 │
│ GET    /api/products      500   12ms  │ 0 queries │ 13:44:30 │ ← ERROR
│ GET    /telescope/requests 200  200ms  │ 5 queries │ 13:44:00 │
└──────────────────────────────────────────────────────────────┘
```

Kolom:

| Kolom | Arti |
|-------|------|
| **Method** | GET, POST, PUT, DELETE, PATCH |
| **Path** | URL relative |
| **Status** | 200 (OK), 302 (Redirect), 404, 500 (Error) |
| **Duration** | Waktu eksekusi total (ms) — **perhatikan yang lambat** |
| **Queries** | Jumlah query SQL yang dijalankan |
| **Time** | Jam terjadinya request |

### Detail Request (Klik Salah Satu)

Saat kamu klik satu request, kamu akan melihat halaman detail:

```
┌─────────────────────────────────────────────────────────────┐
│ 🔙 Back to Requests                                         │
│                                                             │
│  GET /shop                     200 OK   120ms               │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│                                                             │
│ ┌─── Headers ───────────────────────────────────────────┐  │
│ │ Accept: text/html                                     │  │
│ │ Host: olshop-koneksi.test                             │  │
│ │ User-Agent: Mozilla/5.0 ...                          │  │
│ │ Cookie: XSRF-TOKEN=...; laravel_session=...          │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── Session ──────────────────────────────────────────┐  │
│ │ _token: abc123                                       │  │
│ │ _flash: {...}                                        │  │
│ │ cart: [{"product_id":1,"qty":2}]                     │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── Queries (2) ─────────────────────────────────────┐  │
│ │ select * from "products" where "active" = ? [1]     │  │
│ │                               → 1.20ms               │  │
│ │ select * from "categories" limit 10                 │  │
│ │                               → 0.80ms               │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── Timeline ────────────────────────────────────────┐  │
│ │ [░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░] 120ms │  │
│ │ Bootstrap     │ Routing │ Controller │ View     │    │
│ │ 20ms          │ 5ms     │ 45ms       │ 50ms    │    │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── User ────────────────────────────────────────────┐  │
│ │ ID: 1 | Email: admin@example.com                    │  │
│ └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Informasi yang Tersedia di Detail Request

| Bagian | Isi | Kegunaan |
|--------|-----|----------|
| **Headers** | Semua HTTP headers | Cek content-type, user-agent, referer |
| **Input** | Data yang dikirim (POST/GET) | Cek data form/AJAX |
| **Session** | Data session saat request | Cek data user, cart, flash messages |
| **Response** | Status code + response body | Lihat response mentah |
| **Queries** | SQL queries yang dijalankan | Temukan query lambat atau N+1 |
| **Timeline** | Breakdown waktu eksekusi | Lihat bagian mana yang lambat |
| **User** | User yang sedang login (jika ada) | Cek siapa yang akses |
| **Exception** | Error (jika request gagal) | Stack trace lengkap |

### Contoh Penggunaan

**Skenario**: Halaman admin produk lambat loading.

1. Buka Telescope → Requests
2. Cari request `GET /admin/products`
3. Lihat **Duration** — 3.5 detik! Sangat lambat
4. Klik untuk detail
5. Buka tab **Queries** — lihat ada 100+ query
6. Sadari ada **N+1 problem** (query produk + query tiap kategori)
7. Perbaiki: tambahkan `->with('category')` di controller

---

## 6. Tab Commands

### Apa yang Direkam

Setiap perintah `php artisan ...` yang dijalankan.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Commands                                 [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ php artisan migrate       --force    ✓    2.5s    │ 13:40:00 │
│ php artisan queue:work     --once    ✓    1.2s    │ 13:35:00 │
│ php artisan route:list     --except-vendor ✓  0.8s│ 13:30:00 │
│ php artisan telescope:prune           ✓    0.3s    │ 13:00:00 │
└──────────────────────────────────────────────────────────────┘
```

### Informasi di Detail

| Informasi | Kegunaan |
|-----------|----------|
| Command name + arguments | Lihat perintah yang dieksekusi |
| Exit code | 0 = sukses, 1 = error |
| Duration | Berapa lama eksekusi |
| Output | Terminal output dari command |

### Contoh Penggunaan

**Skenario**: Sebuah scheduled command gagal di tengah malam.

1. Telescope → Commands
2. Filter waktu: cari jam 02:00
3. Lihat command yang exit code ≠ 0
4. Klik → lihat error output

---

## 7. Tab Schedule

### Apa yang Direkam

Cron job / scheduled tasks yang dijalankan oleh `php artisan schedule:run`.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Schedule                                [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 13:45 │ app\console\Commands\GenerateReport  ✓  5.2s       │
│ 13:00 │ app\console\Commands\CleanupOldData   ✓  12.3s     │
│ 12:00 │ app\console\Commands\SendNewsletter   ✗  30.0s     │ ← GAGAL
└──────────────────────────────────────────────────────────────┘
```

### Informasi di Detail

| Informasi | Kegunaan |
|-----------|----------|
| Command expression | `* * * * *` atau `dailyAt('13:00')` |
| Timezone | Zona waktu yang digunakan |
| Last run time | Kapan terakhir jalan |
| Next run time | Kapan akan jalan lagi |
| Output | Hasil eksekusi |

---

## 8. Tab Jobs

### Apa yang Direkam

Semua queue job yang diproses (via `php artisan queue:work`).

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Jobs                                   [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ ProcessOrder          ✓  success   2.1s   │ 3 attempts │
│ SendWelcomeEmail      ✗  failed    0.5s   │ 1 attempt  │ ← GAGAL
│ GenerateInvoice       ✓  success   1.8s   │ 1 attempt  │
└──────────────────────────────────────────────────────────────┘
```

### Informasi di Detail

| Informasi | Kegunaan |
|-----------|----------|
| Job class | Nama class job (`App\Jobs\ProcessOrder`) |
| Payload | Data yang dikirim ke job |
| Attempts | Berapa kali dicoba |
| Max tries | Batas maksimal percobaan |
| Queue name | Nama queue (default, emails, dll) |
| Failed at | Waktu gagal (jika gagal) |
| Exception | Error detail (jika gagal) |
| Duration | Waktu eksekusi |

### Contoh Penggunaan

**Skenario**: Order tidak terproses setelah pembayaran sukses.

1. Telescope → Jobs
2. Cari job `ProcessOrder` — status **failed**
3. Klik → lihat exception:
   ```
   App\Jobs\ProcessOrder: Call to a member function update() on null
   ```
4. Berarti `$order` tidak ditemukan — cek apakah ID order benar

---

## 9. Tab Exceptions

### Apa yang Direkam

Semua exception/error yang terjadi di aplikasi — **ini tab yang paling sering kamu gunakan**.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Exceptions                             [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ ✗  Class "App\Models\NonExistent" not found    13:45:01    │
│ ✗  SQLSTATE[42S02]: Base table or view not found 13:44:30 │
│ ✗  Call to undefined method getPrice()         13:40:00    │
└──────────────────────────────────────────────────────────────┘
```

### Detail Exception

```
┌─────────────────────────────────────────────────────────────┐
│ ✗  SQLSTATE[42S02]: Base table or view not found            │
│                                                             │
│ ┌─── Message ──────────────────────────────────────────┐  │
│ │ Base table or view not found: 1146 Table              │  │
│ │ 'olshop.products' doesn't exist                       │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── Stack Trace ─────────────────────────────────────┐  │
│ │ #0 app/Http/Controllers/ShopController.php:25      │  │
│ │ #1 vendor/laravel/framework/src/...                 │  │
│ │ #2 vendor/laravel/framework/src/...                 │  │
│ │ #3 vendor/laravel/framework/src/...                 │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── Occurrences (3) ────────────────────────────────┐  │
│ │ 13:45:01 - GET /shop                               │  │
│ │ 13:44:30 - GET /products?category=1                │  │
│ │ 13:44:00 - GET /admin/products                     │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── Request ────────────────────────────────────────┐  │
│ │ Method: GET                                         │  │
│ │ URL: /shop                                          │  │
│ │ User: admin@example.com                             │  │
│ └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Keunggulan Tab Exceptions Dibanding Log File

| Log File | Telescope Exceptions |
|----------|---------------------|
| Mentah, harus `tail -f` | Tampilan visual, rapi |
| Susah cari exception tertentu | Search + filter mudah |
| Tidak ada konteks request | Ada data request, user, session |
| Semua level campur jadi satu | Exception khusus, terpisah dari info/debug |
| Stack trace mentah | Stack trace rapi + file reference |

### Contoh Penggunaan

**Skenario**: Tiba-tiba halaman produk error 500.

1. Buka Telescope → Exceptions
2. Lihat exception paling atas (terbaru)
3. Baca pesan: `Call to undefined method App\Models\Product::getDiscountedPrice()`
4. Stack trace: `ShopController.php:45`
5. Buka file tersebut — ternyata method-nya bernama `getDiscountPrice()`
6. Fix: ganti nama method

---

## 10. Tab Logs

### Apa yang Direkam

Semua log yang direkam melalui `Log::info()`, `Log::error()`, dll. — termasuk log dari Laravel sendiri.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Logs                                    [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ [INFO]  Processing payment for order #1234        13:45:00  │
│ [ERROR] Midtrans API call failed: Connection timeout 13:44:30│
│ [WARNING] Product stock low: Nike Air (ID: 45)    13:40:00  │
│ [DEBUG] Cart updated for user #5                  13:35:00  │
└──────────────────────────────────────────────────────────────┘
```

### Filter Berdasarkan Level

Kamu bisa filter:

| Level | Warna | Arti |
|-------|-------|------|
| EMERGENCY | 🔴 | Sistem tidak bisa jalan |
| ALERT | 🔴 | Perlu tindakan segera |
| CRITICAL | 🔴 | Komponen tidak berfungsi |
| ERROR | 🟠 | Error, perlu diperbaiki |
| WARNING | 🟡 | Ada yang tidak biasa |
| NOTICE | 🔵 | Normal tapi signifikan |
| INFO | 🟢 | Informasi umum |
| DEBUG | ⚪ | Informasi detail developer |

### Membuat Log Sendiri

```php
// Di controller atau service mana pun:
use Illuminate\Support\Facades\Log;

Log::info('User checkout', [
    'user_id' => auth()->id(),
    'order_id' => $order->id,
    'total' => $order->total,
]);

Log::error('Midtrans callback failed', [
    'order_id' => $orderId,
    'response' => $response,
    'error' => $e->getMessage(),
]);
```

Log ini langsung muncul di Telescope → Logs. **Jauh lebih praktis daripada buka file `laravel.log`**.

---

## 11. Tab Dumps

### Apa yang Direkam

Output dari `dump()` yang dipanggil di aplikasi.

### Kenapa Ini Berguna?

`dd()` menghentikan eksekusi — kamu tidak bisa lihat halaman. `dump()` menampilkan data di halaman — tapi berantakan.

Dengan Telescope, `dump()` disimpan di dashboard:

```php
// Di controller:
dump($products, 'Ini data produk');
dump($request->all());
```

Hasilnya muncul di Telescope → Dumps:

```
┌──────────────────────────────────────────────────────────────┐
│ Dumps                                   [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 13:45:01  │ ShopController.php:25                           │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ array:3 [                                               │ │
│ │   0 => Product { id: 1, name: "Sepatu Nike", ... }    │ │
│ │   1 => Product { id: 2, name: "Tas Eiger", ... }      │ │
│ │   2 => Product { id: 3, name: "Jam Tangan", ... }     │ │
│ │ ]                                                       │ │
│ │ "Ini data produk"                                       │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ 13:44:30  │ CheckoutController.php:55                       │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ array:4 [                                               │ │
│ │   "name" => "Fathan"                                   │ │
│ │   "email" => "fathan@example.com"                      │ │
│ │   "address" => "Jl. Merdeka No. 1"                     │ │
│ │   "phone" => "08123456789"                             │ │
│ │ ]                                                       │ │
│ └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Tips: dump() vs dd()

| Fungsi | Efek | Telescope |
|--------|------|-----------|
| `dump($data)` | Lanjut eksekusi, tampilkan di halaman | Tercatat |
| `dd($data)` | Hentikan eksekusi | Tercatat |
| `ddd($data)` | Hentikan + tampilan Whoops | Tercatat |

**Best practice**: Pakai `dump()` saja, cek hasilnya di Telescope, lalu hapus. Halaman tetap berfungsi normal.

---

## 12. Tab Queries

### Apa yang Direkam

Setiap query SQL yang dijalankan oleh aplikasi — **sangat penting untuk optimasi**.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Queries                                [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 14.50ms  select * from "products"       ProductsController  │
│          where "category_id" = ?        index:45            │
│          and "active" = ?               [1, 1]             │
│─────────────────────────────────────────────────────────────│
│ 350.20ms select * from "products"       ShopController      │
│          left join "reviews" on ...      index:22            │
│          group by "products"."id"        []                 │
│          order by "rating" desc                              │
│          ⚠️ SLOW QUERY                                      │
│─────────────────────────────────────────────────────────────│
│ 1.20ms   select * from "categories"     View: shop          │
│          where "id" in (1,2,3)          index.blade.php:25  │
│─────────────────────────────────────────────────────────────│
│ 0.80ms   select * from "categories"     View: shop          │
│          where "id" = ?                 index.blade.php:25  │
│          🔴 N+1 DETECTED!                                   │
└──────────────────────────────────────────────────────────────┘
```

### Kolom Informasi

| Kolom | Arti |
|-------|------|
| **Duration** | Waktu eksekusi (ms) — >100ms perlu diwaspadai |
| **SQL** | Query mentah (binding diganti `?`) |
| **Location** | File + baris tempat query dipanggil |
| **Bindings** | Nilai parameter query (`?` diganti angka) |
| **Warning** | Slow query / N+1 detected |

### Slow Query — Apa Itu?

Query dengan durasi > 100ms (default). Laravel marked sebagai slow.

**Penyebab umum**:
- JOIN terlalu banyak
- Tidak ada index di kolom `WHERE` / `JOIN`
- Data terlalu besar tanpa pagination
- Loop query (N+1)

**Cara fix**:
```php
// ❌ Lambat: query di loop
$products = Product::all();
foreach ($products as $product) {
    $category = Category::find($product->category_id); // Query N+1!
}

// ✅ Cepat: eager loading
$products = Product::with('category')->get();
```

### N+1 Problem — Deteksi Otomatis

Telescope bisa **mendeteksi N+1** secara otomatis. N+1 adalah ketika kamu menjalankan 1 query untuk mengambil data, lalu N query tambahan untuk relasi di loop.

Contoh:
```php
// Ini causes N+1:
$products = Product::all(); // 1 query
foreach ($products as $product) {
    echo $product->category->name; // N query tambahan!
}
```

Total = 1 + N query. Telescope akan menandainya dengan **🔴 N+1 DETECTED!**

### Filter Queries

Kamu bisa filter queries berdasarkan:
- **Slow** — hanya query lambat (>100ms)
- **Duplicate** — query yang sama dijalankan berulang
- **By duration** — urutkan dari yang paling lambat

### Contoh Penggunaan

**Skenario**: Halaman shop loading lambat.

1. Telescope → Queries
2. Urutkan berdasarkan duration (descending)
3. Lihat query paling lambat:
   ```
   850ms - select * from products order by created_at desc
   ```
4. Tidak ada `WHERE` dan `LIMIT` — ambil semua produk!
5. Fix di controller: tambahkan `->paginate(12)`

---

## 13. Tab Mail

### Apa yang Direkam

Semua email yang "dikirim" oleh aplikasi. Karena di `.env` kita punya `MAIL_MAILER=log`, email sebenarnya tidak dikirim — hanya dicatat. Telescope menangkapnya.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Mail                                    [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 13:45:00  │ Order Confirmation #1234 → fathan@email.com    │
│ 13:40:00  │ Welcome! → fathan@email.com                    │
│ 13:30:00  │ Payment Received → admin@example.com           │
└──────────────────────────────────────────────────────────────┘
```

### Detail Email

```
┌─────────────────────────────────────────────────────────────┐
│ To: fathan@email.com                                        │
│ Subject: Order Confirmation #1234                           │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│                                                             │
│ ┌─── HTML Preview ─────────────────────────────────────┐  │
│ │                                                       │  │
│ │   Koneksi Store                                       │  │
│ │   ─────────────────────────                           │  │
│ │   Hi Fathan!                                         │  │
│ │   Your order #1234 has been confirmed.                │  │
│ │   Total: Rp 250.000                                  │  │
│ │                                                       │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                             │
│ ┌─── Raw Data ────────────────────────────────────────┐  │
│ │ Mailable: App\Mail\OrderConfirmation                 │  │
│ │ Attachments: none                                    │  │
│ │ Headers: Message-ID: <...>                           │  │
│ └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Keuntungan

- **Lihat HTML email** — tanpa benar-benar kirim email
- **Cek data dinamis** — apakah nama, total, dll. sudah benar
- **Debug template** — lihat apakah Blade email dirender dengan benar
- **Cek attachment** — apakah file terlampir

### Contoh Penggunaan

**Skenario**: Pelanggan komplain tidak terima email konfirmasi.

1. Telescope → Mail
2. Cari email ke alamat pelanggan
3. Lihat isi email — apakah data sudah benar
4. Cek apakah email terkirim (tercatat) atau tidak tercatat sama sekali
5. Jika tidak tercatat → ada bug di code yang mengirim email

---

## 14. Tab Notifications

### Apa yang Direkam

Semua notifikasi (database notification, email notification, dll).

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Notifications                          [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 13:45:00  │ OrderStatusChanged → User #1 (fathan@email.com)│
│           │ Status: paid → processing                       │
│─────────────────────────────────────────────────────────────│
│ 13:40:00  │ WelcomeNotification → User #5 (umar@email.com) │
└──────────────────────────────────────────────────────────────┘
```

### Detail

| Informasi | Kegunaan |
|-----------|----------|
| Notification class | Nama class notifikasi |
| Recipient | User penerima |
| Channels | Email, database, SMS, dll. |
| Data | Data yang dikirim di notifikasi |
| Read at | Waktu dibaca (untuk database notification) |

---

## 15. Tab Cache

### Apa yang Direkam

Semua operasi cache: get, set, forget, hit, dan miss.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Cache                                 [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 13:45:00  │ GET  products:all           ❌ MISS  1.2ms     │
│ 13:44:55  │ SET  products:all           ✅       15.3ms    │
│ 13:44:50  │ GET  products:all           ✅ HIT   0.3ms     │
│ 13:44:00  │ FORGET products:all         ✅                  │
│ 13:40:00  │ GET  categories:all         ✅ HIT   0.2ms     │
└──────────────────────────────────────────────────────────────┘
```

### Informasi

| Ikon | Arti |
|------|------|
| ✅ HIT | Cache ditemukan (cepat) |
| ❌ MISS | Cache tidak ditemukan (perlu set ulang) |
| ✅ SET | Data disimpan ke cache |
| ✅ FORGET | Cache dihapus |

### Contoh Penggunaan

**Skenario**: Cache products clearing terlalu sering.

1. Telescope → Cache
2. Lihat pattern: `products:all` di-set, lalu di-forget dalam 1 menit
3. Cek kode — mungkin ada event yang salah trigger

---

## 16. Tab Events

### Apa yang Direkam

Semua event yang di-dispatch (via `Event::dispatch()` atau `event()`).

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Events                                [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 13:45:00  │ App\Events\OrderStatusChanged                   │
│           │ Listener: SendOrderStatusNotification           │
│           │ Listener: UpdateOrderInCache                    │
│─────────────────────────────────────────────────────────────│
│ 13:40:00  │ App\Events\ProductViewed                        │
│           │ Listener: IncrementProductViewCount             │
└──────────────────────────────────────────────────────────────┘
```

### Informasi

| Kolom | Arti |
|-------|------|
| Event class | Nama event |
| Listeners | Semua listener yang dipanggil |
| Payload | Data yang dikirim event |
| Duration | Waktu total eksekusi semua listener |

### Contoh Penggunaan

**Skenario**: Notifikasi tidak terkirim saat status order berubah.

1. Telescope → Events
2. Cari event `OrderStatusChanged`
3. Lihat apakah event di-dispatch
4. Lihat listener terdaftar — ada `SendOrderStatusNotification`?
5. Jika ya, lihat apakah listener error atau sukses

---

## 17. Tab Gates

### Apa yang Direkam

Semua pengecekan otorisasi via `Gate` atau `Policy`.

### Tampilan

```
┌──────────────────────────────────────────────────────────────┐
│ Gates                                 [Search...] [Filter] │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│ 13:45:00  │ ✅  update-order → User #1 can update Order #123│
│ 13:45:00  │ ❌  update-order → User #5 cannot update Order  │
│           │     #123  (Policy: User is not the owner)      │
│─────────────────────────────────────────────────────────────│
│ 13:40:00  │ ✅  view-admin-dashboard → User #1              │
│           │     (Role: admin)                               │
└──────────────────────────────────────────────────────────────┘
```

### Informasi

| Kolom | Arti |
|-------|------|
| Gate/Policy name | Nama gate yang dicek |
| Result | ✅ ability/allowed atau ❌ deny/forbidden |
| User | User yang dicek |
| Target | Model yang diotorisasi (jika ada) |
| Reason | Alasan ditolak (jika ada) |

### Contoh Penggunaan

**Skenario**: User tidak bisa update order padahal dia pemiliknya.

1. Telescope → Gates
2. Cari `update-order` untuk user tersebut
3. Lihat hasilnya — denied dengan reason: "User is not the owner"
4. Cek Policy → `OrderPolicy::update()` — mungkin logika pengecekan owner-nya salah

---

## 18. Filtering & Pencarian Data

### Search Box

Di setiap tab ada search box. Cara pakai:

```
# Cari berdasarkan URL
/shop

# Cari berdasarkan status code
500

# Cari berdasarkan user email
fathan@email.com

# Cari berdasarkan exception message
"Base table not found"

# Cari query SQL
select * from products
```

### Filter Tag

Telescope mendukung **tag** — label khusus yang bisa ditambahkan ke entry.

Tag otomatis:
- `Slow` — untuk query > 100ms
- `N+1` — untuk query N+1 detected

Tag manual via Watches (lihat bagian Watches).

### Filter Berdasarkan Waktu

Di bagian atas, ada dropdown untuk filter waktu:

```
Last 15 minutes
Last 30 minutes
Last hour
Last 24 hours
Today
All time
```

---

## 19. Watching (Watches) — Memonitor Tertentu

### Apa Itu Watch?

**Watch** adalah fitur untuk memonitor hal-hal spesifik. Kamu bisa memberi tag otomatis pada entry yang memenuhi kriteria tertentu.

### Cara Konfigurasi Watch

Di `app/Providers/TelescopeServiceProvider.php`:

```php
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filter(function (IncomingEntry $entry) {
        // ... filter code
    });

    // Watch specific things
    Telescope::watch('slow-queries', function (IncomingEntry $entry) {
        return $entry->type === 'query' && $entry->content['duration'] > 500;
    });

    Telescope::watch('payment-errors', function (IncomingEntry $entry) {
        return $entry->type === 'exception' && 
               str_contains($entry->content['message'], 'Midtrans');
    });

    Telescope::watch('user-registrations', function (IncomingEntry $entry) {
        return $entry->type === 'request' && 
               $entry->content['uri'] === '/register' &&
               $entry->content['method'] === 'POST';
    });
}
```

### Manfaat Watch

Setelah watch dibuat, entry akan punya tag khusus:

```
┌──────────────────────────────────────────────────────────────┐
│ Exceptions                   [Search...] [Filter: payment]   │
│━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━│
│    Midtrans API call failed: Connection timeout    13:45     │
│    Tag: payment-errors                                       │
└──────────────────────────────────────────────────────────────┘
```

Kamu bisa filter berdasarkan tag di dropdown filter.

---

## 20. Konfigurasi Lanjutan

### 20.1 Mematikan Watcher Tidak Berguna

Tidak semua tab perlu aktif. Matikan yang tidak dipakai di `config/telescope.php`:

```php
'watchers' => [
    // Matikan yang tidak perlu
    Watchers\CacheWatcher::class => env('TELESCOPE_CACHE_WATCHER', true),
    Watchers\CommandWatcher::class => env('TELESCOPE_COMMAND_WATCHER', false),
    Watchers\DumpWatcher::class => env('TELESCOPE_DUMP_WATCHER', true),
    Watchers\EventWatcher::class => env('TELESCOPE_EVENT_WATCHER', true),
    Watchers\ExceptionWatcher::class => env('TELESCOPE_EXCEPTION_WATCHER', true),
    Watchers\GateWatcher::class => env('TELESCOPE_GATE_WATCHER', true),
    Watchers\JobWatcher::class => env('TELESCOPE_JOB_WATCHER', true),
    Watchers\LogWatcher::class => env('TELESCOPE_LOG_WATCHER', true),
    Watchers\MailWatcher::class => env('TELESCOPE_MAIL_WATCHER', true),
    Watchers\ModelWatcher::class => env('TELESCOPE_MODEL_WATCHER', false),
    Watchers\NotificationWatcher::class => env('TELESCOPE_NOTIFICATION_WATCHER', true),
    Watchers\QueryWatcher::class => env('TELESCOPE_QUERY_WATCHER', true),
    Watchers\RedisWatcher::class => env('TELESCOPE_REDIS_WATCHER', false),
    Watchers\RequestWatcher::class => env('TELESCOPE_REQUEST_WATCHER', true),
    Watchers\ScheduleWatcher::class => env('TELESCOPE_SCHEDULE_WATCHER', false),
    Watchers\ViewWatcher::class => env('TELESCOPE_VIEW_WATCHER', false),
];
```

Di `.env` kamu bisa override:

```
TELESCOPE_DUMP_WATCHER=false
TELESCOPE_QUERY_WATCHER=true
TELESCOPE_MAIL_WATCHER=true
```

### 20.2 Filter Data Sensitif

Telescope otomatis menyembunyikan data sensitif:

```php
// Di TelescopeServiceProvider.php
protected function hideSensitiveRequestDetails(): void
{
    Telescope::hideRequestParameters(['_token', 'password', 'password_confirmation']);

    Telescope::hideRequestHeaders([
        'cookie',
        'x-csrf-token',
        'x-xsrf-token',
    ]);
}
```

### 20.3 Filter Entries

Kamu bisa filter entry apa yang masuk ke Telescope:

```php
// Di TelescopeServiceProvider.php
Telescope::filter(function (IncomingEntry $entry) {
    if ($this->app->environment('local')) {
        return true; // Rekam semua di local
    }

    // Di production, rekam hanya exception
    return $entry->isReportableException();
});
```

### 20.4 Storage Configuration

Telescope menggunakan database untuk menyimpan data:

```
TELESCOPE_DRIVER=database
```

Tabel yang dibuat:

| Tabel | Fungsi |
|-------|--------|
| `telescope_entries` | Semua data entry |
| `telescope_entries_tags` | Tags untuk filter |
| `telescope_monitoring` | Data monitoring |

### 20.5 Pruning (Membersihkan Data Lama)

Telescope bisa sangat besar jika tidak dibersihkan. Gunakan pruning:

```bash
# Hapus data > 24 jam
php artisan telescope:prune

# Hapus data > 7 hari
php artisan telescope:prune --hours=168
```

Jadwalkan di `routes/console.php`:

```php
Schedule::command('telescope:prune --hours=168')->daily();
```

---

## 21. Telescope di Production

### 21.1 Apakah Aman untuk Production?

**Hati-hati!** Telescope di production:
- **Boros storage** — setiap request, query, log disimpan
- **Security risk** — data sensitif (input, session) bisa bocor
- **Performance impact** — setiap request butuh INSERT ke database

### 21.2 Recommended Production Setup

**Pendekatan 1: Telescope hanya untuk admin**

```php
// Di AppServiceProvider atau middleware
if (app()->environment('production') && ! auth()->user()?->isAdmin()) {
    Telescope::stopRecording();
}
```

**Pendekatan 2: Record hanya exception**

```php
Telescope::filter(function (IncomingEntry $entry) {
    return $entry->isReportableException();
});
```

**Pendekatan 3: Environment-based**

```php
// .env
TELESCOPE_ENABLED=false  # Production
TELESCOPE_ENABLED=true   # Local
```

**Pendekatan 4: Gunakan storage berbeda**

```php
// config/telescope.php
'storage' => [
    'database' => [
        'connection' => env('TELESCOPE_DB_CONNECTION', 'mysql'),
    ],
],
```

Gunakan database terpisah untuk Telescope agar tidak membebani database utama.

### 21.3 Security — Batasi Akses

Pastikan hanya admin yang bisa akses Telescope:

```php
// Di AppServiceProvider
Telescope::auth(function ($request) {
    return app()->environment('local') || auth()->user()?->isAdmin();
});
```

Atau proteksi via middleware:

```php
// routes/web.php tambahin
Route::middleware(['auth', 'role:admin,super-admin'])->group(function () {
    // Telescope routes already registered by service provider
});
```

---

## 22. Studi Kasus — Debugging Real World

### Studi Kasus 1: Halaman Checkout Error

**Laporan**: User tidak bisa checkout, dapat halaman putih.

**Langkah debugging**:

```
1. Buka Telescope → Exceptions
   → Ada: "Call to undefined method App\Services\RajaOngkirService::getShippingRates()"

2. Stack trace:
   CheckoutController.php:35 → RajaOngkirService.php

3. Cek metode di RajaOngkirService.php
   → Namanya getRates(), bukan getShippingRates()

4. Fix: ganti nama fungsi atau panggil fungsi yang benar
```

**Alternatif**: Buka Telescope → Requests → cari POST `/checkout/process` → lihat response 500 → tab Exception untuk lihat pesan error.

### Studi Kasus 2: Produk Tidak Muncul di Halaman Shop

**Laporan**: Halaman shop kosong, tidak ada produk.

**Langkah debugging**:

```
1. Buka Telescope → Queries
   → Urutkan descending by duration

2. Lihat query:
   select * from "products" where "active" = ? [1]

3. Cek hasil:
   Telescope → Requests → GET /shop → Queries
   → Query return 0 rows

4. Cek database:
   Apakah ada produk dengan active = 1?
   → Ternyata semua produk active = 0

5. Fix: Update produk atau ubah query
```

### Studi Kasus 3: Email Konfirmasi Tidak Sampai

**Laporan**: Pelanggan bayar tapi tidak dapat email.

**Langkah debugging**:

```
1. Buka Telescope → Mail
   → Cari email ke pelanggan
   → Tidak ada!

2. Buka Telescope → Logs
   → Cari log payment success

3. Buka Telescope → Exceptions
   → Cek apakah ada error di pembayaran
   → Tidak ada error

4. Buka Telescope → Events
   → Cari event OrderStatusChanged
   → Event tidak di-dispatch!

5. Cek kode PaymentController:
   → Ternyata lupa panggil event setelah payment success

6. Fix: Tambahkan event dispatch
```

### Studi Kasus 4: Halaman Admin Loading 10 Detik

**Laporan**: Dashboard admin sangat lambat.

**Langkah debugging**:

```
1. Buka Telescope → Requests → GET /admin

2. Lihat Duration: 10.2s

3. Klik request → tab Queries
   → Ada 250 query!

4. Lihat query berulang:
   select * from "orders" where "user_id" = ? — diulang 50x
   select * from "order_items" where "order_id" = ? — diulang 100x

5. N+1 problem!
   Fix: eager loading
   Order::with('items')->get()
```

### Studi Kasus 5: Midtrans Callback Gagal

**Laporan**: Pembayaran sukses di Midtrans tapi status order tidak berubah.

**Langkah debugging**:

```
1. Buka Telescope → Requests
   → Cari POST /midtrans/webhook
   → Status 500!

2. Klik → tab Exception
   → "Signature key mismatch"

3. Cek MidtransService:
   → Ternyata server key di .env salah (beda dengan Midtrans dashboard)

4. Fix: Update MIDTRANS_SERVER_KEY di .env
```

---

## 23. Tips & Trik

### Tip 1: Filter by Tag dengan Cepat

Gunakan search box dengan format `tag:nama-tag`:

```
tag:slow-queries
tag:payment-errors
tag:N+1
```

### Tip 2: Batch Hapus Telescope Entry

Jangan hapus manual via database. Pakai pruning:

```bash
# Hapus semua data
php artisan telescope:clear

# Hapus data > tertentu
php artisan telescope:prune --hours=1
```

### Tip 3: Export Data Telescope

Telescope tidak punya fitur export bawaan. Tapi kamu bisa query langsung:

```sql
-- Lihat recent exceptions
SELECT * FROM telescope_entries 
WHERE type = 'exception' 
ORDER BY created_at DESC 
LIMIT 10;
```

### Tip 4: Monitoring Real-Time

Untuk real-time monitoring, kombinasikan dengan:
- **Laragon APM Monitor** — untuk CPU/memory
- **Laravel Pail** — untuk log terminal real-time: `php artisan pail`
- **Telescope + refresh manual** — Telescope tidak auto-refresh, tap F5

### Tip 5: Debug dengan dump() + Telescope

Kebiasaan buruk: pakai `dd()` dan berhenti.

Kebiasaan baik: pakai `dump()` dan lanjutkan — hasilnya tetap di Telescope.

```php
// ❌ Buruk: halaman jadi putih
dd($products);

// ✅ Baik: halaman tetap jalan, data ada di Telescope
dump($products);
```

### Tip 6: Cek Dependency Injection

Bingung kenapa error "Target class ... does not exist"?

```php
// Telescope → Exceptions akan menangkap:
Target [App\Services\CartService] is not instantiable.
```

Itu berarti class tidak di-inject dengan benar ke constructor.

### Tip 7: Debug Queue Jobs

Kalau ada job gagal:

1. Telescope → Jobs → cari failed job
2. Klik → lihat exception dan payload
3. Payload berisi data yang dikirim ke job
4. Simulasi manual dengan data yang sama di tinker:
   ```bash
   php artisan tinker
   > $job = new ProcessOrder(123);
   > $job->handle();
   ```

### Tip 8: Gunakan Environment Indicator

Biar tahu sedang di environment apa:

```
Telescope di local → record everything
Telescope di production → record exception only
Telescope di staging → record request + exception
```

Setting di `.env`:

```bash
# .env.local
TELESCOPE_ENABLED=true

# .env.production
TELESCOPE_ENABLED=false
```

### Tip 9: Pahami Batch vs Real-Time

Telescope tidak real-time — data ditulis ke database saat request selesai. Ada jeda beberapa milidetik.

Untuk debugging yang benar-benar real-time, pakai Laravel Pail:

```bash
php artisan pail
```

Output:
```
[2026-06-13 13:45:01] local.ERROR: Something went wrong
[2026-06-13 13:45:02] local.INFO: User logged in
```

### Tip 10: Gunakan Telescope untuk Belajar

Telescope bukan cuma debugging — kamu bisa belajar dari Telescope:

| Yang Dipelajari | Caranya |
|----------------|---------|
| Berapa query yang dijalankan setiap halaman | Buka tab Queries |
| Apakah cache bekerja | Buka tab Cache |
| Event apa yang dipicu saat checkout | Buka tab Events |
| Middleware apa yang dijalankan | Buka tab Requests → Headers |
| Data session apa yang disimpan | Buka tab Requests → Session |
| Apakah policy bekerja dengan benar | Buka tab Gates |

---

## 24. Perintah Artisan Telescope

| Perintah | Fungsi |
|----------|--------|
| `php artisan telescope:install` | Install Telescope (migration + config) |
| `php artisan telescope:clear` | Hapus semua data Telescope |
| `php artisan telescope:prune` | Hapus data lama (default > 24 jam) |
| `php artisan telescope:prune --hours=168` | Hapus data > 7 hari |

---

## Penutup

Telescope adalah **debugging assistant nomor 1** di ekosistem Laravel. Dengan Telescope, kamu tidak perlu lagi:

- Buka-buka file `storage/logs/laravel.log` secara manual
- Menebak-nebak SQL apa yang dijalankan
- Bingung kenapa email/notifikasi tidak terkirim
- Susah tracking error yang muncul di production

**Alur debugging pakai Telescope**:

```
1. Ada error → buka Telescope
2. Cek tab Exceptions (error) atau Requests (behavior)
3. Lihat stack trace → tahu file dan baris
4. Lihat query SQL yang dijalankan
5. Lihat data request (input, session, user)
6. Fix kode
7. Test lagi
8. Cek Telescope lagi → error hilang
```

Semakin sering kamu pakai Telescope, semakin cepat kamu debug. Ini investasi waktu yang sangat berharga.

---

*Dokumen ini dibuat untuk keperluan belajar tim Koneksi Store. Disimpan di .openkb sebagai referensi pengetahuan terbuka.*
