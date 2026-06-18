# 10-02: Apache vs Nginx

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-01-apa-itu-web-server  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: Apache, Nginx, PHP-FPM, mod_php, web server comparison, performance

---

## 📋 Ringkasan

Apache dan Nginx adalah 2 web server paling populer. Keduanya bisa menjalankan Laravel, tapi pendekatan dan performanya berbeda.

**Target pemahaman:**
- Kamu paham perbedaan arsitektur Apache vs Nginx
- Kamu paham kenapa Nginx lebih cocok untuk production
- Kamu paham cara konfigurasi dasar keduanya

---

## 1. Perbandingan

| Aspek | Apache | Nginx |
|-------|--------|-------|
| **Arsitektur** | Process-based (thread per request) | Event-driven (single thread, async) |
| **PHP** | `mod_php` (built-in module) | PHP-FPM (separate process) |
| **Konfigurasi** | `.htaccess` per directory | Single config file |
| **Static file** | OK | Sangat cepat |
| **High traffic** | Boros memory | Lebih hemat |
| **SSL** | OK | Sangat cepat (OpenSSL) |
| **Laravel** | Bisa | Recommended |
| **Hosting** | cPanel, shared hosting | VPS, cloud |

---

## 2. Apache (Development)

### 2.1 Cara Kerja

```
Request masuk
    │
    ▼
Apache (prefork MPM)
    ├── Buat thread/process baru
    ├── Load mod_php
    ├── PHP process
    └── Return response

Setiap request → thread baru → boros memory di traffic tinggi.
```

### 2.2 Konfigurasi

Di Laragon, Apache dikonfigurasi via GUI → bisa ubah port, PHP version, dll.

### 2.3 .htaccess

Apache izinkan konfigurasi per folder:

```apache
# public/.htaccess — rewrite rules untuk Laravel
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

Kelebihan: Mudah diubah tanpa restart server.
Kekurangan: Setiap request, Apache baca semua `.htaccess` di path → lambat.

---

## 3. Nginx (Production)

### 3.1 Cara Kerja

```
Request masuk
    │
    ▼
Nginx (event loop — satu thread)
    ├── Static file → return langsung (super cepat)
    └── PHP → kirim ke PHP-FPM via fastcgi
         │
         ▼
    PHP-FPM (pool of worker processes)
         ├── Worker 1: proses PHP
         ├── Worker 2: proses PHP
         └── ...
         │
         ▼
    Return response ke Nginx → kirim ke client
```

**Kenapa lebih cepat?**
- Event-driven: satu thread handle ribuan koneksi
- Tidak ada `.htaccess` — konfigurasi dibaca sekali saat startup
- PHP dipisah (PHP-FPM) — tidak memblokir Nginx

### 3.2 Konfigurasi Nginx untuk Laravel

```nginx
# /etc/nginx/sites-available/olshop-koneksi
server {
    listen 80;
    server_name olshop-koneksi.test;
    root /var/www/olshop-koneksi/public;

    index index.php;

    # Static files — langsung serve
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP — kirim ke PHP-FPM
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Security: block .env
    location ~ /\.env {
        deny all;
    }
}
```

---

## 4. Skala Prioritas

| Lingkungan | Web Server | Kenapa? |
|------------|-----------|---------|
| **Development** (Laragon) | Apache | Mudah, sudah include, .htaccess |
| **VPS / Cloud** | Nginx + PHP-FPM | Performa, hemat memory |
| **Shared hosting** (cPanel) | Apache | cPanel standarnya Apache |
| **Docker** | Nginx | Ringan, cocok container |

---

## 5. Benchmark (Ilustrasi)

```
100 concurrent requests:

Apache:    100 threads × ~10MB = ~1GB memory
Nginx:     1 event loop + 10 PHP-FPM workers = ~150MB memory

Nginx bisa handle ~10x lebih banyak traffic dengan resource sama.
```

---

## 🧪 Latihan

1. **Cek web server.** Akses aplikasi, buka DevTools → Network → cari response header `Server`. Apa nilainya?

2. **.htaccess.** Nonaktifkan sementara `public/.htaccess` (rename). Akses `/shop`. Apa yang terjadi? Kenapa?

3. **Cari tahu.** Di Laragon, bagaimana cara ganti Apache port 80 → 8080? (Hint: Laragon menu → Apache → Port)

4. **Simulasi.** Jika aplikasi ini di-deploy ke VPS dengan Nginx, apa keuntungannya dibanding Laragon + Apache?

---

## 🔗 Referensi

- [Nginx + Laravel](https://laravel.com/docs/11.x/deployment#nginx)
- [Apache vs Nginx](https://www.digitalocean.com/community/tutorials/apache-vs-nginx-practical-considerations)
- Codebase: `public/.htaccess`
