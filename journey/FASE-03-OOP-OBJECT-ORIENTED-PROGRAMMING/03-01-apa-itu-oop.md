# 03-01: Apa Itu OOP? — Paradigma yang Mengubah Cara Kita Menulis Kode

> **Fase**: 3 — Object Oriented Programming  
> **Prasyarat**: FASE 2 (PHP Fundamental)  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: OOP, paradigma, procedural vs OOP, class, object, state, behavior, blueprint, instance, 4 pillars

---

## 📋 Ringkasan

Selama Fase 2, kamu menulis PHP secara **procedural** — fungsi dipanggil, data lewat parameter, hasil kembali. Itu cara yang valid, tapi untuk aplikasi besar seperti Laravel, ada cara yang lebih kuat: **Object-Oriented Programming (OOP)**.

OOP bukan sekadar "cara lain menulis kode". OOP adalah **cara berpikir** tentang software — memandang program sebagai kumpulan **objek** yang saling berkomunikasi, bukan sebagai urutan instruksi.

Setelah dokumen ini kamu akan paham:
- Apa perbedaan procedural vs OOP
- Apa itu class dan object (dengan analogi mendalam)
- Mengapa OOP ada — masalah apa yang dipecahkan
- Gambaran besar 4 pilar OOP
- Bagaimana OOP terlihat di codebase ini

---

## 1. Procedural vs OOP — Dua Cara Berpikir

### 1.1 Procedural Programming

Procedural programming memandang program sebagai **urutan langkah**:

```
1. Ambil data dari form
2. Validasi data
3. Simpan ke database
4. Kirim email
5. Redirect ke halaman sukses
```

Semua kode ditulis sebagai fungsi, data lewat parameter.

```php
<?php
// Gaya PROCEDURAL
$cart = getCartFromSession();
$total = calculateTotal($cart);
$order = createOrderInDatabase($cart, $total);
sendOrderConfirmationEmail($order);
redirectToSuccessPage();
```

**Masalah dengan procedural untuk aplikasi besar:**

1. **Tidak ada organisasi** — fungsi bertebaran, sulit tahu fungsi untuk apa
2. **Data dan fungsi terpisah** — `$cart` lewat dari fungsi ke fungsi, siapa pun bisa mengubahnya
3. **Sulit diperluas** — menambah fitur baru berarti mengubah fungsi yang sudah ada
4. **Sulit dites** — fungsi bergantung pada state global

### 1.2 Object-Oriented Programming

OOP memandang program sebagai **kumpulan objek** yang memiliki **data (state)** dan **perilaku (behavior)**:

```
Objek CartService:
  Data: sessionKey = 'shopping_cart'
  Perilaku: getCart(), add(), remove(), getTotal()

Objek Order:
  Data: status, total_amount, order_number
  Perilaku: getStatusLabel(), items(), payment()

Objek Mail:
  Perilaku: send(to, subject, body)
```

Setiap objek **tanggung jawab sendiri** dan **saling berkirim pesan** (method call):

```php
<?php
// Gaya OOP
$cart = $cartService->getCart();
$order = $orderService->placeOrder($cart, $data);
$order->user->notify(new OrderStatusChanged($order));
return redirect()->route('orders.show', $order);
```

### 1.3 Perbandingan Nyata: Cart Total

**Procedural:**
```php
function getCartTotal(array $cart): int
{
    $total = 0;
    foreach ($cart as $item) {
        $total += $item['price'] * $item['quantity'];
    }
    return $total;
}

// Dipanggil: $total = getCartTotal($cart);
```

**OOP (di codebase — CartService.php:86-91):**
```php
public function getTotal(): int
{
    $cart = $this->getCart();
    return array_reduce($cart, function ($total, $item) {
        return $total + ($item['price'] * $item['quantity']);
    }, 0);
}

// Dipanggil: $total = $cartService->getTotal();
```

Perbedaan kunci:
- Procedural: fungsi **terbang sendiri**, data lewat parameter
- OOP: method **milik objek**, data diambil dari internal objek (`$this->getCart()`)

---

## 2. Analogi Paling Penting: Cetakan Kue

Ini analogi yang akan melekat seumur hidupmu.

### 2.1 Class = Cetakan Kue

```
Class (Cetakan):
┌─────────────────────┐
│  Cetakan Kue        │
│  Bentuk: Bintang    │
│  Ukuran: 10cm       │
│  Bahan: Adonan      │
└─────────────────────┘
      │
      │ pakai cetakan → menghasilkan
      ▼
Object (Kue jadi):
┌─────────────────────┐
│  Kue Bintang #1     │ ← berbeda dengan #2
├─────────────────────┤
│  bentuk: bintang    │
│  ukuran: 10cm       │
│  warna: coklat      │
│  topping: keju      │
└─────────────────────┘
```

- **Class** adalah cetakan — mendefinisikan **struktur dan perilaku**
- **Object** adalah kue jadi — hasil dari cetakan, punya **data spesifik**

### 2.2 Class Product

Di codebase, `Product` adalah class — cetakan untuk produk:

```php
// app/Models/Product.php
class Product extends Model implements HasMedia
{
    // Ini "cetakan" — menentukan properti apa yang dimiliki setiap produk
}
```

Setiap baris di tabel `products` adalah **object/instance** dari class `Product`:

```php
$laptop  = Product::find(1); // instance 1 — nama: "Laptop Gaming"
$mouse   = Product::find(2); // instance 2 — nama: "Mouse Wireless"
$keyboard = Product::find(3); // instance 3 — nama: "Keyboard Mechanical"
```

Tiga object berbeda, tapi semuanya dari cetakan (class) yang sama.

### 2.3 State dan Behavior

Setiap objek punya dua hal:

**State (data/keadaan):** Properti yang membedakan satu instance dari instance lain.
```
$laptop->name  = "Laptop Gaming"
$laptop->price = 15000000
$laptop->stock_quantity = 10

$mouse->name  = "Mouse Wireless"
$mouse->price = 250000
$mouse->stock_quantity = 50
```

**Behavior (perilaku):** Method yang bisa dilakukan objek.
```php
$laptop->category();    // ambil kategori produk
$laptop->brand();       // ambil merek produk
$laptop->images();      // ambil gambar produk
```

---

## 3. Mengapa OOP Ada — Masalah yang Dipecahkan

### 3.1 Masalah 1: Organisasi Kode

**Tanpa OOP:** Semua fungsi dan data tercampur. Sulit mencari bagian yang perlu diubah.

**Dengan OOP:** Setiap class punya tanggung jawab jelas.
```
app/
├── Models/       → Cetakan data (Product, Order, User, ...)
├── Services/     → Logika bisnis (CartService, PaymentService, ...)
├── Http/
│   ├── Controllers/  → Penangan request HTTP
│   ├── Middleware/    → Filter sebelum controller
│   └── Requests/     → Validasi form
└── Providers/    → Konfigurasi framework
```

Ini disebut **separation of concerns** — setiap class urus satu hal.

### 3.2 Masalah 2: Duplikasi Kode

**Tanpa OOP:** Kamu menulis kode yang sama berulang kali.

**Dengan OOP:** Gunakan **inheritance** (pewarisan).

```php
// Semua controller punya akses ke helper yang sama
abstract class Controller
{
    // Helper methods yang bisa dipakai semua controller
}

// Tanpa menulis ulang, CheckoutController punya semua kemampuan Controller
class CheckoutController extends Controller { ... }
```

### 3.3 Masalah 3: Perubahan Berantai

**Tanpa OOP:** Mengubah satu fungsi bisa merusak 10 bagian lain.

**Dengan OOP: Encapsulation** — detail internal tersembunyi.

```php
class CartService
{
    protected $sessionKey = 'shopping_cart';

    public function getCart(): array
    {
        return Session::get($this->sessionKey, []);
    }
}
```

Kalau suatu saat kamu ganti penyimpanan cart dari session ke database:

```php
public function getCart(): array
{
    return CartModel::where('user_id', Auth::id())->get()->toArray();
}
```

**Tidak ada kode lain yang berubah!** Semua controller yang memanggil `$cartService->getCart()` tetap jalan. Encapsulation menyembunyikan perubahan.

### 3.4 Masalah 4: Testing

**Tanpa OOP:** Susah test fungsi yang bergantung pada database, session, atau API.

**Dengan OOP:** Dependency Injection — ganti dependency nyata dengan **mock** saat testing.

```php
// Di production:
$service = new OrderService(new CartService());

// Di test:
$mockCart = $this->createMock(CartService::class);
$service = new OrderService($mockCart); // ← testing jadi mudah
```

---

## 4. Empat Pilar OOP (Sekilas)

Empat konsep fundamental yang akan kita dalami di dokumen terpisah:

### 4.1 Encapsulation — Bungkus dan Sembunyikan

Menyembunyikan detail internal objek, hanya暴露 method yang aman.

```
Kamu tidak perlu tahu bagaimana Remote AC bekerja di dalam.
Kamu cukup tekan tombol ON → AC menyala.
              ↑                    ↑
           public method      internal complexity hidden
```

### 4.2 Inheritance — Turunkan Sifat

Class bisa mewarisi sifat dari class lain.

```
Model (induk)
├── Product ──→ punya $fillable, casts(), category()
├── Order   ──→ punya $fillable, casts(), items()
├── User    ──→ punya $fillable, casts(), roles()
└── Setting ──→ punya $fillable, casts(), get(), set()
```

Semua model mewarisi kemampuan dari `Model` (Eloquent ORM):
- `$model->save()` — menyimpan ke database
- `$model::find($id)` — mencari record
- `$model->where(...)` — filter query

### 4.3 Polymorphism — Banyak Bentuk

Objek berbeda bisa dipanggil dengan cara yang sama.

```php
// Semua model bisa dipanggil dengan cara yang sama
$product = Product::find(1);   // "cari di tabel products"
$user    = User::find(1);      // "cari di tabel users"
$order   = Order::find(1);     // "cari di tabel orders"

// Method find() bekerja berbeda untuk setiap class
// Tapi programmer memanggil dengan cara yang SAMA
```

### 4.4 Abstraction — Sederhanakan Kerumitan

Sembunyikan detail yang tidak perlu, tampilkan hanya yang relevan.

```php
// Kamu panggil ini:
$snapToken = $midtransService->getSnapToken($order);

// Tapi di dalamnya terjadi:
// 1. Baca konfigurasi Midtrans
// 2. Setup parameter
// 3. Panggil API Midtrans
// 4. Handle error
// 5. Simpan token ke database
// 6. Return token

// Kamu tidak perlu tahu #1-#6 — cukup panggil method-nya
```

---

## 5. OOP dalam Satu Paragraf

OOP adalah cara mengorganisir kode di mana:

1. **Class** = cetakan yang mendefinisikan **data** (properti) dan **perilaku** (method)
2. **Object** = instance dari class, punya data spesifik
3. **Encapsulation** = detail internal disembunyikan
4. **Inheritance** = class bisa mewarisi dari class lain
5. **Polymorphism** = interface yang sama, implementasi berbeda
6. **Abstraction** = kompleksitas disembunyikan di balik method sederhana

Di Laravel, **semuanya OOP**:
- Setiap model adalah class yang extend `Model`
- Setiap controller adalah class yang extend `Controller`
- Setiap service adalah class yang di-inject ke controller
- Request, response, middleware, queue — semua objek

---

## 6. Yang Akan Kamu Pelajari di Fase 3

```
Fase 3:
├── 03-01 APA ITU OOP               ← Kamu di sini
├── 03-02 CLASS OBJECT PROPERTY     → Class, instance, $this, properti, method
├── 03-03 INHERITANCE POLYMORPHISM  → extends, override, parent, polymorphic
├── 03-04 ENCAPSULATION ABSTRACTION → private/protected, interface, trait
├── 03-05 DEPENDENCY INJECTION      → Constructor DI, Laravel container
├── 03-06 OOP DI PHP               → PHP 8 OOP features
└── 03-07 OOP DI CODEBASE INI      → Tour semua OOP pattern di project ini
```

---

## 7. Bagaimana OOP Terlihat di Codebase — Sekilas

Kita sudah lihat beberapa contoh tanpa sadar. Perhatikan pola ini:

### Di Controller

```php
class CheckoutController extends Controller   // ← Inheritance
{
    protected $cartService;                     // ← Encapsulation

    public function __construct(                 // ← Dependency Injection
        CartService $cartService                 // ← Type hint (class sebagai tipe)
    ) {
        $this->cartService = $cartService;
    }

    public function index()
    {
        $cart = $this->cartService->getCart();  // ← Object communication
        return view('shop.checkout', compact('cart'));
    }
}
```

### Di Model

```php
class Product extends Model implements HasMedia  // ← Inheritance + Interface
{
    use SoftDeletes, InteractsWithMedia;          // ← Trait

    public function category()                    // ← Method
    {
        return $this->belongsTo(Category::class); // ← Polymorphism (relasi)
    }
}
```

### Di Service

```php
class RajaOngkirService
{
    private function getDummyProvinces(): array  // ← Encapsulation (private)
    {
        return [/* ... */];
    }

    public function getProvinces(): array         // ← Public API
    {
        if (empty($this->apiKey)) {
            return $this->getDummyProvinces();    // ← Panggil method sendiri
        }
        // ...
    }
}
```

---

## 8. Mindset: Objek = Pelayan (Servant)

Untuk memahami OOP, pikirkan setiap objek sebagai **pelayan pribadi**:

```
Kamu (programmer): "Hei CartService, berapa total belanjaan saya?"

CartService: "Tunggu sebentar, saya hitung dulu..."
             (mengambil data dari session, menjumlahkan, mengembalikan)

Kamu: "Terima kasih!"
```

Setiap pelayan punya **spesialisasi**:
- `CartService` — ahli keranjang belanja
- `PaymentService` — ahli pembayaran
- `OrderService` — ahli pemesanan
- `Product` — ahli data produk

Kamu tidak perlu tahu **bagaimana** mereka bekerja. Kamu hanya perlu tahu **perintah apa** yang bisa kamu berikan (method apa yang bisa dipanggil).

---

## 🧪 Latihan

1. **Identifikasi class dan object.** Buka `app/Models/Order.php`. Catat:
   - Nama class
   - Parent class (extends)
   - Properti (yang didefinisikan)
   - Method (fungsi dalam class)

2. **Buat class sederhana.** Di `php -a` atau file `.php`:

```php
<?php
class Kue
{
    public string $rasa;
    public string $topping;

    public function __construct(string $rasa, string $topping)
    {
        $this->rasa = $rasa;
        $this->topping = $topping;
    }

    public function describe(): string
    {
        return "Kue {$this->rasa} dengan topping {$this->topping}";
    }
}

$kue1 = new Kue('coklat', 'keju');
$kue2 = new Kue('vanila', 'stroberi');

echo $kue1->describe();
echo $kue2->describe();
```

3. **Refleksi.** Buka `app/Http/Controllers/CartController.php`. Identifikasi:
   - Class apa yang digunakan?
   - Properti apa yang dimiliki?
   - Method apa saja?
   - Bagaimana class ini "berbicara" dengan class lain?

4. **Cari perbedaan.** Buka file procedural (misal helper) vs OOP. Apa perbedaan strukturnya?

---

## 🔗 Referensi

- [PHP Manual: Classes and Objects](https://www.php.net/manual/en/language.oop5.php)
- [Wikipedia: Object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming)
- Codebase: `app/Models/Product.php` — contoh class + inheritance + interface + trait
- Codebase: `app/Http/Controllers/Controller.php` — abstract class
- Codebase: `app/Services/CartService.php` — service class murni
- Codebase: `app/Http/Controllers/CheckoutController.php` — controller dengan DI
