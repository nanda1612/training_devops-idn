# Penjelasan Materi Jenkins

## Apa itu Jenkins?

Jenkins adalah tools open-source untuk **Continuous Integration / Continuous Delivery (CI/CD)**. Jenkins membantu tim developer untuk mengotomatiskan proses build, test, dan deploy aplikasi secara konsisten dan berulang tanpa intervensi manual.

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

## 2. Agent

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

## 3. Pipeline / Jenkinsfile

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

## 4. Penjelasan Tiap Stage

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
