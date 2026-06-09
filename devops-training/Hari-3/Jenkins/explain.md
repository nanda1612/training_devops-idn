# Penjelasan Materi Jenkins

## Apa itu CI/CD?

**CI/CD** adalah singkatan dari **Continuous Integration** dan **Continuous Delivery/Deployment** — sebuah praktik DevOps untuk mengotomatiskan seluruh proses pengiriman software mulai dari kode ditulis hingga berjalan di production.

### Continuous Integration (CI)

CI adalah praktik di mana setiap developer secara rutin menggabungkan (merge) perubahan kodenya ke repository bersama, lalu proses **build dan testing dijalankan secara otomatis**. Tujuannya:
- Mendeteksi bug atau konflik kode sedini mungkin.
- Memastikan kode selalu dalam kondisi siap di-build.
- Menghindari "integration hell" — kondisi di mana kode banyak developer sulit digabungkan karena terlalu lama terpisah.

```
Developer push code
        ↓
Build otomatis
        ↓
Unit Test + Integration Test otomatis
        ↓
Laporan hasil (pass/fail)
```

### Continuous Delivery (CD)

CD adalah kelanjutan dari CI — setelah kode lulus semua pengujian, proses **pengiriman ke staging atau production disiapkan secara otomatis**. Perbedaan antara Delivery dan Deployment:

| | Continuous Delivery | Continuous Deployment |
|--|--------------------|-----------------------|
| Arti | Siap deploy kapan saja, tapi butuh approval manual | Deploy ke production sepenuhnya otomatis |
| Cocok untuk | Tim yang butuh kontrol sebelum rilis | Tim dengan test coverage sangat tinggi |

### Kenapa CI/CD Penting?

Tanpa CI/CD, proses rilis software dilakukan manual — build manual, test manual, upload manual — yang lambat dan rawan human error. Dengan CI/CD:

- Rilis bisa dilakukan lebih sering dengan risiko lebih kecil.
- Bug terdeteksi lebih cepat sebelum sampai ke production.
- Developer bisa fokus menulis kode, bukan mengurus proses deployment.

---

## Apa itu Jenkins?

Jenkins adalah tools open-source untuk **Continuous Integration / Continuous Delivery (CI/CD)**. Jenkins membantu tim developer untuk mengotomatiskan proses build, test, dan deploy aplikasi secara konsisten dan berulang tanpa intervensi manual.

---

## Hubungan Jenkins dengan CI/CD

Jenkins adalah salah satu **implementasi** dari CI/CD. Jika CI/CD adalah konsepnya, maka Jenkins adalah tools yang mewujudkan konsep tersebut.

```
Konsep CI/CD  →  Diwujudkan oleh  →  Jenkins
```

Berikut bagaimana Jenkins menjalankan tiap bagian CI/CD:

| Konsep CI/CD | Peran Jenkins |
|--------------|---------------|
| Trigger otomatis saat ada push kode | Jenkins memantau GitHub via webhook/polling |
| Build otomatis | Jenkins menjalankan stage `Build` (npm install, dll) |
| Test otomatis | Jenkins menjalankan stage `Testing` (npm test) |
| Code quality check | Jenkins menjalankan stage `Code Review` (sonar-scanner) |
| Deploy otomatis ke staging/production | Jenkins menjalankan stage `Deploy` (docker compose up) |
| Backup artifact | Jenkins menjalankan stage `Backup` (docker compose push) |

Jenkins menggunakan **Jenkinsfile** (Pipeline Script) untuk mendefinisikan seluruh alur CI/CD dalam bentuk kode. Setiap kali ada perubahan di repository, Jenkins otomatis menjalankan pipeline dari awal hingga akhir.

```
GitHub (push) → Jenkins (trigger) → Pipeline berjalan otomatis
    └─ Build → Test → Code Review → Deploy → Backup
```

---

## 1. Install Jenkins

Proses instalasi Jenkins di Debian/Ubuntu dilakukan dalam 3 tahap:

### a. Tambahkan GPG Key
```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```
GPG key digunakan untuk memverifikasi bahwa package Jenkins yang diunduh benar-benar berasal dari sumber resmi, bukan dari pihak ketiga yang berbahaya.

### b. Tambahkan Repository Jenkins
```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
```
Perintah ini mendaftarkan repository resmi Jenkins ke sistem APT, sehingga Jenkins bisa diinstal dan diupdate melalui `apt`.

### c. Install Jenkins
```bash
sudo apt-get update
sudo apt-get install fontconfig openjdk-17-jre
sudo apt-get install jenkins
```
- `fontconfig` — library font yang dibutuhkan oleh tampilan Jenkins.
- `openjdk-17-jre` — Jenkins berjalan di atas Java, sehingga Java Runtime Environment (JRE) wajib dipasang terlebih dahulu.
- Setelah instalasi, Jenkins berjalan sebagai service di port **8080** secara default.

---

## 2. Cara Menjalankan Jenkins

Setelah Jenkins terinstal, Jenkins berjalan sebagai **service (daemon)** di sistem operasi. Berikut cara mengelola dan mengaksesnya.

### a. Kelola Service Jenkins

```bash
# Jalankan Jenkins
sudo systemctl start jenkins

# Hentikan Jenkins
sudo systemctl stop jenkins

# Restart Jenkins
sudo systemctl restart jenkins

# Cek status Jenkins (running/stopped)
sudo systemctl status jenkins

# Aktifkan Jenkins agar otomatis berjalan saat server reboot
sudo systemctl enable jenkins
```

Jika status menunjukkan `active (running)`, berarti Jenkins sudah berjalan dengan benar.

### b. Akses Jenkins via Browser

Jenkins memiliki antarmuka web (Web UI). Setelah service berjalan, akses melalui browser:

```
http://<ip-server>:8080
```

Contoh: `http://192.168.1.10:8080`

### c. Setup Awal Jenkins (Pertama Kali)

Saat pertama kali diakses, Jenkins meminta **Administrator Password** untuk unlock. Password tersebut tersimpan otomatis di server:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Salin password tersebut, paste ke kolom yang tersedia di browser, lalu lanjutkan setup:

1. **Unlock Jenkins** — masukkan initial admin password.
2. **Install Plugins** — pilih *"Install suggested plugins"* untuk menginstal plugin yang umum dipakai.
3. **Buat Admin User** — isi username, password, dan email untuk akun admin Jenkins.
4. **Jenkins siap digunakan.**

### d. Alur Penggunaan Jenkins

Setelah Jenkins berjalan, berikut alur umum penggunaannya:

```
1. Login ke Web UI (http://ip:8080)
         ↓
2. Tambahkan Agent (node) di menu "Manage Jenkins > Nodes"
         ↓
3. Buat Job/Pipeline baru (New Item > Pipeline)
         ↓
4. Hubungkan ke repository GitHub (SCM)
         ↓
5. Tulis atau arahkan ke Jenkinsfile
         ↓
6. Jalankan Build (manual / otomatis via webhook)
         ↓
7. Pantau hasil di Dashboard Jenkins
```

---

## 4. Agent

Agent adalah mesin (node) yang menjalankan job/pipeline Jenkins. Jenkins Controller (master) mendistribusikan pekerjaan ke agent.

```bash
apt install openjdk-17-jdk openjdk-17-jre -y
sudo update-alternatives --config java
```

- **openjdk-17-jdk** — JDK (Java Development Kit) dipasang di agent karena agent perlu menjalankan proses Java.
- `update-alternatives --config java` — memilih versi Java aktif jika terdapat beberapa versi yang terinstal.

### Kenapa menggunakan Agent?
- Memisahkan beban kerja dari server Jenkins utama.
- Memungkinkan pipeline berjalan paralel di beberapa mesin sekaligus.
- Agent bisa dikonfigurasi khusus untuk kebutuhan tertentu (misalnya agent khusus untuk build Docker).

---

## 5. Pipeline / Jenkinsfile

Pipeline adalah serangkaian tahapan (stages) otomatis yang mendefinisikan alur CI/CD dari suatu aplikasi. Pipeline ditulis dalam file bernama `Jenkinsfile` menggunakan sintaks **Groovy DSL**.

### Struktur Dasar Pipeline

```groovy
pipeline {
    agent { label 'devops1-nama' }

    stages {
        stage('Nama Stage') {
            steps {
                // perintah yang dijalankan
            }
        }
    }
}
```

- `agent` — menentukan di mesin mana pipeline akan dijalankan. `label` digunakan untuk memilih agent berdasarkan nama/label yang sudah dikonfigurasi.
- `stages` — kumpulan dari stage-stage yang berjalan secara berurutan.
- `stage` — satu tahapan dalam pipeline (misal: Build, Test, Deploy).
- `steps` — perintah-perintah yang dieksekusi dalam sebuah stage.

---

## 6. Penjelasan Tiap Stage

### Stage 1: Pull SCM
```groovy
stage('Pull SCM') {
    steps {
        git branch: 'main', url: 'https://github.com/username/simple-apps.git'
    }
}
```
Menarik (pull) source code terbaru dari repository Git (GitHub). Jenkins akan mengunduh kode dari branch `main` setiap kali pipeline dijalankan, sehingga selalu menggunakan versi kode terkini.

---

### Stage 2: Build
```groovy
stage('Build') {
    steps {
        sh'''
        cd app
        npm install
        '''
    }
}
```
Menjalankan proses build aplikasi Node.js. `npm install` mengunduh semua dependency yang tercantum di `package.json`. Tahap ini memastikan aplikasi bisa dikompilasi/disiapkan dengan benar sebelum diuji.

---

### Stage 3: Testing
```groovy
stage('Testing') {
    steps {
        sh'''
        cd app
        npm test
        npm run test:coverage
        '''
    }
}
```
Menjalankan unit test secara otomatis:
- `npm test` — menjalankan seluruh test case yang ada.
- `npm run test:coverage` — menghasilkan laporan code coverage (berapa persen kode yang tercakup oleh test).

Jika ada test yang gagal, pipeline akan berhenti dan tidak lanjut ke tahap berikutnya.

---

### Stage 4: Code Review
```groovy
stage('Code Review') {
    steps {
        sh'''
        cd app
        sonar-scanner \
            -Dsonar.projectKey=Simple-Apps \
            -Dsonar.sources=. \
            -Dsonar.host.url=http://172.23.x.x:9000 \
            -Dsonar.login=token-sonar
        '''
    }
}
```
Menjalankan analisis kode statis menggunakan **SonarQube**. Scanner akan membaca kode sumber dan mengirimkan hasilnya ke server SonarQube untuk dianalisis:
- Bug, vulnerability, dan code smell akan terdeteksi.
- Laporan code coverage dari tahap Testing juga bisa diintegrasikan ke sini.
- `sonar.projectKey` — identitas unik project di SonarQube.
- `sonar.host.url` — alamat server SonarQube.
- `sonar.login` — token autentikasi untuk mengakses SonarQube.

---

### Stage 5: Deploy
```groovy
stage('Deploy') {
    steps {
        sh'''
        docker compose up --build -d
        '''
    }
}
```
Melakukan deployment aplikasi menggunakan Docker Compose:
- `--build` — membangun ulang image Docker dari Dockerfile sebelum menjalankan container.
- `-d` (detached mode) — container berjalan di background, sehingga pipeline tidak tertahan menunggu container.

---

### Stage 6: Backup
```groovy
stage('Backup') {
    steps {
        sh 'docker compose push'
    }
}
```
Mendorong (push) Docker image yang sudah di-build ke Docker Registry (misalnya Docker Hub atau private registry). Ini berfungsi sebagai backup image dan memudahkan deployment ke environment lain (staging, production) menggunakan image yang sama.

---

## Alur CI/CD Secara Keseluruhan

```
Developer push code ke GitHub
        ↓
Jenkins mendeteksi perubahan (webhook/polling)
        ↓
Pull SCM → ambil kode terbaru
        ↓
Build → install dependency
        ↓
Testing → jalankan unit test & coverage
        ↓
Code Review → analisis kualitas kode via SonarQube
        ↓
Deploy → jalankan aplikasi via Docker Compose
        ↓
Backup → push image ke Docker Registry
```

Pipeline ini memastikan setiap perubahan kode melewati serangkaian validasi otomatis sebelum benar-benar berjalan di lingkungan production.
