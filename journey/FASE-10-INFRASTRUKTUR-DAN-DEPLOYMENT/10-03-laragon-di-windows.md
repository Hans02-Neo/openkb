# 10-03: Laragon di Windows

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-02-apache-vs-nginx  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: Laragon, Windows, WAMP, local development, Apache, PHP, MySQL, Laragon

---

## 📋 Ringkasan

Laragon adalah **local development environment** untuk Windows. Satu paket: Apache + PHP + MySQL + Node.js + Python. Dibanding XAMPP/WampServer, Laragon lebih modern dan cocok untuk Laravel.

**Target pemahaman:**
- Kamu paham peran Laragon
- Kamu paham cara kerja Laragon
- Kamu bisa manage service Laragon
- Kamu tahu lokasi file penting Laragon

---

## 1. Laragon — All-in-One

### 1.1 Isi Paket

| Komponen | Path di Laragon |
|----------|----------------|
| **Apache** | `laragon/bin/apache/` |
| **PHP** | `laragon/bin/php/` |
| **MySQL** | `laragon/bin/mysql/` |
| **Node.js** | `laragon/bin/nodejs/` |
| **Composer** | `laragon/bin/composer/` |
| **Git** | `laragon/bin/git/` |

### 1.2 Fitur Penting

| Fitur | Fungsi |
|-------|--------|
| **Auto virtual hosts** | `project.test` → langsung akses |
| **Quick create** | Buat project Laravel baru |
| **Port management** | Ganti port 80 → 8080 |
| **PHP version switcher** | Ganti PHP 8.2 ↔ 8.3 |
| **SSL** | Auto HTTPS untuk development |
| **Services** | Start/stop Apache, MySQL, Nginx, Redis |

---

## 2. Cara Kerja Laragon

### 2.1 Folder Struktur

```
C:\laragon\
├── www\                          ← Semua project di sini
│   ├── olshop-koneksi\          ← Project kamu
│   └── project-lain\
├── bin\                          ← Software (Apache, PHP, MySQL)
├── etc\                          ← Konfigurasi
│   └── apache\                  ← Apache config
└── tmp\                          ← Temporary files
```

### 2.2 Auto Virtual Host

Saat Laragon jalan:

```
Folder:  C:\laragon\www\olshop-koneksi
URL:     http://olshop-koneksi.test
         https://olshop-koneksi.test (SSL)
```

Laragon otomatis buat virtual host entry tanpa perlu edit `hosts` file manual.

### 2.3 Services Control

```
Laragon Menu →
  ├── Start All       → Apache + MySQL + Node.js
  ├── Stop All        → Stop semua
  ├── Apache          → Start / Stop / Restart
  ├── MySQL           → Start / Stop / Restart
  └── PHP             → Version / Settings
```

---

## 3. Laragon vs Alternatif

| Fitur | Laragon | XAMPP | WampServer |
|-------|---------|-------|------------|
| **Auto vhost** | ✅ | ❌ Manual | ❌ Manual |
| **PHP version switch** | ✅ Mudah | ✅ Manual | ✅ Manual |
| **Node.js bundled** | ✅ | ❌ | ❌ |
| **Composer bundled** | ✅ | ❌ | ❌ |
| **SSL auto** | ✅ | ❌ | ❌ |
| **Quick app** | ✅ Laravel, WordPress | ❌ | ❌ |
| **Portable** | ✅ Bisa pindah folder | ❌ | ❌ |

---

## 4. Tips Laragon

### 4.1 Ganti Port

```
Kalau port 80 dipake (Skype, IIS):
  Laragon Menu → Apache → Port → 8080
  Akses via http://olshop-koneksi.test:8080
```

### 4.2 Ganti PHP Version

```
Laragon Menu → PHP → Version → 8.3 (atau versi lain)
→ Apache otomatis restart
```

### 4.3 Let's Encrypt for Local

Laragon bisa generate self-signed SSL untuk `.test` domain → HTTPS di lokal.

---

## 5. Commands Penting

```bash
# Di Laragon terminal, path PHP sudah include:
php --version           # Cek PHP version
composer --version      # Cek Composer version
node --version          # Cek Node version
npm --version           # Cek NPM version
mysql --version         # Cek MySQL version

# Install Laravel project:
composer create-project laravel/laravel project-name

# Akses database:
# Buka Laragon → Database → HeidiSQL (or Adminer)
```

---

## 🧪 Latihan

1. **Cek Laragon.** Buka Laragon. Service apa yang sedang running?

2. **Cek folder.** Buka `C:\laragon\www\`. Project apa saja yang ada?

3. **Cek PHP.** Di Laragon terminal, jalankan `php --version`. Pastikan sesuai dengan requirement Laravel 11 (^8.3).

4. **Cek hosts.** Buka `C:\Windows\System32\drivers\etc\hosts`. Apakah ada entry untuk `olshop-koneksi.test`? (Laragon manage otomatis.)

5. **Restart service.** Stop Apache lewat Laragon menu, lalu akses `http://olshop-koneksi.test`. Apa yang terjadi?

---

## 🔗 Referensi

- [Laragon Official](https://laragon.org/)
- [Laragon Docs](https://laragon.org/docs/)
- Instalasi lokal: `C:\laragon\`
