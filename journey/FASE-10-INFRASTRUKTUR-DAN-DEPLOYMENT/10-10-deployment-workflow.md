# 10-10: Deployment Workflow

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-09-env-dan-konfigurasi  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: deployment, production, workflow, git, CI/CD, Forge, Deployer, zero downtime

---

## 📋 Ringkasan

Deployment adalah proses memindahkan aplikasi dari development ke production. Workflow yang baik membuat deploy cepat, aman, dan bisa rollback jika ada masalah.

**Target pemahaman:**
- Kamu paham deployment workflow dasar
- Kamu bisa deploy manual ke VPS
- Kamu paham automasi deployment
- Kamu paham checklist sebelum deploy

---

## 1. Deployment Workflow

### 1.1 Flow Dasar

```
Local (Development)          Server (Production)
     │                             │
     ├── Git commit ──────────────►│ Git clone / pull
     │          push ke GitHub     │
     │                             │
     │                             ├── composer install
     │                             │    (--no-dev, optimized)
     │                             │
     │                             ├── npm run build
     │                             │    (build frontend)
     │                             │
     │                             ├── php artisan migrate
     │                             │    (migrasi database)
     │                             │
     │                             ├── php artisan config:cache
     │                             │    (cache config)
     │                             │
     │                             ├── php artisan route:cache
     │                             │    (cache route)
     │                             │
     │                             ├── php artisan view:cache
     │                             │    (cache blade)
     │                             │
     │                             └── Restart queue worker
     │                                  (supervisor restart)
     │
     └── Selesai! ✅
```

---

## 2. Pre-Deployment Checklist

### 2.1 Sebelum Deploy

```text
□ Semua code sudah di-commit dan di-push ke GitHub
□ .env production sudah disiapkan di server
□ APP_DEBUG=false
□ APP_ENV=production
□ Database sudah siap (create database, user, privilege)
□ Storage folder sudah writable
□ Domain DNS sudah指向 server IP
□ SSL certificate sudah terinstall (HTTPS)
□ Queue worker setup (Supervisor)
□ Cron job setup (schedule)
□ Backup strategy sudah siap
□ Uji coba di staging environment (kalau ada)
```

### 2.2 Composer Production

```bash
# Production: no-dev, classmap autoload, optimize
composer install --optimize-autoloader --no-dev

# Bandingkan dengan development:
composer install
```

---

## 3. Deploy Manual (VPS)

### 3.1 SSH + Git Pull

```bash
# 1. SSH ke server
ssh olshop-prod

# 2. Masuk folder project
cd /var/www/olshop-koneksi

# 3. Backup dulu (kalau perlu)
cp .env .env.backup

# 4. Pull code terbaru
git pull origin main

# 5. Restore .env (jika overwrite)
cp .env.backup .env

# 6. Install dependencies
composer install --optimize-autoloader --no-dev

# 7. Build frontend
npm run build

# 8. Run migration (cek dulu: --pretend)
php artisan migrate --force

# 9. Cache
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 10. Restart queue
sudo supervisorctl restart all

# 11. Cek
php artisan optimize:clear   # ❌ Jangan! (di prod, ini hapus cache)
php artisan optimize          # ✅ Ini yang benar (re-cache)
```

### 3.2 Deployment Script

Bikin file `deploy.sh` di server:

```bash
#!/bin/bash
echo "=== Deploying olshop-koneksi ==="

cd /var/www/olshop-koneksi

# Maintenance mode ON
php artisan down

# Pull code
git pull origin main

# Dependencies
composer install --optimize-autoloader --no-dev

# Frontend
npm run build

# Database
php artisan migrate --force

# Cache
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Restart queue
sudo supervisorctl restart all

# Maintenance mode OFF
php artisan up

echo "=== Deploy selesai! ==="
```

```bash
# Jalankan:
chmod +x deploy.sh
./deploy.sh
```

---

## 4. Automasi Deployment

### 4.1 Laravel Forge

[Laravel Forge](https://forge.laravel.com/) adalah service untuk manage server Laravel:

```text
Forge Workflow:
1. Push ke GitHub
2. Forge detect push (via webhook)
3. Forge SSH ke server
4. Forge jalankan deploy script
5. Kirim notifikasi (Slack, email)
```

**Keuntungan Forge:**
- Setup server dalam 5 menit
- Manage Nginx, PHP, MySQL, Redis
- Queue worker (Supervisor)
- SSL (Let's Encrypt auto)
- One-click deploy
- Rollback

### 4.2 Deployer (PHP)

[Deployer](https://deployer.org/) — open source deployment tool:

```php
// deploy.php
namespace Deployer;

host('production')
    ->set('hostname', '203.0.113.50')
    ->set('remote_user', 'forge')
    ->set('identity_file', '~/.ssh/id_ed25519');

task('deploy', [
    'deploy:prepare',
    'deploy:vendors',
    'artisan:config:cache',
    'artisan:route:cache',
    'artisan:view:cache',
    'artisan:migrate',
    'deploy:publish',
]);
```

```bash
dep deploy production
```

### 4.3 GitHub Actions (CI/CD)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/olshop-koneksi
            git pull origin main
            composer install --no-dev
            php artisan migrate --force
            php artisan config:cache
            php artisan queue:restart
```

---

## 5. Rollback Strategy

### 5.1 Git Rollback

```bash
# Jika deploy bermasalah:
git revert HEAD --no-commit
git commit -m "Rollback: penyebab error"
git push origin main

# Atau reset ke commit sebelumnya:
git reset --hard HEAD~1
git push origin main --force   # Hati-hati!
```

### 5.2 Database Rollback

```bash
php artisan migrate:rollback --step=1
```

**Best practice:** Backup database sebelum migrate:
```bash
mysqldump -u root -p olshop > backup-$(date +%Y%m%d).sql
```

---

## 6. Zero Downtime Deployment

Agar user tidak melihat halaman maintenance:

**Strategy 1:** `php artisan down` + queue drain + deploy + `php artisan up`
→ Downtime beberapa detik.

**Strategy 2:** Load balancer + multiple server:
→ Satu server deploy, satunya serve traffic. Bergantian.

**Strategy 3:** Blue-green deployment:
→ Environment biru (lama) dan hijau (baru).
→ Switch traffic setelah hijau siap.

---

## 7. Deployment Checklist

```text
Sebelum Deploy:
□ Code sudah di-test (php artisan test)
□ Tidak ada debug code (dd, dump, ray)
□ APP_DEBUG=false
□ .env production siap
□ Database backup

Saat Deploy:
□ Git pull
□ composer install --no-dev
□ npm run build
□ php artisan migrate --force
□ php artisan config:cache
□ php artisan route:cache
□ php artisan view:cache
□ Queue restart
□ Permission storage

Setelah Deploy:
□ Cek halaman utama
□ Cek halaman error
□ Cek log (storage/logs/laravel.log)
□ Cek queue worker
□ Monitor selama 15 menit
```

---

## 🧪 Latihan

1. **Buat deploy script.** Tulis file `deploy.sh` untuk VPS (sesuai langkah di atas).

2. **Buat rollback plan.** Tulis langkah-langkah rollback jika deploy menyebabkan error 500.

3. **Simulasi.** Jika codebase ini akan di-deploy besok, apa yang perlu disiapkan? Buat checklist.

4. **Cari tahu.** Apa perbedaan `php artisan optimize` vs `php artisan optimize:clear`? Kapan pakai yang mana?

---

## 🔗 Referensi

- [Laravel Docs: Deployment](https://laravel.com/docs/11.x/deployment)
- [Laravel Forge](https://forge.laravel.com/)
- [Deployer](https://deployer.org/)
- [GitHub Actions](https://github.com/features/actions)
- [DigitalOcean: Laravel Deployment](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-laravel-application-on-ubuntu)
