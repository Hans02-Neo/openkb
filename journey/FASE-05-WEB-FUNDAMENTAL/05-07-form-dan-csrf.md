# 05-07: Form dan CSRF — Input dari User ke Server

> **Fase**: 5 — Web Fundamental  
> **Prasyarat**: 05-06-client-vs-server  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: form, input, method spoofing, CSRF, XSS, file upload, validation, error handling, old input

---

## 📋 Ringkasan

Form adalah jembatan antara **user** dan **server** — cara user mengirim data ke aplikasi. Tapi form juga adalah **titik masuk paling rawan** — CSRF, XSS, injection. Dokumen ini membahas form HTML, method spoofing, CSRF protection, dan penanganan error di Laravel.

**Target pemahaman:**
- Kamu paham struktur form HTML
- Kamu bisa form dengan PUT/DELETE di Laravel
- Kamu paham CSRF dan bagaimana Laravel melindunginya
- Kamu bisa menampilkan error validasi di Blade

---

## 1. Form HTML — Dasar

### 1.1 Struktur Form

```html
<form action="/checkout" method="POST">
    @csrf
    <div class="mb-3">
        <label for="name" class="form-label">Nama</label>
        <input type="text" class="form-control" id="name" name="name"
               value="{{ old('name') }}" required>
    </div>

    <div class="mb-3">
        <label for="email" class="form-label">Email</label>
        <input type="email" class="form-control" id="email" name="email"
               value="{{ old('email') }}" required>
    </div>

    <div class="mb-3">
        <label for="message" class="form-label">Pesan</label>
        <textarea class="form-control" id="message" name="message"
                  rows="3">{{ old('message') }}</textarea>
    </div>

    <button type="submit" class="btn btn-primary">Kirim</button>
</form>

<!-- Input type yang umum:
     text, email, password, number, date, file,
     checkbox, radio, hidden, submit -->
```

### 1.2 Attribute Input Penting

| Attribute | Guna | Contoh |
|-----------|------|--------|
| `name` | Key saat dikirim ke server | `name="email"` → `$_POST['email']` |
| `value` | Nilai default | `value="{{ old('name') }}"` |
| `placeholder` | Teks petunjuk | `placeholder="Masukkan email"` |
| `required` | Wajib diisi (HTML5) | `required` |
| `disabled` | Nonaktif (tidak dikirim) | `disabled` |
| `readonly` | Tidak bisa diedit (tetap dikirim) | `readonly` |
| `min` / `max` | Batas angka/tanggal | `min="1" max="100"` |
| `maxlength` | Batas karakter | `maxlength="255"` |
| `accept` | Filter file upload | `accept="image/*"` |
| `multiple` | Multiple file | `multiple` |

### 1.3 HTTP Methods di Form

HTML form hanya mendukung **GET** dan **POST**:

```html
<form method="GET" action="/shop">
    <input name="search" placeholder="Cari produk...">
    <button type="submit">Cari</button>
</form>
<!-- Hasil: /shop?search=laptop -->
```

Untuk PUT, PATCH, DELETE — Laravel menggunakan **method spoofing**.

---

## 2. Method Spoofing — PUT/DELETE via Form

### 2.1 Masalah

HTML form hanya bisa GET dan POST. Tahu gimana caranya route PUT dan DELETE?

### 2.2 Solusi: `@method`

Laravel menyediakan directive `@method` yang membuat hidden input `_method`:

```html
{{-- Form untuk UPDATE cart item --}}
<form action="/cart/5" method="POST">
    @csrf
    @method('PUT')  {{-- Spoof metode PUT --}}
    <input type="number" name="quantity" value="3">
    <button type="submit">Update</button>
</form>
```

Hasil HTML:
```html
<form action="/cart/5" method="POST">
    <input type="hidden" name="_token" value="abc123...">
    <input type="hidden" name="_method" value="PUT">
    <input type="number" name="quantity" value="3">
    <button type="submit">Update</button>
</form>
```

Laravel membaca `_method` dan **memperlakukan request sebagai PUT**, bukan POST.

```php
// Route tetap pakai PUT:
Route::put('/cart/{cart}', [CartController::class, 'update'])->name('cart.update');
// Dengan _method=PUT di form POST, route ini tetap cocok!
```

### 2.3 Method Spoofing di Codebase

```php
// routes/web.php
Route::put('/cart/{cart}', [CartController::class, 'update'])->name('cart.update');
Route::delete('/cart/{cart}', [CartController::class, 'destroy'])->name('cart.destroy');
```

```html
{{-- resources/views/cart/index.blade.php --}}
<form action="{{ route('cart.destroy', $item['product_id']) }}" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit" class="btn btn-danger btn-sm">Hapus</button>
</form>
```

### 2.4 Semua Method di Form

```html
<form method="POST" action="/resource">
    @csrf
    @method('GET')     {{-- Jadi GET request --}}
    @method('PUT')     {{-- Jadi PUT --}}
    @method('PATCH')   {{-- Jadi PATCH --}}
    @method('DELETE')  {{-- Jadi DELETE --}}
</form>
```

---

## 3. CSRF — Cross-Site Request Forgery

### 3.1 Apa Itu CSRF?

CSRF adalah serangan di mana **situs jahat** mengirim request ke **situs target** tanpa sepengetahuan user.

```
1. User login ke Koneksi Store (session aktif)
2. User (tanpa sadar) buka situs jahat: evil.com
3. Situs jahat kirim FORM ke Koneksi Store:
   <form action="https://olshop-koneksi.test/order/5/delete" method="POST">
   </form>
   <script>document.forms[0].submit();</script>
4. Koneksi Store terima request — karena session masih aktif,
   request dianggap SAH! Order terhapus!
```

### 3.2 Solusi: CSRF Token

Laravel melindungi dengan **CSRF token** — token unik yang:
- Dibuat per session
- Disimpan di session server
- Dikirim ke browser via `@csrf` (hidden input)
- Dicocokkan setiap POST/PUT/PATCH/DELETE request

```
Form di Koneksi Store:
<form action="/order/5/delete" method="POST">
    <input type="hidden" name="_token" value="8a3f7b2e..."> ✅ Token cocok!
</form>

Form di evil.com:
<form action="https://olshop-koneksi.test/order/5/delete" method="POST">
    <!-- Tidak ada token → ❌ Token mismatch → 419 -->
</form>
```

### 3.3 CSRF di Codebase

```php
// Semua form pakai @csrf:
<form method="POST" action="/checkout">
    @csrf
    <!-- ... -->
</form>

// Untuk AJAX, token ada di meta tag:
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
</head>

<script>
    // Kirim token di header via JavaScript:
    fetch('/checkout', {
        method: 'POST',
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        },
        body: data,
    });
</script>
```

### 3.4 Exclude CSRF (API atau Webhook)

Beberapa endpoint perlu di-exclude dari CSRF (biasanya API eksternal):

```php
// app/Http/Middleware/VerifyCsrfToken.php
class VerifyCsrfToken extends Middleware
{
    protected $except = [
        'webhook/midtrans',  // Midtrans callback — tidak punya token Laravel
        'api/*',              // API routes — pakai token sendiri
    ];
}
```

---

## 4. XSS — Cross-Site Scripting

### 4.1 Apa Itu XSS?

XSS adalah serangan memasukkan **skrip jahat** ke halaman web.

```
User input: <script>alert('Hacked!')</script>

❌ Tanpa proteksi:
<h1><script>alert('Hacked!')</script></h1>
→ Browser jalankan script — berbahaya!

✅ Dengan proteksi (escaping):
<h1>&lt;script&gt;alert('Hacked!')&lt;/script&gt;</h1>
→ Browser tampilkan sebagai teks — aman!
```

### 4.2 Blade — Auto Escaping

```blade
{{-- {{ }} → OTOMATIS escape HTML — AMAN --}}
<p>{{ $product->name }}</p>
<!-- <script> → &lt;script&gt; -->

{{-- {!! !!} → TANPA escape — BERBAHAYA! --}}
<p>{!! $product->description !!}</p>
<!-- <script> → <script> (dijalankan browser!) -->
```

**Aturan:**
- **Selalu** pakai `{{ }}` untuk data dari user/database
- Hanya pakai `{!! !!}` jika benar-benar perlu HTML (dan yakin bersih)
- Untuk HTML dari user, gunakan **HTML Purifier** atau `Str::markdown()`

### 4.3 XSS Prevention Checklist

```blade
{{-- ✅ AMAN --}}
{{ $product->title }}
{{ $product->description }}

{{-- ❌ BERBAHAYA — jika $description dari user --}}
{!! $product->description !!}

{{-- ✅ Alternatif untuk HTML yang perlu styling --}}
{!! Str::markdown($product->description) !!}
{!! strip_tags($product->description, '<b><i><u>') !!}
```

---

## 5. Form Validation dan Error

### 5.1 Menampilkan Error di Blade

```blade
{{-- resources/views/cart/checkout.blade.php --}}

{{-- Menampilkan semua error --}}
@if($errors->any())
    <div class="alert alert-danger">
        <ul class="mb-0">
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

{{-- Per-field error --}}
<div class="mb-3">
    <label for="email">Email</label>
    <input type="email" name="email"
           class="form-control @error('email') is-invalid @enderror"
           value="{{ old('email') }}">
    @error('email')
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror
</div>
```

### 5.2 Old Input — Mengisi Ulang Form

```blade
{{-- old() mengisi nilai dari request sebelumnya --}}
<input type="text" name="name" value="{{ old('name') }}">
<input type="email" name="email" value="{{ old('email') }}">
<textarea name="message">{{ old('message') }}</textarea>

<select name="category">
    <option value="1" {{ old('category') == 1 ? 'selected' : '' }}>Elektronik</option>
    <option value="2" {{ old('category') == 2 ? 'selected' : '' }}>Fashion</option>
</select>

<input type="checkbox" name="agree" value="1"
    {{ old('agree') ? 'checked' : '' }}>
```

### 5.3 FormRequest Validation

```php
// app/Http/Requests/CheckoutRequest.php
class CheckoutRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // semua user boleh checkout
    }

    public function rules(): array
    {
        return [
            'customer_name' => 'required|string|max:255',
            'email' => 'required|email|max:255',
            'phone' => 'required|string|max:20',
            'address' => 'required|string|min:10',
            'city' => 'required|string|max:100',
            'postal_code' => 'required|string|size:5',
            'items' => 'required|array|min:1',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
        ];
    }

    public function messages(): array
    {
        return [
            'customer_name.required' => 'Nama wajib diisi',
            'email.required' => 'Email wajib diisi',
            'email.email' => 'Format email tidak valid',
            'address.min' => 'Alamat minimal 10 karakter',
            'postal_code.size' => 'Kode pos harus 5 digit',
            'items.required' => 'Keranjang belanja kosong',
        ];
    }
}
```

### 5.4 Custom Error Messages di Controller

```php
// Kadang validasi manual di controller:
public function store(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'price' => 'required|numeric|min:0',
        'stock' => 'required|integer|min:0',
    ], [
        'name.required' => 'Nama produk harus diisi!',
        'price.min' => 'Harga tidak boleh negatif!',
    ]);

    Product::create($validated);
    return redirect()->route('products.index')
        ->with('success', 'Produk berhasil ditambahkan');
}
```

---

## 6. File Upload

### 6.1 Form untuk File

```html
<form action="/products" method="POST" enctype="multipart/form-data">
    @csrf

    <div class="mb-3">
        <label for="image" class="form-label">Gambar Produk</label>
        <input type="file" class="form-control" id="image" name="image"
               accept="image/*" required>
    </div>

    <div class="mb-3">
        <label for="gallery" class="form-label">Galeri (bisa banyak)</label>
        <input type="file" class="form-control" id="gallery" name="gallery[]"
               accept="image/*" multiple>
    </div>

    <button type="submit" class="btn btn-primary">Simpan</button>
</form>
```

**Penting:** `enctype="multipart/form-data"` **WAJIB** untuk upload file!

### 6.2 Memproses File di Controller

```php
public function store(ProductRequest $request)
{
    // Simpan file tunggal
    if ($request->hasFile('image')) {
        $path = $request->file('image')->store('products', 'public');
        // $path = "products/abc123.jpg"
    }

    // Simpan multiple files
    if ($request->hasFile('gallery')) {
        foreach ($request->file('gallery') as $file) {
            $path = $file->store('products/gallery', 'public');
        }
    }
}
```

### 6.3 Validasi File

```php
'image' => 'required|image|mimes:jpeg,png,jpg,webp|max:2048',
// image: file harus gambar
// mimes: ekstensi yang diizinkan
// max: ukuran maksimal (kilobytes)

'gallery.*' => 'image|mimes:jpeg,png,jpg|max:1024',
// gallery.* → validasi setiap file dalam array
```

---

## 7. Flash Message — Feedback Setelah Submit

### 7.1 with() — Flash Data ke Session

```php
// Controller — setelah sukses:
return redirect()->route('orders.index')
    ->with('success', 'Pesanan berhasil dibuat!');

// Atau error:
return redirect()->back()
    ->with('error', 'Terjadi kesalahan, coba lagi.');
```

### 7.2 Tampilkan di Blade

```blade
{{-- resources/views/layouts/app.blade.php --}}
@if(session('success'))
    <div class="alert alert-success alert-dismissible fade show">
        {{ session('success') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif

@if(session('error'))
    <div class="alert alert-danger alert-dismissible fade show">
        {{ session('error') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif
```

### 7.3 Jenis Flash Message

```php
// Biasanya pakai Bootstrap alert classes:
redirect()->route('orders.index')
    ->with('success', 'Berhasil!')
    ->with('info', 'Info')
    ->with('warning', 'Peringatan')
    ->with('error', 'Gagal!');
```

---

## 8. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ FORM HTML                                                          │
│  <form action="/path" method="POST">                              │
│      @csrf                → CSRF token                            │
│      @method('PUT')       → Method spoofing                       │
│      <input name="email"> → data dikirim                          │
│  </form>                                                           │
├─────────────────────────────────────────────────────────────────────┤
│ CSRF PROTECTION                                                    │
│  @csrf → <input type="hidden" name="_token" value="...">         │
│  Laravel cocokkan token di session dengan token di request        │
│  ❌ Token tidak cocok → 419 Page Expired                          │
├─────────────────────────────────────────────────────────────────────┤
│ VALIDASI                                                           │
│  Client: HTML5 (required, type) + JS (cepat)                      │
│  Server: FormRequest (final, aman)                                │
│  Error: @error('field'), $errors, old('field')                    │
├─────────────────────────────────────────────────────────────────────┤
│ XSS PREVENTION                                                     │
│  {{ $var }} → escape HTML — AMAN                                  │
│  {!! $var !!} → raw HTML — HATI-HATI!                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Buka form checkout.** Buka `olshop-koneksi.test/cart`, klik tombol checkout. Inspect form — cari:
   - `@csrf` — lihat value token
   - `method` — POST atau GET?
   - Input fields — name apa saja?

2. **Test CSRF.** Buka form → Inspect → hapus input `_token`. Submit. Error apa yang muncul?

3. **Method spoofing.** Cari form DELETE di cart. Buka Inspect → lihat hidden input `_method`. Apa nilainya? Route apa yang cocok?

4. **Validasi.** Submit form checkout dengan data kosong. Error apa yang muncul? Bagaimana tampilan error per field?

5. **Buat form.** Buat form sederhana di Blade dengan:
   - Input nama (text)
   - Input email (email)  
   - Select kategori
   - Submit button
   - Validasi dengan error messages

---

## 🔗 Referensi

- [MDN: Form Guide](https://developer.mozilla.org/en-US/docs/Learn/Forms)
- [Laravel: Form Method Spoofing](https://laravel.com/docs/11.x/routing#form-method-spoofing)
- [Laravel: CSRF Protection](https://laravel.com/docs/11.x/csrf)
- [Laravel: Validation](https://laravel.com/docs/11.x/validation)
- [Laravel: Error Messages](https://laravel.com/docs/11.x/validation#quick-displaying-the-validation-errors)
- [OWASP: XSS](https://owasp.org/www-community/attacks/xss/)
- Codebase: `app/Http/Middleware/VerifyCsrfToken.php`
- Codebase: `app/Http/Requests/CheckoutRequest.php`
- Codebase: `resources/views/cart/index.blade.php` — form cart
