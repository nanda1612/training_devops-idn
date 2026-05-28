# Cara Kerja Docker Container

## 1. Apakah Docker Container Selalu Butuh Image?

**Ya — setiap Docker container wajib dibuat dari sebuah image.**

Image adalah "cetakan" (blueprint) yang berisi sistem file, library, konfigurasi, dan kode aplikasi. Tanpa image, Docker tidak tahu apa yang harus dijalankan di dalam container. Hubungan ini bersifat mutlak: tidak ada container tanpa image.

Analoginya:
- **Image** = resep masakan
- **Container** = masakan yang sudah jadi dari resep tersebut
- Satu resep (image) bisa menghasilkan banyak hidangan (container) sekaligus

---

## 2. Dari Mana Image Berasal?

Ada tiga cara mendapatkan image:

| Cara | Perintah | Keterangan |
|---|---|---|
| Unduh dari registry | `docker pull nginx` | Ambil image siap pakai dari Docker Hub |
| Bangun dari Dockerfile | `docker build -t myapp .` | Buat image custom dari instruksi |
| Simpan dari container aktif | `docker commit` | Jadikan container yang sudah dimodifikasi sebagai image baru |

---

## 3. Proses Pembuatan Container (Step by Step)

### Alur Lengkap

```
┌─────────────────────────────────────────────────────────────────┐
│                        SUMBER IMAGE                             │
│                                                                 │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────────────┐  │
│   │  Docker Hub │   │  Dockerfile │   │  Container (commit) │  │
│   │  (Registry) │   │  (Build)    │   │  (Snapshot)         │  │
│   └──────┬──────┘   └──────┬──────┘   └──────────┬──────────┘  │
│          │                 │                      │             │
│          └─────────────────┴──────────────────────┘            │
│                            │                                    │
└────────────────────────────┼────────────────────────────────────┘
                             ▼
                    ┌────────────────┐
                    │  Docker Image  │  ← disimpan di host lokal
                    │  (Read-Only)   │
                    └───────┬────────┘
                            │
                    docker run <image>
                            │
                            ▼
              ┌─────────────────────────┐
              │     Docker Container    │
              │  ┌───────────────────┐  │
              │  │  Writable Layer   │  │  ← perubahan data ditulis di sini
              │  └───────────────────┘  │
              │  ┌───────────────────┐  │
              │  │  Image Layers     │  │  ← read-only, dari image
              │  │  (Read-Only)      │  │
              │  └───────────────────┘  │
              └─────────────────────────┘
```

### Penjelasan Tiap Tahap

**Tahap 1 — Image tersedia di lokal**
Docker memeriksa apakah image sudah ada di host. Jika belum, Docker secara otomatis mengunduhnya dari registry (`docker pull` implisit).

**Tahap 2 — docker run dipanggil**
```bash
docker run -d -p 8080:80 --name webserver nginx
```
Docker membaca image `nginx`, lalu membuat container baru dari image tersebut.

**Tahap 3 — Writable Layer dibuat**
Docker menambahkan satu layer tulis (writable layer) di atas semua layer image yang read-only. Semua perubahan dalam container (file baru, log, instalasi paket) ditulis di layer ini saja — image aslinya tidak pernah berubah.

**Tahap 4 — Container berjalan**
Proses utama yang didefinisikan di image (misalnya nginx worker) dijalankan. Container kini aktif dan siap melayani request.

---

## 4. Topologi: Satu Image, Banyak Container

```
                    ┌────────────────────────┐
                    │      Docker Image      │
                    │         nginx          │
                    │   (Read-Only Layers)   │
                    └────────────┬───────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
    ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
    │   Container A    │  │   Container B    │  │   Container C    │
    │   webserver-1    │  │   webserver-2    │  │   webserver-3    │
    │  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────┐  │
    │  │  RW Layer  │  │  │  │  RW Layer  │  │  │  │  RW Layer  │  │
    │  └────────────┘  │  │  └────────────┘  │  │  └────────────┘  │
    │  port: 8081      │  │  port: 8082      │  │  port: 8083      │
    └──────────────────┘  └──────────────────┘  └──────────────────┘
```

Tiga container berjalan dari image yang **sama**. Masing-masing memiliki writable layer sendiri sehingga perubahan di satu container tidak mempengaruhi container lainnya.

---

## 5. Topologi: Struktur Layer Image

Docker image dibangun berlapis-lapis. Setiap instruksi `RUN`, `COPY`, atau `ADD` di Dockerfile menghasilkan satu layer baru.

```
Dockerfile:
  FROM ubuntu:22.04          ─── Layer 1 (base OS)
  RUN apt-get install nginx  ─── Layer 2 (nginx binary)
  COPY ./html /var/www/html  ─── Layer 3 (app files)
  CMD ["nginx", "-g", ...]   ─── (metadata, bukan layer)

                    ┌────────────────────────┐
                    │  Layer 3: app files    │  ← paling atas
                    ├────────────────────────┤
                    │  Layer 2: nginx        │
                    ├────────────────────────┤
                    │  Layer 1: ubuntu:22.04 │  ← base
                    └────────────────────────┘
                              IMAGE
                                │
                    docker run  │
                                ▼
                    ┌────────────────────────┐
                    │  Writable Layer        │  ← milik container ini saja
                    ├────────────────────────┤
                    │  Layer 3 (read-only)   │
                    ├────────────────────────┤
                    │  Layer 2 (read-only)   │
                    ├────────────────────────┤
                    │  Layer 1 (read-only)   │
                    └────────────────────────┘
                            CONTAINER
```

Layer yang sama dapat **dipakai bersama** oleh banyak image dan container. Inilah yang membuat Docker hemat disk — ubuntu base layer tidak perlu disimpan dua kali meski dipakai oleh banyak image.

---

## 6. Siklus Hidup Container

```
              docker create / docker run
                          │
                          ▼
                   ┌─────────────┐
                   │   CREATED   │  ← container ada tapi belum jalan
                   └──────┬──────┘
                          │ docker start
                          ▼
                   ┌─────────────┐
             ┌────►│   RUNNING   │◄────┐
             │     └──────┬──────┘     │
             │            │            │
     docker  │     docker │ stop    docker restart
      start  │            ▼            │
             │     ┌─────────────┐     │
             └─────│   STOPPED   │─────┘
                   └──────┬──────┘
                          │ docker rm
                          ▼
                   ┌─────────────┐
                   │   DELETED   │  ← container dan writable layer-nya hilang
                   └─────────────┘
                                       (image tetap ada di lokal)
```

**Poin penting:** saat container dihapus (`docker rm`), hanya writable layer-nya yang hilang. Image sumber tetap utuh dan bisa dipakai membuat container baru.

---

## 7. Ringkasan

| Pertanyaan | Jawaban |
|---|---|
| Apakah container butuh image? | **Ya, selalu.** Container tidak bisa dibuat tanpa image. |
| Apakah image berubah saat container berjalan? | **Tidak.** Image selalu read-only. Perubahan ditulis di writable layer container. |
| Berapa container yang bisa dibuat dari satu image? | **Tidak terbatas**, selama resource host mencukupi. |
| Apa yang terjadi pada data jika container dihapus? | **Hilang**, kecuali data disimpan di Docker Volume. |
| Bagaimana menyimpan perubahan container sebagai image baru? | Gunakan `docker commit <container> <nama-image-baru>`. |
