# 10-05: SSH & Terminal

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-04-linux-dasar-untuk-dev  
> **Waktu baca**: 40-50 menit  
> **Kata kunci**: SSH, terminal, remote server, key pair, connection, pem, private key

---

## 📋 Ringkasan

SSH (Secure Shell) adalah protokol untuk mengakses server remote dari terminal. Setiap developer Laravel harus bisa SSH ke server untuk deploy, troubleshoot, dan manage aplikasi.

**Target pemahaman:**
- Kamu paham cara kerja SSH
- Kamu bisa konek ke server via SSH
- Kamu paham SSH key (public/private)
- Kamu bisa menjalankan command di server remote

---

## 1. SSH — Cara Kerja

```
Local Computer (Client)                  Server (VPS)
    │                                        │
    ├── 🔑 "Saya punya private key" ────────► │
    │                                        ├── Cek public key
    │                                        ├── Cocok? → izinkan
    │                                        └── Kirim konfirmasi
    │ ◄── "Silakan masuk" ────────────────── │
    │                                        │
    ├── "ls /var/www" ─────────────────────► │
    │                                        ├── Eksekusi command
    │ ◄── "olshop-koneksi" ───────────────── │
    │                                        │
    ├── "exit" ────────────────────────────► │
    │                                        └── Tutup koneksi
```

---

## 2. SSH Key Pair

### 2.1 Generate Key

```bash
# Di terminal lokal (PowerShell / Git Bash)
ssh-keygen -t ed25519 -C "email@example.com"

# Output:
# ~/.ssh/id_ed25519          ← Private key (JANGAN dibagikan!)
# ~/.ssh/id_ed25519.pub      ← Public key (ditaruh di server)
```

### 2.2 Cara Kerja Key

```text
Private key (local)    Public key (server)
   ┌──────┐              ┌──────┐
   │ 🔑   │              │ 🔓   │
   │ rahasia│             │ umum  │
   └──────┘              └──────┘

Kunci private: JANGAN PERNAH dibagikan!
Kunci public: Bisa ditaruh di mana saja (GitHub, server, dll)
```

### 2.3 Copy Public Key ke Server

```bash
# Cara 1: ssh-copy-id
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server-ip

# Cara 2: Manual copy
cat ~/.ssh/id_ed25519.pub
# → Copy output, lalu paste di server:
# nano ~/.ssh/authorized_keys
# atau:
echo "ssh-ed25519 AAAA..." >> ~/.ssh/authorized_keys
```

---

## 3. Koneksi ke Server

### 3.1 Basic Connection

```bash
# Format: ssh user@host
ssh root@192.168.1.100        # Login sebagai root
ssh ubuntu@203.0.113.50       # Login sebagai ubuntu

# Port spesifik (default 22)
ssh -p 2222 user@host
```

### 3.2 Dengan Private Key File

```bash
# Kalau pake .pem file (AWS, DigitalOcean)
ssh -i ~/Downloads/key.pem ubuntu@203.0.113.50

# Atau define di ~/.ssh/config
```

### 3.3 Setelah Masuk Server

```bash
# Kamu dapat terminal Linux server
whoami          # Lihat user
hostname        # Lihat hostname
ls /var/www     # Lihat folder web
php -v          # Cek PHP version
```

---

## 4. SSH Config File

Bikin file `~/.ssh/config` untuk shortcut:

```text
Host olshop-prod
    HostName 203.0.113.50
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host olshop-staging
    HostName 203.0.113.51
    User ubuntu
    Port 2222
```

Koneksi jadi mudah:

```bash
ssh olshop-prod       # Sama dengan ssh ubuntu@203.0.113.50
ssh olshop-staging    # Sama dengan ssh ubuntu@203.0.113.51 -p 2222
```

---

## 5. Common SSH Tasks

### 5.1 Jalankan Command Remote

```bash
# Satu command
ssh olshop-prod "php artisan cache:clear"
ssh olshop-prod "df -h"          # Cek disk usage
ssh olshop-prod "free -m"        # Cek memory

# Multiple commands
ssh olshop-prod "cd /var/www/olshop && git pull && php artisan migrate"
```

### 5.2 Copy Files (SCP)

```bash
# Local → Server
scp file.txt ubuntu@203.0.113.50:/home/ubuntu/

# Server → Local
scp ubuntu@203.0.113.50:/var/log/nginx/error.log ./error.log

# Folder
scp -r ./folder ubuntu@203.0.113.50:/var/www/
```

### 5.3 SSH Tunnel (Port Forwarding)

```bash
# Akses local database (port 3306 via SSH tunnel)
ssh -L 3306:localhost:3306 olshop-prod
# → Buka HeidiSQL di local, konek ke localhost:3306
# → Sebenarnya konek ke database server production!
```

---

## 6. Security Tips

```
✅ Gunakan SSH key (jangan password)
✅ Private key: chmod 600 ~/.ssh/id_ed25519
✅ Jangan login sebagai root — pakai user biasa + sudo
✅ Ganti default port 22 (optional, security by obscurity)
✅ Block IP setelah N failed login (fail2ban)
❌ Jangan bagikan private key ke siapa pun
```

---

## 🧪 Latihan

1. **Generate key.** Jalankan `ssh-keygen -t ed25519` (kalau belum punya).

2. **Cek key.** Buka folder `~/.ssh/` (di PowerShell: `$env:USERPROFILE\.ssh\`). File apa saja yang ada?

3. **Simulasi.** Jika kamu punya VPS Ubuntu, bagaimana cara copy public key ke server?

4. **SCP.** Jika ada file `backup.sql` di server, bagaimana cara download ke lokal?

---

## 🔗 Referensi

- [SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)
- [Laravel Deployment: SSH](https://laravel.com/docs/11.x/deployment)
- Tool: PuTTY (SSH client untuk Windows) — tapi PowerShell built-in sudah support OpenSSH
