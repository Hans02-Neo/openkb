# 14-05: Persiapan Wawancara Teknis

> **Fase**: 14 — Portfolio & Karir  
> **Prasyarat**: 14-04-resume-untuk-junior-dev  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: interview, wawancara, technical interview, live coding, whiteboard, preparation

---

## 📋 Ringkasan

Wawancara teknis biasanya terdiri dari: tanya jawab konsep, studi kasus, dan live coding. Persiapan yang baik meningkatkan kepercayaan diri.

**Target pemahaman:**
- Kamu paham format interview teknis
- Kamu siap jawab pertanyaan umum
- Kamu bisa presentasikan project
- Kamu paham tips live coding

---

## 1. Format Interview Teknis

| Tahap | Durasi | Isi |
|-------|--------|-----|
| **HR Screening** | 15-30 menit | Pengalaman, motivasi, ekspektasi gaji |
| **Technical Interview** | 45-90 menit | Konsep, studi kasus, kode |
| **Live Coding** | 30-60 menit | Nulis kode langsung |
| **User / Final Interview** | 30-60 menit | Culture fit, visi |

---

## 2. Pertanyaan Umum

### 2.1 Konsep Laravel

```text
- Jelaskan alur request Laravel dari awal sampai response.
- Apa perbedaan middleware dan policy?
- Kapan pakai service layer?
- Apa itu Eloquent ORM? Keuntungannya?
- Apa itu N+1 problem? Gimana cara fix?
- Perbedaan eager loading vs lazy loading?
- Bagaimana cara kerja Blade?
```

### 2.2 OOP & Design Pattern

```text
- Jelaskan MVC.
- Apa itu Dependency Injection?
- Jelaskan SOLID (minimal 3 prinsip).
- Perbedaan interface vs abstract class?
- Apa itu Service Container?
```

### 2.3 Database

```text
- Perbedaan LEFT JOIN vs INNER JOIN?
- Apa itu indexing?
- Jelaskan migration.
```

---

## 3. Presentasi Project

Siapkan jawaban untuk:

```text
Q: "Ceritakan project yang paling membanggakan."
A: "Project e-commerce Laravel. Saya mendesain arsitektur
    MVC + Service Layer. Fitur unggulan: checkout flow
    dengan Midtrans payment. Tantangan: handling N+1
    query. Solusi: eager loading. Hasil: response time
    turun dari 2s ke 200ms."

Q: "Apa tantangan terbesar?"
A: "Integrasi Midtrans. Documentasi kurang jelas.
    Selesaikan dengan baca source code library dan testing."

Q: "Apa yang akan kamu tingkatkan?"
A: "Tambahkan queue untuk email notification,
    dan caching untuk product listing."
```

---

## 4. Live Coding Tips

```text
✅ Pahami soal dulu — tanya kalau kurang jelas
✅ Tulis pseudocode dulu baru kode
✅ Verbal thinking — bicarakan proses berpikir
✅ Handle edge cases (null, empty, invalid)
✅ Test dengan contoh

❌ Jangan diam lama tanpa bicara
❌ Jangan panik kalau error — debug sambil bicarakan
❌ Jangan gunakan shortcut/fitur yang tidak kamu pahami
```

### 4.1 Contoh Live Coding

```php
// Soal: Buat function untuk filter produk aktif dalam range harga

// 1. Pahami soal
// 2. Tulis function signature
public function getActiveProductsInRange(
    int $minPrice,
    int $maxPrice
): Collection {
    // 3. Tulis logika
    return Product::where('is_active', true)
        ->whereBetween('price', [$minPrice, $maxPrice])
        ->get();
}
// 4. Test
// getActiveProductsInRange(10000, 50000)
```

---

## 5. Persiapan

```text
Seminggu Sebelum Interview:
□ Review seluruh journey FASE 1-13
□ Latihan presentasi project (5 menit, 10 menit)
□ Latihan live coding (PHP, Laravel)
□ Siapkan 3 pertanyaan untuk interviewer

Sehari Sebelum:
□ Cek koneksi internet / venue
□ Siapkan contoh kode yang rapi
□ Istirahat cukup

Saat Interview:
□ Bicara jelas, tidak terburu-buru
□ Kalau tidak tahu → "Saya belum pernah, tapi saya akan..."
□ Akhiri dengan tanya tentang tim dan tech stack
```

---

## 🧪 Latihan

1. **Simulasi.** Minta teman jadi interviewer. Jawab pertanyaan MVC, Service Layer, N+1.

2. **Latihan live coding.** Buka text editor kosong. Buat function PHP dalam 15 menit: filter, sorting, array manipulation.

3. **Review konsep.** Buka ulang FASE 3 (OOP), FASE 6 (Laravel), FASE 8 (Design Pattern). Hafalkan konsep kunci.

4. **Siapkan 3 pertanyaan.** Tulis 3 pertanyaan yang akan kamu tanyakan ke interviewer.

---

## 🔗 Referensi

- [Laravel Interview Questions](https://www.laravelinterviewquestions.com/)
- [PHP Interview Questions](https://www.toptal.com/php/interview-questions)
- [Coding Interview University](https://github.com/jwasham/coding-interview-university)
