# Prometheus: Penjelasan dan Tutorial

## Daftar Isi
1. [Apa itu Prometheus?](#apa-itu-prometheus)
2. [Instalasi Prometheus dengan Docker](#instalasi-prometheus-dengan-docker)
3. [Mengakses Prometheus UI](#mengakses-prometheus-ui)
4. [Query Dasar PromQL](#query-dasar-promql)
5. [Setup Exporter](#setup-exporter)
   - [A. Node Exporter](#a-node-exporter)
   - [B. cAdvisor (Docker Monitoring)](#b-cadvisor-docker-monitoring)
   - [C. Nginx Exporter](#c-nginx-exporter)
6. [Setup Rules](#setup-rules)
   - [A. Record Rule](#a-record-rule)
   - [B. Alert Rule: Server Down](#b-alert-rule-server-down)
   - [C. Alert Rule: CPU & Memory > 50%](#c-alert-rule-cpu--memory--50)

---

## Apa itu Prometheus?

**Prometheus** adalah sistem monitoring dan alerting open source yang dirancang untuk **mengumpulkan, menyimpan, dan mengolah metrik** dari server, aplikasi, dan container secara real-time.

Cara kerjanya sederhana: Prometheus secara rutin **mendatangi** (scrape) setiap target untuk mengambil data metrik, lalu menyimpannya sebagai *time-series data* (data berdasarkan waktu). Ini seperti petugas sensus yang setiap 15 detik mengunjungi setiap mesin untuk mencatat kondisinya.

![Arsitektur Prometheus](https://prometheus.io/assets/docs/tutorial/architecture.png)

### Bagaimana Prometheus Bekerja?

```
[Target / Exporter]          [Prometheus Server]        [Visualisasi]
                                                     
 Node Exporter (9100) ──┐                            
 cAdvisor      (8080) ──┼──► scrape tiap 15s ──► simpan ──► Grafana
 Nginx Exporter(9113) ──┘    data metrik         data       Dashboard
 App itu sendiri       ──┘
```

Prometheus menggunakan model **pull** — server Prometheus yang aktif mengambil data dari target, bukan sebaliknya. Ini berbeda dari sistem monitoring lain yang menunggu target mengirim data.

### Kapan Menggunakan Prometheus?
- Monitoring performa server (CPU, RAM, disk, network)
- Monitoring container Docker / Kubernetes
- Monitoring aplikasi web (request per second, error rate)
- Membuat alert otomatis saat sistem bermasalah

---

## Instalasi Prometheus dengan Docker

### 1. Buat Direktori dan File Konfigurasi

```bash
mkdir /prometheus
touch /prometheus/prometheus.yml
```

Direktori `/prometheus` akan digunakan sebagai tempat menyimpan file konfigurasi Prometheus (`prometheus.yml`) yang akan di-mount ke dalam container.

### 2. Jalankan Container Prometheus

```bash
docker run -d \
  --name prometheus-container \
  -e TZ=UTC \
  -p 9090:9090 \
  --restart=always \
  -v /prometheus/:/etc/prometheus/ \
  -v prometheus-data:/prometheus \
  prom/prometheus
```

### Penjelasan Setiap Flag

| Flag | Penjelasan |
|------|-----------|
| `-d` | Container berjalan di latar belakang (*detached*) |
| `--name prometheus-container` | Nama container agar mudah dikelola |
| `-e TZ=UTC` | Set timezone container ke UTC agar timestamp metrik konsisten |
| `-p 9090:9090` | Buka port 9090 — Prometheus UI bisa diakses dari luar container |
| `--restart=always` | Container otomatis restart jika crash atau server reboot |
| `-v /prometheus/:/etc/prometheus/` | Mount folder konfigurasi lokal ke dalam container — perubahan pada `prometheus.yml` langsung terbaca |
| `-v prometheus-data:/prometheus` | Simpan data time-series ke Docker volume agar tidak hilang saat container diperbarui |
| `prom/prometheus` | Image resmi Prometheus dari Docker Hub |

### Verifikasi Container Berjalan

```bash
docker ps
```

```
CONTAINER ID   IMAGE             STATUS         PORTS                    NAMES
a1b2c3d4e5f6   prom/prometheus   Up 2 minutes   0.0.0.0:9090->9090/tcp   prometheus-container
```

---

## Mengakses Prometheus UI

Buka browser dan akses:

```
http://<ip-server>:9090
```

Contoh:
```
http://172.23.0.11:9090
```

Di Prometheus UI kamu bisa:
- **Graph** — tulis query PromQL dan lihat hasilnya dalam bentuk grafik atau tabel
- **Targets** — cek status semua target yang sedang di-scrape (UP/DOWN)
- **Alerts** — lihat alert yang sedang aktif
- **Status → Configuration** — lihat isi konfigurasi `prometheus.yml` yang sedang berjalan

### Query Pertama untuk Verifikasi

Setelah masuk ke UI, coba query berikut di tab **Graph**:

```
prometheus_target_interval_length_seconds
```

Jika muncul data, berarti Prometheus berjalan normal dan sudah berhasil men-scrape dirinya sendiri.

![Contoh Grafik Prometheus](https://prometheus.io/assets/docs/tutorial/sample_graph.png)

---

## Query Dasar PromQL

**PromQL** (Prometheus Query Language) adalah bahasa yang digunakan untuk mengambil dan mengolah data metrik di Prometheus.

| Query | Artinya |
|-------|---------|
| `up` | Status semua target: `1` = UP, `0` = DOWN |
| `node_cpu_seconds_total` | Total waktu CPU yang digunakan (mentah) |
| `rate(node_cpu_seconds_total[1m])` | Rata-rata penggunaan CPU per detik dalam 1 menit terakhir |
| `node_memory_MemAvailable_bytes` | RAM yang masih tersedia (dalam bytes) |
| `nginx_connections_active` | Jumlah koneksi aktif ke Nginx saat ini |

---

## Setup Exporter

**Exporter** adalah program kecil yang dipasang di target (server atau aplikasi) untuk **mengubah data metrik menjadi format yang bisa dibaca Prometheus**, lalu menyajikannya di endpoint `/metrics`.

```
[Server / Aplikasi]
        │
        ▼
   [Exporter]  ──► /metrics  ──► Prometheus scrape
```

---

### A. Node Exporter

**Node Exporter** digunakan untuk memonitor **metrik sistem operasi Linux** — CPU, RAM, disk, network, dan lainnya.

#### 1. Jalankan Node Exporter

```bash
docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  --name ct-node-exporter \
  --restart=always \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
```

#### Penjelasan Flag Node Exporter

| Flag | Penjelasan |
|------|-----------|
| `--net="host"` | Gunakan jaringan host langsung — agar exporter bisa membaca metrik network interface dengan benar |
| `--pid="host"` | Akses ke proses host — agar exporter bisa membaca informasi proses yang berjalan |
| `-v "/:/host:ro,rslave"` | Mount seluruh filesystem host sebagai read-only — exporter membaca `/proc`, `/sys`, dan `/dev` untuk mengambil metrik |
| `--path.rootfs=/host` | Beri tahu exporter bahwa root filesystem ada di `/host` (sesuai mount di atas) |

#### 2. Verifikasi Metrik Node Exporter

Buka browser:
```
http://<ip-server>:9100/metrics
```

Akan muncul ratusan baris metrik dalam format teks. Ini adalah data mentah yang akan di-scrape oleh Prometheus.

![Node Exporter Metrics](https://prometheus.io/assets/docs/tutorial/node_exporter.png)

#### 3. Daftarkan ke Prometheus

Edit file konfigurasi Prometheus:

```bash
nano /prometheus/prometheus.yml
```

Isi dengan konfigurasi berikut:

```yaml
global:
  scrape_interval: 15s       # Prometheus mengambil data dari semua target setiap 15 detik
  evaluation_interval: 15s   # Rules dievaluasi setiap 15 detik

rule_files:
  # - "first_rules.yml"      # Aktifkan jika sudah ada file rules

scrape_configs:
  - job_name: "prometheus"           # Prometheus memonitor dirinya sendiri
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "Server monitoring"    # Job untuk memonitor server via Node Exporter
    scrape_interval: 5s              # Khusus job ini, data diambil setiap 5 detik
    static_configs:
      - targets:
          - "172.23.0.11:9100"       # Tambahkan IP server yang dipasang Node Exporter
          - "172.23.0.12:9100"
          - "172.23.0.13:9100"
        labels:
          os: 'linux'                # Label untuk memudahkan filter di query/dashboard
```

#### Penjelasan Konfigurasi prometheus.yml

| Parameter | Penjelasan |
|-----------|-----------|
| `scrape_interval` | Seberapa sering Prometheus mengambil data dari target. Default: 15s |
| `evaluation_interval` | Seberapa sering rules (alert/record) dievaluasi |
| `job_name` | Nama kelompok target. Muncul sebagai label `job` di setiap metrik |
| `targets` | Daftar alamat target dalam format `ip:port` |
| `labels` | Label tambahan yang ditempelkan ke semua metrik dari target ini |

#### 4. Restart Prometheus

```bash
docker restart prometheus-container
```

#### 5. Query Metrik CPU Server

```
rate(node_cpu_seconds_total{mode="system"}[1m])
```

Query ini menghitung **rata-rata penggunaan CPU dalam mode system** selama 1 menit terakhir.

---

### B. cAdvisor (Docker Monitoring)

**cAdvisor** (Container Advisor) digunakan untuk memonitor **metrik container Docker** — CPU, RAM, dan network yang digunakan oleh setiap container.

#### 1. Jalankan cAdvisor

```bash
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:v0.49.1
```

#### Penjelasan Flag cAdvisor

| Flag | Penjelasan |
|------|-----------|
| `--volume=/:/rootfs:ro` | Akses ke seluruh filesystem host (read-only) untuk membaca info container |
| `--volume=/var/run:/var/run:ro` | Akses ke Docker socket untuk membaca status container |
| `--volume=/sys:/sys:ro` | Akses ke `/sys` untuk membaca metrik kernel dan hardware |
| `--volume=/var/lib/docker/` | Akses ke data Docker untuk membaca info storage container |
| `--privileged` | Izin akses tinggi ke host — diperlukan agar cAdvisor bisa membaca semua metrik |
| `--device=/dev/kmsg` | Akses ke kernel message buffer |

#### 2. Verifikasi cAdvisor

```
http://172.23.0.11:8080/
```

#### 3. Daftarkan ke Prometheus

Tambahkan ke `prometheus.yml`:

```yaml
- job_name: "Docker monitoring"
  scrape_interval: 5s
  static_configs:
    - targets: ["172.23.0.11:8080"]
      labels:
        app: 'docker'
```

```bash
docker restart prometheus-container
```

#### 4. Query Metrik Container

```
rate(container_cpu_usage_seconds_total{name="redis"}[1m])
```

Contoh query ini melihat **penggunaan CPU container bernama `redis`** dalam 1 menit terakhir. Ganti `redis` dengan nama container yang ingin dipantau.

---

### C. Nginx Exporter

**Nginx Exporter** digunakan untuk memonitor **metrik web server Nginx** — jumlah koneksi aktif, request per second, dan lainnya.

#### 1. Aktifkan `stub_status` di Nginx

Nginx perlu dikonfigurasi agar mengekspos data statusnya di endpoint `/metrics`:

```bash
nano /etc/nginx/sites-available/default.conf
```

Tambahkan blok `location /metrics` di dalam blok `server`:

```nginx
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # Tambahkan blok ini
    location /metrics {
        stub_status;
    }
}
```

> `stub_status` adalah modul bawaan Nginx yang menampilkan statistik koneksi secara real-time. Endpoint ini yang akan dibaca oleh Nginx Exporter.

Reload Nginx setelah edit konfigurasi:
```bash
nginx -s reload
```

#### 2. Jalankan Nginx Exporter

```bash
docker container run -d \
  --rm \
  -p 9113:9113 \
  --name nginx-exporter \
  nginx/nginx-prometheus-exporter \
  -nginx.scrape-uri http://172.23.0.11/metrics
```

> Ganti `172.23.0.11` dengan IP server tempat Nginx berjalan. Exporter ini membaca data dari `/metrics` Nginx lalu mengubahnya ke format Prometheus di port `9113`.

#### 3. Daftarkan ke Prometheus

Tambahkan ke `prometheus.yml`:

```yaml
- job_name: "nginx monitoring"
  scrape_interval: 5s
  static_configs:
    - targets: ["172.23.0.11:9113"]
      labels:
        group: 'nginx'
```

```bash
docker restart prometheus-container
```

#### 4. Query Metrik Nginx

```
nginx_connections_active
```

---

## Setup Rules

**Rules** adalah aturan yang dievaluasi Prometheus secara berkala. Ada dua jenis:

| Jenis | Fungsi |
|-------|--------|
| **Record Rule** | Menghitung query yang kompleks dan menyimpan hasilnya sebagai metrik baru — agar tidak dihitung ulang setiap kali ada request |
| **Alert Rule** | Memicu alert jika kondisi tertentu terpenuhi (misal: server mati, CPU tinggi) |

---

### A. Record Rule

Record Rule berguna untuk **menyimpan hasil kalkulasi berat** agar query di Grafana lebih cepat.

#### 1. Buat File Rules

```bash
nano /prometheus/prometheus.rules.yml
```

```yaml
groups:
  - name: cpu-node
    rules:
    - record: node_cpu          # Nama metrik baru yang disimpan
      expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
      # Kalkulasi rata-rata CPU per job, instance, dan mode dalam 5 menit
```

**Penjelasan:**
- `record: node_cpu` — nama metrik baru yang bisa langsung di-query seperti metrik biasa
- `expr` — rumus PromQL yang dihitung. Hasilnya disimpan sebagai `node_cpu`

#### 2. Daftarkan ke prometheus.yml

```bash
nano /prometheus/prometheus.yml
```

```yaml
rule_files:
  - prometheus.rules.yml
```

```bash
docker restart prometheus-container
```

---

### B. Alert Rule: Server Down

Alert ini **mendeteksi jika ada target yang tidak bisa dijangkau** oleh Prometheus (status `UP = 0`).

```bash
nano /prometheus/prometheus.rules.yml
```

Tambahkan grup baru di bawah grup yang sudah ada:

```yaml
  - name: server_down
    rules:
    - alert: server_down          # Nama alert
      expr: up == 0               # Kondisi: target tidak bisa di-scrape
      for: 1m                     # Alert baru dipicu setelah kondisi berlangsung 1 menit
      labels:
        severity: critical        # Tingkat keparahan
      annotations:
        summary: Server are down. # Pesan singkat yang muncul di alert
```

#### Penjelasan Parameter Alert

| Parameter | Penjelasan |
|-----------|-----------|
| `alert` | Nama alert yang akan muncul di UI dan notifikasi |
| `expr` | Ekspresi PromQL — alert aktif jika hasil ekspresi ini `true` |
| `for` | Kondisi harus bertahan selama durasi ini sebelum alert benar-benar dipicu. Mencegah alert palsu dari gangguan sesaat |
| `labels.severity` | Label keparahan: `info`, `warning`, `critical` |
| `annotations.summary` | Deskripsi singkat yang muncul di notifikasi alert |

```bash
docker restart prometheus-container
```

---

### C. Alert Rule: CPU & Memory > 50%

Alert ini **memicu peringatan jika CPU server berada di atas 50% selama lebih dari 1 menit**.

```bash
nano /prometheus/cpu.rules.yml
```

```yaml
groups:
  - name: example_alerts
    rules:
    - alert: HighCPUUtilization
      expr: avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[1m]) * 100) < 50
      # Jika rata-rata CPU idle < 50%, artinya CPU aktif > 50%
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: High CPU utilization on host {{ $labels.instance }}
        description: The CPU utilization on host {{ $labels.instance }} has exceeded 50 for 1 minutes.
```

> **Logika query:** `node_cpu_seconds_total{mode="idle"}` mengukur waktu CPU yang **tidak digunakan**. Jika nilai ini `< 50`, artinya lebih dari 50% CPU sedang dipakai. `{{ $labels.instance }}` adalah template yang otomatis diisi dengan nama server saat alert dipicu.

#### Daftarkan File Rules ke prometheus.yml

```bash
nano /prometheus/prometheus.yml
```

```yaml
rule_files:
  - prometheus.rules.yml
  - cpu.rules.yml
```

```bash
docker restart prometheus-container
```

#### Cek Alert di Prometheus UI

Buka `http://<ip-server>:9090/alerts` — alert yang aktif akan muncul di sini dengan status:
- **Inactive** — kondisi belum terpenuhi
- **Pending** — kondisi terpenuhi tapi belum melewati durasi `for`
- **Firing** — alert aktif dan sudah dikirim ke Alertmanager

---

## Ringkasan Arsitektur Monitoring

```
┌─────────────────────────────────────────────────────────────────┐
│                      STACK MONITORING                            │
│                                                                   │
│  [Server Linux]   [Docker Host]   [Nginx]                        │
│       │                │             │                            │
│  Node Exporter    cAdvisor      Nginx Exporter                   │
│   (port 9100)    (port 8080)    (port 9113)                      │
│       │                │             │                            │
│       └────────────────┴─────────────┘                           │
│                         │ scrape tiap 5–15s                       │
│                         ▼                                         │
│              ┌─────────────────────┐                             │
│              │  Prometheus Server  │ ◄── prometheus.yml          │
│              │    (port 9090)      │ ◄── rules files             │
│              └──────────┬──────────┘                             │
│                         │                                         │
│                         ▼                                         │
│              ┌─────────────────────┐                             │
│              │       Grafana       │  ← visualisasi dashboard    │
│              │    (port 3000)      │                             │
│              └─────────────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

| Komponen | Port | Fungsi |
|----------|------|--------|
| Prometheus Server | 9090 | Scrape, simpan, query metrik |
| Node Exporter | 9100 | Metrik OS Linux (CPU, RAM, disk) |
| cAdvisor | 8080 | Metrik container Docker |
| Nginx Exporter | 9113 | Metrik web server Nginx |
| Grafana | 3000 | Visualisasi dashboard dari data Prometheus |

*Untuk menampilkan data Prometheus di Grafana, tambahkan Prometheus sebagai Data Source di Grafana dengan URL `http://<ip-prometheus>:9090`. Lihat [explain-grafana.md](../Grafana/explain-grafana.md) untuk panduan lengkapnya.*
