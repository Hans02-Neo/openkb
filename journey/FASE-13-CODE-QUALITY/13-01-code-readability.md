# 13-01: Code Readability

> **Fase**: 13 — Code Quality  
> **Prasyarat**: FASE 6 (Laravel Deep Dive)  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: readability, clean code, naming, consistency, comments, formatting

---

## 📋 Ringkasan

Kode yang baik adalah kode yang **mudah dibaca** manusia — bukan hanya yang jalan di mesin. Readability menentukan seberapa cepat developer lain (atau kamu sendiri 6 bulan lagi) bisa memahami kode.

**Target pemahaman:**
- Kamu paham prinsip readability
- Kamu bisa menulis kode yang jelas
- Kamu bisa bedain kode bersih vs kode kotor

---

## 1. Naming Convention

### 1.1 Variable & Property

```php
// ❌ Buruk — tidak deskriptif
$q = Product::where('is_active', true);
$d = $q->get();
$t = $d->first()->price * 1.11;

// ✅ Baik — deskriptif
$activeProducts = Product::where('is_active', true);
$products = $activeProducts->get();
$priceWithTax = $products->first()->price * 1.11;
```

### 1.2 Method

```php
// ❌ Buruk — tidak jelas
public function doStuff($id) { ... }
public function handle() { ... }

// ✅ Baik — jelas apa yang dilakukan
public function getActiveProducts(): Collection { ... }
public function addItemToCart(int $productId, int $quantity): void { ... }
```

### 1.3 Boolean

```php
// ❌ Buruk
public function check($user) { ... }
if ($product->active) { ... }

// ✅ Baik — prefix is/has/can
public function isAdmin(): bool { ... }
public function hasStock(): bool { ... }
public function canCheckout(): bool { ... }
if ($product->isActive) { ... }
```

---

## 2. Function Length

```php
// ❌ Buruk — terlalu panjang (50+ baris)
public function checkout(Request $request)
{
    // validasi, buat order, kurangi stok, kirim email,
    // log activity, notifikasi admin, redirect ...
    // 80 baris!
}

// ✅ Baik — delegasi ke method/class
public function checkout(CheckoutRequest $request): RedirectResponse
{
    $order = $this->orderService->createOrder($request->validated());
    return redirect()->route('orders.show', $order);
}
```

**Aturan praktis:** Satu method = satu level abstraksi. Jika perlu scroll, terlalu panjang.

---

## 3. Early Return

```php
// ❌ Buruk — nested if
public function show(Order $order): View
{
    if (auth()->check()) {
        if ($order->user_id === auth()->id()) {
            return view('orders.show', compact('order'));
        }
    }
    abort(403);
}

// ✅ Baik — early return
public function show(Order $order): View
{
    abort_unless(auth()->check(), 403);
    abort_if($order->user_id !== auth()->id(), 403);

    return view('orders.show', compact('order'));
}
```

---

## 4. Consistent Style

```php
// ❌ Tidak konsisten
public function getData() { return Product::all(); }
public function update_order($id) { ... }
public function StoreProduct() { ... }

// ✅ Konsisten (ikuti PSR / convention codebase)
public function getData(): Collection { return Product::all(); }
public function updateOrder(int $id): void { ... }
public function storeProduct(ProductRequest $request): RedirectResponse { ... }
```

---

## 5. Readability Checklist

```
✅ Nama variable menjelaskan isi
✅ Method melakukan satu hal
✅ Method punya satu level abstraksi
✅ Tidak ada nested if > 2 level
✅ Early return daripada nested
✅ Konsisten (camelCase, type hints)
✅ Tidak ada magic number (pakai constant)
✅ Tidak ada komentar yang menjelaskan "apa" (kode sudah jelas)
```

---

## 🧪 Latihan

1. **Review file.** Buka `app/Http/Controllers/CheckoutController.php`. Evaluasi readability: 1-10.

2. **Perbaiki nama.** Cari variable dengan nama tidak deskriptif di codebase. Rename.

3. **Early return.** Cari nested if statement. Refaktor pakai early return.

4. **Bandingkan.** Buka `app/Services/CartService.php` dan `app/Http/Controllers/CartController.php`. Mana yang lebih readable? Kenapa?

---

## 🔗 Referensi

- [Clean Code: Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [PHP FIG: PSR](https://www.php-fig.org/psr/)
- Codebase: file mana pun — evaluasi readability
