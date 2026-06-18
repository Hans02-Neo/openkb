# 03-04: Encapsulation dan Abstraction

> **Fase**: 3 — Object Oriented Programming  
> **Prasyarat**: 03-03-inheritance-polymorphism  
> **Waktu baca**: 70-85 menit  
> **Kata kunci**: encapsulation, information hiding, getter, setter, accessor, mutator, abstraction, interface, contract, service layer

---

## 📋 Ringkasan

Dua pilar OOP yang membuat kode **aman** dan **mudah digunakan**: Encapsulation dan Abstraction.

- **Encapsulation:** Sembunyikan detail internal, hanya buka yang perlu
- **Abstraction:** Sederhanakan kompleksitas, pengguna cukup tahu "apa" bukan "bagaimana"

**Target pemahaman:**
- Kamu paham `private`/`protected` dan kapan menggunakannya
- Kamu bisa membuat getter/setter/accessor
- Kamu paham abstraction via interface dan service layer
- Kamu bisa membaca encapsulation/abstraction pattern di codebase

---

## 1. Encapsulation — Bungkus dan Sembunyikan

### 1.1 Definisi

Encapsulation adalah **menyembunyikan detail internal** sebuah objek dari dunia luar. Dunia luar hanya bisa berinteraksi melalui **method publik** yang disediakan.

```
┌─────────────────────────────────┐
│          Objek: Mesin Kopi       │
├─────────────────────────────────┤
│  PRIVATE (sembunyi):            │
│  - suhuAir: int                 │
│  - tekananUap: float            │
│  - heatElement()                │
│  - pumpWater()                  │
├─────────────────────────────────┤
│  PUBLIC (buka):                 │
│  - buatKopi(): void             │ ← satu-satunya cara pakai
└─────────────────────────────────┘

Kamu tidak perlu tahu cara kerja heat element atau pompa.
Kamu cukup pencet tombol "buat kopi".
```

### 1.2 Encapsulation Tanpa Encapsulation (Buruk)

```php
<?php

class CartService
{
    public $sessionKey = 'shopping_cart'; // public — siapa pun bisa ubah
    public $cart = [];                     // public — siapa pun bisa langsung modifikasi
}

// Masalah:
$cartService->sessionKey = 'another_cart'; // ubah key sembarangan
$cartService->cart = [];                    // hapus cart tanpa lewat logic
```

**Masalah:** Tidak ada kontrol. Data bisa diubah seenaknya, melompati validasi.

### 1.3 Encapsulation dengan Private/Protected

```php
<?php

class CartService
{
    protected string $sessionKey = 'shopping_cart'; // protected — tidak bisa diubah dari luar

    public function getCart(): array
    {
        return Session::get($this->sessionKey, []);
    }

    public function add(int $productId, int $quantity = 1): array
    {
        $cart = $this->getCart();
        // Validasi dan logika bisnis di sini...
        Session::put($this->sessionKey, $cart);
        return $cart;
    }

    public function clear(): void
    {
        Session::forget($this->sessionKey);
    }
}
```

**Sekarang:** Semua modifikasi cart harus lewat method `add()`, `remove()`, `clear()`. Tidak bisa langsung `$cartService->cart = [...]`.

### 1.4 Encapsulation di Codebase

**RajaOngkirService.php — private method untuk detail internal:**

```php
class RajaOngkirService
{
    // PUBLIC — API untuk controller
    public function getProvinces(): array
    {
        // Kalau gagal, panggil dummy
        return $this->getDummyProvinces();
    }

    // PRIVATE — detail internal, tidak untuk dipanggil dari luar
    private function getDummyProvinces(): array
    {
        return [
            ['province_id' => '5', 'province' => 'DI Yogyakarta'],
            // ...
        ];
    }

    private function getDummyCities($provinceId = null): array { /* ... */ }
    private function getDummyCost($courier): array { /* ... */ }
}
```

**Kenapa private?** `getDummyProvinces()` adalah **fallback** untuk development. Controller tidak perlu tahu ada dummy data atau tidak. Yang penting controller panggil `getProvinces()` — dapat array provinsi.

**CartService.php — protected property:**

```php
class CartService
{
    protected $sessionKey = 'shopping_cart';
    // Kalau diubah jadi public:
    //   $cartService->sessionKey = 'hacked'; — bisa kacau
    // Dengan protected:
    //   Hanya class ini dan subclass yang bisa ubah
}
```

### 1.5 Getter dan Setter

**Pattern klasik encapsulation:**

```php
<?php

class Product
{
    private float $price;

    // Getter — baca data dengan aman
    public function getPrice(): float
    {
        return $this->price;
    }

    // Setter — ubah data dengan validasi
    public function setPrice(float $price): void
    {
        if ($price <= 0) {
            throw new \InvalidArgumentException('Harga harus positif');
        }
        $this->price = $price;
    }
}
```

### 1.6 Accessor — Getter ala Laravel (Eloquent)

Laravel punya cara sendiri: **accessor** (getter otomatis).

**OrderItem.php:35-38:**
```php
public function getSubtotalAttribute(): float
{
    return $this->price * $this->quantity;
}
```

Diakses sebagai properti:
```php
echo $orderItem->subtotal; // bukan $orderItem->getSubtotalAttribute()
```

Laravel memanggil method `getSubtotalAttribute()` saat kamu mengakses `->subtotal`. Ini encapsulation yang elegan — logika komputasi tersembunyi di balik properti.

**Order.php:59-68 — accessor untuk status label:**
```php
public function getStatusLabelAttribute(): string
{
    return match ($this->status) {
        'pending'    => 'Menunggu Pembayaran',
        'processing' => 'Diproses',
        'shipped'    => 'Dikirim',
        'completed'  => 'Selesai',
        'cancelled'  => 'Dibatalkan',
        default      => ucfirst($this->status),
    };
}
```

Dipanggil:
```php
$order->status_label; // "Menunggu Pembayaran" — bukan status mentah "pending"
```

### 1.7 Mutator — Setter ala Laravel

Kebalikan accessor — dipanggil saat **menulis** properti.

```php
<?php

class User extends Authenticatable
{
    public function setPasswordAttribute(string $value): void
    {
        $this->attributes['password'] = bcrypt($value);
    }
}

// Saat赋值:
$user->password = 'secret123'; // otomatis di-hash sebelum disimpan
```

---

## 2. Encapsulation vs Performance

Encapsulation kadang dianggap "tambah ribet". Kenapa perlu getter/setter kalau bisa langsung `$obj->prop`?

```php
<?php

// Tanpa encapsulation:
function calculateTotal(Order $order): float
{
    return $order->subtotal + $order->shippingCost - $order->discount;
    // Siapa pun bisa ubah subtotal seenaknya
}

// Dengan encapsulation:
function calculateTotal(Order $order): float
{
    return $order->total; // computed via accessor, data internal aman
}
```

**Manfaat encapsulation di tim besar:**
1. API stabil — method `getTotal()` tidak akan berubah walau implementasi diubah
2. Validasi terpusat — semua perubahan lewat method yang sama
3. Debugging lebih mudah — tinggal cari method yang mengubah data

---

## 3. Abstraction — Sederhanakan Kompleksitas

### 3.1 Definisi

Abstraction adalah **menyembunyikan detail implementasi** dan hanya menampilkan **fungsi esensial**. Pengguna tidak perlu tahu **bagaimana** sesuatu bekerja, cukup **apa** yang bisa dilakukan.

```
┌─────────────────────────────────┐
│      Remote TV (Abstraction)     │
├─────────────────────────────────┤
│  TOMBOL:                        │
│  - power()    → ON/OFF          │
│  - volumeUp() → naikkan suara   │
│  - channelNext() → ganti ch.    │
├─────────────────────────────────┤
│  (DETAIL TERSEMBUNYI):          │
│  - sinyal infrared              │
│  - protokol komunikasi          │
│  - encoding remote code         │
└─────────────────────────────────┘
```

### 3.2 Tanpa Abstraction

```php
<?php
// Di controller — langsung panggil Midtrans API
$serverKey = config('services.midtrans.server_key');
$params = [...];
$snapToken = Snap::getSnapToken($params); // detail Midtrans bocor ke controller
$payment = Payment::create([...]);
```

**Masalah:** Controller tahu terlalu banyak tentang Midtrans. Kalau ganti payment gateway, semua controller harus diubah.

### 3.3 Dengan Abstraction (Service Layer)

**MidtransService.php:**
```php
class MidtransService
{
    public function getSnapToken(Order $order): string
    {
        // Semua detail Midtrans di sini
        $midtransOrderId = $order->order_number . '-' . time();
        $params = [
            'transaction_details' => [
                'order_id' => $midtransOrderId,
                'gross_amount' => (int) $order->total_amount,
            ],
            'customer_details' => [
                'first_name' => $order->user->name,
                'email' => $order->user->email,
            ],
        ];

        $itemDetails = [];
        foreach ($order->items as $item) {
            $itemDetails[] = [
                'id' => $item->product_id,
                'price' => (int) $item->price,
                'quantity' => $item->quantity,
                'name' => substr($item->product_name, 0, 50),
            ];
        }
        $params['item_details'] = $itemDetails;

        $snapToken = Snap::getSnapToken($params);
        // Simpan ke database...
        return $snapToken;
    }
}
```

**Controller hanya perlu:**
```php
$snapToken = $midtransService->getSnapToken($order);
// Selesai — tidak perlu tahu Midtrans API
```

**Ini abstraction:** Controller tidak tahu detail Midtrans. Service menyembunyikan kompleksitas.

### 3.4 Abstraction via Interface

Interface adalah **abstraction murni** — hanya mendefinisikan **apa** yang bisa dilakukan, bukan **bagaimana**.

```php
<?php

interface PaymentGateway
{
    public function charge(Order $order): array;
    public function refund(Order $order): array;
}
```

Implementasi berbeda:
```php
class MidtransGateway implements PaymentGateway
{
    public function charge(Order $order): array
    {
        // Panggil API Midtrans
    }

    public function refund(Order $order): array
    {
        // Panggil API refund Midtrans
    }
}

class ManualTransferGateway implements PaymentGateway
{
    public function charge(Order $order): array
    {
        // Simpan instruksi transfer, tidak ada API
    }

    public function refund(Order $order): array
    {
        // Catat refund manual
    }
}
```

Pengguna (controller) hanya peduli `PaymentGateway` interface:

```php
class PaymentController extends Controller
{
    public function __construct(
        private PaymentGateway $gateway // ← abstraction!
    ) {}

    public function charge(Order $order)
    {
        $result = $this->gateway->charge($order);
        // Tidak peduli gateway apa — interface saja yang penting
    }
}
```

### 3.5 Interface vs Abstract Class untuk Abstraction

| | Interface | Abstract Class |
|---|-----------|---------------|
| Level abstraction | **Paling abstrak** — kontrak murni | Sebagian abstrak, sebagian konkret |
| Method | Semua abstract (tanpa body) | Bisa campur abstract + konkret |
| Properti | Tidak bisa | Bisa |
| Penggunaan | "Apa yang bisa dilakukan" | "Apa yang bisa dilakukan + default behavior" |

**Contoh abstract class di codebase:**
```php
abstract class Controller
{
    // Bisa nanti ditambah method konkret yang diwarisi semua controller
}
```

**Contoh interface di codebase:**
```php
class Product extends Model implements HasMedia
{
    // HasMedia adalah kontrak: "Product bisa punya media"
}
```

---

## 4. Service Layer — Abstraction Paling Penting di Codebase

### 4.1 Pattern Service Layer

Service layer adalah **lapisan abstraksi** antara controller dan logika bisnis.

```
Controller (HTTP)
    ↓ panggil method service
Service (Business Logic)
    ↓ panggil model / API
Model / API (Data)
```

**Tanpa service layer:**
```php
class OrderController extends Controller
{
    public function store(Request $request)
    {
        // VALIDASI
        // HITUNG TOTAL
        // CEK STOK
        // BUAT ORDER
        // BUAT ORDER ITEMS
        // KIRIM EMAIL
        // CLEAR CART
        // REDIRECT
        // — SEMUA di satu method — ribuan baris!
    }
}
```

**Dengan service layer (abstraction):**
```php
class CheckoutController extends Controller
{
    public function process(Request $request)
    {
        $order = $this->orderService->placeOrder($request->all());
        return redirect()->route('payments.confirmation', $order->id);
    }
}
```

Controller tidak tahu **bagaimana** order dibuat — hanya tahu **apa** yang terjadi: `placeOrder()` → dapat `$order`.

### 4.2 Service Layer di Codebase

```
app/Services/
├── CartService        → Kelola cart (session)
├── OrderService       → Buat order (DB transaction)
├── PaymentService     → Konfirmasi/reject pembayaran
├── ProductService     → CRUD produk + media
├── BrandService       → CRUD brand
├── CategoryService    → CRUD kategori
├── AddressService     → CRUD alamat user
├── RajaOngkirService  → Integrasi shipping API
└── MidtransService    → Integrasi payment gateway
```

Setiap service adalah **abstraksi** dari satu tanggung jawab bisnis.

### 4.3 Manfaat Service Layer

1. **Controller tetap ramping** — hanya urus HTTP
2. **Logika bisnis terpusat** — tidak tersebar di banyak controller
3. **Mudah ditest** — test service tanpa HTTP
4. **Mudah diganti implementasi** — ganti Midtrans ke Stripe? Cukup ubah MidtransService

---

## 5. Information Hiding — Jangan Ekspos Lebih dari Perlu

### 5.1 Prinsip

**"Kamu harus membuka method yang paling sedikit memungkinkan, tapi tidak kurang dari yang dibutuhkan."**

```php
<?php

// ❌ Terlalu terbuka:
class OrderService
{
    public function validateStock(array $cart): void { /* ... */ }
    public function calculateTotal(array $cart): float { /* ... */ }
    public function createOrderRecord(array $data): Order { /* ... */ }
    public function createOrderItems(Order $order, array $cart): void { /* ... */ }
    public function clearCart(): void { /* ... */ }
    public function sendNotification(Order $order): void { /* ... */ }
}
// → Semua public — siapa pun bisa panggil step individual, skip validasi

// ✅ Encapsulated:
class OrderService
{
    public function placeOrder(array $data): Order
    {
        return DB::transaction(function () use ($data) {
            // Semua step di sini — dijamin lengkap
        });
    }

    private function validateStock(array $cart): void { /* ... */ }
    private function calculateTotal(array $cart): float { /* ... */ }
    // private — tidak bisa dipanggil sendiri-sendiri
}
```

### 5.2 Ketika Encapsulation Dilanggar

**Di MidtransWebhookController.php — ada method protected:**

```php
protected function handleSuccessPayment(Payment $payment, Order $order)
{
    // Dipanggil dari dalam class sendiri, dan dari subclass jika ada
}
```

Ini sengaja `protected` (bukan `private`) karena suatu saat mungkin ada subclass yang perlu override. Tapi tetap tidak bisa dipanggil dari luar.

---

## 6. Encapsulation vs Abstraction — Bedanya?

Sering tertukar. Ini perbedaan sederhana:

| | Encapsulation | Abstraction |
|---|--------------|-------------|
| **Fokus** | **Sembunyikan** data internal | **Sederhanakan** kompleksitas |
| **Caranya** | Private/protected + getter/setter | Interface + method publik yang jelas |
| **Tujuan** | Mencegah penyalahgunaan data | Memudahkan penggunaan |
| **Contoh** | `private $password` | `$service->placeOrder($data)` |

**Analogi:**
- **Encapsulation:** Di dalam remote TV ada sirkuit rumit — kamu tidak bisa akses langsung
- **Abstraction:** Remote punya tombol power, volume, channel — itulah interface-nya

---

## 7. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│ ENCAPSULATION                                            │
│  "Tidak usah tahu detail dalamnya"                       │
│                                                          │
│  public function getTotal(): int {                       │
│      // Panggil method private di sini                   │
│      return $this->calculateFromSession();               │
│  }                                                       │
│                                                          │
│  private function calculateFromSession(): int {          │
│      // Detail implementasi — disembunyikan              │
│  }                                                       │
├──────────────────────────────────────────────────────────┤
│ ABSTRACTION                                              │
│  "Kamu cuma perlu tahu tombol apa yang bisa dipencet"    │
│                                                          │
│  interface PaymentGateway {                              │
│      public function charge(Order $order): array;        │
│  }                                                       │
│                                                          │
│  class Midtrans implements PaymentGateway { /* ... */ }  │
│  class ManualTransfer implements PaymentGateway { ... }  │
│                                                          │
│  Controller hanya peduli PaymentGateway, bukan           │
│  Midtrans atau ManualTransfer specific                   │
├──────────────────────────────────────────────────────────┤
│ DI CODEBASE:                                             │
│                                                          │
│  Private:  RajaOngkirService → getDummyProvinces()       │
│  Protected: CartService → $sessionKey                    │
│  Accessor: OrderItem → getSubtotalAttribute()            │
│  Service:  MidtransService → getSnapToken($order)        │
│  Interface: Product implements HasMedia                  │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Encapsulation practice.** Buat class `BankAccount` dengan:
   - `private float $balance = 0`
   - `public function deposit(float $amount): void` (validasi: amount > 0)
   - `public function withdraw(float $amount): void` (validasi: saldo cukup)
   - `public function getBalance(): float`

2. **Abstraction practice.** Buat interface `ShippingProvider` dengan method `calculateCost(int $weight, string $destination): array`. Buat dua implementasi: `JNEShipping` dan `POSShipping`.

3. **Analisis `app/Models/OrderItem.php`.**
   - Apa accessor yang tersedia?
   - Data apa yang di-encapsulate?
   - Method mana yang public, mana yang protected?

4. **Analisis `app/Services/PaymentService.php`.**
   - Method apa yang public?
   - Apa yang terjadi di `confirmPayment()`? Apa detail yang disembunyikan dari controller?
   - Kenapa `handleSuccessPayment` (di MidtransWebhookController) dibuat protected?

5. **Cari private method.** Buka `app/Services/RajaOngkirService.php`. Ada berapa private method? Kenapa mereka private?

---

## 🔗 Referensi

- [PHP Manual: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php)
- [Laravel Docs: Accessors & Mutators](https://laravel.com/docs/11.x/eloquent-mutators)
- [Laravel Docs: Service Container](https://laravel.com/docs/11.x/container)
- Codebase: `app/Services/RajaOngkirService.php` — private methods
- Codebase: `app/Models/OrderItem.php` — accessor
- Codebase: `app/Services/PaymentService.php` — service layer abstraction
- Codebase: `app/Http/Controllers/CheckoutController.php` — controller menggunakan abstraction
