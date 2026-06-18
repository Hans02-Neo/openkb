# Laragon `.test` vs `php artisan serve` — Perbandingan untuk Development

## Ringkasan

Anggapan bahwa Laragon `.test` hanya untuk "preview" dan `php artisan serve` lebih baik untuk development adalah **kurang tepat**. Keduanya punya kegunaan masing-masing. Di konteks Laragon, `.test` domain **seharusnya menjadi pilihan utama** untuk development Laravel.

---

## 1. Laragon Auto Virtual Hosts (`olshop-koneksi.test`)

### Cara Kerja

Laragon menggunakan Apache/Nginx asli sebagai web server. Ketika folder `olshop-koneksi` ada di `C:\laragon\www`, Laragon secara otomatis:

1. Menambahkan entry ke `C:\Windows\System32\drivers\etc\hosts`:
   ```
   127.0.0.1 olshop-koneksi.test
   ```
2. Membuat virtual host di Apache/Nginx yang mengarah ke `C:\laragon\www\olshop-koneksi\public`
3. DNS resolver Laragon menangani resolver `.test` di level sistem (tanpa perlu edit `hosts` manual)

### Kelebihan

| Aspek | Penjelasan |
|-------|-----------|
| **Web Server Riil** | Menggunakan Apache/Nginx asli — sama seperti production. Semua fitur web server (mod_rewrite, .htaccess, gzip, caching) berfungsi penuh. |
| **Multi-request** | Apache/Nginx bisa handle banyak request bersamaan (multithreaded). Cocok untuk testing AJAX, fetch, atau concurrent requests. |
| **Isolasi Domain** | Setiap proyek punya domain sendiri (`proyek1.test`, `proyek2.test`). Tidak perlu ribet ganti port. |
| **Persistent** | Server jalan terus di background (via Laragon). Tidak perlu `php artisan serve` setiap mau coding. |
| **Email Catcher** | Laragon integrasi dengan Mailpit untuk catching email di `mailpit.test` |
| **SSL Ready** | Bisa aktifkan HTTPS via Laragon (sertifikat self-signed) |
| **CORS & Cookie** | Domain `.test` adalah TLD valid untuk lokal. Cookie & CORS bekerja seperti di production. |

### Kekurangan

| Aspek | Penjelasan |
|-------|-----------|
| **DNS Harus Running** | DNS resolver Laragon harus aktif (biasanya jalan otomatis di background). Kalau mati, domain `.test` tidak bisa diakses. |
| **APP_URL Wajib Sama** | `APP_URL` di `.env` harus diset `http://olshop-koneksi.test`. Kalau tidak, URL yang digenerate Laravel (route, asset, email link) akan salah. |
| **Heavier** | Apache/Nginx makan lebih banyak resource dibanding PHP built-in server. |
| **Restart Diperlukan** | Perubahan konfigurasi tertentu kadang perlu restart Apache/Nginx dari Laragon. |

---

## 2. `php artisan serve` (http://127.0.0.1:8000)

### Cara Kerja

Menggunakan **PHP Built-in Development Server**. Server sederhana yang langsung jalan dari PHP, tanpa perlu Apache/Nginx.

### Kelebihan

| Aspek | Penjelasan |
|-------|-----------|
| **Sederhana & Ringan** | Satu command langsung jalan. Tanpa dependency web server. Cocok untuk development cepat. |
| **Auto Reload (Hot Reload)** | Bisa dikombinasi dengan `composer run dev` yang juga jalankan Vite + Queue listener secara bersamaan. |
| **Portable** | Tidak perlu setup DNS, hosts file, atau virtual host. Bisa akses dari perangkat lain di jaringan via IP. |
| **Logging Terminal** | Output request langsung terlihat di terminal — HTTP status, durasi, error. |

### Kekurangan

| Aspek | Penjelasan |
|-------|-----------|
| **Single-threaded** | PHP built-in server hanya bisa handle **satu request dalam satu waktu**. Request kedua akan mengantre sampai yang pertama selesai. Ini bisa jadi masalah saat development fitur async/API. |
| **Bukan Web Server Riil** | Banyak fitur web server tidak tersedia (mod_rewrite, .htaccess diabaikan, tidak ada concurrency). **Perilaku bisa berbeda dari production.** |
| **Manual Start/Stop** | Harus dijalankan manual tiap sesi development. Kalau terminal ditutup, server mati. |
| **Sesi Tidak Persisten** | Session/file upload kadang bermasalah karena server built-in PHP tidak optimal untuk file handling. |
| **Port Conflict** | Port 8000 mungkin sudah dipakai aplikasi lain. |
| **URL Berbeda** | Route URL yang digenerate Laravel pakai `127.0.0.1:8000`, sementara di production pakai domain asli. Bisa menyebabkan bug URL. |

---

## 3. Mitos: "Laragon `.test` hanya untuk preview"

Ini tidak benar. Berikut faktanya:

### Kenapa orang menganggap `.test` hanya untuk preview?

1. **Issue DNS Resolver**: Kadang DNS resolver Laragon tidak berfungsi (terutama setelah Windows update atau install ulang). Akibatnya domain `.test` tidak bisa diakses, sehingga developer beralih ke `127.0.0.1:8000`.

2. **`APP_URL` Bermasalah**: Developer lupa mengganti `APP_URL` di `.env` atau set ke `http://127.0.0.1:8000` tapi akses via `.test`. Akibatnya route URL yang digenerate Laravel salah.

3. **Kebiasaan dari Non-Laragon**: Developer yang sebelumnya pakai XAMPP atau manual setup terbiasa dengan `localhost/folder` atau `php artisan serve`.

### Kenapa `.test` sebenarnya LEBIH BAIK untuk development Laravel?

| Skenario | `.test` (Apache/Nginx) | `artisan serve` |
|----------|----------------------|-----------------|
| **Route dengan mod_rewrite** | Berfungsi penuh | Tidak didukung |
| **Upload file besar** | Stabil | Sering timeout/error |
| **Concurrent AJAX request** | OK | Antre (blocking) |
| **Webhook testing (Midtrans)** | OK | Bermasalah (single-threaded) |
| **Cookie-based auth lintas subdomain** | OK | Terbatas |
| **Mendekati production** | Ya | Tidak (beda environment) |

---

## 4. Rekomendasi: Kapan Pakai yang Mana?

### Pakai Laragon `.test` (Apache/Nginx) jika:

- Kamu sedang **mengerjakan fitur yang bergantung pada web server** (rewrite rule, .htaccess, upload file, webhook)
- Kamu ingin **environment sedekat mungkin dengan production**
- Kamu butuh **multitasking** (beberapa request berjalan bersamaan)
- Kamu ingin **server jalan terus** tanpa perlu manual start

### Pakai `php artisan serve` jika:

- Kamu sedang **debugging cepat** dan butuh lihat output log di terminal real-time
- **DNS Laragon bermasalah** dan kamu tidak punya waktu untuk troubleshoot
- Kamu sedang **offline/tidak terhubung ke jaringan** (Laragon DNS kadang butuh network)
- Kamu ingin **development super ringan** tanpa beban Apache/Nginx

### Hybrid Approach (Best of Both Worlds):

```
# Terminal 1: Laragon (Apache/Nginx) untuk serve halaman
# Akses via: http://olshop-koneksi.test

# Terminal 2: npm run dev untuk Vite (hot reload CSS/JS)

# Terminal 3: php artisan queue:listen untuk background job
```

Atau cukup jalankan `composer run dev` yang sudah mengatur semua:

```bash
composer run dev
# => menjalankan: php artisan serve + queue:listen + pail + npm run dev
```

---

## 5. Kesimpulan

| Pertanyaan | Jawaban |
|-----------|---------|
| Apakah `.test` hanya untuk preview? | **Tidak.** `.test` adalah development environment yang legitimate dan lebih mendekati production. |
| Apakah `artisan serve` lebih baik untuk development? | **Tergantung kebutuhan.** Untuk debug cepat mungkin lebih praktis, tapi untuk development fitur serius, `.test` lebih unggul. |
| Mana yang harus saya pakai? | **Utamakan `.test`** karena menggunakan web server riil. Gunakan `artisan serve` sebagai fallback jika DNS bermasalah atau untuk debugging cepat. |

### Catatan Penting

Agar `.test` berfungsi optimal, pastikan:

- **DNS Resolver Laragon menyala** (lihat tray icon Laragon → DNS → harus centang)
- **`APP_URL` di `.env`** sudah diset sesuai domain: `APP_URL=http://olshop-koneksi.test`
- **Apache/Nginx sudah running** di Laragon
- Jika akses via `php artisan serve`, set sementara `APP_URL=http://127.0.0.1:8000` agar URL yang digenerate Laravel sesuai

---

*Dokumen ini dibuat untuk keperluan belajar dan referensi tim Koneksi Store.*
