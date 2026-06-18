# 13-05: Dokumentasi Kode

> **Fase**: 13 — Code Quality  
> **Prasyarat**: 13-04-refactoring-dasar  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: dokumentasi, PHPDoc, comment, type hint, self-documenting, DocBlock

---

## 📋 Ringkasan

Dokumentasi kode yang baik membantu developer memahami **kenapa** kode ditulis, bukan hanya **apa** yang dilakukan. Kode yang bersih seharusnya sudah menjelaskan "apa" — dokumentasi menjelaskan "kenapa".

**Target pemahaman:**
- Kamu paham kapan perlu komentar
- Kamu bisa menulis PHPDoc yang baik
- Kamu bedain komentar baik vs buruk
- Kamu paham self-documenting code

---

## 1. Self-Documenting Code

**Kode yang baik = dokumentasi itu sendiri:**

```php
// ❌ Komentar tidak perlu — kode sudah jelas
// Set price dari request
$product->price = $request->price;

// ✅ Tidak perlu komentar — kode sudah self-documenting
$product->price = $request->validated()['price'];
```

**Ciri self-documenting:**
- Nama variable jelas
- Method melakukan satu hal
- Nama method deskriptif
- Tidak ada magic number

---

## 2. Kapan Perlu Komentar?

```php
// ✅ Komentar untuk "kenapa" (bukan "apa")
// Radius 5km karena kurir hanya melayani area tertentu
if ($distance > 5) {
    throw new \Exception('Luar area pengiriman');
}

// ✅ Komentar untuk business rule
// Diskon 10% hanya untuk order pertama (aturan dari owner)
if ($user->orders()->count() === 0) {
    $total *= 0.9;
}

// ✅ Komentar untuk workaround / tech debt
// TODO: Refactor after API v2 release
// Midtrans API kadang return null untuk transaction_time
if ($response->transaction_time === null) {
    $response->transaction_time = now();
}
```

---

## 3. PHPDoc

### 3.1 Method

```php
/**
 * Menambahkan produk ke cart.
 *
 * @param int $productId ID produk yang akan ditambahkan
 * @param int $quantity Jumlah (min 1)
 *
 * @throws \InvalidArgumentException Jika quantity < 1
 *
 * @return void
 */
public function addItem(int $productId, int $quantity): void
{
    // ...
}
```

### 3.2 Property

```php
class Order extends Model
{
    /**
     * Status order.
     * 
     * @var string (pending|processing|shipped|delivered|cancelled)
     */
    public string $status;
}
```

### 3.3 Type Hints > PHPDoc

**Modern PHP:** type hints sudah mengurangi kebutuhan PHPDoc:

```php
// ✅ Type hints sudah cukup
public function addItem(int $productId, int $quantity): void
{
    // PHP akan enforce type — tidak perlu @param
}
}

// PHPDoc hanya untuk:
// 1. @throws — exception yang dilempar
// 2. @return — kalau return type kompleks (Collection format tertentu)
// 3. Deskripsi — kalau method tidak obvious
```

---

## 4. DocBlock vs No DocBlock

| Situasi | PHPDoc | Contoh |
|---------|--------|--------|
| Method sederhana | ❌ Tidak perlu | `getTotal(): int` |
| Method kompleks | ✅ Perlu | Proses yang mempengaruhi banyak state |
| Method dengan exception | ✅ @throws | `@throws PaymentFailedException` |
| Interface / contract | ✅ Perlu | Biar implementor paham |
| Property dengan format khusus | ✅ Perlu | Format `Y-m-d`, status values |

---

## 5. Dokumentasi Kode Buruk

```php
// ❌ Komentar tidak berguna
class Product extends Model
{
    // This function gets the product name
    public function getName()  // ← jelas! tidak perlu komentar
    {
        return $this->name;
    }
}

// ❌ Komentar yang basi (tidak diupdate)
// 2023-01-01: Fix bug #123 — jangan dihapus!
$product->update($request->all());
// → Komentar ini sudah tidak relevan

// ❌ Comment-out code
// $oldLogic = Product::where('status', 'old')->get();
// $oldLogic->each->delete();
// → Hapus saja! Git menyimpan history
```

---

## 6. Dokumentasi di Codebase

```bash
# Generate IDE helper (otomatis PHPDoc untuk Facades, Models)
php artisan ide-helper:generate       # Facades
php artisan ide-helper:models         # Models (PHPDoc untuk Eloquent)
```

**Fungsi:** Membuat PHPDoc otomatis untuk Eloquent — jadi IDE tahu method apa yang tersedia di Model.

---

## 🧪 Latihan

1. **Cari komentar.** Cari komentar di codebase. Mana yang berguna? Mana yang tidak?

2. **Self-documenting.** Cari kode yang butuh komentar. Refactor jadi self-documenting (rename variable, extract method).

3. **PHPDoc.** Buka `app/Services/CartService.php`. Apakah method-methodnya punya PHPDoc? Apakah perlu?

4. **IDE Helper.** Jalankan `php artisan ide-helper:models` (kalau package terinstall). Cek perubahan di `_ide_helper.php`.

5. **Evaluasi.** Dalam 1 kalimat: kapan kamu perlu menulis komentar?

---

## 🔗 Referensi

- [PHPDoc](https://docs.phpdoc.org/)
- [Laravel IDE Helper](https://github.com/barryvdh/laravel-ide-helper)
- [Self-Documenting Code](https://www.geeksforgeeks.org/self-documenting-code/)
