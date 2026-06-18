# 11-05: Tinker & REPL

> **Fase**: 11 — Tools & Workflow  
> **Prasyarat**: 11-04-artisan-cli  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: Tinker, REPL, interactive shell, Laravel Tinker, PsySH, debugging

---

## 📋 Ringkasan

Tinker adalah **REPL (Read-Eval-Print Loop)** untuk Laravel. Kamu bisa menulis PHP langsung di terminal — akses database, test logic, debug — tanpa harus buka browser.

**Target pemahaman:**
- Kamu paham konsep REPL
- Kamu bisa pakai Tinker
- Kamu bisa test query / logic di Tinker
- Kamu bisa troubleshoot pakai Tinker

---

## 1. REPL — Apa Itu?

```
REPL = Read → Eval → Print → Loop

Baca input → Evaluasi → Cetak hasil → Ulang

Contoh:
> 2 + 2
= 4

> Product::count()
= 15

> auth()->user()
= App\Models\User {#123 ...}
```

---

## 2. Masuk Tinker

```bash
php artisan tinker
```

```text
Psy Shell v0.12.x (PHP 8.3.x — ⚕️)
> 
```

Dari sini kamu bisa nulis **PHP apa saja** — seperti di kode.

---

## 3. Common Tinker Commands

### 3.1 Query Data

```bash
> Product::count()
= 15

> Product::first()
= App\Models\Product {#123
    id: 1,
    name: "Kemeja Flanel",
    price: 125000,
  }

> Product::where('price', '>', 100000)->get()
= Illuminate\Database\Eloquent\Collection {#456
    all: [
      App\Models\Product {#...},
      App\Models\Product {#...},
    ],
  }

> Product::with('category')->find(1)
```

### 3.2 Test Logic

```bash
> $cart = app(App\Services\CartService::class)
= App\Services\CartService {#...}

> $cart->addItem(1, 2)
= null

> $cart->getTotal()
= 250000

> $cart->clear()
```

### 3.3 Auth / User

```bash
> $user = User::find(1)
> auth()->login($user)
> auth()->user()
> auth()->user()->hasRole('admin')
```

### 3.4 Debug Helper

```bash
> $product = Product::find(1)
> $product->load('category', 'media')
> $product->toArray()
> $product->toJson()
```

---

## 4. Tinker untuk Troubleshoot

### 4.1 Cek Config

```bash
> config('app.env')
= "local"

> config('midtrans.server_key')
= "Mid-server-xxx"

> config('database.connections.mysql')
```

### 4.2 Cek Cache

```bash
> Cache::get('key')
> Cache::put('key', 'value', 3600)
> Cache::forget('key')
```

### 4.3 Cek Error

```bash
> try {
    OrderService::createOrder([]);
} catch (\Exception $e) {
    echo $e->getMessage();
}
```

---

## 5. Tips Tinker

```bash
# Keluar Tinker
> Ctrl+C atau exit

# History (akses command sebelumnya)
> ↑↓ panah

# Dapatkan help object
> help Product

# Evaluasi file
> eval(file_get_contents('test.php'))
```

---

## 🧪 Latihan

1. **Masuk Tinker.** `php artisan tinker`. Coba `Product::count()`, `User::count()`, `Order::count()`.

2. **Test query.** Ambil produk pertama: `Product::first()`. Apa saja field-nya?

3. **Test service.** Panggil `CartService`: `$cart = app(App\Services\CartService::class)`. Coba method-methodnya.

4. **Create data.** `Product::create(['name' => 'Test', 'price' => 10000])`. Cek di database.

5. **Error handling.** Coba panggil method yang tidak ada. Apa yang terjadi? Bagaimana cara lihat error message?

---

## 🔗 Referensi

- [Laravel Docs: Tinker](https://laravel.com/docs/11.x/artisan#tinker)
- [PsySH (Tinker engine)](https://psysh.org/)
- Codebase: `php artisan tinker` (jalankan langsung)
