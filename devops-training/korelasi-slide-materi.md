# Korelasi Slide Training DevOps (PDF) dengan File Materi
**Training DevOps - ID-Networkers** | Instruktur: Bazigan Tsamara

---

## 1. DevOps Concept (Slide 3–12)

### Penjelasan dari Slide

- **DevOps** (Development & Operations) adalah kombinasi filosofi, praktik, dan tools untuk mempercepat pengembangan software.
- **Siklus DevOps:** Plan → Code → Build → Test → Release → Deploy → Operate → Monitor *(simbol infinity/loop)*
- **Pendekatan Tradisional:** Developer menulis kode, push ke GitHub, tim infrastruktur deploy dan monitor secara manual (terpisah).
- **Pendekatan DevOps:** Alur yang sama namun diotomasi melalui CI/CD sehingga build, test, deploy berjalan otomatis tanpa intervensi manual.
- **DevOps Labs Architecture (Slide 12):**

```
Git (Local) → GitHub (SCM) → Jenkins (CI/CD) → SonarQube (Code Review)
→ Docker (Server) → Docker Hub (Registry) → Prometheus (Monitoring) → Grafana (Dashboard)
```

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `README.md` | Rundown training yang mencerminkan alur DevOps Labs (Git, GitHub, SonarQube, Docker, Jenkins, Prometheus, Grafana) |
| `docs-apps.md` | Simple Apps (ExpressJS + MariaDB) adalah aplikasi yang menjadi objek penerapan seluruh pipeline DevOps |

---

## 2. Version Control System / Git (Slide 13–20)

### Penjelasan dari Slide

- **Version Control System (VCS)** adalah sistem yang menyimpan perubahan file ke dalam sebuah versi (snapshot).
- **Tanpa VCS:** file disimpan manual dengan nama berbeda (Skripsi-Revisi-1, Skripsi-Revisi-2, dst) — tidak terstruktur dan rawan hilang.
- **Dengan VCS:** setiap perubahan tercatat dengan pesan (commit) yang menjelaskan apa yang berubah. Hanya ada SATU file, tapi semua riwayat tersimpan di dalam repository.
- **Git** adalah salah satu open source distributed version control system. Artinya setiap developer memiliki salinan penuh repository di komputernya.

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-git.md` | Referensi lengkap perintah Git (config, init, add, commit, branch, merge, tag, revert) |
| `tutorial-git.md` | Tutorial praktik Git step-by-step dengan latihan commit pertama, branching, dan simulasi konflik |
| `penjelasan-git.md` | Penjelasan konsep Git dalam Bahasa Indonesia (konfigurasi, status file, staging, branching, tagging) |

### Perintah Kunci

```bash
git config --global user.name "nama"
git config --global user.email "email"
git init
git status
git add . / git add <file>
git commit -m "pesan"
git branch / git switch / git merge
git tag -a v1.0.0 -m "pesan"
git revert -n <commit-id>
```

---

## 3. Source Code Management / GitHub (Slide 21–26)

### Penjelasan dari Slide

- **GitHub** adalah Source Code Management (SCM) yang berfungsi menyimpan repository untuk digunakan berkolaborasi tim.
- **Perbedaan Git vs GitHub:**

| | Git | GitHub |
|--|-----|--------|
| Instalasi | Di-install di host | Tidak perlu install |
| Koneksi | Offline | Online |
| Lisensi | Open source | Gratis (ada tier berbayar) |
| User Management | Tidak ada | Ada |

- **Remote Repository:** Developer A dan B sama-sama terhubung ke satu Remote Repository di GitHub.

| Operasi | Keterangan |
|---------|------------|
| `CLONE` | Menyalin repo dari GitHub ke lokal |
| `FETCH` | Mengunduh perubahan tanpa merge |
| `PULL` | Mengunduh + merge (fetch + merge) |
| `PUSH` | Mengirim perubahan lokal ke GitHub |

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-github.md` | Panduan singkat perintah git remote, push, fetch, pull dan langkah kolaborasi via GitHub Organization |
| `tutorial-github.md` | Tutorial lengkap GitHub: membuat akun, git remote, push/pull/fetch, kolaborasi Organization, alur Pull Request beserta tabel permission (Read/Triage/Write/Maintain/Admin) |

### Perintah Kunci

```bash
git remote add origin <url>
git remote -v
git push origin main
git pull origin main
git fetch origin main
git clone <url>
```

---

## 4. Simple Apps - IDN (Slide 27–29)

### Penjelasan dari Slide

- **Simple Apps** adalah aplikasi contoh/sampel yang akan diterapkan culture DevOps dalam training ini.
- **Teknologi:** NodeJS + NPM + ExpressJS (backend) + MariaDB (database)
- **Arsitektur:**
  ```
  User → ExpressJS (route: /, /app1, /app2, /users)
       → MariaDB (db: training, user: peserta, pass: password, table: tb_data)
  ```

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-apps.md` | Panduan setup lengkap Simple Apps: clone source code, install NodeJS v18.x, install & setup MariaDB, import database (training.sql), jalankan app: `npm install && npm start`, akses di `http://ip_server:3000` |

---

## 5. SonarQube - Source Code Analysis (Slide 32–36)

### Penjelasan dari Slide

- **Clean Code:** penulisan kode yang mudah dibaca, dimaintain, dipahami, diubah strukturnya, konsisten, dan tetap secure.
- **SonarQube:** Automatic code review tool untuk menerapkan clean and secure code.
- **Alur SonarQube dalam DevOps:**
  ```
  Code → Commit → Push → SonarLint (IDE) → Build & Test
  → SonarQube/SonarCloud → Quality Gate
  → PASS: Merge/Promote | FAIL: kembali ke developer
  ```
- **Arsitektur SonarQube:**
  ```
  Scanner (CI/CD) → kirim Analysis Report via HTTP
  → SonarQube Server (Web Server + Compute Engine + Search Server)
  → Database Server
  ```

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-sonarqube.md` | Instalasi SonarQube Server (manual & via Docker): requirement Java 17, PostgreSQL 15, spek server Ubuntu 4 Core / 8 GB / 30 GB. Konfigurasi sonar.properties, setup systemd service, akses di port 9000 |
| `docs-sonar-scanner.md` | Instalasi sonar-scanner CLI, konfigurasi sonar-project.properties, menjalankan scan dengan/tanpa exclusion, dengan test coverage (lcov.info) |
| `docs-testing.md` | Testing aplikasi (unit test & integration test) menggunakan `npm test` & `npm run test:coverage` yang hasilnya dipakai oleh sonar-scanner |

---

## 6. Docker Containerization (Slide 37–54)

### Penjelasan dari Slide

- **Containerization:** proses packaging software code dan system library untuk menjalankan sebuah aplikasi secara portable, scalable, efficient, consistent.
- **Analogi:** *"Makanan di bungkus = Take away, Aplikasi di bungkus = Containerization"*
- **Evolusi deployment:**
  ```
  Traditional (bare metal) → Virtualized (VM + Hypervisor) → Container Deployment
  ```
- **Docker Architecture:**
  ```
  Client (docker run/build/pull) → Docker Daemon
  → Images/Containers di Docker Host → Registry (Docker Hub)
  ```
- **Dockerfile:** template untuk membuat docker image
  ```
  Dockerfile → (build) → Docker Image → (run) → Docker Container
  ```
- **Docker Compose:** alat untuk menjalankan multi-container dari satu file YAML
  ```
  Dockerfile → Compose File → Docker Compose (start semua service)
  ```

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-docker.md` | Panduan lengkap Docker: install Docker, perintah dasar, praktik pull/run/exec/commit/save/load container, membuat Dockerfile, push ke Docker Hub, Docker Volume, Docker Network, Logging & Troubleshooting, Portainer (dashboard), Docker Compose |

---

## 7. Jenkins CI/CD Tools (Slide 55–59)

### Penjelasan dari Slide

- **CI/CD Pipeline** adalah otomasi alur dari commit kode sampai ke production:
  ```
  Commit → Trigger Build → Build → Run Tests → Deliver Build → Deploy
  ```
- **CI Pipeline:** Build + Unit Tests + Integration Tests
- **CD Pipeline:** Review + Staging + Production
- **Jenkins** adalah free open source CI/CD tools yang menjalankan build, testing, deployment, dan step lainnya secara otomatis.

| Platform | Tipe | Integrasi |
|----------|------|-----------|
| Jenkins | Open source, gratis | ⭐⭐⭐⭐⭐ |
| CircleCI | Berbayar | ⭐⭐⭐⭐ |
| TeamCity | Berbayar | ⭐⭐⭐⭐ |
| Bamboo | Berbayar | ⭐⭐⭐ |
| GitLab CI | Berbayar (ada free tier) | ⭐⭐⭐⭐ |

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-jenkins.md` | Instalasi Jenkins via apt repository (Java 17 required), konfigurasi Jenkins Agent, dan contoh Jenkinsfile/Pipeline Script |

**Stage dalam Jenkinsfile:**

| Stage | Perintah |
|-------|----------|
| Pull SCM | `git clone` dari GitHub |
| Build | `npm install` |
| Testing | `npm test` + `npm run test:coverage` |
| Code Review | `sonar-scanner` (integrasi SonarQube) |
| Deploy | `docker compose up --build -d` |
| Backup | `docker compose push` (ke Docker Hub) |

---

## 8. Prometheus Monitoring System (Slide 60–67)

### Penjelasan dari Slide

- **Monitoring System:** software untuk memantau kinerja infrastruktur, system device, traffic, dan aplikasi, serta memberi alarm jika ada masalah.
- **Prometheus:** aplikasi perangkat lunak gratis untuk event monitoring dan alerting.
- **Cara kerja Prometheus:** setiap scrape interval, Prometheus mengambil data metrics dari endpoint `/metrics` di setiap service instance.
  - Contoh metrics: `memory_bytes_used`, `http_requests_total`, `jobs_in_queue`
- **Arsitektur Prometheus:**
  ```
  Targets (Jobs/Exporters) → pull metrics → Prometheus Server (TSDB)
  → PromQL query → Web UI / Grafana / API clients
  → Alertmanager → Email / PagerDuty / dll
  ```

| Exporter | Metrics | Port |
|----------|---------|------|
| Node Exporter | CPU, disk, memory Linux server | 9100 |
| cAdvisor | Metrics container Docker | 8080 |
| Nginx Exporter | Metrics Nginx | 9113 |

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-prometheus.md` | Instalasi Prometheus via Docker (port 9090), konfigurasi prometheus.yml untuk scrape targets, setup Node Exporter, cAdvisor, Nginx Exporter, setup Alert Rules (server down, CPU/memory > 50%) |
| `Prometheus d4e0cdf4ff4240198960cf21d5d36e1e.md` | Instalasi Prometheus manual (tanpa Docker): setup user/group prometheus, download binary, konfigurasi systemd service, setup node_exporter sebagai systemd service |

---

## 9. Grafana Dashboarding (Slide 68–72)

### Penjelasan dari Slide

- **Grafana:** platform dashboarding atau data visualizer yang dapat memvisualisasikan data dari berbagai sumber (multi-platform).
- **Integrasi dengan Prometheus:**
  ```
  Prometheus mengumpulkan metrics dari exporters (node.js, nginx, mysql)
  → kirim ke Grafana via PromQL query
  → Grafana menampilkan dalam bentuk dashboard visual (grafik, chart)
  ```
- **Grafana Cloud:** hosted Grafana yang mendukung long term storage, efficient querying, dan scalable infrastructure.

### Korelasi dengan File Materi

| File | Keterangan |
|------|------------|
| `docs-grafana.md` | Instalasi Grafana via Docker (port 3000), akses dashboard di `http://ip:3000`, login default `admin/admin`. Grafana terhubung ke Prometheus sebagai data source untuk visualisasi metrics |

---

## Ringkasan: Alur Pipeline DevOps Training IDN

```
[Developer]
    |
    | git add . && git commit -m "..." && git push origin main
    v
[GitHub] ──────────────────────────── docs-github.md / tutorial-github.md
    |
    | webhook / trigger
    v
[Jenkins] ─────────────────────────── docs-jenkins.md
    |
    |── Stage: Pull SCM   (git clone dari GitHub)
    |── Stage: Build      (npm install)              ── docs-apps.md
    |── Stage: Testing    (npm test + coverage)       ── docs-testing.md
    |── Stage: Code Review (sonar-scanner)            ── docs-sonarqube.md
    |                                                    docs-sonar-scanner.md
    |── Stage: Deploy     (docker compose up)         ── docs-docker.md
    |── Stage: Backup     (docker compose push)       ── docs-docker.md (registry)
    v
[Docker Container berjalan]
    |
    | metrics di-scrape setiap interval
    v
[Prometheus] ──────────────────────── docs-prometheus.md
    |  Node Exporter (server metrics)
    |  cAdvisor (container metrics)
    |
    | PromQL query sebagai data source
    v
[Grafana] ─────────────────────────── docs-grafana.md
    |  Dashboard visual: CPU, memory, container health, request rate
    v
[Alert] ───────────────────────────── docs-prometheus.md (bagian setup rule)
    (Alertmanager: server down, CPU > 50%)
```

---

*Dibuat dari: Slide Training DevOps.pdf (80 halaman) + 13 file materi*  
*Trainer: Bazigan Tsamara (System Administrator, Certified Kubernetes Admin)*  
*Penyelenggara: ID-Networkers — Indonesian IT Expert Factory*
