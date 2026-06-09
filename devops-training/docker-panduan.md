# Panduan Docker: Dari Script ke Container

## Gambaran Besar Alur

```
Folder Project (Dockerfile + Source Code)
        ↓  docker compose build
Docker Image  (blueprint aplikasi)
        ↓  docker compose up -d
Docker Container  (aplikasi berjalan)
        ↓  docker save + tar volume
File Backup (.tar / .tar.gz)
        ↓  scp / rsync
Server Baru
        ↓  docker load + restore volume + compose up
Container Berjalan di Server Baru ✓
```

---

## 1. Struktur Folder Project

```
/root/my-app/
├── docker-compose.yml       ← orkestrasi semua container
├── .env                     ← variabel sensitif (tidak di-commit ke git)
├── app/
│   ├── Dockerfile           ← instruksi build image aplikasi
│   ├── requirements.txt     ← daftar library Python
│   └── main.py              ← source code utama
└── nginx/
    ├── Dockerfile           ← instruksi build image nginx
    └── nginx.conf           ← konfigurasi reverse proxy
```

---

## 2. Dockerfile

Dockerfile adalah file instruksi untuk membangun image. Setiap baris instruksi menghasilkan satu **layer** yang di-cache — jika kode berubah, hanya layer itu dan di bawahnya yang direbuild.

### Contoh: `app/Dockerfile` (Python Flask)

```dockerfile
FROM python:3.11-slim        # base image (OS + runtime)
WORKDIR /app                 # direktori kerja di container
COPY requirements.txt .      # copy dependency dulu (agar layer ini di-cache)
RUN pip install --no-cache-dir -r requirements.txt
COPY . .                     # copy source code
EXPOSE 5000                  # buka port
CMD ["python", "main.py"]    # perintah saat container start
```

### Contoh: `nginx/Dockerfile`

```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

### Cara Kerja Layer Caching

```
FROM python:3.11-slim    → Layer 0 (base OS + Python)        ← jarang berubah
WORKDIR /app             → Layer 1
COPY requirements.txt    → Layer 2
RUN pip install          → Layer 3 (library)                 ← di-cache jika requirements tidak berubah
COPY . .                 → Layer 4 (source code)             ← rebuild setiap kode berubah
```

---

## 3. docker-compose.yml

Docker Compose menjalankan dan menghubungkan banyak container sekaligus dari satu file konfigurasi.

```yaml
version: '3.8'

services:

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

### File `.env`

```env
DB_USER=myappuser
DB_PASS=r4hasi4Ku!
```

---

## 4. Build Image

```bash
cd /root/my-app

# Build semua service via docker compose
docker compose build

# Build ulang tanpa cache (jika ada perubahan besar)
docker compose build --no-cache

# Verifikasi image berhasil dibuat
docker image ls
```

---

## 5. Jalankan Container

```bash
# Jalankan semua container di background
docker compose up -d

# Cek semua container berjalan
docker ps

# Pantau log
docker compose logs -f
```

### Topologi Container yang Berjalan

```
HOST SERVER
┌─────────────────────────────────────────────────────────┐
│  Network: myapp-network (bridge)                         │
│                                                          │
│  myapp-nginx ──→ myapp-app ──→ myapp-db                 │
│  port: 80        port: 5000    port: 5432 (internal)    │
│  (proxy)         (Flask API)   (PostgreSQL)              │
│                      │              │                    │
│                 app_uploads    db_data                   │
│                  (volume)       (volume)                 │
└─────────────────────────────────────────────────────────┘
         ↑
  http://IP-SERVER (akses dari browser)
```

---

## 6. Verifikasi Container

```bash
# Status container
docker ps

# Log container tertentu
docker logs myapp-app
docker logs myapp-db

# Masuk ke dalam container (debugging)
docker exec -it myapp-app bash
docker exec -it myapp-db psql -U myappuser -d myapp

# Monitor penggunaan resource
docker stats

# Lihat volume dan network
docker volume ls
docker network ls
```

---

## 7. Backup (Sebelum Pindah Server)

Hentikan container terlebih dahulu agar data konsisten:

```bash
cd /root/my-app
docker compose down

mkdir -p /root/backup-myapp
cd /root/backup-myapp
```

### 7A. Backup Docker Images

```bash
docker save -o myapp-app.tar     myapp-app:1.0
docker save -o myapp-nginx.tar   myapp-nginx:latest
docker save -o postgres-16.tar   postgres:16

# Verifikasi
ls -lh /root/backup-myapp/*.tar
```

### 7B. Backup Docker Volumes (Data Aplikasi)

```bash
# Backup database (PALING PENTING)
docker run --rm \
  -v myapp_db_data:/source \
  -v /root/backup-myapp:/backup \
  alpine tar czf /backup/vol-db-data.tar.gz -C /source .

# Backup uploads
docker run --rm \
  -v myapp_app_uploads:/source \
  -v /root/backup-myapp:/backup \
  alpine tar czf /backup/vol-app-uploads.tar.gz -C /source .
```

> Cek nama volume aktual dengan `docker volume ls`. Format nama volume: `<nama_project>_<nama_volume>`

### 7C. Backup Konfigurasi

```bash
cp -r /root/my-app /root/backup-myapp/my-app-config
```

### 7D. Transfer ke Server Baru

```bash
# Rsync (direkomendasikan, bisa resume jika putus)
rsync -avz --progress /root/backup-myapp/ user@IP_SERVER_BARU:/root/backup-myapp/

# Atau via SCP
scp -r /root/backup-myapp/ user@IP_SERVER_BARU:/root/backup-myapp/
```

### Yang Wajib Di-backup

| Item | Perintah | Keterangan |
|------|----------|------------|
| Docker Image | `docker save -o nama.tar <image>` | Blueprint container |
| Docker Volume | `tar czf` via container alpine | Data aplikasi & database |
| docker-compose.yml | `cp` langsung | Konfigurasi orkestrasi |
| file .env | `cp` langsung | Variabel environment |
| Container | **Tidak perlu** | Dibuat ulang dari image + volume |

---

## 8. Restore di Server Baru

### 8A. Install Docker

```bash
apt update && apt install -y ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update
apt install -y docker-ce docker-ce-cli containerd.io \
               docker-buildx-plugin docker-compose-plugin

systemctl enable docker && systemctl start docker
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
```

### 8D. Restore Konfigurasi dan Jalankan

```bash
cp -r /root/backup-myapp/my-app-config /root/my-app

cd /root/my-app
docker compose up -d

# Verifikasi
docker ps
curl -I http://localhost   # HTTP/1.1 200 OK → berhasil
```

---

## 9. Checklist Migrasi

### Server Lama — Backup
- [ ] `docker compose down`
- [ ] `docker save` → image app, nginx, postgres
- [ ] `tar czf` → volume db_data
- [ ] `tar czf` → volume app_uploads
- [ ] `cp` → folder config (termasuk `.env`)
- [ ] Verifikasi semua file backup ada
- [ ] Transfer semua file ke server baru

### Server Baru — Restore
- [ ] Install Docker + Docker Compose
- [ ] `docker load` → semua image
- [ ] `docker volume create` → semua volume
- [ ] `tar xzf` → restore semua volume
- [ ] `cp` → restore folder config
- [ ] `docker compose up -d`
- [ ] `docker ps` → verifikasi semua container running
- [ ] Buka browser → verifikasi aplikasi berjalan

---

## 10. Troubleshooting

| Masalah | Cara Cek | Solusi |
|---------|----------|--------|
| `docker compose build` gagal | Baca error message | `docker compose build --no-cache`, periksa Dockerfile |
| Container langsung exit | `docker logs <nama-container>` | Periksa CMD/ENTRYPOINT dan environment variable |
| App tidak bisa konek ke DB | Cek nama service di `DATABASE_URL` | Nama `@db:5432` harus sama dengan nama service di compose |
| Volume kosong setelah restore | `ls /var/lib/docker/volumes/<nama>/_data/` | Ulangi restore, pastikan nama volume sama persis |
| Port tidak bisa diakses dari luar | `ufw status` | `ufw allow 80/tcp && ufw reload` |
