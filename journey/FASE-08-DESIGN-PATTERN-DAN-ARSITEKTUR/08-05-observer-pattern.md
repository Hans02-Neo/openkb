# 08-05: Observer Pattern — Event & Listener

> **Fase**: 8 — Design Pattern & Arsitektur  
> **Prasyarat**: 08-04-repository-pattern  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: observer, event, listener, model observer, hook, decoupled, notification

---

## 📋 Ringkasan

Observer pattern memungkinkan satu object (subject) memberi tahu object lain (observer) ketika terjadi perubahan — tanpa perlu **tight coupling**.

Di Laravel ada 2 implementasi:
1. **Events & Listeners** — generic event system
2. **Model Observers** — khusus Eloquent model events

**Target pemahaman:**
- Kamu paham konsep observer pattern
- Kamu bisa membedakan Event vs Observer
- Kamu bisa membaca dan membuat observer
- Kamu paham observer di codebase ini (OrderObserver)

---

## 1. Konsep Observer Pattern

### 1.1 Analogi

```text
YouTube:
  Subject: Channel A (upload video)
  Observer 1: Subscriber Budi → dapat notifikasi
  Observer 2: Subscriber Siti → dapat notifikasi
  Observer 3: Subscriber Andi → dapat notifikasi

  Channel tidak perlu tahu detail subscriber.
  Subscriber bisa datang & pergi kapan saja.
```

### 1.2 Diagram

```text
┌──────────┐  notify()   ┌──────────────┐
│ Subject  │ ──────────▶ │ Observer     │
│ (Event)  │             │ (Listener)   │
└──────────┘             └──────────────┘
      │                          │
      │ event terjadi            │ reaksi
      ▼                          ▼
  "OrderCreated"            "SendEmailToCustomer"
                            "UpdateStock"
                            "LogActivity"
```

### 1.3 Tanpa Observer (Tight Coupling)

```php
// ❌ Controller harus tahu SEMUA reaksi
class OrderController extends Controller
{
    public function store(CheckoutRequest $request): RedirectResponse
    {
        $order = Order::create($request->validated());

        // Reaksi 1: kirim email
        Mail::to($order->user)->send(new OrderConfirmation($order));

        // Reaksi 2: update stok
        foreach ($order->items as $item) {
            $item->product->decrement('stock', $item->quantity);
        }

        // Reaksi 3: log activity
        ActivityLog::log('Order created', $order);

        // Reaksi 4: notify admin
        AdminNotification::send('New order!');

        return redirect()->route('orders.show', $order);
    }
}
```

### 1.4 Dengan Observer (Decoupled)

```php
// ✅ Event: cukup trigger event
class OrderController extends Controller
{
    public function store(CheckoutRequest $request): RedirectResponse
    {
        $order = Order::create($request->validated());

        // Trigger event — tidak tahu siapa yang mendengarkan
        OrderCreated::dispatch($order);

        return redirect()->route('orders.show', $order);
    }
}

// Listener 1: SendOrderConfirmation
class SendOrderConfirmation implements ShouldQueue
{
    public function handle(OrderCreated $event): void
    {
        Mail::to($event->order->user)
            ->send(new OrderConfirmation($event->order));
    }
}

// Listener 2: DecrementStock
class DecrementStock implements ShouldQueue
{
    public function handle(OrderCreated $event): void
    {
        foreach ($event->order->items as $item) {
            $item->product->decrement('stock', $item->quantity);
        }
    }
}
```

---

## 2. Events & Listeners di Laravel

### 2.1 Struktur

```
app/Events/
├── OrderCreated.php        ← Event class
└── OrderStatusChanged.php

app/Listeners/
├── SendOrderConfirmation.php  ← Listener class
├── DecrementStock.php
└── NotifyAdmin.php
```

### 2.2 Event Class

```php
<?php
namespace App\Events;

use App\Models\Order;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderCreated
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order  // Data yang dibawa event
    ) {}
}
```

### 2.3 Listener Class

```php
<?php
namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

// ShouldQueue = proses async (biar response cepet)
class SendOrderConfirmation implements ShouldQueue
{
    public function handle(OrderCreated $event): void
    {
        // $event->order available di sini
        Mail::to($event->order->user)
            ->send(new OrderConfirmation($event->order));
    }
}
```

### 2.4 Registrasi

```php
// app/Providers/EventServiceProvider.php
class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        OrderCreated::class => [
            SendOrderConfirmation::class,
            DecrementStock::class,
            NotifyAdmin::class,
        ],
    ];

    public function boot(): void
    {
        parent::boot();
    }
}
```

> **Catatan:** Codebase ini tidak punya `EventServiceProvider`. Events/listeners belum digunakan.

---

## 3. Model Observers

### 3.1 Eloquent Events

Eloquent punya event bawaan per model:

| Event | Dipanggil ketika |
|-------|-----------------|
| `retrieved` | Model di-load dari DB |
| `creating` | Sebelum INSERT (belum tersimpan) |
| `created` | Setelah INSERT |
| `updating` | Sebelum UPDATE |
| `updated` | Setelah UPDATE |
| `saving` | Sebelum INSERT atau UPDATE |
| `saved` | Setelah INSERT atau UPDATE |
| `deleting` | Sebelum DELETE |
| `deleted` | Setelah DELETE |
| `restoring` | Sebelum restore soft delete |
| `restored` | Setelah restore soft delete |
| `forceDeleting` | Sebelum force delete |
| `forceDeleted` | Setelah force delete |

### 3.2 Observer Class

Daripada membuat banyak listener, gabung di satu class:

```php
<?php
namespace App\Observers;

use App\Models\Order;
use App\Models\OrderStatusHistory;
use Illuminate\Support\Facades\Auth;

class OrderObserver
{
    public function updating(Order $order): void
    {
        // Catat perubahan status ke history
        if ($order->isDirty('status')) {
            OrderStatusHistory::create([
                'order_id'   => $order->id,
                'old_status' => $order->getOriginal('status'),
                'new_status' => $order->status,
                'changed_by' => Auth::id(),
                'changed_at' => now(),
            ]);
        }
    }
}
```

### 3.3 Registrasi Observer

```php
// app/Providers/AppServiceProvider.php
use App\Models\Order;
use App\Observers\OrderObserver;

public function boot(): void
{
    Order::observe(OrderObserver::class);
}
```

---

## 4. Observer di Codebase Ini

### 4.1 File Observer

```
app/Observers/OrderObserver.php
```

```php
// Isi: mencatat history status order
class OrderObserver
{
    public function updating(Order $order): void
    {
        if ($order->isDirty('status')) {
            OrderStatusHistory::create([
                'order_id'   => $order->id,
                'old_status' => $order->getOriginal('status'),
                'new_status' => $order->status,
                'changed_by' => Auth::id(),
                'changed_at' => now(),
            ]);
        }
    }
}
```

### 4.2 Alur

```
User update status order (via admin panel)
    │
    ▼
OrderController@updateStatus()
    │
    ▼
$order->update(['status' => 'shipped'])
    │
    ▼
Eloquent trigger UPDATING event
    │
    ▼
OrderObserver@updating()
    │
    ▼
Buat record di order_status_histories
    │
    ▼
Order tersimpan dengan status baru
```

### 4.3 Kenapa Observer?

- **Separation of concerns:** Controller tidak perlu tahu soal history
- **Auto:** Setiap kali status berubah, history tercatat otomatis
- **Maintainable:** Mau nambah kolom di history? Edit satu file

---

## 5. Events vs Observers — Kapan Pakai Apa?

| | Events & Listeners | Model Observers |
|---|-------------------|-----------------|
| **Trigger** | Manual (`Event::dispatch()`) | Otomatis (Eloquent events) |
| **Scope** | Cross-domain (order → email + stock + log) | Satu model |
| **Async** | Bisa (`ShouldQueue`) | Bisa (Laravel 11+) |
| **Kapan** | Logic setelah aksi (email, notif) | Logic sebelum/sesudah simpan |
| **Contoh** | OrderCreated → SendEmail | OrderObserver → catat history |

### 5.1 Pake Events Kalau:

```php
// 1. Reaksi lintas domain
OrderCreated → SendEmail (Mail)
            → UpdateStock (Inventory)
            → NotifyAdmin (Notification)
            → UpdateStats (Analytics)

// 2. Butuh async (biar response cepet)
class SendEmail implements ShouldQueue {
    public function handle(OrderCreated $event): void {
        // Proses di queue, user langsung dapet response
    }
}

// 3. Reaksi opsional (boleh gagal, tidak kritikal)
```

### 5.2 Pake Observer Kalau:

```php
// 1. Auto-logging / audit trail
OrderObserver@updating → catat perubahan status

// 2. Validasi sebelum simpan
ProductObserver@creating → pastikan slug unik

// 3. Sync data terkait
UserObserver@created → buat default settings
```

---

## 6. Queue dengan Events

Listener bisa di-queue supaya response cepat:

```php
// app/Listeners/SendOrderConfirmation.php
class SendOrderConfirmation implements ShouldQueue
{
    public $queue = 'emails';  // optional: queue name

    public function handle(OrderCreated $event): void
    {
        // Proses ini jalan di background queue worker
        Mail::to($event->order->user)
            ->send(new OrderConfirmation($event->order));
    }
}
```

**Flow:**
```
Request → Controller → dispatch event → response (cepat!)
Queue worker → listener handle email → kirim email (lambat)
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ OBSERVER PATTERN                                                   │
├─────────────────────────────────────────────────────────────────────┤
│  Tanpa Observer:                                                    │
│  Controller → email + stock + log + notif (semua di satu tempat)   │
│                                                                     │
│  Dengan Observer:                                                   │
│  Controller → dispatch(event)                                       │
│                    │                                                │
│                    ├── Listener 1: SendEmail (queue)               │
│                    ├── Listener 2: UpdateStock (queue)             │
│                    └── Listener 3: LogActivity                      │
│                                                                     │
│  Atau (Model Observer):                                             │
│  Eloquent save() → trigger updating() → OrderObserver              │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE INI                                                     │
│  app/Observers/OrderObserver.php — catat history status            │
│  Tidak ada Events/Listeners (belum diperlukan)                     │
│  Observer didaftarkan di AppServiceProvider@boot                   │
├─────────────────────────────────────────────────────────────────────┤
│ EVENTS vs OBSERVERS                                                 │
│  Event: manual dispatch, cross-domain, cocok untuk queue           │
│  Observer: otomatis (Eloquent), satu model, cocok untuk logging    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Baca OrderObserver.** Buka `app/Observers/OrderObserver.php`. Pahami kapan `updating()` dipanggil dan apa yang dilakukan.

2. **Registrasi.** Cek `app/Providers/AppServiceProvider.php`. Bagaimana observer didaftarkan?

3. **Buat observer baru.** Buat `ProductObserver` yang:
   - `creating`: set slug otomatis dari nama
   - `updating`: update slug jika nama berubah

4. **Event manual.** Rancang event `OrderStatusChanged` dengan:
   - Event: membawa `Order` + old status + new status
   - Listener: kirim notifikasi ke customer
   - Dispatch di controller saat status diupdate

5. **Analisis.** Apakah codebase ini perlu Events & Listeners untuk fitur checkout? Atau observer cukup?

---

## 🔗 Referensi

- [Laravel Docs: Events](https://laravel.com/docs/11.x/events)
- [Laravel Docs: Observers](https://laravel.com/docs/11.x/eloquent#observers)
- [Refactoring Guru: Observer](https://refactoring.guru/design-patterns/observer)
- Codebase: `app/Observers/OrderObserver.php`
- Codebase: `app/Providers/AppServiceProvider.php` (registrasi observer)
