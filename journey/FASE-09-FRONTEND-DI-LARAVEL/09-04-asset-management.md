# 09-04: Asset Management — CSS, JS, Images, Fonts

> **Fase**: 9 — Frontend di Laravel  
> **Prasyarat**: 09-01-cara-kerja-vite  
> **Waktu baca**: 40-55 menit  
> **Kata kunci**: asset, CSS, JS, image, font, storage, public, Vite

---

## 📋 Ringkasan

Asset management mengatur bagaimana CSS, JavaScript, gambar, font, dan file statis lainnya dikelola — dari development sampai production. Di Laravel + Vite, ada beberapa pendekatan tergantung jenis aset.

**Target pemahaman:**
- Kamu paham perbedaan public/ vs resources/
- Kamu tahu cara handle CSS, JS, gambar, font
- Kamu paham storage untuk upload user
- Kamu bisa nambah aset baru

---

## 1. Lokasi Asset

### 1.1 Managed by Vite

```
resources/
├── css/app.css      ← CSS (diproses Vite + Tailwind)
└── js/app.js        ← JS (diproses Vite + Alpine)
```

File di `resources/` diproses oleh Vite → output ke `public/build/`.

### 1.2 Static (Public)

```
public/
├── favicon.ico      ← Langsung diakses via /favicon.ico
├── images/          ← Gambar statis (logo, icon)
└── storage/         ← User uploads (symlink ke storage/app/public)
```

File di `public/` bisa langsung diakses via URL `/nama-file`.

### 1.3 Storage (User Uploads)

```
storage/app/public/
├── products/        ← Gambar produk
├── payments/        ← Bukti pembayaran
└── categories/      ← Gambar kategori
```

Diakses via `Storage::url()` atau symlink `public/storage/`.

---

## 2. CSS — Cara Kerja

### 2.1 Entry Point

```css
/* resources/css/app.css */
@import "tailwindcss";

@layer components {
    .btn-primary { @apply ... }
    .card { @apply ... }
}
```

### 2.2 Di Blade

```blade
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

### 2.3 Output (Production)

```css
/* public/build/assets/app-D1x3Q2aD.css */
/* Sudah minified + purged (hanya class yang dipakai) */
```

---

## 3. JavaScript — Cara Kerja

### 3.1 Entry Point

```js
// resources/js/app.js
import './bootstrap';
import Alpine from 'alpinejs';
window.Alpine = Alpine;
Alpine.start();
```

```js
// resources/js/bootstrap.js
import axios from 'axios';
window.axios = axios;
```

### 3.2 Di Blade

```blade
@vite(['resources/js/app.js'])
```

### 3.3 Output (Production)

```js
/* public/build/assets/app-C0dE8fGh.js */
/* Minified + tree-shaken (hanya kode yang dipakai) */
```

---

## 4. Gambar

### 4.1 Gambar Statis (di public/)

Taruh di `public/images/` atau `public/`:

```
public/
├── images/
│   ├── logo.png
│   └── icon-cart.svg
└── favicon.ico
```

Akses di Blade:

```blade
<img src="/images/logo.png" alt="Logo">
<link rel="icon" href="/favicon.ico">
```

### 4.2 Gambar User Upload (storage)

File upload disimpan di `storage/app/public/`:

```php
// Controller — simpan file
$path = $request->file('image')->store('products', 'public');

// Akses di Blade
<img src="{{ Storage::url($product->image) }}" alt="Product">
```

**Prasyarat:** Symlink `public/storage → storage/app/public`.

```bash
php artisan storage:link
```

### 4.3 Media Library (Spatie)

Codebase ini pakai **Spatie Media Library**:

```php
// Model
$product->addMedia($request->file('image'))->toMediaCollection('images');

// Blade
<img src="{{ $media->getUrl() }}">
<img src="{{ $media->getUrl('thumb') }}">  {{-- conversion --}}
```

---

## 5. Font

### 5.1 Google Font (HTML)

```blade
{{-- layouts/app.blade.php --}}
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700&display=swap" rel="stylesheet">
```

### 5.2 Di CSS

```css
/* resources/css/app.css */
body {
    font-family: 'Outfit', sans-serif;
}
```

---

## 6. Summary: Asset Types

| Asset | Lokasi | Cara Panggil |
|-------|--------|-------------|
| CSS | `resources/css/app.css` | `@vite(...)` |
| JS | `resources/js/app.js` | `@vite(...)` |
| Gambar statis | `public/images/` | `/images/file.png` |
| Favicon | `public/favicon.ico` | `/favicon.ico` |
| User uploads | `storage/app/public/` | `Storage::url()` |
| Media Library | `storage/app/public/N/` | `$media->getUrl()` |
| Font | Google Fonts CDN | `<link>` di layout |
| Library (npm) | `node_modules/` | `import` di JS |

---

## 🧪 Latihan

1. **Cek CSS.** Buka `resources/css/app.css`. Pahami isinya.

2. **Cek JS.** Buka `resources/js/app.js` dan `bootstrap.js`. Apa yang di-import?

3. **Cari gambar statis.** Cek folder `public/images/` atau `public/`. Ada file apa?

4. **Cek storage.** Cek `storage/app/public/`. Folder apa yang ada?

5. **Tambah asset.** Tambahkan file CSS baru `resources/css/custom.css`, import di `app.js`, dan daftarkan di `vite.config.js`.

---

## 🔗 Referensi

- [Laravel Docs: Vite](https://laravel.com/docs/11.x/vite)
- [Laravel Docs: Filesystem](https://laravel.com/docs/11.x/filesystem)
- [Spatie Media Library](https://spatie.be/docs/laravel-medialibrary)
- Codebase: `resources/css/app.css`
- Codebase: `resources/js/app.js`
- Codebase: `resources/views/layouts/app.blade.php`
