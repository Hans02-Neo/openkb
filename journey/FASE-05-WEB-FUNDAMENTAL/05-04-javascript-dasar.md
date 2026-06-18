# 05-04: JavaScript Dasar — Otak dari Frontend

> **Fase**: 5 — Web Fundamental  
> **Prasyarat**: 05-03-html-css-dasar  
> **Waktu baca**: 70-85 menit  
> **Kata kunci**: JavaScript, ES6, variable, function, object, array, event, fetch, async, await, JSON

---

## 📋 Ringkasan

HTML = struktur, CSS = tampilan, **JavaScript = perilaku**. JS membuat halaman web menjadi interaktif — validasi form, animasi, fetch data dari server tanpa reload (AJAX).

**Target pemahaman:**
- Kamu paham sintaks dasar JavaScript
- Kamu bisa memanipulasi DOM
- Kamu bisa melakukan fetch API
- Kamu paham async/await

---

## 1. JavaScript vs PHP — Perbedaan Kunci

| Aspek | PHP | JavaScript |
|-------|-----|-----------|
| **Jalan di** | Server (backend) | Browser (frontend) |
| **Guna** | Logika, database, API | Interaksi, animasi, DOM |
| **Loading** | Sebelum halaman dikirim | Setelah halaman sampai di browser |
| **Syntax** | Mirip C, ada `$` | Mirip C, tanpa `$` |
| **Types** | Loosely typed | Loosely typed |
| **Array** | `array()` / `[]` | `[]` |
| **Object** | `['key' => 'val']` (array) | `{key: 'val'}` (object literal) |

---

## 2. Sintaks Dasar JavaScript

### 2.1 Variable

```javascript
// var — OLD (function scope) — jangan pakai!
var nama = 'Budi';

// let — bisa diubah (block scope) — PAKAI INI!
let umur = 25;
umur = 26; // boleh

// const — tidak bisa diubah (block scope) — PAKAI INI!
const pi = 3.14;
// pi = 3; // ❌ Error!

// const untuk object — referensi tidak berubah, isi bisa
const user = { nama: 'Budi' };
user.nama = 'Siti'; // ✅ isi berubah
// user = { nama: 'Siti' }; // ❌ Error — reassign tidak boleh
```

### 2.2 Tipe Data

```javascript
// String
const nama = 'Budi';
const pesan = `Halo ${nama}`; // template literal (backtick!)

// Number
const harga = 15000;
const diskon = 0.1;

// Boolean
const aktif = true;
const admin = false;

// Null / Undefined
const kosong = null;       // sengaja kosong
let tidakDidefinisikan;    // undefined (belum diisi)

// Array
const products = ['Laptop', 'Mouse', 'Keyboard'];
products[0]; // 'Laptop'
products.push('Monitor'); // tambah

// Object
const product = {
    name: 'Laptop',
    price: 15000000,
    stock: 5,
    isActive: true,
};
product.name;       // 'Laptop'
product['price'];   // 15000000
```

### 2.3 Operator

```javascript
// Aritmatika — sama dengan PHP
+ - * / % **
let total = 100 + 50; // 150

// Perbandingan
==  // sama nilai (5 == '5' → true) — HINDARI!
=== // sama nilai DAN tipe (5 === '5' → false) — PAKAI INI!
!=  // tidak sama nilai
!== // tidak sama nilai ATAU tipe

// Logika
&& // AND
|| // OR
!  // NOT

// Conditional (ternary)
const status = umur >= 17 ? 'Dewasa' : 'Anak-anak';
```

### 2.4 Conditional

```javascript
const nilai = 85;

if (nilai >= 90) {
    console.log('A');
} else if (nilai >= 80) {
    console.log('B');
} else {
    console.log('C');
}

// Switch
switch (nilai) {
    case 100: console.log('Sempurna!'); break;
    default:  console.log('Terus belajar');
}
```

### 2.5 Loops

```javascript
// For
for (let i = 0; i < 5; i++) {
    console.log(i); // 0, 1, 2, 3, 4
}

// For...of (array)
const items = ['a', 'b', 'c'];
for (const item of items) {
    console.log(item);
}

// For...in (object)
const user = { name: 'Budi', age: 25 };
for (const key in user) {
    console.log(`${key}: ${user[key]}`);
}

// While
let count = 0;
while (count < 3) {
    console.log(count);
    count++;
}

// Array methods (functional style)
products.forEach(product => console.log(product));
const names = products.map(p => p.name);
const cheap = products.filter(p => p.price < 50000);
const total = products.reduce((sum, p) => sum + p.price, 0);
```

---

## 3. Function di JavaScript

### 3.1 Function Declaration vs Expression

```javascript
// Declaration (hoisted — bisa dipanggil sebelum deklarasi)
function tambah(a, b) {
    return a + b;
}

// Expression (tidak hoisted)
const kali = function(a, b) {
    return a * b;
};

// Arrow function (ES6) — PAKAI INI!
const kurang = (a, b) => {
    return a - b;
};

// Arrow function — satu baris (implisit return)
const bagi = (a, b) => a / b;

// Arrow function — satu parameter
const double = n => n * 2;
```

### 3.2 Function Default Parameter

```javascript
function greet(name = 'Guest') {
    return `Hello, ${name}!`;
}

console.log(greet('Budi')); // 'Hello, Budi!'
console.log(greet());       // 'Hello, Guest!'
```

### 3.3 Callback

```javascript
// Fungsi yang menerima fungsi lain
function processData(data, callback) {
    const result = data.map(callback);
    return result;
}

const numbers = [1, 2, 3];
const doubled = processData(numbers, n => n * 2);
// [2, 4, 6]

// Contoh nyata — event listener
document.getElementById('btn').addEventListener('click', function() {
    alert('Tombol diklik!');
});
```

---

## 4. DOM Manipulation — Mengubah HTML dengan JS

### 4.1 Select Element

```javascript
// Single element
const title = document.getElementById('title');
const firstCard = document.querySelector('.card'); // CSS selector

// Multiple elements
const cards = document.getElementsByClassName('card'); // HTMLCollection
const cards = document.querySelectorAll('.card');       // NodeList (forEach bisa!)
```

### 4.2 Modify Element

```javascript
const title = document.querySelector('h1');

// Content
title.textContent = 'Judul Baru';           // teks saja (aman)
title.innerHTML = '<span>Judul Baru</span>'; // HTML (hati-hati XSS!)

// Attributes
title.classList.add('highlight');
title.classList.remove('old');
title.classList.toggle('active'); // tambah jika tidak ada, hapus jika ada

title.style.color = 'red';
title.style.fontSize = '24px'; // camelCase (bukan font-size)

// Create element
const newCard = document.createElement('div');
newCard.className = 'card';
newCard.textContent = 'Card Baru';
document.querySelector('.container').appendChild(newCard);
```

### 4.3 Event Listeners

```javascript
const button = document.querySelector('#submit-btn');

button.addEventListener('click', function(event) {
    event.preventDefault(); // cegah default (misal reload form)
    console.log('Tombol diklik!');
});

// Event umum:
// click       → klik mouse
// submit      → form disubmit
// keydown     → tombol keyboard ditekan
// change      → nilai input berubah
// mouseover   → mouse di atas elemen
// scroll      → scroll halaman
// load        → halaman selesai loading
```

### 4.4 Form Handling

```javascript
const form = document.querySelector('#checkout-form');

form.addEventListener('submit', function(event) {
    event.preventDefault();

    const formData = new FormData(form);
    const data = Object.fromEntries(formData);

    // Validasi client-side
    if (!data.email) {
        alert('Email wajib diisi!');
        return;
    }

    // Kirim ke server via fetch
    fetch('/checkout', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        },
        body: JSON.stringify(data),
    });
});
```

---

## 5. Fetch API — AJAX Modern

### 5.1 Apa Itu Fetch?

**Fetch** adalah cara modern browser untuk melakukan request HTTP dari JavaScript — tanpa reload halaman.

Dulu: `XMLHttpRequest` (ribet).
Sekarang: `fetch()` (sederhana, Promise-based).

### 5.2 GET Request

```javascript
fetch('/api/products')
    .then(response => response.json()) // parse JSON
    .then(data => {
        console.log(data); // array of products
    })
    .catch(error => {
        console.error('Error:', error);
    });
```

### 5.3 POST Request

```javascript
fetch('/checkout', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-TOKEN': csrfToken,
    },
    body: JSON.stringify({
        customer_id: 1,
        items: [
            { product_id: 5, quantity: 2 },
        ],
    }),
})
    .then(response => response.json())
    .then(data => {
        console.log('Order created:', data);
    })
    .catch(error => {
        console.error('Error:', error);
    });
```

### 5.4 Async/Await — Lebih Bersih

```javascript
// Daripada .then() chain, pakai async/await:

async function loadProducts() {
    try {
        const response = await fetch('/api/products');
        const products = await response.json();
        console.log(products);
    } catch (error) {
        console.error('Error:', error);
    }
}

loadProducts(); // panggil fungsi async
```

### 5.5 Contoh di Konteks Codebase

Misal kita punya tombol "Tambah ke Cart" yang pakai AJAX:

```javascript
async function addToCart(productId, quantity) {
    try {
        const response = await fetch('/cart/add', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
                'Accept': 'application/json',
            },
            body: JSON.stringify({
                product_id: productId,
                quantity: quantity,
            }),
        });

        const result = await response.json();

        if (response.ok) {
            // Update cart badge
            document.querySelector('#cart-count').textContent = result.cartCount;
            alert('Produk ditambahkan ke cart!');
        } else {
            alert(result.message || 'Gagal menambahkan ke cart');
        }
    } catch (error) {
        console.error('Network error:', error);
        alert('Terjadi kesalahan jaringan');
    }
}
```

---

## 6. JSON — Bahasa Data Client-Server

### 6.1 Apa Itu JSON?

JSON (JavaScript Object Notation) adalah format pertukaran data paling populer di web.

```json
{
    "name": "Laptop Gaming",
    "price": 15000000,
    "stock": 5,
    "is_active": true,
    "tags": ["elektronik", "laptop"],
    "category": {
        "id": 1,
        "name": "Elektronik"
    }
}
```

### 6.2 JSON di PHP vs JavaScript

```php
// PHP
$data = ['name' => 'Laptop', 'price' => 15000];
$json = json_encode($data);    // jadi JSON string
$array = json_decode($json, true); // JSON → array
```

```javascript
// JavaScript
const data = { name: 'Laptop', price: 15000 };
const json = JSON.stringify(data);    // jadi JSON string
const obj = JSON.parse(json);         // JSON → object
```

### 6.3 JSON di Response Laravel

```php
// Controller mengembalikan JSON:
return response()->json([
    'success' => true,
    'data' => $product,
]);
```

Hasiinya di browser:
```json
{"success": true, "data": {"id": 1, "name": "Laptop"}}
```

---

## 7. JavaScript di Codebase

### 7.1 File JS

```
resources/js/
├── app.js      → Bootstrap JS, initializations
├── bootstrap.js → Bootstrap setup
└── cart.js     → Cart logic (kalau ada)
```

### 7.2 Menambahkan JavaScript ke Blade

```html
{{-- Via Vite (Laravel 11 default) --}}
@vite(['resources/js/app.js'])

{{-- Via asset helper (jika sudah compiled) --}}
<script src="{{ asset('js/app.js') }}"></script>

{{-- Script langsung (hanya untuk kecil) --}}
<script>
    document.addEventListener('DOMContentLoaded', function() {
        console.log('Halaman siap!');
    });
</script>

{{-- @push untuk script spesifik halaman --}}
@push('scripts')
<script>
    // Cart page specific JS
    document.querySelector('#update-cart').addEventListener('click', function() {
        // ...
    });
</script>
@endpush
```

---

## 8. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ JAVASCRIPT FUNDAMENTALS                                            │
├─────────────────────────────────────────────────────────────────────┤
│ Variable:  let, const (jangan var!)                                │
│ Function:  const fn = (params) => { ... }  (arrow function)       │
│ Array:     map, filter, reduce, forEach                            │
│ Object:    const obj = { key: 'value' };  obj.key                 │
├─────────────────────────────────────────────────────────────────────┤
│ DOM:                                                               │
│  document.querySelector('.class') → select                         │
│  element.textContent = '...'       → modify                        │
│  element.addEventListener('click') → events                        │
│  element.classList.add('active')   → CSS classes                   │
├─────────────────────────────────────────────────────────────────────┤
│ FETCH / AJAX:                                                      │
│  async function loadData() {                                       │
│      const res = await fetch('/api/data');                        │
│      const data = await res.json();                              │
│  }                                                                 │
├─────────────────────────────────────────────────────────────────────┤
│ JSON:                                                              │
│  JSON.stringify(obj)  → JSON string                               │
│  JSON.parse(jsonStr)  → JavaScript object                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Console.** Buka `olshop-koneksi.test/shop`, F12 → Console. Ketik:
   ```javascript
   document.querySelector('h1').textContent
   // → lihat judul halaman
   ```

2. **Manipulasi DOM.** Di console yang sama, ketik:
   ```javascript
   document.querySelectorAll('.card-title').forEach(el => {
       el.style.color = 'red';
   });
   ```
   Lihat semua judul card berubah merah!

3. **Fetch.** Coba fetch RajaOngkir API (via proxy Laravel jika ada endpoint-nya):
   ```javascript
   fetch('/api/cities')
       .then(r => r.json())
       .then(d => console.log(d));
   ```

4. **AJAX Cart.** Buka `resources/views/cart/index.blade.php`. Cari event listener atau form submit. Apakah cart menggunakan form submit biasa atau AJAX?

5. **Async function.** Tulis fungsi async yang:
   ```javascript
   async function getProduct(id) {
       const res = await fetch(`/api/products/${id}`);
       return await res.json();
   }
   getProduct(1).then(console.log);
   ```

---

## 🔗 Referensi

- [MDN: JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: DOM Manipulation](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Manipulating_documents)
- [JavaScript.info](https://javascript.info/)
- Codebase: `resources/js/app.js`
- Codebase: `resources/views/cart/index.blade.php` — frontend cart
