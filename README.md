
<p align="center">
  <img src="https://img.shields.io/badge/PHP-8.3-777BB4?style=for-the-badge&logo=php&logoColor=white" alt="PHP 8.3" />
  <img src="https://img.shields.io/badge/Laravel-13-FF2D20?style=for-the-badge&logo=laravel&logoColor=white" alt="Laravel 13" />
  <img src="https://img.shields.io/badge/Fase-15-success?style=for-the-badge" alt="15 Fase" />
  <img src="https://img.shields.io/badge/Bahasa-Indonesia-blue?style=for-the-badge" alt="Bahasa Indonesia" />
</p>

<h1 align="center">📚 OpenKB — Learning Journey</h1>
<h3 align="center">Dari Nol Menjadi Junior Laravel Developer</h3>

<p align="center">
  <i>Structured, self-paced, project-based curriculum embedded in a real e-commerce codebase.</i>
</p>

> 📖 **Tentang OpenKB:** Repo ini adalah implementasi dari protokol **OpenKB** — sistem direktori knowledge base terstruktur oleh sandikodev. Bukan protokol itu sendiri. — [Baca selengkapnya →](TENTANG-OPENKB.md)

---

## 🧭 Apa Ini?

**OpenKB (Open Knowledge Base)** adalah kurikulum belajar **Laravel** terstruktur yang dibangun di atas codebase nyata — aplikasi e-commerce **Koneksi Store**.

Berbeda dengan tutorial biasa yang mengajarkan konsep secara terpisah, OpenKB menggunakan **codebase production-grade sebagai buku teks**. Setiap konsep dijelaskan lalu langsung di-map ke file asli di project, sehingga kamu membangun familiarity dengan struktur project nyata sejak hari pertama.

> 💡 **Filosofi:** *Learn by reading real code. Build by understanding the whole picture.*

---

## 🎯 Target Peserta

| Level | Deskripsi |
|-------|-----------|
| 🟢 **Absolute Beginner** | Belum pernah coding? Mulai dari Fase 01 |
| 🟡 **Pernah belajar PHP** | Langsung ke Fase 03 (OOP) atau Fase 06 (Laravel) |
| 🔵 **Sudah familiar Laravel** | Eksplorasi Fase 08 (Design Pattern) atau Fase 12 (Testing) |

---

## 📦 Prasyarat

Sebelum memulai, pastikan kamu sudah menyiapkan:

```bash
# 1. Clone project Laravel (codebase pendamping)
git clone <url-repo-olshop-koneksi>
cd olshop-koneksi

# 2. Install dependencies
composer install
npm install

# 3. Environment setup
cp .env.example .env
php artisan key:generate

# 4. Jalankan development server
php artisan serve
# atau via Laragon
```

> ⚠️ OpenKB ini adalah **materi belajar pendamping**. Kamu menjalankan codebase `olshop-koneksi` dan membaca OpenKB secara bersamaan.

---

## 🗺️ Peta Belajar — 15 Fase

```
FASE 1 ──▶ FASE 2 ──▶ FASE 3 ──▶ ... ──▶ FASE 15
  │           │           │                     │
  ▼           ▼           ▼                     ▼
Fondasi     Bangun      Perdalam            Siap Kerja
Mutlak      Pondasi     Pemahaman           & Berkarir
```

| # | Fase | Topik | Estimasi |
|---|------|-------|----------|
| 01 | **Fondasi Mutlak** | Apa itu programming, cara kerja komputer, variabel, algoritma | 1-2 minggu |
| 02 | **PHP Fundamental** | Sintaks PHP, array, string, fungsi, web server | 2-3 minggu |
| 03 | **OOP** | Class, inheritance, polymorphism, DI, OOP di codebase | 2-3 minggu |
| 04 | **Functional Programming** | Paradigma fungsional, first-class citizen, map/filter/reduce | 1 minggu |
| 05 | **Web Fundamental** | HTTP, REST, HTML, CSS, JS, DOM, CSRF | 2 minggu |
| 06 | **Laravel Deep Dive** | Routing, controller, Eloquent, Blade, middleware, service layer, queue, notifikasi | 4-6 minggu |
| 07 | **Database & SQL** | SQL, relationship, ERD, N+1 problem, indexing | 2 minggu |
| 08 | **Design Pattern** | MVC, Service, Repository, Observer, Policy, SOLID | 2-3 minggu |
| 09 | **Frontend di Laravel** | Vite, Tailwind, Alpine.js, asset management, HMR | 1-2 minggu |
| 10 | **Infrastruktur & Deployment** | Web server, Laragon, Linux, Docker, cPanel, SSH, deployment | 2-3 minggu |
| 11 | **Tools & Workflow** | Composer, NPM, Git, Artisan, Tinker, IDE | 1-2 minggu |
| 12 | **Testing & QA** | PHPUnit, Feature Test, debugging strategy | 2 minggu |
| 13 | **Code Quality** | PSR standard, refactoring, code documentation | 1 minggu |
| 14 | **Portfolio & Karir** | Resume, GitHub, LinkedIn, interview preparation | 1-2 minggu |
| 15 | **Bisnis & Industri** | Agile, SDLC, peran tim, dari kode ke produk | 1 minggu |

---

## 📂 Struktur Repository

```
.openkb/
├── README.md                          ← Kamu di sini
│
├── journey/                           ← Kurikulum utama (15 fase)
│   ├── 00-CURRICULUM.md               ← Daftar isi induk
│   ├── FASE-01-FONDASI-MUTLAK/        ← 4 dokumen
│   ├── FASE-02-PHP-FUNDAMENTAL/       ← 5 dokumen
│   ├── FASE-03-OOP/                   ← 7 dokumen
│   ├── FASE-04-FUNCTIONAL/            ← 4 dokumen
│   ├── FASE-05-WEB-FUNDAMENTAL/       ← 7 dokumen
│   ├── FASE-06-LARAVEL-DEEP-DIVE/     ← 11 dokumen
│   ├── FASE-07-DATABASE/              ← 7 dokumen
│   ├── FASE-08-DESIGN-PATTERN/        ← 7 dokumen
│   ├── FASE-09-FRONTEND/              ← 6 dokumen
│   ├── FASE-10-INFRASTRUKTUR/         ← 10 dokumen
│   ├── FASE-11-TOOLS/                 ← 6 dokumen
│   ├── FASE-12-TESTING/               ← 5 dokumen
│   ├── FASE-13-CODE-QUALITY/          ← 5 dokumen
│   ├── FASE-14-PORTFOLIO/             ← 9 dokumen
│   ├── FASE-15-BISNIS/                ← 6 dokumen
│   └── PRAKTIK-CODING/                ← 8 latihan praktik
│
├── laravel-telescope.md               ← Panduan debugging Telescope
├── memahami-ekosistem-web-laravel.md  ← Ekosistem web & Laravel
├── laragon-vs-artisan-serve.md        ← Dev environment comparison
└── strategi-penyelesaian-project.md   ← Strategi menyelesaikan fitur
```

---

## 🚀 Cara Menggunakan

### Metode 1: Belajar Berurutan (Rekomendasi)

```bash
# Mulai dari Fase 01, baca setiap file .md secara berurutan
# Setiap dokumen punya prasyarat yang harus diselesaikan sebelumnya
```

1. Buka `journey/00-CURRICULUM.md` — pahami peta belajarnya
2. Mulai dari `FASE-01/01-01-apa-itu-pemrograman.md`
3. Setiap dokumen punya bagian **Latihan** di akhir — kerjakan!
4. Setelah beberapa fase, buka `PRAKTIK-CODING/` untuk latihan langsung
5. Lanjut ke fase berikutnya

### Metode 2: Belajar Per Topik

```bash
# Langsung ke topik yang kamu butuhkan
```

| Butuh | Buka |
|-------|------|
| Paham MVC di Laravel | `FASE-08/08-02-mvc-architecture.md` |
| Cara kerja Eloquent | `FASE-06/06-05-model-dan-eloquent-orm.md` |
| Deploy ke hosting | `FASE-10/10-07-cpanel-dan-hosting.md` |
| Persiapan interview | `FASE-14/14-06-pertanyaan-interview-laravel.md` |

### Metode 3: Belajar Sambil Praktek

```bash
# Setelah membaca beberapa fase, langsung praktek:
# Buka PRAKTIK-CODING/ dan kerjakan latihan
```

Latihan tersedia:
- `latihan-01` — Variabel dan tipe data ⚡
- `latihan-02` — Loop dan kondisi ⚡
- `latihan-03` — Class sederhana
- `latihan-04` — Routing dan controller ⚡
- `latihan-05` — Eloquent query ⚡
- `latihan-06` — Membuat fitur CRUD ⚡
- `latihan-07` — Integrasi API payment
- `capstone` — Buat fitur sederhana ⚡

---

## 🛠️ Tools Pendukung

Selama belajar, kamu akan sering menggunakan:

| Tool | Fungsi |
|------|--------|
| **Laragon** | Development environment (Windows) |
| **Composer** | Dependency manager PHP |
| **NPM** | Dependency manager JavaScript |
| **Git** | Version control |
| **Artisan** | CLI Laravel (php artisan) |
| **Tinker** | REPL interaktif Laravel |
| **Telescope** | Debugging & monitoring |
| **PHPUnit** | Testing framework |

---

## 📖 Cara Membaca Dokumen

Setiap dokumen mengikuti format:

```markdown
## 📋 Ringkasan          ← Apa yang akan kamu pelajari
## 1. Konsep Dasar       ← Penjelasan teori
## 2. Implementasi       ← Contoh kode dan mapping ke codebase
## 3. Di Codebase Ini    ← 🔑 Bagian PALING penting — lihat file asli!
## 🧪 Latihan            ← Coba sendiri
## 🔗 Referensi          ← Link dan file terkait
```

> **🔑 Tips:** Bagian **"Di Codebase Ini"** adalah yang paling berharga. Jangan hanya membaca — buka file yang disebutkan, baca kode aslinya, pahami strukturnya.

---

## 📊 Status Pengerjaan

| Materi | Status |
|--------|--------|
| Fase 01 — Fondasi Mutlak | ✅ Selesai |
| Fase 02 — PHP Fundamental | ✅ Selesai |
| Fase 03 — OOP | ✅ Selesai |
| Fase 04 — Functional Programming | ✅ Selesai |
| Fase 05 — Web Fundamental | ✅ Selesai |
| Fase 06 — Laravel Deep Dive | ✅ Selesai |
| Fase 07 — Database & SQL | ✅ Selesai |
| Fase 08 — Design Pattern | ✅ Selesai |
| Fase 09 — Frontend di Laravel | ✅ Selesai |
| Fase 10 — Infrastruktur & Deployment | ✅ Selesai |
| Fase 11 — Tools & Workflow | ✅ Selesai |
| Fase 12 — Testing & QA | ✅ Selesai |
| Fase 13 — Code Quality | ✅ Selesai |
| Fase 14 — Portfolio & Karir | ✅ Selesai |
| Fase 15 — Bisnis & Industri | ✅ Selesai |

---

## 🤝 Kontribusi

Materi ini dikembangkan untuk kebutuhan belajar tim internal. Jika kamu ingin berkontribusi:

1. **Perbaiki typo/error** — buka PR, kami review
2. **Tambah latihan** — tambahkan di `PRAKTIK-CODING/`
3. **Saran perbaikan** — buka issue di repository ini

---

## 📄 Lisensi

Hak cipta milik tim pengembang. Digunakan untuk keperluan belajar internal dan publik.

---

<p align="center">
  <b>Selamat belajar! 🚀</b><br>
  <i>"The only way to learn a new programming language is by writing programs in it." — Dennis Ritchie</i>
</p>
