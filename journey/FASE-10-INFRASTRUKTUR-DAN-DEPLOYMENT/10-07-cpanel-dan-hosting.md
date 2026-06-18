# 10-07: cPanel & Shared Hosting

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-06-docker-dasar  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: cPanel, shared hosting, File Manager, phpMyAdmin, DNS, domain, hosting panel

---

## 📋 Ringkasan

cPanel adalah panel kontrol hosting yang paling populer. Banyak Laravel developer mulai dari shared hosting sebelum beralih ke VPS. Kamu perlu paham cPanel untuk manage domain, database, dan file.

**Target pemahaman:**
- Kamu paham fitur utama cPanel
- Kamu bisa deploy Laravel ke cPanel
- Kamu paham keterbatasan shared hosting

---

## 1. cPanel — Fitur Utama

```
cPanel Dashboard
├── File
│   ├── File Manager        ← Upload/edit file via browser
│   └── Git Version Control ← Clone repository
├── Database
│   ├── phpMyAdmin          ← Manage MySQL via UI
│   └── MySQL Databases     ← Create DB, user, privilege
├── Domain
│   ├── Domains             ← Add domain, subdomain
│   ├── Zone Editor         ← DNS record (A, CNAME, MX)
│   └── Redirects           ← URL redirect
├── Email
│   ├── Email Accounts      ← Buat email (info@domain.com)
│   └── Forwarders          ← Forward email
├── Security
│   ├── SSL/TLS             ← Install SSL certificate
│   └── IP Blocker          ← Block IP address
└── Software
    ├── PHP Selector         ← Pilih PHP version
    └── Composer             ← Jalankan composer
```

---

## 2. Deploy Laravel ke cPanel

### 2.1 Persiapan

```text
Domain:   olshop-koneksi.com
Root:     /home/user/public_html/
Laravel:  /home/user/laravel/ (di luar public_html)
Public:   /home/user/public_html/ → symlink ke /home/user/laravel/public/
```

### 2.2 Langkah-langkah

```bash
# 1. Upload file Laravel via File Manager atau Git
#    Letakkan di folder di luar public_html:
/home/user/
├── laravel/           ← Semua file Laravel
│   ├── app/
│   ├── bootstrap/
│   ├── config/
│   ├── public/        ← Isi public dipindah ke symlink
│   └── ...
└── public_html/       ← Document root (symlink ke laravel/public)
```

```bash
# 2. Set Document Root ke /home/user/laravel/public
#    (via cPanel → Domains → Document Root)
#    Atau buat symlink:
ln -s /home/user/laravel/public /home/user/public_html/olshop

# 3. Edit .env → production settings
APP_ENV=production
APP_DEBUG=false
APP_URL=https://olshop-koneksi.com
```

```bash
# 4. Setup storage permission
chmod -R 775 /home/user/laravel/storage
chmod -R 775 /home/user/laravel/bootstrap/cache
```

### 2.3 Modify public/index.php

Beberapa shared hosting perlu ubah path bootstrap:

```php
// public/index.php
$app = require_once __DIR__.'/../bootstrap/app.php';
// → Pastikan path benar karena Laravel di luar public_html
```

Alternatif: **Symlink** dari `public_html` ke `laravel/public`.

---

## 3. Keterbatasan Shared Hosting

| Fitur | Shared Hosting | VPS |
|-------|---------------|-----|
| **PHP version** | Terbatas (selector) | Bebas |
| **PHP extensions** | Terbatas | Full control |
| **Artisan commands** | Via cron job | Terminal penuh |
| **Queue worker** | ❌ Sulit | ✅ Supervisor |
| **SSH access** | Kadang ada | ✅ Full |
| **Composer** | Limited (memory) | ✅ Full |
| **Resources** | Sharing (noisy neighbor) | Dedicated |
| **Harga** | Murah (~$5/bln) | Mulai ~$5/bln |

### 3.1 Shared Hosting — Masalah untuk Laravel

```text
Queue worker:  Shared hosting tidak jalan background process
               ❌ php artisan queue:work tidak bisa

Artisan command:  Bisa via cron, tapi ada timeout
                  ❌ composer install sering timeout memory

Storage link:     php artisan storage:link kadang tidak bisa
                  (manual symlink)

Environment:      APP_ENV=production, APP_DEBUG=false
                  APP_URL harus HTTPS
```

---

## 4. Alternatif Hosting untuk Laravel

| Provider | Type | Harga Mulai | Cocok untuk |
|----------|------|-------------|-------------|
| **DigitalOcean** | VPS (Droplet) | $6/bln | Production |
| **Linode** | VPS | $5/bln | Production |
| **Vultr** | VPS | $2.5/bln | Production |
| **AWS EC2** | VPS | Gratis 1 tahun | Production |
| **RunCloud** | Managed VPS | $6/bln + VPS | Production |
| **Forge** | Managed VPS | $12/bln + VPS | Production |
| **cPanel** | Shared | $3-10/bln | Development / small |

---

## 🧪 Latihan

1. **Cek phpMyAdmin.** Buka laragon → Database → phpMyAdmin (atau HeidiSQL). Familiar dengan UI database.

2. **Simulasi.** Buat file `simulasi-deploy.txt`. Tulis langkah-langkah deploy Laravel ke cPanel versi kamu sendiri.

3. **Cari tahu.** Cek hosting yang ada (jika punya). Apa PHP version yang tersedia? Apa extension yang aktif?

4. **Bandingkan.** Jika harus pilih hosting untuk `olshop-koneksi`, mana yang kamu pilih: shared hosting atau VPS? Kenapa?

---

## 🔗 Referensi

- [cPanel Official](https://cpanel.net/)
- [Laravel Docs: Deployment](https://laravel.com/docs/11.x/deployment)
- [DigitalOcean: Laravel Deployment](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-laravel-application-on-ubuntu)
