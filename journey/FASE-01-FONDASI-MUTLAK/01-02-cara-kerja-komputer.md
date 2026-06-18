# 01-02: Cara Kerja Komputer

> **Fase**: 1 — Fondasi Mutlak  
> **Prasyarat**: 01-01-apa-itu-pemrograman  
> **Waktu baca**: 45-60 menit  
> **Kata kunci**: CPU, RAM, storage, clock cycle, ALU, register, bus, memory hierarchy, von Neumann, fetch-execute cycle, proses, thread, kernel, sistem operasi  

---

## 📋 Ringkasan  

Dokumen sebelumnya menjelaskan apa itu program. Sekarang kita akan membedah **komputer itu sendiri** — bagaimana mesin ini benar-benar bekerja di level hardware.  

Kamu akan memahami:  
- Arsitektur dasar komputer (von Neumann)  
- Bagaimana CPU benar-benar mengeksekusi instruksi  
- Hirarki memori — kenapa ada RAM, cache, SSD  
- Bagaimana sistem operasi menjadi "manajer" komputer  
- Semua ini akan kamu lihat relevansinya saat kita masuk ke Laravel  

**Target pemahaman:**  
- Kamu bisa menjelaskan perjalanan sebuah instruksi dari kode PHP sampai ke CPU  
- Kamu paham kenapa RAM penting untuk web development  
- Kamu mengerti istilah-istilah seperti proses, thread, kernel  

---

## 1. Arsitektur von Neumann — Cetak Biru Semua Komputer  

Hampir semua komputer modern (laptop, server, HP) menggunakan arsitektur yang dirancang oleh **John von Neumann** pada tahun 1945.  

### Empat Komponen Utama  

```
┌─────────────────────────────────────────────────────────────┐
│                    ARSITEKTUR VON NEUMANN                    │
│                                                              │
│  ┌──────────────┐          ┌──────────────────────────┐     │
│  │   MEMORY     │          │        CPU                │     │
│  │   (RAM)      │◄────────▶│  ┌────────┐ ┌────────┐  │     │
│  │              │          │  │  CU    │ │  ALU   │  │     │
│  │ Menyimpan    │          │  │Control │ │Arithmetic│  │     │
│  │ data +       │          │  │ Unit   │ │ Logic   │  │     │
│  │ instruksi    │          │  │        │ │ Unit    │  │     │
│  └──────────────┘          │  └────────┘ └────────┘  │     │
│               ▲            │  ┌──────────────────┐   │     │
│               │            │  │    REGISTERS     │   │     │
│               │            │  │ (local memory    │   │     │
│               │            │  │  inside CPU)     │   │     │
│               │            │  └──────────────────┘   │     │
│               │            └──────────────────────────┘     │
│               │                        ▲                    │
│               ▼                        ▼                    │
│  ┌──────────────────────┐  ┌──────────────────────────┐     │
│  │   INPUT DEVICES      │  │   OUTPUT DEVICES          │     │
│  │   Keyboard, Mouse,   │  │   Monitor, Printer,       │     │
│  │   Network Card       │  │   Speaker                 │     │
│  └──────────────────────┘  └──────────────────────────┘     │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   BUS SYSTEM                          │   │
│  │  (Jalur data yang menghubungkan semua komponen)       │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 1.1 Memory (RAM)  

Bukan harddisk! RAM adalah **memori kerja** — tempat data dan instruksi disimpan **sementara** saat komputer menyala.  

| Sifat | RAM | Harddisk/SSD |
|-------|-----|-------------|
| Kecepatan | Sangat cepat (ns) | Lambat (ms) |
| Volatilitas | **Hilang** saat mati listrik | **Tetap** walau mati |
| Fungsi | Tempat kerja CPU | Penyimpanan permanen |
| Ukuran tipikal | 8-64 GB | 256 GB - 2 TB |
| Harga per GB | Lebih mahal | Lebih murah |

**Kenapa RAM penting untuk web development?**  

```
Saat kamu buka aplikasi Laravel:
1. File .php di-harddisk/SSD
2. Saat diakses, file .php disalin ke RAM ← LEBIH CEPAT
3. CPU ambil instruksi dari RAM
4. Hasil dikembalikan ke RAM
5. Dikirim ke browser

Semakin besar RAM → semakin banyak data yang bisa
ditampung → semakin jarang akses harddisk (yang lambat)
→ semakin cepat aplikasi.
```

### 1.2 CPU — Central Processing Unit  

CPU adalah **otak** komputer. Tugasnya: **mengambil instruksi dari RAM, mengeksekusinya, lalu menyimpan hasilnya kembali**.  

#### Komponen Internal CPU  

| Komponen | Fungsi | Analogi |
|----------|--------|---------|
| **Control Unit (CU)** | Mengatur alur eksekusi, membaca instruksi, mengarahkan komponen lain | **Dirigen orkestra** — mengatur kapan instrumen mana yang dimainkan |
| **Arithmetic Logic Unit (ALU)** | Melakukan operasi matematika (+, -, ×, ÷) dan logika (AND, OR, NOT, comparison) | **Kalkulator** — menghitung semua angka |
| **Registers** | Memori super-cepat di dalam CPU (beberapa byte saja) | **Meja kerja** — tempat menyimpan angka yang sedang dihitung |
| **Cache** | Memori kecil (MB) di dalam/di dekat CPU, lebih cepat dari RAM | **Nampan** — menyimpan bahan yang paling sering dipakai agar tidak ambil dari lemari setiap saat |

#### Fetch-Decode-Execute Cycle  

Ini adalah **detak jantung** komputer. CPU mengulangi siklus ini milyaran kali per detik.  

```
FASE 1: FETCH (Ambil)
   │
   │  CU membaca alamat memori berikutnya dari Program Counter (PC)
   │  CU mengambil instruksi dari RAM pada alamat tersebut
   │  Instruksi disimpan di Register Instruksi (IR)
   │
   ▼
FASE 2: DECODE (Terjemahkan)
   │
   │  CU menerjemahkan binary instruksi menjadi sinyal kontrol
   │  Contoh: 10001001 11001011 → "MOV BL, CL"
   │  (Artinya: pindahkan nilai dari register CL ke register BL)
   │
   ▼
FASE 3: EXECUTE (Laksanakan)
   │
   │  ALU melakukan operasi (jika matematika)
   │  atau data dipindahkan antar register
   │  atau data dibaca/ditulis ke RAM
   │
   ▼
KEMBALI KE FASE 1 (siklus berikutnya)
```

**Kecepatan siklus ini diukur dalam Hertz (Hz):**  

| Satuan | Arti | Contoh |
|--------|------|--------|
| 1 Hz | 1 siklus per detik | Tidak ada komputer seencer ini |
| 1 MHz | 1 juta siklus per detik | Komputer tahun 1980-an |
| 1 GHz | 1 milyar siklus per detik | **CPU kamu: ~2-4 GHz** |

CPU kamu (AMD Ryzen 7 PRO 4750U) memiliki kecepatan dasar ~1.7 GHz,  
bisa naik (boost) hingga ~4.1 GHz. Artinya: **4,1 MILYAR siklus per detik**.  

Setiap siklus, CPU bisa mengeksekusi 1 atau lebih instruksi (tergantung arsitektur — CPU modern bisa 4-8 instruksi per siklus via **pipelining** dan **superscalar**).  

### 1.3 System Bus — Jalan Data  

Komponen-komponen di atas saling terhubung melalui **bus** — kumpulan jalur listrik yang membawa data, alamat, dan sinyal kontrol.  

```
CPU ─── Data Bus ───→ RAM  (membawa data)
CPU ─── Address Bus ─→ RAM  (menunjukkan alamat memori)
CPU ─── Control Bus ─→ RAM  (sinyal: baca/tulis)
```

Lebar bus menentukan berapa banyak data yang bisa dipindahkan sekaligus:  

| Lebar Bus | Data per Transfer | Contoh |
|-----------|------------------|--------|
| 8-bit | 1 byte | Komputer jadul |
| 32-bit | 4 byte | Komputer era 2000-an |
| 64-bit | 8 byte | **Semua komputer modern** |

CPU kamu adalah **64-bit** — bisa memproses 8 byte data per siklus.  

---

## 2. Memory Hierarchy — Kenapa Ada Banyak Jenis Memori  

Jika semua memori bisa secepat register, tentu ideal. Tapi kecepatan berbanding terbalik dengan harga dan ukuran.  

```
Kecepatan ▲                        Harga per GB ▲
(Rendah)  │                        (Mahal)      │
          │  ┌──────────────────┐                │
          │  │   REGISTERS      │ ← 0.5 KB       │  Termahal
          │  │  (dalam CPU)     │   1 siklus     │
          │  ├──────────────────┤                │
          │  │   L1 CACHE       │ ← 32-64 KB     │  ▲
          │  │  (dalam inti CPU)│   3-5 siklus   │  │
          │  ├──────────────────┤                │  │
          │  │   L2 CACHE       │ ← 256-512 KB   │  │
          │  │  (dalam inti CPU)│   10-20 siklus  │  │
          │  ├──────────────────┤                │  │
          │  │   L3 CACHE       │ ← 4-16 MB      │  │
          │  │  (shared antar   │   30-50 siklus  │  │
          │  │   inti CPU)      │                │  │
          │  ├──────────────────┤                │  │
          │  │   RAM (DDR4/5)   │ ← 8-64 GB     │  │
          │  │  (external)      │   100-300      │  │
          │  │                  │   siklus       │  │
          │  ├──────────────────┤                │  │
          │  │   SSD/NVMe       │ ← 256 GB-2 TB │  │
          │  │  (storage)       │   JUTAAN       │  │
          │  │                  │   siklus       │  │  Termurah
          │  └──────────────────┘                ▼
          ▼
(Tinggi)                                        (Murah)
```

**Mengapa CPU tidak langsung ambil dari RAM/SSD?**  

Perbedaan kecepatan sangat ekstrem:  

| Operasi | Waktu (dalam nanodetik) | Skala (jika 1 siklus = 1 detik) |
|---------|------------------------|-------------------------------|
| 1 siklus CPU (4 GHz) | 0.25 ns | 1 detik |
| Baca dari L1 cache | 1 ns | 4 detik |
| Baca dari L2 cache | 3 ns | 12 detik |
| Baca dari L3 cache | 10 ns | 40 detik |
| Baca dari RAM | 100 ns | **6,5 menit** |
| Baca dari SSD | 150.000 ns | **~7 hari** |

Tanpa cache, CPU akan menghabiskan 99% waktunya hanya **menunggu** data dari RAM.  

### Contoh Relevan untuk Laravel  

```php
<?php
// Operasi dalam cache/RAM
$products = cache()->remember('products.all', 3600, function () {
    return Product::all();
});

// Pertama kali: query dari database (SSD → RAM)
// Berikutnya: ambil dari cache (RAM) — 1000x lebih cepat

// Tanpa cache: setiap halaman load, ambil dari database
// Dengan cache: ambil dari RAM — instant
?>
```

---

## 3. Proses, Thread, dan Sistem Operasi  

### 3.1 Apa Itu Sistem Operasi?  

Komputer tanpa sistem operasi (OS) hanyalah logam dan silikon.  
OS adalah **software pertama** yang jalan saat komputer dinyalakan — tugasnya:  

1. **Manajemen proses** — menentukan program mana yang jalan dan kapan  
2. **Manajemen memori** — membagi RAM ke program-program yang berjalan  
3. **Manajemen file** — mengatur data di harddisk/SSD  
4. **Manajemen I/O** — mengatur komunikasi dengan keyboard, mouse, monitor  
5. **Manajemen jaringan** — menangani koneksi internet  

**Contoh OS:** Windows 11, Linux (Ubuntu, Debian), macOS, Android.  

### 3.2 Proses vs Thread  

```
┌─────────────────────────────────────────────────────┐
│                  SISTEM OPERASI                       │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │         RAM (Memori Fisik)                    │   │
│  │                                                │   │
│  │  ┌─────────────┐  ┌─────────────┐             │   │
│  │  │  PROSES A    │  │  PROSES B   │             │   │
│  │  │  (Browser)   │  │  (VS Code)  │             │   │
│  │  │              │  │             │             │   │
│  │  │  Thread 1    │  │  Thread 1   │             │   │
│  │  │  Thread 2    │  │  Thread 2   │             │   │
│  │  │  Thread 3    │  │             │             │   │
│  │  └─────────────┘  └─────────────┘             │   │
│  │                                                │   │
│  │  ┌─────────────┐  ┌─────────────┐             │   │
│  │  │  PROSES C    │  │  PROSES D   │             │   │
│  │  │  (PHP)       │  │  (MySQL)    │             │   │
│  │  │              │  │             │             │   │
│  │  │  Thread 1    │  │  Thread 1   │             │   │
│  │  │              │  │  Thread 2   │             │   │
│  │  └─────────────┘  └─────────────┘             │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │              CPU (4 Cores / 8 Threads)        │   │
│  │                                                │   │
│  │  Core 1  Core 2  Core 3  Core 4                │   │
│  │  [  A1  ][  B1  ][  C1  ][  D1  ]             │   │
│  │  [  A2  ][  B2  ][  C1  ][  D2  ]             │   │
│  │  [  A3  ][  B1  ][  C1  ][  D1  ]             │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

| Konsep | Definisi | Analogi |
|--------|----------|---------|
| **Proses** | Program yang sedang berjalan — punya memori sendiri, terisolasi dari proses lain | **Restoran** — punya dapur sendiri, staf sendiri |
| **Thread** | "Alur eksekusi" di dalam proses — satu proses bisa punya banyak thread yang berbagi memori | **Koki** di restoran — bekerja parallel dalam dapur yang sama |
| **Multithreading** | Satu proses menjalankan beberapa thread secara bersamaan | Banyak koki masak dalam satu dapur |
| **Multitasking** | OS menjalankan banyak proses secara bergantian | OS jadi manajer yang bagi-bagi waktu CPU ke semua restoran |

### 3.3 CPU Anda: AMD Ryzen 7 PRO 4750U  

Spesifikasi dari sistem kamu:  

| Spesifikasi | Nilai |
|-------------|-------|
| Cores (inti) | 8 cores |
| Threads | 16 threads (via **Simultaneous Multithreading** / SMT) |
| Base clock | 1.7 GHz |
| Max boost | 4.1 GHz |
| Cache L2 | 4 MB (512 KB per core) |
| Cache L3 | 8 MB (shared) |
| TDP | 15W (efisien untuk laptop) |

**Apa artinya?**  

CPU ini bisa menangani **16 thread sekaligus** secara paralel.  
Saat kamu menjalankan:  

```
Laragon (Apache + PHP + MySQL) ─────────── 3 thread
Browser (dengan banyak tab) ────────────── 4 thread
VS Code ────────────────────────────────── 2 thread
Terminal ───────────────────────────────── 1 thread
Spotify ────────────────────────────────── 1 thread
Windows sendiri ────────────────────────── 3-5 thread

Total: ~15 thread — CPU kamu masih nyaman.
```

### 3.4 Context Switching — Ilusi "Bersamaan"  

Meskipun CPU punya banyak core, jumlah program yang berjalan biasanya lebih banyak dari jumlah core. OS menciptakan ilusi "semua berjalan bersamaan" dengan **context switching**:  

```
Waktu ────────────────────────────────────────────────────▶

Core 1: [Proses A][Proses A][Proses B][Proses A][Proses C]
Core 2: [Proses B][Proses C][Proses A][Proses C][Proses B]
Core 3: [Proses C][Proses B][Proses C][Proses B][Proses A]

Setiap proses dapat giliran beberapa milidetik,
kemudian diganti (switch) ke proses lain.
Karena switching sangat cepat, kita merasa semua berjalan
bersamaan (terutama di CPU multi-core).
```

**Biaya context switching:**  
- OS harus menyimpan state proses yang keluar (register, program counter)  
- Memuat state proses yang masuk  
- Butuh waktu ~1-10 mikrodetik — kecil, tapi terasa jika terlalu sering  

**Relevansi ke Laravel:**  

Saat web server Apache/Nginx menerima request:  
1. OS buat proses/thread baru untuk menangani request  
2. PHP dieksekusi dalam proses tersebut  
3. Setelah selesai, proses ditutup  
4. Siap untuk request berikutnya  

Ini kenapa web server bisa menangani banyak request "bersamaan" meskipun CPU terbatas.  

---

## 4. Booting — Dari Tombol ON sampai ke Desktop  

Apa yang terjadi saat kamu menekan tombol power?  

```
1. POWER ON
   │  Listrik mengalir ke motherboard
   │  CPU reset, mulai dari alamat tetap (0xFFFFFFF0)
   │
   ▼
2. BIOS/UEFI (Firmware)
   │  Program kecil di motherboard (dalam chip ROM)
   │  POST: Power-On Self-Test (cek RAM, CPU, GPU)
   │  Deteksi harddisk, keyboard, mouse
   │  Cari bootloader di harddisk/SSD
   │
   ▼
3. BOOTLOADER (misal: GRUB untuk Linux)
   │  Program kecil di awal harddisk/SSD
   │  Memberi pilihan: mau boot OS mana? (jika dual-boot)
   │  Muat kernel OS ke RAM
   │
   ▼
4. KERNEL (inti OS)
   │  Kernel adalah "inti" dari sistem operasi
   │  Dialah yang pertama kali di-load ke RAM
   │  Tugas kernel:
   │    - Inisialisasi hardware (driver)
   │    - Setup memory management
   │    - Setup process scheduler
   │    - Mount filesystem (buka harddisk)
   │
   ▼
5. SYSTEM SERVICES
   │  Service manager (systemd di Linux, svchost di Windows)
   │  Memulai service-service latar belakang:
   │    - Network manager (koneksi internet)
   │    - Display manager (GUI)
   │    - Audio service
   │    - SSH server (jika ada)
   │
   ▼
6. LOGIN SCREEN / DESKTOP
   │  Kamu bisa login dan mulai bekerja
```

**Relevansi ke Laravel:**  
Server di production (VPS) biasanya:  
- Tidak punya desktop (GUI) — hanya terminal  
- Boot langsung ke **multi-user mode** (runlevel 3)  
- Service yang jalan: Nginx/Apache, PHP-FPM, MySQL, Redis, SSH  
- Kamu masuk via SSH, bukan via layar monitor  

---

## 5. Interrupt — Cara Hardware Berbicara ke CPU  

Bayangkan kamu sedang membaca buku, dan seseorang menepuk bahumu.  
Itulah **interrupt** — sinyal dari hardware ke CPU yang bilang:  
"HEI! Ada sesuatu yang perlu ditangani SEKARANG!"  

### Cara Kerja Interrupt  

```
CPU sedang mengeksekusi program A
         │
         ▼
Keyboard mengirim sinyal INTERRUPT ke CPU
         │
         ▼
CPU menyimpan state program A (register, PC)
         │
         ▼
CPU menjalankan Interrupt Service Routine (ISR)
   → "Oh, user menekan tombol 'A'"
   → Simpan karakter 'A' ke buffer keyboard
         │
         ▼
CPU restore state program A
         │
         ▼
Lanjut mengeksekusi program A
```

### Jenis Interrupt  

| Jenis | Sumber | Contoh |
|-------|--------|--------|
| **Hardware Interrupt** | Perangkat keras | Keyboard, mouse, network card, harddisk |
| **Software Interrupt** | Program/sistem | System call (program minta layanan OS) |
| **Exception** | Error program | Division by zero, segfault |

**Relevansi ke Laravel:**  
Setiap kali kamu menekan tombol di keyboard, ada interrupt.  
Setiap kali data datang dari network (request HTTP), ada interrupt.  
CPU berhenti sejenak, proses data, lalu kembali ke pekerjaan sebelumnya.  
Semua ini terjadi tanpa kamu sadari — puluhan ribu kali per detik.  

---

## 6. System Call — Cara Program Minta Bantuan OS  

Program tidak bisa langsung mengakses hardware. Mereka harus minta izin ke OS melalui **system call**.  

```
Program PHP (user space)
   │
   │ "Saya mau menulis file log.txt"
   │
   ▼
System Call: write()
   │
   ▼
KERNEL (kernel space — privileged)
   │
   │ Driver harddisk dipanggil
   │ Data ditulis ke sektor tertentu
   │
   ▼
Harddisk menyimpan data
```

### Ringkasan Hak Akses  

```
Ring 0 (Kernel Mode) — OS, driver
   ▲── Hanya OS yang boleh akses langsung hardware
   │
Ring 3 (User Mode) — Program aplikasi
   └── Program harus panggil system call untuk akses hardware
```

Kenapa dipisah? **Keamanan dan stabilitas**.  
Jika program bisa langsung nulis ke harddisk, satu bug bisa  
merusak seluruh sistem. Dengan system call, OS bisa  
memvalidasi: "Apakah program ini berhak menulis file di sini?"  

**Relevansi ke Laravel:**  
```php
<?php
// Ini panggil system call secara tidak langsung
Storage::put('file.txt', 'Hello'); // write()
Log::info('User login');            // write()
Product::find(1);                   // read() — database via disk
echo "Hello";                       // write() — ke output
?>
```

Setiap operasi di atas memicu **ribuan system calls** yang berlapis-lapis.  

---

## 7. Virtual Memory — Membohongi Program  

Setiap program berpikir dia memiliki RAM sendiri. Ini karena **virtual memory**.  

```
Program A melihat: [0x0000...0xFFFF] ← 4 GB RAM virtual
Program B melihat: [0x0000...0xFFFF] ← 4 GB RAM virtual
                             │
                             ▼
                       RAM FISIK: 16 GB

MMU (Memory Management Unit) di CPU:
┌──────────────────────────────────────────────┐
│ Virtual A: 0x1000 → Fisik: 0xABCD            │
│ Virtual A: 0x2000 → Fisik: 0x5678            │
│ Virtual B: 0x1000 → Fisik: 0x9876            │
│ Virtual B: 0x2000 → Fisik: 0x4321            │
└──────────────────────────────────────────────┘
```

**Manfaat:**  
1. **Isolasi** — Program A tidak bisa akses memori Program B  
2. **Efisiensi** — Program bisa dialokasikan memori lebih besar dari RAM fisik (via **swap** ke harddisk)  
3. **Keamanan** — Program tidak tahu alamat fisik sebenarnya  

### Apa Itu Swap?  

Saat RAM penuh, OS memindahkan sebagian data ke harddisk/SSD (swap).  
Ini **jauh lebih lambat** (SSD 100x lebih lambat dari RAM).  

```
RAM penuh → OS pindahkan data yang jarang dipakai ke swap (harddisk)
           → RAM jadi longgar
           → Saat data di swap dibutuhkan lagi → dipindah balik ke RAM

☠️ JIKA TERLALU SERING → THRASHING → KOMPUTER NGE-LAG PARAH
```

**Relevansi ke Laravel:**  
```bash
# Cek penggunaan swap (di Linux):
free -h
              total        used        free      shared  buff/cache   available
Mem:           15Gi       8.2Gi       2.1Gi       0.5Gi       4.7Gi       6.1Gi
Swap:          4.0Gi       0.5Gi       3.5Gi
# Swap terpakai 0.5 GB — masih aman
```

Jika kamu menjalankan Laravel + MySQL + Redis + Node + Browser  
di laptop dengan RAM 8 GB, bisa cepat penuh dan mulai swap.  
Ini penyebab umum "laptop lemot" saat development.  

---

## 8. Ringkasan — Peta Mental Komputer

```
KOMPUTER
   │
   ├── HARDWARE
   │   ├── CPU (Otak)
   │   │   ├── Control Unit — Mengatur alur
   │   │   ├── ALU — Menghitung
   │   │   └── Registers — Memori super-cepat
   │   │
   │   ├── MEMORY (RAM) — Meja kerja
   │   │   └── Cache — Nampan (L1, L2, L3)
   │   │
   │   ├── STORAGE (SSD/HDD) — Lemari
   │   │
   │   └── I/O — Keyboard, Mouse, Monitor
   │
   ├── SOFTWARE
   │   ├── Sistem Operasi — Manajer
   │   │   ├── Kernel — Inti
   │   │   ├── Driver — Penterjemah hardware
   │   │   └── System Calls — Layanan untuk program
   │   │
   │   ├── Program/Aplikasi
   │   │   ├── Proses — Program jalan
   │   │   └── Thread — Alur dalam proses
   │   │
   │   └── User Interface — Interaksi manusia
   │
   └── ALUR EKSEKUSI
       └── Fetch → Decode → Execute → (ulang)
           [Ambil] [Terjemah] [Laksana]  MILYAR kali/detik
```

---

## 9. Glosarium Istilah Penting  

| Istilah | Arti |
|---------|------|
| **CPU** | Central Processing Unit — otak komputer |
| **Core** | Unit pemrosesan independen dalam CPU |
| **Thread (hardware)** | Alur eksekusi paralel dalam core (SMT/Hyperthreading) |
| **Clock Speed** | Kecepatan siklus CPU (GHz) |
| **ALU** | Arithmetic Logic Unit — bagian yang menghitung |
| **CU** | Control Unit — bagian yang mengatur |
| **Register** | Memori paling cepat, di dalam CPU |
| **Cache** | Memori kecil cepat antara CPU dan RAM |
| **RAM** | Random Access Memory — memori kerja sementara |
| **SSD** | Solid State Drive — penyimpanan permanen cepat |
| **Bus** | Jalur data antar komponen |
| **Interrupt** | Sinyal hardware yang menyela CPU |
| **System Call** | Cara program minta layanan OS |
| **Kernel** | Inti sistem operasi |
| **Proses** | Program yang sedang berjalan |
| **Thread (software)** | Sub-eksekusi dalam proses |
| **Context Switch** | Berganti proses oleh OS |
| **Virtual Memory** | Ilusi setiap program punya RAM sendiri |
| **Swap** | RAM virtual di harddisk |
| **POST** | Power-On Self-Test — cek hardware saat boot |
| **BIOS/UEFI** | Firmware di motherboard |
| **Bootloader** | Program yang memuat OS |
| **von Neumann** | Arsitektur standar komputer modern |

---

## 10. Relevansi ke Laravel Development  

Setiap konsep di atas berhubungan langsung dengan pekerjaanmu nanti:  

| Konsep Komputer | Relevansi ke Laravel |
|----------------|---------------------|
| **RAM & Cache** | `cache()->remember()` — hindari akses database berulang |
| **Memory hierarchy** | Kenapa Redis (in-memory) lebih cepat dari MySQL (disk) |
| **Proses & Thread** | Queue worker (`php artisan queue:work`) jalan sebagai proses |
| **Context switching** | Jangan buka terlalu banyak tool berat saat development |
| **System call** | `Storage::put()`, `Log::info()` — semua panggil kernel |
| **Interrupt** | Request HTTP → network interrupt → CPU proses |
| **Virtual memory** | Laravel bisa pakai lebih banyak memori dari yang tersedia (hati-hati!) |
| **Boot sequence** | Saat server restart, service harus auto-start (Supervisor) |

---

## 📌 PRAKTIK — Kerjakan Ini  

```
✍️  Buka Task Manager (Ctrl+Shift+Esc)
    → Tab Performance
    → Lihat: CPU (berapa core/logical processor?)
    → Lihat: Memory (berapa total RAM? Berapa yang terpakai?)
    → Lihat: Disk (apa tipe disk kamu? SSD/HDD?)

✍️  Di terminal, jalankan:
    php -r "for(\$i=0;\$i<1000000;\$i++){}; echo 'Selesai';"
    → Coba tebak berapa lama? (jawaban: sangat cepat — PHP + CPU 4 GHz)

✍️  Buka Task Manager → Tab Details
    → Cari proses php.exe, httpd.exe (Apache), mysqld.exe (MySQL)
    → Lihat berapa memory yang dipakai masing-masing
    → Catat: Laravel development bisa pakai 100-300 MB RAM per request

🔬  Nonaktifkan beberapa aplikasi, lalu refresh halaman Laravel
    → Apakah terasa lebih cepat? (jika iya — RAM sebelumnya penuh)
```

---

## 🔗 Referensi & Lanjutan  

| Topik | Di Journey Ini |
|-------|----------------|
| Binary & representasi data | FASE-01: 01-03-variabel-tipe-data-logika |
| Algoritma & logika | FASE-01: 01-04-algoritma-dasar |
| PHP — bagaimana kode dieksekusi | FASE-02: semua dokumen |
| Web server (Apache/Nginx) | FASE-10: 10-01-apa-itu-web-server |
| Deployment ke server (Linux) | FASE-10: semua dokumen |

---

*"Komputer adalah mesin yang paling bodoh sekaligus paling pintar —  
dia hanya melakukan apa yang diperintahkan,  
tapi melakukannya milyaran kali lebih cepat dari manusia."*

*Simpan pemahaman ini: setiap baris kode yang kamu tulis pada akhirnya  
hanyalah 0 dan 1 yang bergerak melalui sirkuit silikon, milyaran kali per detik,  
untuk menghasilkan halaman web yang kamu lihat di layar.*
