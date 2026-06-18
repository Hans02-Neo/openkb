# 12-04: Testing di Codebase Ini

> **Fase**: 12 — Testing & QA  
> **Prasyarat**: 12-03-feature-test  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: codebase testing, existing tests, test patterns, coverage, improve

---

## 📋 Ringkasan

Dokumen ini memetakan **semua test yang ada** di codebase dan polanya. Kamu akan paham apa yang sudah di-test, apa yang belum, dan bagaimana pola penulisan test di proyek ini.

**Target pemahaman:**
- Kamu paham test apa saja yang ada
- Kamu bisa baca dan mengerti pola test
- Kamu bisa menambah test untuk fitur baru
- Kamu paham area yang belum di-test

---

## 1. Existing Tests

```
tests/
├── Feature/
│   ├── Auth/
│   │   ├── AuthenticationTest.php         ✅ Login/logout
│   │   ├── RegistrationTest.php           ✅ Register
│   │   ├── PasswordResetTest.php          ✅ Reset password
│   │   ├── PasswordUpdateTest.php         ✅ Update password
│   │   ├── PasswordConfirmationTest.php   ✅ Confirm password
│   │   └── EmailVerificationTest.php      ✅ Verifikasi email
│   │
│   ├── AdminOrderTest.php                 ✅ Admin update order
│   ├── EndToEndCheckoutTest.php           ✅ Checkout flow
│   ├── MidtransWebhookTest.php            ✅ Payment callback
│   ├── ProfileTest.php                    ✅ Update profile
│   └── ExampleTest.php                    ✅ Example
│
└── Unit/
    └── ExampleTest.php                    ✅ Example
```

---

## 2. Test Patterns

### 2.1 DatabaseTransactions

Semua feature test pakai `DatabaseTransactions`:

```php
class EndToEndCheckoutTest extends TestCase
{
    use DatabaseTransactions;  // ← Semua perubahan di-rollback setelah test
}
```

**Kenapa?** Biar database tetap bersih — tidak perlu manual cleanup.

### 2.2 setUp Method

```php
protected function setUp(): void
{
    parent::setUp();

    // Setup role yang dibutuhkan
    Role::firstOrCreate(['slug' => 'admin'], ['name' => 'Admin']);
}
```

### 2.3 Acting As

```php
// Customer
$user = User::factory()->create();
$this->actingAs($user);

// Admin
$admin = User::factory()->create();
$admin->roles()->attach(Role::where('slug', 'admin')->first());
$this->actingAs($admin);
```

### 2.4 Assertion Pattern

```php
// Response
$response->assertRedirect();
$response->assertSessionHas('success');
$response->assertSessionHasNoErrors();

// Database
$this->assertDatabaseHas('orders', [ ... ]);
$this->assertDatabaseMissing('orders', [ ... ]);
```

---

## 3. Coverage Map

| Fitur | Test | Status |
|-------|------|--------|
| Auth (login, register, password) | ✅ | Complete |
| Profil user | ✅ | Complete |
| Checkout flow | ✅ | Complete |
| Admin order status | ✅ | Complete |
| Midtrans webhook | ✅ | Complete |
| **Cart** (add, remove, total) | ❌ | **Belum ada** |
| **Product** (listing, search) | ❌ | **Belum ada** |
| **Address** (CRUD) | ❌ | **Belum ada** |
| **Order** (history, detail) | ❌ | **Belum ada** |
| **Service layer** (unit test) | ❌ | **Belum ada** |

---

## 4. Unit Test yang Kurang

Service class **belum ada** unit test:

```php
class CartServiceTest extends TestCase
{
    public function test_add_item(): void
    {
        $cart = new CartService($this->createMock(CustomerService::class));
        $cart->addItem(1, 2);
        $this->assertEquals(2, $cart->getItemCount());
    }

    public function test_total_calculation(): void
    {
        $cart = new CartService(...);
        $cart->addItem(1, 3);  // product price = 50000
        $this->assertEquals(150000, $cart->getTotal());
    }
}
```

---

## 5. Menjalankan Test

```bash
# Semua test
php artisan test

# Verbose
php artisan test -v

# Filter
php artisan test --filter="checkout"
php artisan test --filter="admin"
php artisan test --filter="EndToEnd"

# Format (JUnit)
php artisan test --log-junit report.xml
```

---

## 🧪 Latihan

1. **Run all.** `php artisan test`. Berapa test, berapa assertions?

2. **Map test.** Buka setiap file test. Catat: fitur apa yang di-test?

3. **Tambah unit test.** Buat `tests/Unit/CartServiceTest.php` test method `getTotal()`.

4. **Tambah feature test.** Buat test untuk halaman `/shop`:
   - Assert response OK
   - Assert view has `products`
   - Assert halaman menampilkan nama produk

5. **Analisis coverage.** Fitur apa yang paling riskan tapi belum ada testnya?

---

## 🔗 Referensi

- [Laravel: Testing](https://laravel.com/docs/11.x/testing)
- Codebase: `tests/Feature/` — semua feature test
- Codebase: `tests/Unit/` — unit test
