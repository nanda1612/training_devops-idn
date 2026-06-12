# Grafana: Penjelasan dan Tutorial

## Daftar Isi
1. [Apa itu Grafana?](#apa-itu-grafana)
2. [Instalasi Grafana dengan Docker](#instalasi-grafana-dengan-docker)
3. [Mengakses Dashboard Grafana](#mengakses-dashboard-grafana)
4. [Login ke Grafana](#login-ke-grafana)
5. [Konsep Dasar Grafana](#konsep-dasar-grafana)
6. [Langkah Pertama Setelah Login](#langkah-pertama-setelah-login)

---

## Apa itu Grafana?

**Grafana** adalah platform open source untuk **visualisasi dan monitoring data**. Grafana mengambil data dari berbagai sumber (seperti Prometheus, MySQL, InfluxDB, dan lainnya) lalu menampilkannya dalam bentuk **grafik, chart, dan dashboard** yang interaktif secara real-time.

Bayangkan Grafana seperti **papan monitor di ruang kontrol** — semua data dari berbagai sistem ditampilkan dalam satu layar agar mudah dipantau dan dianalisis.

```
[Sumber Data]          [Grafana]            [Pengguna]
                                        
 Prometheus   ──┐                         
 MySQL        ──┼──► Grafana Server ──►  Dashboard
 InfluxDB     ──┤    (port 3000)          (grafik, alert,
 Loki         ──┘                          laporan)
```

**Kegunaan utama Grafana:**
- Memantau performa server (CPU, RAM, disk)
- Memantau status aplikasi secara real-time
- Membuat alert otomatis jika ada metrik yang melebihi batas
- Membuat laporan berbasis data dalam bentuk visual

---

## Instalasi Grafana dengan Docker

```bash
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  -v grafana-data:/var/lib/grafana \
  -v grafana-conf:/usr/share/grafana \
  grafana/grafana
```

### Penjelasan Setiap Bagian Perintah

| Bagian | Penjelasan |
|--------|-----------|
| `docker run` | Perintah untuk menjalankan container Docker baru |
| `-d` | *Detached mode* — container berjalan di latar belakang, terminal tetap bisa digunakan |
| `--name=grafana` | Memberi nama container `grafana` agar mudah dikelola (stop, start, logs) |
| `-p 3000:3000` | Memetakan port 3000 di komputer ke port 3000 di dalam container. Format: `port-mesin:port-container` |
| `-v grafana-data:/var/lib/grafana` | Menyimpan data Grafana (dashboard, user, konfigurasi) ke volume Docker bernama `grafana-data` agar tidak hilang saat container di-restart |
| `-v grafana-conf:/usr/share/grafana` | Menyimpan file konfigurasi Grafana ke volume `grafana-conf` |
| `grafana/grafana` | Nama image Docker yang dipakai — diambil dari Docker Hub (image resmi Grafana) |

> **Kenapa pakai volume (`-v`)?**  
> Tanpa volume, semua data (dashboard yang sudah dibuat, akun user) akan **hilang** setiap kali container dihapus atau diperbarui. Volume menyimpan data di luar container sehingga tetap aman.

### Verifikasi Container Berjalan

```bash
docker ps
```

Pastikan container `grafana` muncul dengan status `Up`:
```
CONTAINER ID   IMAGE             STATUS         PORTS                    NAMES
a1b2c3d4e5f6   grafana/grafana   Up 2 minutes   0.0.0.0:3000->3000/tcp   grafana
```

### Perintah Pengelolaan Container

```bash
# Melihat log Grafana
docker logs grafana

# Menghentikan Grafana
docker stop grafana

# Menjalankan kembali Grafana
docker start grafana

# Menghapus container (data tetap aman di volume)
docker rm grafana
```

---

## Mengakses Dashboard Grafana

Setelah container berjalan, buka browser dan akses:

```
http://<ip-server>:3000
```

Ganti `<ip-server>` dengan IP address server tempat Docker berjalan. Contoh:
```
http://172.23.0.11:3000
```

Jika dijalankan di komputer sendiri (localhost):
```
http://localhost:3000
```

![Grafana Dashboard](https://grafana.com/media/docs/grafana/dashboards/screenshot-empty-dashboard-v13.0.png)

> **Pastikan port 3000 tidak diblokir firewall** di server. Jika tidak bisa diakses, cek dengan:
> ```bash
> sudo ufw allow 3000/tcp
> ```

---

## Login ke Grafana

Saat pertama kali membuka Grafana, akan muncul halaman login.

**Kredensial default:**

| Field | Nilai |
|-------|-------|
| Username | `admin` |
| Password | `admin` |

Setelah login pertama, Grafana akan meminta kamu **mengganti password**. Sangat disarankan untuk menggantinya demi keamanan.

> Jangan gunakan password `admin` di lingkungan production — akun admin memiliki akses penuh ke seluruh konfigurasi Grafana.

---

## Konsep Dasar Grafana

Sebelum mulai menggunakan Grafana, penting memahami tiga konsep utamanya:

### 1. Data Source
Sumber data yang dihubungkan ke Grafana. Grafana sendiri tidak menyimpan data — ia hanya **membaca** dari data source yang dikonfigurasi.

Contoh data source yang sering dipakai:
| Data Source | Kegunaan |
|-------------|----------|
| **Prometheus** | Metrik server, aplikasi, container |
| **MySQL / PostgreSQL** | Data dari database relasional |
| **InfluxDB** | Time-series data (IoT, metrik performa) |
| **Loki** | Log aplikasi |
| **Elasticsearch** | Log dan pencarian |

### 2. Panel
Panel adalah **satu blok visualisasi** di dashboard — bisa berupa grafik garis, bar chart, gauge, tabel, dan lainnya. Setiap panel memiliki satu query ke data source.

### 3. Dashboard
Dashboard adalah **kumpulan panel** yang disusun dalam satu halaman. Satu dashboard bisa berisi puluhan panel yang memantau aspek berbeda dari sistem yang sama.

```
┌─────────────────────────────────────────────────────┐
│                    DASHBOARD                         │
│                                                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐    │
│  │  Panel 1   │  │  Panel 2   │  │  Panel 3   │    │
│  │ CPU Usage  │  │ RAM Usage  │  │  Disk I/O  │    │
│  │ (grafik)   │  │  (gauge)   │  │  (grafik)  │    │
│  └────────────┘  └────────────┘  └────────────┘    │
│                                                      │
│  ┌──────────────────────────┐  ┌────────────┐       │
│  │       Panel 4            │  │  Panel 5   │       │
│  │  Network Traffic (grafik)│  │  Uptime    │       │
│  └──────────────────────────┘  └────────────┘       │
└─────────────────────────────────────────────────────┘
```

---

## Langkah Pertama Setelah Login

### 1. Tambahkan Data Source

1. Klik ikon **⚙ (Configuration)** di menu kiri → **Data Sources**
2. Klik **"Add data source"**
3. Pilih jenis data source (contoh: Prometheus)
4. Isi URL data source (contoh: `http://prometheus:9090`)
5. Klik **"Save & Test"** — pastikan muncul pesan `Data source is working`

### 2. Buat Dashboard Baru

1. Klik ikon **⊞ (Dashboards)** di menu kiri
2. Klik **"New"** → **"New Dashboard"**
3. Klik **"Add new panel"** untuk menambahkan panel pertama

### 3. Konfigurasi Panel

1. Pilih **Data Source** yang sudah ditambahkan
2. Tulis query untuk mengambil data (contoh di Prometheus: `node_cpu_seconds_total`)
3. Pilih tipe visualisasi: *Time series*, *Gauge*, *Bar chart*, *Table*, dll.
4. Klik **Apply** untuk menyimpan panel

### 4. Simpan Dashboard

1. Klik tombol **💾 Save** di kanan atas
2. Beri nama dashboard (contoh: `Server Monitoring`)
3. Klik **Save**

---

## Ringkasan

| Topik | Detail |
|-------|--------|
| **Instalasi** | `docker run -d --name=grafana -p 3000:3000 grafana/grafana` |
| **Akses** | `http://<ip-server>:3000` |
| **Login default** | `admin` / `admin` (ganti setelah login pertama) |
| **Port** | `3000` |
| **Data tersimpan di** | Docker volume `grafana-data` dan `grafana-conf` |

*Grafana adalah tool visualisasi — ia tidak mengumpulkan data sendiri. Pasangkan dengan Prometheus atau data source lain untuk mendapatkan manfaat penuh.*
