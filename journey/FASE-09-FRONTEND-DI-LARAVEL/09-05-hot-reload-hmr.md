# 09-05: Hot Reload & HMR — Development Speed

> **Fase**: 9 — Frontend di Laravel  
> **Prasyarat**: 09-01-cara-kerja-vite  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: HMR, hot reload, hot module replacement, dev server, live refresh

---

## 📋 Ringkasan

HMR (Hot Module Replacement) memungkinkan perubahan CSS/JS langsung terlihat di browser **tanpa refresh halaman**. Vite + Laravel memberikan HMR out-of-the-box.

**Target pemahaman:**
- Kamu paham cara kerja HMR
- Kamu bisa menjalankan Vite dev server
- Kamu paham refresh vs HMR
- Kamu bisa troubleshoot HMR

---

## 1. Dev Server Flow

### 1.1 Cara Kerja

```
npm run dev
    │
    ▼
Vite dev server (http://localhost:5173)
    │
    ├── Serve ESM modules (native browser modules)
    ├── Watch file changes
    └── WebSocket connection ke browser
         │
         ▼
    File berubah → Vite kirim update via WS
    Browser terima update → HMR (atau full reload)
```

### 1.2 Tahapan

```
1. Kamu edit resources/css/app.css
2. Vite deteksi perubahan
3. Vite kirim CSS update via WebSocket
4. Browser update CSS — tanpa refresh!
5. Halaman tetap di posisi yang sama
```

---

## 2. HMR vs Full Refresh

| | HMR | Full Refresh |
|---|-----|-------------|
| **CSS** | Update style — instant ✅ | Reload halaman |
| **JS/Alpine** | Hot update (state bisa hilang) | Reload halaman |
| **Blade** | N/A (server-side) | Vite trigger `refresh: true` → reload |
| **Kecepatan** | Milidetik | 100-500ms |
| **State** | Dipertahankan | Hilang |

### 2.1 Blade Refresh

Vite tidak bisa HMR untuk Blade (server-side rendered). Tapi dengan `refresh: true` di konfigurasi:

```js
// vite.config.js
laravel({
    input: ['resources/css/app.css', 'resources/js/app.js'],
    refresh: true,  // ← Auto refresh browser saat Blade berubah
})
```

Vite akan trigger **full page refresh** saat file Blade berubah.

---

## 3. Flow Lengkap Dev

```
Terminal 1: npm run dev   → Vite dev server
Terminal 2: php artisan serve  → Laravel server (opsional, Laragon handle)

Browser: http://localhost:8000

✅ Edit app.css → style berubah tanpa refresh
✅ Edit app.js → Alpine update (kadang perlu refresh)
✅ Edit index.blade.php → browser auto refresh
```

### 3.1 Yang Bisa HMR

| File | Efek |
|------|------|
| `resources/css/app.css` | Style update instan |
| File CSS lain di `resources/css/` | Style update instan |
| Tailwind class baru | Style update instan |

### 3.2 Yang Trigger Full Refresh

| File | Efek |
|------|------|
| `resources/js/app.js` | Browser reload |
| File JS lain | Browser reload |
| File Blade `.blade.php` | Browser reload |
| `vite.config.js` | Restart Vite |

---

## 4. Troubleshooting

### 4.1 HMR Tidak Berfungsi

```bash
# 1. Pastikan Vite jalan
npm run dev

# 2. Cek port (default 5173)
# Ada error "port already in use"? Ganti port:
# Bikin .env: VITE_PORT=5174

# 3. Clear cache
rm -rf node_modules/.vite
npm run dev

# 4. Pastikan @vite() ada di layout
# Kalau tidak, browser tidak tahu Vite dev server
```

### 4.2 CORS Error

Jika Vite dev server di port berbeda dari Laravel:

```
Access to script at 'http://localhost:5173/...' from origin 'http://localhost:8000'
has been blocked by CORS policy.
```

**Solusi:** Pastikan `@vite()` digunakan — Laravel handle CORS otomatis.

### 4.3 White Screen

```bash
# Hapus cache Vite
rm -rf node_modules/.vite

# Restart
npm run dev
```

---

## 5. Deteksi Mode — Dev vs Production

```blade
@if (app()->environment('local'))
    {{-- Dev-only content --}}
@endif
```

Atau via Vite:

```blade
@production
    <link rel="stylesheet" href="{{ Vite::asset('resources/css/app.css') }}">
@endproduction
```

Tapi praktik standar cukup:

```blade
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

`@vite()` otomatis pilih mode yang benar.

---

## 6. Ringkasan

```
┌─────────────────────────────────────────────────────────────────────┐
│ HOT MODULE REPLACEMENT (HMR)                                       │
├─────────────────────────────────────────────────────────────────────┤
│  npm run dev                                                        │
│       │                                                             │
│       ▼                                                             │
│  Vite Server :5173                                                  │
│       │                                                             │
│       ├── Watch file changes                                        │
│       ├── WebSocket ke browser                                      │
│       └── Relay perubahan                                           │
│            │                                                        │
│            ▼                                                        │
│  ┌──────────────┐    ┌─────────────────┐                           │
│  │ CSS berubah   │    │ Blade/JS berubah│                           │
│  │ → HMR (instan)│    │ → Full refresh  │                           │
│  └──────────────┘    └─────────────────┘                           │
├─────────────────────────────────────────────────────────────────────┤
│  TROUBLESHOOTING                                                    │
│  ├─ Vite tidak jalan → jalankan npm run dev                        │
│  ├─ HMR tidak jalan → hapus node_modules/.vite                     │
│  └─ CORS error → pastikan @vite() di layout                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Jalankan HMR.** Buka dua terminal: `npm run dev` + akses aplikasi. Edit `resources/css/app.css` — ganti warna `btn-primary`. Lihat perubahan tanpa refresh.

2. **Deteksi.** Edit file Blade (ubah teks di `shop/index.blade.php`). Apakah browser auto refresh?

3. **Troubleshoot.** Matikan `npm run dev`. Refresh browser. Apa yang terjadi dengan CSS/JS? Catat errornya.

4. **Analisis.** Mengapa CSS bisa HMR tapi Blade tidak? (Jawaban: CSS client-side, Blade server-side.)

---

## 🔗 Referensi

- [Vite: HMR](https://vitejs.dev/guide/features.html#hot-module-replacement)
- [Vite: Troubleshooting](https://vitejs.dev/guide/troubleshooting.html)
- Codebase: `vite.config.js` (baris `refresh: true`)
