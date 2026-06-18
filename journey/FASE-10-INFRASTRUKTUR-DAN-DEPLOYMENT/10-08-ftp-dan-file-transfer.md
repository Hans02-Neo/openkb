# 10-08: FTP & File Transfer

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-07-cpanel-dan-hosting  
> **Waktu baca**: 35-45 menit  
> **Kata kunci**: FTP, SFTP, FTPS, FileZilla, transfer file, File Manager, deployment

---

## 📋 Ringkasan

FTP (File Transfer Protocol) adalah cara mentransfer file antara lokal dan server. Meskipun sudah ada Git untuk deployment, FTP masih berguna untuk upload file besar, backup, atau akses ke shared hosting.

**Target pemahaman:**
- Kamu paham FTP vs SFTP
- Kamu bisa menggunakan FTP client (FileZilla)
- Kamu paham kapan pakai FTP vs Git

---

## 1. FTP vs SFTP vs FTPS

| Protokol | Port | Security | Kapan Pakai |
|----------|------|----------|-------------|
| **FTP** | 21 | ❌ Plain text | Jangan pakai (tidak aman) |
| **FTPS** | 990 (implicit) / 21 (explicit) | ✅ SSL/TLS | Shared hosting support |
| **SFTP** | 22 (pakai SSH) | ✅ SSH | VPS, modern hosting |

**Rekomendasi:** Selalu pakai **SFTP** kalau bisa. Hanya pakai FTP(S) kalau hosting tidak support SSH.

---

## 2. FTP Client — FileZilla

### 2.1 Setup Connection

```
Host:      203.0.113.50  (atau domain: server.com)
Port:      22            (SFTP) / 21 (FTP)
Protocol:  SFTP          (lebih aman)
User:      ubuntu        (atau username hosting)
Password:  ********

Atau pakai SSH key:
  Edit → Settings → SFTP → Add key file
  Pilih ~/.ssh/id_ed25519
```

### 2.2 Interface

```
┌─────────────────┬──────────────────────────────┐
│ Local (PC)      │ Remote (Server)              │
├─────────────────┼──────────────────────────────┤
│ C:\laragon\www\ │ /var/www/                    │
│   olshop-koneksi│   olshop-koneksi             │
│   ├── app/      │   ├── app/                   │
│   ├── config/   │   ├── config/                │
│   ├── ...       │   ├── ...                    │
│   └── .env      │   └── .env                   │
├─────────────────┴──────────────────────────────┤
│ Drag & drop untuk upload/download              │
└────────────────────────────────────────────────┘
```

### 2.3 Common Operations

```
Upload:  Drag dari kiri (local) ke kanan (remote)
Download: Drag dari kanan (remote) ke kiri (local)
Sync:    Select folder →右键 → Upload / Download

Permission: 右键 file → File Permissions → 755 / 644
```

---

## 3. FTP vs Git

| Situasi | Pakai FTP | Pakai Git |
|---------|-----------|-----------|
| **First deploy** | Upload semua file | `git clone` |
| **Update minor** | Upload file berubah | `git pull` |
| **File besar** (gambar) | ✅ FTP | ❌ Git (repo besar) |
| **.env file** | ✅ FTP (rahasia) | ❌ Jangan commit .env |
| **Backup** | ✅ Download via FTP | ✅ Push ke GitHub |
| **Collaboration** | ❌ Overwrite risk | ✅ Branch + merge |
| **Rollback** | ❌ Manual backup | ✅ `git revert` |
| **Production** | Emergency fix | Daily workflow |

---

## 4. File Manager (cPanel)

Selain FTP, cPanel punya **File Manager** — akses file via browser:

```
cPanel → File Manager
├── Upload file (max 2GB via browser)
├── Extract ZIP (upload zip, extract di server)
├── Edit file (browser-based text editor)
├── Permission (ubah 755, 644)
├── Copy / Move / Rename / Delete
└── Compress (download folder sebagai zip)
```

**Kapan pakai File Manager?**
- Tidak punya FTP client
- Edit cepat (misal: .env, config)
- Upload sekali-sekali

---

## 5. Security

```
✅ SFTP (port 22) — aman, enkripsi penuh
✅ FTPS (port 990) — aman kalau SSL valid
❌ FTP (port 21) — password + data plain text!

Tips:
- Ganti default port kalau bisa (22 → 2222)
- Gunakan key-based auth (SSH key) bukan password
- Limit IP yang bisa FTP (firewall)
- Jangan simpan password FTP di file teks
```

---

## 🧪 Latihan

1. **Cek FTP.** Kalau punya hosting, coba konek via FileZilla (atau WinSCP).

2. **File Manager.** Buka cPanel → File Manager. Cari folder `public_html`. Apa isinya?

3. **Upload test.** Upload file `test.txt` via FTP ke server. Akses via browser. Hapus setelahnya.

4. **Bandingkan.** Kapan kamu akan pilih Git deploy vs FTP upload untuk update code?

---

## 🔗 Referensi

- [FileZilla Client](https://filezilla-project.org/)
- [WinSCP (alt. FTP client)](https://winscp.net/)
- [cPanel File Manager](https://docs.cpanel.net/cpanel/files/file-manager/)
