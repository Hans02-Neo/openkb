# 11-06: IDE & Produktivitas

> **Fase**: 11 — Tools & Workflow  
> **Prasyarat**: 11-05-tinker-dan-repl  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: IDE, editor, productivity, PHPStorm, VS Code, Laravel Idea, snippets

---

## 📋 Ringkasan

IDE (Integrated Development Environment) yang baik meningkatkan produktivitas developer secara signifikan. Untuk Laravel, pilihan utama adalah PHPStorm atau VS Code + extensions.

**Target pemahaman:**
- Kamu paham fitur IDE yang penting
- Kamu bisa setup VS Code untuk Laravel
- Kamu paham shortcut dan workflow

---

## 1. IDE Options

| IDE | Harga | Weight | Laravel Support |
|-----|-------|--------|-----------------|
| **PHPStorm** | Berbayar (~$199/thn) | Heavy | ✅ Native (Laravel Idea plugin) |
| **VS Code** | Gratis | Light | ✅ Extensions |
| **Sublime Text** | $99 (one-time) | Light | ⚠️ Manual |
| **Cursor** | Berbayar | Medium | ✅ AI-powered |

**Rekomendasi:** VS Code (gratis, ringan) atau PHPStorm (powerful, khusus PHP).

---

## 2. VS Code Extensions untuk Laravel

```
✅ Wajib:
├── Laravel Extension Pack      ← Bundle extension Laravel
├── PHP Intelephense            ← Autocomplete, type hinting
├── Laravel Blade Snippets      ← Snippets untuk Blade
├── Laravel Snippets            ← Snippets untuk PHP Laravel
├── Tailwind CSS IntelliSense   ← Autocomplete Tailwind class
├── Alpine.js IntelliSense      ← Autocomplete Alpine directive
├── EditorConfig               ← Konsistensi coding style
└── GitLens                    ← Git blame inline

🔧 Opsional:
├── phpcs / Pint               ← Code style checker
├── Laravel goto view          ← Navigasi ke view dari controller
├── Laravel Model Snippets     ← Model relationship snippets
└── Error Lens                 ← Tampilkan error inline
```

---

## 3. Fitur IDE Penting

### 3.1 Intelephense (Autocomplete)

```php
// Ketik:
Product::whe
// → Autocomplete: Product::where, Product::whereIn, Product::whereHas

// Ketik:
$product->bel
// → Autocomplete: $product->belongsTo, $product->belongsToMany
```

### 3.2 Go to Definition

```
F12 (VS Code) / Ctrl+B (PHPStorm):
  Klik Product → langsung ke app/Models/Product.php
  Klik ShopController → langsung ke controller file
  Klik shop.index → langsung ke resources/views/shop/index.blade.php
```

### 3.3 Find Usages

```
Shift+F12:
  Cari semua file yang pake CartService
  Cari semua file yang pake Product::ACTIVE
  Cari semua tempat yang panggil method getTotal()
```

### 3.4 Refactoring

```php
// Rename (F2):
// Ganti $product jadi $item → semua referensi otomatis berubah

// Extract Method:
// Pilih blok kode → Ctrl+Shift+R → Extract Method
```

---

## 4. Laravel Idea (PHPStorm Plugin)

PHPStorm + Laravel Idea = superpower:

```php
// ✅ Route autocomplete di controller
// ✅ Model relationship autocomplete
// ✅ Blade directive autocomplete
// ✅ Generate migration dari model
// ✅ Navigasi route → controller → view
// ✅ Live templates (Ctrl+J):
//   - route → Route::get(...)
//   - model → Model + migration + factory + seeder
//   - policy → Policy class + register
```

---

## 5. Keyboard Shortcuts

### 5.1 VS Code

| Shortcut | Action |
|----------|--------|
| `Ctrl+P` | Cari file |
| `Ctrl+Shift+P` | Command palette |
| `F12` | Go to definition |
| `Shift+F12` | Find references |
| `Alt+↑/↓` | Pindah baris |
| `Ctrl+D` | Select next occurrence |
| `Ctrl+/` | Comment/uncomment |
| `Ctrl+\`` | Toggle terminal |
| `Ctrl+B` | Toggle sidebar |

### 5.2 PHPStorm

| Shortcut | Action |
|----------|--------|
| `Ctrl+N` | Cari class |
| `Ctrl+Shift+N` | Cari file |
| `Ctrl+B` | Go to definition |
| `Alt+F7` | Find usages |
| `Ctrl+Alt+L` | Reformat code |
| `Ctrl+Alt+Shift+L` | Reformat + optimize imports |

---

## 6. EditorConfig

```ini
# .editorconfig — konsistensi editor
root = true

[*]
indent_style = space
indent_size = 4
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.blade.php]
indent_size = 4
```





---

## 🧪 Latihan

1. **Cek extension.** Buka VS Code, cek extension yang terinstall. Apakah ada Laravel extension?

2. **Go to definition.** Buka `ShopController.php`. Klik `Product` → F12. File apa yang terbuka?

3. **Snippets.** Coba ketik `route` di controller → muncul snippet untuk Route::get().

4. **Cari file.** `Ctrl+P` → ketik `OrderService`. Cepat sampai ke file.

5. **Reformat.** Pilih semua kode (Ctrl+A) → format (Shift+Alt+F). Apakah format berubah?

---

## 🔗 Referensi

- [VS Code](https://code.visualstudio.com/)
- [PHPStorm](https://www.jetbrains.com/phpstorm/)
- [Laravel Idea](https://laravel-idea.com/)
- [EditorConfig](https://editorconfig.org/)
- Codebase: `.editorconfig`
