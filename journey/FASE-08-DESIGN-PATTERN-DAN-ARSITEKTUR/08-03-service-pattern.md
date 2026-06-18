# 08-03: Service Pattern — Memisahkan Logika Bisnis

> **Fase**: 8 — Design Pattern & Arsitektur  
> **Prasyarat**: 08-02-mvc-architecture  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: service layer, business logic, separation of concerns, thin controller, single responsibility

---

## 📋 Ringkasan

Service pattern adalah **lapisan tambahan** di atas MVC yang menampung **logika bisnis** — operasi yang tidak cocok di Model (terlalu spesifik) atau Controller (membuat gemuk).

**Target pemahaman:**
- Kamu paham kapan perlu service
- Kamu bisa merancang service yang baik
- Kamu paham perbedaan service, model, controller
- Kamu bisa membaca service di codebase

---

## 1. Masalah: Logika Bisnis di Mana?

### 1.1 Dilema

| Taruh di | Masalah |
|----------|---------|
| **Controller** | Gemuk, tidak reusable, sulit test |
| **Model** | Melanggar SRP (model urus data + logika kompleks) |
| **View** | ❌ Jangan pernah! |

### 1.2 Solusi: Service Layer

```
Controller (handle request)
    │
    ▼
Service (business logic — reusable)
    │
    ▼
Model (data / database)
```

---

## 2. Service Pattern — Definisi

### 2.1 Service = Class Biasa

Service hanyalah **class PHP biasa** (tidak extends class khusus):

```php
<?php
namespace App\Services;

class CartService
{
    // Constructor injection
    public function __construct(
        private CustomerService $customerService,
    ) {}

    // Business logic methods
    public function getCart(): array { ... }
    public function addItem(int $productId, int $qty): void { ... }
    public function getTotal(): int { ... }
}
```

### 2.2 Ciri Service yang Baik

```php
// ✅ Baik:
// - Satu tanggung jawab (CartService hanya urusan cart)
// - Method jelas (addItem, removeItem, getTotal)
// - Dependency Injection via constructor
// - Bisa di-test tanpa HTTP

class CartService
{
    public function addItem(int $productId, int $quantity): void { ... }
    public function removeItem(int $productId): void { ... }
    public function getTotal(): int { ... }
    public function clear(): void { ... }
}
```

```php
// ❌ Buruk:
// - Banyak tanggung jawab
// - Method tidak kohesif
// - Static method (susah test)

class HelperService
{
    public static function doEverything() { ... } // terlalu umum
    public static function sendEmailAndUpdateDb() { ... } // dua tanggung jawab
}
```

---

## 3. Service di Codebase

### 3.1 Daftar Service

```
app/Services/
├── CartService.php         → Operasi cart (session-based)
├── OrderService.php        → Create order, proses checkout
├── CustomerService.php     → Data pelanggan
├── RajaOngkirService.php   → API RajaOngkir
├── MidtransService.php     → Payment Midtrans
├── SettingService.php      → Pengaturan aplikasi
└── ProductService.php      → (mungkin) logika produk
```

### 3.2 Service yang Saling Bergantung

```php
// OrderService bergantung pada beberapa service
class OrderService
{
    public function __construct(
        private CartService $cartService,
        private MidtransService $midtransService,
        private RajaOngkirService $rajaOngkirService,
    ) {}
}
```

### 3.3 Service Pattern = Strategy Pattern

Service sering mengimplementasikan **Strategy Pattern** — algorithm bisa diganti:

```php
// Interface untuk payment
interface PaymentGateway
{
    public function charge(Order $order, array $data): PaymentResult;
}

// Strategy 1: Midtrans
class MidtransGateway implements PaymentGateway { ... }

// Strategy 2: Stripe
class StripeGateway implements PaymentGateway { ... }

// Strategy 3: COD
class CashOnDeliveryGateway implements PaymentGateway { ... }
```

---

## 4. Service vs Repository vs Model

| | Model | Service | Repository |
|---|-------|---------|-----------|
| **Tugas** | Data + relasi | Logika bisnis | Query abstraction |
| **Contoh** | `Product::find(1)` | `$cart->getTotal()` | `$productRepo->findById(1)` |
| **Isi** | Properties, casts, relationships | Method bisnis | Query methods |
| **Test** | Feature test | Unit test (mudah) | Unit test |

**Di codebase ini:** Langsung pakai Model (Eloquent), tanpa Repository (tidak diperlukan untuk aplikasi sederhana).

---

## 5. Service Naming Convention

```php
// ❌ Buruk — terlalu umum
class Helper
class Manager
class Utility
class Functions

// ✅ Baik — spesifik per domain
class CartService        // urusan cart
class OrderService       // urusan order
class PaymentService     // urusan payment
class ShippingService    // urusan shipping
class DiscountService    // urusan diskon
class NotificationService // urusan notifikasi
```

---

## 6. Kapan Perlu Service?

### 6.1 Perlu Service

```php
// ✅ PERLU SERVICE — logic kompleks
// 1. Melibatkan multiple model/entity
class OrderService {
    public function createOrder($data): Order {
        // Validasi stok (Product model)
        // Create order (Order model)
        // Create items (OrderItem model)
        // Kurangi stok (Product model)
        // Proses payment (Payment model/api)
        // Kirim notifikasi (Mail/Notification)
        // Clear cart (CartService)
    }
}

// 2. Integrasi eksternal
class RajaOngkirService {
    public function getShippingCost($city, $weight): int {
        // Panggil API RajaOngkir
        // Parse response
        // Return biaya
    }
}

// 3. Operasi bisnis multi-step
class CheckoutService {
    public function processCheckout($data): Order {
        // Validate → Calculate → Reserve → Pay → Confirm
    }
}
```

### 6.2 Tidak Perlu Service

```php
// ❌ TIDAK PERLU — operasi sederhana
class SimpleService {
    public function getActiveProducts() {
        return Product::where('is_active', true)->get();
    }
    // Langsung aja di controller:
    // $products = Product::active()->get();
}
```

---

## 7. Testing Service

Service mudah di-test karena dependency bisa di-mock:

```php
public function test_cart_total()
{
    $cartService = new CartService(
        $this->createMock(CustomerService::class)
    );

    // Add items via reflection atau method
    $cartService->addItem(1, 2);
    $cartService->addItem(2, 1);

    $this->assertEquals(32500, $cartService->getTotal());
}
```

---

## 8. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ SERVICE PATTERN                                                    │
├─────────────────────────────────────────────────────────────────────┤
│  Controller → Service → Model → Database                          │
│                  │                                                  │
│                  ├── CartService (cart logic)                     │
│                  ├── OrderService (order logic)                   │
│                  ├── RajaOngkirService (API call)                 │
│                  └── MidtransService (payment)                    │
├─────────────────────────────────────────────────────────────────────┤
│ KAPAN PAKAI SERVICE?                                                │
│  ✅ Logika multi-step (checkout, order)                           │
│  ✅ Integrasi API eksternal (RajaOngkir, Midtrans)                │
│  ✅ Logika yang reusable di banyak tempat                         │
│  ❌ Query sederhana (langsung di controller)                      │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  app/Services/CartService.php                                      │
│  app/Services/OrderService.php                                     │
│  app/Services/RajaOngkirService.php                                │
│  app/Services/MidtransService.php                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Baca CartService.** Buka `app/Services/CartService.php`. Method apa saja? Tanggung jawab apa yang dipegang?

2. **Baca OrderService.** Service apa yang di-inject ke OrderService? Kenapa perlu service-service itu?

3. **Rancang service.** Rancang interface dan class untuk `ShippingService` yang:
   - Bisa cek ongkos kirim
   - Bisa lacak status pengiriman
   - Support multiple kurir (JNE, TIKI, Pos)

4. **Refaktor.** Cari di controller ada logika yang seharusnya di service. Pindahkan ke service.

5. **Test service.** Tulis skenario unit test untuk `CartService::getTotal()`.

---

## 🔗 Referensi

- [Martin Fowler: Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html)
- [Laravel: Architecture Concepts](https://laravel.com/docs/11.x/architecture)
- Codebase: `app/Services/` — semua service
