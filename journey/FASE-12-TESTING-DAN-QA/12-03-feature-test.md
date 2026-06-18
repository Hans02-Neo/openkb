# 12-03: Feature Test — HTTP & Database

> **Fase**: 12 — Testing & QA  
> **Prasyarat**: 12-02-phpunit-dasar  
> **Waktu baca**: 50-60 menit  
> **Kata kunci**: feature test, HTTP test, database test, actingAs, assertDatabaseHas, factory

---

## 📋 Ringkasan

Feature test menguji **satu fitur lengkap** — dari HTTP request sampai response dan database. Ini adalah jenis test yang paling banyak di codebase ini.

**Target pemahaman:**
- Kamu bisa menulis feature test
- Kamu paham HTTP testing helpers
- Kamu paham database assertions
- Kamu bisa pakai factory

---

## 1. HTTP Testing

### 1.1 HTTP Methods

```php
// GET request
$response = $this->get('/shop');
$response = $this->get(route('shop.index'));

// POST request
$response = $this->post('/cart/add', [
    'product_id' => 1,
    'quantity' => 2,
]);

// PUT / PATCH / DELETE
$response = $this->patch("/orders/{$order->id}", [...]);
$response = $this->delete("/addresses/{$address->id}");
```

### 1.2 Response Assertions

```php
// Status code
$response->assertOk();                  // 200
$response->assertStatus(201);           // Created
$response->assertRedirect();            // 302
$response->assertNotFound();            // 404
$response->assertForbidden();           // 403

// Content
$response->assertSee('Checkout Berhasil');
$response->assertDontSee('Error');
$response->assertJson(['status' => 'success']);

// Session
$response->assertSessionHas('success');
$response->assertSessionHasErrors('email');
$response->assertSessionHasNoErrors();

// View
$response->assertViewHas('products');
$response->assertViewIs('shop.index');
```

---

## 2. Authentication

```php
// Login user
$user = User::factory()->create();
$this->actingAs($user);

// Admin user
$admin = User::factory()->create();
$admin->roles()->attach(Role::where('slug', 'admin')->first());
$this->actingAs($admin);
```

---

## 3. Database Assertions

```php
// Cek data ada di database
$this->assertDatabaseHas('orders', [
    'user_id' => $user->id,
    'status' => 'pending',
]);

// Cek data tidak ada
$this->assertDatabaseMissing('products', [
    'slug' => 'duplicate-slug',
]);

// Cek count
$this->assertDatabaseCount('orders', 1);
```

---

## 4. Factory

```php
// Buat 1 record
$user = User::factory()->create();

// Buat dengan attribute spesifik
$product = Product::factory()->create([
    'price' => 100000,
    'is_active' => true,
]);

// Buat multiple
$products = Product::factory()->count(5)->create();

// Buat dengan relasi
$order = Order::factory()
    ->has(OrderItem::factory()->count(3), 'items')
    ->create(['user_id' => $user->id]);
```

---

## 5. DatabaseTransactions

```php
use Illuminate\Foundation\Testing\DatabaseTransactions;

class EndToEndCheckoutTest extends TestCase
{
    use DatabaseTransactions;

    // Setiap test di-wrap dalam database transaction
    // Otomatis rollback setelah test selesai
    // → Database tetap bersih, tidak perlu manual cleanup
}
```

---

## 6. Contoh Feature Test Lengkap

```php
<?php

namespace Tests\Feature;

use App\Models\Order;
use App\Models\Product;
use App\Models\User;
use App\Models\Address;
use App\Models\Role;
use Illuminate\Foundation\Testing\DatabaseTransactions;
use Tests\TestCase;

class EndToEndCheckoutTest extends TestCase
{
    use DatabaseTransactions;

    protected function setUp(): void
    {
        parent::setUp();
        Role::firstOrCreate(['slug' => 'admin'], ['name' => 'Admin']);
    }

    public function test_end_to_end_checkout_to_shipped()
    {
        // 1. Setup
        $user = User::factory()->create();
        $address = Address::create([...]);
        $product = Product::create([...]);
        $this->actingAs($user);

        // 2. Add to cart
        $response = $this->post(route('cart.add'), [
            'product_id' => $product->id,
            'quantity' => 2,
        ]);
        $response->assertRedirect();
        $response->assertSessionHas('success');

        // 3. Checkout
        $response = $this->post(route('checkout.process'), [
            'address_id' => $address->id,
            'shipping_courier' => 'jne',
            'shipping_service' => 'REG',
            'shipping_cost' => 15000,
        ]);
        $response->assertRedirect();

        // 4. Assert database
        $this->assertDatabaseHas('orders', [
            'user_id' => $user->id,
            'status' => 'pending',
        ]);
    }
}
```

---

## 🧪 Latihan

1. **Baca test.** Buka `tests/Feature/EndToEndCheckoutTest.php`. Trace alur test dari awal sampai akhir.

2. **Baca AdminOrderTest.** Buka `tests/Feature/AdminOrderTest.php`. Perhatikan bagaimana admin dibuat dan digunakan.

3. **Jalankan satu test.** `php artisan test --filter test_admin_can_update_order`.

4. **Buat test sederhana.** Buat test yang akses halaman `/shop` dan assert response OK + viewHas products.

5. **Modifikasi test.** Di AdminOrderTest, tambah assertion `assertDatabaseHas('order_status_histories', [...])` untuk mengecek OrderObserver.

---

## 🔗 Referensi

- [Laravel: HTTP Tests](https://laravel.com/docs/11.x/http-tests)
- [Laravel: Database Testing](https://laravel.com/docs/11.x/database-testing)
- Codebase: `tests/Feature/EndToEndCheckoutTest.php`
- Codebase: `tests/Feature/AdminOrderTest.php`
