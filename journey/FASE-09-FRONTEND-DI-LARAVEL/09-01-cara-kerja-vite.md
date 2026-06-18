# 09-01: Cara Kerja Vite — Build Tool Frontend

> **Fase**: 9 — Frontend di Laravel  
> **Prasyarat**: FASE 6 (Laravel Deep Dive)  
> **Waktu baca**: 40-55 menit  
> **Kata kunci**: vite, bundler, build tool, HMR, ESM, laravel-vite-plugin

---

## 📋 Ringkasan

Vite adalah **build tool** untuk frontend. Di Laravel, Vite menggantikan Laravel Mix / Webpack. Vite mengelola CSS, JavaScript, dan aset lainnya — menggabungkan, mengoptimasi, dan menyajikannya ke browser.

**Target pemahaman:**
- Kamu paham peran Vite di Laravel
- Kamu bisa membaca konfigurasi Vite
- Kamu paham `@vite()` directive
- Kamu bisa menjalankan dev server

---

## 1. Kenapa Vite?

### 1.1 Sebelum Vite (Laravel Mix / Webpack)

```
Edit file → tunggu Webpack build (30-60 detik) → refresh → lihat hasil
```

### 1.2 Dengan Vite

```
Edit file → <1 detik (HMR) → lihat hasil langsung di browser
```

**Perbedaan:**

| | Mix / Webpack | Vite |
|---|--------------|------|
| Dev server | Tidak ada (file-based) | Ada (server-based) |
| HMR | ❌ (full refresh) | ✅ (hot module replacement) |
| Speed | Lambat (bundle dulu) | Cepat (native ESM) |
| Build | Webpack | Rollup (optimized) |
| Konfigurasi | `webpack.mix.js` | `vite.config.js` |

---

## 2. Vite + Laravel

### 2.1 File Konfigurasi

```js
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        tailwindcss(),
    ],
});
```

### 2.2 Entry Points

Vite tahu file mana yang perlu diproses dari `input`:

```js
input: [
    'resources/css/app.css',   // CSS entry
    'resources/js/app.js',     // JS entry
]
```

### 2.3 @vite() Directive

Di Blade, gunakan `@vite()` untuk inject CSS/JS:

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html>
<head>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    @yield('content')
</body>
</html>
```

**Yang dilakukan `@vite()`:**
- **Dev mode:** inject script untuk HMR + module
- **Production:** inject link ke file hasil build (versi hashed)

---

## 3. Dev vs Production

### 3.1 Dev Mode (Development)

```
npm run dev
```

Vite menjalankan server di `http://localhost:5173`:

```
vite dev server running at:
  ➜ Local:   http://localhost:5173/
  ➜ Network: http://192.168.x.x:5173/
```

- File tidak di-minify
- Source maps tersedia
- HMR aktif
- `@vite()` inject: `<script type="module" src="http://localhost:5173/...">`

### 3.2 Build Mode (Production)

```
npm run build
```

Vite menghasilkan file di `public/build/`:

```
public/build/
├── assets/
│   ├── app-D1x3Q2aD.css    ← CSS minified + hashed
│   └── app-C0dE8fGh.js     ← JS minified + hashed
└── manifest.json            ← mapping original → built
```

- File di-minify
- Hash di filename (cache busting)
- `@vite()` inject: `<link rel="stylesheet" href="/build/assets/app-xxx.css">`

---

## 4. Scripts di package.json

```json
{
    "scripts": {
        "dev": "vite",          // Jalankan dev server
        "build": "vite build"   // Build untuk production
    }
}
```

**Usage:**
```
npm run dev     → development (HMR aktif)
npm run build   → production (file di public/build/)
```

---

## 5. Vite di Codebase Ini

### 5.1 Konfigurasi

```js
// vite.config.js
plugins: [
    laravel({
        input: ['resources/css/app.css', 'resources/js/app.js'],
        refresh: true,
    }),
    tailwindcss(),
]
```

### 5.2 Entry Files

```
resources/
├── css/
│   └── app.css        ← Tailwind CSS + custom components
└── js/
    ├── app.js          ← Alpine.js import + start
    └── bootstrap.js    ← Axios setup
```

### 5.3 Template

```blade
{{-- resources/views/layouts/app.blade.php --}}
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

---

## 6. Vite Commands

```bash
# Development (saat coding)
npm run dev
# → Jalankan di terminal terpisah
# → Buka browser, edit file, lihat perubahan real-time

# Build (sebelum deploy)
npm run build
# → File siap di public/build/
# → Commit ke git

# Preview build (test production build lokal)
npx vite preview
```

---

## 🧪 Latihan

1. **Jalankan.** Buka terminal, jalankan `npm run dev`. Apa yang muncul?

2. **Cek response.** Buka aplikasi di browser, inspect `<head>`. Apa yang di-inject oleh `@vite()`?

3. **Build.** Jalankan `npm run build`. Cek isi `public/build/assets/`. Ada file apa saja?

4. **Manifest.** Buka `public/build/manifest.json`. Pahami mapping-nya.

5. **Analisis.** Kenapa file di `public/build/` pakai hash di nama file?

---

## 🔗 Referensi

- [Laravel Docs: Vite](https://laravel.com/docs/11.x/vite)
- [Vite Official](https://vitejs.dev/)
- [laravel-vite-plugin](https://github.com/laravel/vite-plugin)
- Codebase: `vite.config.js`
- Codebase: `package.json` (scripts)
- Codebase: `resources/views/layouts/app.blade.php` (penggunaan @vite)
