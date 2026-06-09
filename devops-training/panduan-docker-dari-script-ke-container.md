# Panduan Lengkap Docker
**Dari Kumpulan Script → Image → Container → Backup → Pindah Server**

---

## Gambaran Besar Alur

```
FOLDER SCRIPT/APLIKASI
      |
      | [1] Tulis Dockerfile
      v
DOCKERFILE + SOURCE CODE
      |
      | [2] docker build
      v
DOCKER IMAGE  (blueprint, tersimpan lokal)
      |
      | [3] docker run / docker compose up
      v
DOCKER CONTAINER  (aplikasi berjalan)
      |
      | [4] Backup image + volume
      v
FILE BACKUP (.tar + .tar.gz)
      |
      | [5] Transfer ke server baru
      v
SERVER BARU  (docker load + restore volume)
      |
      | [6] docker compose up
      v
CONTAINER BERJALAN DI SERVER BARU
```

---

## Tahap 1 — Siapkan Struktur Folder Aplikasi

Contoh: Aplikasi web Python (Flask) + Database PostgreSQL

```
/root/my-app/                        ← folder project utama
│
├── docker-compose.yml               ← orkestrasi semua container
├── .env                             ← variabel sensitif (password, dsb)
│
├── app/                             ← folder source code aplikasi
│   ├── Dockerfile                   ← instruksi build image app
│   ├── requirements.txt             ← daftar library Python
│   ├── main.py                      ← kode utama aplikasi
│   └── templates/
│       └── index.html               ← file HTML
│
└── nginx/                           ← folder config web server
    ├── Dockerfile                   ← instruksi build image nginx
    └── nginx.conf                   ← konfigurasi nginx
```

| File | Fungsi |
|------|--------|
| `Dockerfile` | Resep untuk membuat Docker Image |
| `docker-compose.yml` | Menjalankan & menghubungkan banyak container |
| `.env` | Menyimpan variabel (tidak di-commit ke git) |
| `requirements.txt` | Daftar dependency/library yang dibutuhkan |
| `main.py` | Source code aplikasi utama |
| `nginx.conf` | Konfigurasi reverse proxy |

---

## Tahap 2 — Tulis Dockerfile

Dockerfile adalah file instruksi untuk membangun image. Setiap baris instruksi = 1 layer baru di dalam image.

### Contoh: `app/Dockerfile` (Python Flask)

```dockerfile
# Baris 1: Pilih base image (OS + runtime sudah tersedia)
FROM python:3.11-slim

# Baris 2: Set direktori kerja di dalam container
WORKDIR /app

# Baris 3: Copy file dependency dulu (agar layer ini di-cache)
COPY requirements.txt .

# Baris 4: Install dependency
RUN pip install --no-cache-dir -r requirements.txt

# Baris 5: Copy seluruh source code ke dalam image
COPY . .

# Baris 6: Buka port yang digunakan aplikasi
EXPOSE 5000

# Baris 7: Perintah yang dijalankan saat container start
CMD ["python", "main.py"]
```

### Contoh: `nginx/Dockerfile`

```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

### Cara Kerja Layer

Setiap instruksi Dockerfile menghasilkan layer yang di-cache. Jika kode berubah (`COPY . .`) hanya layer itu dan di bawahnya yang direbuild — layer di atasnya tetap pakai cache sehingga build lebih cepat.

```
FROM python:3.11-slim    → Layer 0 (base OS + Python)
WORKDIR /app             → Layer 1 (set workdir)
COPY requirements.txt    → Layer 2 (file dependency)
RUN pip install          → Layer 3 (library terinstall)  ← di-cache!
COPY . .                 → Layer 4 (source code)         ← rebuild jika kode berubah
EXPOSE 5000              → metadata (bukan layer)
CMD [...]                → metadata (bukan layer)
```

---

## Tahap 3 — Tulis docker-compose.yml

Docker Compose menjalankan dan menghubungkan banyak container sekaligus.

### Contoh: `/root/my-app/docker-compose.yml`

```yaml
version: '3.8'

services:

  # Container 1: Database
  db:
    image: postgres:16
    container_name: myapp-db
    restart: always
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASS}
      POSTGRES_DB: myapp
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - myapp-network

  # Container 2: Aplikasi
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: myapp-app
    restart: always
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASS}@db:5432/myapp
    volumes:
      - app_uploads:/app/uploads
    ports:
      - "5000:5000"
    depends_on:
      - db
    networks:
      - myapp-network

  # Container 3: Web Server
  nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    container_name: myapp-nginx
    restart: always
    ports:
      - "80:80"
    depends_on:
      - app
    networks:
      - myapp-network

volumes:
  db_data:
  app_uploads:

networks:
  myapp-network:
    driver: bridge
```

### Contoh: `/root/my-app/.env`

```env
DB_USER=myappuser
DB_PASS=r4hasi4Ku!
```

---

## Tahap 4 — Build Image

```bash
cd /root/my-app

# Build image untuk app (tanpa docker-compose)
docker build -t myapp-app:1.0 ./app

# Build semua service via docker compose
docker compose build

# Build ulang tanpa cache (jika ada perubahan besar)
docker compose build --no-cache
```

### Proses yang terjadi saat build:

1. Docker baca Dockerfile baris per baris
2. Tiap instruksi dijalankan dalam container sementara
3. Hasilnya di-snapshot menjadi layer baru
4. Semua layer digabung menjadi 1 image
5. Image disimpan di `/var/lib/docker/overlay2/`

```bash
# Verifikasi image berhasil dibuat
docker image ls

# Output:
# REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
# myapp-app     1.0       a1b2c3d4e5f6   2 minutes ago   245MB
# myapp-nginx   latest    b2c3d4e5f6a7   2 minutes ago   43MB
# postgres      16        c3d4e5f6a7b8   2 weeks ago     438MB
```

---

## Tahap 5 — Jalankan Container

```bash
cd /root/my-app

# Jalankan semua container di background
docker compose up -d

# Pantau proses startup
docker compose logs -f

# Cek semua container berjalan
docker ps
```

### Urutan yang terjadi:

1. Docker buat network: `myapp-network`
2. Docker buat volume: `db_data`, `app_uploads`
3. Container `db` (postgres) dijalankan pertama (`depends_on`)
4. Container `app` dijalankan setelah db siap
5. Container `nginx` dijalankan setelah app siap
6. Port `80` dan `5000` terbuka di host

### Topologi Container yang Berjalan

```
HOST SERVER (/root/my-app/)
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Network: myapp-network (bridge)                               │
│                                                                │
│  ┌──────────────┐    ┌──────────────┐    ┌─────────────┐      │
│  │ myapp-nginx  │    │  myapp-app   │    │  myapp-db   │      │
│  │ nginx:alpine │───>│ myapp-app:1.0│───>│ postgres:16 │      │
│  │ port: 80     │    │ port: 5000   │    │ port: 5432  │      │
│  │ (proxy)      │    │ (Flask API)  │    │ (internal)  │      │
│  └──────────────┘    └──────┬───────┘    └──────┬──────┘      │
│         ↑                   │                   │             │
│  Port 80 terbuka      app_uploads          db_data            │
│  ke luar              (volume)             (volume)           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
        |
        v  Akses dari browser
  http://IP-SERVER
```

---

## Tahap 6 — Verifikasi Container Berjalan

```bash
# Lihat semua container yang running
docker ps

# Log spesifik satu container
docker logs myapp-app
docker logs myapp-db
docker logs myapp-nginx

# Masuk ke dalam container (debugging)
docker exec -it myapp-app bash
docker exec -it myapp-db psql -U myappuser -d myapp

# Monitor penggunaan resource
docker stats

# Detail konfigurasi container
docker inspect myapp-app

# Lihat volume dan network
docker volume ls
docker network ls
```

---

## Tahap 7 — Backup (Image + Volume + Config)

Sebelum backup, pastikan data konsisten:

```bash
cd /root/my-app
docker compose down      # hentikan semua container
docker ps                # verifikasi semua sudah stop

mkdir -p /root/backup-myapp
cd /root/backup-myapp
```

### 7A. Backup Docker Images

```bash
docker save -o myapp-app.tar    myapp-app:1.0
docker save -o myapp-nginx.tar  myapp-nginx:latest
docker save -o postgres-16.tar  postgres:16

# Verifikasi
ls -lh /root/backup-myapp/*.tar
```

### 7B. Backup Docker Volumes (Data Aplikasi)

```bash
# Backup volume database (PALING PENTING)
docker run --rm \
  -v myapp_db_data:/source \
  -v /root/backup-myapp:/backup \
  alpine tar czf /backup/vol-db-data.tar.gz -C /source .

# Backup volume uploads
docker run --rm \
  -v myapp_app_uploads:/source \
  -v /root/backup-myapp:/backup \
  alpine tar czf /backup/vol-app-uploads.tar.gz -C /source .
```

> **Catatan:** Nama volume = `<nama_project>_<nama_volume>`. Cek nama aktual dengan `docker volume ls`.

### 7C. Backup File Konfigurasi

```bash
cp -r /root/my-app /root/backup-myapp/my-app-config
```

### 7D. Verifikasi Semua File Backup

```bash
ls -lh /root/backup-myapp/

# Output yang diharapkan:
# myapp-app.tar           (image aplikasi)
# myapp-nginx.tar         (image nginx)
# postgres-16.tar         (image database)
# vol-db-data.tar.gz      (data database)
# vol-app-uploads.tar.gz  (data upload)
# my-app-config/          (folder config lengkap)
```

### 7E. Transfer ke Server Baru

```bash
# Rsync (lebih cepat, bisa resume jika putus)
rsync -avz --progress /root/backup-myapp/ user@IP_SERVER_BARU:/root/backup-myapp/

# SCP
scp -r /root/backup-myapp/ user@IP_SERVER_BARU:/root/backup-myapp/

# Compress dulu lalu kirim
tar czf /root/backup-myapp-full.tar.gz /root/backup-myapp/
scp /root/backup-myapp-full.tar.gz user@IP_SERVER_BARU:/root/
```

---

## Tahap 8 — Restore dan Jalankan di Server Baru

### 8A. Install Docker di Server Baru

```bash
apt update && apt install -y ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update
apt install -y docker-ce docker-ce-cli containerd.io \
               docker-buildx-plugin docker-compose-plugin

systemctl enable docker
systemctl start docker

# Verifikasi
docker --version
docker compose version
```

### 8B. Load Docker Images

```bash
cd /root/backup-myapp

docker load -i myapp-app.tar
docker load -i myapp-nginx.tar
docker load -i postgres-16.tar

# Verifikasi
docker image ls
```

### 8C. Buat Volume dan Restore Data

```bash
# Buat volume (nama harus sama persis dengan server lama)
docker volume create myapp_db_data
docker volume create myapp_app_uploads

# Restore database
docker run --rm \
  -v myapp_db_data:/target \
  -v /root/backup-myapp:/backup \
  alpine tar xzf /backup/vol-db-data.tar.gz -C /target

# Restore uploads
docker run --rm \
  -v myapp_app_uploads:/target \
  -v /root/backup-myapp:/backup \
  alpine tar xzf /backup/vol-app-uploads.tar.gz -C /target

# Verifikasi isi volume
ls /var/lib/docker/volumes/myapp_db_data/_data/
ls /var/lib/docker/volumes/myapp_app_uploads/_data/
```

### 8D. Restore Konfigurasi

```bash
cp -r /root/backup-myapp/my-app-config /root/my-app

# Pastikan file .env ada dan benar
cat /root/my-app/.env
```

### 8E. Jalankan Container

```bash
cd /root/my-app
docker compose up -d

# Pantau log startup
docker compose logs -f

# Cek status
docker ps
```

### 8F. Verifikasi Aplikasi Berjalan

```bash
curl -I http://localhost
# HTTP/1.1 200 OK  →  berhasil!

# Akses dari browser
# http://IP_SERVER_BARU
```

---

## Diagram Alur Lengkap End-to-End

```
SERVER LAMA                           TRANSFER              SERVER BARU
──────────────────────────            ────────           ──────────────────

/root/my-app/
├── Dockerfile
├── docker-compose.yml
└── app/main.py
        |
        | docker compose build
        v
┌──────────────────┐
│  Docker Images   │
│  myapp-app:1.0   │
│  myapp-nginx     │
│  postgres:16     │
└────────┬─────────┘
         |
         | docker compose up -d
         v
┌──────────────────┐
│Docker Containers │
│  myapp-nginx     │
│  myapp-app       │──> Volume: app_uploads
│  myapp-db        │──> Volume: db_data
└────────┬─────────┘
         |
         | docker compose down
         | docker save → *.tar
         | tar czf     → *.tar.gz
         | cp config   → folder
         v
┌──────────────────┐
│  Backup Files    │  ── SCP / Rsync ──>  ┌──────────────────┐
│  myapp-app.tar   │                      │  Backup Files    │
│  postgres-16.tar │                      │  (sama persis)   │
│  vol-db.tar.gz   │                      └────────┬─────────┘
│  vol-uploads.gz  │                               |
│  my-app-config/  │                               | docker load
└──────────────────┘                               | volume restore
                                                   | docker compose up
                                                   v
                                          ┌──────────────────┐
                                          │Docker Containers │
                                          │  myapp-nginx     │
                                          │  myapp-app       │
                                          │  myapp-db        │
                                          │  (data lengkap)  │
                                          └────────┬─────────┘
                                                   |
                                          http://IP_SERVER_BARU
```

---

## Checklist Migrasi

### Server Lama — Backup
- [ ] `docker compose down`
- [ ] `docker save` → image app
- [ ] `docker save` → image nginx
- [ ] `docker save` → image postgres
- [ ] `tar czf` → backup volume db_data
- [ ] `tar czf` → backup volume app_uploads
- [ ] `cp` → copy folder config (termasuk `.env`)
- [ ] Verifikasi semua file backup ada
- [ ] Transfer semua file ke server baru

### Server Baru — Restore
- [ ] Install Docker + Docker Compose
- [ ] `docker load` → semua image
- [ ] `docker volume create` → semua volume
- [ ] `tar xzf` → restore volume db_data
- [ ] `tar xzf` → restore volume app_uploads
- [ ] `cp` → restore folder config
- [ ] `docker compose up -d`
- [ ] `docker ps` → verifikasi semua container running
- [ ] `curl` / buka browser → verifikasi aplikasi berjalan

---

## Troubleshooting

### `docker compose build` gagal
- **Gejala:** error saat build image
- **Cek:** apakah Dockerfile ada dan benar? apakah nama file di `COPY` sesuai?
- **Solusi:** `docker compose build --no-cache`, periksa error message dengan seksama

### Container langsung exit setelah di-start
- **Gejala:** `docker ps` menunjukkan container `Exited`
- **Cek:** `docker logs <nama-container>`
- **Solusi:** periksa apakah `CMD`/`ENTRYPOINT` di Dockerfile benar, periksa environment variable

### Container tidak bisa saling konek
- **Gejala:** app tidak bisa konek ke db
- **Cek:** apakah nama service di `DATABASE_URL` sesuai? (misal: `postgresql://user:pass@db:5432/myapp` — `db` harus sama dengan nama service di compose)
- **Solusi:** periksa `docker-compose.yml` bagian services dan network

### Volume kosong setelah restore
- **Gejala:** data tidak ada setelah pindah server
- **Cek:** `ls /var/lib/docker/volumes/<nama_volume>/_data/`
- **Solusi:** ulangi langkah restore, pastikan nama volume sama persis (`docker volume ls` di server lama vs server baru)

### Port tidak bisa diakses dari luar
- **Gejala:** `curl localhost` berhasil tapi dari browser luar tidak bisa
- **Cek:** `ufw status`
- **Solusi:** `ufw allow 80/tcp && ufw allow 443/tcp && ufw reload`
