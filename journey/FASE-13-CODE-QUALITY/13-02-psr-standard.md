# 13-02: PSR Standard

> **Fase**: 13 — Code Quality  
> **Prasyarat**: 13-01-code-readability  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: PSR, PHP-FIG, coding standard, PSR-1, PSR-4, PSR-12, interoperability

---

## 📋 Ringkasan

PSR (PHP Standard Recommendation) adalah standar coding PHP yang dibuat oleh PHP-FIG. Framework dan library PHP mengikuti PSR agar kode konsisten dan interoperable.

**Target pemahaman:**
- Kamu paham PSR yang relevan
- Kamu bisa menulis kode sesuai PSR-12
- Kamu paham PSR-4 autoloading
- Kamu bisa jalankan Laravel Pint

---

## 1. PSR yang Penting

| PSR | Nama | Isi |
|-----|------|-----|
| **PSR-1** | Basic Coding Standard | Naming class, method, file |
| **PSR-4** | Autoloading | Namespace → folder mapping |
| **PSR-12** | Extended Coding Style | Style guide (extends PSR-1) |

---

## 2. PSR-1 — Basic

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    const STATUS_ACTIVE = 'active';  // UPPER_SNAKE_CASE

    public $fillable = ['name', 'price'];  // camelCase

    public function getPriceAttribute($value)  // camelCase
    {
        return number_format($value);
    }
}
```

**Aturan:**
- File PHP: `<?php` atau `<?php + declare(strict_types=1)`
- Namespace: `App\Models\Product`
- Class: `PascalCase`
- Method: `camelCase`
- Constant: `UPPER_SNAKE_CASE`
- File: satu class per file

---

## 3. PSR-4 — Autoloading

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    }
}
```

| Namespace | File |
|-----------|------|
| `App\Models\Product` | `app/Models/Product.php` |
| `App\Services\CartService` | `app/Services/CartService.php` |
| `App\Http\Controllers\ShopController` | `app/Http/Controllers/ShopController.php` |

---

## 4. PSR-12 — Style Guide

### 4.1 Braces

```php
// ✅ PSR-12
class Product extends Model
{
    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }
}

// ❌ Bukan PSR-12
class Product extends Model {
    public function scopeActive($query) {
        return $query->where('is_active', true);
    }
}
```

### 4.2 Keywords

```php
// ✅ PSR-12
if ($condition) {
    // ...
} elseif ($other) {
    // ...
} else {
    // ...
}

foreach ($items as $item) {
    // ...
}

try {
    // ...
} catch (\Exception $e) {
    // ...
}
```

### 4.3 Type Hints

```php
// ✅ PSR-12
public function getActiveProducts(): Collection
{
    return Product::where('is_active', true)->get();
}

public function setPrice(int $price): void
{
    $this->price = $price;
}
```

---

## 5. Laravel Pint

Pint adalah **code formatter** Laravel (berbasis PHP-CS-Fixer).

```bash
# Cek perubahan (dry run)
./vendor/bin/pint --test

# Format semua file
./vendor/bin/pint

# Format folder spesifik
./vendor/bin/pint app/Models/

# Format file spesifik
./vendor/bin/pint app/Http/Controllers/ShopController.php
```

**Yang dilakukan Pint:**
- Atur braces, spacing, indentation
- Atur import ordering
- Atur trailing commas
- Hapus unused imports

---

## 6. PSR di Codebase Ini

```bash
# Cek apakah kode sudah sesuai PSR
./vendor/bin/pint --test

# Kalau ada error → format
./vendor/bin/pint
```

---

## 🧪 Latihan

1. **Run Pint test.** `./vendor/bin/pint --test`. Apakah ada file yang tidak sesuai standar?

2. **Format.** `./vendor/bin/pint`. Jalankan, lalu cek apa yang berubah (via git diff).

3. **Cek PSR-4.** Buka `composer.json` → `autoload.psr-4`. Apakah mapping sesuai dengan struktur folder?

4. **Review file.** Buka file PHP di `app/`. Periksa: apakah mengikuti PSR-12 spacing dan braces?

---

## 🔗 Referensi

- [PHP-FIG: PSR](https://www.php-fig.org/psr/)
- [PSR-1](https://www.php-fig.org/psr/psr-1/)
- [PSR-4](https://www.php-fig.org/psr/psr-4/)
- [PSR-12](https://www.php-fig.org/psr/psr-12/)
- [Laravel Pint](https://laravel.com/docs/11.x/pint)
- Codebase: `composer.json` (autoload PSR-4)
