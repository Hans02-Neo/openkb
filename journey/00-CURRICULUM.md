# 📚 Learning Journey — Koneksi Store

## Dari Nol Menjadi Junior Developer (Laravel Web Engineer)

Dokumen ini adalah **kurikulum induk** — peta jalan belajar yang terstruktur dan bertahap.
Setiap fase membangun fondasi dari fase sebelumnya. Bacalah secara berurutan.

---

## 🧭 Cara Menggunakan Journey Ini

```
Fase 1 ──▶ Fase 2 ──▶ Fase 3 ──▶ ... ──▶ Fase 10
  │           │           │                     │
  ▼           ▼           ▼                     ▼
  Fondasi     Bangun      Perdalam              Integrasi
  Mutlak      Pondasi     Pemahaman             & Produksi
```

| Simbol | Arti |
|--------|------|
| 📖 | Baca & pahami (teori) |
| ✍️ | Praktik langsung (coding) |
| 🔬 | Eksplorasi (sendiri) |
| ⚡ | Penting / wajib dikuasai |

**Estimasi waktu**: Tidak ada target — pahami setiap konsep sampai kamu bisa
menjelaskan ke orang lain sebelum lanjut ke fase berikutnya.

---

## 📋 Daftar Dokumen Journey

```
.openkb/journey/
│
├── 00-CURRICULUM.md              ← Kamu di sini (daftar isi induk)
│
├── FASE-01-FONDASI-MUTLAK/
│   ├── 01-01-apa-itu-pemrograman.md         ✨ BARU
│   ├── 01-02-cara-kerja-komputer.md         ✨ BARU
│   ├── 01-03-variabel-tipe-data-logika.md   ✨ BARU
│   └── 01-04-algoritma-dasar.md             ✨ BARU
│
├── FASE-02-PHP-FUNDAMENTAL/
│   ├── 02-01-pengantar-php.md               ✨ BARU
│   ├── 02-02-sintaks-dasar-php.md           ✨ BARU
│   ├── 02-03-array-dan-string.md            ✨ BARU
│   ├── 02-04-fungsi-dan-scope.md            ✨ BARU
│   └── 02-05-php-dan-web-server.md          ✨ BARU
│
├── FASE-03-OOP-OBJECT-ORIENTED-PROGRAMMING/
│   ├── 03-01-apa-itu-oop.md                 ✨ BARU
│   ├── 03-02-class-object-property.md       ✨ BARU
│   ├── 03-03-inheritance-polymorphism.md    ✨ BARU
│   ├── 03-04-encapsulation-abstraction.md   ✨ BARU
│   ├── 03-05-dependency-injection.md        ✨ BARU
│   ├── 03-06-oop-di-php.md                  ✨ BARU
│   └── 03-07-oop-di-codebase-ini.md         ✨ BARU
│
├── FASE-04-FUNCTIONAL-PROGRAMMING/
│   ├── 04-01-paradigma-functional.md        ✨ BARU
│   ├── 04-02-first-class-citizen.md         ✨ BARU
│   ├── 04-03-array-map-filter-reduce.md     ✨ BARU
│   └── 04-04-functional-di-laravel.md       ✨ BARU
│
├── FASE-05-WEB-FUNDAMENTAL/
│   ├── 05-01-cara-kerja-web.md              ✨ BARU
│   ├── 05-02-http-dan-rest.md               ✨ BARU
│   ├── 05-03-html-css-dasar.md              ✨ BARU
│   ├── 05-04-javascript-dasar.md            ✨ BARU
│   ├── 05-05-dom-dan-browser.md             ✨ BARU
│   ├── 05-06-client-vs-server.md            ✨ BARU
│   └── 05-07-form-dan-csrf.md              ✨ BARU
│
├── FASE-06-LARAVEL-DEEP-DIVE/
│   ├── 06-01-apa-itu-framework.md           ✨ BARU
│   ├── 06-02-arsitektur-laravel.md          ✨ BARU
│   ├── 06-03-routing-web-php.md             ✨ BARU
│   ├── 06-04-controller-dan-request.md      ✨ BARU
│   ├── 06-05-model-dan-eloquent-orm.md      ✨ BARU
│   ├── 06-06-blade-template-engine.md       ✨ BARU
│   ├── 06-07-middleware-dan-auth.md          ✨ BARU
│   ├── 06-08-service-layer.md               ✨ BARU
│   ├── 06-09-migration-dan-seeder.md        ✨ BARU
│   ├── 06-10-queue-dan-job.md               ✨ BARU
│   └── 06-11-notifikasi-dan-mail.md         ✨ BARU
│
├── FASE-07-DATABASE-DAN-SQL/
│   ├── 07-01-apa-itu-database.md            ✨ BARU
│   ├── 07-02-sql-dasar-crud.md              ✨ BARU
│   ├── 07-03-relationship-dan-join.md       ✨ BARU
│   ├── 07-04-erd-dan-schema-design.md       ✨ BARU
│   ├── 07-05-index-dan-performance.md       ✨ BARU
│   ├── 07-06-n-plus-one-problem.md          ✨ BARU
│   └── 07-07-database-di-codebase-ini.md   ✨ BARU
│
├── FASE-08-DESIGN-PATTERN-DAN-ARSITEKTUR/
│   ├── 08-01-apa-itu-design-pattern.md      ✨ BARU
│   ├── 08-02-mvc-architecture.md            ✨ BARU
│   ├── 08-03-service-pattern.md             ✨ BARU
│   ├── 08-04-repository-pattern.md          ✨ BARU
│   ├── 08-05-observer-pattern.md            ✨ BARU
│   ├── 08-06-policy-dan-gate.md             ✨ BARU
│   └── 08-07-solid-principles.md            ✨ BARU
│
├── FASE-09-FRONTEND-DI-LARAVEL/
│   ├── 09-01-cara-kerja-vite.md             ✨ BARU
│   ├── 09-02-tailwind-css.md                ✨ BARU
│   ├── 09-03-alpine-js.md                   ✨ BARU
│   ├── 09-04-asset-management.md            ✨ BARU
│   ├── 09-05-hot-reload-hmr.md              ✨ BARU
│   └── 09-06-buildtime-vs-runtime.md        ✨ BARU
│
├── FASE-10-INFRASTRUKTUR-DAN-DEPLOYMENT/
│   ├── 10-01-apa-itu-web-server.md          ✨ BARU
│   ├── 10-02-apache-vs-nginx.md             ✨ BARU
│   ├── 10-03-laragon-di-windows.md          ✨ BARU
│   ├── 10-04-linux-dasar-untuk-dev.md       ✨ BARU
│   ├── 10-05-ssh-dan-terminal.md            ✨ BARU
│   ├── 10-06-docker-dasar.md                ✨ BARU
│   ├── 10-07-cpanel-dan-hosting.md          ✨ BARU
│   ├── 10-08-ftp-dan-file-transfer.md       ✨ BARU
│   ├── 10-09-env-dan-konfigurasi.md         ✨ BARU
│   └── 10-10-deployment-workflow.md         ✨ BARU
│
├── FASE-11-TOOLS-DAN-WORKFLOW/
│   ├── 11-01-apa-itu-composer.md            ✨ BARU
│   ├── 11-02-npm-dan-nodejs.md              ✨ BARU
│   ├── 11-03-git-dan-version-control.md     ✨ BARU
│   ├── 11-04-artisan-cli.md                 ✨ BARU
│   ├── 11-05-tinker-dan-repl.md             ✨ BARU
│   └── 11-06-ide-dan-produktivitas.md       ✨ BARU
│
├── FASE-12-TESTING-DAN-QA/
│   ├── 12-01-apa-itu-testing.md             ✨ BARU
│   ├── 12-02-phpunit-dasar.md               ✨ BARU
│   ├── 12-03-feature-test.md                ✨ BARU
│   ├── 12-04-testing-di-codebase-ini.md    ✨ BARU
│   └── 12-05-debugging-strategy.md          ✨ BARU
│
├── FASE-13-CODE-QUALITY/
│   ├── 13-01-code-readability.md            ✨ BARU
│   ├── 13-02-psr-standard.md                ✨ BARU
│   ├── 13-03-convention-di-codebase.md      ✨ BARU
│   ├── 13-04-refactoring-dasar.md           ✨ BARU
│   └── 13-05-dokumentasi-kode.md            ✨ BARU
│
├── PRAKTIK-CODING/
│   ├── latihan-01-variabel-dan-tipe-data.md     ✨ BARU
│   ├── latihan-02-loop-dan-kondisi.md           ✨ BARU
│   ├── latihan-03-class-sederhana.md            ✨ BARU
│   ├── latihan-04-routing-dan-controller.md     ✨ BARU
│   ├── latihan-05-eloquent-query.md             ✨ BARU
│   ├── latihan-06-membuat-fitur-crud.md         ✨ BARU
│   ├── latihan-07-integrasi-api-payment.md      ✨ BARU
│   └── capstone-buat-fitur-sederhana.md         ✨ BARU
│
├── FASE-14-PORTFOLIO-DAN-KARIR/
│   ├── 14-01-portfolio-dari-codebase-ini.md       ✨ BARU
│   ├── 14-02-github-sebagai-portfolio.md          ✨ BARU
│   ├── 14-03-readme-yang-menarik.md               ✨ BARU
│   ├── 14-04-resume-untuk-junior-dev.md           ✨ BARU
│   ├── 14-05-persiapan-wawancara-teknis.md        ✨ BARU
│   ├── 14-06-pertanyaan-interview-laravel.md      ✨ BARU
│   ├── 14-07-linkedin-personal-branding.md        ✨ BARU
│   ├── 14-08-menjawab-pengalaman-kosong.md        ✨ BARU
│   └── 14-09-roadmap-menjadi-junior.md            ✨ BARU
│
└── FASE-15-BISNIS-DAN-INDUSTRI/
    ├── 15-01-web-industri-ekosistem.md            ✨ BARU
    ├── 15-02-dari-kode-ke-produk.md               ✨ BARU
    ├── 15-03-peran-tim-pengembang.md              ✨ BARU
    ├── 15-04-agile-dan-sdlc.md                    ✨ BARU
    ├── 15-05-codebase-ini-dalam-bisnis.md         ✨ BARU
    └── 15-06-menjadi-junior-developer.md          ✨ BARU
```

---

## 🗺️ Peta Belajar — Per Fase

### FASE 1 — Fondasi Mutlak
**Tujuan**: Paham apa itu "komputer", "program", "data", dan "logika".
| Dokumen | Prasyarat | Waktu |
|---------|-----------|-------|
| 01-01-apa-itu-pemrograman | Tidak ada | ⏱️ 30 menit |
| 01-02-cara-kerja-komputer | 01-01 | ⏱️ 30 menit |
| 01-03-variabel-tipe-data-logika | 01-02 | ⏱️ 45 menit |
| 01-04-algoritma-dasar | 01-03 | ⏱️ 45 menit |

**Output**: Kamu bisa menjelaskan apa itu program, variabel, dan logika dasar.

---

### FASE 2 — PHP Fundamental
**Tujuan**: Paham sintaks PHP dasar dan bagaimana PHP bekerja di web.
| Dokumen | Prasyarat |
|---------|-----------|
| 02-01-pengantar-php | FASE 1 |
| 02-02-sintaks-dasar-php | 02-01 |
| 02-03-array-dan-string | 02-02 |
| 02-04-fungsi-dan-scope | 02-03 |
| 02-05-php-dan-web-server | 02-04 |

**Output**: Bisa menulis skrip PHP sederhana yang menerima input dan menghasilkan output.

---

### FASE 3 — Object Oriented Programming
**Tujuan**: Paham paradigma OOP — ini WAJIB untuk Laravel.
| Dokumen | Prasyarat |
|---------|-----------|
| 03-01-apa-itu-oop | FASE 2 |
| 03-02-class-object-property | 03-01 |
| 03-03-inheritance-polymorphism | 03-02 |
| 03-04-encapsulation-abstraction | 03-03 |
| 03-05-dependency-injection | 03-04 |
| 03-06-oop-di-php | 03-05 |
| 03-07-oop-di-codebase-ini | 03-06 |

**Output**: Bisa membaca dan memahami class, object, inheritance, DI di kode Laravel.

---

### FASE 4 — Functional Programming
**Tujuan**: Paham paradigma functional — melengkapi OOP di PHP modern.
| Dokumen | Prasyarat |
|---------|-----------|
| 04-01-paradigma-functional | FASE 3 |
| 04-02-first-class-citizen | 04-01 |
| 04-03-array-map-filter-reduce | 04-02 |
| 04-04-functional-di-laravel | 04-03 |

**Output**: Paham closures, callables, collection pipeline di Laravel.

---

### FASE 5 — Web Fundamental
**Tujuan**: Paham cara kerja web, HTTP, browser, client vs server.
| Dokumen | Prasyarat |
|---------|-----------|
| 05-01-cara-kerja-web | FASE 2 |
| 05-02-http-dan-rest | 05-01 |
| 05-03-html-css-dasar | 05-02 |
| 05-04-javascript-dasar | 05-03 |
| 05-05-dom-dan-browser | 05-04 |
| 05-06-client-vs-server | 05-05 |
| 05-07-form-dan-csrf | 05-06 |

**Output**: Paham perjalanan request dari browser ke server dan kembali.

---

### FASE 6 — Laravel Deep Dive
**Tujuan**: Paham framework Laravel secara arsitektural.
| Dokumen | Prasyarat |
|---------|-----------|
| 06-01-apa-itu-framework | FASE 3 + FASE 5 |
| 06-02-arsitektur-laravel | 06-01 |
| 06-03-routing-web-php | 06-02 |
| 06-04-controller-dan-request | 06-03 |
| 06-05-model-dan-eloquent-orm | 06-04 |
| 06-06-blade-template-engine | 06-05 |
| 06-07-middleware-dan-auth | 06-06 |
| 06-08-service-layer | 06-07 |
| 06-09-migration-dan-seeder | 06-08 |
| 06-10-queue-dan-job | 06-09 |
| 06-11-notifikasi-dan-mail | 06-10 |

**Output**: Bisa menjelaskan alur request Laravel dari awal hingga response.

---

### FASE 7 — Database & SQL
**Tujuan**: Paham database relational, SQL, dan desain schema.
| Dokumen | Prasyarat |
|---------|-----------|
| 07-01-apa-itu-database | FASE 6 |
| 07-02-sql-dasar-crud | 07-01 |
| 07-03-relationship-dan-join | 07-02 |
| 07-04-erd-dan-schema-design | 07-03 |
| 07-05-index-dan-performance | 07-04 |
| 07-06-n-plus-one-problem | 07-05 |
| 07-07-database-di-codebase-ini | 07-06 |

**Output**: Bisa membaca migration, memahami relasi tabel, dan mengoptimasi query.

---

### FASE 8 — Design Pattern & Arsitektur
**Tujuan**: Paham pola arsitektur yang digunakan di codebase.
| Dokumen | Prasyarat |
|---------|-----------|
| 08-01-apa-itu-design-pattern | FASE 3 |
| 08-02-mvc-architecture | FASE 6 |
| 08-03-service-pattern | 08-02 |
| 08-04-repository-pattern | 08-03 |
| 08-05-observer-pattern | 08-04 |
| 08-06-policy-dan-gate | 08-05 |
| 08-07-solid-principles | 08-06 |

**Output**: Bisa mengidentifikasi pattern yang digunakan di setiap bagian codebase.

---

### FASE 9 — Frontend di Laravel
**Tujuan**: Paham bagaimana Vite, Tailwind, Alpine.js bekerja bersama Laravel.
| Dokumen | Prasyarat |
|---------|-----------|
| 09-01-cara-kerja-vite | FASE 5 |
| 09-02-tailwind-css | 09-01 |
| 09-03-alpine-js | 09-02 |
| 09-04-asset-management | 09-03 |
| 09-05-hot-reload-hmr | 09-04 |
| 09-06-buildtime-vs-runtime | 09-05 |

**Output**: Paham perbedaan build-time (Vite) vs runtime (PHP), dan bagaimana keduanya terintegrasi.

---

### FASE 10 — Infrastruktur & Deployment
**Tujuan**: Paham web server, hosting, dan cara deploy aplikasi.
| Dokumen | Prasyarat |
|---------|-----------|
| 10-01-apa-itu-web-server | FASE 5 |
| 10-02-apache-vs-nginx | 10-01 |
| 10-03-laragon-di-windows | 10-02 |
| 10-04-linux-dasar-untuk-dev | 10-03 |
| 10-05-ssh-dan-terminal | 10-04 |
| 10-06-docker-dasar | 10-05 |
| 10-07-cpanel-dan-hosting | 10-06 |
| 10-08-ftp-dan-file-transfer | 10-07 |
| 10-09-env-dan-konfigurasi | 10-08 |
| 10-10-deployment-workflow | 10-09 |

**Output**: Paham bagaimana aplikasi dari lokal bisa berjalan di production.

---

### FASE 11 — Tools & Workflow
**Tujuan**: Paham tools penting dalam ekosistem Laravel.
| Dokumen | Prasyarat |
|---------|-----------|
| 11-01-apa-itu-composer | FASE 3 |
| 11-02-npm-dan-nodejs | FASE 9 |
| 11-03-git-dan-version-control | FASE 2 |
| 11-04-artisan-cli | FASE 6 |
| 11-05-tinker-dan-repl | FASE 6 |
| 11-06-ide-dan-produktivitas | FASE 1 |

**Output**: Paham dan bisa menggunakan tools utama sehari-hari.

---

### FASE 12 — Testing & QA
**Tujuan**: Paham pentingnya testing dan cara menulis test.
| Dokumen | Prasyarat |
|---------|-----------|
| 12-01-apa-itu-testing | FASE 6 |
| 12-02-phpunit-dasar | 12-01 |
| 12-03-feature-test | 12-02 |
| 12-04-testing-di-codebase-ini | 12-03 |
| 12-05-debugging-strategy | 12-04 |

**Output**: Bisa menjalankan dan membaca test yang ada.

---

### FASE 13 — Code Quality
**Tujuan**: Paham standar kode yang baik dan convention.
| Dokumen | Prasyarat |
|---------|-----------|
| 13-01-code-readability | FASE 6 |
| 13-02-psr-standard | 13-01 |
| 13-03-convention-di-codebase | 13-02 |
| 13-04-refactoring-dasar | 13-03 |
| 13-05-dokumentasi-kode | 13-04 |

**Output**: Bisa menulis kode yang bersih, konsisten, dan mudah dibaca.

---

### FASE 14 — Portfolio & Karir
**Tujuan**: Mengemas ilmu yang sudah dipelajari menjadi portfolio yang bisa dipamerkan ke HR dan klien.
| Dokumen | Prasyarat |
|---------|-----------|
| 14-01-portfolio-dari-codebase-ini | FASE 6 |
| 14-02-github-sebagai-portfolio | 14-01 |
| 14-03-readme-yang-menarik | 14-02 |
| 14-04-resume-untuk-junior-dev | 14-03 |
| 14-05-persiapan-wawancara-teknis | 14-04 |
| 14-06-pertanyaan-interview-laravel | 14-05 |
| 14-07-linkedin-personal-branding | 14-06 |
| 14-08-menjawab-pengalaman-kosong | 14-07 |
| 14-09-roadmap-menjadi-junior | 14-08 |

**Output**: Memiliki GitHub portfolio, resume siap kirim, dan tahu cara menjawab pertanyaan teknis di wawancara.

---

### FASE 15 — Bisnis & Industri
**Tujuan**: Paham bagaimana software engineering bekerja dalam konteks bisnis dan tim.
| Dokumen | Prasyarat |
|---------|-----------|
| 15-01-web-industri-ekosistem | FASE 10 |
| 15-02-dari-kode-ke-produk | 15-01 |
| 15-03-peran-tim-pengembang | 15-02 |
| 15-04-agile-dan-sdlc | 15-03 |
| 15-05-codebase-ini-dalam-bisnis | 15-04 |
| 15-06-menjadi-junior-developer | 15-05 |

**Output**: Paham tempatmu dalam industri dan bagaimana software dibuat dari ide ke production.

---

## 🧩 Diagram Ketergantungan Antar Fase

```
FASE 1 (Fondasi)
   │
   ▼
FASE 2 (PHP Dasar)
   │
   ├────────────────────┐
   ▼                    ▼
FASE 3 (OOP)     FASE 5 (Web Fundamental)
   │                    │
   ▼                    │
FASE 4 (Functional)     │
   │                    │
   └──────┬───────────-─┘
          ▼
     FASE 6 (Laravel) ──────────── PRAKTIK-CODING ──┐
          │                              ▲          │
     ┌────┴────┐                         │          │
     ▼         ▼                    Latihan-latihan │
  FASE 7     FASE 8                  praktik        │
  (DB/SQL)   (Pattern)               coding         │
     │         │                       ▲            │
     └────┬────┘                       │            │
          ▼                            │            │
     FASE 9 (Frontend) ────────────────┘            │
          │                                         │
          ▼                                         │
     FASE 10 (Infra)                                │
          │                                         │
          ▼                                         │
     FASE 11 (Tools)                                │
          │                                         │
     ┌────┴────┐                                    │
     ▼         ▼                                    │
  FASE 12    FASE 13                                │
  (Testing)  (Quality)                              │
     │         │                                    │
     └────┬────┘                                    │
          ▼                                         │
     FASE 14 (Portfolio & Karir) ◄──────────────────┘
          │
          ▼
     FASE 15 (Bisnis & Industri)
```

---

## ✅ Checklist Pembelajaran

Gunakan checklist ini untuk melacak progres:

```
╔══════════════════════════════════════════════════════════╗
║                   MASTER CHECKLIST                       ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  FASE 1 — Fondasi Mutlak                                 ║
║  [ ] 01-01 Apa Itu Pemrograman                           ║
║  [ ] 01-02 Cara Kerja Komputer                           ║
║  [ ] 01-03 Variabel, Tipe Data, Logika                   ║
║  [ ] 01-04 Algoritma Dasar                               ║
║                                                          ║
║  FASE 2 — PHP Fundamental                                ║
║  [ ] 02-01 Pengantar PHP                                 ║
║  [ ] 02-02 Sintaks Dasar PHP                             ║
║  [ ] 02-03 Array dan String                              ║
║  [ ] 02-04 Fungsi dan Scope                              ║
║  [ ] 02-05 PHP dan Web Server                            ║
║                                                          ║
║  FASE 3 — OOP                                            ║
║  [ ] 03-01 Apa Itu OOP                                   ║
║  [ ] 03-02 Class, Object, Property                       ║
║  [ ] 03-03 Inheritance & Polymorphism                    ║
║  [ ] 03-04 Encapsulation & Abstraction                   ║
║  [ ] 03-05 Dependency Injection                          ║
║  [ ] 03-06 OOP di PHP                                    ║
║  [ ] 03-07 OOP di Codebase Ini                           ║
║                                                          ║
║  FASE 4 — Functional Programming                         ║
║  [ ] 04-01 Paradigma Functional                          ║
║  [ ] 04-02 First Class Citizen                           ║
║  [ ] 04-03 Array Map, Filter, Reduce                     ║
║  [ ] 04-04 Functional di Laravel                         ║
║                                                          ║
║  FASE 5 — Web Fundamental                                ║
║  [ ] 05-01 Cara Kerja Web                                ║
║  [ ] 05-02 HTTP & REST                                   ║
║  [ ] 05-03 HTML & CSS Dasar                              ║
║  [ ] 05-04 JavaScript Dasar                              ║
║  [ ] 05-05 DOM & Browser                                 ║
║  [ ] 05-06 Client vs Server                              ║
║  [ ] 05-07 Form & CSRF                                   ║
║                                                          ║
║  FASE 6 — Laravel Deep Dive                              ║
║  [ ] 06-01 Apa Itu Framework                             ║
║  [ ] 06-02 Arsitektur Laravel                            ║
║  [ ] 06-03 Routing (web.php)                             ║
║  [ ] 06-04 Controller & Request                          ║
║  [ ] 06-05 Model & Eloquent ORM                          ║
║  [ ] 06-06 Blade Template Engine                         ║
║  [ ] 06-07 Middleware & Auth                             ║
║  [ ] 06-08 Service Layer                                 ║
║  [ ] 06-09 Migration & Seeder                            ║
║  [ ] 06-10 Queue & Job                                   ║
║  [ ] 06-11 Notifikasi & Mail                             ║
║                                                          ║
║  FASE 7 — Database & SQL                                 ║
║  [ ] 07-01 Apa Itu Database                              ║
║  [ ] 07-02 SQL Dasar (CRUD)                              ║
║  [ ] 07-03 Relationship & Join                           ║
║  [ ] 07-04 ERD & Schema Design                           ║
║  [ ] 07-05 Index & Performance                           ║
║  [ ] 07-06 N+1 Problem                                   ║
║  [ ] 07-07 Database di Codebase Ini                      ║
║                                                          ║
║  FASE 8 — Design Pattern & Arsitektur                    ║
║  [ ] 08-01 Apa Itu Design Pattern                        ║
║  [ ] 08-02 MVC Architecture                              ║
║  [ ] 08-03 Service Pattern                               ║
║  [ ] 08-04 Repository Pattern                            ║
║  [ ] 08-05 Observer Pattern                              ║
║  [ ] 08-06 Policy & Gate                                 ║
║  [ ] 08-07 SOLID Principles                              ║
║                                                          ║
║  FASE 9 — Frontend di Laravel                            ║
║  [ ] 09-01 Cara Kerja Vite                               ║
║  [ ] 09-02 Tailwind CSS                                  ║
║  [ ] 09-03 Alpine.js                                     ║
║  [ ] 09-04 Asset Management                              ║
║  [ ] 09-05 Hot Reload & HMR                              ║
║  [ ] 09-06 Buildtime vs Runtime                          ║
║                                                          ║
║  FASE 10 — Infrastruktur & Deployment                    ║
║  [ ] 10-01 Apa Itu Web Server                            ║
║  [ ] 10-02 Apache vs Nginx                               ║
║  [ ] 10-03 Laragon di Windows                            ║
║  [ ] 10-04 Linux Dasar untuk Dev                         ║
║  [ ] 10-05 SSH & Terminal                                ║
║  [ ] 10-06 Docker Dasar                                  ║
║  [ ] 10-07 cPanel & Hosting                              ║
║  [ ] 10-08 FTP & File Transfer                           ║
║  [ ] 10-09 ENV & Konfigurasi                             ║
║  [ ] 10-10 Deployment Workflow                           ║
║                                                          ║
║  FASE 11 — Tools & Workflow                              ║
║  [ ] 11-01 Apa Itu Composer                              ║
║  [ ] 11-02 NPM & Node.js                                 ║
║  [ ] 11-03 Git & Version Control                         ║
║  [ ] 11-04 Artisan CLI                                   ║
║  [ ] 11-05 Tinker & REPL                                 ║
║  [ ] 11-06 IDE & Produktivitas                           ║
║                                                          ║
║  FASE 12 — Testing & QA                                  ║
║  [ ] 12-01 Apa Itu Testing                               ║
║  [ ] 12-02 PHPUnit Dasar                                 ║
║  [ ] 12-03 Feature Test                                  ║
║  [ ] 12-04 Testing di Codebase Ini                       ║
║  [ ] 12-05 Debugging Strategy                            ║
║                                                          ║
║  FASE 13 — Code Quality                                  ║
║  [ ] 13-01 Code Readability                              ║
║  [ ] 13-02 PSR Standard                                  ║
║  [ ] 13-03 Convention di Codebase                        ║
║  [ ] 13-04 Refactoring Dasar                             ║
║  [ ] 13-05 Dokumentasi Kode                              ║
║                                                          ║
║  PRAKTIK-CODING — Latihan Langsung                       ║
║  [ ] Latihan 01: Variabel & Tipe Data                    ║
║  [ ] Latihan 02: Loop & Kondisi                          ║
║  [ ] Latihan 03: Class Sederhana                         ║
║  [ ] Latihan 04: Routing & Controller                    ║
║  [ ] Latihan 05: Eloquent Query                          ║
║  [ ] Latihan 06: Membuat Fitur CRUD                      ║
║  [ ] Latihan 07: Integrasi API Payment                   ║
║  [ ] Capstone: Buat Fitur Sederhana Sendiri              ║
║                                                          ║
║  FASE 14 — Portfolio & Karir                             ║
║  [ ] 14-01 Portfolio dari Codebase Ini                   ║
║  [ ] 14-02 GitHub Sebagai Portfolio                      ║
║  [ ] 14-03 README yang Menarik                           ║
║  [ ] 14-04 Resume untuk Junior Dev                       ║
║  [ ] 14-05 Persiapan Wawancara Teknis                    ║
║  [ ] 14-06 Pertanyaan Interview Laravel                  ║
║  [ ] 14-07 LinkedIn Personal Branding                    ║
║  [ ] 14-08 Menjawab Pengalaman Kosong                    ║
║  [ ] 14-09 Roadmap Menjadi Junior                        ║
║                                                          ║
║  FASE 15 — Bisnis & Industri                             ║
║  [ ] 15-01 Web Industri Ekosistem                        ║
║  [ ] 15-02 Dari Kode ke Produk                           ║
║  [ ] 15-03 Peran Tim Pengembang                          ║
║  [ ] 15-04 Agile & SDLC                                  ║
║  [ ] 15-05 Codebase Ini dalam Bisnis                     ║
║  [ ] 15-06 Menjadi Junior Developer                      ║
╚══════════════════════════════════════════════════════════╝
```

> 📝 **Tips**: Copy checklist ini ke file terpisah `progress.md` dan tandai `[x]`
> setiap selesai membaca satu dokumen.

---

## 🎯 Target Akhir

Setelah menyelesaikan seluruh journey ini, kamu diharapkan mampu:

### 🧠 Pemahaman Teknis
1. **Membaca dan memahami** setiap baris kode di codebase `olshop-koneksi`
2. **Menjelaskan arsitektur** aplikasi secara keseluruhan (dari browser sampai database)
3. **Melakukan debugging** secara sistematis (server-side dan client-side)
4. **Menulis kode** yang mengikuti standar dan convention yang berlaku
5. **Memahami** konsep OOP, MVC, design pattern, dan SOLID principles

### 🛠️ Praktik & Produksi
6. **Membangun fitur** sederhana di atas codebase yang sudah ada
7. **Menjalankan testing** dan memahami laporan hasil test
8. **Men-deploy** aplikasi ke server production (VPS, shared hosting)
9. **Menggunakan tools** industri: Git, Composer, Artisan, NPM, Vite

### 💼 Karir & Portfolio
10. **Memiliki GitHub portfolio** yang menarik dengan README profesional
11. **Memiliki CV/resume** yang siap dilamar ke perusahaan
12. **Mampu menjawab** pertanyaan teknis saat wawancara kerja
13. **Mampu mempresentasikan** codebase ini sebagai bukti kompetensi ke HR atau calon klien
14. **Memahami** peran junior developer dalam tim engineering
15. **Siap melamar** posisi Junior Web Developer / Junior Software Engineer

### 🎯 Target Minimum (Sudah Layak Lamar)

Setelah menyelesaikan **Fase 1-6** + **Praktik Coding Latihan 1-4**, kamu sudah bisa:
- Membaca dan menjelaskan kode Laravel yang ada
- Membuat route, controller, dan blade view sederhana
- Melakukan CRUD dasar dengan Eloquent
- Mengetahui alur request-response Laravel
- **Ini sudah cukup untuk melamar posisi Junior Laravel Developer**

Setelah menyelesaikan **seluruh Fase (1-15)**, kamu sudah setara dengan:
- Junior Developer dengan pengalaman 6-12 bulan
- Mampu bekerja mandiri pada fitur-fitur standar
- Siap untuk posisi Junior Web Engineer / Fullstack Laravel Developer

---

*Dokumen ini adalah curriculum induk — setiap sub-dokumen akan dibuat satu per satu.*
*Simpan dokumen ini sebagai referensi dan peta jalan belajar.*
