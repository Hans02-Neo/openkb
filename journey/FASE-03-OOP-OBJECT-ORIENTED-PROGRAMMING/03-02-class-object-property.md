# 03-02: Class, Object, Property, dan Method

> **Fase**: 3 — Object Oriented Programming  
> **Prasyarat**: 03-01-apa-itu-oop  
> **Waktu baca**: 75-90 menit  
> **Kata kunci**: class, object, instance, property, method, constructor, $this, visibility, static, constant, new, instantiation

---

## 📋 Ringkasan

Dokumen sebelumnya menjelaskan **apa itu OOP** secara konseptual. Sekarang kita akan menulis kode OOP sesungguhnya — membuat class, meng-instance menjadi object, mendefinisikan properti dan method.

**Target pemahaman:**
- Kamu bisa membuat class sendiri dari nol
- Kamu paham perbedaan class vs object vs instance
- Kamu bisa menggunakan `$this`, `new`, constructor, visibility
- Kamu bisa membaca dan memahami class di codebase ini

---

## 1. Class — Cetakan

### 1.1 Syntax Class Paling Sederhana

```php
<?php

class Product
{
    // properti dan method akan ditulis di sini
}
```

Dengan deklarasi `class Product`, PHP sekarang tahu ada cetakan bernama `Product`. Tapi belum ada objek yang dibuat.

### 1.2 Class dengan Properti

```php
<?php

class Product
{
    public string $name;
    public float $price;
    public int $stock;
}
```

**Properti** adalah variabel yang dimiliki oleh class. Setiap objek dari class ini akan punya salinan properti sendiri.

### 1.3 Class dengan Method

```php
<?php

class Product
{
    public string $name;
    public float $price;

    public function formatRupiah(): string
    {
        return 'Rp ' . number_format($this->price, 0, ',', '.');
    }
}
```

**Method** adalah fungsi yang dimiliki oleh class. Method bisa mengakses properti via `$this`.

---

## 2. Object — Instance dari Class

### 2.1 Membuat Object dengan `new`

```php
<?php
$product = new Product(); // ← instance/object dari class Product
```

**Yang terjadi di memori saat `new Product()`:**
1. PHP alokasi memori untuk object baru (heap)
2. Properti diinisialisasi sesuai default
3. Constructor dipanggil (jika ada)
4. Object dikembalikan ke variabel `$product`

### 2.2 Mengisi dan Membaca Properti

```php
<?php
$product = new Product();
$product->name = 'Laptop Gaming';  // ← tulis properti
$product->price = 15000000;

echo $product->name;               // ← baca properti
echo $product->formatRupiah();     // ← panggil method
```

### 2.3 Multiple Instance

```php
<?php
$laptop  = new Product();
$laptop->name = 'Laptop Gaming';

$mouse   = new Product();
$mouse->name = 'Mouse Wireless';

echo $laptop->name; // "Laptop Gaming"
echo $mouse->name;  // "Mouse Wireless"
// Dua objek berbeda — data mereka terpisah
```

**Di memori:**
```
$laptop → [Object Product #1]
              name: "Laptop Gaming"
              price: 0

$mouse  → [Object Product #2]
              name: "Mouse Wireless"
              price: 0
```

Setiap instance punya **ruang sendiri** di heap. `$laptop->name` dan `$mouse->name` adalah lokasi memori yang berbeda.

---

## 3. Properti — Data yang Dimiliki Objek

### 3.1 Type Declaration pada Properti (PHP 7.4+)

```php
<?php

class Product
{
    public string $name;          // string
    public ?string $description;  // nullable string
    public float $price;          // float
    public int $stock = 0;        // int dengan default
    public bool $isActive = true; // bool dengan default
    public array $tags = [];      // array
    public mixed $metadata = null; // mixed (PHP 8.0+)
}
```

**Tipe yang didukung:** `string`, `int`, `float`, `bool`, `array`, `object`, `mixed`, `callable`, `iterable`, `self`, `parent`, `nama_class`, `interface`, union type (`string|int`), `never`, `void` (untuk return type).

### 3.2 Default Value

Properti bisa punya nilai default — dan nilai ini di-set **satu kali** saat class didefinisikan, bukan saat instance dibuat.

```php
<?php

class Product
{
    public int $stock = 0;        // default 0 — jika tidak di-set
    public bool $isActive = true; // default true
    public string $name;          // tanpa default — WAJIB diisi (atau via constructor)
}
```

### 3.3 Properti di Codebase

**OrderItem.php:12-18:**
```php
class OrderItem extends Model
{
    protected $fillable = [
        'order_id', 'product_id', 'product_name', 'quantity', 'price',
    ];

    protected $casts = [
        'price'    => 'decimal:2',
        'quantity' => 'integer',
    ];
}
```

**CartService.php:10:**
```php
class CartService
{
    protected $sessionKey = 'shopping_cart';
    //       ↑ properti dengan visibility protected dan default value
}
```

**OrderStatusChanged.php:14:**
```php
class OrderStatusChanged extends Notification
{
    protected $order;
    //       ↑ properti tanpa type declaration (tipe ditentukan di docblock atau runtime)
}
```

---

## 4. Visibility — Siapa Bisa Mengakses

### 4.1 Tiga Level Akses

| Keyword | Dari dalam class | Dari subclass | Dari luar |
|---------|:---:|:---:|:---:|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

### 4.2 Contoh

```php
<?php

class Product
{
    public string $name;         // siapa pun bisa baca/tulis
    protected float $price;      // hanya class ini dan turunannya
    private string $secretNote;  // hanya class ini
}
```

### 4.3 Kenapa Tidak Semua `public`?

**Prinsip encapsulation:** Objek harus menjaga **integritas datanya** sendiri.

```php
<?php

class Product
{
    private float $price;

    public function getPrice(): float
    {
        return $this->price;
    }

    public function setPrice(float $price): void
    {
        if ($price <= 0) {
            throw new \InvalidArgumentException('Price must be positive');
        }
        $this->price = $price;
    }
}
```

Dengan `private`, harga tidak bisa diubah seenaknya. Semua perubahan harus lewat `setPrice()` yang punya validasi.

### 4.4 `protected` — Paling Sering di Codebase

Di codebase ini, properti service dan controller hampir selalu `protected`:

```php
class CheckoutController extends Controller
{
    protected $cartService;       // ← protected
    protected $addressService;    // ← protected
    protected $orderService;      // ← protected
    protected $rajaOngkirService; // ← protected
}
```

**Kenapa `protected` bukan `private`?** Karena saat testing, kadang perlu akses properti dari subclass atau dari test case. `protected` memberi fleksibilitas tanpa membuka ke publik.

---

## 5. Method — Perilaku Objek

### 5.1 Method Declaration

```php
<?php

class Product
{
    public function formatRupiah(): string
    {
        return 'Rp ' . number_format($this->price, 0, ',', '.');
    }

    public function isAvailable(): bool
    {
        return $this->stock > 0;
    }

    public function reduceStock(int $quantity): void
    {
        if ($this->stock < $quantity) {
            throw new \RuntimeException('Stok tidak cukup');
        }
        $this->stock -= $quantity;
    }
}
```

Method adalah fungsi biasa — boleh punya parameter, return type, visibility. Bedanya: method **milik class** dan bisa akses `$this`.

### 5.2 Method di Codebase

**Coupon.php:30-37 — method dengan return type bool:**
```php
public function isValid(): bool
{
    if (!$this->is_active) return false;
    if ($this->starts_at && now()->lt($this->starts_at)) return false;
    if ($this->expires_at && now()->gt($this->expires_at)) return false;
    if ($this->usage_limit && $this->used_count >= $this->usage_limit) return false;
    return true;
}
```

**Coupon.php:39-52 — method dengan parameter dan return type:**
```php
public function calculateDiscount(float $subtotal): float
{
    if ($subtotal < $this->minimum_order) return 0;

    $discount = $this->type === 'percentage'
        ? $subtotal * ($this->value / 100)
        : $this->value;

    if ($this->maximum_discount) {
        $discount = min($discount, $this->maximum_discount);
    }

    return $discount;
}
```

**OrderItem.php:35-38 — accessor method:**
```php
public function getSubtotalAttribute(): float
{
    return $this->price * $this->quantity;
}
```

### 5.3 Method Chaining

Method yang mengembalikan `$this` atau `static` bisa **dirantai**:

```php
<?php

class QueryBuilder
{
    protected array $wheres = [];

    public function where(string $column, mixed $value): static
    {
        $this->wheres[] = [$column, $value];
        return $this; // ← kunci chaining
    }

    public function orderBy(string $column): static
    {
        // ...
        return $this;
    }

    public function get(): array
    {
        // execute query
        return $results;
    }
}

// Chaining:
$results = (new QueryBuilder())
    ->where('status', 'active')
    ->where('stock', '>', 0)
    ->orderBy('name')
    ->get();
```

---

## 6. Constructor — Method Spesial

### 6.1 Apa Itu Constructor

`__construct()` adalah method yang dipanggil **otomatis** saat objek dibuat dengan `new`. Gunakan constructor untuk:

1. Menerima data awal (dependency injection)
2. Inisialisasi properti
3. Setup yang diperlukan object

### 6.2 Constructor Sederhana

```php
<?php

class Product
{
    public function __construct(
        public string $name,
        public float $price,
        public int $stock = 0
    ) {
        // Constructor body — bisa diisi logika tambahan
    }
}

// Cara pakai:
$product = new Product('Laptop Gaming', 15000000, 10);
echo $product->name;  // "Laptop Gaming"
echo $product->price; // 15000000
```

**Promoted Constructor Property (PHP 8.0+):** Parameter constructor langsung jadi properti. Tiga baris di atas setara dengan:

```php
<?php

class Product
{
    public string $name;
    public float $price;
    public int $stock;

    public function __construct(string $name, float $price, int $stock = 0)
    {
        $this->name = $name;
        $this->price = $price;
        $this->stock = $stock;
    }
}
```

### 6.3 Constructor di Codebase

**CheckoutController.php:19-29 — constructor dengan dependency injection:**
```php
public function __construct(
    CartService $cartService,
    AddressService $addressService,
    OrderService $orderService,
    \App\Services\RajaOngkirService $rajaOngkirService
) {
    $this->cartService = $cartService;
    $this->addressService = $addressService;
    $this->orderService = $orderService;
    $this->rajaOngkirService = $rajaOngkirService;
}
```

**OrderStatusChanged.php:16-19 — constructor sederhana:**
```php
public function __construct($order)
{
    $this->order = $order;
}
```

**MidtransService.php:11-18 — constructor setup konfigurasi:**
```php
public function __construct()
{
    Config::$serverKey = config('services.midtrans.server_key');
    Config::$clientKey = config('services.midtrans.client_key');
    Config::$isProduction = config('services.midtrans.is_production');
    Config::$isSanitized = true;
    Config::$is3ds = true;
}
```

### 6.4 Destructor

`__destruct()` dipanggil saat objek dihapus dari memori. Jarang dipakai di aplikasi web (karena request pendek), tapi berguna untuk cleanup resource.

```php
<?php

class DatabaseConnection
{
    protected $connection;

    public function __destruct()
    {
        // Tutup koneksi database saat objek dihapus
        $this->connection->close();
    }
}
```

---

## 7. `$this` — Diri Sendiri

### 7.1 Fungsi `$this`

`$this` adalah **referensi ke instance saat ini**. Digunakan di dalam method untuk mengakses properti atau method lain dari objek yang sama.

```php
<?php

class Product
{
    public string $name;
    public float $price;

    public function getInfo(): string
    {
        return $this->name . ' - Rp ' . $this->price;
        //    ↑ $this->name mengacu ke properti $name dari INSTANCE INI
    }

    public function callAnother(): void
    {
        $this->getInfo(); // panggil method lain di objek yang sama
    }
}
```

### 7.2 `$this` Tidak Bisa di Static Method

```php
<?php

class Product
{
    public static function staticMethod(): void
    {
        // echo $this->name; ❌ Fatal Error!
        // Static method tidak punya $this
    }
}
```

### 7.3 Ilustrasi Memori

```php
<?php
$a = new Product();
$a->name = 'Product A';

$b = new Product();
$b->name = 'Product B';

$a->getInfo(); // Di sini $this = $a → "Product A - Rp 0"
$b->getInfo(); // Di sini $this = $b → "Product B - Rp 0"
```

Class `Product` hanya ada **satu** di memori (di segmen code). Tiap `$this` menunjuk ke instance yang berbeda.

---

## 8. Static — Milik Class, Bukan Instance

### 8.1 Properti dan Method Static

Properti/method `static` dimiliki oleh **class**, bukan oleh instance.

```php
<?php

class Counter
{
    public static int $count = 0;

    public static function increment(): void
    {
        self::$count++;
    }
}

// Panggil tanpa instance:
echo Counter::$count;     // 0
Counter::increment();
Counter::increment();
echo Counter::$count;     // 2

// Semua instance berbagi nilai yang SAMA
$c1 = new Counter();
$c2 = new Counter();
echo $c1::$count; // 2 (sama dengan $c2::$count)
```

### 8.2 Static di Codebase

**Setting.php:23-34:**
```php
public static function get(string $key, mixed $default = null): mixed
{
    $setting = static::where('key', $key)->first();
    if (!$setting) return $default;

    return match ($setting->type) {
        'boolean'   => (bool) $setting->value,
        'integer'   => (int) $setting->value,
        'json'      => json_decode($setting->value, true),
        default     => $setting->value,
    };
}
```

Dipanggil: `Setting::get('key')` — tanpa `new Setting()`.

**Order.php:28-32 — class constant (implisit static):**
```php
const STATUS_PENDING    = 'pending';
const STATUS_PROCESSING = 'processing';
const STATUS_SHIPPED    = 'shipped';
const STATUS_COMPLETED  = 'completed';
const STATUS_CANCELLED  = 'cancelled';
```

Dipanggil: `Order::STATUS_PENDING`, `Order::STATUS_PROCESSING`, dll.

### 8.3 `self::` vs `static::`

- `self::` — resolves ke class **tempat kode ditulis** (early binding)
- `static::` — resolves ke class **yang dipanggil saat runtime** (late static binding)

```php
<?php
class Parent_
{
    public static function test(): string
    {
        return self::who();   // selalu "Parent" — karena self:: lihat class ini
        // vs
        return static::who(); // tergantung class pemanggil
    }

    protected static function who(): string
    {
        return 'Parent';
    }
}

class Child extends Parent_
{
    protected static function who(): string
    {
        return 'Child';
    }
}

echo Child::test(); // self:: → "Parent", static:: → "Child"
```

Di Eloquent model, `static::where(...)` memastikan query berjalan di tabel class yang benar (misal `Product::where(...)` → tabel `products`, `Order::where(...)` → tabel `orders`).

---

## 9. Class Constant

### 9.1 Definisi

Konstanta class adalah nilai yang **tetap** dan **sama untuk semua instance**.

```php
<?php

class Order
{
    const STATUS_PENDING    = 'pending';
    const STATUS_PROCESSING = 'processing';
    const STATUS_SHIPPED    = 'shipped';
    const STATUS_COMPLETED  = 'completed';
    const STATUS_CANCELLED  = 'cancelled';

    public string $status;

    public function __construct()
    {
        $this->status = self::STATUS_PENDING; // pakai di dalam class
    }
}

// Pakai di luar class:
echo Order::STATUS_PENDING;  // "pending"
echo Order::STATUS_CANCELLED; // "cancelled"
```

**Kenapa pakai constant?** Daripada menulis string `'pending'` di banyak tempat (risiko typo), pakai `Order::STATUS_PENDING`. Jika suatu saat status berubah, cukup ubah di satu tempat.

### 9.2 PHP 8.1: `final` Class Constant

```php
<?php

class Order
{
    final public const STATUS_PENDING = 'pending';
    // ↑ tidak bisa di-override oleh subclass
}
```

---

## 10. Type Declaration di Properti — Mana Saja yang Ada?

### 10.1 Di Codebase Ini

Codebase menggunakan **promoted constructor properties** (PHP 8.0+) dan **properti dengan type declaration** di beberapa tempat:

**RajaOngkirService.php:10-11 — properti tanpa tipe (PHP 7 style):**
```php
protected $apiKey;
protected $baseUrl;
```

**CartService.php:10 — properti dengan default value:**
```php
protected $sessionKey = 'shopping_cart';
```

**MidtransService.php — constructor tanpa parameter, setup static:**
```php
public function __construct()
{
    Config::$serverKey = config('services.midtrans.server_key');
}
```

Beberapa class masih pakai gaya lama (tanpa type declaration di properti). Ini adalah PHP 7 style — masih valid, tapi PHP 8+ lebih baik dengan tipe.

### 10.2 Modern Style dengan PHP 8

```php
<?php

class Product
{
    // Promoted constructor properties — semua jadi properti sekaligus
    public function __construct(
        private string $name,
        private float $price,
        private int $stock = 0,
        private ?string $description = null,
        private array $tags = [],
    ) {}
}
```

---

## 11. Object Comparison

### 11.1 `==` vs `===`

```php
<?php

$a = new Product();
$a->name = 'Laptop';

$b = new Product();
$b->name = 'Laptop';

$c = $a; // reference ke objek yang SAMA

$a == $b;   // true — semua properti sama (comparison)
$a === $b;  // false — objek berbeda (identity)
$a === $c;  // true — objek yang SAMA
```

- `==` (equality) — true jika semua properti sama dan class sama
- `===` (identity) — true jika merujuk ke instance yang sama

### 11.2 Reference

Di PHP, objek selalu **pass by reference** (by default). Tidak seperti array atau string yang di-copy saat assignment.

```php
<?php
$a = new Product();
$a->name = 'Laptop';

$b = $a; // $b menunjuk ke objek yang SAMA
$b->name = 'Mouse';

echo $a->name; // "Mouse" — berubah juga!
```

Ini berbeda dengan array:
```php
<?php
$a = ['name' => 'Laptop'];
$b = $a;        // COPY!
$b['name'] = 'Mouse';

echo $a['name']; // "Laptop" — tidak berubah
```

---

## 12. Anonymous Class

Class yang didefinisikan **tanpa nama** — biasanya untuk object sekali pakai.

```php
<?php
$logger = new class {
    public function log(string $msg): void
    {
        echo "LOG: {$msg}";
    }
};

$logger->log('testing'); // "LOG: testing"
```

Anonymous class jarang dipakai di aplikasi normal. Biasanya di unit test atau mocking.

---

## 13. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────┐
│                  CLASS = CETAKAN                         │
├─────────────────────────────────────────────────────────┤
│  class Product                                          │
│  {                                                      │
│      // PROPERTI — data                                 │
│      public string $name;                               │
│      protected float $price;                            │
│      private int $stock;                                │
│      public static int $totalProducts = 0;  // static   │
│      const TAX_RATE = 0.11;                  // constant │
│                                                          │
│      // CONSTRUCTOR                                     │
│      public function __construct(string $name)           │
│      {                                                  │
│          $this->name = $name;                            │
│          self::$totalProducts++;                         │
│      }                                                   │
│                                                          │
│      // METHOD — perilaku                                │
│      public function getName(): string                   │
│      {                                                  │
│          return $this->name;                             │
│      }                                                   │
│                                                          │
│      public static function resetCount(): void           │
│      {                                                  │
│          self::$totalProducts = 0;                       │
│      }                                                   │
│  }                                                       │
│                                                          │
│  // PENGGUNAAN                                          │
│  $p = new Product('Laptop');   // instance / object      │
│  echo $p->getName();           // method call            │
│  echo Product::TAX_RATE;       // constant access         │
└─────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Buat class `Cart`** dengan properti:
   - `private array $items = []`
   - Method `addItem(string $name, int $price, int $qty)`
   - Method `getTotal(): int`
   - Method `count(): int`

2. **Buat class `Logger`** dengan:
   - `public static function info(string $msg): void`
   - `public static function error(string $msg): void`
   - Setiap method mencetak `[INFO] msg` atau `[ERROR] msg`

3. **Analisis `app/Models/Coupon.php`**:
   - Identifikasi semua properti dan visibility-nya
   - Identifikasi semua method
   - Apa return type dari `isValid()`?
   - Apa return type dari `calculateDiscount()`?

4. **Analisis `app/Notifications/OrderStatusChanged.php`**:
   - Constructor menerima parameter apa?
   - Properti apa yang di-set?
   - Method apa saja yang dimiliki?

5. **Buat instance dari class `Product`** (di `app/Models/Product.php`) via `php artisan tinker`:
   ```bash
   php artisan tinker
   >>> $p = new App\Models\Product();
   >>> $p->name = 'Test';
   >>> $p->save();
   >>> $p->fresh();
   ```

---

## 🔗 Referensi

- [PHP Manual: Classes and Objects](https://www.php.net/manual/en/language.oop5.basic.php)
- [PHP Manual: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php)
- [PHP Manual: Constructors](https://www.php.net/manual/en/language.oop5.decon.php)
- [PHP Manual: Static](https://www.php.net/manual/en/language.oop5.static.php)
- [PHP 8.0: Constructor Promotion](https://www.php.net/releases/8.0/en.php#constructor-property-promotion)
- Codebase: `app/Models/Order.php` — class constants + properties + methods
- Codebase: `app/Models/Coupon.php` — method dengan parameter dan return type
- Codebase: `app/Services/CartService.php` — class dengan properti + method
- Codebase: `app/Http/Controllers/CheckoutController.php` — constructor DI
- Codebase: `app/Notifications/OrderStatusChanged.php` — constructor sederhana
