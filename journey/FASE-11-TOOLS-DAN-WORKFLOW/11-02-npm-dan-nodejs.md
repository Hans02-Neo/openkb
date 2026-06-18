# 11-02: NPM & Node.js

> **Fase**: 11 — Tools & Workflow  
> **Prasyarat**: 11-01-apa-itu-composer  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: NPM, Node.js, package.json, node_modules, frontend dependencies, scripts

---

## 📋 Ringkasan

NPM (Node Package Manager) adalah dependency manager untuk JavaScript. Di Laravel, NPM mengelola frontend dependencies: Vite, Tailwind, Alpine.js, Axios.

**Target pemahaman:**
- Kamu paham peran NPM vs Composer
- Kamu bisa membaca package.json
- Kamu bisa install / build frontend
- Kamu paham node_modules

---

## 1. NPM vs Composer

| | Composer | NPM |
|---|----------|-----|
| **Untuk** | PHP | JavaScript |
| **File** | `composer.json` | `package.json` |
| **Lock file** | `composer.lock` | `package-lock.json` |
| **Folder** | `vendor/` | `node_modules/` |
| **Registry** | Packagist | npmjs.com |
| **Install** | `composer install` | `npm install` |
| **Command** | `composer require` | `npm install` |

---

## 2. package.json

```json
{
    "private": true,
    "type": "module",
    "scripts": {
        "build": "vite build",
        "dev": "vite"
    },
    "devDependencies": {
        "@tailwindcss/vite": "^4.0.0",
        "alpinejs": "^3.4.2",
        "concurrently": "^9.0.1",
        "laravel-vite-plugin": "^3.1",
        "tailwindcss": "^4.0.0",
        "vite": "^8.0.0"
    },
    "dependencies": {
        "axios": "^1.16.1"
    }
}
```

| Section | Arti |
|---------|------|
| `dependencies` | Runtime (axios untuk HTTP calls) |
| `devDependencies` | Build-time (Vite, Tailwind, Alpine) |
| `scripts` | Shortcut commands (`npm run dev`) |

---

## 3. NPM Commands

```bash
# Install semua dependency (dari package-lock.json)
npm install

# Install package baru (production)
npm install axios

# Install package baru (dev)
npm install --save-dev tailwindcss

# Build frontend
npm run build

# Dev server (HMR)
npm run dev
```

---

## 4. Scripts di Codebase

### 4.1 Development

```bash
npm run dev
# → Vite dev server (HMR aktif)
```

### 4.2 Production

```bash
npm run build
# → Vite build → output di public/build/
```

### 4.3 Concurrent (composer.json)

```json
"dev": [
    "npx concurrently ... \"php artisan serve\" \"php artisan queue:listen\" \"npm run dev\""
]
```

Satu command untuk semua service:

```bash
composer run dev
# → Laravel server + queue + Vite + log
```

---

## 5. node_modules

```text
node_modules/  ← Semua dependency JS (bisa 100MB+)
├── .package-lock.json
├── vite/
├── tailwindcss/
├── alpinejs/
├── axios/
└── ... (ratusan package lainnya — termasuk sub-dependency)
```

**Jangan commit node_modules:** sudah ada di `.gitignore`.

---

## 🧪 Latihan

1. **Cek package.json.** Buka `package.json`. Apa perbedaan `dependencies` vs `devDependencies`?

2. **Install.** Jalankan `npm install`. Cek folder `node_modules/`. Berapa banyak folder?

3. **Run dev.** Jalankan `npm run dev`. Apa yang muncul di terminal?

4. **Cek script.** Coba jalankan `npx vite build`. Bandingkan dengan `npm run build`.

---

## 🔗 Referensi

- [NPM Docs](https://docs.npmjs.com/)
- [Node.js](https://nodejs.org/)
- Codebase: `package.json`, `package-lock.json`
