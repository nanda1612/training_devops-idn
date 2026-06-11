# Sonar Scanner: Penjelasan dan Tutorial

## Daftar Isi
1. [Apa itu Sonar Scanner?](#apa-itu-sonar-scanner)
2. [Membuat Project di SonarQube](#membuat-project-di-sonarqube)
3. [Instalasi Sonar Scanner](#instalasi-sonar-scanner)
4. [File Konfigurasi sonar-project.properties](#file-konfigurasi-sonar-projectproperties)
5. [Menjalankan Scanner](#menjalankan-scanner)
6. [Scanning dengan Exclusions](#scanning-dengan-exclusions)
7. [Scanning dengan Testing](#scanning-dengan-testing)
8. [Scanning dengan Coverage](#scanning-dengan-coverage)
9. [Ringkasan Parameter](#ringkasan-parameter)

---

## Apa itu Sonar Scanner?

**Sonar Scanner** (SonarScanner CLI) adalah tool berbasis command-line yang bertugas **membaca source code** lalu mengirim hasil analisisnya ke **SonarQube Server** untuk diolah dan ditampilkan di dashboard.

Scanner berjalan di sisi developer atau CI/CD runner — bukan di dalam server SonarQube itu sendiri.

```
[Source Code di Mesin Kamu]
          │
          ▼
  ┌───────────────────┐
  │  sonar-scanner    │  ← tool yang diinstall di mesin / CI runner
  │  (baca kode)      │
  └────────┬──────────┘
           │  kirim data via HTTP
           ▼
  ┌───────────────────┐
  │  SonarQube Server │  ← analisis, hitung metrik, tampilkan dashboard
  └───────────────────┘
```

**Scanner hanya bertugas mengumpulkan data** — keputusan apakah kode lolos Quality Gate atau tidak ada di Server.

---

## Membuat Project di SonarQube

Sebelum scanner bisa mengirim hasil analisis, project harus dibuat terlebih dahulu di SonarQube Server.

![SonarQube Project Dashboard](https://docs.sonarsource.com/files/UmHOpVCT5ShBivJ60cGK)

### Langkah Membuat Project Manual

1. Login ke SonarQube Server (`http://<ip-server>:9000`)
2. Klik **"Create Project"** → pilih **"Manually"**
3. Isi form:
   - **Project display name** — nama tampilan (contoh: `Simple Apps`)
   - **Project key** — identitas unik, tanpa spasi (contoh: `Simple-Apps`)
4. Klik **"Set Up"**
5. Pilih metode analisis → **"Locally"**
6. **Generate token** — simpan token ini, akan dipakai saat menjalankan scanner

> **Project Key** adalah nilai yang diisi di `sonar.projectKey` pada file konfigurasi.  
> Token berfungsi sebagai autentikasi scanner ke server — **jangan dibagikan**.

---

## Instalasi Sonar Scanner

### 1. Download Package

Tersedia dua versi — pilih sesuai kebutuhan:

```bash
# Versi lama (4.8) — kompatibilitas lebih luas
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip

# Versi terbaru (7.3) — direkomendasikan
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.3.0.5189-linux-x64.zip
```

### 2. Ekstrak File

```bash
unzip sonar-scanner-cli-7.3.0.5189-linux-x64.zip
```

Setelah diekstrak, akan muncul folder `sonar-scanner-7.3.0.5189-linux-x64/` di direktori saat ini.

### 3. Pindahkan ke Direktori Sistem dan Atur Kepemilikan

```bash
mv -v /tmp/sonar-scanner-7.3.0.5189-linux-x64/ /opt/sonar-scanner/
chown -R sonar:sonar /opt/sonar-scanner
```

| Perintah | Penjelasan |
|----------|-----------|
| `mv -v` | Pindahkan folder ke `/opt/sonar-scanner/` (lokasi standar aplikasi sistem) |
| `chown -R sonar:sonar` | Ubah kepemilikan folder ke user `sonar` agar hak akses lebih aman |

### 4. Buat Symbolic Link

```bash
ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/bin/
```

Perintah ini membuat shortcut sehingga `sonar-scanner` bisa dipanggil dari mana saja di terminal tanpa menulis path penuh.

**Verifikasi instalasi:**
```bash
sonar-scanner --version
```

---

## File Konfigurasi sonar-project.properties

File ini berisi konfigurasi project yang dibaca scanner setiap kali analisis dijalankan. File harus berada di **root folder project**.

```bash
vim sonar-project.properties
```

### Konfigurasi Dasar

```properties
# must be unique in a given SonarQube instance
sonar.projectKey=Simple-Apps

# --- optional properties ---

# defaults to project key
sonar.projectName=Simple Apps

# defaults to 'not provided'
sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Defaults to .
sonar.sources=.

# Encoding of the source code. Default is default system encoding
sonar.sourceEncoding=UTF-8
```

### Penjelasan Setiap Parameter

| Parameter | Wajib | Penjelasan |
|-----------|-------|-----------|
| `sonar.projectKey` | **Ya** | ID unik project di SonarQube Server. Harus sama persis dengan yang dibuat di dashboard. Tidak boleh ada spasi — gunakan `-` atau `_`. |
| `sonar.projectName` | Tidak | Nama tampilan project di dashboard. Default: mengikuti `projectKey`. |
| `sonar.projectVersion` | Tidak | Versi project saat ini. Berguna untuk melacak tren kualitas per versi. |
| `sonar.sources` | Tidak | Folder source code yang akan di-scan. `.` berarti semua file di folder saat ini. Default: `.` |
| `sonar.sourceEncoding` | Tidak | Encoding file source code. `UTF-8` adalah standar yang paling umum. |

---

## Menjalankan Scanner

Setelah konfigurasi siap, jalankan scanner dari root folder project:

```bash
sonar-scanner \
  -Dsonar.projectKey=Simple-Apps \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://10.23.0.11:9000 \
  -Dsonar.login=sqp_1405bc476b26d2b88fd512d79d8f3383f08e8dff
```

### Penjelasan Parameter CLI

| Parameter | Penjelasan |
|-----------|-----------|
| `-Dsonar.projectKey` | Project Key yang sudah dibuat di SonarQube Server |
| `-Dsonar.sources` | Folder source code yang di-scan (`.` = folder aktif) |
| `-Dsonar.host.url` | Alamat SonarQube Server yang dituju |
| `-Dsonar.login` | Token autentikasi yang di-generate dari dashboard SonarQube |

> **Catatan:** Parameter yang sudah ada di `sonar-project.properties` tidak perlu ditulis ulang di CLI. Parameter CLI akan **menimpa** nilai di file jika ada konflik.

### Output Normal Saat Scanner Berjalan

```
INFO: Scanner configuration file: /opt/sonar-scanner/conf/sonar-scanner.properties
INFO: Project root configuration file: /path/to/project/sonar-project.properties
INFO: SonarScanner 7.3.0
INFO: ------------- Scan Simple-Apps
INFO: Load server rules
INFO: Indexing files...
INFO: Load coverage report from '.../coverage/lcov.info'
INFO: Analysis report generated
INFO: ANALYSIS SUCCESSFUL, you can find the results at:
INFO: http://10.23.0.11:9000/dashboard?id=Simple-Apps
INFO: Note that you will be able to access the URL only once the server has processed the submitted analysis report
INFO: More about the report processing at http://10.23.0.11:9000/api/ce/task?id=...
INFO: Analysis total time: 12.345 s
INFO: ------------------------------------------------------------------------
INFO: EXECUTION SUCCESS
INFO: ------------------------------------------------------------------------
```

---

## Scanning dengan Exclusions

Digunakan untuk **mengecualikan folder atau file** tertentu dari proses analisis — misalnya `node_modules`, `coverage`, atau folder test.

### Konfigurasi di sonar-project.properties

```bash
vim sonar-project.properties
```

Tambahkan atau edit baris berikut:

```properties
sonar.exclusions=coverage/**, node_modules/**, testing/**
```

### Penjelasan Pola Exclusion

| Pola | Artinya |
|------|---------|
| `coverage/**` | Semua file dan subfolder di dalam folder `coverage/` |
| `node_modules/**` | Semua file di `node_modules/` — wajib dikecualikan di project Node.js |
| `testing/**` | Semua file di folder `testing/` |
| `**/*.test.js` | Semua file berekstensi `.test.js` di semua folder |
| `src/generated/**` | Folder kode yang di-generate otomatis — tidak perlu dianalisis |

### Jalankan Scanner

```bash
sonar-scanner \
  -Dsonar.projectKey=Simple-Apps \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://10.23.0.11:9000 \
  -Dsonar.login=sqp_1405bc476b26d2b88fd512d79d8f3383f08e8dff
```

> Exclusion sudah terbaca dari `sonar-project.properties` — tidak perlu ditambahkan di CLI.

---

## Scanning dengan Testing

Digunakan agar SonarQube **mengetahui lokasi folder unit test**, sehingga kode test tidak dihitung sebagai kode produksi dan metrik lebih akurat.

### Konfigurasi di sonar-project.properties

```bash
vim sonar-project.properties
```

Tambahkan atau edit baris berikut:

```properties
sonar.tests=testing/
```

### Penjelasan

| Parameter | Penjelasan |
|-----------|-----------|
| `sonar.tests` | Path ke folder unit test. SonarQube akan memisahkan analisis kode test dari kode produksi. |

**Mengapa ini penting?**
- Tanpa `sonar.tests`, SonarQube menganggap semua file sebagai kode produksi
- Dengan `sonar.tests`, metrik seperti *coverage* dan *code smells* hanya dihitung dari kode produksi — hasilnya lebih representatif

### Jalankan Scanner

```bash
sonar-scanner \
  -Dsonar.projectKey=Simple-Apps \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://10.23.0.11:9000 \
  -Dsonar.login=sqp_1405bc476b26d2b88fd512d79d8f3383f08e8dff
```

---

## Scanning dengan Coverage

Digunakan untuk melaporkan **persentase kode yang tercakup oleh unit test** ke SonarQube. Coverage tidak dihitung langsung oleh scanner — scanner hanya membaca **laporan coverage** yang sudah dihasilkan oleh testing framework.

### Alur Kerja Coverage

```
[Unit Test dijalankan]
        │
        ▼
[Testing framework hasilkan laporan]
 → Jest     → coverage/lcov.info
 → Istanbul → coverage/lcov.info
 → PHPUnit  → clover.xml
        │
        ▼
[sonar-scanner membaca laporan tersebut]
        │
        ▼
[Hasil coverage muncul di SonarQube dashboard]
```

### Generate Laporan Coverage (Contoh dengan Jest/Node.js)

Sebelum menjalankan scanner, jalankan unit test dengan opsi coverage:

```bash
# Dengan npm
npm test -- --coverage

# Atau dengan Jest langsung
jest --coverage
```

Laporan akan tersimpan di `coverage/lcov.info`.

### Konfigurasi di sonar-project.properties

```bash
vim sonar-project.properties
```

Tambahkan atau edit baris berikut:

```properties
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

### Parameter Coverage per Bahasa

| Bahasa | Parameter | File Laporan |
|--------|-----------|-------------|
| JavaScript / TypeScript | `sonar.javascript.lcov.reportPaths` | `coverage/lcov.info` |
| Python | `sonar.python.coverage.reportPaths` | `coverage.xml` |
| Java | `sonar.coverage.jacoco.xmlReportPaths` | `target/site/jacoco/jacoco.xml` |
| PHP | `sonar.php.coverage.reportPaths` | `clover.xml` |
| Go | `sonar.go.coverage.reportPaths` | `coverage.out` |

### Jalankan Scanner

```bash
sonar-scanner \
  -Dsonar.projectKey=Simple-Apps \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://10.23.0.11:9000 \
  -Dsonar.login=sqp_1405bc476b26d2b88fd512d79d8f3383f08e8dff
```

---

## Ringkasan Parameter

Berikut adalah `sonar-project.properties` lengkap yang menggabungkan semua konfigurasi di atas:

```properties
# ── Identitas Project ─────────────────────────────────────────
sonar.projectKey=Simple-Apps
sonar.projectName=Simple Apps
sonar.projectVersion=1.0

# ── Source Code ───────────────────────────────────────────────
sonar.sources=.
sonar.sourceEncoding=UTF-8

# ── Exclusions ────────────────────────────────────────────────
sonar.exclusions=coverage/**, node_modules/**, testing/**

# ── Testing ───────────────────────────────────────────────────
sonar.tests=testing/

# ── Coverage ──────────────────────────────────────────────────
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

### Tabel Lengkap Semua Parameter

| Parameter | Default | Keterangan |
|-----------|---------|-----------|
| `sonar.projectKey` | — | **Wajib.** ID unik project di server |
| `sonar.projectName` | = projectKey | Nama tampilan di dashboard |
| `sonar.projectVersion` | `not provided` | Versi project |
| `sonar.sources` | `.` | Folder source code yang di-scan |
| `sonar.sourceEncoding` | System default | Encoding file (gunakan `UTF-8`) |
| `sonar.exclusions` | — | Pola file/folder yang dikecualikan |
| `sonar.tests` | — | Folder unit test |
| `sonar.javascript.lcov.reportPaths` | — | Path laporan coverage untuk JS/TS |
| `sonar.host.url` | `http://localhost:9000` | Alamat SonarQube Server |
| `sonar.login` | — | Token autentikasi |

---

*Lihat konsep dan arsitektur SonarQube secara menyeluruh di [explain.md](explain.md). Lihat panduan instalasi SonarQube Server dan database di [docs-sonarqube.md](docs-sonarqube.md).*
