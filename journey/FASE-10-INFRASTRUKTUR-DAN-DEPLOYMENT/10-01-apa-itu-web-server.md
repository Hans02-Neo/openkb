# 10-01: Apa Itu Web Server

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: FASE 5 (Web Fundamental)  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: web server, Apache, Nginx, HTTP, reverse proxy, PHP-FPM

---

## 📋 Ringkasan

Web server adalah software yang menerima HTTP request dari browser dan mengirimkan response (HTML, CSS, JS, JSON, dll). Di codebase ini, web server mengatur bagaimana PHP Laravel menerima dan merespon request.

**Target pemahaman:**
- Kamu paham peran web server
- Kamu bisa bedakan web server dengan application server
- Kamu paham bagaimana request diproses

---

## 1. Web Server — Analogi

```
Kamu (Browser)            Restoran (Web Server)            Dapur (PHP/Laravel)
    │                          │                               │
    ├── "Saya mau liat menu" ──►                               │
    │                          ├── "Menu apa aja?" ──────────► │
    │                          │                               ├── Query DB
    │                          │                               ├── Render Blade
    │                          │                               └── "Ini menunya" ──►
    │                          │                               │
    │                          ◄── HTML ────────────────────── │
    │ ◄── Tampilkan menu ──────                                │
```

**Web server tugasnya:**
- Menerima request (HTTP)
- Menentukan cara handle (file statis? atau kirim ke PHP?)
- Mengirim response

---

## 2. Cara Kerja

### 2.1 Request Flow

```
Browser ──GET /shop──► Web Server (Apache/Nginx)
                           │
                           ├── If file exists (CSS, JS, image)
                           │   → Return langsung (static file)
                           │
                           └── If needs PHP (index.php)
                               → Kirim ke PHP-FPM
                               → Laravel proses
                               → Return HTML
```

### 2.2 Front Controller Pattern

```apache
{{-- public/.htaccess --}}
RewriteEngine On

# Jika bukan file/directory nyata → kirim ke index.php
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

**Semua request masuk lewat `public/index.php`** — ini disebut front controller.

```php
// public/index.php
$app = require_once __DIR__.'/../bootstrap/app.php';
$kernel = $app->make(Kernel::class);
$response = $kernel->handle($request);
$response->send();
```

---

## 3. Web Server di Ekosistem

### 3.1 Development (Lokal)

```
Laragon: Apache + PHP + MySQL
         ├── Apache: web server (port 80)
         ├── PHP: language processor
         └── MySQL: database
```

### 3.2 Production (Server)

```
Nginx + PHP-FPM + MySQL
├── Nginx: web server (port 80/443)
├── PHP-FPM: PHP processor
└── MySQL: database
```

### 3.3 Perbedaan Development vs Production

| Aspek | Development (Laragon) | Production |
|-------|----------------------|------------|
| Web server | Apache (prefork) | Nginx (recommended) |
| PHP handler | mod_php (Apache module) | PHP-FPM (separate process) |
| Konfigurasi | Laragon GUI | File config (nginx.conf) |
| HTTPS | Self-signed | Let's Encrypt / SSL |

---

## 4. HTTP Protocol Recap

### 4.1 Request

```
GET /shop HTTP/1.1
Host: olshop-koneksi.test
Accept: text/html
Cookie: session=abc123
```

### 4.2 Response

```
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=xyz789

<!DOCTYPE html>
<html>...
```

### 4.3 Status Codes

| Code | Arti | Contoh |
|------|------|--------|
| 200 | OK | Halaman berhasil dimuat |
| 301 | Redirect | HTTP → HTTPS |
| 404 | Not Found | Halaman tidak ada |
| 500 | Server Error | Ada error di kode |

---

## 🧪 Latihan

1. **Cek .htaccess.** Buka `public/.htaccess`. Pahami setiap baris.

2. **Cek index.php.** Buka `public/index.php`. Apa yang dilakukan file ini?

3. **Akses langsung.** Coba akses `http://olshop-koneksi.test/test.php` (ada di `public/test.php`). Bandingkan dengan akses route `/shop`.

4. **Cek header.** Buka browser DevTools → Network tab. Cari request ke `/shop`. Lihat header response (status code, content-type).

---

## 🔗 Referensi

- [MDN: What is a web server?](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Web_mechanics/What_is_a_web_server)
- [Apache HTTP Server](https://httpd.apache.org/)
- [Nginx](https://nginx.org/)
- Codebase: `public/.htaccess`
- Codebase: `public/index.php`
