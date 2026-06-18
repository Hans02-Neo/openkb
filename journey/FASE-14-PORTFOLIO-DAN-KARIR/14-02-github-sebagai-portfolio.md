# 14-02: GitHub Sebagai Portfolio

> **Fase**: 14 — Portfolio & Karir  
> **Prasyarat**: 14-01-portfolio-dari-codebase-ini  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: GitHub, portfolio, profile, repository, open source, contribution graph

---

## 📋 Ringkasan

GitHub adalah **social network untuk developer**. Profile GitHub yang rapi menunjukkan kompetensi dan aktivitas coding. HR dan interviewer pasti cek GitHub kamu.

**Target pemahaman:**
- Kamu bisa mengatur GitHub profile
- Kamu bisa manage repository
- Kamu paham contribution graph
- Kamu tahu cara presentasikan repo

---

## 1. GitHub Profile

### 1.1 Profile README

Buat repository `username/username` (special repo):

```markdown
# Hi, I'm Budi! 👋

## 🚀 About Me
Junior Laravel developer with experience in:
- PHP 8.3 / Laravel 13
- Tailwind CSS / Alpine.js
- MySQL / SQLite
- Git / REST API

## 🛠️ Project
🔭 **[olshop-koneksi](https://github.com/username/olshop-koneksi)** — E-commerce app
- MVC + Service Layer architecture
- Midtrans payment integration
- RajaOngkir shipping API
- Feature testing with PHPUnit

## 📫 Contact
- Email: budi@email.com
- LinkedIn: linkedin.com/in/budi
```

### 1.2 Contribution Graph

```
Jangan khawatir kalau contribution graph kosong.
Yang penting:
- Ada 1-2 repository bagus yang aktif
- Commit message jelas (bukan "update" terus)
- README rapi
```

---

## 2. Repository Setup

### 2.1 Checklist Repository

```text
✅ Nama repository profesional (bukan "test-laravel")
✅ README.md dengan deskripsi
✅ Screenshot (letak di folder screenshots/)
✅ Tech stack badges
✅ Instalasi instructions
✅ .gitignore benar
✅ License (MIT)
```

### 2.2 README Template

```markdown
# Olshop Koneksi

E-commerce app built with Laravel 13.

## 🧰 Tech Stack
- Laravel 13 / PHP 8.3
- Tailwind CSS 4 + Alpine.js 3
- Vite
- MySQL / SQLite
- Midtrans Payment Gateway
- RajaOngkir Shipping API

## ✨ Features
- Product catalog with categories
- Shopping cart (session-based)
- Checkout with shipping cost calculation
- Midtrans payment integration
- Admin dashboard
- Order management

## 📸 Screenshots
![Homepage](screenshots/home.png)
![Cart](screenshots/cart.png)

## 🚀 Installation
```bash
git clone https://github.com/username/olshop-koneksi.git
cd olshop-koneksi
composer install
cp .env.example .env
php artisan key:generate
npm install
npm run build
```

## 🧪 Testing
```bash
php artisan test
```

## 📄 License
MIT
```

---

## 3. Badges

Tambahkan badge di README untuk tampilan profesional:

```markdown
![PHP](https://img.shields.io/badge/PHP-8.3-777BB4)
![Laravel](https://img.shields.io/badge/Laravel-13-FF2D20)
![Tailwind](https://img.shields.io/badge/Tailwind-4-06B6D4)
![License](https://img.shields.io/badge/License-MIT-green)
```

---

## 4. Git Commit Messages

```
✅ "Add cart feature with session-based storage"
✅ "Fix N+1 query in product listing"
✅ "Add feature test for checkout flow"
❌ "update"
❌ "fix bug"
❌ "asdf"
```

---

## 🧪 Latihan

1. **Cek GitHub.** Buka github.com/username-mu. Apakah profile sudah rapi?

2. **Buat profile README.** Buat repository `username/username`, tambah README.

3. **Rapikan repo.** Cek repository olshop-koneksi di GitHub. Apakah README, .gitignore, dan struktur sudah rapi?

4. **Tambahkan screenshot.** Buat folder `screenshots/`, screenshot halaman utama, cart, checkout.

---

## 🔗 Referensi

- [GitHub Profile README](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-profile)
- [Shields.io (Badges)](https://shields.io/)
- [Awesome README](https://github.com/matiassingers/awesome-readme)
