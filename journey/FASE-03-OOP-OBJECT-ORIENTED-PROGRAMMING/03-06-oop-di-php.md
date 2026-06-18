# 03-06: OOP di PHP 8+ — Fitur Modern

> **Fase**: 3 — Object Oriented Programming  
> **Prasyarat**: 03-05-dependency-injection  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: PHP 8, constructor promotion, readonly, enum, match, named argument, attribute, union type, mixed, nullsafe, fiber

---

## 📋 Ringkasan

PHP 7 memperkenalkan **type hints** dan **return types**. PHP 8 membawa **revolusi OOP** — fitur yang membuat kode lebih pendek, lebih aman, dan lebih ekspresif.

**Target pemahaman:**
- Kamu bisa membaca dan menulis promoted constructor properties
- Kamu paham `readonly`, `enum`, dan `match`
- Kamu bisa menggunakan named arguments dan attributes
- Kamu tahu fitur PHP 8 apa yang dipakai di codebase ini

---

## 1. Constructor Property Promotion (PHP 8.0)

### 1.1 Sebelum PHP 8.0

```php
<?php

class Product
{
    private string $name;
    private float $price;
    private int $stock;

    public function __construct(string $name, float $price, int $stock = 0)
    {
        $this->name = $name;
        $this->price = $price;
        $this->stock = $stock;
    }
}
```

12 baris untuk deklarasi + assignment properti.

### 1.2 Dengan PHP 8.0

```php
<?php

class Product
{
    public function __construct(
        private string $name,
        private float $price,
        private int $stock = 0,
    ) {}
}
```

4 baris. Sama persis fungsinya.

**Cara kerja:** Parameter constructor dengan visibility (`private`, `protected`, `public`) secara otomatis:
1. Mendeklarasikan properti dengan nama yang sama
2. Mengisi properti dengan nilai parameter

### 1.3 Di Codebase Ini

Codebase ini **belum banyak** menggunakan promoted constructor. Sebagian besar masih menulis assignment manual:

```php
// CheckoutController.php — masih manual (valid, tapi lebih panjang)
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

Versi promoted (akan lebih pendek):
```php
public function __construct(
    private CartService $cartService,
    private AddressService $addressService,
    private OrderService $orderService,
    private \App\Services\RajaOngkirService $rajaOngkirService,
) {}
```

### 1.4 Aturan Promoted Properties

```php
<?php

class User
{
    public function __construct(
        // ✅ Boleh campur promoted dan non-promoted
        private string $name,
        string $password, // parameter biasa (tidak jadi properti)

        // ✅ Boleh dengan default
        private bool $isActive = true,

        // ✅ Boleh dengan type union
        private string|null $phone = null,

        // ✅ Boleh dengan readonly (PHP 8.1)
        private readonly string $uuid,

        // ✅ Boleh dengan atribut (PHP 8.0)
        #[Email]
        private string $email,
    ) {
        // Properti non-promoted harus di-assign manual
        $this->password = bcrypt($password);
    }
}
```

---

## 2. Readonly Properties (PHP 8.1)

### 2.1 Definisi

Properti `readonly` hanya bisa di-set **sekali** — biasanya di constructor. Setelah itu tidak bisa diubah.

```php
<?php

class Order
{
    public function __construct(
        private readonly string $orderNumber,
        private readonly float $totalAmount,
    ) {}

    public function getOrderNumber(): string
    {
        return $this->orderNumber;
    }
}

$order = new Order('ORD-123', 50000);
echo $order->getOrderNumber(); // "ORD-123"
// $order->orderNumber = 'ORD-456'; ❌ Error! Readonly!
```

**Kenapa readonly?**
1. **Immutability** — objek tidak berubah setelah dibuat, lebih aman
2. **Intent** — jelas bahwa nilai ini tidak akan berubah
3. **Performance** — PHP bisa optimasi memory

### 2.2 Readonly Class (PHP 8.2)

PHP 8.2 menambahkan `readonly` untuk seluruh class — semua properti otomatis `readonly`:

```php
<?php

readonly class Configuration
{
    public function __construct(
        public string $apiKey,
        public string $baseUrl,
        public int $timeout,
    ) {}
}
```

---

## 3. Enums (PHP 8.1)

### 3.1 Definisi

**Enum** adalah tipe data yang nilainya terbatas pada **kumpulan nilai yang sudah ditentukan**.

```php
<?php

enum OrderStatus: string
{
    case Pending = 'pending';
    case Processing = 'processing';
    case Shipped = 'shipped';
    case Completed = 'completed';
    case Cancelled = 'cancelled';
}
```

### 3.2 Penggunaan

```php
<?php
// Assign
$status = OrderStatus::Pending;

// Baca value
echo $status->value; // "pending"

// Dari string
$status = OrderStatus::from('processing'); // OrderStatus::Processing

// Match dengan enum
$label = match ($status) {
    OrderStatus::Pending    => 'Menunggu Pembayaran',
    OrderStatus::Processing => 'Diproses',
    OrderStatus::Shipped    => 'Dikirim',
    OrderStatus::Completed  => 'Selesai',
    OrderStatus::Cancelled  => 'Dibatalkan',
};
```

### 3.3 Enum Methods

```php
<?php

enum OrderStatus: string
{
    case Pending = 'pending';
    case Processing = 'processing';
    case Shipped = 'shipped';
    case Completed = 'completed';
    case Cancelled = 'cancelled';

    public function label(): string
    {
        return match ($this) {
            self::Pending    => 'Menunggu Pembayaran',
            self::Processing => 'Diproses',
            self::Shipped    => 'Dikirim',
            self::Completed  => 'Selesai',
            self::Cancelled  => 'Dibatalkan',
        };
    }

    public function color(): string
    {
        return match ($this) {
            self::Pending    => 'yellow',
            self::Processing => 'blue',
            self::Shipped    => 'indigo',
            self::Completed  => 'green',
            self::Cancelled  => 'red',
        };
    }
}

echo OrderStatus::Pending->label(); // "Menunggu Pembayaran"
```

### 3.4 Di Codebase Ini

Codebase ini **belum** menggunakan Enum — status masih pakai string constant:

```php
class Order extends Model
{
    const STATUS_PENDING    = 'pending';
    const STATUS_PROCESSING = 'processing';
    const STATUS_SHIPPED    = 'shipped';
    const STATUS_COMPLETED  = 'completed';
    const STATUS_CANCELLED  = 'cancelled';
}
```

Enum akan lebih baik karena:
1. Type safety — `function (OrderStatus $status)` bukan `function (string $status)`
2. Auto-completion di IDE
3. Method bisa ditempel di enum (label, color, dll)

---

## 4. Match Expression (PHP 8.0)

### 4.1 Match vs Switch

`match` adalah **expression** (mengembalikan nilai), bukan statement seperti `switch`.

```php
<?php
// Switch (statement — tidak return nilai)
switch ($status) {
    case 'pending':
        $label = 'Menunggu Pembayaran';
        break;
    case 'processing':
        $label = 'Diproses';
        break;
    default:
        $label = ucfirst($status);
}

// Match (expression — return nilai)
$label = match ($status) {
    'pending'    => 'Menunggu Pembayaran',
    'processing' => 'Diproses',
    'shipped'    => 'Dikirim',
    'completed'  => 'Selesai',
    'cancelled'  => 'Dibatalkan',
    default      => ucfirst($status),
};
```

### 4.2 Match di Codebase

**Order.php:59-68:**
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

**Setting.php:28-33:**
```php
return match ($setting->type) {
    'boolean'   => (bool) $setting->value,
    'integer'   => (int) $setting->value,
    'json'      => json_decode($setting->value, true),
    default     => $setting->value,
};
```

### 4.3 Keunggulan Match

| | Switch | Match |
|---|--------|-------|
| Return value | Tidak (butuh variable) | **Ya** (expression) |
| `break` | Wajib | Tidak perlu |
| Perbandingan | `==` (loose) | `===` (strict) |
| Multiple case | `case 1: case 2:` | `1, 2 =>` |
| Unhandled | Tidak error | **Throw `UnhandledMatchError`** |

---

## 5. Named Arguments (PHP 8.0)

### 5.1 Syntax

Named arguments memungkinkan passing parameter **berdasarkan nama**, bukan posisi.

```php
<?php

function createUser(
    string $name,
    string $email,
    bool $isAdmin = false,
    bool $isActive = true,
    ?string $phone = null,
) {}

// Tanpa named argument — harus ikut urutan:
createUser('Budi', 'budi@email.com', false, true, null);

// Dengan named argument — skip parameter default:
createUser(
    name: 'Budi',
    email: 'budi@email.com',
    phone: '08123456789',
    // isAdmin dan isActive pakai default
);
```

### 5.2 Di Codebase

**RegisteredUserController.php:56:**
```php
return redirect(route('dashboard', absolute: false));
```

Tanpa named argument: `route('dashboard', [], false, null)` — tidak jelas apa artinya `false`.

### 5.3 Aturan Named Arguments

```php
<?php
function test(string $a, string $b, string $c = 'default') {}

// ✅ Boleh:
test(a: '1', b: '2');           // semua named
test('1', b: '2');               // campur positional + named
test('1', '2', c: 'custom');     // positional dulu, named setelah

// ❌ Tidak boleh:
test(b: '2', '1');               // named argument sebelum positional!
test(a: '1', a: '2');            // duplikat parameter!
```

---

## 6. Attributes (PHP 8.0)

### 6.1 Definisi

**Attributes** (`#[...]`) adalah metadata terstruktur yang ditempel ke class, method, properti, atau parameter.

```php
<?php

#[Route('/api/products', methods: ['GET'])]
class ProductController
{
    #[Inject]
    private ProductService $service;

    #[Authorized('admin')]
    public function index(): array { /* ... */ }
}
```

### 6.2 Di Codebase

**Product.php:12-18:**
```php
#[Fillable([
    'category_id', 'brand_id', 'name', 'slug', 'sku',
    'short_description', 'description', 'price',
    'compare_at_price', 'cost_price', 'stock_quantity',
    'low_stock_threshold', 'weight_grams', 'dimensions_json',
    'is_active', 'is_featured', 'meta_title', 'meta_description'
])]
class Product extends Model implements HasMedia
```

**User.php:13-14:**
```php
#[Fillable(['name', 'email', 'password', 'phone', 'avatar', 'is_active', 'last_login_at'])]
#[Hidden(['password', 'remember_token'])]
class User extends Authenticatable
```

Ini adalah **alternative** untuk properti `$fillable` dan `$hidden` — lebih modern.

---

## 7. Union Types (PHP 8.0)

### 7.1 Syntax

Parameter/properti bisa menerima **lebih dari satu tipe**.

```php
<?php

class Product
{
    // Union type: string ATAU int
    public function find(int|string $id): Product|null {}

    // Nullable union
    public function getDiscount(): float|int|null {}

    // Mixed (semua tipe)
    public function setMeta(mixed $data): void {}
}
```

### 7.2 Di Codebase

**Setting.php:23:**
```php
public static function get(string $key, mixed $default = null): mixed
```

---

## 8. Nullsafe Operator (PHP 8.0)

### 8.1 Masalah

Sebelum PHP 8, akses properti/method dari object yang mungkin `null` butuh pengecekan:

```php
<?php
// Sebelum PHP 8:
$country = null;
if ($order !== null) {
    if ($order->address !== null) {
        $country = $order->address->country;
    }
}

// Atau dengan null coalescing:
$country = $order?->address?->country;
// Tapi ini bisa error kalau intermediate null
```

### 8.2 Nullsafe

```php
<?php
// PHP 8.0:
$country = $order?->address?->country;
// Jika $order null → return null
// Jika $order->address null → return null
// Jika semua ada → return $order->address->country
```

### 8.3 Di Codebase

**OrderItem.php:42:**
```php
return $this->product_name ?? ($this->product?->name ?? 'Produk Dihapus');
```

`$this->product?->name` — jika `$this->product` null, tidak throw error.

---

## 9. Apa yang BELUM Dipakai di Codebase Ini

Codebase ini sudah menggunakan beberapa fitur PHP 8:
- ✅ Match expression (Order.php, Setting.php, ShopController.php)
- ✅ Named arguments (`absolute: false`)
- ✅ Attributes (`#[Fillable]`, `#[Hidden]`)
- ✅ Union types (`mixed`)
- ✅ Nullsafe (`$this->product?->name`)

Yang **belum** dipakai:
- ❌ Constructor promotion (masih manual assignment)
- ❌ Enum (masih pakai class constant)
- ❌ Readonly properties
- ❌ First-class callable (`Str::slug(...)`)
- ❌ `str_contains`, `str_starts_with`, `str_ends_with` (di kode aplikasi — mungkin dipakai di vendor)

---

## 10. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│ FITUR PHP 8+ UNTUK OOP                                   │
├──────────────────────────────────────────────────────────┤
│  Constructor Promotion (8.0)                             │
│    → Deklarasi + assignment properti dalam 1 langkah      │
│    → public function __construct(private string $name)   │
│                                                          │
│  Readonly (8.1)                                          │
│    → Properti hanya bisa di-set sekali                   │
│    → public function __construct(private readonly $id)  │
│                                                          │
│  Enum (8.1)                                              │
│    → Kumpulan nilai tetap dengan type safety             │
│    → enum OrderStatus: string { case Pending = 'p'; }    │
│                                                          │
│  Match (8.0)                                             │
│    → Switch expression yang return nilai                 │
│    → $x = match($a) { 1 => 'one', default => 'other' }  │
│                                                          │
│  Named Arguments (8.0)                                   │
│    → Skip parameter default, sebut nama parameter        │
│    → f(name: 'Budi', email: 'b@b.com')                  │
│                                                          │
│  Attributes (8.0)                                        │
│    → Metadata terstruktur untuk class/properti/method    │
│    → #[Route('/api')] class ProductController            │
│                                                          │
│  Union Types (8.0)                                       │
│    → int|string, float|int|null, mixed                   │
│                                                          │
│  Nullsafe (8.0)                                          │
│    → $order?->address?->country (tidak error di null)    │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Ubah class constant jadi enum.** Buat file `StatusEnum.php`:
```php
<?php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Processing = 'processing';
    case Shipped = 'shipped';
    case Completed = 'completed';
    case Cancelled = 'cancelled';

    public function label(): string { /* ... */ }
    public function color(): string { /* ... */ }
}
```

2. **Refactor constructor.** Buka `app/Http/Controllers/CartController.php`. Tulis ulang constructornya menggunakan **promoted constructor properties**.

3. **Match practice.** Tanpa buka referensi, tulis `match` expression yang mengubah angka 1-5 menjadi teks (one, two, three, four, five).

4. **Named arguments.** Cari semua penggunaan `route('...', absolute: false)` di codebase. Apa parameter `absolute` lakukan?

5. **Analisis `app/Models/Setting.php`.** Identifikasi fitur PHP 8 apa yang dipakai di file ini (match, mixed, union type, dll).

---

## 🔗 Referensi

- [PHP 8.0 Release Notes](https://www.php.net/releases/8.0/en.php)
- [PHP 8.1 Release Notes](https://www.php.net/releases/8.1/en.php)
- [PHP 8.2 Release Notes](https://www.php.net/releases/8.2/en.php)
- [PHP Manual: Enums](https://www.php.net/manual/en/language.enumerations.php)
- [PHP Manual: Attributes](https://www.php.net/manual/en/language.attributes.php)
- Codebase: `app/Models/Order.php` — match expression
- Codebase: `app/Models/Setting.php` — match + mixed
- Codebase: `app/Models/Product.php` — attributes
