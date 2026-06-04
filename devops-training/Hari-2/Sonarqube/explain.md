# SonarQube: Platform Kualitas Kode
### *"Temukan bug sebelum bug menemukan kamu di production"*

---

## Apa itu SonarQube?

**SonarQube** adalah platform *open source* untuk melakukan **analisis kualitas kode secara otomatis** (Static Code Analysis). SonarQube membaca source code kamu — tanpa menjalankannya — lalu melaporkan masalah yang ditemukan: bug, celah keamanan, kode yang tidak efisien, hingga duplikasi.

SonarQube dikembangkan oleh **SonarSource** (Swiss, 2008) dan saat ini menjadi standar industri untuk *code quality gate* di pipeline CI/CD.

---

## Mengapa Kita Butuh SonarQube?

Tanpa tool seperti SonarQube, masalah kode baru terdeteksi saat:
- Review manual (lambat, tidak konsisten)
- Testing (hanya menemukan bug fungsional)
- **Production sudah down** (terlambat)

Dengan SonarQube, masalah terdeteksi **otomatis di setiap push kode**, jauh sebelum sampai ke server production.

```
Developer push kode
        ↓
   Sonar Scanner membaca kode
        ↓
   SonarQube Server menganalisis
        ↓
   Laporan muncul di dashboard
        ↓
   Quality Gate: PASS ✅ atau FAIL ❌
        ↓
   (jika FAIL) Pipeline dihentikan — kode tidak boleh masuk
```

---

## Komponen SonarQube

SonarQube terdiri dari **tiga bagian utama** yang bekerja bersama:

```
┌─────────────────────────────────────────────────────────────┐
│                        ARSITEKTUR                           │
│                                                             │
│   [Source Code]                                             │
│        │                                                    │
│        ▼                                                    │
│  ┌─────────────┐     laporan     ┌──────────────────────┐  │
│  │ Sonar       │ ─────────────► │  SonarQube Server    │  │
│  │ Scanner     │                │  (Analisis + UI)     │  │
│  └─────────────┘                └──────────┬───────────┘  │
│   (di mesin developer              simpan  │               │
│    atau CI/CD runner)                      ▼               │
│                                   ┌────────────────┐       │
│                                   │   PostgreSQL   │       │
│                                   │   Database     │       │
│                                   └────────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

| Komponen | Fungsi | Referensi |
|----------|--------|-----------|
| **SonarQube Server** | Menerima hasil scan, menganalisis, menampilkan dashboard | [docs-sonarqube.md](docs-sonarqube.md) |
| **Sonar Scanner** | Membaca source code dan mengirim data ke server | [docs-sonar-scanner.md](docs-sonar-scanner.md) |
| **PostgreSQL** | Menyimpan seluruh riwayat analisis, project, dan konfigurasi | [docs-sonarqube.md](docs-sonarqube.md) |

---

## Apa yang Dianalisis SonarQube?

SonarQube mengklasifikasikan temuan ke dalam lima kategori:

### 1. Bugs
Kode yang **berpotensi crash** atau menghasilkan output yang salah.

```
Contoh: Null pointer dereference, infinite loop, salah operator
Dampak: Error di runtime, data corrupt
```

### 2. Vulnerabilities (Celah Keamanan)
Kode yang **bisa dieksploitasi** oleh penyerang.

```
Contoh: SQL injection, hardcoded password, XSS
Dampak: Data breach, akses tidak sah ke sistem
```

### 3. Code Smells
Kode yang **tidak salah secara fungsional**, tapi sulit dibaca, dipahami, atau di-maintain.

```
Contoh: Fungsi terlalu panjang, variabel tidak dipakai, duplikasi logika
Dampak: Technical debt — makin lama makin sulit dikembangkan
```

### 4. Security Hotspots
Kode yang **perlu ditinjau manual** — bukan pasti masalah, tapi berisiko jika tidak diperhatikan.

```
Contoh: Penggunaan kriptografi, penanganan input eksternal
Dampak: Butuh keputusan developer untuk konfirmasi aman atau tidak
```

### 5. Coverage
Persentase kode yang **dicakup oleh unit test**.

```
Contoh: 70% coverage = 30% kode tidak diuji sama sekali
Dampak: Bug di kode yang tidak ditest tidak akan terdeteksi otomatis
```

---

## Metrik Kualitas

SonarQube melaporkan beberapa angka kunci pada setiap analisis:

| Metrik | Arti |
|--------|------|
| **Bugs** | Jumlah bug yang ditemukan |
| **Vulnerabilities** | Jumlah celah keamanan |
| **Code Smells** | Jumlah masalah maintainability |
| **Coverage** | Persentase kode yang ditest |
| **Duplications** | Persentase kode yang duplikat |
| **Technical Debt** | Estimasi waktu untuk memperbaiki semua Code Smells |
| **Reliability Rating** | A–E, dinilai dari jumlah dan severity bug |
| **Security Rating** | A–E, dinilai dari severity vulnerability |

**Skala Rating:**

```
A  → Tidak ada issue (atau minor saja)   ← target ideal
B  → Minor issue
C  → Major issue
D  → Critical issue
E  → Blocker issue                       ← berbahaya, harus segera diperbaiki
```

---

## Quality Gate

**Quality Gate** adalah kebijakan yang menentukan apakah kode **layak diteruskan** ke proses berikutnya (merge, deploy, release).

Quality Gate bekerja seperti gerbang tol: kode hanya bisa lewat jika **semua kondisi terpenuhi**.

```
Quality Gate DEFAULT "Sonar Way":
─────────────────────────────────────────────────────
✓ Coverage on New Code         ≥ 80%
✓ Duplicated Lines on New Code ≤ 3%
✓ Maintainability Rating       = A
✓ Reliability Rating           = A
✓ Security Rating              = A
✓ Security Hotspots Reviewed   = 100%
─────────────────────────────────────────────────────
Semua kondisi PASS → Quality Gate: ✅ PASSED
Satu saja FAIL    → Quality Gate: ❌ FAILED
```

> Quality Gate bisa dikustomisasi sesuai standar tim atau perusahaan.
> Misalnya: coverage minimal 70%, tidak boleh ada Blocker bug sama sekali.

---

## Alur Kerja dengan CI/CD

SonarQube paling efektif diintegrasikan ke dalam pipeline CI/CD:

```
┌─────────────────────────────────────────────────────────┐
│                    PIPELINE CI/CD                        │
│                                                          │
│  git push                                                │
│     │                                                    │
│     ▼                                                    │
│  ┌──────────┐    ┌──────────┐    ┌─────────────────┐   │
│  │  Build   │ -> │  Test    │ -> │  Sonar Scanner  │   │
│  └──────────┘    └──────────┘    └────────┬────────┘   │
│                                           │ kirim hasil  │
│                                           ▼             │
│                                  ┌────────────────┐     │
│                                  │ SonarQube      │     │
│                                  │ Server         │     │
│                                  └───────┬────────┘     │
│                                          │               │
│                          ┌───────────────┤               │
│                          │               │               │
│                    PASSED ✅           FAILED ❌         │
│                          │               │               │
│                          ▼               ▼               │
│                       Deploy          Pipeline           │
│                                        Stop              │
└─────────────────────────────────────────────────────────┘
```

---

## Cara Kerja Sonar Scanner

Scanner adalah komponen yang **membaca source code** di sisi developer atau CI runner.

```bash
# Scanner membaca konfigurasi project dari sonar-project.properties
# lalu mengirim seluruh data analisis ke SonarQube Server

sonar-scanner \
  -Dsonar.projectKey=nama-project \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://sonarqube-server:9000 \
  -Dsonar.login=<token>
```

File `sonar-project.properties` berisi konfigurasi project seperti:
- `sonar.projectKey` — identitas unik project di server
- `sonar.sources` — folder source code yang akan di-scan
- `sonar.exclusions` — folder yang dikecualikan (misal: `node_modules/`)
- `sonar.tests` — folder unit test
- `sonar.javascript.lcov.reportPaths` — path laporan coverage

> Lihat cara lengkap instalasi dan konfigurasi scanner di [docs-sonar-scanner.md](docs-sonar-scanner.md)

---

## Instalasi SonarQube Server

SonarQube Server bisa diinstall dengan dua cara:

### Cara 1: Manual (Bare Metal / VM)
Cocok untuk environment production yang butuh kontrol penuh.

**Prasyarat yang dibutuhkan:**
- Java 17
- PostgreSQL 15
- RAM minimal 8 GB

> Lihat panduan instalasi lengkap di [docs-sonarqube.md](docs-sonarqube.md)

### Cara 2: Docker
Lebih cepat untuk development atau testing environment.

```bash
docker run -d --name sonarqube \
    -p 9000:9000 \
    -e SONAR_JDBC_URL=jdbc:postgresql://<db-host>/sonarqube \
    -e SONAR_JDBC_USERNAME=sonar \
    -e SONAR_JDBC_PASSWORD=sonar \
    sonarqube:community
```

> Akses dashboard: `http://server:9000`  
> Login default: `admin` / `admin` (ganti segera setelah login pertama)

---

## Ringkasan

| Komponen | Dokumen |
|----------|---------|
| Instalasi SonarQube Server + Database | [docs-sonarqube.md](docs-sonarqube.md) |
| Instalasi Sonar Scanner + cara scanning | [docs-sonar-scanner.md](docs-sonar-scanner.md) |
| Konsep, arsitektur, dan cara kerja | explain.md *(file ini)* |

---

*SonarQube Community Edition gratis untuk digunakan. Edisi Developer dan Enterprise menambahkan fitur seperti analisis branch, pull request decoration, dan dukungan bahasa tambahan.*
