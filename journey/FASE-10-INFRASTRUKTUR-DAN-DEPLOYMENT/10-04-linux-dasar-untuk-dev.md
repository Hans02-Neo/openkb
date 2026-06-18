# 10-04: Linux Dasar untuk Developer

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-03-laragon-di-windows  
> **Waktu baca**: 45-55 menit  
> **Kata kunci**: Linux, Ubuntu, terminal, command line, file system, permission, VPS

---

## 📋 Ringkasan

Sebagian besar server production (VPS, cloud) menggunakan Linux. Sebagai developer Laravel, kamu perlu paham command dasar Linux: navigasi file, permission, process management, dan text editor.

**Target pemahaman:**
- Kamu paham struktur folder Linux
- Kamu bisa command dasar (cd, ls, cp, mv, chmod, nano)
- Kamu bisa manage file permission
- Kamu bisa restart service (PHP-FPM, Nginx)

---

## 1. Kenapa Linux?

```
95% web server = Linux
5%  = Windows Server

Alasan:
- Gratis (open source)
- Stabil (bisa jalan bertahun-tahun tanpa restart)
- Ringan (minimal OS, semua resource untuk aplikasi)
- Tooling (apt, systemctl, cron, ssh)
- Community support
```

---

## 2. Struktur Folder Linux

```
/                   ← Root (beda dengan Windows C:\)
├── etc/            ← Konfigurasi
│   ├── nginx/      ← Nginx config
│   ├── php/        ← PHP config
│   └── mysql/      ← MySQL config
├── var/            ← Variable data
│   ├── www/        ← Web project (html, laravel)
│   ├── log/        ← Log files
│   └── lib/        ← Library
├── home/           ← User home folders
│   └── ubuntu/     ← Home untuk user ubuntu
├── root/           ← Home untuk user root (admin)
├── tmp/            ← Temporary files (dibersihkan otomatis)
└── usr/            ← Installed software
```

**Bandingkan dengan Laragon (Windows):**

| Windows (Laragon) | Linux (Server) |
|-------------------|----------------|
| `C:\laragon\www\olshop-koneksi` | `/var/www/olshop-koneksi` |
| `C:\laragon\bin\php\` | `/etc/php/8.3/` |
| `C:\laragon\etc\apache\` | `/etc/nginx/sites-available/` |

---

## 3. Command Dasar

### 3.1 Navigasi

```bash
pwd                     # Print working directory (lihat folder saat ini)
ls                      # List files
ls -la                  # List semua (termasuk hidden) + detail
cd /var/www             # Pindah folder
cd ~                    # Pindah ke home
cd ..                   # Naik satu level
mkdir project           # Buat folder baru
rm -rf folder           # Hapus folder (hati-hati!)
cp file1 file2          # Copy file
mv file1 folder/        # Pindah / rename file
```

### 3.2 File

```bash
nano file.txt           # Edit file (simple editor)
vim file.txt            # Edit file (advanced, butuh belajar)
cat file.txt            # Lihat isi file di terminal
tail -f file.log        # Follow log (real-time)
less file.txt           # Lihat file scrollable
```

### 3.3 Permission

```bash
# Format permission: rwx rwx rwx
#                    user group other
#                    7    5     5
# r=4, w=2, x=1

chmod 755 file          # rwx r-x r-x (standar file)
chmod 644 file          # rw- r-- r-- (file tidak executable)
chmod -R 775 folder     # Recursive untuk folder

chown user:group file   # Ganti owner
chown -R www-data:www-data /var/www   # web server ownership
```

**Untuk Laravel:**
```bash
# Folder storage & cache harus writable oleh web server
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

---

## 4. Process & Service

```bash
# Service management (systemctl — Ubuntu 16.04+)
sudo systemctl start nginx         # Start Nginx
sudo systemctl stop nginx          # Stop Nginx
sudo systemctl restart nginx       # Restart Nginx
sudo systemctl reload nginx        # Reload config (tanpa restart)
sudo systemctl status nginx        # Cek status

sudo systemctl start php8.3-fpm   # Start PHP-FPM
sudo systemctl restart php8.3-fpm # Restart PHP-FPM
sudo systemctl enable nginx       # Auto start saat boot
```

---

## 5. Package Manager (apt)

```bash
sudo apt update                   # Update daftar package
sudo apt upgrade                  # Upgrade semua package
sudo apt install nginx            # Install Nginx
sudo apt install php8.3-fpm       # Install PHP
sudo apt remove package-name      # Uninstall
```

---

## 6. Text Editor di Terminal

| Editor | Level | Cara Pakai |
|--------|-------|-----------|
| **nano** | Beginner | `nano file.txt` — Ctrl+O simpan, Ctrl+X keluar |
| **vim** | Advanced | `vim file.txt` — butuh belajar (i=insert, :wq=save quit) |

**Rekomendasi:** Mulai dengan **nano** untuk editing cepat.

---

## 🧪 Latihan

1. **Simulasi.** Buka terminal (PowerShell / CMD). Praktikkan: `cd`, `ls`, `pwd`, `mkdir`, `nano` (atau notepad).

2. **Bandingkan.** Cari folder mana di Windows yang setara dengan `/etc/nginx/`, `/var/www/`, `/var/log/`.

3. **Permission analogi.** Di Windows, apa analogi `chmod 777` dan `chown www-data:www-data`?

4. **Cari tahu.** Apa perintah untuk melihat daftar proses di Linux? (Hint: `ps aux` atau `top`)

---

## 🔗 Referensi

- [Linux Command Cheat Sheet](https://www.linuxtrainingacademy.com/linux-commands-cheat-sheet/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [DigitalOcean Linux Basics](https://www.digitalocean.com/community/tutorials/linux-commands)
