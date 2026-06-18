# 09-03: Alpine.js — Reactive Frontend

> **Fase**: 9 — Frontend di Laravel  
> **Prasyarat**: 09-02-tailwind-css  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: alpine.js, reactive, x-data, x-show, @click, javascript, frontend interactivity

---

## 📋 Ringkasan

Alpine.js adalah **JavaScript framework minimalis** untuk menambahkan interaktivitas di halaman. Tidak perlu React/Vue — cukup tambahkan attribute di HTML. Codebase ini menggunakan Alpine.js untuk semua interaksi frontend.

**Target pemahaman:**
- Kamu paham cara kerja Alpine.js
- Kamu bisa membaca dan menulis `x-data`, `x-show`, `@click`
- Kamu bisa membuat komponen interaktif sederhana
- Kamu paham pola Alpine di codebase ini

---

## 1. Alpine.js — Filosofi

### 1.1 Tanpa Alpine

```blade
{{-- Interaktivitas butuh vanilla JS atau jQuery --}}
<button id="toggleBtn">Menu</button>
<div id="menu" class="hidden">...</div>

<script>
document.getElementById('toggleBtn').addEventListener('click', function() {
    document.getElementById('menu').classList.toggle('hidden');
});
</script>
```

### 1.2 Dengan Alpine

```blade
<div x-data="{ open: false }">
    <button @click="open = !open">Menu</button>
    <div x-show="open">...</div>
</div>
```

### 1.3 Kenapa Alpine?

| | jQuery | React/Vue | Alpine |
|---|--------|-----------|--------|
| **Setup** | Include JS | Vite/npm build | Include JS |
| **Syntax** | `$()` | JSX/SFC | HTML attribute |
| **Weight** | ~30KB | ~40KB+ | ~8KB |
| **Learn** | Mudah | Berat | Sangat mudah |
| **Use case** | Warisan | SPA besar | Interaksi ringan |

---

## 2. Setup di Codebase

```js
// resources/js/app.js
import './bootstrap';
import Alpine from 'alpinejs';

window.Alpine = Alpine;
Alpine.start();
```

Itu saja. Alpine siap digunakan di semua Blade view.

---

## 3. Directive Utama

### 3.1 `x-data` — State Deklarasi

```blade
<div x-data="{ open: false, count: 0, name: 'Budi' }">
    ...
</div>
```

State hanya berlaku di dalam elemen dan child-nya.

### 3.2 `x-show` — Tampilkan/Sembunyikan

```blade
<button @click="open = !open">Toggle</button>
<div x-show="open" x-cloak>
    Konten yang bisa di-toggle
</div>
```

**`x-cloak`:** Mencegah flash sebelum Alpine siap.

### 3.3 `x-text` — Teks Dinamis

```blade
<div x-data="{ count: 0 }">
    <p x-text="count"></p>
    <button @click="count++">Tambah</button>
</div>
```

### 3.4 `x-model` — Two-way Binding

```blade
<div x-data="{ search: '' }">
    <input type="text" x-model="search" placeholder="Cari...">
    <p>Kamu mencari: <span x-text="search"></span></p>
</div>
```

### 3.5 `x-init` — Inisialisasi

```blade
<div x-data="{ show: true }"
     x-init="setTimeout(() => show = false, 2000)">
    <div x-show="show">
        Notifikasi ini akan hilang setelah 2 detik
    </div>
</div>
```

### 3.6 `x-for` — Looping

```blade
<div x-data="{ items: ['a', 'b', 'c'] }">
    <template x-for="(item, index) in items" :key="index">
        <p x-text="item"></p>
    </template>
</div>
```

### 3.7 `x-if` — Conditional Render

```blade
<template x-if="isAdmin">
    <button>Admin Panel</button>
</template>
```

---

## 4. Event Handling

### 4.1 `@click` — Klik

```blade
<button @click="open = !open">Toggle</button>
<button @click="handleClick($event)">Handler</button>
```

### 4.2 Event Modifiers

```blade
{{-- Prevent default --}}
<a href="/" @click.prevent="alert('stopped')">Klik</a>

{{-- Stop propagation --}}
<div @click="outer">
    <button @click.stop="inner">Klik saya</button>
</div>

{{-- Key events --}}
<input @keydown.escape="open = false">
<input @keydown.enter="submit()">
```

### 4.3 Custom Events — `$dispatch`

```blade
{{-- Kirim event ke parent --}}
<button @click="$dispatch('close-modal')">Tutup</button>

{{-- Tangkap event --}}
<div @close-modal.window="show = false">
    ...
</div>
```

---

## 5. Alpine di Codebase Ini

### 5.1 Mobile Menu Toggle

```blade
{{-- components/shop-layout.blade.php --}}
<nav x-data="{ mobileOpen: false }">
    {{-- Tombol hamburger --}}
    <button @click="mobileOpen = !mobileOpen" class="sm:hidden">
        <svg x-show="!mobileOpen">...</svg>
        <svg x-show="mobileOpen">...</svg>
    </button>

    {{-- Mobile menu --}}
    <div x-show="mobileOpen" x-cloak>
        Menu items...
    </div>
</nav>
```

### 5.2 Dropdown User

```blade
<div x-data="{ open: false }">
    <button @click="open = !open">
        Akun Saya ▼
    </button>
    <div x-show="open" @click.away="open = false" x-cloak>
        <a href="/profile">Profile</a>
        <a href="/orders">Pesanan</a>
        <form method="POST" action="/logout">Logout</form>
    </div>
</div>
```

### 5.3 Product Image Gallery

```blade
{{-- shop/show.blade.php --}}
<div x-data="{ activeImage: '{{ $product->thumbnail }}', activeTab: 'deskripsi' }">
    {{-- Image gallery --}}
    <img :src="activeImage" class="w-full">
    @foreach($product->media as $media)
        <img src="{{ $media->getUrl() }}"
             @click="activeImage = '{{ $media->getUrl() }}'"
             :class="{ 'ring-2 ring-indigo-500': activeImage === '{{ $media->getUrl() }}' }">
    @endforeach

    {{-- Tab content --}}
    <button @click="activeTab = 'deskripsi'">Deskripsi</button>
    <button @click="activeTab = 'spesifikasi'">Spesifikasi</button>

    <div x-show="activeTab === 'deskripsi'">{{ $product->description }}</div>
    <div x-show="activeTab === 'spesifikasi'">...</div>
</div>
```

### 5.4 Toast Notifications (Auto Dismiss)

```blade
{{-- components/shop-layout.blade.php --}}
<div x-data="{ show: true }"
     x-init="setTimeout(() => show = false, 4000)">
    <div x-show="show" class="toast toast-success">
        {{ session('success') }}
        <button @click="show = false">&times;</button>
    </div>
</div>
```

### 5.5 Modal Component

```blade
{{-- components/modal.blade.php --}}
<div x-data="{ show: false }"
     x-on:open-modal.window="$event.detail == '{{ $name }}' ? show = true : null"
     x-on:close-modal.window="show = false"
     x-show="show"
     x-cloak>
    <div x-show="show" @click="show = false">Backdrop</div>
    <div x-show="show">Modal content...</div>
</div>
```

---

## 6. Magic Properties

| Property | Arti |
|----------|------|
| `$el` | Referensi ke elemen DOM |
| `$refs` | Akses elemen dengan `x-ref` |
| `$event` | Native event object |
| `$dispatch` | Kirim custom event |
| `$watch` | Pantau perubahan state |
| `$store` | Global store |

---

## 7. Ringkasan

```
┌─────────────────────────────────────────────────────────────────────┐
│ ALPINE.JS — INTERAKTIVITAS TANPA REACT                             │
├─────────────────────────────────────────────────────────────────────┤
│  x-data="{ state }"        Deklarasi state komponen                │
│  x-show="expr"             Tampilkan/sembunyikan (CSS display)     │
│  x-text="expr"             Update text content                     │
│  x-model="var"             Two-way binding (input)                 │
│  x-init="..."              Fungsi inisialisasi                     │
│  x-cloak                   Cegah flash sebelum Alpine ready        │
│  @click="..."              Event listener                          │
│  @click.away               Klik di luar elemen                     │
│  :src="expr"               Bind attribute dinamis                  │
│  $dispatch('event')        Kirim custom event                      │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE INI                                                     │
│  resources/js/app.js       Import + start Alpine                   │
│  shop-layout.blade.php     Mobile nav, dropdown, toast             │
│  shop/show.blade.php       Gallery + tab                           │
│  modal.blade.php           Modal component                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Cari Alpine.** Buka `resources/views/shop/show.blade.php`. Temukan semua directive Alpine.

2. **Buat counter.** Buat komponen counter sederhana dengan `x-data`, `x-text`, `@click`.

3. **Analisis modal.** Buka `resources/views/components/modal.blade.php`. Pahami alur `$dispatch`.

4. **Toast notification.** Buka `resources/views/components/shop-layout.blade.php`. Pahami kenapa pakai `x-init` untuk auto dismiss.

5. **Buat component dropdown sendiri.** Buat dropdown dengan Alpine:
   - `x-data="{ open: false }"`
   - `@click` untuk toggle
   - `@click.away` untuk tutup

---

## 🔗 Referensi

- [Alpine.js Docs](https://alpinejs.dev/)
- [Alpine.js Directives](https://alpinejs.dev/directives/data)
- Codebase: `resources/js/app.js`
- Codebase: `resources/views/components/shop-layout.blade.php`
- Codebase: `resources/views/components/modal.blade.php`
- Codebase: `resources/views/shop/show.blade.php`
