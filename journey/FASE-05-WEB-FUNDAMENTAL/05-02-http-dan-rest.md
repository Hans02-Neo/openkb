# 05-02: HTTP dan REST API — Bahasa Komunikasi Web

> **Fase**: 5 — Web Fundamental  
> **Prasyarat**: 05-01-cara-kerja-web  
> **Waktu baca**: 70-85 menit  
> **Kata kunci**: HTTP methods, status codes, headers, body, REST, API, JSON, stateless, idempotent, Postman

---

## 📋 Ringkasan

Setelah paham perjalanan request, sekarang kita bedah **protokol HTTP** secara detail — method, status code, header, body. Dan kita lihat bagaimana **REST API** menggunakan HTTP sebagai arsitektur komunikasi antara client dan server.

**Target pemahaman:**
- Kamu paham semua HTTP method dan kapan pakainya
- Kamu bisa membedakan status code 2xx, 3xx, 4xx, 5xx
- Kamu paham REST principles
- Kamu bisa membaca dan membuat API endpoint

---

## 1. HTTP Methods — Kata Kerja Web

### 1.1 CRUD Mapping

| HTTP Method | CRUD | SQL | Efek |
|-------------|------|-----|------|
| **GET** | Read | SELECT | Tidak mengubah data |
| **POST** | Create | INSERT | Membuat resource baru |
| **PUT** | Update (full) | UPDATE | Mengganti seluruh resource |
| **PATCH** | Update (partial) | UPDATE | Mengganti sebagian resource |
| **DELETE** | Delete | DELETE | Menghapus resource |

### 1.2 GET — Minta Data

```php
// GET tidak punya body — data lewat URL query string
GET /api/products?category=laptop&sort=price HTTP/1.1
```

**Di codebase:**
```php
// web.php
Route::get('/shop', [ShopController::class, 'index'])->name('shop.index');

// Query string diakses via $request
$category = $request->query('category');   // ?category=laptop
$search   = $request->query('search');     // ?search=laptop
$sort     = $request->input('sort', 'terbaru'); // ?sort=terbaru (dengan default)
```

### 1.3 POST — Kirim Data Baru

```php
// POST data di body, bukan URL
POST /checkout HTTP/1.1
Content-Type: application/json

{
    "customer_id": 1,
    "items": [
        {"product_id": 5, "quantity": 2},
        {"product_id": 3, "quantity": 1}
    ]
}
```

**Di codebase:**
```php
// web.php
Route::post('/checkout', [CheckoutController::class, 'process'])
    ->name('checkout.process');

// Controller menerima data via $request
public function process(CheckoutRequest $request)
{
    $validated = $request->validated();
    // $validated['customer_id'], $validated['items'], etc.
}
```

### 1.4 PUT vs PATCH

**PUT** — mengganti seluruh resource:
```
PUT /api/products/5
Body: {"name": "Laptop", "price": 15000000, "stock": 10}
→ Seluruh data produk 5 diganti dengan data ini
```

**PATCH** — mengganti sebagian:
```
PATCH /api/products/5
Body: {"price": 14000000}
→ Hanya price yang berubah, name dan stock tetap
```

**Di codebase:**
```php
// Cart update
Route::put('/cart/{cart}', [CartController::class, 'update'])->name('cart.update');
```

### 1.5 DELETE — Hapus Resource

```php
DELETE /api/products/5 HTTP/1.1
```

**Di codebase:**
```php
Route::delete('/cart/{cart}', [CartController::class, 'destroy'])->name('cart.destroy');
```

### 1.6 Idempotent — Panggil Sekali vs Berulang

**Idempotent:** Panggil 1x atau 100x, hasilnya sama.

| Method | Idempotent? | Keterangan |
|--------|------------|------------|
| GET | ✅ Ya | Baca data 100x → data tidak berubah |
| PUT | ✅ Ya | Set price=1000 100x → tetap 1000 |
| DELETE | ✅ Ya | Hapus 1x atau 100x → sudah terhapus |
| PATCH | ❌ Tidak | `PATCH /cart/1 {"qty": +1}` → tiap panggil qty nambah |
| POST | ❌ Tidak | POST order 2x → 2 order masuk |

---

## 2. HTTP Status Code — Bahasa Server

### 2.1 2xx Success

| Kode | Nama | Arti |
|------|------|------|
| 200 | OK | Request berhasil |
| 201 | Created | Resource berhasil dibuat |
| 204 | No Content | Berhasil, tidak ada response body |

**Di codebase:**
```php
// 200 — default return
return response()->json($products);

// 201 — setelah checkout sukses
return response()->json(['order' => $order], 201);

// 204 — setelah hapus item cart
return response()->json(null, 204);
```

### 2.2 3xx Redirection

| Kode | Nama | Arti |
|------|------|------|
| 301 | Moved Permanently | URL sudah pindah selamanya |
| 302 | Found | URL pindah sementara |
| 304 | Not Modified | Gunakan cache |

**Di codebase:**
```php
// 302 — redirect setelah login
return redirect()->route('orders.index');

// 301 — redirect permanent (di .htaccess)
```

### 2.3 4xx Client Error

| Kode | Nama | Arti |
|------|------|------|
| 400 | Bad Request | Request salah format |
| 401 | Unauthorized | Belum login |
| 403 | Forbidden | Tidak punya akses |
| 404 | Not Found | Resource tidak ditemukan |
| 419 | Page Expired | CSRF token mismatch |
| 422 | Unprocessable Entity | Validasi gagal |
| 429 | Too Many Requests | Rate limit exceeded |

**Di codebase:**
```php
// 401 — middleware auth
Route::middleware(['auth'])->group(function () { ... });

// 403 — policy
$this->authorize('view', $order); // Gate/Policy

// 404 — findOrFail
$product = Product::findOrFail($id); // 404 jika tidak ada

// 419 — CSRF token expired (halaman form)

// 422 — validasi gagal
// CheckoutRequest → validasi → 422 + error messages

// 429 — throttle
Route::middleware(['throttle:60,1'])->group(function () { ... });
```

### 2.4 5xx Server Error

| Kode | Nama | Arti |
|------|------|------|
| 500 | Internal Server Error | Exception tidak terduga |
| 502 | Bad Gateway | Server upstream error |
| 503 | Service Unavailable | Server sibuk/maintenance |
| 504 | Gateway Timeout | Server upstream timeout |

**Di codebase:**
```php
// 500 — exception tidak ter-handle
// Telescope akan mencatat semua 500

// 503 — maintenance mode
// php artisan down → return 503
```

---

## 3. HTTP Headers — Metadata

### 3.1 Request Headers (Browser → Server)

| Header | Contoh | Guna |
|--------|--------|------|
| `Host` | `olshop-koneksi.test` | Domain tujuan |
| `User-Agent` | `Mozilla/5.0 ...` | Identitas browser |
| `Accept` | `application/json` | Response yang diinginkan |
| `Content-Type` | `application/json` | Format body request |
| `Authorization` | `Bearer token123` | Autentikasi |
| `Cookie` | `session=abc123` | Data session |
| `Referer` | `https://google.com` | Halaman asal |

### 3.2 Response Headers (Server → Browser)

| Header | Contoh | Guna |
|--------|--------|------|
| `Content-Type` | `text/html; charset=UTF-8` | Format response |
| `Content-Length` | `1234` | Ukuran body (bytes) |
| `Set-Cookie` | `session=abc; path=/` | Set cookie |
| `Cache-Control` | `max-age=3600` | Cache policy |
| `X-Frame-Options` | `DENY` | Security (clickjacking) |

### 3.3 Content-Type — Format Data

| Content-Type | Guna |
|-------------|------|
| `text/html` | Halaman web (HTML) |
| `application/json` | Data JSON (API) |
| `multipart/form-data` | Form dengan file upload |
| `application/x-www-form-urlencoded` | Form biasa |
| `text/plain` | Teks mentah |

**Bagaimana Laravel menentukan Content-Type:**
```php
// View → HTML
return view('shop.index'); // Content-Type: text/html

// JSON → response->json()
return response()->json($data); // Content-Type: application/json

// Redirect → 302
return redirect('/'); // Content-Type: text/html; charset=utf-8
```

---

## 4. REST API — Arsitektur Web Service

### 4.1 Apa Itu REST?

REST (Representational State Transfer) adalah gaya arsitektur untuk mendesain **API** (Application Programming Interface).

**Prinsip REST:**
1. **Resource-based** — setiap "sesuatu" adalah resource (produk, order, user)
2. **HTTP methods** — GET/POST/PUT/DELETE untuk operasi CRUD
3. **Stateless** — setiap request berdiri sendiri
4. **Uniform interface** — URL konsisten

### 4.2 RESTful URL Design

```
Resource: /api/products
Single resource: /api/products/{id}

GET    /api/products          → List semua produk
POST   /api/products          → Buat produk baru
GET    /api/products/5        → Detail produk 5
PUT    /api/products/5        → Update produk 5
DELETE /api/products/5        → Hapus produk 5

Nested (relasi):
GET    /api/categories/3/products → Produk di kategori 3
GET    /api/orders/5/items        → Item di order 5
```

### 4.3 Contoh REST API di Codebase

```php
// routes/api.php (belum dipakai di codebase ini — semua via web.php)
// Tapi pattern REST-nya tetap sama:

// Resource routes:
Route::resource('products', ProductController::class);
// → GET    /products         → index()
// → GET    /products/create  → create()
// → POST   /products         → store()
// → GET    /products/{id}    → show()
// → GET    /products/{id}/edit → edit()
// → PUT    /products/{id}    → update()
// → DELETE /products/{id}    → destroy()
```

### 4.4 API Resource di Laravel 11

```php
// routes/api.php
use App\Http\Controllers\Api\ProductController;

Route::apiResource('products', ProductController::class);
// Hanya: index, store, show, update, destroy
// Tanpa: create, edit (karena API tidak perlu form)
```

### 4.5 JSON Response Format

Response API biasanya konsisten:

```json
{
    "success": true,
    "data": {
        "id": 1,
        "name": "Laptop Gaming",
        "price": 15000000
    },
    "message": "Product retrieved successfully"
}
```

Atau untuk error:

```json
{
    "success": false,
    "message": "Validation failed",
    "errors": {
        "name": ["The name field is required."],
        "price": ["The price must be a number."]
    }
}
```

---

## 5. Stateless — Setiap Request Berdiri Sendiri

### 5.1 Konsep

**Stateless:** Server tidak menyimpan informasi tentang client antar request. Setiap request adalah independent.

```
❌ Stateful:
Request 1: GET /orders → Server ingat "user ini admin" 
                        (karena session tersimpan di server)
Request 2: GET /orders/5/edit → Server cek session → "oh user ini admin, boleh"

✅ Stateless (REST API):
Request 1: GET /orders
           Header: Authorization: Bearer token123
           → Server: verifikasi token, cek role → response

Request 2: GET /orders/5/edit
           Header: Authorization: Bearer token123
           → Server: verifikasi token lagi!
           (Tidak ingat request 1 — verifikasi ulang setiap kali)
```

### 5.2 Di Codebase

```php
// Web routes (stateful — pakai session):
Route::middleware(['auth'])->group(function () {
    Route::get('/orders', [OrderController::class, 'index']);
    // Session menyimpan status login
});

// API routes (stateless — pakai token):
Route::middleware(['auth:sanctum'])->group(function () {
    Route::get('/api/orders', [Api\OrderController::class, 'index']);
    // Setiap request harus kirim token
});
```

---

## 6. API Testing dengan Postman / Thunder Client

### 6.1 Kenapa Test API?

Browser hanya bisa **GET** request (via URL). Untuk POST, PUT, DELETE — butuh alat khusus:

- **Postman** — aplikasi desktop, fitur lengkap
- **Thunder Client** — ekstensi VS Code, ringan
- **curl** — command line

### 6.2 Contoh dengan curl

```bash
# GET request
curl http://olshop-koneksi.test/shop

# POST request dengan JSON body
curl -X POST http://olshop-koneksi.test/checkout \
  -H "Content-Type: application/json" \
  -H "X-CSRF-TOKEN: xxx" \
  -d '{"customer_id": 1, "items": [{"product_id": 5, "quantity": 2}]}'

# DELETE request
curl -X DELETE http://olshop-koneksi.test/cart/1
```

### 6.3 Contoh dengan PHP (Guzzle)

```php
// Laravel punya Http Client untuk testing API
use Illuminate\Support\Facades\Http;

$response = Http::get('https://api.rajaongkir.com/starter/city');
$cities = $response->json();
```

**Ini yang dipakai di RajaOngkirService:**
```php
$response = Http::withHeaders([
    'key' => $this->apiKey,
])->get($this->baseUrl . '/starter/city');
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ HTTP METHODS                                                       │
│  GET    → Baca data (tidak mengubah)                               │
│  POST   → Buat data baru                                           │
│  PUT    → Ganti seluruh data                                       │
│  PATCH  → Ganti sebagian data                                      │
│  DELETE → Hapus data                                               │
├─────────────────────────────────────────────────────────────────────┤
│ STATUS CODE                                                        │
│  2xx → Sukses        │  3xx → Redirect                             │
│  4xx → Client error  │  5xx → Server error                         │
├─────────────────────────────────────────────────────────────────────┤
│ REST API                                                           │
│  GET    /api/products      → list                                   │
│  POST   /api/products      → create                                │
│  GET    /api/products/{id} → detail                                │
│  PUT    /api/products/{id} → update                                │
│  DELETE /api/products/{id} → delete                                │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  web.php → GET/POST/PUT/DELETE untuk web (stateful)                │
│  api.php → (belum dipakai) — untuk REST API (stateless)            │
│  RajaOngkirService → HTTP GET ke API RajaOngkir                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Identifikasi method.** Buka `routes/web.php`. Temukan:
   - Route GET
   - Route POST
   - Route PUT/PATCH
   - Route DELETE

2. **Status code test.** Buka browser dan cek status code:
   - `http://olshop-koneksi.test/shop` → ?
   - `http://olshop-koneksi.test/login` → ?
   - `http://olshop-koneksi.test/orders` → ? (kalau belum login)

3. **API simulation.** Bayangkan kamu buat REST API untuk produk. Tulis URL dan method untuk:
   - Melihat semua produk
   - Membuat produk baru
   - Mengupdate harga produk id 5
   - Menghapus produk id 3

4. **Baca RajaOngkirService.** Buka `app/Services/RajaOngkirService.php`. Metode HTTP apa yang dipakai? Header apa yang dikirim?

5. **CSRF test.** Buka form checkout di browser. Inspect element, cari input `_token`. Apa yang terjadi jika token diubah?

---

## 🔗 Referensi

- [MDN: HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [MDN: HTTP Status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [REST API Tutorial](https://restfulapi.net/)
- [Laravel: HTTP Client](https://laravel.com/docs/11.x/http-client)
- [Laravel: API Resources](https://laravel.com/docs/11.x/eloquent-resources)
- [Laravel: API Authentication (Sanctum)](https://laravel.com/docs/11.x/sanctum)
- Codebase: `routes/web.php` — semua route web
- Codebase: `app/Services/RajaOngkirService.php` — HTTP Client
