# 10-09: ENV & Konfigurasi

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-08-ftp-dan-file-transfer  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: .env, environment, configuration, APP_ENV, APP_DEBUG, APP_KEY, environment variables

---

## 📋 Ringkasan

File `.env` menyimpan **konfigurasi spesifik environment** — database, API key, mode aplikasi. File ini **tidak boleh di-commit** ke Git karena berisi kredensial.

**Target pemahaman:**
- Kamu paham cara kerja .env
- Kamu bisa set environment untuk dev vs production
- Kamu paham APP_KEY, APP_DEBUG, APP_ENV
- Kamu bisa troubleshoot konfigurasi

---

## 1. Environment Variables

### 1.1 Cara Kerja

```
.env file kamu             .env.example (template)
┌──────────────────┐       ┌──────────────────┐
│ DB_HOST=localhost│       │ DB_HOST=127.0.0.1│
│ DB_PASSWORD=secret│      │ DB_PASSWORD=     │ ← kosong
│ MIDTRANS_SERVER=xxx│     │ MIDTRANS_SERVER= │ ← template
│ API_KEY=abc123    │      │ API_KEY=         │ ← template
└──────────────────┘       └──────────────────┘
     │                            │
     │ Tidak di-commit            │ Di-commit ke Git
     ▼                            ▼
  config/database.php        config/midtrans.php
  (baca via env())           (baca via env())
```

### 1.2 File .env di Git

```text
.env                  ← ❌ Jangan commit! (ada di .gitignore)
.env.example          ← ✅ Commit (template kosong)
```

**Cek .gitignore:**

```gitignore
# .gitignore
.env
.env.local
.env.production
```

---

## 2. Production vs Development

### 2.1 Settings Penting

```env
# Development (Laragon)
APP_ENV=local
APP_DEBUG=true
APP_URL=http://olshop-koneksi.test
DB_CONNECTION=sqlite

# Production (Server)
APP_ENV=production
APP_DEBUG=false
APP_URL=https://olshop-koneksi.com
DB_CONNECTION=mysql
DB_HOST=localhost
DB_DATABASE=olshop_prod
DB_USERNAME=olshop_user
DB_PASSWORD=your_db_password
```

### 2.2 APP_DEBUG = false

```
APP_DEBUG=true:
  Error → tampilkan stack trace detail (berbahaya di production!)

APP_DEBUG=false:
  Error → tampilkan halaman 500 generic
  Stack trace di-log ke storage/logs/laravel.log
```

**Jangan pernah set APP_DEBUG=true di production** — bisa bocorkan struktur database, path file, dan kredensial.

### 2.3 APP_KEY

```env
APP_KEY=base64:lKp2lq6HfqoirVaIcx+VWSZNpMRTqwiXgKkUm6qrHyY=
```

APP_KEY adalah **encryption key** untuk Laravel (session, cookie, enkripsi data).

```bash
# Generate key baru:
php artisan key:generate

# Production: harus SETIAP deploy pakai key yang SAMA
# Kalau ganti → semua session invalid, data terenkripsi tidak bisa dibaca
```

---

## 3. Konfigurasi di Codebase

### 3.1 File .env

```env
APP_NAME=Laravel
APP_ENV=local
APP_DEBUG=true
APP_URL=http://olshop-koneksi.test

DB_CONNECTION=sqlite

SESSION_DRIVER=database
CACHE_STORE=database
QUEUE_CONNECTION=database

MAIL_MAILER=log

MIDTRANS_MERCHANT_ID=your_merchant_id
MIDTRANS_SERVER_KEY=Mid-server-your_server_key_here
MIDTRANS_CLIENT_KEY=Mid-client-your_client_key_here
MIDTRANS_IS_PRODUCTION=false
```

### 3.2 Yang Berubah di Production

```env
# Production changes:
APP_ENV=production
APP_DEBUG=false
APP_URL=https://olshop-koneksi.com

DB_CONNECTION=mysql
DB_HOST=localhost
DB_DATABASE=olshop_prod
DB_USERNAME=olshop_user
DB_PASSWORD=xxx

MIDTRANS_IS_PRODUCTION=true    # Aktifkan payment beneran

MAIL_MAILER=smtp               # Kirim email beneran
MAIL_HOST=smtp.gmail.com
MAIL_USERNAME=admin@olshop.com
MAIL_PASSWORD=xxx
```

---

## 4. Config Caching

Production: gabung semua config jadi satu file biar cepat.

```bash
# Di server production (setelah update .env)
php artisan config:cache
# → Semua config di-cache ke bootstrap/cache/config.php

# Kalau ada error config → akan ketahuan di sini
# Jangan lupa setelah update .env:
php artisan config:clear   # Dev
php artisan config:cache   # Prod
```

**Penting:** Setelah `config:cache`, function `env()` tidak bekerja di production — gunakan `config()`.

---

## 5. Troubleshooting .env

### 5.1 .env Tidak Terbaca

```bash
# Cache config masih lama → config lama dipakai
php artisan config:clear

# Atau .env tidak ada → copy dari .env.example
cp .env.example .env
php artisan key:generate
```

### 5.2 Cache Bermasalah

```bash
# Clear semua cache
php artisan optimize:clear
# → config, route, view, event, cache clear semua
```

---

## 🧪 Latihan

1. **Baca .env.** Buka `.env` codebase. Identifikasi 3 hal yang bakal berbeda di production.

2. **Buat .env.example.** Cek apakah `.env.example` up-to-date dengan `.env`. Tambah variable yang kurang.

3. **Simulasi.** Ganti `APP_ENV=production` dan `APP_DEBUG=false`. Akses halaman error (misal route salah). Bandingkan tampilan error dengan mode `local`.

4. **Key.** Jalankan `php artisan key:generate`. Catat key baru. Ganti APP_KEY di .env dengan key baru. Apa efeknya pada session?

---

## 🔗 Referensi

- [Laravel Docs: Configuration](https://laravel.com/docs/11.x/configuration)
- [Laravel Docs: Environment](https://laravel.com/docs/11.x/configuration#environment-file-types)
- Codebase: `.env`
- Codebase: `.env.example`
