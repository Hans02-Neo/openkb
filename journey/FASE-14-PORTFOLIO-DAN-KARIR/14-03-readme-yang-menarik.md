# 14-03: README yang Menarik

> **Fase**: 14 — Portfolio & Karir  
> **Prasyarat**: 14-02-github-sebagai-portfolio  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: README, documentation, project overview, markdown, first impression

---

## 📋 Ringkasan

README adalah **halaman depan** repository — first impression untuk HR, interviewer, atau kontributor. README yang baik menjelaskan "apa, kenapa, bagaimana" dalam beberapa detik.

**Target pemahaman:**
- Kamu bisa menulis README yang profesional
- Kamu paham struktur README
- Kamu bisa tambah badges, screenshot, dan link

---

## 1. Struktur README

```markdown
# Nama Project

[Badges] [Link Demo]

## 📋 Deskripsi
1-2 paragraf jelas tentang project ini.

## 🧰 Tech Stack
Daftar teknologi yang dipakai.

## ✨ Features
- Fitur 1
- Fitur 2

## 📸 Screenshots
Gambar tampilan aplikasi.

## 🚀 Installation
Langkah-langkah menjalankan project.

## 🧪 Testing
Cara menjalankan test.

## 📄 License
MIT
```

---

## 2. Contoh README Codebase Ini

```markdown
# Olshop Koneksi

![Laravel](https://img.shields.io/badge/Laravel-13-FF2D20)
![PHP](https://img.shields.io/badge/PHP-8.3-777BB4)
![Tailwind](https://img.shields.io/badge/Tailwind-4-06B6D4)
![License](https://img.shields.io/badge/License-MIT-green)

E-commerce application built with Laravel 13, featuring product catalog,
shopping cart, checkout with Midtrans payment, and admin dashboard.

## Tech Stack

- **Backend:** Laravel 13, PHP 8.3, MySQL/SQLite
- **Frontend:** Tailwind CSS 4, Alpine.js 3, Vite
- **Payment:** Midtrans (Snap API)
- **Shipping:** RajaOngkir API
- **Testing:** PHPUnit (Feature Tests)

## Features

- Product catalog with category filtering
- Shopping cart (session-based)
- Checkout with shipping cost
- Midtrans payment integration
- Order management admin panel
- Customer address management

## Installation

```bash
git clone https://github.com/username/olshop-koneksi.git
cd olshop-koneksi
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed
npm install
npm run build
```

## Testing

```bash
php artisan test
```

## License

MIT
```

---

## 3. Tips README

```text
✅ Judul jelas
✅ Badges di atas (tech stack, license)
✅ Screenshot (letak di folder /screenshots)
✅ Installation step-by-step (copas-able)
✅ Bahasa Inggris (untuk exposure global)
❌ Jangan terlalu panjang (scroll > 3x)
❌ Jangan typo
❌ Jangan instruction yang salah
```

---

## 🧪 Latihan

1. **Baca README.md.** Buka `README.md` codebase (kalau ada). Evaluasi dari 1-10.

2. **Tulis ulang.** Tulis README baru untuk codebase ini dengan struktur di atas.

3. **Tambahkan screenshot.** Screenshot fitur utama (shop, cart, checkout, admin). Taruh di folder `screenshots/`.

4. **Tambahkan badges.** Cari badge untuk Laravel, PHP, Tailwind di shields.io. Tambahkan ke README.

---

## 🔗 Referensi

- [Make a README](https://www.makeareadme.com/)
- [Awesome README Examples](https://github.com/matiassingers/awesome-readme)
- [Shields.io](https://shields.io/)
