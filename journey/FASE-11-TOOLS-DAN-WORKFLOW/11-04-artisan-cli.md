# 11-04: Artisan CLI

> **Fase**: 11 — Tools & Workflow  
> **Prasyarat**: 11-03-git-dan-version-control  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: Artisan, CLI, command line, php artisan, Laravel CLI, make commands

---

## 📋 Ringkasan

Artisan adalah **command-line interface** Laravel. Dari Artisan kamu bisa generate file, manage database, clear cache, dan banyak lagi.

**Target pemahaman:**
- Kamu paham peran Artisan
- Kamu bisa Artisan command penting
- Kamu bisa generate file (controller, model, migration)
- Kamu paham Artisan di production

---

## 1. Artisan — Dasar

```bash
# Lihat semua command
php artisan list

# Lihat help command tertentu
php artisan help make:model
```

---

## 2. Command Penting Harian

### 2.1 Generate (Make)

```bash
php artisan make:controller ShopController
php artisan make:model Product
php artisan make:model Product -m       # + migration
php artisan make:model Product -mc      # + migration + controller
php artisan make:migration create_products_table
php artisan make:seeder ProductSeeder
php artisan make:factory ProductFactory
php artisan make:request CheckoutRequest
php artisan make:middleware RoleMiddleware
php artisan make:policy OrderPolicy
php artisan make:observer OrderObserver
php artisan make:event OrderCreated
php artisan make:listener SendOrderConfirmation
php artisan make:command SendDailyReport
```

### 2.2 Database

```bash
php artisan migrate              # Jalankan migration
php artisan migrate:fresh        # Drop all tables + migrate
php artisan migrate:refresh      # Rollback + migrate
php artisan migrate:rollback     # Rollback batch terakhir
php artisan db:seed              # Jalankan seeder
php artisan db:seed --class=ProductSeeder
php artisan migrate:fresh --seed # Reset + seed
```

### 2.3 Cache

```bash
php artisan cache:clear          # Cache aplikasi
php artisan config:clear         # Config cache
php artisan config:cache         # Config cache (production)
php artisan route:clear          # Route cache
php artisan route:cache          # Route cache (production)
php artisan view:clear           # Blade cache
php artisan optimize:clear       # Clear semua cache
php artisan optimize             # Cache config + route + view
```

### 2.4 Storage & Link

```bash
php artisan storage:link
# → public/storage → storage/app/public symlink
```

### 2.5 Queue

```bash
php artisan queue:work           # Jalankan queue worker
php artisan queue:restart        # Restart worker setelah deploy
php artisan queue:table          # Migration untuk queue table
```

---

## 3. Artisan di Production

```bash
# Deploy sequence:
php artisan down --retry=60      # Maintenance mode (retry after 60s)
php artisan migrate --force      # Force = skip confirm
php artisan config:cache         # Cache config
php artisan route:cache          # Cache route
php artisan view:cache           # Cache blade
php artisan queue:restart        # Restart worker
php artisan up                   # Maintenance mode off

# Cek status
php artisan about                # Info aplikasi
php artisan optimize             # Re-cache semuanya
```

---

## 4. Artisan di Codebase

```bash
# Command kustom (kalau ada)
php artisan list | findstr "app:"
# Kalau ada App\Console\Commands\*

# Tinker (REPL)
php artisan tinker
```

---

## 5. Artisan vs Manual

| Task | Artisan | Manual |
|------|---------|--------|
| Buat controller | `make:controller` | Buat file manual |
| Migration | `migrate` | Jalanin SQL manual |
| Cache | `cache:clear` | Hapus file folder cache |
| Seed data | `db:seed` | INSERT SQL manual |
| Queue | `queue:work` | ? |

---

## 🧪 Latihan

1. **List command.** Jalankan `php artisan list`. Berapa banyak command?

2. **Generate.** Buat model `Category` dengan migration: `php artisan make:model Category -m`.

3. **Tinker.** `php artisan tinker` → ketik `Product::count()`. Berapa jumlah produk?

4. **About.** `php artisan about`. Informasi apa yang ditampilkan?

5. **Route list.** `php artisan route:list`. Semua route yang terdaftar.

---

## 🔗 Referensi

- [Laravel Docs: Artisan](https://laravel.com/docs/11.x/artisan)
- [Laravel Docs: Artisan Commands](https://laravel.com/docs/11.x/artisan#available-commands)
