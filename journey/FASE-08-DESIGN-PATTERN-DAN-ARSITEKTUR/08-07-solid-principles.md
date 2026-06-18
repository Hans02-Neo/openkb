# 08-07: SOLID Principles — 5 Prinsip OOP

> **Fase**: 8 — Design Pattern & Arsitektur  
> **Prasyarat**: 08-06-policy-dan-gate  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: SOLID, SRP, OCP, LSP, ISP, DIP, clean code, architecture

---

## 📋 Ringkasan

SOLID adalah **5 prinsip desain OOP** yang membuat kode mudah dipelihara, diuji, dan dikembangkan. Setiap huruf mewakili satu prinsip.

**Target pemahaman:**
- Kamu paham 5 prinsip SOLID
- Kamu bisa mengidentifikasi pelanggaran SOLID
- Kamu bisa menerapkan SOLID di Laravel
- Kamu bisa melihat SOLID di codebase ini

---

## 1. S — Single Responsibility Principle (SRP)

> **Satu kelas = satu alasan untuk berubah.**

### 1.1 Pelanggaran SRP

```php
// ❌ Class ini punya 3 tanggung jawab:
class OrderController extends Controller
{
    public function store(Request $request)
    {
        // 1. Validasi
        $request->validate([...]);

        // 2. Logika bisnis (create order + kurangi stok)
        $order = Order::create($request->all());
        foreach ($order->items as $item) {
            $item->product->decrement('stock', $item->quantity);
        }

        // 3. Kirim email
        Mail::to($order->user)->send(new OrderConfirmation($order));

        return redirect()->route('orders.show', $order);
    }
}
```

### 1.2 Sesuai SRP

```php
// Controller: handle HTTP request
class OrderController extends Controller
{
    public function __construct(private OrderService $orderService) {}

    public function store(CheckoutRequest $request): RedirectResponse
    {
        // Delegasi ke service
        $this->orderService->createOrder($request->validated());

        return redirect()->route('orders.show', $order);
    }
}

// Service: business logic
class OrderService
{
    public function createOrder(array $data): Order { ... }
}

// FormRequest: validasi
class CheckoutRequest extends FormRequest
{
    public function rules(): array { return [...]; }
}

// Listener: kirim email (dipicu event)
class SendOrderConfirmation implements ShouldQueue
{
    public function handle(OrderCreated $event): void { ... }
}
```

### 1.3 SRP di Codebase

| File | Tanggung Jawab |
|------|---------------|
| `OrderController` | HTTP handling (request → response) |
| `OrderService` | Logika bisnis order |
| `RajaOngkirService` | Integrasi API ongkir |
| `CartService` | Operasi cart |
| `OrderPolicy` | Authorization |
| `OrderObserver` | Catat history status |
| `Order` (Model) | Data + relasi order |

---

## 2. O — Open/Closed Principle (OCP)

> **Terbuka untuk ekstensi, tertutup untuk modifikasi.**

### 2.1 Pelanggaran OCP

```php
// ❌ Setiap kali ada payment baru, class ini harus diubah:
class PaymentService
{
    public function process(Order $order, string $method): void
    {
        if ($method === 'midtrans') {
            // panggil Midtrans API
        } elseif ($method === 'stripe') {
            // panggil Stripe API
        } elseif ($method === 'cod') {
            // proses COD
        }
        // Tambah elseif setiap ada payment baru!
    }
}
```

### 2.2 Sesuai OCP

```php
// Interface — contract
interface PaymentGateway
{
    public function charge(Order $order, array $data): PaymentResult;
}

// Implementation — tinggal tambah class baru
class MidtransGateway implements PaymentGateway
{
    public function charge(Order $order, array $data): PaymentResult { ... }
}

class StripeGateway implements PaymentGateway
{
    public function charge(Order $order, array $data): PaymentResult { ... }
}

class CashOnDeliveryGateway implements PaymentGateway
{
    public function charge(Order $order, array $data): PaymentResult { ... }
}

// Service — tidak perlu diubah
class PaymentService
{
    public function __construct(
        private PaymentGateway $gateway  // injected dari luar
    ) {}

    public function process(Order $order, array $data): PaymentResult
    {
        return $this->gateway->charge($order, $data);
    }
}
```

### 2.3 OCP di Codebase

Contoh OCP di codebase: **Service Layer**.

```php
// CartService bisa dipanggil dari mana saja:
// - CartController@addItem → CartService@addItem
// - OrderService@createOrder → CartService@clear
// - Artisan command → CartService@cleanup
// Tanpa mengubah CartService itu sendiri.
```

---

## 3. L — Liskov Substitution Principle (LSP)

> **Subclass harus bisa menggantikan parent class-nya.**

### 3.1 Pelanggaran LSP

```php
// ❌ Subclass mengubah behavior parent
abstract class Discount
{
    abstract public function apply(int $total): int;
}

class NoDiscount extends Discount
{
    public function apply(int $total): int
    {
        return $total; // OK — tidak ngubah
    }
}

class BuyOneGetOne extends Discount
{
    public function apply(int $total): int
    {
        // ❌ Bukan return int, tapi return array!
        return ['total' => $total, 'free' => true];
    }
}

// Kalau code期待 int, BOGO akan break.
```

### 3.2 LSP di Laravel

LSP banyak diterapkan oleh **interface/contract** Laravel:

```php
// Interface → semua implementasi bisa saling ganti
use Illuminate\Contracts\Mail\Mailer;

class NotificationService
{
    public function __construct(private Mailer $mailer) {}

    public function send(): void
    {
        // $mailer bisa Mail::mailer('smtp')
        // atau Mail::mailer('ses')
        // atau Mail::mailer('log')
        // Semua implementasi Mailer bisa gantian — LSP!
        $this->mailer->send(...);
    }
}
```

### 3.3 LSP di Codebase

```php
// Policy: method view selalu return bool
class OrderPolicy {
    public function view(User $user, Order $order): bool { ... }
}
class AddressPolicy {
    public function view(User $user, Address $address): bool { ... }
}
// Kedua policy bisa dipanggil via $this->authorize('view', $model)
// Karena signature-nya konsisten — LSP!
```

---

## 4. I — Interface Segregation Principle (ISP)

> **Jangan paksa class implement method yang tidak dibutuhkan.**

### 4.1 Pelanggaran ISP

```php
// ❌ Interface terlalu gemuk
interface PaymentGateway
{
    public function charge(Order $order, array $data): PaymentResult;
    public function refund(Order $order): PaymentResult;
    public function getTransactionStatus(string $id): string;
    public function createSubscription(Plan $plan): Subscription;
    public function cancelSubscription(string $id): void;
}

// COD cuma butuh charge, dipaksa implement refund, subscription, dll
class CashOnDelivery implements PaymentGateway
{
    public function charge(...) { return new PaymentResult(...); }
    public function refund(...) { throw new \Exception('COD tidak bisa refund'); }
    public function getTransactionStatus(...) { throw ...; }
    public function createSubscription(...) { throw ...; }
    public function cancelSubscription(...) { throw ...; }
}
```

### 4.2 Sesuai ISP

```php
// ✅ Interface kecil — per domain
interface Chargeable
{
    public function charge(Order $order, array $data): PaymentResult;
}

interface Refundable
{
    public function refund(Order $order): PaymentResult;
}

interface Subscribable
{
    public function createSubscription(Plan $plan): Subscription;
    public function cancelSubscription(string $id): void;
}

// COD hanya implement yang dibutuhkan:
class CashOnDelivery implements Chargeable
{
    public function charge(Order $order, array $data): PaymentResult { ... }
}

// Midtrans implement semua:
class MidtransGateway implements Chargeable, Refundable, Subscribable
{
    public function charge(...) { ... }
    public function refund(...) { ... }
    public function createSubscription(...) { ... }
    public function cancelSubscription(...) { ... }
}
```

### 4.3 ISP di Codebase

```php
// Service: interface implisit (method spesifik)
interface CartServiceInterface {
    public function addItem(int $productId, int $qty): void;
    public function removeItem(int $productId): void;
    public function getTotal(): int;
    public function clear(): void;
}
// Tidak ada method yang tidak relevan — ISP!
```

---

## 5. D — Dependency Inversion Principle (DIP)

> **Abstraksi jangan bergantung pada detail. Detail bergantung pada abstraksi.**

### 5.1 Pelanggaran DIP

```php
// ❌ High-level module bergantung ke low-level module
class OrderService
{
    private MidtransService $midtrans;
    private RajaOngkirService $rajaOngkir;
    private MailService $mail;

    // OrderService tergantung pada concrete class
    // Ganti Midtrans → Stripe? Ubah semua!
}
```

### 5.2 Sesuai DIP

```php
// ✅ Bergantung pada interface (abstraksi)
interface PaymentGateway
{
    public function charge(Order $order, array $data): PaymentResult;
}

interface ShippingService
{
    public function calculateCost(string $city, int $weight): int;
}

interface NotificationService
{
    public function send(Notifiable $notifiable, Notification $notification): void;
}

class OrderService
{
    public function __construct(
        private PaymentGateway $paymentGateway,       // abstraksi
        private ShippingService $shippingService,      // abstraksi
        private NotificationService $notificationService, // abstraksi
    ) {}

    // Ganti implementasi kapan saja — cukup inject berbeda
}
```

### 5.3 DIP di Laravel

**Service Container** adalah implementasi DIP:

```php
// Binding interface ke implementation
// app/Providers/AppServiceProvider.php
$this->app->bind(PaymentGateway::class, MidtransGateway::class);

// Laravel auto-inject interface → concrete
class OrderController extends Controller
{
    public function __construct(
        private OrderService $orderService
    ) {}
}
```

### 5.4 DIP di Codebase

```php
// Constructor injection — Laravel container yang handle
class CheckoutController extends Controller
{
    public function __construct(
        private CartService $cartService,
        private OrderService $orderService,
        private RajaOngkirService $rajaOngkirService,
        private MidtransService $midtransService,
    ) {}
}
```

---

## 6. SOLID — Tabel Ringkasan

| Prinsip | Arti | Laravel | Codebase Ini |
|---------|------|---------|-------------|
| **S**RP | Satu tanggung jawab | Controller tipis, Service layer | Service, Controller, Policy terpisah |
| **O**CP | Buka untuk ekstensi | Interface, ServiceProvider | Strategy via service injection |
| **L**SP | Substitusi aman | Interface/Contract | Policy, Mailer, Queue |
| **I**SP | Interface kecil | Queued, ShouldQueue | Service per domain |
| **D**IP | Abstraksi > detail | Service Container | Constructor injection |

---

## 7. Contoh SOLID Lengkap

```php
// SRP: setiap class satu tanggung jawab
class CheckoutRequest extends FormRequest { /* validasi */ }
class CheckoutController extends Controller { /* HTTP */ }
class OrderService { /* bisnis logic */ }
class MidtransGateway { /* payment */ }
class SendOrderConfirmation { /* email */ }

// OCP: tambah payment tanpa ubah OrderService
interface PaymentGateway { ... }
class MidtransGateway implements PaymentGateway { ... }
// nanti: class StripeGateway implements PaymentGateway { ... }

// LSP: semua gateway bisa gantian
$gateway = new MidtransGateway(); // atau new StripeGateway();
$orderService = new OrderService($gateway); // sama aja

// ISP: interface kecil
interface Chargeable { ... }
interface Refundable { ... }

// DIP: OrderService terima abstraksi, bukan konkrit
class OrderService {
    public function __construct(
        private PaymentGateway $gateway  // interface, bukan Midtrans
    ) {}
}
```

---

## 🧪 Latihan

1. **Identifikasi SRP.** Buka `app/Http/Controllers/CheckoutController.php`. Apakah ada pelanggaran SRP? Logika apa yang seharusnya dipindah?

2. **OCP.** Rancang sistem diskon dengan OCP:
   - Interface `Discount`
   - Implementasi: `PercentageDiscount`, `FixedDiscount`, `BuyXGetY`
   - `OrderService` menerima array `Discount`

3. **ISP.** Apakah `app/Services/CartService.php` punya method yang tidak kohesif? Jika ya, pisahkan.

4. **DIP.** Cari di codebase untuk constructor injection yang menunjukkan DIP. Catat 3 contoh.

5. **Evaluasi.** Pilih satu file controller. Evaluasi apakah sudah mengikuti SOLID. Apa yang perlu diperbaiki?

---

## 🔗 Referensi

- [Wikipedia: SOLID](https://en.wikipedia.org/wiki/SOLID)
- [Laravel: Service Container](https://laravel.com/docs/11.x/container)
- [Laravel: Contracts](https://laravel.com/docs/11.x/contracts)
- [Refactoring Guru: SOLID](https://refactoring.guru/design-patterns/solid)
- Seluruh file di `app/` — cari pattern SOLID di setiap file
