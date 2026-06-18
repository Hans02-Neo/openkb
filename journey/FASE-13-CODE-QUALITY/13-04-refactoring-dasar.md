# 13-04: Refactoring Dasar

> **Fase**: 13 — Code Quality  
> **Prasyarat**: 13-03-convention-di-codebase  
> **Waktu baca**: 45-55 menit  
> **Kata kunci**: refactoring, code smell, extract method, rename, improve, technical debt

---

## 📋 Ringkasan

Refactoring adalah **mengubah struktur kode tanpa mengubah behavior**. Tujuannya: membuat kode lebih bersih, lebih mudah dipelihara, tanpa merusak fungsionalitas yang ada.

**Target pemahaman:**
- Kamu paham kapan perlu refactor
- Kamu bisa teknik refactoring dasar
- Kamu paham code smell
- Kamu refactor dengan aman (pakai test)

---

## 1. Kapan Refactor?

### 1.1 Rule of Three

```
Kali pertama: tulis kode
Kali kedua:   Copy-paste → ok
Kali ketiga:  Copy-paste lagi → REFACTOR!

→ Duplikasi = sinyal untuk refactor
```

### 1.2 Code Smell

```
🔴 SMELL → waktunya refactor:
  - Duplikasi kode
  - Method terlalu panjang (>20 baris)
  - Banyak parameter (lebih dari 3)
  - Nested if terlalu dalam (>2 level)
  - Class terlalu besar (>200 baris)
  - Nama variable tidak jelas
```

---

## 2. Teknik Refactoring

### 2.1 Extract Method

**Sebelum:**

```php
public function show(Order $order): View
{
    $this->authorize('view', $order);

    $order->load(['items.product.media', 'payment', 'shipment']);

    $subtotal = $order->items->sum(fn($item) => $item->price * $item->quantity);
    $shippingCost = $order->shipment?->cost ?? 0;
    $total = $subtotal + $shippingCost;

    return view('orders.show', compact('order', 'subtotal', 'shippingCost', 'total'));
}
```

**Sesudah:**

```php
public function show(Order $order): View
{
    $this->authorize('view', $order);

    $order->load(['items.product.media', 'payment', 'shipment']);

    return view('orders.show', [
        'order' => $order,
        'subtotal' => $this->calculateSubtotal($order),
        'shippingCost' => $this->getShippingCost($order),
        'total' => $this->calculateTotal($order),
    ]);
}

private function calculateSubtotal(Order $order): int
{
    return $order->items->sum(fn($item) => $item->price * $item->quantity);
}

private function getShippingCost(Order $order): int
{
    return $order->shipment?->cost ?? 0;
}

private function calculateTotal(Order $order): int
{
    return $this->calculateSubtotal($order) + $this->getShippingCost($order);
}
```

### 2.2 Rename Variable

**Sebelum:**

```php
$p = Product::where('is_active', true)->get();
$c = $p->count();
$t = $p->sum('price');
```

**Sesudah:**

```php
$activeProducts = Product::where('is_active', true)->get();
$productCount = $activeProducts->count();
$totalPrice = $activeProducts->sum('price');
```

### 2.3 Extract Class

**Sebelum — semua di controller:**

```php
class CheckoutController extends Controller
{
    public function process(CheckoutRequest $request): RedirectResponse
    {
        // create order
        // calculate shipping
        // process payment
        // send email
        // clear cart
    }
}
```

**Sesudah — extract ke service:**

```php
class CheckoutController extends Controller
{
    public function __construct(
        private CheckoutService $checkoutService,  // extracted
    ) {}

    public function process(CheckoutRequest $request): RedirectResponse
    {
        $order = $this->checkoutService->checkout($request->validated());
        return redirect()->route('orders.show', $order);
    }
}
```

### 2.4 Replace Magic Number

**Sebelum:**

```php
if ($order->total > 100000) {
    $discount = $order->total * 0.1;
}
```

**Sesudah:**

```php
const FREE_SHIPPING_THRESHOLD = 100000;
const DISCOUNT_PERCENTAGE = 0.1;

if ($order->total > FREE_SHIPPING_THRESHOLD) {
    $discount = $order->total * DISCOUNT_PERCENTAGE;
}
```

---

## 3. Refactoring Aman

```
1. Pastikan ada test (feature/unit)
2. Jalankan test → semua PASS
3. Refactor (ubah struktur, bukan behavior)
4. Jalankan test lagi → semua harus tetap PASS
5. Jika ada fail → batalkan refactor atau perbaiki

Golden rule: Jangan refactor tanpa test coverage!
```

---

## 4. Contoh Refactor di Codebase

### 4.1 Controller → Service

```php
// Sebelum: cart logic di controller
public function addItem(Request $request)
{
    $product = Product::findOrFail($request->product_id);
    $cart = session()->get('cart', []);
    // ... 15 baris logic cart
}

// Sesudah: panggil CartService
public function addItem(AddToCartRequest $request)
{
    $this->cartService->addItem(
        $request->product_id,
        $request->quantity
    );
    return redirect()->back()->with('success', 'Produk ditambahkan');
}
```

---

## 🧪 Latihan

1. **Identifikasi code smell.** Buka controller mana pun. Cari: method panjang, duplikasi, magic number.

2. **Extract method.** Ambil satu method panjang di controller. Extract ke method private.

3. **Rename.** Cari variable dengan nama 1-2 huruf. Rename jadi deskriptif.

4. **Extract class.** Cari controller yang masih ada logika bisnis. Pindahkan ke service.

5. **Refactor + test.** Sebelum refactor, jalankan `php artisan test`. Refactor. Jalankan test lagi. Semua tetap PASS.

---

## 🔗 Referensi

- [Refactoring Guru](https://refactoring.guru/)
- [Martin Fowler: Refactoring](https://martinfowler.com/books/refactoring.html)
- [Code Smell](https://refactoring.guru/refactoring/smells)
