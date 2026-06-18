# 11-03: Git & Version Control

> **Fase**: 11 — Tools & Workflow  
> **Prasyarat**: 11-02-npm-dan-nodejs  
> **Waktu baca**: 45-55 menit  
> **Kata kunci**: Git, version control, commit, branch, merge, push, pull, GitHub

---

## 📋 Ringkasan

Git adalah **version control system** — melacak setiap perubahan kode. Setiap developer Laravel WAJIB paham Git: commit, branch, push, pull, merge.

**Target pemahaman:**
- Kamu paham konsep Git (working tree, staging, commit)
- Kamu bisa Git workflow dasar
- Kamu paham branch dan merge
- Kamu bisa baca .gitignore

---

## 1. Git — Konsep Dasar

### 1.1 Three States

```
Working Directory     Staging Area     Git Repository
(file system)        (index)          (commit history)
    │                    │                  │
    ├── git add ────────►│                  │
    │                    ├── git commit ────►│
    │                    │                  │
    └── file berubah ───►│                  │
                         │                  │
                    git status           git log
```

### 1.2 Flow Harian

```bash
# 1. Lihat status
git status

# 2. Tambah file ke staging
git add app/Models/Product.php
git add resources/views/shop/index.blade.php
# atau semua:
git add .

# 3. Commit
git commit -m "Add product model and shop listing"

# 4. Push ke GitHub
git push origin main
```

---

## 2. Branching

### 2.1 Konsep

```
main: ●────●────●────●────●
          \         /
feature:   ●──●──●

main: develop → release → bugfix
feature: fitur baru (branch terpisah)
```

### 2.2 Commands

```bash
# Lihat branch
git branch

# Buat branch baru
git checkout -b feature/wishlist

# Pindah branch
git checkout main

# Merge branch ke main
git checkout main
git merge feature/wishlist

# Delete branch (sudah di-merge)
git branch -d feature/wishlist

# Push branch ke remote
git push origin feature/wishlist
```

---

## 3. Remote (GitHub)

```bash
# Clone repository
git clone https://github.com/user/olshop-koneksi.git

# Lihat remote
git remote -v

# Pull update dari remote
git pull origin main

# Push ke remote
git push origin main

# Push branch baru
git push -u origin feature/wishlist
```

---

## 4. .gitignore

File `gitignore` menentukan file/folder yang **tidak boleh** di-track Git:

```gitignore
# Codebase ini:
.env                    # Kredensial (jangan pernah commit!)
/node_modules           # JS dependencies (bisa install ulang)
/vendor                 # PHP dependencies (bisa install ulang)
/public/build           # Frontend build (bisa build ulang)
/storage/*.key          # Encryption keys
.idea/                  # IDE config
.vscode/                # Editor config
```

---

## 5. Git Workflow

### 5.1 Solo Developer

```
main: ●──●──●──●──●
        (commit langsung di main — OK untuk solo)
```

### 5.2 Team Workflow (GitHub Flow)

```
main: ●──●────────────●──────────●
          \            /          /
feature:   ●──●──●──●            /
                                /
hotfix:   ●────────────────────●
```

---

## 6. Useful Commands

```bash
# Lihat history
git log --oneline --graph --all

# Lihat perubahan (belum di-stage)
git diff

# Lihat perubahan yang sudah di-stage
git diff --staged

# Undo file yang belum di-stage
git checkout -- file.php

# Unstage file
git reset HEAD file.php

# Amend commit terakhir (ubah pesan / tambah file)
git commit --amend

# Stash (simpan perubahan sementara)
git stash
git stash pop
```

---

## 7. Git di Codebase Ini

```bash
# Cek status (apakah ada file berubah?)
git status

# Lihat history
git log --oneline -10

# Lihat siapa yang nulis kode
git blame app/Models/Product.php
```

---

## 🧪 Latihan

1. **Cek status.** Jalankan `git status`. Apa kata Git?

2. **Cek log.** Jalankan `git log --oneline -5`. 5 commit terakhir apa?

3. **Coba branching.** Buat branch `git checkout -b latihan-git`. Tambah file, commit, merge ke `main`.

4. **Baca .gitignore.** Buka `.gitignore`. Pahami kenapa setiap baris di-exclude.

5. **Simulasi.** Jika kamu lupa commit, lalu mengedit file yang sama di 2 tempat — apa yang terjadi saat merge?

---

## 🔗 Referensi

- [Git Official Docs](https://git-scm.com/docs)
- [Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials)
- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)
- Codebase: `.gitignore`
