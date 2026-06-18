# 03-07: OOP di Codebase Ini — Tur Lengkap

> **Fase**: 3 — Object Oriented Programming  
> **Prasyarat**: 03-06-oop-di-php  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: codebase tour, arsitektur, service pattern, controller, model, provider, middleware, policy, observer, notification

---

## 📋 Ringkasan

Dokumen ini adalah **tur OOP** dari seluruh codebase `olshop-koneksi`. Kita akan melihat bagaimana semua konsep OOP dari dokumen sebelumnya diterapkan di kode nyata.

**Target pemahaman:**
- Kamu bisa membuka file mana pun dan mengidentifikasi pola OOP-nya
- Kamu paham arsitektur folder dan tanggung jawab masing-masing
- Kamu bisa melihat bagaimana class saling berhubungan

---

## 1. Arsitektur Folder — Peta Aplikasi

```
app/
├── Console/
│   └── Kernel.php
│
├── Exceptions/
│   └── Handler.php
│
├── Http/
│   ├── Controllers/
│   │   ├── Controller.php            (abstract class)
│   │   ├── Admin/                    (7 controllers)
│   │   ├── Auth/                     (6 controllers)
│   │   └── 7 controllers lain
│   ├── Middleware/
│   │   └── RoleMiddleware.php
│   └── Requests/
│       └── Auth/
│           └── LoginRequest.php
│
├── Models/
│   ├── User.php
│   ├── Product.php
│   ├── Order.php
│   ├── OrderItem.php
│   ├── Payment.php
│   ├── Category.php
│   ├── Brand.php
│   ├── Address.php
│   ├── Setting.php
│   ├── Coupon.php
│   ├── Shipment.php
│   ├── InventoryMovement.php
│   ├── ProductImage.php
│   ├── Role.php
│   └── CouponUsage.php
│
├── Notifications/
│   └── OrderStatusChanged.php
│
├── Observers/
│   └── OrderObserver.php
│
├── Policies/
│   ├── OrderPolicy.php
│   └── AddressPolicy.php
│
├── Providers/
│   ├── AppServiceProvider.php
│   └── TelescopeServiceProvider.php
│
└── Services/
    ├── CartService.php
    ├── OrderService.php
    ├── PaymentService.php
    ├── ProductService.php
    ├── BrandService.php
    ├── CategoryService.php
    ├── AddressService.php
    ├── RajaOngkirService.php
    └── MidtransService.php

routes/
├── web.php              (semua route HTTP)
├── auth.php             (route auth)
└── console.php          (route Artisan)

resources/views/          (Blade template)
bootstrap/app.php        (konfigurasi middleware dll)
```

---

## 2. Inheritance Chain

### 2.1 Model Inheritance

```
Illuminate\Database\Eloquent\Model
├── App\Models\Product      → implements HasMedia
├── App\Models\Order
├── App\Models\OrderItem
├── App\Models\Payment
├── App\Models\Category
├── App\Models\Brand
├── App\Models\Address
├── App\Models\Setting
├── App\Models\Coupon
├── App\Models\CouponUsage
├── App\Models\Shipment
├── App\Models\OrderStatusHistory
├── App\Models\InventoryMovement
├── App\Models\ProductImage
└── App\Models\Role

Illuminate\Foundation\Auth\User (extends Model)
└── App\Models\User
```

**Yang diwarisi dari Model:**
```php
$model->save();              // Simpan ke DB
$model->update([...]);       // Update record
$model->delete();            // Hapus
Model::find($id);            // Cari by primary key
Model::where(...)->get();    // Query builder
Model::with('relation');     // Eager loading
$model->relation;            // Relasi
$model->toArray();           // Serialisasi
```

### 2.2 Controller Inheritance

```php
abstract class Controller            // app/Http/Controllers/Controller.php
│
├── class CheckoutController
├── class ShopController
├── class CartController
├── class OrderController
├── class PaymentController
├── class AddressController
├── class RegionController
├── class HomeController
├── class ProfileController
├── class MidtransWebhookController
├── class NotificationController
│
├── class Admin\DashboardController
├── class Admin\ProductController
├── class Admin\CategoryController
├── class Admin\BrandController
├── class Admin\OrderController
├── class Admin\PaymentController
└── class Admin\CustomerController
```

Semua controller adalah `final` secara implisit — tidak ada yang extends controller lain.

---

## 3. Trait Usage

| File | Trait | Fungsi |
|------|-------|--------|
| `Product.php` | `SoftDeletes` | Hapus lembut (set `deleted_at`) |
| `Product.php` | `InteractsWithMedia` | Upload/manage media (Spatie) |
| `User.php` | `HasFactory` | Factory untuk testing |
| `User.php` | `Notifiable` | Method `notify()` |
| `Order.php` | `HasFactory` | Factory untuk testing |
| `OrderItem.php` | `HasFactory` | Factory untuk testing |
| `Payment.php` | `HasFactory` | Factory untuk testing |
| `OrderStatusChanged.php` | `Queueable` | Kirim notifikasi via queue |

---

## 4. Interface

**Satu-satunya interface explicit:**

```php
// Product.php
class Product extends Model implements HasMedia
{
    use InteractsWithMedia; // ← implementasi dari HasMedia
}
```

**Interface implisit (Laravel auto-resolve):**

```php
// Policies — Laravel mencari policy berdasarkan nama model
// Tidak perlu implements, Laravel pakai convention
class OrderPolicy   { public function view(User $user, Order $order): bool { ... } }
class AddressPolicy { public function view(User $user, Address $address): bool { ... } }

// FormRequest — validasi otomatis
class LoginRequest extends FormRequest { ... }
```

---

## 5. Service Pattern

### 5.1 Semua Service Class

```
app/Services/
├── CartService        ← Cart di session
├── OrderService       ← Order + transaction
├── PaymentService     ← Payment confirmation
├── ProductService     ← Product CRUD
├── BrandService       ← Brand CRUD
├── CategoryService    ← Category CRUD
├── AddressService     ← Address management
├── RajaOngkirService  ← Shipping API
└── MidtransService    ← Payment gateway
```

### 5.2 Pola yang Konsisten

Setiap service mengikuti pola yang sama:

```php
<?php

namespace App\Services;

class CartService
{
    // 1. Properti
    protected $sessionKey = 'shopping_cart';

    // 2. Constructor (opsional — jika butuh DI)
    // public function __construct(...) {}

    // 3. Public methods (business logic API)
    public function getCart(): array { /* ... */ }
    public function add(int $productId, int $quantity = 1): array { /* ... */ }
    public function remove(int $productId): array { /* ... */ }
    public function clear(): void { /* ... */ }
    public function getTotal(): int { /* ... */ }

    // 4. Private methods (detail internal)
    // private function validateStock(...) { }
}
```

### 5.3 Service → Service DI

```php
// OrderService.php
class OrderService
{
    public function __construct(CartService $cartService)
    {
        $this->cartService = $cartService;
    }

    public function placeOrder(array $data)
    {
        $cart = $this->cartService->getCart();
        // ...
    }
}
```

---

## 6. Dependency Injection Map

```
CheckoutController
  ├── CartService
  ├── AddressService
  ├── OrderService
  │     └── CartService  ← DI lagi
  └── RajaOngkirService

CartController
  └── CartService

AddressController
  └── AddressService

RegionController
  └── RajaOngkirService

PaymentController
  ├── MidtransService
  └── app(PaymentService)  → PaymentService

Admin\ProductController
  ├── ProductService
  ├── CategoryService
  └── BrandService

Admin\BrandController
  └── BrandService

Admin\CategoryController
  └── CategoryService

Admin\PaymentController
  └── PaymentService

OrderService
  └── CartService
```

---

## 7. Polymorphism in Action

### 7.1 Eloquent Query Builder

Semua model dipanggil dengan cara yang sama — hasilnya query ke tabel berbeda:

```php
Product::find(1);       // SELECT * FROM products WHERE id = 1
Order::find(1);         // SELECT * FROM orders WHERE id = 1
User::find(1);          // SELECT * FROM users WHERE id = 1
Setting::where('key', 'x')->first();  // SELECT * FROM settings WHERE key = 'x'
```

### 7.2 Route Model Binding

```php
// Route: /orders/{order}
public function show(Order $order)
// Laravel lihat {order} → cari Order::find($id) → inject ke parameter
```

### 7.3 Authorization (Policy)

```php
$this->authorize('view', $order);     // → OrderPolicy@view
$this->authorize('view', $address);   // → AddressPolicy@view
```

Method `authorize()` tidak peduli model apa — polymorphism menentukan policy mana yang dipanggil.

### 7.4 Polimorphic Relationship

```php
// InventoryMovement.php
public function reference()
{
    return $this->morphTo('reference');
}
```

`$movement->reference` bisa mengembalikan `Order`, `OrderItem`, atau model lain — tergantung nilai `reference_type` di database.

---

## 8. Encapsulation Map

### 8.1 Protected Properties

Semua service dan controller menggunakan `protected` untuk properti:

```
CheckoutController:
  protected $cartService;
  protected $addressService;
  protected $orderService;
  protected $rajaOngkirService;

CartService:
  protected $sessionKey = 'shopping_cart';

OrderService:
  protected $cartService;
```

### 8.2 Private Methods

Hanya di `RajaOngkirService`:

```
RajaOngkirService:
  private function getDummyProvinces(): array
  private function getDummyCities($provinceId = null): array
  private function getDummyCost($courier): array
```

### 8.3 Accessors (Getters)

```
Order:
  getStatusLabelAttribute(): string
  getStatusColorAttribute(): string

OrderItem:
  getSubtotalAttribute(): float
  getDisplayNameAttribute(): string
```

---

## 9. PHP 8 Features Used

| Feature | Where | Line |
|---------|-------|------|
| **Match expression** | `Order.php` | 61-68 |
| | `Setting.php` | 28-33 |
| | `ShopController.php` | 49-54 |
| **Named arguments** | `RegisteredUserController.php` | 56 |
| | `AuthenticatedSessionController.php` | 31 |
| | `EmailVerificationPromptController.php` | 18 |
| **Attributes** | `Product.php` | 12-18 |
| | `User.php` | 13-14 |
| **Union type `mixed`** | `Setting.php` | 23 |
| **Nullsafe** | `OrderItem.php` | 42 |
| **`str_contains`** | Not yet in app code | — |
| **Constructor promotion** | Not yet in app code | — |
| **Enum** | Not yet in app code | — |

---

## 10. Class Diagram — Order Creation Flow

```
User (Browser)
  │ POST /checkout/process
  ▼
Route: web.php
  │ [CheckoutController@process]
  ▼
CheckoutController
  │ $request->validate([...])
  │ $cart = $this->cartService->getCart()
  │ $order = $this->orderService->placeOrder([...])
  ▼
OrderService
  │ DB::transaction(function () {
  │     foreach ($cart as $item) → validasi stok
  │     Order::create([...])
  │     Shipment::create([...])
  │     foreach ($cart as $item) → OrderItem::create([...])
  │     $this->cartService->clear()
  │ })
  ▼
  return $order
  ▼
CheckoutController
  │ redirect()->route('payments.confirmation', $order->id)
  ▼
Browser → redirect ke halaman pembayaran

OOP Patterns in this flow:
  - Inheritance:   Order extends Model, OrderItem extends Model
  - Encapsulation: CartService detail internal tersembunyi
  - Polymorphism:  Order::create() → INSERT ke orders
  - DI:            CheckoutController menerima service via constructor
  - Composition:   OrderService has-a CartService
  - Trait:         Order uses HasFactory
```

---

## 11. Ringkasan: OOP Pattern per Folder

| Folder | OOP Pattern | Contoh |
|--------|-------------|--------|
| `Models/` | Inheritance, Accessor, Trait | `Product extends Model implements HasMedia` |
| `Controllers/` | Inheritance, DI | `CheckoutController extends Controller` |
| `Services/` | Composition, Encapsulation | `OrderService has-a CartService` |
| `Policies/` | Polymorphism, Authorization | `OrderPolicy`, `AddressPolicy` |
| `Middleware/` | Variadic, DI | `RoleMiddleware` with `...$roles` |
| `Providers/` | Service Container, Binding | `AppServiceProvider` |
| `Notifications/` | Inheritance, Queueable | `OrderStatusChanged extends Notification` |
| `Observers/` | Event-driven OOP | `OrderObserver` |

---

## 🧪 Latihan Akhir Fase 3

1. **Buka setiap file di `app/Services/`.** Untuk setiap service, catat:
   - Properti (visibility, type)
   - Constructor (parameter, DI)
   - Public methods
   - Private methods

2. **Buka `app/Models/`.** Identifikasi untuk setiap model:
   - Parent class
   - Interface
   - Traits
   - Relationships (belongsTo, hasMany, etc.)

3. **Trace dependency chain.** Mulai dari `CheckoutController`, trace semua dependency sampai ke ujung. Service apa saja yang terlibat?

4. **Buat class diagram sederhana** untuk `Order`, `OrderItem`, `Payment`, `Shipment`. Gambar relasi antar class.

5. **Identify patterns.** Cari contoh berikut di codebase:
   - Sebuah class yang `extends` class lain
   - Sebuah class yang `implements` interface
   - Sebuah class yang `use` trait
   - Sebuah method `private`
   - Sebuah method `protected`
   - Sebuah method `static`
   - Sebuah constructor dengan DI
   - Sebuah accessor (`getXAttribute`)

6. **Buat class sendiri** yang menggunakan semua konsep OOP: inheritance, interface, trait, DI, encapsulation.

---

## 🔗 Referensi

- Semua file di `app/` — tur langsung
- [Laravel Docs: Architecture](https://laravel.com/docs/11.x/architecture)
- [Laravel Docs: Eloquent](https://laravel.com/docs/11.x/eloquent)
- [Laravel Docs: Service Container](https://laravel.com/docs/11.x/container)
