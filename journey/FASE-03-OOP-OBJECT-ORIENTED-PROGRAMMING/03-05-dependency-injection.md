# 03-05: Dependency Injection

> **Fase**: 3 — Object Oriented Programming  
> **Prasyarat**: 03-04-encapsulation-abstraction  
> **Waktu baca**: 70-85 menit  
> **Kata kunci**: dependency injection, DI, service container, IoC, inversion of control, constructor injection, setter injection, auto-resolution, binding

---

## 📋 Ringkasan

Dependency Injection (DI) adalah **teknik** di mana sebuah class menerima dependency (ketergantungan) dari **luar**, bukan membuatnya sendiri.

Laravel dibangun di atas DI. **Setiap controller, service, middleware, job — semuanya menggunakan DI.**

**Target pemahaman:**
- Kamu paham apa itu DI dan kenapa penting
- Kamu bisa membaca dan menulis constructor injection
- Kamu paham Laravel service container dan auto-resolution
- Kamu bisa melihat pola DI di seluruh codebase

---

## 1. Masalah: Tanpa DI

### 1.1 Tight Coupling (Keterikatan Kuat)

```php
<?php

class OrderController extends Controller
{
    public function store(Request $request)
    {
        // ❌ OrderController membuat dependency-nya sendiri
        $cartService = new CartService();
        $orderService = new OrderService();

        $cart = $cartService->getCart();
        $order = $orderService->placeOrder($cart, $request->all());

        return redirect()->route('orders.show', $order);
    }
}
```

**Masalah:**
1. **Tight coupling** — `OrderController` terikat ke `CartService` dan `OrderService` secara langsung
2. **Sulit ditest** — tidak bisa mengganti `CartService` dengan mock
3. **Tidak fleksibel** — kalau constructor `CartService` berubah, semua tempat yang `new CartService()` harus diubah

### 1.2 Masalah Berantai

```php
<?php
// Kalau CartService butuh parameter baru:
class CartService
{
    public function __construct(private string $sessionKey) {}
}

// Maka semua kode yang new CartService() harus diubah:
$cartService = new CartService('shopping_cart'); // ubah di sini
$cartService2 = new CartService('shopping_cart'); // ubah di sini juga
$cartService3 = new CartService('shopping_cart'); // dan di sini
```

---

## 2. Solusi: Dependency Injection

### 2.1 Constructor Injection

Dependency diberikan dari luar via constructor.

```php
<?php

class OrderService
{
    // Dependency diterima dari luar, bukan dibuat sendiri
    public function __construct(
        private CartService $cartService
    ) {}
}

class CheckoutController extends Controller
{
    // Semua dependency diterima dari luar
    public function __construct(
        private CartService $cartService,
        private AddressService $addressService,
        private OrderService $orderService,
        private RajaOngkirService $rajaOngkirService
    ) {}

    public function process(Request $request)
    {
        // Tinggal pakai, tinggal panggil
        $cart = $this->cartService->getCart();
        $order = $this->orderService->placeOrder($cart, $request->all());
        // ...
    }
}
```

**Siapa yang membuat semua dependency ini?** Laravel service container.

```php
// Di belakang layar, Laravel melakukan:
$controller = new CheckoutController(
    new CartService(),
    new AddressService(),
    new OrderService(new CartService()), // OrderService juga butuh CartService!
    new RajaOngkirService()
);
```

### 2.2 Setter Injection (Alternatif)

```php
<?php

class PaymentService
{
    private ?Logger $logger = null;

    // Setter injection — dependency di-set via method
    public function setLogger(Logger $logger): void
    {
        $this->logger = $logger;
    }
}
```

**Kapan pakai setter injection:**
- Dependency **opsional** (class bisa jalan tanpa)
- Ada **circular dependency** (jarang terjadi)

**Di codebase ini:** Tidak ada setter injection. Semua DI menggunakan constructor injection.

### 2.3 Interface Injection

```php
<?php

interface LoggerAware
{
    public function setLogger(Logger $logger): void;
}

class PaymentService implements LoggerAware
{
    public function setLogger(Logger $logger): void { /* ... */ }
}
```

Jarang dipakai di PHP. Lebih umum di Java.

---

## 3. Laravel Service Container

### 3.1 Apa Itu Service Container

Service container adalah "wadah" yang:
1. Mengetahui **cara membuat** semua class di aplikasi
2. Menyimpan **instance** (singleton) jika diperlukan
3. **Auto-resolve** dependency secara rekursif

```
Service Container:
┌──────────────────────────────────────┐
│  Class → Cara Membuat                 │
├──────────────────────────────────────┤
│  CartService      → new CartService() │
│  OrderService     → new OrderService( │
│                       CartService)    │
│  RajaOngkirService → new RajaOngkir()│
│  CheckoutController → new Checkout(  │
│                        CartService,  │
│                        AddressService,│
│                        OrderService,  │
│                        RajaOngkir)   │
└──────────────────────────────────────┘
```

### 3.2 Auto-Resolution

Saat Laravel perlu membuat `CheckoutController`, container:
1. Lihat constructor: `__construct(CartService $cs, AddressService $as, OrderService $os, RajaOngkirService $ro)`
2. Cari `CartService` → buat `new CartService()`
3. Cari `AddressService` → buat `new AddressService()`
4. Cari `OrderService` → lihat constructor `__construct(CartService $cs)`
5. Buat `CartService` (sekali lagi — atau pakai instance yang sama)
6. Buat `OrderService` dengan `CartService` tadi
7. Cari `RajaOngkirService` → buat `new RajaOngkirService()`
8. Buat `CheckoutController` dengan semua dependency

**Semua otomatis.** Kamu tidak perlu konfigurasi apa pun.

### 3.3 Explicit Binding

Untuk interface atau class yang butuh konfigurasi khusus:

```php
<?php
// Di AppServiceProvider.php atau bootstrap/app.php

// Interface → implementation
$app->bind(PaymentGateway::class, MidtransGateway::class);

// Singleton (instance yang sama setiap kali)
$app->singleton(CartService::class, function () {
    return new CartService();
});

// Instance yang sudah ada
$app->instance('key', $someObject);
```

**Di codebase — bootstrap/app.php:14-16:**
```php
$middleware->alias([
    'role' => \App\Http\Middleware\RoleMiddleware::class,
]);
```

Ini mendaftarkan middleware alias, bukan binding. Binding default (tanpa interface) tidak perlu explicit — container auto-resolve.

### 3.4 `app()` Helper

Untuk resolve class secara manual:

```php
<?php
// Via helper:
$service = app(PaymentService::class);
$service->confirmPayment($payment);

// Via facade:
$service = App::make(PaymentService::class);

// Via method injection di closure:
Route::get('/test', function (CartService $cartService) {
    return $cartService->getTotal();
});
```

**Di codebase — PaymentController.php:61:**
```php
app(\App\Services\PaymentService::class)->confirmPayment($payment);
```

Mengapa tidak pakai constructor injection di sini? Karena `confirm()` adalah method spesifik yang jarang dipakai — tidak perlu membebani constructor semua controller.

---

## 4. DI Pattern di Codebase

### 4.1 Controller → Service (Pola Paling Umum)

```php
class PaymentController extends Controller
{
    public function __construct(
        private MidtransService $midtransService   // DI via constructor
    ) {}

    public function showConfirmation(Order $order)
    {
        $snapToken = $this->midtransService->getSnapToken($order);
        return view('payments.confirmation', compact('snapToken', 'order'));
    }
}
```

### 4.2 Service → Service

Service juga bisa inject service lain:

```php
// OrderService.php
class OrderService
{
    public function __construct(
        private CartService $cartService  // OrderService butuh CartService
    ) {}

    public function placeOrder(array $data)
    {
        $cart = $this->cartService->getCart(); // pakai service lain
        // ...
    }
}
```

### 4.3 Controller → Model (Route Model Binding)

Ini bukan DI di constructor, tapi DI di method (method injection):

```php
// OrderController.php
public function show(Order $order) // Order $order di-inject oleh Laravel
{
    // Laravel otomatis cari Order::find($id) dari route parameter
    $this->authorize('view', $order);
    return view('orders.show', compact('order'));
}
```

### 4.4 Closure Route Injection

```php
Route::get('/test', function (CartService $cartService) {
    // Closure bisa inject juga!
    return $cartService->getTotal();
});
```

### 4.5 Queue Job Injection

```php
class ProcessOrder implements ShouldQueue
{
    public function __construct(
        private Order $order
    ) {}

    public function handle(
        PaymentService $paymentService // DI di method handle
    ) {
        $paymentService->confirmPayment($this->order->payment);
    }
}
```

---

## 5. Inversion of Control (IoC)

### 5.1 Definisi

**Inversion of Control** adalah prinsip di balik DI: alihkan kontrol pembuatan dependency dari class ke container.

```
Tanpa IoC:
  Class A → membuat Class B sendiri → kontrol ada di A

Dengan IoC (DI):
  Container → membuat Class A dan B → kontrol ada di container
  Container → inject Class B ke Class A
```

### 5.2 Analogi

```
Tanpa DI (kamu buat sendiri):
  Kamu: "Saya mau kopi."
  Kamu: (beli biji kopi, sangrai, giling, seduh) → capek sendiri

Dengan DI (ada barista):
  Kamu: "Saya mau kopi."
  Barista: (buatin kopi) → kamu tinggal nikmati
```

`CheckoutController` adalah kamu — dia tinggal minta `$cartService` ke container, container yang buatin.

### 5.3 Keuntungan

1. **Decoupling** — class tidak tergantung implementasi konkret
2. **Testability** — dependency bisa diganti mock di test
3. **Flexibility** — ubah implementasi tanpa ubah pengguna
4. **Reusability** — service bisa dipakai banyak controller

---

## 6. DI vs Service Locator

### 6.1 Service Locator (Kurang Baik)

```php
<?php

class OrderController extends Controller
{
    public function store(Request $request)
    {
        // Service locator — minta langsung ke container di dalam method
        $cartService = app(CartService::class);
        $orderService = app(OrderService::class);

        $order = $orderService->placeOrder(
            $cartService->getCart(),
            $request->all()
        );
    }
}
```

**Kekurangan:**
- Dependency **tidak terlihat** — lihat signature method tidak tahu butuh apa
- Sulit dilacak — `app()` bisa dipanggil di mana pun
- Testing lebih susah

### 6.2 Constructor Injection (Lebih Baik)

```php
<?php

class OrderController extends Controller
{
    public function __construct(
        private CartService $cartService,
        private OrderService $orderService
    ) {}

    public function store(Request $request)
    {
        $order = $this->orderService->placeOrder(
            $this->cartService->getCart(),
            $request->all()
        );
    }
}
```

**Keunggulan:**
- Dependency **jelas** — lihat constructor langsung tahu
- Container auto-resolve
- Mudah ditest — cukup inject mock

### 6.3 Pengecualian: Kadang `app()` OK

`app()` di dalam method **kadang** diperlukan:

```php
// PaymentController.php
public function confirm(int $paymentId)
{
    $payment = Payment::findOrFail($paymentId);

    // app() di sini karena PaymentService jarang dipakai
    // Tidak perlu di-inject di constructor semua controller
    app(PaymentService::class)->confirmPayment($payment);

    return redirect()->back()->with('success', 'Payment confirmed!');
}
```

Tapi sebagai aturan: **prioritaskan constructor injection.**

---

## 7. Testing dengan DI

Keuntungan terbesar DI adalah **testing**. Kamu bisa mengganti dependency nyata dengan **mock**:

```php
<?php
// Di test:
public function test_order_creation()
{
    // Buat mock CartService
    $mockCart = $this->createMock(CartService::class);
    $mockCart->method('getCart')->willReturn([
        ['id' => 1, 'price' => 10000, 'quantity' => 2]
    ]);
    $mockCart->method('getTotal')->willReturn(20000);
    $mockCart->method('getTotalWeight')->willReturn(500);

    // Inject mock ke OrderService
    $orderService = new OrderService($mockCart);

    // Test — tidak perlu session beneran
    $order = $orderService->placeOrder([...]);

    $this->assertInstanceOf(Order::class, $order);
}
```

**Tanpa DI:** Testing `OrderService` berarti testing `CartService` juga — dan butuh session, butuh database. Sangat berat.

---

## 8. Ringkasan Visual

```
┌──────────────────────────────────────────────────────────┐
│ DEPENDENCY INJECTION                                      │
│                                                          │
│  ❌ Tanpa DI:                                            │
│  class A {                                               │
│      public function doSomething() {                     │
│          $b = new B();    // A membuat B sendiri         │
│          $b->run();                                      │
│      }                                                   │
│  }                                                       │
│                                                          │
│  ✅ Dengan DI:                                           │
│  class A {                                               │
│      public function __construct(                        │
│          private B $b       // B di-inject dari luar    │
│      ) {}                                                │
│                                                          │
│      public function doSomething() {                     │
│          $this->b->run();   // A tinggal pakai          │
│      }                                                   │
│  }                                                       │
│                                                          │
│  🎯 Laravel Service Container:                            │
│  - Lihat constructor type hint                           │
│  - Resolve semua dependency secara rekursif              │
│  - Buat instance → inject → return                      │
├──────────────────────────────────────────────────────────┤
│ DI DI CODEBASE:                                          │
│                                                          │
│  Constructor DI:      Setiap controller + beberapa service│
│  Method Injection:    $request, $order (route model)     │
│  app() helper:        PaymentController@confirm          │
│  Closure injection:   Route closure                      │
└──────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Identifikasi DI di codebase.** Buka `app/Http/Controllers/Admin/ProductController.php`. Service apa saja yang di-inject? Bagaimana cara inject-nya?

2. **Buat class dengan DI.** Buat class `OrderReport` yang membutuhkan `OrderService` dan `CartService` di constructor. Tulis method `getDailyReport(): array`.

3. **Route injection.** Buat route `/test-cart` yang inject `CartService` dan mengembalikkan total cart sebagai JSON:
```php
Route::get('/test-cart', function (CartService $cartService) {
    return response()->json(['total' => $cartService->getTotal()]);
});
```

4. **Analisis `app/Http/Controllers/CheckoutController.php`:**
   - Ada berapa dependency yang di-inject?
   - Dari mana dependency itu berasal?
   - Apa yang terjadi jika `CartService` juga punya constructor dengan dependency?

5. **Practice with Tinker:**
```bash
php artisan tinker
>>> $cartService = app(CartService::class)
>>> $cartService->getCart()
>>> app(OrderService::class)
```

---

## 🔗 Referensi

- [Laravel Docs: Service Container](https://laravel.com/docs/11.x/container)
- [PHP Manual: Dependency Injection](https://www.php.net/manual/en/language.oop5.decon.php)
- [Martin Fowler: Inversion of Control](https://martinfowler.com/articles/injection.html)
- Codebase: `app/Http/Controllers/CheckoutController.php` — constructor DI
- Codebase: `app/Http/Controllers/PaymentController.php` — constructor DI + app()
- Codebase: `app/Services/OrderService.php` — service-to-service DI
- Codebase: `app/Http/Controllers/Admin/ProductController.php` — multiple DI
