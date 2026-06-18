# 13-03: Convention di Codebase

> **Fase**: 13 — Code Quality  
> **Prasyarat**: 13-02-psr-standard  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: convention, consistency, codebase pattern, Laravel convention, team standard

---

## 📋 Ringkasan

Setiap codebase punya **convention** sendiri — pola spesifik yang konsisten di seluruh proyek. Convention ini kadang tidak tertulis (tapi terlihat dari kode yang ada).

**Target pemahaman:**
- Kamu paham convention codebase ini
- Kamu bisa mengikuti pola yang ada
- Kamu bisa bedakan convention vs PSR
- Kamu paham Laravel convention

---

## 1. Codebase Convention

| Area | Convention | Contoh |
|------|-----------|--------|
| **Controller** | Tipis, delegasi ke Service | `CartController → CartService` |
| **Service** | Constructor injection, method spesifik | `CartService::addItem()` |
| **Model** | `$fillable`, `$casts`, relationships | `Product::belongsTo(Category::class)` |
| **View** | Blade components, Tailwind utility | `@extends`, `@section`, `@vite` |
| **Route** | Route groups + name prefix | `Route::prefix('admin')->name('admin.')` |
| **Validation** | FormRequest class | `CheckoutRequest`, `ProductRequest` |
| **Authorization** | Policy class + Gate | `OrderPolicy`, `AddressPolicy` |
| **Middleware** | RoleMiddleware untuk role check | `->middleware('role:admin')` |

---

## 2. Naming Convention

### 2.1 Route

```php
// Pattern: resource.name + action
Route::get('/shop', [ShopController::class, 'index'])->name('shop.index');
Route::get('/shop/{product}', [ShopController::class, 'show'])->name('shop.show');

Route::post('/cart/add', [CartController::class, 'addItem'])->name('cart.add');
Route::delete('/cart/remove/{product}', [CartController::class, 'removeItem'])->name('cart.remove');
```

### 2.2 Controller Method

```php
// Resource pattern
index()     → GET /resource
create()    → GET /resource/create
store()     → POST /resource
show()      → GET /resource/{id}
edit()      → GET /resource/{id}/edit
update()    → PUT/PATCH /resource/{id}
destroy()   → DELETE /resource/{id}
```

Untuk non-resource:

```php
// Verb + Noun (jelas)
addItem()
removeItem()
updateStatus()
processCheckout()
```

### 2.3 Database

```php
// Table: snake_case, plural
Schema::create('order_items', function (Blueprint $table) { ... });

// Pivot: alphabetical order
'category_product'  // bukan product_category
'order_item'        // bukan item_order
```

---

## 3. File Structure

```
app/
├── Http/
│   ├── Controllers/      ← Controller (tipis)
│   ├── Middleware/        ← Middleware (auth, role)
│   └── Requests/          ← FormRequest (validasi)
├── Models/                ← Eloquent Model
├── Services/              ← Business logic
├── Policies/              ← Authorization
└── Observers/             ← Model observers

resources/
├── views/
│   ├── layouts/           ← Layout blade
│   ├── components/        ← Shared components
│   ├── shop/              ← Shop pages
│   ├── admin/             ← Admin pages
│   └── auth/              ← Auth pages
└── css/
    └── app.css            ← Tailwind + custom components
```

---

## 4. Convention Checklist

```text
Ketika menambah fitur baru:

□ Ikuti pola resource controller (index, create, store, show, edit, update, destroy)
□ Buat Service class untuk logika bisnis (jika > 1 langkah)
□ Buat FormRequest untuk validasi (jika > 2 field)
□ Buat Policy untuk authorization
□ Gunakan constructor injection
□ Route name prefix yang jelas
□ Blade extends layout yang sesuai (app / guest / admin)
□ @vite untuk CSS/JS
□ .env.example update kalau ada konfigurasi baru
```

---

## 🧪 Latihan

1. **Baca routing.** Buka `routes/web.php`. Identifikasi 3 pola route yang konsisten.

2. **Cek controller.** Bandingkan `ShopController` dan `CartController`. Mana yang resource pattern? Mana yang custom?

3. **Cek views.** Buka folder `resources/views/shop/` dan `resources/views/admin/`. Apakah pola folder konsisten?

4. **Tambah fitur.** Jika kamu tambah fitur "Wishlist", file apa saja yang perlu dibuat? (ikuti convention codebase)

---

## 🔗 Referensi

- [Laravel: Convention](https://laravel.com/docs/11.x/controllers#resource-controllers)
- Codebase: `routes/web.php` (routing pattern)
- Codebase: `app/Http/Controllers/` (controller pattern)
- Codebase: `app/Services/` (service pattern)
