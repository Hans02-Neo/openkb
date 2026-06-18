# 05-05: DOM dan Browser API — Halaman Hidup

> **Fase**: 5 — Web Fundamental  
> **Prasyarat**: 05-04-javascript-dasar  
> **Waktu baca**: 65-80 menit  
> **Kata kunci**: DOM, event propagation, event delegation, localStorage, sessionStorage, browser API, DevTools

---

## 📋 Ringkasan

HTML adalah **dokumen statis**. JavaScript membuatnya **hidup** dengan mengakses dan memanipulasi DOM (Document Object Model). Dokumen ini membahas DOM secara mendalam dan berbagai Browser API yang penting.

**Target pemahaman:**
- Kamu paham pohon DOM dan bagaimana browser membangunnya
- Kamu bisa event delegation untuk performa
- Kamu paham localStorage vs sessionStorage
- Kamu bisa menggunakan Browser DevTools secara efektif

---

## 1. DOM — Document Object Model

### 1.1 Apa Itu DOM?

Ketika browser menerima HTML, ia mengurai (parse) teks HTML menjadi **pohon object** yang disebut DOM.

```
HTML:                        DOM Tree:
<html>                       document
  <body>                       └── html
    <h1>Judul</h1>                   └── body
    <p>Halo</p>                        ├── h1 (text: "Judul")
  </body>                              └── p  (text: "Halo")
</html>
```

Setiap **node** di pohon ini adalah object yang bisa diakses dan dimanipulasi via JavaScript.

```javascript
// Mengakses node DOM
const title = document.querySelector('h1');
title.textContent = 'Judul Baru';

// Melihat struktur DOM
console.log(document.body.children);
// HTMLCollection [h1, p]
```

### 1.2 Node Types

| Node Type | Contoh | constant |
|-----------|--------|----------|
| Element node | `<h1>`, `<div>`, `<p>` | `Node.ELEMENT_NODE` (1) |
| Text node | "Halo dunia" | `Node.TEXT_NODE` (3) |
| Comment node | `<!-- comment -->` | `Node.COMMENT_NODE` (8) |
| Document node | `document` | `Node.DOCUMENT_NODE` (9) |

### 1.3 Traversing DOM — Berjalan di Pohon

```html
<div class="card">
    <h2 class="card-title">Judul</h2>
    <p class="card-text">Deskripsi</p>
</div>
```

```javascript
const card = document.querySelector('.card');

// Turun (children)
card.children;              // HTMLCollection [h2, p]
card.firstElementChild;     // h2
card.lastElementChild;      // p

// Naik (parent)
card.parentElement;         // parent element

// Samping (siblings)
card.nextElementSibling;    // element setelah card
card.previousElementSibling; // element sebelum card

// All nodes (include text/comment nodes)
card.childNodes;            // NodeList [text, h2, text, p, text]
```

---

## 2. Event Propagation — Bagaimana Event Bekerja

### 2.1 Tiga Fase Event

Saat kamu klik sebuah tombol, event tidak hanya terjadi di tombol itu — ia **berjalan** melalui pohon DOM dalam 3 fase:

```
                    ┌──────────┐
                    │ document │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │   html    │
                    └────┬─────┘
                         │
                    ┌────▼─────┐       FASE 1: CAPTURING
                    │   body    │       (dari atas ke bawah)
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │   div     │
                    └────┬─────┘
                         │
                    ┌────▼─────┐       FASE 2: TARGET
                    │  button   │       (event sampai di target)
                    └────┬─────┘
                         │
                    ┌────▼─────┐       FASE 3: BUBBLING
                    │   div     │       (dari target naik ke atas)
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │   body    │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │ document │
                    └──────────┘
```

**Capturing** → event turun dari `document` ke target.
**Target** → event sampai di element yang diklik.
**Bubbling** → event naik kembali ke `document`.

```javascript
// Default: event listener di fase bubbling (capture: false)
document.querySelector('div').addEventListener('click', function(e) {
    console.log('div clicked (bubbling)');
});

// Dengan capture: true — event listener di fase capturing
document.querySelector('div').addEventListener('click', function(e) {
    console.log('div clicked (capturing)');
}, true);
```

### 2.2 Event Bubbling — Kenapa Penting?

```html
<div class="card" onclick="console.log('card')">
    <h2 class="title" onclick="console.log('title')">Judul</h2>
</div>
<!-- Klik "Judul" → output: "title" lalu "card" (bubbling naik) -->
```

### 2.3 stopPropagation vs stopImmediatePropagation

```javascript
// stopPropagation — hentikan bubbling (masih jalan listener lain di element yang sama)
element.addEventListener('click', function(e) {
    e.stopPropagation(); // event tidak naik ke parent
});

// stopImmediatePropagation — hentikan semua (termasuk listener lain di element yang sama)
element.addEventListener('click', function(e) {
    e.stopImmediatePropagation(); // hentikan SEMUA
});
```

### 2.4 Event Delegation — Pattern Penting!

Bayangkan kamu punya 100 tombol "delete" di dalam tabel. Daripada attach event listener ke 100 tombol, attach **satu** listener ke parent-nya:

```javascript
// ❌ Buruk — 100 event listener
document.querySelectorAll('.delete-btn').forEach(btn => {
    btn.addEventListener('click', handleDelete);
});

// ✅ Baik — 1 event listener + delegasi
document.querySelector('table').addEventListener('click', function(e) {
    if (e.target.classList.contains('delete-btn')) {
        handleDelete(e);
    }
});
```

**Event delegation bekerja karena bubbling** — klik tombol naik ke `table`, dan kita cek apakah `e.target` adalah tombol yang kita cari.

---

## 3. Browser Storage — Data di Client

### 3.1 localStorage vs sessionStorage vs Cookie

| | localStorage | sessionStorage | Cookie |
|---|-------------|---------------|--------|
| **Usia** | Sampai dihapus manual | Sampai tab ditutup | Sesuai expires |
| **Ukuran** | ~5-10MB | ~5-10MB | ~4KB |
| **Dikirim ke server** | ❌ Tidak | ❌ Tidak | ✅ Ya (setiap request) |
| **Akses** | JavaScript | JavaScript | Server + JS |
| **Guna** | Preferensi user, cache | Data sementara session | Auth token, session ID |

### 3.2 localStorage API

```javascript
// Simpan
localStorage.setItem('theme', 'dark');
localStorage.setItem('cart', JSON.stringify([
    { id: 1, qty: 2 },
    { id: 5, qty: 1 },
]));

// Baca
const theme = localStorage.getItem('theme'); // 'dark'
const cart = JSON.parse(localStorage.getItem('cart')); // array

// Hapus satu
localStorage.removeItem('theme');

// Hapus semua
localStorage.clear();
```

### 3.3 sessionStorage

```javascript
// Sama API-nya dengan localStorage
sessionStorage.setItem('temp', 'data');
const data = sessionStorage.getItem('temp');
sessionStorage.removeItem('temp');
sessionStorage.clear();
```

### 3.4 Kapan Pakai yang Mana?

```javascript
// ✅ Pakai localStorage:
// - Theme preference (dark/light)
// - Cart items (agar tidak hilang saat tab ditutup)
// - Caching API responses

// ✅ Pakai sessionStorage:
// - Data form sementara (agar tidak hilang saat refresh)
// - Wizard/multi-step form state

// ❌ Jangan pakai localStorage/sessionStorage untuk:
// - Data sensitif (password, token) — rentan XSS
// - Data yang perlu diakses server (pakai cookie)
```

---

## 4. Browser API Lainnya

### 4.1 console API

```javascript
console.log('Pesan biasa');
console.info('Info');
console.warn('Peringatan');
console.error('Error');

console.table([{name: 'A'}, {name: 'B'}]); // tabel!
console.group('Group');
console.log('item 1');
console.log('item 2');
console.groupEnd();

console.time('proses');
// ... kode ...
console.timeEnd('proses'); // "proses: 123.45ms"
```

### 4.2 window API

```javascript
// Alert / Confirm / Prompt
alert('Pesan!');
const yes = confirm('Yakin?'); // true/false
const name = prompt('Nama:', 'Guest'); // input string

// Location (URL)
window.location.href;        // URL lengkap
window.location.pathname;    // /shop
window.location.search;      // ?category=laptop
window.location.reload();    // reload halaman
window.location.href = '/';  // redirect

// History
window.history.back();       // kembali
window.history.forward();    // maju

// Screen
window.innerWidth;           // lebar viewport
window.innerHeight;          // tinggi viewport

// Scroll
window.scrollTo(0, 0);       // scroll ke atas
window.scrollBy(0, 100);     // scroll 100px ke bawah
```

### 4.3 document API

```javascript
document.title;              // judul halaman
document.URL;                // URL halaman
document.referrer;           // halaman sebelumnya

document.domain;             // domain
document.cookie;             // semua cookie

document.documentElement;    // <html>
document.body;               // <body>
document.head;               // <head>
```

### 4.4 navigator API

```javascript
navigator.userAgent;         // identitas browser
navigator.language;          // 'id-ID'
navigator.onLine;            // true/false (koneksi internet?)
navigator.geolocation;       // GPS (butuh izin)
```

---

## 5. Browser DevTools — Senjata Utama

### 5.1 Membuka DevTools

| Browser | Shortcut |
|---------|----------|
| Chrome | F12 atau Ctrl+Shift+I |
| Firefox | F12 atau Ctrl+Shift+I |
| Edge | F12 atau Ctrl+Shift+I |

### 5.2 Tab-Tab Penting

| Tab | Guna | Kapan Dipakai |
|-----|------|---------------|
| **Elements** | Lihat & edit HTML/CSS langsung | Debug layout, test styling |
| **Console** | Jalankan JS, lihat log/error | Debug kode JS |
| **Network** | Lihat semua request, response, timing | Debug API, cek status code |
| **Sources** | Lihat semua file JS/CSS | Debug JS dengan breakpoint |
| **Application** | Lihat storage, cookies, cache | Debug localStorage/session |
| **Performance** | Ukur performa halaman | Optimasi loading |

### 5.3 Elements Tab — Debug Layout

Buka Elements → pilih element → lihat Styles di sebelah kanan:

```
Elements tab:
├── HTML tree (klik untuk select)
├── Styles panel:
│   ├── Applied CSS (dari paling spesifik)
│   ├── Filter (cari property tertentu)
│   └── Box model visual
└── Computed panel:
    └── Nilai final (setelah semua CSS dihitung)
```

**Tips:**
- Klik element → Delete untuk sementara hapus
- Double-click text di HTML tree → edit langsung
- Add new style rule (+) di Styles panel → test CSS

### 5.4 Network Tab — Debug Request

```
Network tab:
├── Name — file/request URL
├── Status — 200, 404, 500
├── Type — document, xhr, css, js, img
├── Size — ukuran response
├── Time — durasi request
└── Waterfall — timeline visual
```

**Filter request:**
- `status-code:200` — hanya yang 200
- `method:POST` — hanya POST
- `mime-type:application/json` — hanya JSON

### 5.5 Console Tips

```javascript
// $0 = element yang terpilih di Elements tab
$0.style.background = 'red';

// $$ = querySelectorAll
$$('.card').length;

// Copy object ke clipboard
copy(responseData);

// Monitor events
monitorEvents(window, 'scroll');
```

---

## 6. DOM Performance

### 6.1 Reflow vs Repaint

| Operasi | Guna | Mahal? |
|---------|------|--------|
| **Repaint** | Ubah warna, visibility | Murah |
| **Reflow** | Ubah layout (width, height, position) | **Mahal!** |

```javascript
// ❌ Buruk — 3 reflow!
element.style.width = '100px';
element.style.height = '200px';
element.style.margin = '10px';

// ✅ Baik — 1 reflow!
element.style.cssText = 'width:100px; height:200px; margin:10px;';

// ✅ Lebih baik — classList
element.classList.add('card-large');
```

### 6.2 Document Fragment

Untuk menambah banyak element ke DOM sekaligus:

```javascript
// ❌ Buruk — setiap appendChild trigger reflow!
const list = document.querySelector('ul');
for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    list.appendChild(li);
}

// ✅ Baik — pakai document fragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    fragment.appendChild(li);
}
list.appendChild(fragment); // 1 reflow saja!
```

### 6.3 Debounce dan Throttle

```javascript
// Debounce — tunggu jeda setelah kejadian terakhir
function debounce(fn, delay) {
    let timeout;
    return function(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => fn.apply(this, args), delay);
    };
}

// Gunakan untuk search suggestion
searchInput.addEventListener('input', debounce(function() {
    fetchSuggestions(this.value);
}, 300)); // tunggu 300ms setelah user selesai ngetik

// Throttle — batasi eksekusi maksimal sekali per periode
function throttle(fn, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            fn.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}

// Gunakan untuk scroll handler
window.addEventListener('scroll', throttle(function() {
    checkInfiniteScroll();
}, 200)); // maksimal sekali per 200ms
```

---

## 7. Ringkasan Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│ DOM TREE                                                           │
│  document                                                          │
│    └── html                                                        │
│          ├── head                                                  │
│          └── body                                                  │
│                ├── header → nav → a, a, a                         │
│                ├── main → div.card → h2, p                        │
│                └── footer → p                                      │
├─────────────────────────────────────────────────────────────────────┤
│ EVENT FLOW                                                         │
│  Capturing: document → html → body → div → button                 │
│  Target:    button                                                 │
│  Bubbling:  button → div → body → html → document                 │
├─────────────────────────────────────────────────────────────────────┤
│ BROWSER STORAGE                                                    │
│  localStorage  → permanen, 5MB, client-only                       │
│  sessionStorage → per tab, hilang saat tab ditutup                │
│  Cookie        → 4KB, dikirim ke server tiap request              │
├─────────────────────────────────────────────────────────────────────┤
│ DEVTOLS (F12)                                                      │
│  Elements  → HTML/CSS live edit                                    │
│  Console   → JS REPL, log, error                                  │
│  Network   → request/response timeline                            │
│  Application → storage, cookies                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Latihan

1. **Explore DOM tree.** Buka F12 → Elements. Expand `<body>` → lihat struktur halaman. Klik element, lihat Style panel.

2. **localStorage.** Buka Console dan ketik:
   ```javascript
   localStorage.setItem('test', 'hello');
   localStorage.getItem('test');
   localStorage.removeItem('test');
   ```
   Lalu buka Application → Local Storage → lihat data.

3. **Event delegation.** Di console:
   ```javascript
   document.querySelector('.container').addEventListener('click', function(e) {
       if (e.target.matches('.btn')) {
           console.log('Tombol diklik:', e.target.textContent);
       }
   });
   ```

4. **Network tab.** Buka Network tab → reload halaman. Lihat request pertama (document). Berapa status code-nya? Berapa size-nya?

5. **Debounce search.** Bayangkan search field di shop. Tulis fungsi debounce yang fetch produk setelah user berhenti mengetik 500ms.

---

## 🔗 Referensi

- [MDN: DOM Introduction](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
- [MDN: Event Bubbling](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#event_bubbling)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [Chrome DevTools](https://developer.chrome.com/docs/devtools/)
- Codebase: F12 → Network → lihat request Laravel
