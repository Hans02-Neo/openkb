# 09-06: Buildtime vs Runtime — Dua Dunia

> **Fase**: 9 — Frontend di Laravel  
> **Prasyarat**: 09-05-hot-reload-hmr  
> **Waktu baca**: 40-55 menit  
> **Kata kunci**: buildtime, runtime, compile, server-side, client-side, SSR, CSR

---

## 📋 Ringkasan

Setiap aplikasi web modern punya **2 fase eksekusi**: buildtime (sebelum deploy) dan runtime (saat user akses). Memahami perbedaan ini penting untuk troubleshooting dan optimasi.

**Target pemahaman:**
- Kamu paham buildtime vs runtime
- Kamu paham mana yang terjadi kapan
- Kamu bisa bedain server-side vs client-side
- Kamu paham implikasinya di codebase ini

---

## 1. Buildtime vs Runtime

### 1.1 Buildtime

Terjadi **sebelum** user mengakses aplikasi:

```bash
# Buildtime — developer menjalankan ini
npm run build
# → Vite proses CSS + JS
# → Minify, hash, tree-shaking
# → Output ke public/build/
```

**Yang terjadi di buildtime:**
- Vite compile Tailwind → CSS murni
- Vite bundle JavaScript (tree-shaking)
- Vite minify + hash file
- File siap di `public/build/`

### 1.2 Runtime

Terjadi **saat** user mengakses aplikasi:

```text
User buka browser → GET /shop
    │
    ▼
Laravel (PHP) — RUNTIME
    ├── Route match
    ├── Query database
    ├── Render Blade (server-side)
    └── Return HTML + link ke file build

Browser (Client) — RUNTIME
    ├── Parse HTML
    ├── Download CSS (dari public/build/)
    ├── Download JS (Alpine.js jalan)
    └── Render halaman
```

---

## 2. Buildtime Detail

### 2.1 Yang Terjadi di Buildtime

```
npm run build (atau npm run dev)
    │
    ├── Vite baca vite.config.js
    ├── Load plugin: laravel, tailwindcss
    ├── Baca entry points:
    │   ├── resources/css/app.css
    │   └── resources/js/app.js
    │
    ├── CSS pipeline:
    │   ├── @import "tailwindcss" → Tailwind generate semua utility
    │   ├── @layer components → custom components
    │   └── Purge unused CSS → hapus class tak terpakai
    │
    ├── JS pipeline:
    │   ├── Import Alpine.js
    │   ├── Import Axios
    │   ├── Tree-shaking (buang kode tak terpakai)
    │   └── Minify
    │
    └── Output:
        ├── public/build/assets/app-xxx.css
        ├── public/build/assets/app-xxx.js
        └── public/build/manifest.json
```

### 2.2 Manifest

```json
// public/build/manifest.json
{
    "resources/css/app.css": {
        "file": "assets/app-D1x3Q2aD.css",
        "src": "resources/css/app.css"
    },
    "resources/js/app.js": {
        "file": "assets/app-C0dE8fGh.js",
        "src": "resources/js/app.js"
    }
}
```

`@vite()` directive membaca manifest ini di production.

---

## 3. Runtime Detail

### 3.1 Server-Side (PHP / Laravel)

| Proses | Waktu | Output |
|--------|-------|--------|
| Route match | Runtime | Controller method |
| Database query | Runtime | Data dari MySQL |
| Blade render | Runtime | HTML |
| PHP logic | Runtime | Loop,条件, formatting |
| **Total** | **~50-200ms** | **HTML response** |

### 3.2 Client-Side (Browser)

| Proses | Waktu | Output |
|--------|-------|--------|
| Download HTML | Network | DOM |
| Download CSS | Network | Style |
| Download JS | Network | Alpine siap |
| Alpine init | Client | x-data diaktifkan |
| Render halaman | Client | Tampilan final |

---

## 4. Buildtime vs Runtime — Tabel

| Aspek | Buildtime | Runtime |
|-------|-----------|---------|
| **Kapan** | `npm run build` | User akses aplikasi |
| **CSS** | Tailwind → CSS murni, minified | Browser aplikasikan style |
| **JS** | Bundle, minify, tree-shake | Browser execute |
| **Blade** | Tidak diproses | PHP render → HTML |
| **Database** | Tidak | Query |
| **User auth** | Tidak | Login session |
| **File berada** | `public/build/` (file statis) | Server + Browser |
| **Error** | Syntax/import error | 404, 500, logic error |

---

## 5. Implikasi Praktis

### 5.1 Kenapa Ada Buildtime?

**Tailwind example:**

```blade
{{-- Kamu pakai class --}}
<div class="bg-indigo-600 text-white p-4"></div>
```

**Di buildtime:** Tailwind generate CSS untuk `bg-indigo-600`, `text-white`, `p-4`.
**Di runtime:** Browser tinggal pakai CSS yang sudah jadi.

**Kalau tidak ada buildtime:** Tailwind harus generate CSS setiap request → lambat!

### 5.2 Kenapa Build Error Muncul di Dev?

```bash
npm run dev
# Error: Cannot find module 'alpinejs'
```

Ini **buildtime error** — harus diperbaiki sebelum deploy.
Kalau tidak ketahuan, di production juga akan error.

### 5.3 Kenapa 404.css di Production?

```
npm run build  ← lupa jalanin!
```

Atau:

```
@vite(...)  ← tapi file entry tidak ada di manifest
```

---

## 6. Diagram Visual

```
═══════════════════════════════════════════════════════════════
BUILDTIME (Developer Machine)
═══════════════════════════════════════════════════════════════
                           │
  npm run build            │
      │                    │
  resources/css/app.css ───┤
  resources/js/app.js  ────┤
      │                    │
      ▼                    ▼
  Tailwind compile ─── public/build/assets/app-xxx.css
  Vite bundle       ─── public/build/assets/app-xxx.js
  Minify + Hash     ─── public/build/manifest.json
                           │
  Commit ke git ◄──────────┘
                           │
═══════════════════════════╧═══════════════════════════════════
RUNTIME (Server + Browser)
═══════════════════════════════════════════════════════════════
                           │
  User → GET /shop         │
      │                    │
      ▼                    │
  Laravel (PHP):           │
  ├── Route → Controller   │
  ├── Query → Product::all │
  ├── Blade → HTML         │
  └── @vite() → link CSS   │
      │                    │
      ▼                    │
  Browser:                 │
  ├── Parse HTML           │
  ├── Download app-xxx.css │
  ├── Download app-xxx.js  │
  ├── Alpine.js init       │
  └── Render halaman       │
                           │
═══════════════════════════╧═══════════════════════════════════
```

---

## 🧪 Latihan

1. **Buildtime.** Jalankan `npm run build`. Catat apa yang terjadi. File apa yang muncul di `public/build/`?

2. **Runtime.** Buka halaman shop di browser. Network tab → lihat file apa yang di-download (CSS, JS).

3. **Analisis.** Buka `public/build/manifest.json`. Mapping antara file sumber dan file output.

4. **Bandingkan.** Matikan `npm run dev`, refresh browser. Bandingkan waktu load vs saat dev server jalan.

5. **Kesimpulan.** Dalam satu kalimat: jelaskan perbedaan buildtime dan runtime ke teman non-teknis.

---

## 🔗 Referensi

- [Vite: Build](https://vitejs.dev/guide/build.html)
- [Laravel: Deployment](https://laravel.com/docs/11.x/deployment)
- Codebase: `vite.config.js`
- Codebase: `package.json`
- Codebase: `public/build/manifest.json`
