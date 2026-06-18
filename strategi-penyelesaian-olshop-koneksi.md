# Strategi Penyelesaian & Finalisasi Koneksi Store
## Panduan Arsitektural untuk Membangun Portofolio Junior Programmer

Dokumen ini dirancang khusus untuk Anda yang sedang belajar pengembangan aplikasi *enterprise-grade* dengan Laravel. Proyek **Koneksi Store** ini sangat ideal sebagai portofolio karena tidak hanya menampilkan antarmuka (UI), tetapi juga menangani logika bisnis yang kompleks seperti gerbang pembayaran (Payment Gateway), perhitungan logistik, dan integritas data pesanan.

Berikut adalah strategi terstruktur dari yang paling **KRITIS** hingga **OPSIONAL**, lengkap dengan alasan arsitekturalnya. Pendekatan ini akan melatih Anda berpikir seperti seorang *Software Engineer* profesional, bukan sekadar pengetik kode.

---

## FASE 1: KRITIS (Integritas Data & Operasional Inti)
Bagian ini wajib dikerjakan pertama karena menyangkut kebenaran data uang dan barang. Aplikasi e-commerce yang baik tidak boleh memiliki cacat di bagian ini.

### 1. Sistem Pencatatan Inventaris (*Inventory Movement Logging*)
*   **Konteks Masalah**: Saat ini, stok dikurangi dengan mengubah langsung nilai di tabel `products`. Di dunia nyata, ini adalah *red flag* bagi auditor. Kita butuh jejak rekam kapan stok masuk, keluar, dan karena alasan apa (terjual, retur, atau penyesuaian manual).
*   **Apa yang Harus Dikerjakan**:
    *   Setiap kali pesanan berhasil dibayar (melalui Webhook Midtrans), selain memotong stok di tabel `products`, sistem harus memasukkan baris baru ke tabel `inventory_movements`.
    *   Saat pesanan dibatalkan dan stok dikembalikan, catat juga di tabel ini.
    *   **Di Admin**: Buat fitur *Manual Stock Adjustment* agar admin bisa menambahkan/mengurangi stok secara manual dengan catatan (misal: "barang rusak", "restock dari supplier").
*   **Nilai Jual di Portofolio**: Menunjukkan bahwa Anda paham mengenai **Data Traceability** (Keterlacakan Data), sebuah konsep krusial di ERP dan sistem finansial.

### 2. Manajemen Pengaturan Toko (*Store Settings Management*)
*   **Konteks Masalah**: Konfigurasi kota asal pengiriman (untuk RajaOngkir) dan detail toko saat ini masih di-*hardcode* di *Controller* atau bergantung pada file `.env`. 
*   **Apa yang Harus Dikerjakan**:
    *   Buat `SettingController` di Admin.
    *   Buat halaman UI agar Admin bisa mengubah Nama Toko, Logo, Kontak, API Key Midtrans (opsional), dan ID Kota Asal Pengiriman tanpa perlu membuka kode program.
    *   Tulis *Service Class* atau *Helper* untuk memuat pengaturan ini ke dalam Cache agar tidak membebani database di setiap *request* halaman.
*   **Nilai Jual di Portofolio**: Membuktikan Anda mampu membuat aplikasi yang dinamis (*dynamic configuration*) dan memahami optimasi *Caching* dengan Redis/File Cache.

---

## FASE 2: PENTING (Nilai Bisnis & UX Pelanggan)
Fase ini berfokus pada fitur yang paling sering ditanyakan oleh klien bisnis saat membeli sistem e-commerce.

### 3. Sistem Kupon & Diskon (*Coupon System*)
*   **Konteks Masalah**: Tabel dan Model sudah ada, namun admin belum bisa membuat kupon, dan pelanggan belum bisa memakainya di halaman Checkout.
*   **Apa yang Harus Dikerjakan**:
    *   **Admin**: Buat `CouponController` untuk CRUD (Create, Read, Update, Delete) kupon dengan atribut: Tipe (Persentase/Nominal), Minimum Belanja, Kuota Penggunaan, dan Masa Berlaku.
    *   **Frontend**: Di halaman Checkout, buat form input kupon. Buat endpoint via AJAX untuk memvalidasi kupon.
    *   **Backend Logic**: Di `CheckoutController@process`, tambahkan logika untuk memotong harga total jika kupon valid, serta catat penggunaan kupon ke tabel `coupon_usages`.
*   **Nilai Jual di Portofolio**: Anda akan mendemonstrasikan pemahaman tentang **Business Rules Validation** (Validasi Aturan Bisnis) yang kompleks.

### 4. Dasbor Pengiriman Khusus (*Shipment Dashboard*)
*   **Konteks Masalah**: Admin saat ini harus membuka detail order satu per satu untuk memasukkan nomor resi pengiriman.
*   **Apa yang Harus Dikerjakan**:
    *   Buat `ShipmentController` di Admin.
    *   Tampilkan daftar semua pesanan yang berstatus `processing` di satu halaman yang ringkas, dengan *input form inline* untuk langsung mengisi nomor resi pengiriman massal.
    *   Saat resi disimpan, status order otomatis berubah menjadi `shipped` dan notifikasi terkirim.
*   **Nilai Jual di Portofolio**: Menunjukkan pemikiran UI/UX yang berpusat pada efisiensi operasional staf (*Operational Excellence*).

---

## FASE 3: OPSIONAL TAPI IMPRESIF (Penyempurnaan Profesional)
Fase ini akan mengubah portofolio Anda dari "Standar" menjadi "Sangat Mengesankan" di mata HRD atau Senior Engineer.

### 5. Modul Laporan & Ekspor (*Reports & Analytics*)
*   **Apa yang Harus Dikerjakan**:
    *   Pisahkan laporan dari Dashboard utama. Buat halaman *Laporan Penjualan*.
    *   Tambahkan *Date Range Picker* (Pilih Rentang Tanggal) agar Admin bisa melihat omset di periode tertentu.
    *   Gunakan *package* seperti `maatwebsite/excel` atau `barryvdh/laravel-dompdf` untuk menambahkan tombol **Export to CSV / PDF**.
*   **Nilai Jual di Portofolio**: Menguasai *Database Aggregation Query* tingkat lanjut dan manipulasi *File Exporting*.

### 6. Implementasi Background Jobs (Queue) untuk Notifikasi
*   **Konteks Masalah**: Mengirim email secara langsung (*synchronous*) saat checkout atau update status akan membuat *loading* halaman menjadi lambat dan berpotensi gagal jika server email bermasalah.
*   **Apa yang Harus Dikerjakan**:
    *   Ubah sistem pengiriman Email Notifikasi menggunakan **Laravel Queue** (Job yang berjalan di *background*).
*   **Nilai Jual di Portofolio**: Kemampuan mengelola *Asynchronous Processing* dan *Message Queues* adalah syarat mutlak untuk skala *Enterprise*.

---

## CARA BELAJAR DAN MENGERJAKAN (Saran untuk Junior Programmer)

1.  **Gunakan Pola "Service Class"**: Jangan letakkan semua logika di *Controller*. Perhatikan bahwa proyek ini menggunakan folder `app/Services/`. Terus gunakan pola ini. Controller hanya bertugas menerima HTTP Request dan mengembalikan Response. Urusan *database* dan perhitungan logika masuk ke Service.
2.  **Kerjakan 1 Fitur Sampai Tuntas (End-to-End)**: Jangan berpindah-pindah. Jika Anda memilih mengerjakan Kupon, selesaikan dari Tabel -> Model -> Controller Admin -> UI Admin -> UI Checkout -> Validasi Backend. Ini melatih pola pikir sistematis.
3.  **Terapkan "Defensive Programming"**: Saat membuat fitur inventaris atau kupon, selalu asumsikan *user* akan melakukan hal yang salah (memasukkan kupon kadaluarsa, memasukkan huruf di form angka). Validasi secara ketat menggunakan Laravel Form Requests.
4.  **Lakukan Commit secara Atomik**: Di Git, lakukan *commit* per fitur. Contoh: `feat: add coupon validation on checkout`. Ini sangat dilihat oleh Senior Developer saat me-review portofolio GitHub Anda.

Dengan mengikuti struktur roadmap di atas, aplikasi **Koneksi Store** Anda tidak hanya akan berfungsi 100%, tetapi kode sumbernya (*source code*) bisa Anda pamerkan untuk membuktikan bahwa Anda memahami arsitektur perangkat lunak yang bersih (*Clean Architecture*).
