# 06-06: Blade Template Engine — Tampilan yang Cerdas

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-05-model-dan-eloquent-orm  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: Blade, template inheritance, component, directive, section, yield, include, slot, stack

---

## 📋 Ringkasan

Blade adalah template engine Laravel. Ia mengubah file `.blade.php` menjadi HTML murni. Blade punya fitur seperti layout inheritance, component, directive, dan escaping otomatis.

**Target pemahaman:**
- Kamu paham template inheritance (@extends, @section, @yield)
- Kamu bisa membuat dan menggunakan Blade component
- Kamu paham directive penting (@if, @foreach, @csrf, dll)
- Kamu bisa menggunakan slot dan stack

---

## 1. Template Inheritance — Layout Induk

### 1.1 Layout Utama

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>@yield('title', 'Koneksi Store')</title>
    @vite(['resources/css/app.css'])
    @stack('styles')
</head>
<body>
    @include('partials.navbar')

    <main class="container py-4">
        @yield('content')
    </main>

    @include('partials.footer')

    @vite(['resources/js/app.js'])
    @stack('scripts')
</body>
</html>
```

### 1.2 Halaman Menggunakan Layout

```blade
{{-- resources/views/shop/index.blade.php --}}
@extends('layouts.app')

@section('title', 'Shop - Koneksi Store')

@section('content')
    <h1>Semua Produk</h1>
    <div class="row">
        @foreach($products as $product)
            <div class="col-md-4 col-lg-3 mb-4">
                @include('partials.product-card', ['product' => $product])
            </div>
        @endforeach
    </div>

    {{ $products->links() }}
@endsection

@push('scripts')
    <script>
        // Script spesifik halaman ini
    </script>
@endpush
```

### 1.3 @yield vs @section vs @show

```blade
{{-- @yield — tempatkan section dari child --}}
@yield('content')
@yield('title', 'Default Title')  {{-- dengan default --}}

{{-- @section — isi section --}}
@section('content')
    <p>Ini konten</p>
@endsection  {{-- atau @stop --}}

{{-- @section + @show — isi dan tampilkan langsung (bukan di layout) --}}
@section('sidebar')
    <p>Sidebar</p>
@show  {{-- @show = @endsection + @yield --}}
```

---

## 2. Blade Components

### 2.1 Class-Based Component (Laravel 7+)

```bash
php artisan make:component Alert
# Membuat:
# app/View/Components/Alert.php
# resources/views/components/alert.blade.php
```

```php
<?php
// app/View/Components/Alert.php
namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class Alert extends Component
{
    public function __construct(
        public string $type = 'info',
        public ?string $message = null,
    ) {}

    public function shouldRender(): bool
    {
        return $this->message !== null;
    }

    public function render(): View
    {
        return view('components.alert');
    }
}
```

```blade
{{-- resources/views/components/alert.blade.php --}}
<div class="alert alert-{{ $type }} alert-dismissible fade show" role="alert">
    {{ $message ?? $slot }}
    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
</div>
```

```blade
{{-- Menggunakan component --}}
<x-alert type="success" message="Berhasil!">
    {{-- Atau pakai slot: --}}
    <x-alert type="danger">
        Terjadi kesalahan!
    </x-alert>
</x-alert>
```

### 2.2 Anonymous Component

```blade
{{-- resources/views/components/product-card.blade.php --}}
{{-- Tanpa class — cukup blade file --}}
<div class="card h-100">
    <img src="{{ $image ?? 'https://placehold.co/300' }}" class="card-img-top" alt="{{ $name }}">
    <div class="card-body">
        <h5 class="card-title">{{ $name }}</h5>
        <p class="card-text">{{ $slot }}</p>
        @isset($price)
            <p class="fw-bold">Rp {{ number_format($price, 0, ',', '.') }}</p>
        @endisset
    </div>
</div>
```

```blade
{{-- Menggunakan --}}
<x-product-card :name="$product->name" :price="$product->price">
    {{ $product->short_description }}
</x-product-card>
```

### 2.3 Component Attributes

```blade
{{-- Component --}}
<div {{ $attributes->merge(['class' => 'card mb-3']) }}>
    {{ $slot }}
</div>

{{-- Saat dipakai --}}
<x-card class="bg-dark text-white" id="special-card">
    Konten
</x-card>
{{-- Hasil: <div class="card mb-3 bg-dark text-white" id="special-card"> --}}
```

---

## 3. Blade Directives

### 3.1 Control Structure

```blade
{{-- If / Elseif / Else --}}
@if($products->count() > 0)
    <p>Ada {{ $products->count() }} produk.</p>
@elseif($isLoading)
    <p>Memuat...</p>
@else
    <p>Tidak ada produk.</p>
@endif

{{-- Unless (if not) --}}
@unless(auth()->check())
    <p>Silakan login.</p>
@endunless

{{-- Switch --}}
@switch($status)
    @case('pending')
        <span class="badge bg-warning">Pending</span>
        @break
    @case('paid')
        <span class="badge bg-success">Paid</span>
        @break
    @default
        <span class="badge bg-secondary">{{ $status }}</span>
@endswitch
```

### 3.2 Loops

```blade
{{-- Foreach --}}
@foreach($products as $product)
    <p>{{ $product->name }}</p>
@empty
    <p>Tidak ada produk.</p>
@endforeach

{{-- For --}}
@for($i = 0; $i < 10; $i++)
    <span>{{ $i }}</span>
@endfor

{{-- While --}}
@while($condition)
    <p>Looping...</p>
@endwhile

{{-- Loop variable --}}
@foreach($products as $product)
    @if($loop->first) <div class="row"> @endif
    @if($loop->last) </div> @endif
    <p>Item {{ $loop->iteration }} dari {{ $loop->count }}</p>
    {{-- $loop->first, $loop->last, $loop->iteration --}}
    {{-- $loop->index, $loop->remaining, $loop->count --}}
    {{-- $loop->parent  (jika nested loop) --}}
@endforeach
```

### 3.3 Form Directives

```blade
{{-- CSRF token --}}
<form method="POST" action="/checkout">
    @csrf
    {{-- Hasil: <input type="hidden" name="_token" value="..."> --}}
</form>

{{-- Method spoofing --}}
<form method="POST" action="/cart/5">
    @csrf
    @method('DELETE')
    {{-- Hasil: <input type="hidden" name="_method" value="DELETE"> --}}
    <button type="submit">Hapus</button>
</form>
```

### 3.4 Include

```blade
{{-- Include partial — data dari parent --}}
@include('partials.navbar')

{{-- Include dengan data tambahan --}}
@include('partials.product-card', ['product' => $product])

{{-- Include if exists --}}
@includeIf('partials.ads')

{{-- Include when true --}}
@includeWhen($showAds, 'partials.ads')

{{-- Include first yang ada --}}
@includeFirst(['partials.custom-nav', 'partials.navbar'])
```

---

## 4. Slot dan Nested Component

### 4.1 Named Slots

```blade
{{-- resources/views/components/card.blade.php --}}
<div class="card">
    <div class="card-header">
        {{ $header }}
    </div>
    <div class="card-body">
        {{ $slot }} {{-- default slot --}}
    </div>
    @isset($footer)
        <div class="card-footer">
            {{ $footer }}
        </div>
    @endisset
</div>
```

```blade
<x-card>
    <x-slot:header>
        <h3>Judul Card</h3>
    </x-slot:header>

    <p>Ini konten utama.</p>

    <x-slot:footer>
        <small>Footer</small>
    </x-slot:footer>
</x-card>
```

### 4.2 Component dengan Data dari Controller

```php
// Controller — kirim data ke component via view
return view('shop.index', [
    'products' => $products,
    'categories' => $categories,
]);
```

```blade
{{-- Loop component --}}
@foreach($products as $product)
    <x-product-card
        :name="$product->name"
        :price="$product->price"
        :image="$product->getFirstMediaUrl('images')"
        :slug="$product->slug"
    >
        {{ $product->short_description }}
    </x-product-card>
@endforeach
```

---

## 5. Stack — Push Script ke Layout

### 5.1 @stack dan @push

```blade
{{-- layouts/app.blade.php --}}
<head>
    @stack('styles')
</head>
<body>
    @stack('scripts')
</body>
```

```blade
{{-- shop/index.blade.php --}}
@push('styles')
    <link rel="stylesheet" href="/css/shop.css">
@endpush

@push('scripts')
    <script src="/js/shop.js"></script>
@endpush

{{-- Multiple push — urutan sesuai push --}}
@push('scripts')
    <script>
        console.log('Pertama');
    </script>
@endpush

@push('scripts')
    <script>
        console.log('Kedua');
    </script>
@endpush
```

### 5.2 @prepend — Push ke Awal Stack

```blade
@prepend('styles')
    {{-- Akan muncul SEBELUM push lain --}}
    <link rel="stylesheet" href="/css/critical.css">
@endprepend
```

---

## 6. Echo — {{ }} vs {!! !!}

```blade
{{-- {{ }} — escape HTML (AMAN) --}}
{{ $product->name }}
{{-- <script>alert('xss')</script> → &lt;script&gt; --}}

{{-- {!! !!} — raw HTML (BERBAHAYA) --}}
{!! $product->description !!}
{{-- <script>alert('xss')</script> → script JALAN! --}}

{{-- @verbatim — blade tidak parse --}}
@verbatim
    {{ ini bukan blade, ini Vue.js }}
    @if (ini Vue, bukan PHP)
@endverbatim
```

---

## 7. Blade di Codebase

### 7.1 Struktur View

```
resources/views/
├── layouts/
│   └── app.blade.php          → Layout utama
├── partials/
│   ├── navbar.blade.php       → Navigasi
│   └── footer.blade.php       → Footer
├── shop/
│   ├── index.blade.php         → Halaman shop
│   └── show.blade.php          → Detail produk
├── cart/
│   └── index.blade.php         → Halaman cart
├── checkout/
│   └── index.blade.php         → Form checkout
├── orders/
│   ├── index.blade.php         → Daftar order
│   └── show.blade.php          → Detail order
└── components/
    └── product-card.blade.php  → Card produk reusable
```

### 7.2 Contoh Layout Lengkap

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>@yield('title', config('app.name'))</title>
    @vite(['resources/css/app.css'])
    @stack('styles')
</head>
<body class="d-flex flex-column min-vh-100">
    @include('partials.navbar')

    <main class="flex-grow-1">
        @if(session('success'))
            <div class="alert alert-success mb-0 rounded-0">{{ session('success') }}</div>
        @endif
        @if(session('error'))
            <div class="alert alert-danger mb-0 rounded-0">{{ session('error') }}</div>
        @endif

        @yield('content')
    </main>

    @include('partials.footer')

    @vite(['resources/js/app.js'])
    @stack('scripts')
</body>
</html>
```

---

## 8. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ BLADE TEMPLATE                                                     │
├─────────────────────────────────────────────────────────────────────┤
│ LAYOUT INHERITANCE                                                 │
│  layouts/app.blade.php  (induk)                                    │
│    ├── @yield('title')  → diisi child                              │
│    ├── @yield('content') → diisi child                             │
│    └── @stack('scripts') → diisi child via @push                   │
│         ↑                                                            │
│  child.blade.php                                                    │
│    @extends('layouts.app')                                          │
│    @section('content') ... @endsection                              │
├─────────────────────────────────────────────────────────────────────┤
│ COMPONENTS                                                         │
│  Class: app/View/Components/Alert.php                              │
│  View:  resources/views/components/alert.blade.php                 │
│  Use:   <x-alert type="success">Message</x-alert>                 │
├─────────────────────────────────────────────────────────────────────┤
│ DIRECTIVES                                                         │
│  @if, @foreach, @csrf, @method, @include                          │
│  @push/@stack, @slot, @verbatim, @php                             │
├─────────────────────────────────────────────────────────────────────┤
│ ECHO                                                                │
│  {{ $var }} → escaped (AMAN)                                       │
│  {!! $var !!} → raw (HATI-HATI)                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Buka layout.** Buka `resources/views/layouts/app.blade.php`. Identifikasi:
   - @yield — section apa yang disediakan?
   - @include — partial apa yang di-include?
   - @stack — stack apa yang disediakan?
   - @if(session(...)) — flash message apa?

2. **Buka halaman shop.** Buka `resources/views/shop/index.blade.php`:
   - Apa yang di-extend?
   - @section apa yang diisi?
   - Bagaimana loop produk dilakukan?
   - Component apa yang dipakai?

3. **Buat component.** Buat `resources/views/components/button.blade.php`:
   ```blade
   <button {{ $attributes->merge(['class' => 'btn btn-' . ($type ?? 'primary')]) }}>
       {{ $slot }}
   </button>
   ```
   Pakai di view: `<x-button type="danger">Hapus</x-button>`

4. **Stack.** Tambah `@push('scripts')` di halaman shop dengan console.log. Cek apakah muncul di HTML.

5. **Edit view.** Ubah title halaman shop dari "Shop - Koneksi Store" jadi "Belanja - Koneksi Store".

---

## 🔗 Referensi

- [Laravel Docs: Blade Templates](https://laravel.com/docs/11.x/blade)
- [Laravel Docs: Blade Components](https://laravel.com/docs/11.x/blade#components)
- [Laravel Docs: Blade Directives](https://laravel.com/docs/11.x/blade#blade-directives)
- Codebase: `resources/views/layouts/app.blade.php`
- Codebase: `resources/views/shop/index.blade.php`
- Codebase: `resources/views/components/`
