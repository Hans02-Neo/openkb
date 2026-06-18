# 11-01: Apa Itu Composer

> **Fase**: 11 — Tools & Workflow  
> **Prasyarat**: FASE 3 (OOP)  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: Composer, dependency manager, autoload, Packagist, vendor, PSR-4

---

## 📋 Ringkasan

Composer adalah **dependency manager** untuk PHP. Composer menginstall library yang dibutuhkan aplikasi (dari Packagist) dan mengatur autoloading.

**Target pemahaman:**
- Kamu paham peran Composer
- Kamu bisa membaca `composer.json`
- Kamu bisa install / update package
- Kamu paham autoload PSR-4

---

## 1. Tanpa Composer

```php
// ❌ Dulu: download ZIP, extract, require manual
require_once 'lib/midtrans/Midtrans.php';
require_once 'lib/midtrans/Transaction.php';
require_once 'lib/spatie/MediaLibrary.php';
// ... 50 require lagi
```

**Masalah:**
- Manual download → repot
- Update? Download ulang
- Dependency conflict? Cari manual

---

## 2. Dengan Composer

```bash
composer require spatie/laravel-medialibrary
```

Composer:
1. Download package + semua dependency-nya
2. Simpan di `vendor/`
3. Register autoload (tinggal `use`)

```php
// ✅ Cukup use — autoload otomatis
use Spatie\MediaLibrary\HasMedia;
use Spatie\MediaLibrary\InteractsWithMedia;
```

---

## 3. composer.json

```json
{
    "require": {
        "php": "^8.3",
        "laravel/framework": "^13.8",
        "laravel/tinker": "^3.0",
        "midtrans/midtrans-php": "^2.6",
        "spatie/laravel-medialibrary": "^11.0"
    },
    "require-dev": {
        "laravel/pint": "^1.27",
        "phpunit/phpunit": "^12.5",
        "laravel/telescope": "^5.20"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Database\\Factories\\": "database/factories/"
        }
    }
}
```

| Section | Arti |
|---------|------|
| `require` | Dependency production (wajib) |
| `require-dev` | Dependency development (testing, debug) |
| `autoload.psr-4` | Mapping namespace → folder |

---

## 4. Composer Commands

```bash
# Install semua dependency (dari composer.lock)
composer install

# Install + update (ke versi terbaru sesuai ^ constraint)
composer update

# Tambah package baru (production)
composer require spatie/laravel-medialibrary

# Tambah package dev
composer require --dev laravel/pint

# Hapus package
composer remove spatie/laravel-medialibrary

# Optimasi autoload (production)
composer dump-autoload -o
composer install --optimize-autoloader --no-dev
```

---

## 5. composer.lock

```
composer.json: "laravel/framework": "^13.8"  (versi 13.8.x)
composer.lock: "laravel/framework": "13.8.5" (versi PASTI)

Kenapa perlu lock?
→ Semua developer & server pakai versi yang SAMA
→ Tidak ada "works on my machine"
→ commit composer.lock ke Git!
```

---

## 6. Autoload PSR-4

```json
"autoload": {
    "psr-4": {
        "App\\": "app/"
    }
}
```

Artinya:

```
Namespace:  App\Models\Product
File:       app/Models/Product.php

Namespace:  App\Services\CartService
File:       app/Services/CartService.php

Namespace:  App\Http\Controllers\ShopController
File:       app/Http/Controllers/ShopController.php
```

---

## 7. Composer di Codebase

```bash
# Cek package terinstall
composer show

# Cek versi Laravel
composer show laravel/framework

# Cek package yang bisa update
composer outdated
```

---

## 🧪 Latihan

1. **Cek composer.json.** Buka `composer.json`. Package production apa saja yang dipakai?

2. **Cek vendor.** Buka folder `vendor/`. Cari `midtrans/` dan `spatie/`. Apa isinya?

3. **Cek autoload.** Cari file `vendor/composer/autoload_psr4.php`. Mapping apa yang ada?

4. **Tambah package.** `composer require --dev barryvdh/laravel-ide-helper`. Jalankan `php artisan ide-helper:generate`.

---

## 🔗 Referensi

- [Composer Official](https://getcomposer.org/)
- [Packagist](https://packagist.org/)
- [Laravel: Composer](https://laravel.com/docs/11.x/#composer)
- Codebase: `composer.json`, `composer.lock`
