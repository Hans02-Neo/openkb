# 03-03: Inheritance dan Polymorphism

> **Fase**: 3 — Object Oriented Programming  
> **Prasyarat**: 03-02-class-object-property  
> **Waktu baca**: 75-90 menit  
> **Kata kunci**: inheritance, extends, parent, override, is-a, polymorphism, subtype, interface, morphTo, Eloquent

---

## 📋 Ringkasan

Dua pilar OOP yang paling kuat: **Inheritance** (pewarisan) dan **Polymorphism** (banyak bentuk). Keduanya memungkinkan kode ditulis **sekali** dan dipakai **berulang** dengan variasi.

- **Inheritance:** Class anak mewarisi semua kemampuan class induk
- **Polymorphism:** Objek berbeda bisa dipanggil dengan cara yang sama

**Target pemahaman:**
- Kamu paham `extends`, `parent::`, dan method overriding
- Kamu bisa membaca dan memahami rantai inheritance di codebase
- Kamu paham kapan pakai inheritance vs komposisi
- Kamu paham polymorphism — satu interface, banyak implementasi

---

## 1. Inheritance — Pewarisan

### 1.1 Konsep Dasar

Inheritance memungkinkan sebuah class (child) **mewarisi** properti dan method dari class lain (parent).

```
        ┌──────────────────┐
        │   Model           │  ← Induk: ORM, query, save
        │   (Eloquent ORM)  │
        └────────┬─────────┘
                 │ extends
       ┌─────────┴──────────┐
       │                    │
┌──────▼──────┐    ┌───────▼───────┐
│   Product   │    │    Order      │
│ - category  │    │ - items()     │
│ - brand()   │    │ - payment()   │
│ - images()  │    │ - shipment()  │
└─────────────┘    └───────────────┘
```

**Apa yang diwarisi `Product` dari `Model`:**
- `save()`, `update()`, `delete()` — CRUD
- `find()`, `where()`, `orderBy()` — query builder
- `with()`, `load()` — eager loading relasi
- `$fillable`, `$casts` — konfigurasi

### 1.2 Syntax

```php
<?php

class Animal
{
    public string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function speak(): string
    {
        return "{$this->name} membuat suara";
    }
}

// Inheritance:
class Dog extends Animal
{
    // Dog mewarisi: $name, __construct(), speak()
}

class Cat extends Animal
{
    // Cat mewarisi: $name, __construct(), speak()
}

$dog = new Dog('Buddy');
$cat = new Cat('Kitty');

echo $dog->speak(); // "Buddy membuat suara"
echo $cat->speak(); // "Kitty membuat suara"
```

**Konsep "is-a":** `Dog` **is-a** `Animal`. `Product` **is-a** `Model`.

### 1.3 Inheritance di Codebase

**Semua model extends `Model`:**
```php
class Product extends Model implements HasMedia    // Product is-a Model
class Order extends Model                           // Order is-a Model
class User extends Authenticatable                  // User is-a Authenticatable (which is-a Model)
class Setting extends Model                         // Setting is-a Model
class Coupon extends Model                          // Coupon is-a Model
```

**Semua controller extends `Controller`:**
```php
abstract class Controller { /* ... */ }

class CheckoutController extends Controller  // CheckoutController is-a Controller
class ShopController extends Controller       // ShopController is-a Controller
class CartController extends Controller       // CartController is-a Controller
// ... 20+ controller lainnya
```

**Contoh rantai inheritance lebih panjang:**
```
User
  └── extends Authenticatable (Laravel)
        └── extends Model (Eloquent ORM)
              └── trait HasRelationships
              └── trait HasTimestamps
```

Ketika kamu memanggil `$user->save()`, method itu tidak ada di `User` class — tapi diwarisi dari `Model`.

---

## 2. Method Overriding

### 2.1 Menimpa Method Parent

Child class bisa **menimpa** (override) method dari parent — mengganti implementasi tanpa mengubah parent.

```php
<?php

class Animal
{
    public function speak(): string
    {
        return '...';
    }
}

class Dog extends Animal
{
    public function speak(): string  // ← override method parent
    {
        return 'Woof!';
    }
}

class Cat extends Animal
{
    public function speak(): string  // ← override method parent
    {
        return 'Meow!';
    }
}

echo (new Dog())->speak(); // "Woof!"
echo (new Cat())->speak(); // "Meow!"
```

### 2.2 Override di Codebase

**Method `casts()` di berbagai model — semuanya override method dari `Model`:**

```php
// User.php
protected function casts(): array
{
    return [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
        'is_active' => 'boolean',
    ];
}

// Product.php
protected function casts(): array
{
    return [
        'price' => 'decimal:2',
        'stock_quantity' => 'integer',
        'is_active' => 'boolean',
        'dimensions_json' => 'array',
    ];
}

// Order.php
protected $casts = [ // ← menggunakan PROPERTY override, bukan method
    'total_amount' => 'decimal:2',
    'paid_at' => 'datetime',
];
```

**Ada dua cara override di Eloquent Model:**
1. Properti (`protected $casts = [...]`) — PHP 7 style
2. Method (`protected function casts(): array`) — PHP 8+ style

Keduanya menghasilkan hal yang sama: memberitahu Eloquent cara meng-cast kolom database.

### 2.3 `parent::` — Memanggil Method Parent

Terkadang override membutuhkan logika parent, lalu ditambah.

```php
<?php

class Animal
{
    public function speak(): string
    {
        return 'Suara umum';
    }
}

class Dog extends Animal
{
    public function speak(): string
    {
        $parentSound = parent::speak(); // panggil method asli
        return $parentSound . ' yang menggonggong: Woof!';
    }
}
```

Di codebase ini tidak ada `parent::` yang eksplisit. Ini wajar — Laravel menyediakan hook method (seperti `casts()`, `register()`, `boot()`) yang dipanggil secara otomatis oleh framework. Kamu cukup override, tidak perlu memanggil parent.

### 2.4 Aturan Override — Signature Compatibility

PHP 8.1+ mewajibkan **covariant** (lebih spesifik) untuk return type dan **contravariant** (lebih umum) untuk parameter.

```php
<?php
class Parent_
{
    public function getModel(): object { /* ... */ }
}

class Child extends Parent_
{
    // ✅ BOLEH: return type lebih spesifik
    public function getModel(): Product { /* ... */ }
}

// ❌ TIDAK BOLEH: parameter type lebih spesifik
// public function getModel(Product $p): object — error!
```

---

## 3. Polymorphism — Banyak Bentuk

### 3.1 Definisi

**Polymorphism** (Yunani: poly = banyak, morph = bentuk) adalah kemampuan objek berbeda untuk merespons **method yang sama** dengan cara berbeda.

```php
<?php

// Semua class ini punya method speak()
// Tapi output-nya berbeda
$animals = [new Dog(), new Cat(), new Cow()];

foreach ($animals as $animal) {
    echo $animal->speak(); // "Woof!", "Meow!", "Moo!"
}
```

### 3.2 Polymorphism via Interface

Ini bentuk polymorphism yang paling umum di aplikasi nyata.

```php
<?php

interface PaymentGateway
{
    public function charge(float $amount): array;
}

class Midtrans implements PaymentGateway
{
    public function charge(float $amount): array
    {
        // Panggil API Midtrans
        return ['status' => 'success', 'provider' => 'midtrans'];
    }
}

class Stripe implements PaymentGateway
{
    public function charge(float $amount): array
    {
        // Panggil API Stripe
        return ['status' => 'success', 'provider' => 'stripe'];
    }
}

// Polymorphism in action:
function processPayment(PaymentGateway $gateway, float $amount): array
{
    // Tidak peduli Midtrans atau Stripe
    // Yang penting punya method charge()
    return $gateway->charge($amount);
}

$result1 = processPayment(new Midtrans(), 50000);
$result2 = processPayment(new Stripe(), 50000);
```

### 3.3 Polymorphism via Inheritance (Subtype)

```php
<?php

abstract class Animal
{
    abstract public function speak(): string;
}

class Dog extends Animal
{
    public function speak(): string { return 'Woof!'; }
}

class Cat extends Animal
{
    public function speak(): string { return 'Meow!'; }
}

// Polymorphism:
function makeSound(Animal $animal): string
{
    return $animal->speak();
}

echo makeSound(new Dog()); // "Woof!"
echo makeSound(new Cat()); // "Meow!"
```

### 3.4 Polymorphism di Codebase

**Eloquent Model — method yang sama, tabel berbeda:**

```php
Product::find(1);  // SELECT * FROM products WHERE id = 1
Order::find(1);    // SELECT * FROM orders WHERE id = 1
User::find(1);     // SELECT * FROM users WHERE id = 1
```

Method `find()` yang **sama** menghasilkan query ke **tabel berbeda** — inilah polymorphism.

**Eloquent Relationship — method yang sama, relasi berbeda:**

```php
$product->category();  // belongsTo(Category::class) → ambil 1 kategori
$order->items();       // hasMany(OrderItem::class)  → ambil banyak item
$user->roles();        // belongsToMany(Role::class) → ambil banyak role via pivot
```

Semua adalah **method relationship**, tapi menghasilkan query dan hasil berbeda.

**Policy — authorization untuk model berbeda:**

```php
// OrderPolicy.php
public function view(User $user, Order $order): bool
{
    return $user->id === $order->user_id;
}

// AddressPolicy.php
public function view(User $user, Address $address): bool
{
    return $user->id === $address->user_id;
}
```

Dipanggil dengan cara yang sama:
```php
$this->authorize('view', $order);    // → panggil OrderPolicy@view
$this->authorize('view', $address);  // → panggil AddressPolicy@view
```

### 3.5 Polymorphic Relationship — `morphTo`

Ini polymorphism di level database. Satu tabel bisa berelasi dengan **banyak tabel lain**.

**InventoryMovement.php:27-30:**
```php
public function reference()
{
    return $this->morphTo('reference');
}
```

Tabel `inventory_movements` punya kolom:
```
reference_type  → "App\Models\Order" atau "App\Models\OrderItem"
reference_id    → 1, 2, 3, ...
```

Artinya:
- `$movement->reference` bisa mengembalikan object `Order` atau `OrderItem` — tergantung nilai `reference_type`
- Satu method `reference()`, dua tipe return berbeda — inilah polymorphism

---

## 4. Abstract Class vs Interface

### 4.1 Abstract Class

Class yang **tidak bisa di-instance langsung**. Hanya bisa diturunkan.

```php
<?php

abstract class Controller
{
    // Boleh punya method konkret (dengan implementasi)
    public function authorize(string $ability, mixed $model): void
    {
        // implementasi default
    }

    // Atau method abstract (tanpa implementasi — harus diimplementasi child)
    abstract protected function middleware(): void;
}

// ❌ Tidak bisa: $c = new Controller();
// ✅ Harus:     class CheckoutController extends Controller { ... }
```

**Di codebase — Controller.php:5:**
```php
abstract class Controller
{
    //
}
```

Abstract — agar programmer tidak bisa membuat `new Controller()` secara langsung. Semua controller harus `extends Controller`.

### 4.2 Interface

**Kontrak** yang harus dipenuhi oleh class yang mengimplementasikannya. Semua method di interface harus diimplementasi.

```php
<?php

interface HasMedia
{
    public function addMedia($file): Media;
    public function getFirstMediaUrl(string $collection = ''): string;
}

class Product extends Model implements HasMedia
{
    // WAJIB implementasi semua method dari HasMedia
    use InteractsWithMedia; // trait ini menyediakan implementasi
}
```

**Perbedaan utama:**

| Abstract Class | Interface |
|---------------|-----------|
| Bisa punya properti | Hanya method |
| Bisa punya implementasi method | Semua abstract (tanpa body) |
| Bisa constructor | Tidak bisa |
| Class hanya bisa **extends 1** | Class bisa **implements banyak** |
| `abstract class Foo` | `interface Foo` |

### 4.3 Multiple Inheritance via Interface

PHP tidak mendukung multiple inheritance (seperti C++). Tapi class bisa `implements` banyak interface:

```php
<?php

class Product extends Model implements
    HasMedia,
    HasSlug,
    HasTranslations
{
    // Harus implementasi method dari SEMUA interface
}
```

---

## 5. Trait — Alternatif Inheritance

### 5.1 Masalah yang Dipecahkan Trait

PHP hanya mendukung **single inheritance** — satu class hanya bisa `extends` satu parent. Lalu bagaimana kalau beberapa class butuh kemampuan yang sama?

**Solusi: Trait** — kumpulan method yang bisa dipakai ulang di berbagai class.

```php
<?php

trait Loggable
{
    public function log(string $msg): void
    {
        echo "[LOG] {$msg}";
    }
}

class Product extends Model implements HasMedia
{
    use SoftDeletes, InteractsWithMedia, Loggable;
    //   ↑ trait yang sudah ada   ↑ trait bikinan sendiri
}

class Order extends Model
{
    use Loggable; // Order juga bisa log
}
```

### 5.2 Trait di Codebase

**Product.php:21:**
```php
class Product extends Model implements HasMedia
{
    use SoftDeletes, InteractsWithMedia;
}
```

- `SoftDeletes` — menambah method `delete()` yang tidak benar-benar hapus (set `deleted_at`)
- `InteractsWithMedia` — implementasi dari `HasMedia` interface (Spatie)

**User.php:18:**
```php
class User extends Authenticatable
{
    use HasFactory, Notifiable;
}
```

- `HasFactory` — factory untuk database seeding
- `Notifiable` — method `notify()` untuk kirim notifikasi

**OrderStatusChanged.php:12:**
```php
class OrderStatusChanged extends Notification
{
    use Queueable; // bisa di-queue (async)
}
```

### 5.3 Trait vs Inheritance vs Interface

```
           │   Trait    │ Inheritance │ Interface
───────────┼────────────┼─────────────┼──────────────
Bisa punya  │    ✅      │     ✅      │     ❌
implementasi │           │             │
───────────┼────────────┼─────────────┼──────────────
Bisa punya  │    ✅      │     ✅      │     ❌
properti   │           │             │
───────────┼────────────┼─────────────┼──────────────
Bisa dipakai│    ✅      │     ❌      │     ✅
banyak     │  (use)    │  (extends 1)│  (implements)
───────────┼────────────┼─────────────┼──────────────
Mewakili   │   Perilaku  │    "is-a"   │   Kontrak
konsep     │  reusable  │             │
```

---

## 6. Inheritance vs Komposisi

### 6.1 Masalah Inheritance Berlebihan

Inheritance yang terlalu dalam menyulitkan perubahan:

```
Model
  └── ProductWithDiscount
        └── DigitalProduct
              └── DigitalProductWithDiscount
                    └── ...
```

Setiap level baru menambah kompleksitas. Ini disebut **fragile base class problem** — mengubah parent bisa merusak semua child.

### 6.2 Komposisi — "Has-a" vs "Is-a"

**Komposisi** lebih fleksibel: objek mengandung objek lain (has-a), bukan mewarisi (is-a).

```php
<?php

// Inheritance (is-a):
class OrderService extends Service
{
    // OrderService IS-A Service — kaku
}

// Composition (has-a):
class OrderService
{
    protected CartService $cartService;  // OrderService HAS-A CartService

    public function __construct(CartService $cartService)
    {
        $this->cartService = $cartService; // komposisi via DI
    }
}
```

**Kapan pakai apa:**
- **Inheritance:** `class Product extends Model` — karena Product **is-a** Model (benar secara konsep)
- **Komposisi:** `class OrderService { protected CartService $cartService; }` — karena OrderService **has-a** CartService (bukan is-a)

### 6.3 Favor Komposisi

Ini yang dilakukan Laravel menggunakan **Dependency Injection** — lebih memilih komposisi daripada inheritance.

```
❌ Buruk:
class CheckoutController extends Controller
{
    // Kalau butuh CartService → bikin sendiri
    // Akibat: kode kaku, susah ditest
}

✅ Baik:
class CheckoutController extends Controller
{
    protected CartService $cartService;

    public function __construct(CartService $cartService)
    {
        $this->cartService = $cartService; // komposisi
    }
}
```

---

## 7. Final Keyword

### 7.1 `final class` — Tidak Bisa Diturunkan

```php
<?php

final class EncryptionService
{
    // Class ini TIDAK bisa di-extends
}

// ❌ Error:
// class MyEncryption extends EncryptionService { }
```

### 7.2 `final method` — Tidak Bisa Di-Override

```php
<?php

class Animal
{
    final public function speak(): string
    {
        return '...';
    }
}

class Dog extends Animal
{
    // ❌ Error: tidak bisa override method final
    // public function speak(): string { return 'Woof!'; }
}
```

### 7.3 Final di PHP 8.1+

PHP 8.1 menambahkan `final` untuk class constant:

```php
<?php

class Order
{
    final public const STATUS_PENDING = 'pending';
    // Tidak bisa diubah oleh subclass
}
```

---

## 8. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│ INHERITANCE                                              │
│                                                          │
│  class Animal { speak() }                                │
│       ↑ extends                                          │
│  ┌────┴────┐                                            │
│  │         │                                            │
│ Dog      Cat                                             │
│ (woof)   (meow)                                          │
│                                                          │
│  $dog = new Dog();  $dog->speak();  // "Woof!"           │
│  $cat = new Cat();  $cat->speak();  // "Meow!"           │
│                                                          │
│  → Satu method speak(), implementasi berbeda            │
│  → Inilah POLYMORPHISM                                   │
├──────────────────────────────────────────────────────────┤
│ DI CODEBASE:                                             │
│                                                          │
│  Controller (abstract) ← CheckoutController, ShopCtrl... │
│  Model (Eloquent)     ← Product, Order, User, Setting... │
│  Notification         ← OrderStatusChanged               │
├──────────────────────────────────────────────────────────┤
│ INTERFACE  → Kontrak (wajib implementasi method tertentu) │
│  Product implements HasMedia                             │
│                                                          │
│ TRAIT      → Kode reusable (tanpa inheritance)           │
│  use SoftDeletes, InteractsWithMedia, Notifiable         │
│                                                          │
│ COMPOSITION → HAS-A, lebih baik dari inheritance         │
│  use CartService $cartService (injected via constructor) │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Buat inheritance chain:**
```php
<?php
class Kendaraan
{
    public function __construct(public string $merk) {}

    public function klakson(): string
    {
        return "{$this->merk}: Beep!";
    }
}

class Mobil extends Kendaraan
{
    public int $pintu;

    public function __construct(string $merk, int $pintu)
    {
        parent::__construct($merk);
        $this->pintu = $pintu;
    }
}

class Motor extends Kendaraan
{
    public function klakson(): string  // override
    {
        return "{$this->merk}: Niiin!";
    }
}
```

Analisis: apa hubungan `Mobil` dan `Motor` terhadap `Kendaraan`? Method apa yang diwarisi? Method apa yang di-override?

2. **Buka `app/Models/User.php`.** Identifikasi:
   - Parent class (extends apa?)
   - Traits yang digunakan
   - Method yang di-override dari parent

3. **Buka `app/Models/Product.php`.** Identifikasi:
   - Parent class
   - Interface yang diimplementasi
   - Traits yang digunakan

4. **Polymorphism exercise.** Buat interface `PaymentGateway` dengan method `charge(float $amount): array`. Buat dua implementasi: `MidtransGateway` dan `ManualTransfer`. Buat fungsi `processPayment(PaymentGateway $gw, float $amount)` yang memanggil `charge()`.

5. **Buka `app/Models/InventoryMovement.php`.** Jelaskan method `reference()` — apa yang terjadi saat dipanggil? Bagaimana polymorphism bekerja di sini?

---

## 🔗 Referensi

- [PHP Manual: Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php)
- [PHP Manual: Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php)
- [PHP Manual: Traits](https://www.php.net/manual/en/language.oop5.traits.php)
- [PHP Manual: Polymorphism](https://www.php.net/manual/en/language.oop5.polymorphism.php)
- Codebase: `app/Models/Product.php` — inheritance + interface + trait
- Codebase: `app/Models/User.php` — inheritance chain (User → Authenticatable → Model)
- Codebase: `app/Http/Controllers/Controller.php` — abstract class
- Codebase: `app/Policies/OrderPolicy.php` — polymorphism via policy
- Codebase: `app/Models/InventoryMovement.php` — polymorphic relationship
