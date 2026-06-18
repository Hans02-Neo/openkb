# 08-06: Policy & Gate — Authorization Pattern

> **Fase**: 8 — Design Pattern & Arsitektur  
> **Prasyarat**: 08-05-observer-pattern  
> **Waktu baca**: 50-65 menit  
> **Kata kunci**: policy, gate, authorization, ACL, role, permission, Laravel auth

---

## 📋 Ringkasan

Policy dan Gate adalah **pattern authorization** di Laravel. Mereka memisahkan logika "siapa boleh akses apa" dari controller — menjaga controller tetap tipis dan logic terpusat.

**Target pemahaman:**
- Kamu paham perbedaan Policy vs Gate
- Kamu bisa membaca dan membuat policy
- Kamu paham authorization di codebase ini
- Kamu paham hubungan policy dengan role

---

## 1. Masalah: Authorisasi di Mana?

### 1.1 Tanpa Policy (Salah)

```php
// ❌ Logic auth tersebar di controller
class OrderController extends Controller
{
    public function show($id)
    {
        $order = Order::findOrFail($id);

        if (auth()->user()->id !== $order->user_id) {
            abort(403);
        }

        return view('orders.show', compact('order'));
    }
}
```

**Masalah:**
- Duplikasi logic (cek berkali-kali)
- Lupa cek → security hole
- Sulit diubah (ganti aturan → cari semua file)

### 1.2 Dengan Policy (Benar)

```php
// ✅ Logic di satu tempat
class OrderPolicy
{
    public function view(User $user, Order $order): bool
    {
        return $user->id === $order->user_id;
    }
}

class OrderController extends Controller
{
    public function show(Order $order): View
    {
        // Delegasi ke policy
        $this->authorize('view', $order);

        return view('orders.show', compact('order'));
    }
}
```

---

## 2. Policy — CRUD Authorization

### 2.1 Struktur Policy

```
app/Policies/
├── OrderPolicy.php     ← Aturan untuk Order
└── AddressPolicy.php   ← Aturan untuk Address
```

### 2.2 Method Policy Standar

| Method | Arti | Dipanggil via |
|--------|------|--------------|
| `viewAny` | Lihat daftar | `$this->authorize('viewAny', Order::class)` |
| `view` | Lihat detail | `$this->authorize('view', $order)` |
| `create` | Buat baru | `$this->authorize('create', Order::class)` |
| `update` | Edit | `$this->authorize('update', $order)` |
| `delete` | Hapus | `$this->authorize('delete', $order)` |
| `restore` | Restore soft-delete | `$this->authorize('restore', $order)` |
| `forceDelete` | Hapus permanen | `$this->authorize('forceDelete', $order)` |

### 2.3 OrderPolicy — Hanya view

```php
class OrderPolicy
{
    // Lihat detail: hanya pemilik order
    public function view(User $user, Order $order): bool
    {
        return $user->id === $order->user_id;
    }

    // Semua action lain: TIDAK DIIZINKAN
    public function viewAny(User $user): bool   { return false; }
    public function create(User $user): bool     { return false; }
    public function update(User $user, Order $order): bool { return false; }
    public function delete(User $user, Order $order): bool { return false; }
    public function restore(User $user, Order $order): bool { return false; }
    public function forceDelete(User $user, Order $order): bool { return false; }
}
```

### 2.4 AddressPolicy — Full CRUD untuk pemilik

```php
class AddressPolicy
{
    public function view(User $user, Address $address): bool
    {
        return $user->id === $address->user_id;
    }

    public function create(User $user): bool
    {
        return true; // Semua user login bisa buat alamat
    }

    public function update(User $user, Address $address): bool
    {
        return $user->id === $address->user_id;
    }

    public function delete(User $user, Address $address): bool
    {
        return $user->id === $address->user_id;
    }
}
```

---

## 3. Cara Pakai Policy

### 3.1 Di Controller — `$this->authorize()`

```php
use App\Models\Address;

class AddressController extends Controller
{
    public function update(AddressRequest $request, Address $address)
    {
        // Cek: apakah user ini punya akses update address ini?
        $this->authorize('update', $address);

        $address->update($request->validated());

        return redirect()->back()->with('success', 'Alamat diupdate');
    }
}
```

### 3.2 Di Controller — `Gate::authorize()`

```php
use Illuminate\Support\Facades\Gate;

class PaymentController extends Controller
{
    public function success(Request $request, Order $order)
    {
        Gate::authorize('view', $order);

        // hanya pemilik order yang bisa lihat halaman sukses
    }
}
```

### 3.3 Di Blade — `@can`

```blade
@can('update', $address)
    <button>Edit Alamat</button>
@endcan

@cannot('update', $address)
    <p>Kamu tidak bisa edit alamat ini</p>
@endcannot
```

### 3.4 Di Middleware — `can:`

```php
Route::put('/addresses/{address}', [AddressController::class, 'update'])
    ->middleware('can:update,address');
```

### 3.5 Di FormRequest

```php
class UpdateAddressRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('update', $this->route('address'));
    }
}
```

---

## 4. Gate — Closure-Based Authorization

### 4.1 Gate Definition

Policy = untuk model. Gate = untuk aksi umum (bukan model).

```php
// app/Providers/TelescopeServiceProvider.php
Gate::define('viewTelescope', function (User $user) {
    return $user->isAdmin();
});
```

### 4.2 Policy vs Gate

| | Policy | Gate |
|---|--------|------|
| **Target** | Model spesifik | Aksi umum / non-model |
| **Contoh** | `OrderPolicy@view` | `Gate::define('viewTelescope')` |
| **Auto-discovery** | Ya (Laravel 11) | Manual |
| **File** | `app/Policies/*.php` | Di ServiceProvider |

### 4.3 Gate Usage

```php
// Di controller
if (Gate::allows('viewTelescope')) { ... }
if (Gate::denies('viewTelescope')) { abort(403); }
Gate::authorize('viewTelescope');

// Di blade
@can('viewTelescope')
    <a href="/telescope">Debug</a>
@endcan
```

---

## 5. Policy & Role — Bagaimana Hubungannya?

### 5.1 Policy + Role Middleware

Codebase ini pakai **RoleMiddleware** untuk role check:

```php
// app/Http/Middleware/RoleMiddleware.php
class RoleMiddleware
{
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user() || ! $request->user()->hasRole($role)) {
            abort(403, 'Unauthorized action.');
        }
        return $next($request);
    }
}
```

### 5.2 Layering Authorisasi

```
1. Route Middleware: role:admin → apakah user adalah admin?
2. Policy:          dapatPolicy('view', $order) → apakah user punya akses?
```

Contoh:

```php
// Route: hanya admin bisa akses route ini
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::get('/orders/{order}', [OrderController::class, 'show'])
        ->can('view', 'order'); // policy check tambahan
});
```

### 5.3 Policy dengan Role

```php
class OrderPolicy
{
    public function viewAny(User $user): bool
    {
        // Admin bisa lihat semua order
        return $user->hasRole('admin');
    }

    public function view(User $user, Order $order): bool
    {
        // Admin lihat semua, customer hanya punya sendiri
        return $user->hasRole('admin')
            || $user->id === $order->user_id;
    }

    public function update(User $user, Order $order): bool
    {
        // Hanya admin bisa update order
        return $user->hasRole('admin');
    }
}
```

---

## 6. Authorization di Codebase Ini

### 6.1 Ringkasan

| File | Isi |
|------|-----|
| `app/Policies/OrderPolicy.php` | Hanya `view` — user lihat order sendiri |
| `app/Policies/AddressPolicy.php` | `view`, `create`, `update`, `delete` — user punya sendiri |
| `Gate::define('viewTelescope')` | Telescope hanya untuk admin |
| `RoleMiddleware` | Middleware role check |

### 6.2 Flow: Lihat Order

```
GET /orders/123
    │
    ▼
Route: Route::get('/orders/{order}', [OrderController::class, 'show'])
    │
    ▼
OrderController@show(Order $order)
    │
    ▼
$this->authorize('view', $order)
    │
    ▼
OrderPolicy@view(User $user, Order $order)
  → return $user->id === $order->user_id
  → true  → lanjut
  → false → abort(403)
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ AUTHORIZATION PATTERN (Policy & Gate)                              │
├─────────────────────────────────────────────────────────────────────┤
│  POLICY (Model-based)              GATE (Closure-based)            │
│  ┌──────────────────────┐          ┌────────────────────────┐      │
│  │ OrderPolicy           │          │ Gate::define(          │      │
│  │  ├── view(User,Order)│          │   'viewTelescope',     │      │
│  │  ├── create(User)    │          │   fn($user) =>         │      │
│  │  └── delete(User,Ord)│          │     $user->isAdmin()   │      │
│  └──────────────────────┘          │   )                    │      │
│                                    └────────────────────────┘      │
│  AddressPolicy                                                     │
│  ├── view, create, update, delete                                  │
│                                                                     │
│  CARA PAKAI:                                                        │
│  Controller: $this->authorize('update', $address)                  │
│  Gate facade: Gate::authorize('view', $order)                      │
│  Blade:       @can('update', $address)                             │
│  Middleware:  ->middleware('can:update,address')                   │
├─────────────────────────────────────────────────────────────────────┤
│ DI CODEBASE INI                                                     │
│  2 policies: OrderPolicy, AddressPolicy                            │
│  1 gate: Telescope access                                          │
│  1 middleware: RoleMiddleware                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Baca OrderPolicy.** Buka `app/Policies/OrderPolicy.php`. Method apa yang return `true`? Kenapa yang lain `false`?

2. **Baca AddressPolicy.** Bandingkan dengan OrderPolicy. Kenapa AddressPolicy punya lebih banyak akses?

3. **Cek penggunaan.** Cari `$this->authorize()` dan `Gate::authorize()` di controller. Cocokkan dengan policy yang ada.

4. **Buat policy baru.** Buat `ProductPolicy` dengan:
   - `viewAny`: semua user
   - `view`: semua user
   - `create`: hanya admin
   - `update`: hanya admin
   - `delete`: hanya admin

5. **Role + Policy.** Di `OrderPolicy@view`, tambahkan logika: admin bisa lihat semua order, customer hanya order sendiri.

---

## 🔗 Referensi

- [Laravel Docs: Authorization](https://laravel.com/docs/11.x/authorization)
- [Laravel Docs: Gates](https://laravel.com/docs/11.x/authorization#gates)
- [Laravel Docs: Policies](https://laravel.com/docs/11.x/authorization#creating-policies)
- Codebase: `app/Policies/OrderPolicy.php`
- Codebase: `app/Policies/AddressPolicy.php`
- Codebase: `app/Http/Middleware/RoleMiddleware.php`
