# 12-02: PHPUnit Dasar

> **Fase**: 12 — Testing & QA  
> **Prasyarat**: 12-01-apa-itu-testing  
> **Waktu baca**: 45-55 menit  
> **Kata kunci**: PHPUnit, test case, assertions, setUp, data provider, test suite

---

## 📋 Ringkasan

PHPUnit adalah framework testing untuk PHP. Laravel menggunakan PHPUnit sebagai basis, dengan tambahan helper untuk HTTP testing, database, dan mocking.

**Target pemahaman:**
- Kamu paham struktur test class
- Kamu bisa menulis assertion
- Kamu paham setUp dan tearDown
- Kamu bisa menjalankan test

---

## 1. Test Class Structure

```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class CartServiceTest extends TestCase
{
    public function test_cart_total_is_zero_when_empty(): void
    {
        $total = (new CartService())->getTotal();

        $this->assertEquals(0, $total);
    }
}
```

**Aturan:**
- Class extends `TestCase`
- Method test diawali `test` atau pakai `#[Test]` attribute
- Method `public`, return `void`
- Nama method deskriptif: `test_cart_total_is_zero_when_empty`

---

## 2. Assertions Penting

```php
// Equality
$this->assertEquals(100, $total);

// True / False
$this->assertTrue($user->isAdmin());

// Null / Not Null
$this->assertNull($product->deleted_at);

// Array / Collection
$this->assertCount(3, $items);
$this->assertContains('admin', $roles);

// Exception
$this->expectException(\InvalidArgumentException::class);
$this->expectExceptionMessage('Invalid quantity');
```

---

## 3. setUp & tearDown

```php
use Illuminate\Foundation\Testing\DatabaseTransactions;

class CartServiceTest extends TestCase
{
    use DatabaseTransactions;

    private CartService $cartService;
    private Product $product;

    // Jalan SEBELUM setiap test
    protected function setUp(): void
    {
        parent::setUp();

        $this->cartService = new CartService(
            $this->createMock(CustomerService::class)
        );
        $this->product = Product::factory()->create();
    }

    // Jalan SETELAH setiap test (jarang dipakai)
    protected function tearDown(): void
    {
        // cleanup
        parent::tearDown();
    }

    public function test_add_item(): void
    {
        $this->cartService->addItem($this->product->id, 2);
        $this->assertEquals(2, $this->cartService->getItemCount());
    }
}
```

---

## 4. Data Provider

```php
/**
 * @dataProvider priceProvider
 */
public function test_discount_calculation(int $price, int $expected): void
{
    $result = $this->discountService->apply($price);
    $this->assertEquals($expected, $result);
}

public static function priceProvider(): array
{
    return [
        'below minimum' => [50000, 0],
        'at minimum'     => [100000, 10000],
        'above minimum'  => [150000, 15000],
    ];
}
```

PHP 8.x+ juga bisa pakai attribute:

```php
use PHPUnit\Framework\Attributes\DataProvider;

#[DataProvider('priceProvider')]
public function test_discount(int $price, int $expected): void { ... }
```

---

## 5. Running Tests

```bash
# Semua test
php artisan test

# File spesifik
php artisan test tests/Feature/EndToEndCheckoutTest.php

# Method spesifik
php artisan test --filter test_admin_can_update_order

# Dengan output verbose
php artisan test -v

# Hanya unit / feature
php artisan test --testsuite=Unit
php artisan test --testsuite=Feature
```

---

## 6. phpunit.xml

```xml
<phpunit>
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <php>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value="database/testing.sqlite"/>
        <env name="CACHE_STORE" value="array"/>
    </php>
</phpunit>
```

**Testing environment:** Pakai SQLite + cache array — tidak menyentuh database production.

---

## 🧪 Latihan

1. **Cek config.** Buka `phpunit.xml`. Environment variable apa saja yang diset?

2. **Run all test.** `php artisan test`. Catat hasilnya.

3. **Run feature test saja.** `php artisan test --testsuite=Feature`.

4. **Baca TestCase.** Buka `tests/TestCase.php`. Method apa yang tersedia?

5. **Buat test sederhana.** Buat `tests/Unit/ExampleTest.php` dengan assertion `assertTrue(true)`.

---

## 🔗 Referensi

- [PHPUnit Docs](https://docs.phpunit.de/en/11.5/)
- [Laravel: Testing](https://laravel.com/docs/11.x/testing)
- Codebase: `phpunit.xml`
- Codebase: `tests/TestCase.php`
