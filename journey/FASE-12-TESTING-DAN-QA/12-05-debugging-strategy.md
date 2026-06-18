# 12-05: Debugging Strategy

> **Fase**: 12 — Testing & QA  
> **Prasyarat**: 12-04-testing-di-codebase-ini  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: debugging, log, dd, dump, telescope, error handling, troubleshooting

---

## 📋 Ringkasan

Debugging adalah proses menemukan dan memperbaiki error. Laravel punya banyak tool untuk debugging: dari `dd()` sederhana sampai Telescope untuk inspeksi mendalam.

**Target pemahaman:**
- Kamu paham debugging workflow
- Kamu bisa pakai dd, dump, log
- Kamu bisa baca error log
- Kamu tau kapan pakai Telescope

---

## 1. Debugging Workflow

```
1. Error muncul
   ↓
2. Baca error message
   ↓
3. Cek lokasi error (file:line)
   ↓
4. Analisis penyebab
   ↓
5. Fix → test → fix lagi
```

---

## 2. dd() & dump()

### 2.1 dd() — Dump and Die

```php
// Hentikan eksekusi, tampilkan data
dd($request->all());
dd($product, $product->category, $product->media);
dd('sampai sini');

// Output:
// array:2 [
//   "product_id" => 1
//   "quantity" => 2
// ]
```

### 2.2 dump() — Dump (lanjut)

```php
dump($product);  // Tampilkan, tapi halaman tetap jalan
dump($user->name);
```

### 2.3 Kapan Pakai?

```php
// ✅ Cocok: development, cari logic error
// ❌ Jangan: di production! (bocorkan data)
// ❌ Jangan: lupa hapus sebelum commit!
```

---

## 3. Logging

### 3.1 Log Levels

```php
use Illuminate\Support\Facades\Log;

Log::debug('Cart updated', ['user_id' => $user->id]);
Log::info('Order created', ['order_id' => $order->id]);
Log::warning('Stock low', ['product_id' => $product->id]);
Log::error('Payment failed', ['order_id' => $order->id, 'error' => $e->getMessage()]);
```

### 3.2 Channel

```php
// Ke file: storage/logs/laravel.log
Log::info('Test');

// Ke channel spesifik
Log::channel('slack')->warning('Server down!');
Log::channel('daily')->info('Daily log');
```

### 3.3 Log vs dd

| | dd() | Log |
|---|------|-----|
| **Hentikan eksekusi** | ✅ | ❌ |
| **Tampilkan di browser** | ✅ | ❌ (ke file) |
| **Production** | ❌ Berbahaya | ✅ Aman |
| **History** | ❌ | ✅  |

---

## 4. Laravel Log File

```text
storage/logs/laravel.log
  ├── [2026-06-18 10:00:00] local.DEBUG: Cart updated {"user_id":1}
  ├── [2026-06-18 10:01:00] local.INFO: Order created {"order_id":5}
  └── [2026-06-18 10:02:00] local.ERROR: SQLSTATE[23000]: ...
```

```bash
# Di terminal:
tail -f storage/logs/laravel.log

# Di PowerShell:
Get-Content storage/logs/laravel.log -Wait
```

---

## 5. Whoops! (Debug Page)

Saat `APP_DEBUG=true` dan error terjadi:

```
Whoops! There was an error.

  [ErrorException]
  Undefined variable $product

  at app/Http/Controllers/ShopController.php:25
  ...
```

**Informasi yang ditampilkan:**
- Error message
- File + line number
- Stack trace (urutan method dari awal sampai error)
- Request data
- Environment info

---

## 6. Laravel Telescope

### 6.1 Telescope di Codebase

```php
// composer.json
"laravel/telescope": "^5.20"
```

Telescope diinstall, bisa diakses via:

```text
URL: http://olshop-koneksi.test/telescope
(Akses terbatas — hanya admin via Gate::define('viewTelescope'))
```

### 6.2 Yang Dicatat Telescope

```
Telescope Dashboard
├── Requests       ← Semua HTTP request
├── Commands       ← Artisan command
├── Queries        ← Slow database queries (N+1!)
├── Mail           ← Email yang dikirim
├── Notifications  ← Notifikasi
├── Jobs           ← Queue jobs
├── Logs           ← Log entries
├── Dumps          ← dump() calls
└── Exceptions     ← Error yang terjadi
```

### 6.3 Telescope untuk Debugging

```php
// Di kode:
dump($product);  // Telescope tangkap → muncul di Dumps tab
                 // Tidak menghentikan halaman!
```

**Lebih baik dari dd():** Halaman tetap jalan, data bisa dicek nanti.

---

## 7. Debugging Flow

```
Error di halaman?

1. Cek browser DevTools → Console tab
   → Ada 500 error? 404?

2. Cek Whoops page (dev) / error log (prod)
   → Baca message + line number

3. Buka file:line → pahami alur

4. Tambah Log::info() atau dd() sebelum error
   → Apa value variable?

5. Cek Telescope (kalau dev)
   → Lihat query, request, dumps

6. Cek database langsung
   → Data sesuai? Ada yang null?

7. Cek file log
   → tail -f storage/logs/laravel.log
```

---

## 🧪 Latihan

1. **Buat error.** Ubah controller, hapus `$product` variable. Akses halaman. Baca error message.

2. **Pakai dd().** Tambah `dd($product)` di controller. Lihat output.

3. **Pakai Log.** Tambah `Log::info('Shop page accessed', ['user' => auth()->id()])` di controller. Cek log file.

4. **Cek Telescope.** Akses `/telescope` (kalau bisa). Lihat Requests tab.

5. **Baca error log.** `Get-Content storage/logs/laravel.log -Tail 10`. Ada error apa?

---

## 🔗 Referensi

- [Laravel Docs: Logging](https://laravel.com/docs/11.x/logging)
- [Laravel Docs: Telescope](https://laravel.com/docs/11.x/telescope)
- [Laravel Docs: Debugging](https://laravel.com/docs/11.x/helpers#dd)
- Codebase: `storage/logs/laravel.log`
- Codebase: `config/telescope.php`
