# 06-11: Notifikasi dan Mail — Komunikasi dengan User

> **Fase**: 6 — Laravel Deep Dive  
> **Prasyarat**: 06-10-queue-dan-job  
> **Waktu baca**: 55-70 menit  
> **Kata kunci**: mail, notification, mailable, markdown, queue, channel, database notification, log channel

---

## 📋 Ringkasan

Laravel menyediakan sistem **mail** dan **notifikasi** yang terintegrasi. Email bisa dikirim via berbagai driver (SMTP, Mailgun, Log). Notifikasi bisa dikirim via multiple channel (email, database, SMS) dari satu class.

**Target pemahaman:**
- Kamu bisa membuat dan mengirim mail
- Kamu paham notifikasi multi-channel
- Kamu bisa menggunakan Markdown mail template
- Kamu paham konfigurasi mail driver

---

## 1. Mail — Mengirim Email

### 1.1 Mail Driver

```php
// config/mail.php
'default' => env('MAIL_MAILER', 'log'),

// Driver yang tersedia:
// 'smtp'     → SMTP server (Gmail, Mailgun, SendGrid)
// 'log'      → Simpan di log file (development)
// 'array'    → Simpan di array (testing)
// 'sendmail' → Sendmail binary
// 'mailgun'  → Mailgun API
// 'postmark' → Postmark API
// 'ses'      → Amazon SES
```

```php
// .env — development pakai log
MAIL_MAILER=log

// .env — production pakai SMTP
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailgun.org
MAIL_PORT=587
MAIL_USERNAME=postmaster@mg.koneksi-store.com
MAIL_PASSWORD=your-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@koneksi-store.com
MAIL_FROM_NAME="Koneksi Store"
```

### 1.2 Membuat Mail

```bash
php artisan make:mail OrderConfirmation --markdown=emails.orders.confirmed
```

```php
<?php
// app/Mail/OrderConfirmation.php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class OrderConfirmation extends Mailable
{
    use Queueable, SerializesModels;

    public function __construct(
        public Order $order, // otomatis tersedia di view
    ) {}

    // Envelope — subject, from, to
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Pesanan #' . $this->order->order_number . ' Dikonfirmasi',
            from: new Address('noreply@koneksi-store.com', 'Koneksi Store'),
        );
    }

    // Content — view untuk email
    public function content(): Content
    {
        return new Content(
            markdown: 'emails.orders.confirmed',
            with: [
                'order' => $this->order,
                'total' => number_format($this->order->total_amount, 0, ',', '.'),
            ],
        );
    }

    // Attachments
    public function attachments(): array
    {
        return [
            // Attachment::fromPath('/path/to/invoice.pdf'),
        ];
    }
}
```

### 1.3 Markdown Mail Template

```blade
{{-- resources/views/emails/orders/confirmed.blade.php --}}
<x-mail::message>
# Pesanan Dikonfirmasi

Halo **{{ $order->user->name }}**,

Pesanan kamu dengan nomor **{{ $order->order_number }}** telah berhasil dibuat.

<x-mail::panel>
**Total Pembayaran:** Rp {{ $total }}
**Status:** {{ ucfirst($order->status) }}
</x-mail::panel>

<x-mail::table>
| Produk | Harga | Qty | Subtotal |
|--------|-------|-----|----------|
@foreach($order->items as $item)
| {{ $item->product_name }} | Rp {{ number_format($item->price, 0, ',', '.') }} | {{ $item->quantity }} | Rp {{ number_format($item->subtotal, 0, ',', '.') }} |
@endforeach
</x-mail::table>

<x-mail::button :url="route('orders.show', $order)" color="success">
Lihat Detail Pesanan
</x-mail::button>

Terima kasih telah berbelanja di **Koneksi Store**.

<x-mail::subcopy>
Jika kamu mengalami kesulitan, balas email ini atau hubungi customer support.
</x-mail::subcopy>
</x-mail::message>
```

### 1.4 Mengirim Mail

```php
// Di controller/service:
use App\Mail\OrderConfirmation;
use Illuminate\Support\Facades\Mail;

// Kirim email
Mail::to($order->user->email)->send(new OrderConfirmation($order));

// Kirim ke multiple recipients
Mail::to($order->user->email)
    ->cc('admin@koneksi-store.com')
    ->bcc('owner@koneksi-store.com')
    ->send(new OrderConfirmation($order));

// Queue email (biar tidak lambat)
Mail::to($order->user->email)
    ->queue(new OrderConfirmation($order));
// Atau: $order->user->notify(new OrderConfirmed($order));
```

### 1.5 Mail di Codebase

```php
// app/Mail/ — lihat file yang ada
// Biasanya: OrderConfirmation, Invoice, dll.

// Dipanggil di service:
Mail::to($order->user->email)
    ->queue(new OrderConfirmation($order));
```

---

## 2. Notifikasi — Multi-Channel

### 2.1 Konsep

Notifikasi mengirim pesan ke user via **banyak channel** dari satu class:

```
Notifikasi OrderConfirmed
├── mail → email
├── database → table notifications
└── sms → via Twilio/Vonage (jika diatur)
```

### 2.2 Membuat Notifikasi

```bash
php artisan make:notification OrderConfirmed
```

```php
<?php
// app/Notifications/OrderConfirmed.php

namespace App\Notifications;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;

class OrderConfirmed extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Order $order,
    ) {}

    // Channel yang dipakai
    public function via(object $notifiable): array
    {
        return ['mail', 'database'];
        // $notifiable = User — bisa cek preferensi user
    }

    // Channel: mail
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Pesanan #' . $this->order->order_number . ' Dikonfirmasi')
            ->greeting('Halo ' . $notifiable->name . '!')
            ->line('Pesanan kamu telah berhasil dibuat.')
            ->line('Total: Rp ' . number_format($this->order->total_amount, 0, ',', '.'))
            ->action('Lihat Pesanan', route('orders.show', $this->order))
            ->line('Terima kasih telah berbelanja!');
    }

    // Channel: database
    public function toDatabase(object $notifiable): array
    {
        return [
            'order_id' => $this->order->id,
            'order_number' => $this->order->order_number,
            'message' => 'Pesanan #' . $this->order->order_number . ' dikonfirmasi',
            'url' => route('orders.show', $this->order),
        ];
    }

    // Channel: Vonage/SMS
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
            ->content('Pesanan #' . $this->order->order_number . ' dikonfirmasi!');
    }
}
```

### 2.3 Mengirim Notifikasi

```php
// Via Notifiable trait (User model)
$user = User::find(1);
$user->notify(new OrderConfirmed($order));

// Via Notification facade
Notification::send($user, new OrderConfirmed($order));

// Ke multiple users
$admins = User::where('role', 'admin')->get();
Notification::send($admins, new AdminNotification($order));

// Delay
$user->notify((new OrderConfirmed($order))->delay(now()->addMinutes(5)));
```

### 2.4 Database Notification

```php
// Butuh table notifications
php artisan notifications:table
php artisan migrate

// User bisa mengambil notifikasi:
$user = User::find(1);
$user->notifications;        // semua notifikasi
$user->unreadNotifications;  // yang belum dibaca
$user->readNotifications;    // yang sudah dibaca

// Mark as read:
$notification->markAsRead();
$user->unreadNotifications->markAsRead(); // semua
```

### 2.5 Menampilkan Notifikasi di Blade

```blade
{{-- Navbar — badge notifikasi --}}
@auth
    <span class="position-relative">
        🔔
        @if(auth()->user()->unreadNotifications->count() > 0)
            <span class="badge bg-danger position-absolute top-0 start-100 translate-middle">
                {{ auth()->user()->unreadNotifications->count() }}
            </span>
        @endif
    </span>

    {{-- Dropdown notifikasi --}}
    <div class="dropdown-menu">
        @forelse(auth()->user()->unreadNotifications->take(5) as $notification)
            <a class="dropdown-item" href="{{ $notification->data['url'] ?? '#' }}">
                {{ $notification->data['message'] }}
                <small class="text-muted d-block">
                    {{ $notification->created_at->diffForHumans() }}
                </small>
            </a>
        @empty
            <span class="dropdown-item text-muted">Tidak ada notifikasi</span>
        @endforelse
    </div>
@endauth
```

---

## 3. Notifikasi di Codebase

### 3.1 Notifikasi yang Ada

```php
// app/Notifications/
├── OrderConfirmed.php      → saat order berhasil
├── PaymentReceived.php     → saat pembayaran diterima
└── OrderShipped.php        → saat order dikirim
```

### 3.2 Mail vs Notifikasi — Kapan Pakai yang Mana?

| | Mail (Mailable) | Notifikasi |
|---|-----------------|-----------|
| **Channel** | Email saja | Email + database + SMS + ... |
| **Template** | Markdown email | MailMessage + database array |
| **Queue** | `->queue()` | `implements ShouldQueue` |
| **Gunakan** | Email marketing, invoice | Notifikasi multi-channel, sistem |

---

## 4. Konfigurasi Email

### 4.1 Mail di .env

```ini
# Development — cukup log (tidak benar-benar kirim)
MAIL_MAILER=log

# Production — SMTP
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@koneksi-store.com
MAIL_FROM_NAME="Koneksi Store"
```

### 4.2 Test Email

```php
// php artisan tinker
Mail::raw('Test email', function ($message) {
    $message->to('test@example.com')
            ->subject('Test');
});
// Cek storage/logs/laravel.log untuk output
```

---

## 5. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ MAIL                                                               │
│  php artisan make:mail OrderConfirmation --markdown=emails.order   │
│  Mail::to($user)->send(new OrderConfirmation($order))             │
│  Mail::to($user)->queue(new OrderConfirmation($order))  (async)   │
├─────────────────────────────────────────────────────────────────────┤
│ NOTIFICATION                                                       │
│  php artisan make:notification OrderConfirmed                     │
│  via(): ['mail', 'database']                                      │
│  $user->notify(new OrderConfirmed($order))                        │
│  Notification::send($users, new OrderConfirmed($order))           │
├─────────────────────────────────────────────────────────────────────┤
│ CHANNELS                                                           │
│  mail     → email                                                  │
│  database → table notifications                                   │
│  vonage   → SMS                                                    │
│  slack    → Slack webhook                                          │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE                                                        │
│  app/Mail/OrderConfirmation.php — email invoice                   │
│  app/Notifications/OrderConfirmed.php — multi-channel             │
│  MAIL_MAILER=log → lihat di storage/logs/laravel.log              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Cek mail config.** Buka `config/mail.php`. Driver apa yang dipakai? Dari mana driver diambil?

2. **Buat mail.** Buat `InvoiceMail` dengan Markdown template `emails.invoice`. Isi:
   - Subject: "Invoice #ORD-xxx"
   - Menampilkan daftar item order
   - Tombol "Lihat Detail"

3. **Cek log email.** Kirim email via `php artisan tinker`:
   ```php
   Mail::raw('Test', fn($m) => $m->to('test@test.com')->subject('Test'));
   ```
   Cek `storage/logs/laravel.log` — cari email yang "dikirim".

4. **Buat notifikasi.** Buat `OrderShipped` notification dengan channel mail dan database.

5. **Tampilkan notifikasi.** Di Blade navbar, tampilkan badge unread notification count.

---

## 🔗 Referensi

- [Laravel Docs: Mail](https://laravel.com/docs/11.x/mail)
- [Laravel Docs: Markdown Mail](https://laravel.com/docs/11.x/mail#markdown-mailables)
- [Laravel Docs: Notifications](https://laravel.com/docs/11.x/notifications)
- [Laravel Docs: Database Notifications](https://laravel.com/docs/11.x/notifications#database-notifications)
- Codebase: `app/Mail/` — mail classes
- Codebase: `app/Notifications/` — notification classes
- Codebase: `resources/views/emails/` — email templates
- Codebase: `config/mail.php` — mail configuration
