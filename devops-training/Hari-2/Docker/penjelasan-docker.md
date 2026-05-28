# Penjelasan Materi Docker

## 1. Instalasi Docker

Sebelum menggunakan Docker, kita perlu menginstalnya dari repository resmi Docker. Proses instalasi terdiri dari tiga langkah utama: menambahkan GPG key Docker, mendaftarkan repository-nya ke sistem apt, lalu menginstal paket Docker Engine beserta komponen pendukungnya (`containerd`, `buildx`, `compose`).

Menggunakan repository resmi Docker penting agar kita selalu mendapatkan versi terbaru yang stabil, bukan versi lama dari repository bawaan Ubuntu.

---

## 2. Perintah Dasar Docker

### Mengelola Image
Image adalah "cetakan" yang digunakan untuk membuat container. Setiap container selalu dibuat dari sebuah image.

- `docker image ls` atau `docker images` — melihat daftar image yang tersedia di lokal
- `docker pull <image>` — mengunduh image dari Docker Registry (Docker Hub) ke lokal

### Mengelola Container
Container adalah instance yang berjalan dari sebuah image. Satu image bisa dijalankan menjadi banyak container sekaligus.

- `docker ps` — melihat container yang sedang berjalan
- `docker ps -a` — melihat semua container termasuk yang sudah berhenti
- `docker info` — melihat informasi lengkap tentang Docker Engine

---

## 3. Praktik Docker

### Menjalankan Container
`docker run -it ubuntu` menjalankan container Ubuntu secara interaktif. Flag `-i` berarti *interactive* (stdin tetap aktif), dan `-t` berarti *terminal* (menampilkan prompt). Setelah masuk ke dalam container, kita bisa menjalankan perintah Linux seperti biasa.

### Commit Container menjadi Image
`docker commit` digunakan untuk menyimpan kondisi container saat ini (termasuk perubahan yang sudah dilakukan di dalamnya) menjadi sebuah image baru. Ini berguna ketika kita sudah menginstall software di dalam container dan ingin menyimpan hasilnya sebagai image yang bisa dipakai ulang.

### Export dan Import Image
- `docker save -o <file>` — mengekspor image menjadi file arsip `.tar`
- `docker load -i <file>` — mengimpor image dari file arsip

Keduanya berguna untuk memindahkan image antar server tanpa harus push ke registry.

---

## 4. Dockerfile

Dockerfile adalah file teks berisi instruksi untuk membangun (*build*) sebuah Docker image secara otomatis dan konsisten. Dengan Dockerfile, siapa pun bisa mereproduksi image yang sama persis tanpa harus melakukan langkah manual satu per satu.

**Instruksi umum dalam Dockerfile:**
- `FROM` — menentukan image dasar yang digunakan sebagai titik awal
- `RUN` — menjalankan perintah shell saat proses build (misalnya install package)
- `ADD` / `COPY` — menyalin file dari host ke dalam image
- `CMD` — perintah default yang dijalankan saat container pertama kali dinyalakan

**Alur build:**
```
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
```

Setelah Dockerfile dibuat, jalankan `docker build -t <nama-image> .` untuk membangun image. Tanda `.` menunjukkan bahwa Dockerfile ada di direktori saat ini.

---

## 5. Docker Registry

Docker Registry adalah layanan penyimpanan dan distribusi Docker image. Registry publik yang paling populer adalah **Docker Hub**.

**Alur push image ke Docker Hub:**
1. Login: `docker login -u <username>`
2. Beri tag: `docker tag <image> <username>/<image>`
3. Push: `docker push <username>/<image>`

Dengan image tersimpan di registry, siapa pun di tim bisa mengunduhnya (`docker pull`) dan menjalankan aplikasi yang sama di environment mana pun.

---

## 6. Docker Volume

Container bersifat *ephemeral* — data di dalamnya hilang ketika container dihapus. Docker Volume adalah mekanisme untuk menyimpan data secara persisten di luar container.

`docker volume create <nama>` membuat volume, lalu volume tersebut di-*attach* ke container saat dijalankan dengan flag `-v`. Dengan begitu, data tetap ada meskipun containernya dihapus atau diganti dengan yang baru.

---

## 7. Docker Network

Secara default, container yang berbeda tidak bisa saling berkomunikasi langsung. Docker Network memungkinkan beberapa container terhubung dalam satu jaringan virtual yang terisolasi.

`docker network create --subnet <range> <nama-network>` membuat jaringan custom dengan rentang IP tertentu. Container kemudian dihubungkan ke jaringan tersebut saat dijalankan dengan flag `--network`. Untuk menghubungkan container yang sudah berjalan, gunakan `docker network connect`.

---

## 8. Docker Logging dan Troubleshooting

Ketika container bermasalah, ada beberapa perintah untuk mendiagnosis:

- `docker inspect <container>` — menampilkan detail konfigurasi dan status container dalam format JSON (IP, volume, network, environment variable, dsb.)
- `docker logs <container>` — menampilkan output log dari proses yang berjalan di dalam container
- `docker top <container>` — melihat proses yang sedang berjalan di dalam container
- `docker stats` — memantau penggunaan CPU, memori, dan network secara real-time
- `docker image history <image>` — melihat riwayat layer saat image dibangun
- `docker system df` — melihat penggunaan disk oleh Docker (image, container, volume)

---

## 9. Docker Dashboard (Portainer)

Portainer adalah antarmuka web (*web UI*) untuk mengelola Docker secara visual tanpa harus hafal perintah CLI. Portainer dijalankan sebagai container itu sendiri dan diakses melalui browser di port `9443` (HTTPS).

Portainer berguna terutama bagi yang baru belajar Docker atau untuk memantau banyak container sekaligus dari satu layar.

---

## 10. Docker Compose

Docker Compose adalah alat untuk mendefinisikan dan menjalankan aplikasi yang terdiri dari **banyak container** sekaligus menggunakan satu file konfigurasi `docker-compose.yml`.

**Alur kerja Docker Compose:**
1. Buat `Dockerfile` untuk setiap service yang dibutuhkan
2. Definisikan semua service dan hubungannya di `docker-compose.yml`
3. Jalankan semuanya sekaligus dengan `docker compose up -d`

**Perintah utama:**
- `docker compose build` — membangun image dari Dockerfile
- `docker compose up -d` — menjalankan semua container di background
- `docker compose down` — menghentikan dan menghapus semua container

Dibandingkan menjalankan container satu per satu dengan `docker run`, Docker Compose jauh lebih praktis karena seluruh konfigurasi (port, volume, network, environment) tersimpan dalam satu file yang bisa di-*version control* bersama kode aplikasi.
