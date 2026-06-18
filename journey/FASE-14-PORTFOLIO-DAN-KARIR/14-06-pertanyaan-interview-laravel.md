# 14-06: Pertanyaan Interview Laravel

> **Fase**: 14 — Portfolio & Karir  
> **Prasyarat**: 14-05-persiapan-wawancara-teknis  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: interview questions, Laravel, PHP, common questions, junior

---

## 📋 Ringkasan

Kumpulan pertanyaan interview Laravel untuk posisi junior. Pahami konsep, bukan hafalkan jawaban.

**Target pemahaman:**
- Kamu siap menjawab pertanyaan Laravel
- Kamu bisa jelaskan konsep dengan kata sendiri
- Kamu paham pertanyaan trick

---

## 1. Laravel Fundamentals

**Q: Jelaskan alur request Laravel.**

```text
1. Request masuk ke public/index.php (front controller)
2. Bootstrap aplikasi (load container, service provider)
3. Route match (web.php atau api.php)
4. Middleware (auth, role, CSRF)
5. Controller method
6. (Optional) Service → Model → Database
7. Blade view → HTML
8. Response ke browser
```

**Q: Apa perbedaan `php artisan optimize` vs `optimize:clear`?**

```text
optimize:     Config + route + view cache (production)
optimize:clear: Hapus semua cache (development)
```

**Q: Apa itu Service Container?**

```text
Container Laravel yang mengelola dependency injection.
Contoh: class OrderController butuh OrderService,
Laravel auto-inject via constructor.
```

---

## 2. Eloquent ORM

**Q: Apa itu N+1 problem?**

```text
Query: $orders = Order::all();
Loop:  foreach($orders as $order) { $order->user }
→ 1 query untuk orders + N query untuk user = N+1

Solusi: $orders = Order::with('user')->get();
→ 2 query total (eager loading).
```

**Q: Perbedaan `with()` vs `load()`?**

```text
with():  Eager loading — query JOIN (1 call)
load():  Lazy eager loading — query setelah collection ada
```

**Q: Kapan pakai `cursor()` vs `get()`?**

```text
get():     Load semua hasil ke memory (besar)
cursor():  Stream pakai yield (hemat memory, untuk big data)
```

---

## 3. Database & Migration

**Q: Apa itu migration?**

```text
Version control untuk database.
Setiap perubahan schema ada file migration-nya.
Tim: "php artisan migrate" → semua punya struktur DB sama.
```

**Q: `migrate:fresh` vs `migrate:refresh`?**

```text
fresh:  Drop semua tabel + migrate ulang
refresh: Rollback + migrate ulang (data ilang)
Keduanya HAPUS DATA — hati-hati di production!
```

---

## 4. Security

**Q: Apa itu CSRF? Bagaimana Laravel menanganinya?**

```text
CSRF: Cross-Site Request Forgery.
Laravel: @csrf di form, verify token di middleware.
Semua POST/PUT/DELETE butuh CSRF token.
```

**Q: Apa itu Mass Assignment? Gimana mencegahnya?**

```text
Mass assignment: user kirim data yang tidak seharusnya.
Contoh: user kirim is_admin=true di form register.

Cegah: $fillable atau $guarded di Model.
JANGAN pakai Model::create(request()->all()) tanpa filter.
```

---

## 5. Testing

**Q: Perbedaan Unit Test vs Feature Test?**

```text
Unit test: test satu class/method (isolated, mock).
Feature test: test HTTP request → response (integrasi).
Feature test lebih banyak di codebase ini.
```

**Q: Apa itu DatabaseTransactions trait?**

```text
Setiap test di-wrap transaction.
Setelah test selesai → rollback.
Database tetap bersih — tidak perlu manual cleanup.
```

---

## 6. Conceptual

**Q: Kenapa Laravel?**

```text
Ekspresif, elegant syntax. Banyak fitur built-in:
Eloquent, Blade, Artisan, Queue, Mail, Notification.
Ekosistem besar (Forge, Vapor, Telescope).
Komunitas terbesar PHP.
```

**Q: Kelemahan Laravel?**

```text
"Berat" untuk aplikasi sederhana.
Magic methods (__call) kadang bingung.
Upgrade mayor kadang breaking changes.
```

---

## 🧪 Latihan

1. **Jawab lisan.** Tutup dokumen, jawab 5 pertanyaan di atas secara lisan.

2. **Analogikan.** Jelaskan Service Container ke non-developer (pakai analogi restoran).

3. **Deep dive.** Pilih 1 topik (misal: Eloquent). Baca documentation Laravel. Cari 3 hal baru yang kamu belum tahu.

4. **Buat pertanyaan.** Tulis 3 pertanyaan yang akan kamu tanyakan ke calon perusahaan.

---

## 🔗 Referensi

- [Laravel Interview Questions](https://www.laravelinterviewquestions.com/)
- [Laravel Docs](https://laravel.com/docs/11.x/)
- [FASE 3: OOP](/.openkb/journey/FASE-03-OOP/)
- [FASE 6: Laravel Deep Dive](/.openkb/journey/FASE-06-LARAVEL-DEEP-DIVE/)
