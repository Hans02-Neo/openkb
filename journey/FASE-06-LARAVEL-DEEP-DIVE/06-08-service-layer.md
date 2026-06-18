# 06-08: Service Layer — Logika Bisnis yang Rapi

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-07-middleware-dan-auth  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: service layer, business logic, repository, thin controller, fat model, separation of concerns

---

## 📋 Ringkasan

Controller jangan gemuk, Model jangan gemuk. Lalu logika bisnis taruh di mana? **Service Layer** — lapisan terpisah yang berisi **logika bisnis** aplikasi.

**Target pemahaman:**
- Kamu paham masalah fat controller dan fat model
- Kamu bisa membuat dan menggunakan service
- Kamu paham dependency injection antar service
- Kamu bisa membaca service di codebase

---

## 1. Masalah: Fat Controller

### 1.1 Controller yang Gemuk

```php
// ❌ Fat Controller — semua logika di sini
class CheckoutController extends Controller
{
    public function process(Request $request)
    {
        // Validasi manual
        $request->validate([...]);

        // Ambil cart dari session
        $cart = session('cart', []);

        // Hitung total
        $total = 0;
        foreach ($cart as $item) {
            $product = Product::find($item['product_id']);
            $total += $product->price * $item['quantity'];
        }

        // Cek stok
        foreach ($cart as $item) {
            $product = Product::find($item['product_id']);
            if ($product->stock_quantity < $item['quantity']) {
                return back()->with('error', 'Stok tidak cukup');
            }
        }

        // Buat order
        $order = Order::create([
            'user_id' => auth()->id(),
            'total_amount' => $total,
            'status' => 'pending',
        ]);

        // Buat order items
        foreach ($cart as $item) {
            OrderItem::create([
                'order_id' => $order->id,
                'product_id' => $item['product_id'],
                'quantity' => $item['quantity'],
                'price' => Product::find($item['product_id'])->price,
            ]);
        }

        // Kurangi stok
        foreach ($cart as $item) {
            $product = Product::find($item['product_id']);
            $product->decrement('stock_quantity', $item['quantity']);
        }

        // Proses payment
        // ... 50 baris lagi

        // Kirim email
        // ... 20 baris lagi

        // Hapus cart
        session()->forget('cart');

        return redirect()->route('orders.show', $order)
            ->with('success', 'Pesanan berhasil!');
    }
}
```

**Masalah:**
- 80+ baris di satu method
- Tidak bisa di-test terpisah
- Tidak bisa dipakai ulang
- Melanggar Single Responsibility Principle

### 1.2 Solusi: Service Layer

```php
// ✅ Thin Controller — delegasi ke service
class CheckoutController extends Controller
{
    public function __construct(
        private OrderService $orderService,
    ) {}

    public function process(CheckoutRequest $request): RedirectResponse
    {
        $order = $this->orderService->createOrder(
            $request->validated(),
            auth()->user()
        );

        return redirect()->route('orders.show', $order)
            ->with('success', 'Pesanan berhasil!');
    }
}
```

---

## 2. Service Layer — Apa dan Kenapa

### 2.1 Definisi

Service Layer adalah **class biasa** (bukan Controller, bukan Model) yang berisi **logika bisnis** aplikasi.

```
Controller (tugas: handle request/response)
    │
    ▼
Service (tugas: logika bisnis)
    │
    ▼
Model (tugas: data / database queries)
```

### 2.2 Keuntungan Service Layer

| | Tanpa Service | Dengan Service |
|---|-------------|---------------|
| **Controller** | 80+ baris | 5 baris |
| **Testability** | Butuh HTTP | Unit test langsung |
| **Reusability** | Copy-paste | Panggil service | 
| **Complexity** | Satu method besar | Method kecil per operasi |
| **Separation** | Campur aduk | Tiap class punya satu tanggung jawab |

---

## 3. Service di Codebase

### 3.1 CartService

```php
<?php
// app/Services/CartService.php
namespace App\Services;

class CartService
{
    public function __construct(
        private CustomerService $customerService,
    ) {}

    // Cart berbasis session
    public function getCart(): array
    {
        return session('cart', []);
    }

    public function addItem(int $productId, int $quantity = 1): void
    {
        $cart = $this->getCart();

        if (isset($cart[$productId])) {
            $cart[$productId]['quantity'] += $quantity;
        } else {
            $product = Product::findOrFail($productId);
            $cart[$productId] = [
                'product_id' => $productId,
                'name' => $product->name,
                'price' => $product->price,
                'weight_grams' => $product->weight_grams,
                'quantity' => $quantity,
            ];
        }

        session(['cart' => $cart]);
    }

    public function updateItem(int $productId, int $quantity): void
    {
        $cart = $this->getCart();
        if (isset($cart[$productId])) {
            $cart[$productId]['quantity'] = max(1, $quantity);
        }
        session(['cart' => $cart]);
    }

    public function removeItem(int $productId): void
    {
        $cart = $this->getCart();
        unset($cart[$productId]);
        session(['cart' => $cart]);
    }

    public function getTotal(): int
    {
        return array_reduce($this->getCart(), function ($total, $item) {
            return $total + ($item['price'] * $item['quantity']);
        }, 0);
    }

    public function getCount(): int
    {
        return array_reduce($this->getCart(), function ($count, $item) {
            return $count + $item['quantity'];
        }, 0);
    }

    public function getWeight(): int
    {
        return array_reduce($this->getCart(), function ($weight, $item) {
            return $weight + ($item['weight_grams'] * $item['quantity']);
        }, 0);
    }

    public function clear(): void
    {
        session()->forget('cart');
    }
}
```

### 3.2 OrderService

```php
<?php
// app/Services/OrderService.php
namespace App\Services;

use App\Models\Order;
use App\Models\OrderItem;
use App\Models\Product;
use Illuminate\Support\Facades\DB;

class OrderService
{
    public function __construct(
        private CartService $cartService,
        private MidtransService $midtransService,
        private RajaOngkirService $rajaOngkirService,
    ) {}

    public function createOrder(array $data, User $user): Order
    {
        return DB::transaction(function () use ($data, $user) {
            $cart = $this->cartService->getCart();

            // Validasi stok
            $this->validateStock($cart);

            // Hitung total
            $total = $this->cartService->getTotal();
            $weight = $this->cartService->getWeight();

            // Hitung ongkir
            $shippingCost = $this->rajaOngkirService->getShippingCost(
                $data['shipping_city'],
                $weight
            );

            // Create order
            $order = Order::create([
                'user_id' => $user->id,
                'order_number' => 'ORD-' . strtoupper(uniqid()),
                'total_amount' => $total + $shippingCost,
                'shipping_cost' => $shippingCost,
                'status' => 'pending',
                'shipping_address' => $data['shipping_address'],
                'shipping_city' => $data['shipping_city'],
                'shipping_postal_code' => $data['shipping_postal_code'],
                'shipping_phone' => $data['shipping_phone'],
            ]);

            // Create order items
            foreach ($cart as $item) {
                OrderItem::create([
                    'order_id' => $order->id,
                    'product_id' => $item['product_id'],
                    'product_name' => $item['name'],
                    'quantity' => $item['quantity'],
                    'price' => $item['price'],
                    'subtotal' => $item['price'] * $item['quantity'],
                ]);

                // Kurangi stok
                Product::find($item['product_id'])
                    ->decrement('stock_quantity', $item['quantity']);
            }

            // Proses payment
            $this->midtransService->processPayment($order);

            // Clear cart
            $this->cartService->clear();

            return $order;
        });
    }

    private function validateStock(array $cart): void
    {
        foreach ($cart as $item) {
            $product = Product::findOrFail($item['product_id']);
            if ($product->stock_quantity < $item['quantity']) {
                throw new \RuntimeException(
                    "Stok {$product->name} tidak mencukupi"
                );
            }
        }
    }
}
```

### 3.3 RajaOngkirService

```php
<?php
// app/Services/RajaOngkirService.php
namespace App\Services;

use Illuminate\Support\Facades\Http;

class RajaOngkirService
{
    private string $baseUrl;
    private string $apiKey;

    public function __construct()
    {
        $this->baseUrl = config('services.rajaongkir.base_url');
        $this->apiKey = config('services.rajaongkir.key');
    }

    public function getProvinces(): array
    {
        $response = Http::withHeaders([
            'key' => $this->apiKey,
        ])->get($this->baseUrl . '/starter/province');

        return $response->json()['rajaongkir']['results'] ?? [];
    }

    public function getDummyCities(?string $provinceId = null): array
    {
        $allCities = [
            ['city_id' => '39', 'city_name' => 'Bandung', 'province_id' => '5'],
            ['city_id' => '418', 'city_name' => 'Jakarta Pusat', 'province_id' => '5'],
            ['city_id' => '152', 'city_name' => 'Surabaya', 'province_id' => '6'],
            ['city_id' => '153', 'city_name' => 'Malang', 'province_id' => '6'],
            ['city_id' => '114', 'city_name' => 'Medan', 'province_id' => '1'],
        ];

        if ($provinceId) {
            return array_values(array_filter($allCities,
                fn($city) => $city['province_id'] == $provinceId
            ));
        }

        return $allCities;
    }

    public function getShippingCost(string $destination, int $weightGrams): int
    {
        // Dummy shipping cost
        return $weightGrams > 1000 ? 25000 : 10000;
    }
}
```

### 3.4 MidtransService

```php
<?php
// app/Services/MidtransService.php
namespace App\Services;

use App\Models\Order;

class MidtransService
{
    private string $serverKey;
    private string $merchantId;

    public function __construct()
    {
        $this->serverKey = config('services.midtrans.server_key');
        $this->merchantId = config('services.midtrans.merchant_id');
    }

    public function processPayment(Order $order): array
    {
        // Dummy implementation — real code would call Midtrans API
        return [
            'transaction_id' => 'TRX-' . uniqid(),
            'redirect_url' => 'https://app.midtrans.com/payment/' . $order->order_number,
        ];
    }

    public function handleNotification(array $payload): void
    {
        $orderId = $payload['order_id'] ?? null;
        $status = $payload['transaction_status'] ?? null;

        if ($orderId && $status === 'settlement') {
            Order::where('order_number', $orderId)
                ->update(['status' => 'paid']);
        }
    }
}
```

---

## 4. Service Dependency Graph

### 4.1 Bagaimana Service Saling Bergantung

```
CartService
    └── CustomerService

OrderService
    ├── CartService
    ├── MidtransService
    └── RajaOngkirService

RajaOngkirService
    └── (Http Client, Config)

MidtransService
    └── (Config)
```

### 4.2 Container Auto-Resolusi

```php
// Karena semua service di-type-hint di constructor,
// Laravel container auto-resolve semua dependency!

// Cukup minta OrderService → container akan:
// 1. Buat CustomerService
// 2. Buat CartService (dengan CustomerService)
// 3. Buat MidtransService 
// 4. Buat RajaOngkirService
// 5. Buat OrderService (dengan 3 service di atas)

$orderService = app(OrderService::class);
```

---

## 5. Best Practices Service

### 5.1 Thin Controller

```php
// ✅ Controller hanya:
// 1. Terima request
// 2. Validasi (FormRequest otomatis)
// 3. Panggil service
// 4. Return response

class OrderController extends Controller
{
    public function __construct(
        private OrderService $orderService,
    ) {}

    public function index(): View
    {
        $orders = $this->orderService->getUserOrders(auth()->id());
        return view('orders.index', compact('orders'));
    }

    public function show(Order $order): View
    {
        $this->authorize('view', $order);
        return view('orders.show', compact('order'));
    }
}
```

### 5.2 Service Naming

```php
// Service untuk satu domain/entity
CartService       → Operasi terkait cart
OrderService      → Operasi terkait order
PaymentService    → Operasi terkait payment
ProductService    → Operasi terkait product
RajaOngkirService → Integrasi RajaOngkir API
MidtransService   → Integrasi Midtrans API
```

### 5.3 Kapan Perlu Service?

```php
// ✅ PERLU SERVICE — logic kompleks
// - Create order (validasi stok, hitung, simpan, kurangi)
// - Proses payment (multiple API call)
// - Hitung ongkir (logika rumit)

// ❌ TIDAK PERLU SERVICE — operasi sederhana
// - Ambil data (langsung dari controller)
// - Simple CRUD tanpa logic tambahan
// - Filter query tanpa transformasi
```

---

## 6. Testing Service

```php
// Service mudah di-test karena dependency bisa di-mock:
public function test_create_order_reduces_stock()
{
    // Mock dependency
    $cartService = Mockery::mock(CartService::class);
    $cartService->shouldReceive('getCart')->andReturn([...]);
    $cartService->shouldReceive('clear')->once();

    // Test service
    $orderService = new OrderService(
        $cartService,
        $this->mockMidtransService(),
        $this->mockRajaOngkirService(),
    );

    $order = $orderService->createOrder($data, $user);

    $this->assertEquals('pending', $order->status);
    $this->assertDatabaseHas('orders', ['id' => $order->id]);
}
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ SERVICE LAYER ARCHITECTURE                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Controller (handle HTTP)                                          │
│      │                                                             │
│      ▼                                                             │
│  Service (business logic)                                          │
│      │                                                             │
│      ├── CartService: cart CRUD, total, count                     │
│      ├── OrderService: create order, validate stock               │
│      ├── RajaOngkirService: provinces, cities, shipping cost      │
│      └── MidtransService: payment, notification                   │
│      │                                                             │
│      ▼                                                             │
│  Model (data / database)                                           │
│      │                                                             │
│      ▼                                                             │
│  Database                                                           │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│ KEUNTUNGAN SERVICE                                                  │
│  ✅ Controller ramping (5-10 baris)                               │
│  ✅ Logika reusable (CartService dipakai di banyak tempat)         │
│  ✅ Testable (unit test tanpa HTTP)                                │
│  ✅ Single Responsibility                                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Baca CartService.** Buka `app/Services/CartService.php`. Identifikasi:
   - Semua method yang ada
   - Bagaimana cart disimpan (session? database?)
   - Constructor injection — service apa yang dipakai?

2. **Baca OrderService.** Buka `app/Services/OrderService.php` (jika ada):
   - Apa saja yang dilakukan method `createOrder`?
   - Service apa yang dipakai?

3. **Trace dependency.** CartService pakai CustomerService. Buka `app/Services/CustomerService.php` — apa fungsinya?

4. **Buat service.** Buat `ProductService` dengan method:
   - `getActiveProducts()` — produk aktif
   - `getProductBySlug(string $slug)` — cari via slug
   - `getFeaturedProducts(int $limit = 8)` — produk unggulan

5. **Refaktor.** Ambil koneksi dari `CartService` dan `RajaOngkirService`. Service mana yang paling banyak dipakai service lain?

---

## 🔗 Referensi

- [Laravel Architecture: Service Layer](https://laravel.com/docs/11.x/architecture)
- [Martin Fowler: Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html)
- Codebase: `app/Services/CartService.php`
- Codebase: `app/Services/OrderService.php` (jika ada)
- Codebase: `app/Services/RajaOngkirService.php`
- Codebase: `app/Services/MidtransService.php`
