
# 🧠 OpenKB Protocol — Penjelasan Lengkap

> Dokumentasi ini menjelaskan **OpenKB sebagai protokol** — sistem direktori knowledge base terstruktur — serta bagaimana repo ini menjadi salah satu implementasinya.

---

## 1. Apa Itu OpenKB?

**OpenKB (Open Knowledge Base)** adalah sebuah **protokol / standar direktori** untuk menyimpan dan mengorganisir *knowledge base* yang terstruktur di dalam sebuah project. Konsepnya sederhana: setiap project bisa memiliki folder `.openkb/` di root-nya yang berisi dokumentasi, kurikulum, panduan, atau pengetahuan apapun yang relevan dengan project tersebut.

OpenKB dirancang oleh **sandikodev** dan wacananya akan dirilis secara resmi melalui organisasi **ainjiner** di:

- https://github.com/sandikodev/openkb
- https://github.com/ainjiner/openkb

> ⚠️ Status: **Belum dirilis secara publik.** Repo di atas masih dalam pengembangan.

---

## 2. Posisi OpenKB di Ekosistem Knowledge Base

OpenKB berada dalam kategori yang sama dengan protokol knowledge base lainnya:

| Protokol | Basis | Fokus |
|----------|-------|-------|
| **`.openkb`** | Direktori | Knowledge base terbuka & terstruktur |
| **`.sisyphus`** | Direktori | AI agent instructions & workflow |
| **`.codex`** | Direktori | Kode & dokumentasi teknis |
| **`.gemini`** | Direktori | AI prompting & context |
| **`.kiro`** | Direktori | (informasi lebih lanjut menyusul) |

Setiap protokol memiliki format, struktur, dan filosofi yang berbeda — namun semuanya menggunakan pendekatan **directory-as-a-protocol**: folder khusus di root project yang berisi file-file dengan aturan tertentu.

---

## 3. Filosofi OpenKB

```
Project Root/
├── .openkb/          ← Semua pengetahuan tentang project ini
│   ├── README.md     ← Gerbang masuk (onboarding)
│   ├── docs/         ← Dokumentasi teknis
│   ├── journey/      ← Kurikulum / learning path
│   └── ...           ← Apapun yang relevan
├── src/              ← Kode utama
└── ...
```

Prinsip utama OpenKB:

| Prinsip | Penjelasan |
|---------|-----------|
| **🧩 Embedded** | Pengetahuan disimpan **di dalam** project, bukan di wiki/notion terpisah |
| **📂 Directory-as-a-Protocol** | Cukup dengan ada folder `.openkb/`, sebuah project sudah "menerapkan" OpenKB |
| **📖 Language-Agnostic** | Bisa diisi dokumen apapun — Markdown, gambar, kode contoh, dll |
| **🔗 Referensi Kontekstual** | Setiap dokumen bisa merujuk ke file asli di codebase |
| **♻️ Portable** | Folder `.openkb/` bisa di-copy/dipindah ke project lain |
| **🌐 Open** | Siapa pun bisa mengadopsi dan mengadaptasi |

---

## 4. Struktur Standar OpenKB

```
.openkb/
├── README.md                   ← Onboarding & penjelasan isi
├── DOKUMEN-UTAMA.md            ← (opsional) Dokumentasi inti
│
├── docs/                       ← Dokumentasi teknis & arsitektur
│   └── ...
│
├── journey/                    ← Kurikulum / learning path (opsional)
│   ├── 00-CURRICULUM.md        ← Peta belajar induk
│   └── FASE-*/                ← Fase-fase pembelajaran
│
├── referensi/                  ← (opsional) Link & referensi eksternal
│   └── ...
│
└── tools/                      ← (opsional) Script, konfigurasi, panduan tools
    └── ...
```

> **Catatan:** Struktur di atas adalah **saran (convention over configuration)** — bukan aturan kaku. Setiap project bisa menyesuaikan.

---

## 5. Repo Ini: Implementasi OpenKB untuk Laravel

Repo **`Hans02-Neo/openkb`** (yang sedang kamu baca) adalah **salah satu implementasi** dari protokol OpenKB.

| Aspek | Penjelasan |
|-------|-----------|
| **Protokol** | Mengadopsi direktori `.openkb/` sesuai standar OpenKB |
| **Isi** | Kurikulum belajar Laravel 15 fase, dari nol hingga junior developer |
| **Codebase Pendamping** | `olshop-koneksi` — aplikasi e-commerce Laravel production-grade |
| **Target** | Developer Indonesia yang ingin belajar Laravel secara terstruktur |
| **Bahasa** | Indonesia |

Dengan kata lain:

> Repo ini bukan **protokol OpenKB itu sendiri**, melainkan **salah satu instance / implementasi** dari protokol tersebut — khusus untuk learning journey Laravel.

Jika kamu ingin membuat implementasi OpenKB-mu sendiri (misal untuk project Python, Node.js, atau manajemen tim), kamu cukup:

1. Buat folder `.openkb/` di root project-mu
2. Isi dengan dokumentasi, kurikulum, atau pengetahuan yang relevan
3. (Opsional) Publikasikan sebagai repo terpisah agar bisa dipasang sebagai submodule

---

## 6. Perbedaan dengan Protokol Lain

Protokol knowledge base seperti `.sisyphus`, `.codex`, `.gemini`, dan `.kiro` memiliki fokus yang berbeda:

| Protokol | Fokus Utama | Contoh Penggunaan |
|----------|------------|-------------------|
| **OpenKB** | Knowledge base terbuka & terstruktur | Dokumentasi project, kurikulum belajar, wiki internal |
| **.sisyphus** | AI agent instructions | Panduan untuk AI agar konsisten mengerjakan task |
| **.codex** | Kode & dokumentasi teknis | Standar kode, API docs, arsitektur |
| **.gemini** | AI prompting & context | Prompt template, few-shot examples |

Masing-masing bisa digunakan **berdampingan** dalam project yang sama:

```
project/
├── .openkb/          ← Pengetahuan & kurikulum
├── .sisyphus/        ← AI agent instructions
├── .codex/           ← Standar kode & dokumentasi teknis
├── .gemini/          ← AI prompting context
└── src/
```

---

## 7. Cara Mengadopsi OpenKB di Project Lain

### Opsi A: Manual

```bash
mkdir .openkb
echo "# Knowledge Base" > .openkb/README.md
# Mulai tambah dokumen sesuai kebutuhan
```

### Opsi B: Submodule (jika repo ini publik)

```bash
# Di root project-mu:
git submodule add <url-repo-openkb> .openkb
```

### Opsi C: Fork

Fork repo ini, hapus konten yang tidak relevan, ganti dengan konten project-mu sendiri.

---

## 8. Status & Roadmap OpenKB

| Item | Status |
|------|--------|
| Protokol OpenKB oleh sandikodev | 🔜 Belum dirilis |
| Repo `ainjiner/openkb` | 🔜 Rencana rilis |
| Implementasi ini (Laravel Learning Journey) | ✅ Dirilis |

---

## 9. Referensi

- https://github.com/sandikodev/openkb — Creator OpenKB
- https://github.com/ainjiner/openkb — Rencana rilis resmi
- https://github.com/Hans02-Neo/openkb — Implementasi ini

---

<p align="center">
  <i>Dokumen ini dibuat untuk meluruskan pemahaman tentang OpenKB sebagai protokol,</i><br>
  <i>serta posisi repo ini di dalam ekosistem tersebut.</i>
</p>
