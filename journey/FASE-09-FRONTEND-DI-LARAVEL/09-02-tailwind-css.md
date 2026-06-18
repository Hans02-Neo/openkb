# 09-02: Tailwind CSS — Utility-First Styling

> **Fase**: 9 — Frontend di Laravel  
> **Prasyarat**: 09-01-cara-kerja-vite  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: tailwind, utility-first, CSS, design system, responsive, utility class

---

## 📋 Ringkasan

Tailwind CSS adalah **utility-first CSS framework**. Alih-alih menulis CSS kustom, kamu menggunakan class utility yang sudah jadi. Tailwind v4 di codebase ini menggunakan sintaks baru dengan `@import "tailwindcss"`.

**Target pemahaman:**
- Kamu paham konsep utility-first
- Kamu bisa membaca dan menulis Tailwind class
- Kamu paham komponen kustom di codebase ini
- Kamu bisa membuat utility class sendiri

---

## 1. Utility-First CSS

### 1.1 Traditional CSS (Selector-based)

```css
/* Kamu buat class dulu */
.btn-primary {
    background-color: #6366f1;
    color: white;
    padding: 12px 24px;
    border-radius: 999px;
}
```

```html
<button class="btn-primary">Beli</button>
```

### 1.2 Utility-First (Tailwind)

```html
<!-- Class langsung di HTML -->
<button class="bg-indigo-600 text-white px-6 py-3 rounded-full font-semibold hover:bg-indigo-700">
    Beli
</button>
```

### 1.3 Kenapa Utility-First?

| | Traditional | Utility-First |
|---|------------|---------------|
| Naming | Pusing mikir nama class | Pakai utility langsung |
| File | CSS terpisah | Semua di HTML |
| Perubahan | Edit CSS file | Edit class di HTML |
| Konsistensi | Tergantung disiplin | Built-in design system |
| Ukuran file | Bertambah terus | Purged (hanya yang dipakai) |

---

## 2. Sintaks Dasar Tailwind

### 2.1 Pattern

```
{property}-{value}
```

**Contoh:**

| Class | CSS Output |
|-------|-----------|
| `bg-indigo-600` | `background-color: #6366f1;` |
| `text-white` | `color: #ffffff;` |
| `px-6` | `padding-left: 1.5rem; padding-right: 1.5rem;` |
| `py-3` | `padding-top: 0.75rem; padding-bottom: 0.75rem;` |
| `rounded-full` | `border-radius: 9999px;` |
| `font-semibold` | `font-weight: 600;` |
| `hover:bg-indigo-700` | `:hover { background-color: #4338ca; }` |

### 2.2 Responsive Design

```
{breakpoint}:{utility}
```

```html
<div class="text-sm md:text-base lg:text-lg">
    Responsive text
</div>
```

| Prefix | Min-width |
|--------|-----------|
| `sm:` | 640px |
| `md:` | 768px |
| `lg:` | 1024px |
| `xl:` | 1280px |
| `2xl:` | 1536px |

### 2.3 State Variants

```html
<button class="bg-indigo-600 hover:bg-indigo-700 active:scale-95 focus:ring-2">
    Hover, Active, Focus
</button>
```

| Variant | CSS State |
|---------|-----------|
| `hover:` | `:hover` |
| `focus:` | `:focus` |
| `active:` | `:active` |
| `disabled:` | `:disabled` |
| `group-hover:` | Parent hover |

---

## 3. Tailwind di Codebase Ini

### 3.1 Entry File

```css
/* resources/css/app.css */
@import "tailwindcss";

/* Custom components */
@layer components {
    .btn-primary { ... }
    .btn-secondary { ... }
    .card { ... }
    .card-shop { ... }
    .badge-success { ... }
    .input { ... }
    .toast { ... }
    .product-card { ... }
    .section-label { ... }
    .section-title { ... }
}
```

### 3.2 Custom Components

Codebase ini mendefinisikan komponen kustom di `@layer components`:

```css
.btn-primary {
    @apply inline-flex items-center justify-center gap-2 px-6 py-3
           bg-indigo-600 text-white font-semibold rounded-full
           hover:bg-indigo-700 active:scale-95 transition-all duration-150
           shadow-lg shadow-indigo-200 focus:outline-none
           focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2;
}

.card {
    @apply bg-white rounded-2xl border border-gray-100 shadow-sm;
}

.badge-success {
    @apply inline-flex items-center px-2.5 py-0.5 rounded-full
           text-xs font-semibold bg-green-100 text-green-700;
}
```

**Penting:** `@apply` menggabungkan utility class ke dalam satu class kustom.

### 3.3 Contoh di Blade

```blade
{{-- resources/views/shop/index.blade.php --}}
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
    @foreach($products as $product)
        <div class="product-card">
            <img src="{{ $product->thumbnail }}" class="w-full h-48 object-cover">
            <div class="p-4">
                <h3 class="font-semibold text-gray-900">{{ $product->name }}</h3>
                <p class="text-indigo-600 font-bold mt-2">
                    Rp {{ number_format($product->price) }}
                </p>
            </div>
        </div>
    @endforeach
</div>
```

---

## 4. Layout Patterns

### 4.1 Flexbox

```blade
<div class="flex items-center justify-between">
    <h1 class="text-xl font-bold">Judul</h1>
    <button class="btn-primary">Tombol</button>
</div>
```

| Class | Arti |
|-------|------|
| `flex` | `display: flex` |
| `items-center` | `align-items: center` |
| `justify-between` | `justify-content: space-between` |
| `gap-4` | `gap: 1rem` |
| `flex-wrap` | `flex-wrap: wrap` |

### 4.2 Grid

```blade
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
    <div>Item 1</div>
    <div>Item 2</div>
    <div>Item 3</div>
    <div>Item 4</div>
</div>
```

### 4.3 Spacing

```blade
{{-- Margin --}}
<div class="m-4">margin semua sisi</div>
<div class="mx-auto">margin horizontal auto (center)</div>
<div class="mt-8">margin-top: 2rem</div>
<div class="mb-4">margin-bottom: 1rem</div>

{{-- Padding --}}
<div class="p-6">padding semua sisi</div>
<div class="px-4 py-2">padding horizontal + vertikal</div>
```

---

## 5. Tailwind v4 — Fitur Baru

### 5.1 Sintaks Baru

```css
/* Tailwind v3 */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Tailwind v4 (codebase ini) */
@import "tailwindcss";
```

### 5.2 Plugin di Vite

```js
// Tailwind v3: postcss.config.js + tailwind.config.js
// Tailwind v4: langsung plugin Vite
import tailwindcss from '@tailwindcss/vite';
```

---

## 6. Ringkasan Class Sering Dipakai

| Kategori | Class |
|----------|-------|
| **Display** | `flex`, `grid`, `hidden`, `block`, `inline-block` |
| **Width** | `w-full`, `w-1/2`, `max-w-7xl`, `max-w-md` |
| **Text** | `text-sm`, `text-lg`, `text-xl`, `font-bold`, `text-center` |
| **Color** | `bg-gray-100`, `text-indigo-600`, `border-gray-200` |
| **Spacing** | `p-4`, `px-6`, `py-3`, `m-2`, `mx-auto`, `gap-4` |
| **Border** | `border`, `rounded-full`, `rounded-2xl`, `border-b` |
| **Shadow** | `shadow-sm`, `shadow-lg`, `shadow-xl` |
| **Hover** | `hover:bg-indigo-700`, `hover:shadow-xl` |
| **Responsive** | `sm:`, `md:`, `lg:`, `xl:` |

---

## 🧪 Latihan

1. **Baca CSS.** Buka `resources/css/app.css`. Identifikasi semua komponen kustom yang didefinisikan.

2. **Eksplorasi.** Buka halaman shop. Catat 10 class Tailwind yang berbeda. Tebak CSS yang dihasilkan.

3. **Buat komponen.** Tambahkan komponen `.btn-outline-indigo` di CSS:
   - Border indigo, text indigo, hover filled

4. **Responsive.** Cari contoh responsive design di view (class dengan `sm:`, `md:`, `lg:`).

5. **@apply.** Ambil satu komponen dari CSS, coba tulis ulang tanpa @apply (pakai utility langsung di HTML).

---

## 🔗 Referensi

- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [Tailwind Cheat Sheet](https://tailwindcomponents.com/cheatsheet/)
- Codebase: `resources/css/app.css`
- Codebase: `resources/views/shop/`
- Codebase: `resources/views/components/`
