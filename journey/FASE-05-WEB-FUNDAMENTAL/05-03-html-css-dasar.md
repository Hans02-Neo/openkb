# 05-03: HTML dan CSS Dasar — Kulit dari Web

> **Fase**: 5 — Web Fundamental  
> **Prasyarat**: 05-02-http-dan-rest  
> **Waktu baca**: 75-90 menit  
> **Kata kunci**: HTML, CSS, tag, attribute, selector, box model, flexbox, grid, responsive, blade, Bootstrap

---

## 📋 Ringkasan

PHP (backend) mengatur logika dan data. Tapi yang dilihat user adalah **tampilan** — HTML dan CSS. Di Laravel, tampilan dibuat dengan **Blade template**, yang pada akhirnya menghasilkan HTML + CSS.

**Target pemahaman:**
- Kamu paham struktur HTML: tag, attribute, element, DOM
- Kamu paham CSS: selector, box model, layout
- Kamu bisa membaca dan memodifikasi Blade template
- Kamu paham peran Bootstrap di codebase

---

## 1. HTML — Struktur Halaman

### 1.1 Anatomi HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Koneksi Store</title>
    <link rel="stylesheet" href="/css/app.css">
</head>
<body>
    <header>
        <h1>Selamat Datang di Koneksi Store</h1>
        <nav>
            <a href="/">Home</a>
            <a href="/shop">Shop</a>
            <a href="/cart">Cart</a>
        </nav>
    </header>

    <main>
        <p>Ini adalah halaman utama.</p>
    </main>

    <footer>
        <p>&copy; 2026 Koneksi Store</p>
    </footer>
</body>
</html>
```

### 1.2 Tag HTML Paling Umum

| Tag | Guna | Contoh |
|-----|------|--------|
| `<h1>` – `<h6>` | Heading (judul) | `<h1>Judul Utama</h1>` |
| `<p>` | Paragraf | `<p>Ini teks.</p>` |
| `<a>` | Link | `<a href="/shop">Belanja</a>` |
| `<img>` | Gambar | `<img src="logo.png" alt="Logo">` |
| `<ul>` / `<ol>` | List | `<ul><li>Item 1</li></ul>` |
| `<div>` | Container (block) | `<div class="wrapper">...</div>` |
| `<span>` | Inline container | `<span class="price">Rp 1000</span>` |
| `<form>` | Form | `<form method="POST">...</form>` |
| `<input>` | Input field | `<input type="text" name="email">` |
| `<button>` | Tombol | `<button type="submit">Kirim</button>` |
| `<table>` | Tabel | `<table><tr><td>Data</td></tr></table>` |

### 1.3 Attribute HTML

Attribute memberikan informasi tambahan ke tag:

```html
<a href="/shop" class="btn btn-primary" id="shop-link" target="_blank">
    Belanja Sekarang
</a>
<!--  ↑          ↑                    ↑               ↑
       href      class                id              target
       (URL)     (CSS class)          (unique ID)     (buka di tab baru)
-->
```

**Attribute umum:**
- `class` — untuk CSS dan JavaScript
- `id` — unik, untuk anchor/CSS/JS
- `name` — untuk form submission
- `value` — nilai default input
- `placeholder` — teks petunjuk di input
- `disabled` — nonaktifkan elemen
- `data-*` — data custom untuk JavaScript

### 1.4 Semantic HTML

Gunakan tag yang sesuai **makna**, bukan hanya tampilan:

| Tag | Makna | Bukan |
|-----|-------|-------|
| `<header>` | Bagian atas halaman/section | `<div class="header">` |
| `<nav>` | Navigasi | `<div class="nav">` |
| `<main>` | Konten utama | `<div class="main">` |
| `<section>` | Kelompok konten tematis | `<div class="section">` |
| `<article>` | Konten independen (blog post) | `<div class="article">` |
| `<aside>` | Sidebar, konten tambahan | `<div class="sidebar">` |
| `<footer>` | Bagian bawah halaman | `<div class="footer">` |

---

## 2. CSS — Kulit dari HTML

### 2.1 Cara Menulis CSS

```css
/* selector { property: value; } */

h1 {
    color: blue;
    font-size: 24px;
}
```

**Tiga cara pasang CSS:**

1. **Inline** — langsung di tag:
   ```html
   <h1 style="color: blue;">Judul</h1>
   ```

2. **Internal** — di `<head>` dalam `<style>`:
   ```html
   <head>
       <style>
           h1 { color: blue; }
       </style>
   </head>
   ```

3. **External** — file terpisah:
   ```html
   <head>
       <link rel="stylesheet" href="/css/app.css">
   </head>
   ```

### 2.2 CSS Selectors — Cara Menarget Elemen

| Selector | Target | Contoh |
|----------|--------|--------|
| `element` | Semua tag itu | `p { }` — semua `<p>` |
| `.class` | Semua dengan class itu | `.btn { }` — `class="btn"` |
| `#id` | Elemen dengan id itu | `#logo { }` — `id="logo"` |
| `element.class` | Tag dengan class | `a.btn { }` — `<a class="btn">` |
| `parent > child` | Langsung anak | `div > p { }` — p langsung di dalam div |
| `parent desc` | Turunan | `div p { }` — semua p di dalam div |
| `el, el` | Multiple | `h1, h2 { }` |
| `el:hover` | Hover (mouse di atas) | `a:hover { }` |
| `:first-child` | Anak pertama | `li:first-child { }` |
| `:nth-child(n)` | Anak ke-n | `li:nth-child(2) { }` — anak kedua |
| `[attr]` | Punya attribute | `[disabled] { }` |
| `[attr=val]` | Attribute = nilai | `[type="text"] { }` |

### 2.3 Spesifisitas — Rule Mana yang Menang?

```
Inline style    → 1000 poin  (paling tinggi)
#id             → 100 poin
.class          → 10 poin
element         → 1 poin
```

```html
<p id="special" class="highlight" style="color: red;">Teks</p>
```

```css
p    { color: blue; }      /* 1 poin */
.highlight { color: green; } /* 10 poin */
#special { color: yellow; }  /* 100 poin */
<!-- style="color: red" → 1000 poin — MENANG! -->
```

### 2.4 Box Model — Semua Elemen Adalah Box

```
┌──────────────────────────────────┐
│         MARGIN (luar)            │
│  ┌────────────────────────────┐  │
│  │       BORDER (garis)       │  │
│  │  ┌──────────────────────┐  │  │
│  │  │     PADDING (dalam)   │  │  │
│  │  │  ┌────────────────┐  │  │  │
│  │  │  │    CONTENT     │  │  │  │
│  │  │  │  (teks/gambar) │  │  │  │
│  │  │  └────────────────┘  │  │  │
│  │  └──────────────────────┘  │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘
```

```css
.box {
    width: 300px;
    padding: 20px;       /* ruang dalam */
    border: 2px solid black; /* garis tepi */
    margin: 10px;        /* ruang luar */
}
/* Total lebar: 300 + 40 (padding) + 4 (border) + 20 (margin) = 364px */
```

**box-sizing: border-box** — hitung padding & border di dalam width:
```css
* {
    box-sizing: border-box;
}
/* Total lebar: 300px (padding & border di dalam) */
```

---

## 3. Layout CSS

### 3.1 Display Property

```css
.block {
    display: block;      /* default div, p, h1 — ambil seluruh lebar */
}

.inline {
    display: inline;     /* default span, a — hanya selebar konten */
}

.inline-block {
    display: inline-block; /* seperti inline, tapi bisa width/height */
}

.none {
    display: none;       /* hilang dari halaman (seperti tidak ada) */
}
```

### 3.2 Flexbox — Layout 1 Dimensi

Flexbox mengatur item dalam **satu baris atau satu kolom**:

```css
.container {
    display: flex;
    justify-content: center; /* horizontal: flex-start, center, space-between */
    align-items: center;     /* vertical: flex-start, center, stretch */
    gap: 16px;               /* jarak antar item */
    flex-wrap: wrap;         /* turun baris jika penuh */
}

.item {
    flex: 1;                 /* semua item lebar sama */
    /* atau flex: 0 0 300px; — tetap 300px, tidak fleksibel */
}
```

**Flexbox di codebase (Bootstrap menggunakan flexbox):**
```html
<div class="d-flex justify-content-between align-items-center">
    <h3>Keranjang Belanja</h3>
    <span>Total: Rp 150.000</span>
</div>
```

### 3.3 Grid — Layout 2 Dimensi

Grid mengatur item dalam **baris DAN kolom**:

```css
.container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr; /* 3 kolom, sama lebar */
    grid-template-columns: 250px 1fr;   /* sidebar 250px + konten fleksibel */
    gap: 20px;
}
```

### 3.4 Posisi — Position

```css
.static  { position: static; }   /* default — mengalir normal */
.relative { position: relative; top: 10px; } /* geser dari posisi asli */
.absolute { position: absolute; top: 0; }   /* relatif ke parent terdekat yang position bukan static */
.fixed   { position: fixed; top: 0; }       /* tetap di layar — navbar sticky */
.sticky  { position: sticky; top: 0; }      /* normal sampai scroll melewatinya */
```

---

## 4. Bootstrap — CSS Framework di Codebase

### 4.1 Kenapa Bootstrap?

Bootstrap adalah CSS framework paling populer. Codebase ini menggunakan Bootstrap untuk layout dan komponen.

**Keuntungan Bootstrap:**
- **Grid system** — layout responsive dengan class
- **Pre-built components** — navbar, card, button, form, modal
- **Responsive** — mobile-friendly otomatis
- **Consistent** — tampilan sama di semua browser

### 4.2 Bootstrap Grid

```html
<div class="container">
    <div class="row">
        <div class="col-md-8">
            <!-- Konten utama — 8/12 lebar di desktop -->
        </div>
        <div class="col-md-4">
            <!-- Sidebar — 4/12 lebar di desktop -->
        </div>
    </div>
</div>
```

**Breakpoint Bootstrap:**
| Class | Lebar layar | Target |
|-------|------------|--------|
| `col-*` | <576px | Mobile |
| `col-sm-*` | ≥576px | Tablet kecil |
| `col-md-*` | ≥768px | Tablet |
| `col-lg-*` | ≥992px | Desktop |
| `col-xl-*` | ≥1200px | Desktop besar |

### 4.3 Bootstrap Utility Classes

```html
<!-- Spacing -->
<div class="m-2 p-3">margin 2, padding 3</div>
<!-- m = margin, p = padding, t/r/b/l/x/y = top/right/bottom/left/horizontal/vertical -->

<!-- Text -->
<p class="text-center text-primary fw-bold">Rata tengah, warna biru, tebal</p>

<!-- Display -->
<div class="d-none d-md-block">Hilang di mobile, muncul di tablet+</div>
<div class="d-flex justify-content-between">Flexbox dengan Bootstrap</div>

<!-- Color -->
<button class="btn btn-primary">Tombol Biru</button>
<button class="btn btn-success">Tombol Hijau</button>
<button class="btn btn-danger">Tombol Merah</button>
<button class="btn btn-warning">Tombol Kuning</button>

<!-- Card -->
<div class="card">
    <div class="card-body">
        <h5 class="card-title">Judul Card</h5>
        <p class="card-text">Isi card</p>
    </div>
</div>
```

### 4.4 Bootstrap di Codebase

File: `resources/views/shop/index.blade.php` — lihat penggunaan Bootstrap:

```html
@extends('layouts.app')
@section('content')
<div class="container">
    <div class="row">
        @foreach($products as $product)
        <div class="col-md-4 col-lg-3 mb-4">
            <div class="card h-100">
                <img src="{{ $product->image }}" class="card-img-top" alt="{{ $product->name }}">
                <div class="card-body">
                    <h5 class="card-title">{{ $product->name }}</h5>
                    <p class="card-text">{{ $product->short_description }}</p>
                    <p class="fw-bold">Rp {{ number_format($product->price, 0, ',', '.') }}</p>
                    <a href="{{ route('shop.show', $product->slug) }}" class="btn btn-primary">
                        Detail
                    </a>
                </div>
            </div>
        </div>
        @endforeach
    </div>
</div>
@endsection
```

---

## 5. Blade Template — HTML dengan PHP

### 5.1 Layout (Template Inheritance)

**layouts/app.blade.php:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title', 'Koneksi Store')</title>
    <link rel="stylesheet" href="/css/app.css">
</head>
<body>
    @include('partials.navbar')

    <main>
        @yield('content')
    </main>

    @include('partials.footer')
</body>
</html>
```

**shop/index.blade.php (menggunakan layout):**
```html
@extends('layouts.app')

@section('title', 'Shop - Koneksi Store')

@section('content')
<div class="container">
    <h1>Produk</h1>
    <!-- konten halaman -->
</div>
@endsection
```

### 5.2 Blade Directives Penting

| Directive | Guna | Contoh |
|-----------|------|--------|
| `@if` / `@else` / `@endif` | Kondisi | `@if($products->count()) ... @endif` |
| `@foreach` / `@endforeach` | Loop | `@foreach($products as $p) ... @endforeach` |
| `@for` / `@endfor` | For loop | `@for($i=0; $i<5; $i++) ... @endfor` |
| `@while` / `@endwhile` | While | `@while($condition) ... @endwhile` |
| `@include` | Include partial | `@include('partials.card')` |
| `@extends` | Extend layout | `@extends('layouts.app')` |
| `@section` / `@yield` | Section | `@section('content') ... @endsection` |
| `@csrf` | CSRF token | `<form> @csrf </form>` (hasil: `<input type="hidden" name="_token" value="...">`) |
| `@method` | Spoof method | `@method('PUT')` (hasil: `<input type="hidden" name="_method" value="PUT">`) |
| `{{ }}` | Echo (escaped) | `{{ $product->name }}` (aman — HTML di-escape) |
| `{!! !!}` | Echo (raw) | `{!! $product->description !!}` (berbahaya — XSS risk!) |

### 5.3 Blade Components

```html
{{-- resources/views/components/product-card.blade.php --}}
<div class="card">
    <img src="{{ $image }}" alt="{{ $name }}">
    <div class="card-body">
        <h5>{{ $name }}</h5>
        <p>{{ $price }}</p>
        {{ $slot }} {{-- konten yang ditempatkan di antara tag --}}
    </div>
</div>

{{-- Menggunakan component --}}
<x-product-card :name="$product->name" :price="$product->price" :image="$product->image">
    <a href="/product/{{ $product->id }}">Detail</a>
</x-product-card>
```

---

## 6. Frontend Assets di Codebase

### 6.1 Struktur Asset

```
resources/
├── views/           → Blade templates
├── css/             → CSS files
├── js/              → JavaScript files
└──── image/            → Gambar (biasanya di public/)

public/
├── css/             → Compiled CSS
├── js/              → Compiled JS
├── storage/         → Symlink ke storage/app/public
└── index.php        → Entry point Laravel
```

### 6.2 Asset Compilation (Laravel Mix / Vite)

```php
// Di Blade, asset dipanggil via helper:
<link rel="stylesheet" href="{{ asset('css/app.css') }}">
<script src="{{ asset('js/app.js') }}"></script>

// Atau via Vite (Laravel 11 default):
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

### 6.3 Custom CSS vs Bootstrap

Gunakan Bootstrap untuk **layout dan komponen umum**. Custom CSS untuk **tampilan spesifik**:

```css
/* resources/css/app.css — custom styles supplementing Bootstrap */
.product-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0,0,0,0.1);
    transition: all 0.3s ease;
}

.cart-badge {
    position: absolute;
    top: -8px;
    right: -8px;
    background: red;
    color: white;
    border-radius: 50%;
    padding: 2px 6px;
    font-size: 12px;
}
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ HTML (Struktur)           │ CSS (Tampilan)                          │
├───────────────────────────┼─────────────────────────────────────────┤
│ <html>                   │ selector { property: value; }           │
│   <head>                 │ h1 { color: blue; }                     │
│   <body>                 │ .card { border: 1px solid #ddd; }       │
│     <header>             │ #logo { width: 100px; }                 │
│     <main>               │                                         │
│     <footer>             │ Box Model: content → padding → border   │
│ </html>                  │            → margin                     │
│                           │                                         │
│ Tags: h1, p, a, div,     │ Layout: flexbox, grid, position         │
│       img, ul, form      │                                         │
└───────────────────────────┴─────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────────┐
│ BOOTSTRAP DI CODEBASE                                              │
├─────────────────────────────────────────────────────────────────────┤
│ Layout:  container → row → col-md-4 col-lg-3                       │
│        "Biar responsive tanpa custom CSS"                          │
│ Komponen: card, navbar, button, form, modal, badge                 │
│ Utility:  d-flex, text-center, m-2, p-3, fw-bold                  │
├─────────────────────────────────────────────────────────────────────┤
│ BLADE TEMPLATE                                                     │
│  @extends('layouts.app')    → template inheritance                 │
│  @section('content')        → mengisi bagian layout               │
│  @foreach($products as $p)  → loop data                           │
│  {{ $p->name }}             → echo aman                           │
│  @csrf                      → CSRF token                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Buka Blade template.** Buka `resources/views/shop/index.blade.php`. Identifikasi:
   - Layout mana yang dipakai?
   - Bootstrap class apa yang digunakan?
   - Bagaimana loop produk dilakukan?

2. **Modifikasi tampilan.** Ubah warna tombol "Detail" dari `btn-primary` (biru) jadi `btn-success` (hijau) di shop/index.blade.php. Reload halaman.

3. **Buat component.** Buat file `resources/views/components/alert.blade.php`:
   ```html
   <div class="alert alert-{{ $type ?? 'info' }}">
       {{ $slot }}
   </div>
   ```
   Gunakan di halaman: `<x-alert type="danger">Error!</x-alert>`

4. **CSS box model.** Buka browser → Inspect element → pilih produk card. Lihat tab Computed → box model (padding, border, margin).

5. **Responsive test.** Buka `olshop-koneksi.test/shop`. Resize browser lebar → sempit. Bagaimana grid berubah? (col-lg-3 → col-md-4 → full width di mobile)

---

## 🔗 Referensi

- [MDN: HTML](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [MDN: CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [Bootstrap 5 Docs](https://getbootstrap.com/docs/5.3/getting-started/introduction/)
- [Laravel Blade Docs](https://laravel.com/docs/11.x/blade)
- Codebase: `resources/views/layouts/app.blade.php` — layout utama
- Codebase: `resources/views/shop/index.blade.php` — halaman shop
- Codebase: `resources/views/cart/index.blade.php` — halaman cart
