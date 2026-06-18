# 08-02: MVC Architecture — Model View Controller

> **Fase**: 8 — Design Pattern & Arsitektur  
> **Prasyarat**: FASE 6 (Laravel Deep Dive)  
> **Waktu baca**: 55-70 menit  
> **Kata kunci**: MVC, model, view, controller, separation of concerns, fat model thin controller, Laravel MVC

---

## 📋 Ringkasan

MVC (Model-View-Controller) adalah pola arsitektur yang memisahkan aplikasi menjadi 3 komponen utama. Laravel mengikuti MVC, meski dengan beberapa adaptasi (Service Layer, Blade, Eloquent).

**Target pemahaman:**
- Kamu paham peran Model, View, Controller
- Kamu bisa mapping file ke komponen MVC
- Kamu paham alur MVC di Laravel
- Kamu paham variasi MVC (HMVC, MVVM)

---

## 1. Tiga Komponen MVC

### 1.1 Diagram

```
User ──► Controller ──► Model ──► Database
           │                │
           │                └── Query, logic
           │
           ▼
          View ◄──── Data dari controller
           │
           ▼
          Browser (HTML)
```

### 1.2 Model — Data

```php
// Model = data + business logic
// Di Laravel: Eloquent Model

class Product extends Model
{
    // Data: representasi table products
    protected $fillable = ['name', 'price', 'category_id'];

    // Business logic: relationship
    public function category(): BelongsTo { ... }

    // Business logic: scope
    public function scopeActive($q) { ... }

    // Business logic: accessor
    public function getPriceAttribute($value) { ... }
}
```

### 1.3 View — Tampilan

```blade
{{-- View = tampilan — hanya presentasi --}}
@extends('layouts.app')

@section('content')
    @foreach($products as $product)
        <div class="card">
            <h3>{{ $product->name }}</h3>
            <p>{{ $product->formatted_price }}</p>
        </div>
    @endforeach
@endsection
```

### 1.4 Controller — Jembatan

```php
// Controller = jembatan antara Model dan View
class ShopController extends Controller
{
    public function index(): View
    {
        // 1. Ambil data dari Model
        $products = Product::active()
            ->with('category')
            ->paginate(12);

        // 2. Kirim ke View
        return view('shop.index', compact('products'));
    }
}
```

---

## 2. Alur MVC di Laravel

### 2.1 Request Lengkap

```
1. Route:      GET /shop → ShopController@index
2. Controller: ShopController@index()
3. Model:      Product::active()->with('category')->paginate(12)
4. Database:   SELECT * FROM products WHERE is_active = 1 ...
5. Controller: return view('shop.index', $data)
6. View:       Blade render → HTML
7. Response:   HTTP 200 → browser
```

### 2.2 Mapping File ke MVC

| MVC | Laravel | Contoh File |
|-----|---------|-------------|
| **Model** | `app/Models/` | `Product.php`, `Order.php` |
| **View** | `resources/views/` | `shop/index.blade.php` |
| **Controller** | `app/Http/Controllers/` | `ShopController.php` |
| **Route** | `routes/web.php` | Route → Controller mapping |
| **Service** | `app/Services/` | Tambahan (bukan MVC murni) |

### 2.3 Variasi: MVC + Service Layer

Laravel modern sering menambah **Service Layer**:

```
Route → Controller → Service → Model → Database
                        │
                        ▼
                      View
```

Controller tidak langsung ke Model, tapi lewat Service:

```php
// Controller (tipis)
class OrderController extends Controller
{
    public function __construct(private OrderService $orderService) {}

    public function store(CheckoutRequest $request): RedirectResponse
    {
        $order = $this->orderService->createOrder(
            $request->validated(),
            auth()->user()
        );
        return redirect()->route('orders.show', $order);
    }
}

// Service (logika bisnis)
class OrderService
{
    public function createOrder(array $data, User $user): Order
    {
        return DB::transaction(function () use ($data, $user) {
            // validasi stok, create order, kurangi stok, dll
        });
    }
}
```

---

## 3. Fat Model vs Thin Controller

### 3.1 Prinsip MVC

```
Model:  Data + Business Logic (boleh agak gemuk)
View:   Tampilan saja (tidak boleh ada logic)
Controller:  Jembatan (harus tipis)
```

### 3.2 Thin Controller (Benar)

```php
// ✅ Controller tipis — hanya delegasi
class ProductController extends Controller
{
    public function index(): View
    {
        // Ambil data, kirim ke view
        return view('products.index', [
            'products' => Product::active()->paginate(20),
        ]);
    }

    public function store(ProductRequest $request): RedirectResponse
    {
        Product::create($request->validated());
        return redirect()->route('products.index')
            ->with('success', 'Produk berhasil ditambahkan');
    }
}
```

### 3.3 Fat Controller (Salah)

```php
// ❌ Controller gemuk — semua logika di sini
public function store(Request $request)
{
    $request->validate([...]);
    $image = $request->file('image');
    $path = $image->store('products');
    $product = new Product();
    $product->name = $request->name;
    $product->price = $request->price;
    // ... 30 baris lagi
    $product->save();
    // Kirim email
    Mail::to('admin@test.com')->send(...);
    // Log activity
    ActivityLog::log('Product created');
}
```

---

## 4. MVC Variations

### 4.1 HMVC — Hierarchical MVC

Sub-request di dalam request:

```php
// Controller memanggil controller lain
$widget = Request::create('/widget/popular-products', 'GET');
$response = app()->handle($widget);
```

### 4.2 MVVM — Model-View-ViewModel

Digunakan di frontend framework (Vue, Angular):

```
Model  ↔  ViewModel (reactive state)  ↔  View (template)
              │
              └── Data binding otomatis
```

### 4.3 MVC di Laravel vs Framework Lain

| Aspek | Laravel | Rails | Django | Spring |
|-------|---------|-------|--------|--------|
| Model | Eloquent ORM | ActiveRecord | ORM | JPA |
| View | Blade | ERB | Django Templates | JSP/Thymeleaf |
| Controller | Controller | Controller | View | Controller |
| Routing | web.php | routes.rb | urls.py | RequestMapping |

---

## 5. MVC di Codebase

### 5.1 Per File

```
routes/web.php            → Routing
app/Http/Controllers/     → Controller
app/Models/               → Model (Eloquent)
resources/views/          → View (Blade)
app/Services/             → Service Layer (tambahan)
```

### 5.2 Contoh: Satu Fitur Lengkap

```php
// 1. Route (routes/web.php)
Route::get('/orders/{order}', [OrderController::class, 'show'])
    ->name('orders.show');

// 2. Controller (app/Http/Controllers/OrderController.php)
public function show(Order $order): View
{
    $this->authorize('view', $order);
    $order->load('items.product');
    return view('orders.show', compact('order'));
}

// 3. Model (app/Models/Order.php)
class Order extends Model
{
    public function items(): HasMany { ... }
    public function user(): BelongsTo { ... }
}

// 4. View (resources/views/orders/show.blade.php)
@extends('layouts.app')
@section('content')
    <h1>Order #{{ $order->order_number }}</h1>
    @foreach($order->items as $item)
        <p>{{ $item->product->name }} x {{ $item->quantity }}</p>
    @endforeach
@endsection
```

---

## 6. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ MVC PATTERN                                                        │
├─────────────────────────────────────────────────────────────────────┤
│  MODEL              │  VIEW                │  CONTROLLER           │
│  Data & logic       │  Tampilan            │  Jembatan             │
│  app/Models/        │  resources/views/    │  app/Http/Controllers/│
│                     │                       │                       │
│  Product.php        │  shop/index.blade.php │  ShopController.php   │
│  Order.php          │  orders/show.blade.php│  OrderController.php  │
│  User.php           │  layouts/app.blade.php│  CartController.php   │
├─────────────────────┴───────────────────────┴───────────────────────┤
│ ALUR REQUEST                                                        │
│  Route → Controller → (Service) → Model → DB                       │
│                        → View → HTML → Response                    │
├─────────────────────────────────────────────────────────────────────┤
│ BEST PRACTICE                                                       │
│  ✅ Controller tipis (delegasi ke service/model)                   │
│  ✅ Model untuk data + logic                                       │
│  ✅ View hanya presentasi (tidak ada query)                        │
│  ❌ Fat controller (semua logic di controller)                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Mapping komponen.** Untuk fitur "Cart", identifikasi:
   - Model: file apa?
   - View: file apa?
   - Controller: file apa?
   - Service: file apa?

2. **Thin controller check.** Buka `app/Http/Controllers/CartController.php`. Apakah controller tipis? Atau ada logika yang seharusnya di service?

3. **Alur lengkap.** Trace alur dari route `POST /checkout` sampai response. Sebutkan file dan method yang terlibat.

4. **View tanpa logic.** Cek `resources/views/cart/index.blade.php`. Apakah ada query di view? (seharusnya tidak ada)

5. **Buat fitur MVC.** Rancang MVC untuk fitur "Wishlist":
   - Route apa?
   - Controller method apa?
   - Model apa?
   - View apa?

---

## 🔗 Referensi

- [Wikipedia: MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)
- [Laravel: Architecture](https://laravel.com/docs/11.x/architecture)
- [Martin Fowler: MVC](https://martinfowler.com/eaaCatalog/modelViewController.html)
- Codebase: `app/Http/Controllers/` — controller
- Codebase: `app/Models/` — model
- Codebase: `resources/views/` — view
- Codebase: `app/Services/` — service layer
