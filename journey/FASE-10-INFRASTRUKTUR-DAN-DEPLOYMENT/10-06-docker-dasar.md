# 10-06: Docker Dasar

> **Fase**: 10 — Infrastruktur & Deployment  
> **Prasyarat**: 10-05-ssh-dan-terminal  
> **Waktu baca**: 45-55 menit  
> **Kata kunci**: Docker, container, image, Dockerfile, docker-compose, containerization

---

## 📋 Ringkasan

Docker adalah platform **containerization** — menjalankan aplikasi dalam lingkungan terisolasi. Setiap komponen (PHP, Nginx, MySQL) jalan di container sendiri.

**Target pemahaman:**
- Kamu paham konsep container vs VM
- Kamu bisa membaca Dockerfile dan docker-compose
- Kamu paham kenapa Docker penting untuk deployment

---

## 1. Container vs Virtual Machine

### 1.1 Virtual Machine

```
┌─────────────────────────────────────┐
│ Host OS (Windows/Linux)              │
├─────────────────────────────────────┤
│ Hypervisor                           │
├──────────────┬──────────────┬────────┤
│ Guest OS     │ Guest OS     │ Guest  │
│ (Ubuntu)     │ (Ubuntu)     │ (Cent) │
│ App + Libs   │ App + Libs   │ App    │
│ 1GB+ RAM     │ 1GB+ RAM     │ ...    │
└──────────────┴──────────────┴────────┘
```

### 1.2 Container (Docker)

```
┌─────────────────────────────────────┐
│ Host OS (Linux)                      │
├─────────────────────────────────────┤
│ Docker Engine                        │
├──────────────┬──────────────┬────────┤
│ Container 1  │ Container 2  │ Cont.3 │
│ Nginx        │ PHP-FPM      │ MySQL  │
│ 50MB         │ 100MB        │ 200MB  │
│ (share host  │ (share host  │ share  │
│  kernel)     │  kernel)     │ kernel)│
└──────────────┴──────────────┴────────┘
```

### 1.3 Perbandingan

| | VM | Docker |
|---|-----|--------|
| **Boot time** | Menit | Detik |
| **Size** | GB | MB |
| **Resource** | Heavy (OS penuh) | Lightweight (share kernel) |
| **Isolasi** | Kernel level | Process level |
| **Portability** | Export OVA | Docker image (registry) |

---

## 2. Docker Basic Concepts

### 2.1 Image vs Container

```
Image = blueprint / template (read-only)
Container = instance dari image (running process)

Image: laravel-app:latest (1.2 GB)
    │
    ├── docker run → Container 1 (port 8000)
    ├── docker run → Container 2 (port 8001)
    └── docker run → Container 3 (port 8002)
```

### 2.2 Dockerfile

```dockerfile
# Dockerfile — blueprint image
FROM php:8.3-fpm

# Install dependencies
RUN apt-get update && apt-get install -y \
    git unzip libpng-dev libonig-dev

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Copy aplikasi
COPY . /var/www

# Set working directory
WORKDIR /var/www

# Expose port
EXPOSE 9000
```

### 2.3 docker-compose.yml

```yaml
# docker-compose.yml — orchestration multi-container
services:
  app:
    build: .
    image: olshop-app
    ports:
      - "9000:9000"
    volumes:
      - .:/var/www

  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - .:/var/www
      - ./nginx.conf:/etc/nginx/conf.d/default.conf

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: olshop
    ports:
      - "3306:3306"
```

---

## 3. Common Docker Commands

```bash
# Image
docker images                          # List images
docker pull php:8.3-fpm                # Download image
docker build -t olshop-app .           # Build dari Dockerfile
docker rmi olshop-app                  # Hapus image

# Container
docker ps                              # List running containers
docker ps -a                           # List all containers
docker run -d -p 8000:80 nginx         # Run container (detached)
docker stop container_id               # Stop container
docker start container_id              # Start container
docker rm container_id                 # Hapus container

# Compose
docker-compose up -d                   # Start semua service
docker-compose down                    # Stop + hapus container
docker-compose logs -f                 # Follow logs
docker-compose exec app php -v         # Run command di container
```

---

## 4. Docker untuk Laravel

### 4.1 Development vs Production

**Development (lokal):**
- Laragon lebih simple (Apache + PHP + MySQL built-in)
- Docker: setup Laravel Sail (official Laravel Docker)

**Production:**
- Docker: konsisten antara staging dan production
- Mudah scaling (horizontal)
- Isolasi service

### 4.2 Laravel Sail

```bash
# Kalau mau Docker untuk development
composer require laravel/sail --dev
php artisan sail:install

# Jalankan
./vendor/bin/sail up -d
# alias: sail up -d
```

---

## 5. Kode Ini — Perlu Docker?

Codebase ini pakai **Laragon**, bukan Docker. Tapi untuk deployment:

```
Production:
  Server (VPS) → Nginx + PHP-FPM + MySQL
  (Manual setup — tidak pakai Docker)

  Kalau mau:
  Server (VPS) → Docker → Nginx container + PHP container + MySQL container
  (Lebih konsisten, lebih mudah scaling)
```

---

## 🧪 Latihan

1. **Cek instalasi.** Jalankan `docker --version` (kalau sudah install). Kalau belum, install Docker Desktop.

2. **Pull image.** `docker pull nginx:alpine`. Lihat ukuran image.

3. **Run container.** `docker run -d -p 8080:80 nginx:alpine`. Akses `http://localhost:8080`.

4. **Bandingkan.** Menurutmu, kenapa codebase ini pakai Laragon, bukan Docker?

---

## 🔗 Referensi

- [Docker Official](https://www.docker.com/)
- [Laravel Sail](https://laravel.com/docs/11.x/sail)
- [DigitalOcean: Docker for Laravel](https://www.digitalocean.com/community/tutorials/how-to-set-up-laravel-with-docker-compose-on-ubuntu)
