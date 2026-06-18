# 05-06: Client vs Server — Siapa Ngapain

> **Fase**: 5 — Web Fundamental  
> **Prasyarat**: 05-05-dom-dan-browser  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: client-side, server-side, rendering, SSR, CSR, hybrid, validation, security, Blade, SPA

---

## 📋 Ringkasan

Di aplikasi web, ada **dua dunia** yang bekerja bersama: **client** (browser) dan **server** (Laravel). Tugas mereka berbeda tapi saling melengkapi. Dokumen ini membahas batas dan tanggung jawab masing-masing.

**Target pemahaman:**
- Kamu paham perbedaan SSR vs CSR
- Kamu tahu mana logika di client dan mana di server
- Kamu paham validasi ganda (client + server)
- Kamu tahu keamanan: apa yang aman di client, apa yang tidak

---

## 1. Pembagian Tugas

```
┌─────────────────────────────────┐     ┌─────────────────────────────────┐
│          CLIENT (Browser)        │     │          SERVER (Laravel)       │
├─────────────────────────────────┤     ├─────────────────────────────────┤
│  Menampilkan HTML/CSS           │     │  Menyimpan data (database)      │
│  Interaksi user (click, drag)   │     │  Logika bisnis (harga, diskon)  │
│  Validasi form (cepat)          │     │  Validasi final (keamanan)      │
│  Animasi, transisi              │     │  Autentikasi & otorisasi        │
│  AJAX fetch data                │     │  Session management             │
│  Caching (localStorage)         │     │  Integrasi pihak ketiga         │
│  Routing (SPA)                  │     │  API endpoints                  │
│  State UI (loading, error)      │     │  File storage                   │
└─────────────────────────────────┘     └─────────────────────────────────┘
```

**Prinsip penting:**
- **Client untuk UX** — kecepatan, interaksi, feedback instan
- **Server untuk kebenaran** — validasi final, keamanan, data permanen
- **Jangan percaya client!** — semua data dari client harus divalidasi ulang di server

---

## 2. Rendering — Siapa yang Membuat HTML?

### 2.1 SSR (Server-Side Rendering) — Cara Tradisional

**Ini yang dipakai codebase ini** — semua HTML dihasilkan di server.

```
1. User klik link → browser request ke server
2. Server (Laravel):
   - Query database
   - Render Blade template → HTML lengkap
3. Kirim HTML ke browser
4. Browser tampilkan HTML
5. (Optional) Hydrate JS untuk interaktivitas
```

```php
// Controller: semua logic di server
public function index()
{
    $products = Product::where('is_active', true)->get();
    return view('shop.index', compact('products'));
    // Semua HTML dihasilkan di sini
}
```

**Keuntungan SSR:**
- SEO bagus (HTML sudah jadi saat Google crawl)
- First paint cepat (HTML langsung tampil)
- Lebih sederhana (tidak perlu JS framework)

**Kekurangan SSR:**
- Setiap interaksi butuh reload halaman (atau partial fetch)
- Server lebih berat (render HTML untuk setiap request)

### 2.2 CSR (Client-Side Rendering) — Cara SPA

SPA (Single Page Application) seperti React, Vue, Angular.

```
1. Browser request → server kirim HTML KOSONG + JS bundle besar
2. Browser download + jalankan JS
3. JS fetch data dari API
4. JS generate HTML di browser
5. User klik link → JS render ulang (tanpa reload)
```

```javascript
// Contoh CSR: semua logic di client (React/Vue)
// Server hanya menyediakan API

fetch('/api/products')
    .then(res => res.json())
    .then(products => {
        // Generate HTML di client
        products.forEach(p => {
            document.querySelector('#app').innerHTML += `
                <div class="card">
                    <h3>${p.name}</h3>
                    <p>${p.price}</p>
                </div>
            `;
        });
    });
```

**Keuntungan CSR:**
- Navigasi cepat (tanpa reload)
- Server lebih ringan (kirim data JSON saja)
- UX lebih mulus (animasi transisi)

**Kekurangan CSR:**
- SEO jelek (Google lihat HTML kosong — tapi sudah membaik)
- First paint lambat (tunggu JS download + eksekusi)
- Lebih kompleks (butuh JS framework)

### 2.3 Hybrid — Best of Both

```html
<!-- Halaman pertama di-render server (SSR) →
     navigasi berikutnya via AJAX (CSR-like) -->
<!-- Ini yang dilakukan Inertia.js, Livewire, atau Turbo -->

<!-- Laravel punya Livewire → interaktivitas tanpa SPA -->
<!-- Tapi codebase ini pure SSR + vanilla JS -->
```

### 2.4 Codebase: SSR dengan Blade

Semua halaman di `olshop-koneksi` adalah SSR:

```php
// 1. Browser request /shop
// 2. Laravel query produk
// 3. Blade render HTML lengkap
// 4. Kirim ke browser
// 5. JS hanya untuk interaktivitas tambahan

// Kalau JS dimatikan di browser, halaman tetap berfungsi
// Form submit biasa (bukan AJAX)
```

---

## 3. Validasi — Client vs Server

### 3.1 Kenapa Validasi Ganda?

```
User ──isi form──► CLIENT VALIDATION ──kirim──► SERVER VALIDATION ──save──► DB
                      │                           │
                      │ Cepat, feedback instan     │ Final, keamanan
                      │ Tapi BISA DIMATIKAN!       │ Tidak bisa diakali
```

### 3.2 Client-Side Validation (UX)

```html
<!-- HTML5 validation — built-in browser -->
<input type="email" required minlength="3">
<!-- Browser cek otomatis sebelum submit -->

<!-- Custom JS validation -->
<script>
    document.querySelector('form').addEventListener('submit', function(e) {
        const email = this.querySelector('#email').value;
        if (!email.includes('@')) {
            alert('Email tidak valid!');
            e.preventDefault();
        }
    });
</script>
```

**Client validation mudah dilewati:**
- Matikan JavaScript di browser
- Kirim request langsung via curl/Postman
- Edit HTML via DevTools

### 3.3 Server-Side Validation (Final)

```php
// Laravel FormRequest — validasi di server
// app/Http/Requests/CheckoutRequest.php

class CheckoutRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'customer_id' => 'required|exists:customers,id',
            'items'       => 'required|array|min:1',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity'   => 'required|integer|min:1',
            'shipping_address'   => 'required|string|min:10',
        ];
    }

    public function messages(): array
    {
        return [
            'items.*.product_id.exists' => 'Produk tidak ditemukan',
            'shipping_address.min' => 'Alamat terlalu pendek',
        ];
    }
}
```

### 3.4 Contoh: Validasi Ganda di Checkout

| Lapisan | Lokasi | Cek | Bisa Dilewati? |
|---------|--------|-----|----------------|
| HTML5 | Browser | `required`, `type="email"` | ✅ Bisa (edit HTML) |
| JS | Browser | Format email, min chars | ✅ Bisa (matikan JS) |
| FormRequest | Server | `required`, `email`, `exists` | ❌ Tidak bisa |
| Controller | Server | Stok cukup, harga sesuai | ❌ Tidak bisa |
| Database | Server | Constraint, foreign key | ❌ Tidak bisa |

---

## 4. Keamanan — Jangan Percaya Client

### 4.1 Aturan Emas

> **"Never trust client input"** — semua data dari client (termasuk header, cookie, file) bisa dimanipulasi.

### 4.2 Apa yang Tidak Boleh di Client

```javascript
// ❌ Jangan pernah simpan ini di client:
// - Password / API keys
// - Secret tokens
// - Logika harga (diskon, pajak)
// - Role user (admin/user)

// ❌ Contoh BERBAHAYA:
const user = { role: 'admin' }; // Bisa diubah di DevTools!
localStorage.setItem('isAdmin', true); // Bisa diubah!

// ✅ Harga dan diskon harus dihitung SERVER
// Client hanya menampilkan hasil akhir
```

### 4.3 Apa yang Aman di Client

```javascript
// ✅ Aman di client:
// - UI state (tab aktif, modal terbuka/tutup)
// - Theme preference (dark/light)
// - Cart items (tapi untuk tampilan saja — final di server)
// - Form input sementara
// - Cache data publik (produk, kategori)
```

### 4.4 Contoh: Manipulasi Harga

```javascript
// ❌ JANGAN PERNAH — harga dari client!
fetch('/cart/add', {
    method: 'POST',
    body: JSON.stringify({
        product_id: 5,
        price: 100, // User bisa ganti jadi 100!
    }),
});

// ✅ Harga harus dari SERVER!
fetch('/cart/add', {
    method: 'POST',
    body: JSON.stringify({
        product_id: 5,
        // quantity saja — harga diambil dari database di server
        quantity: 1,
    }),
});
```

**Di CartService.php — harga selalu dari database:**
```php
// CartService hanya menerima product_id
// Harga diambil dari Product::find($product_id) di server
$product = Product::findOrFail($item['product_id']);
$price = $product->price; // HARGA DARI DATABASE!
```

---

## 5. State Management — Data Tinggal di Mana?

### 5.1 State Distribution

```
┌─────────────────────┐
│     DATABASE        │  ← Sumber kebenaran final
├─────────────────────┤
│     SESSION         │  ← State user sementara (login, cart)
├─────────────────────┤
│   LARAVEL CACHE     │  ← Data sementara yang mahal di-query
├─────────────────────┤
│     COOKIE          │  ← Data kecil yang perlu dikirim tiap request
├─────────────────────┤
│  localStorage       │  ← Data client-only (tema, preferensi)
├─────────────────────┤
│  JavaScript Memory  │  ← State UI sementara (loading, error, tab)
└─────────────────────┘
```

### 5.2 Contoh: Cart State

| Data | Lokasi | Kenapa? |
|------|--------|---------|
| Cart items detail | **Session** (server) | Final — harga dari DB |
| Cart count | **Session** count | Untuk badge di navbar |
| Cart (cache) | **localStorage** (opsional) | Agar tidak hilang saat refresh |
| Cart UI state | **JS memory** | Loading, error, empty state |

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ SERVER-SIDE (Laravel)              │ CLIENT-SIDE (Browser)          │
├────────────────────────────────────┼────────────────────────────────┤
│ Blade rendering (SSR)              │ DOM manipulation               │
│ Database queries                   │ Event handling                 │
│ Authentication & authorization     │ Client validation (UX)         │
│ Business logic (harga, diskon)     │ Animasi & transitions         │
│ Session management                 │ Fetch API (AJAX)               │
│ File upload & storage              │ localStorage caching          │
│ CSRF protection                    │ UI state (loading, error)     │
│ API endpoints                      │ Console & Debug               │
│ Queue & job processing             │ Responsive design via CSS     │
├────────────────────────────────────┼────────────────────────────────┤
│ YANG HANYA BOLEH DI SERVER:        │ YANG AMAN DI CLIENT:          │
│  - Logika harga & diskon           │  - Theme preference           │
│  - Validasi final                  │  - UI state                   │
│  - Akses database                  │  - Cache publik               │
│  - Secret keys & tokens            │  - Form input sementara       │
│  - Role & permission check         │  - Cart (tampilan saja)       │
└────────────────────────────────────┴────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Identifikasi SSR.** Buka `olshop-koneksi.test/shop`. Klik kanan → View Page Source. Apakah HTML produk sudah ada? Atau kosong dan diisi JS?

2. **Test client validation.** Buka form checkout. Isi dengan data salah (email tanpa @). Apakah browser mencegah submit? Matikan JS, coba lagi.

3. **Manipulasi client.** Buka F12 → Console. Ubah harga produk di halaman:
   ```javascript
   document.querySelector('.price').textContent = 'Rp 100';
   ```
   Apakah ini mengubah harga di database? Kenapa?

4. **Cek CartService.** Buka `app/Services/CartService.php`. Method `addItem` menerima parameter apa? Apakah harga diterima dari client atau diambil dari database?

5. **Debate: SSR vs CSR.** Menurutmu, untuk aplikasi seperti Koneksi Store (e-commerce), mana yang lebih cocok: SSR atau CSR? Kenapa?

---

## 🔗 Referensi

- [MDN: Client-Side vs Server-Side](https://developer.mozilla.org/en-US/docs/Learn/Server-side/First_steps/Client-Server_overview)
- [Laravel: Validation](https://laravel.com/docs/11.x/validation)
- [Laravel: FormRequest](https://laravel.com/docs/11.x/validation#form-request-validation)
- [OWASP: Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- Codebase: `app/Services/CartService.php` — harga dari DB, bukan client
- Codebase: `app/Http/Requests/CheckoutRequest.php` — validasi server
