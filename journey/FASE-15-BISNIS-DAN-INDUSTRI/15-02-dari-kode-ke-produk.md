# 15-02: Dari Kode ke Produk

> **Fase**: 15 — Bisnis & Industri  
> **Prasyarat**: 15-01-web-industri-ekosistem  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: produk, requirement, feature, user story, development cycle, launch

---

## 📋 Ringkasan

Kode yang kamu tulis bukan hanya kode — itu adalah **produk** yang dipakai user. Memahami siklus dari ide ke produksi membantumu menulis kode yang lebih baik.

**Target pemahaman:**
- Kamu paham siklus hidup produk
- Kamu bisa baca user story
- Kamu paham prioritasi fitur

---

## 1. Siklus Produk

```
Idea → Research → Design → Development → Testing → Deploy → Monitor
  │                                                              │
  └────────────────────── Feedback Loop ◄─────────────────────────┘
```

### 1.1 Tahapan Detail

```text
1. IDEATION
   - Ide dari client / user research / kompetitor
   - Prioritasi: "fitur apa yang paling penting?"

2. SPECIFICATION
   - PRD (Product Requirement Document)
   - User story: "Sebagai user, saya ingin..."

3. DESIGN
   - Wireframe → Mockup → Prototype
   - UI/UX designer

4. DEVELOPMENT
   - Sprint (2 minggu)
   - Backend (Laravel) + Frontend (Tailwind/Alpine)
   - Testing

5. DEPLOYMENT
   - Staging → QA → Production
   - Feature flag / gradual rollout

6. MONITOR
   - Log, error tracking, user feedback
   - Iterate!
```

---

## 2. User Story

```text
Format:
  Sebagai [role], saya ingin [fitur] sehingga [benefit].

Contoh:
  "Sebagai customer, saya ingin checkout tanpa login
   sehingga saya bisa belanja lebih cepat."

  "Sebagai admin, saya ingin melihat daftar order
   sehingga saya bisa memproses pengiriman."
```

**Dari user story → code:**

```php
// User story: "Sebagai customer, saya ingin tambah produk ke cart"
// → Route:  POST /cart/add
// → Controller: CartController@addItem
// → Service: CartService@addItem
// → View: cart badge update via Alpine.js
```

---

## 3. Prioritasi Fitur

```text
Harus ada (MVP):   Cart, checkout, payment
Seharusnya ada:    Wishlist, review produk
Nice to have:      Rekomendasi produk, chatbot
Tidak sekarang:    AI-powered search
```

**Codebase ini (olshop-koneksi):**

| Fitur | Prioritas | Status |
|-------|-----------|--------|
| Produk + kategori | MVP | ✅ |
| Cart | MVP | ✅ |
| Checkout | MVP | ✅ |
| Payment | MVP | ✅ |
| Admin order | MVP | ✅ |
| Wishlist | Seharusnya | ❌ |
| Review produk | Nice to have | ❌ |

---

## 4. Developer dalam Siklus Produk

```text
Developer tidak hanya nulis kode:
- Baca PRD / spesifikasi
- Tanya ke PM kalau requirement tidak jelas
- Estimasi effort
- code review
- Deploy
- Fix bug setelah launch
```

---

## 🧪 Latihan

1. **Tulis user story.** Tulis 3 user story untuk fitur "Wishlist" di codebase ini.

2. **Prioritasi.** Dari semua fitur codebase ini, mana yang "harus ada" vs "nice to have"?

3. **Trace siklus.** Ambil fitur checkout. Trace dari user story → route → controller → service → model → view.

---

## 🔗 Referensi

- [Atlassian: Product Discovery](https://www.atlassian.com/agile/product-discovery)
- [Roman Pichler: Product Management](https://www.romanpichler.com/)
