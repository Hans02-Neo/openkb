# 06-10: Queue dan Job — Tugas Latar Belakang

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-09-migration-dan-seeder  
> **Waktu baca**: 60-75 menit  
> **Kata kunci**: queue, job, worker, dispatch, horizon, sync, database queue, failed jobs, delay

---

## 📋 Ringkasan

Beberapa tugas tidak perlu dijalankan **saat itu juga** — kirim email, proses gambar, generate laporan. **Queue** memungkinkan tugas dijalankan **nanti** oleh worker di latar belakang.

**Target pemahaman:**
- Kamu paham kapan perlu queue
- Kamu bisa membuat dan menjalankan job
- Kamu paham queue driver (sync, database, redis)
- Kamu bisa handle failed jobs

---

## 1. Konsep Queue

### 1.1 Request Lifecycle

**Tanpa queue: semua dikerjakan di request yang sama**

```
Browser ──POST /order──► Server ──create order──► send email──► response
                              ▲         │             │
                              │         ▼             ▼
                              │    100ms          3000ms (lambat!)
                              │
                              └──── Total: 3100ms — user nunggu!
```

**Dengan queue: tugas berat dikirim ke antrian**

```
Browser ──POST /order──► Server ──create order──► dispatch job──► response
                              ▲         │             │
                              │         ▼             ▼
                              │    100ms          1ms (masukkan queue!)
                              │
                              └──── Total: ~101ms — user tidak nunggu!

                              ┌── Queue Worker ── (proses di latar belakang)
                              │   send email (3000ms)
                              │   user sudah dapat response!
```

### 1.2 Analogi Restoran

| Tanpa Queue | Dengan Queue |
|-------------|-------------|
| Pelanggan pesan → koki masak → pelanggan tunggu | Pelanggan pesan → koki tulis order → tempel di papan → "selanjutnya" koki masak |
| Pelanggan antri sampai masakan selesai | Pelanggan duduk, pesanan diantri |

---

## 2. Job — Unit Pekerjaan

### 2.1 Membuat Job

```bash
php artisan make:job SendOrderConfirmation
```

```php
<?php
// app/Jobs/SendOrderConfirmation.php

namespace App\Jobs;

use App\Models\Order;
use App\Mail\OrderConfirmation;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Support\Facades\Mail;

class SendOrderConfirmation implements ShouldQueue
{
    use Queueable;

    // Job menerima data via constructor
    public function __construct(
        public Order $order,
    ) {}

    // handle() — method yang dijalankan worker
    public function handle(): void
    {
        Mail::to($this->order->user->email)
            ->send(new OrderConfirmation($this->order));
    }
}
```

### 2.2 Dispatch — Masukkan ke Queue

```php
// Dari controller atau service:
public function process(CheckoutRequest $request): RedirectResponse
{
    $order = $this->orderService->createOrder($request->validated());

    // Dispatch job — langsung return, job jalan di background
    SendOrderConfirmation::dispatch($order);

    return redirect()->route('orders.show', $order);
}

// Dispatch dengan delay:
SendOrderConfirmation::dispatch($order)
    ->delay(now()->addMinutes(10)); // kirim email 10 menit lagi

// Dispatch ke queue spesifik:
SendOrderConfirmation::dispatch($order)
    ->onQueue('emails');
```

### 2.3 Menjalankan Queue Worker

```bash
# Jalankan worker — proses job antrian
php artisan queue:work

# Worker spesifik queue
php artisan queue:work --queue=emails,default

# Worker dengan limit memory
php artisan queue:work --memory=128

# Proses sekali saja
php artisan queue:work --once

# Lihat job gagal
php artisan queue:failed
```

---

## 3. Queue Driver — Penyimpanan Antrian

### 3.1 sync (Development)

```php
// config/queue.php — default di development
'default' => env('QUEUE_CONNECTION', 'sync'),

// Sync = jalan SEKARANG, bukan di background
// Tidak perlu worker — cocok untuk development
```

### 3.2 database (Production Sederhana)

```php
// config/queue.php
'default' => env('QUEUE_CONNECTION', 'database'),

// Butuh table jobs:
php artisan queue:table
php artisan migrate

// Jobs tersimpan di table 'jobs'
// Worker membaca table dan menjalankan job
```

### 3.3 redis (Production Skala Besar)

```php
// config/queue.php
'default' => env('QUEUE_CONNECTION', 'redis'),

// Butuh Redis server
// Lebih cepat dari database — cocok untuk high traffic
```

### 3.4 Queue di Codebase

```php
// .env — queue connection
QUEUE_CONNECTION=sync
// atau database

// config/queue.php
'default' => env('QUEUE_CONNECTION', 'sync'),
```

---

## 4. Job Middleware dan Event

### 4.1 Job Middleware

```php
<?php
// app/Jobs/SendOrderConfirmation.php

use Illuminate\Support\Facades\Log;

public function middleware(): array
{
    return [
        // Jangan ulang jika sudah punya unique ID
        new \Illuminate\Queue\Middleware\WithoutOverlapping($this->order->id),

        // Batasi maksimal 3 kali percobaan
        new \Illuminate\Queue\Middleware\ThrottlesExceptions(3, 10),
    ];
}
```

### 4.2 Job Events

```php
// app/Providers/AppServiceProvider.php

use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobProcessing;
use Illuminate\Queue\Events\JobFailed;

public function boot(): void
{
    Queue::before(function (JobProcessing $event) {
        Log::info('Processing job: ' . $event->job->getName());
    });

    Queue::after(function (JobProcessed $event) {
        Log::info('Job completed: ' . $event->job->getName());
    });

    Queue::failing(function (JobFailed $event) {
        Log::error('Job failed: ' . $event->job->getName(), [
            'exception' => $event->exception,
        ]);
    });
}
```

### 4.3 Failed Jobs

```php
// Setelah job gagal beberapa kali → masuk ke table failed_jobs
// Butuh migration:
php artisan queue:failed-table
php artisan migrate

// Lihat job gagal
php artisan queue:failed

// Retry job gagal
php artisan queue:retry all
php artisan queue:retry 5  // specific ID

// Hapus job gagal
php artisan queue:forget 5
php artisan queue:flush  // hapus semua
```

---

## 5. Job Chaining dan Batches

### 5.1 Job Chain — Berurutan

```php
// Jalankan job berurutan — job 2 jalan setelah job 1 sukses
use App\Jobs\ProcessOrder;
use App\Jobs\SendOrderConfirmation;
use App\Jobs\UpdateInventory;

ProcessOrder::withChain([
    new SendOrderConfirmation($order),
    new UpdateInventory($order),
])->dispatch();
```

### 5.2 Job Batch — Paralel + Callback

```php
// Batch — jalankan beberapa job paralel, lalu callback
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([
    new ProcessOrder($order),
    new SendOrderConfirmation($order),
    new UpdateInventory($order),
])->then(function (Batch $batch) {
    // Semua job sukses
    Log::info('All jobs completed');
})->catch(function (Batch $batch, Throwable $e) {
    // Salah satu job gagal
    Log::error('Batch failed: ' . $e->getMessage());
})->finally(function (Batch $batch) {
    // Selesai (sukses atau gagal)
})->dispatch();
```

---

## 6. Queue di Codebase

### 6.1 Job Check Midtrans Status

```php
<?php
// app/Jobs/CheckMidtransStatus.php

namespace App\Jobs;

use App\Models\Order;
use App\Services\MidtransService;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class CheckMidtransStatus implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Order $order,
        public int $attempt = 0,
    ) {}

    public function handle(MidtransService $midtrans): void
    {
        $status = $midtrans->checkStatus($this->order);

        if ($status === 'pending' && $this->attempt < 5) {
            // Masih pending → cek lagi 5 menit lagi
            self::dispatch($this->order, $this->attempt + 1)
                ->delay(now()->addMinutes(5));
        } elseif ($status === 'settlement') {
            $this->order->update(['status' => 'paid']);
        } elseif ($status === 'expire' || $status === 'deny') {
            $this->order->update(['status' => 'cancelled']);
        }
    }
}
```

### 6.2 Dispatch Job di Checkout

```php
// Setelah checkout — dispatch job check status
CheckMidtransStatus::dispatch($order)
    ->delay(now()->addMinutes(5)); // cek 5 menit setelah order
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ QUEUE FLOW                                                         │
│  Controller → dispatch job → return response (cepat!)              │
│  Job masuk queue (database/redis)                                  │
│  Worker baca queue → handle() → job selesai                       │
├─────────────────────────────────────────────────────────────────────┤
│ DRIVERS                                                            │
│  sync      → langsung di request (development)                     │
│  database  → table 'jobs' (sederhana)                             │
│  redis     → Redis list (cepat, skalabel)                         │
├─────────────────────────────────────────────────────────────────────┤
│ COMMANDS                                                           │
│  php artisan queue:work      → jalankan worker                    │
│  php artisan queue:failed    → lihat job gagal                    │
│  php artisan queue:retry all → ulang job gagal                    │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  app/Jobs/CheckMidtransStatus.php                                  │
│  Dispatch di CheckoutController setelah order                     │
│  Queue connection: sync (default) atau database                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Cari job.** Buka `app/Jobs/`. Job apa saja yang ada? Apa yang dilakukan `handle()`?

2. **Cek config queue.** Buka `config/queue.php`. Queue driver apa yang dikonfigurasi? Apa default-nya?

3. **Buat job.** Buat `LogNewOrder` job yang mencatat order baru ke log:
   ```php
   Log::info('Order created: ' . $order->order_number);
   ```

4. **Dispatch job.** Di controller, dispatch job setelah order berhasil dibuat.

5. **Simulasi queue.** Ubah `QUEUE_CONNECTION` ke `database`:
   - Buat migration `queue:table` dan `failed:table` (jika belum)
   - Jalankan `php artisan queue:work` di terminal lain
   - Buat order — lihat apakah job masuk ke table `jobs` dan diproses

---

## 🔗 Referensi

- [Laravel Docs: Queues](https://laravel.com/docs/11.x/queues)
- [Laravel Docs: Job Middleware](https://laravel.com/docs/11.x/queues#job-middleware)
- [Laravel Docs: Queue Workers](https://laravel.com/docs/11.x/queues#running-the-queue-worker)
- [Laravel Docs: Failed Jobs](https://laravel.com/docs/11.x/queues#dealing-with-failed-jobs)
- Codebase: `app/Jobs/CheckMidtransStatus.php`
- Codebase: `config/queue.php`
