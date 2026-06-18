# 12-01: Apa Itu Testing

> **Fase**: 12 — Testing & QA  
> **Prasyarat**: FASE 6 (Laravel Deep Dive)  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: testing, QA, manual vs automated, types of testing, TDD, test pyramid

---

## 📋 Ringkasan

Testing adalah proses memverifikasi bahwa kode berjalan sesuai yang diharapkan. Automated testing lebih dapat diandalkan daripada manual testing — dan Laravel memiliki tool testing yang lengkap.

**Target pemahaman:**
- Kamu paham kenapa testing penting
- Kamu bisa bedain jenis testing
- Kamu paham test pyramid
- Kamu bisa bedain manual vs automated

---

## 1. Manual vs Automated

### 1.1 Manual Testing

```
Developer: "Buka browser, klik shop, cek produk..."
           "Ubah status order, refresh, cek lagi..."
           "Isi form checkout, submit..."
           "Ulang 100x setiap ada perubahan..."
```

**Masalah:**
- Membosankan → cepat malas → sering skip
- Tidak konsisten (beda orang beda hasil)
- Tidak scalable (setiap fitur baru → test ulang semua)

### 1.2 Automated Testing

```php
public function test_checkout_creates_order(): void
{
    $response = $this->post('/checkout', [
        'product_id' => 1,
        'quantity' => 2,
    ]);

    $response->assertRedirect('/orders/1');
    $this->assertDatabaseHas('orders', [
        'user_id' => $this->user->id,
    ]);
}
```

**Keuntungan:**
- Jalan otomatis (tinggal `php artisan test`)
- Konsisten (sama persis setiap kali jalan)
- Cepat (ratusan test dalam detik)
- Regression guard (perubahan tidak sengaja merusak fitur lain)

---

## 2. Test Pyramid

```
          ╱╲
         ╱  ╲
        ╱ E2E╲         ← End-to-End (sedikit)
       ╱──────╲
      ╱ Feature╲       ← Feature test (sedang)
     ╱──────────╲
    ╱   Unit     ╲     ← Unit test (banyak)
   ╱──────────────╲
```

| Level | Cepat | Biaya | Contoh |
|-------|-------|-------|--------|
| **Unit** | ⚡ Sangat cepat | Murah | Test service method |
| **Feature** | ⚡ Cepat | Sedang | Test HTTP request |
| **E2E** | 🐢 Lambat | Mahal | Browser automation |

**Best practice:** Banyak unit test, sedikit feature test, lebih sedikit E2E.

---

## 3. Jenis Test di Laravel

| Jenis | Deskripsi | Contoh |
|-------|-----------|--------|
| **Unit test** | Test method/satu kelas | `CartService@getTotal` |
| **Feature test** | Test HTTP request → response | `POST /checkout` |
| **Browser test** (Laravel Dusk) | Test via browser | Klik, isi form, submit |
| **Database test** | Test query/model | `Product::active()` |

---

## 4. Testing di Codebase Ini

```text
tests/
├── Feature/
│   ├── EndToEndCheckoutTest.php    ← E2E checkout flow
│   ├── AdminOrderTest.php          ← Admin update order
│   ├── MidtransWebhookTest.php     ← Payment webhook
│   ├── Auth/                       ← Authentication tests
│   └── ProfileTest.php
├── Unit/
│   └── ExampleTest.php
└── TestCase.php                    ← Base test class
```

Yang sudah ada: **Feature test** untuk flow utama (checkout, admin, auth).

---

## 🧪 Latihan

1. **Run test.** `php artisan test`. Berapa banyak test yang lulus?

2. **Baca test.** Buka `tests/Feature/EndToEndCheckoutTest.php`. Pahami alurnya.

3. **Coba fail.** Ubah kode di controller, jalankan test lagi. Apakah ada yang fail?

4. **Analisis.** Menurut test pyramid, test apa yang kurang di codebase ini?

---

## 🔗 Referensi

- [Laravel Docs: Testing](https://laravel.com/docs/11.x/testing)
- [PHPUnit Docs](https://docs.phpunit.de/)
- [Martin Fowler: Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html)
