# Tutorial GitHub: Source Code Management

## Daftar Isi
1. [Apa itu GitHub?](#apa-itu-github)
2. [Membuat Akun GitHub](#membuat-akun-github)
3. [Git Remote](#git-remote)
4. [Kolaborasi dengan GitHub Organization](#kolaborasi-dengan-github-organization)

---

## Apa itu GitHub?

GitHub adalah platform hosting untuk version control berbasis Git. GitHub memungkinkan developer menyimpan kode secara online, berkolaborasi dengan tim, serta melacak perubahan kode dari waktu ke waktu.

**Konsep utama:**
- **Repository (Repo)** — folder proyek yang disimpan di GitHub
- **Remote** — referensi ke repository yang ada di server (GitHub), bukan di komputer lokal
- **Push** — mengirim perubahan dari lokal ke remote
- **Pull/Fetch** — mengambil perubahan dari remote ke lokal

---

## Membuat Akun GitHub

1. Buka [https://github.com](https://github.com)
2. Klik tombol **Sign up**
3. Isi form:
   - Username (contoh: `nanda-dev`)
   - Email address
   - Password
4. Verifikasi akun melalui email yang dikirimkan GitHub
5. Pilih plan **Free** untuk penggunaan dasar

---

## Git Remote

`git remote` adalah perintah untuk mengelola koneksi antara repository lokal dengan repository yang ada di GitHub (atau server lainnya).

### 1. Melihat Daftar Remote

Perintah ini menampilkan semua remote yang terdaftar beserta URL-nya.

```bash
git remote -v
```

**Contoh output:**
```
origin  https://github.com/username/nama-repo.git (fetch)
origin  https://github.com/username/nama-repo.git (push)
```

---

### 2. Menambahkan Remote

Gunakan perintah ini untuk menghubungkan repository lokal dengan repository di GitHub.

```bash
git remote add <nama-remote> <link-repository>
```

**Contoh:**
```bash
git remote add origin https://github.com/username/my-project.git
```

> **Konvensi:** Nama remote biasanya menggunakan `origin` untuk repository utama.

**Langkah lengkap menghubungkan repo lokal ke GitHub:**
```bash
# 1. Inisialisasi git di folder proyek
git init

# 2. Tambahkan semua file
git add .

# 3. Buat commit pertama
git commit -m "initial commit"

# 4. Tambahkan remote
git remote add origin https://github.com/username/my-project.git

# 5. Push ke GitHub
git push origin main
```

---

### 3. Menghapus Remote

Gunakan perintah ini jika ingin menghapus koneksi remote yang sudah tidak digunakan.

```bash
git remote remove <nama-remote>
```

**Contoh:**
```bash
git remote remove origin
```

---

### 4. Push — Mengirim Perubahan ke GitHub

Mengirim commit dari branch lokal ke repository di GitHub.

```bash
git push <remote> <branch>
```

**Contoh:**
```bash
# Push branch main ke remote origin
git push origin main

# Push pertama kali dengan set upstream (agar cukup `git push` saja berikutnya)
git push -u origin main
```

**Alur kerja push:**
```
[Edit file] → git add . → git commit -m "pesan" → git push origin main
```

---

### 5. Fetch — Mengambil Perubahan tanpa Merge

`git fetch` mengunduh perubahan dari GitHub ke lokal **tanpa** langsung menggabungkannya ke branch aktif.

```bash
git fetch <remote> <branch>
```

**Contoh:**
```bash
git fetch origin main
```

> Gunakan `git fetch` jika ingin melihat perubahan terlebih dahulu sebelum menggabungkannya.

---

### 6. Pull — Mengambil dan Menggabungkan Perubahan

`git pull` mengunduh perubahan dari GitHub **sekaligus** menggabungkannya ke branch aktif (fetch + merge).

```bash
git pull <remote> <branch>
```

**Contoh:**
```bash
git pull origin main
```

**Perbedaan fetch vs pull:**

| Perintah | Unduh | Merge otomatis |
|----------|-------|----------------|
| `git fetch` | Ya | Tidak |
| `git pull` | Ya | Ya |

---

## Kolaborasi dengan GitHub Organization

GitHub Organization memungkinkan tim bekerja bersama dalam satu wadah yang terorganisir, lengkap dengan manajemen akses dan repository.

### Langkah-langkah Setup Kolaborasi

#### 1. Membuat GitHub Organization

1. Login ke GitHub
2. Klik ikon profil → **Your organizations**
3. Klik **New organization**
4. Pilih plan (Free untuk tim kecil)
5. Isi nama organization dan email
6. Klik **Create organization**

---

#### 2. Membuat Team

Team digunakan untuk mengelompokkan anggota berdasarkan peran atau divisi.

1. Buka halaman organization
2. Klik tab **Teams**
3. Klik **New team**
4. Isi nama team (contoh: `backend`, `frontend`, `devops`)
5. Tentukan visibilitas: **Visible** (publik dalam org) atau **Secret**
6. Klik **Create team**

---

#### 3. Menambahkan Anggota ke Team

1. Masuk ke halaman team yang dibuat
2. Klik tab **Members**
3. Klik **Add a member**
4. Ketik username GitHub anggota
5. Klik **Invite** — anggota akan menerima email undangan

---

#### 4. Membuat Aturan untuk Anggota Team

Atur hak akses (permission) anggota terhadap repository:

| Role | Hak Akses |
|------|-----------|
| **Read** | Hanya bisa melihat dan clone repository |
| **Triage** | Bisa mengelola issues dan PR, tidak bisa push |
| **Write** | Bisa push ke branch, membuat PR |
| **Maintain** | Manajemen repository tanpa akses pengaturan sensitif |
| **Admin** | Akses penuh termasuk pengaturan repository |

**Cara mengatur permission:**
1. Buka halaman team → tab **Repositories**
2. Klik nama repository
3. Pilih level permission yang sesuai

**Branch Protection Rules** (opsional tapi disarankan):
1. Buka repository → **Settings** → **Branches**
2. Klik **Add rule**
3. Isi nama branch (contoh: `main`)
4. Centang opsi seperti **Require pull request reviews** agar setiap perubahan harus di-review sebelum di-merge

---

#### 5. Membuat Repository

1. Buka halaman organization
2. Klik **New repository** atau tab **Repositories** → **New**
3. Isi detail:
   - **Repository name** (contoh: `backend-api`)
   - **Description** (opsional)
   - Pilih **Public** atau **Private**
   - Centang **Add a README file** jika perlu
4. Klik **Create repository**

---

#### 6. Menambahkan Kolaborasi ke Repository

Ada dua cara menambahkan akses ke repository:

**Cara A — Melalui Team:**
1. Buka repository → **Settings** → **Collaborators and teams**
2. Klik **Add teams**
3. Cari nama team → pilih level permission

**Cara B — Tambah individu langsung:**
1. Buka repository → **Settings** → **Collaborators and teams**
2. Klik **Add people**
3. Ketik username → pilih permission → klik **Add**

---

### Alur Kerja Kolaborasi Tim (Workflow)

```
[Developer A]                    [GitHub]                    [Developer B]
     |                               |                               |
     |-- git clone repo -----------> |                               |
     |                               |                               |
     |                               | <-- git clone repo -----------|
     |                               |                               |
     |-- (edit file) --------------> |                               |
     |-- git push origin feature --> |                               |
     |                               |                               |
     |-- (buat Pull Request) ------> |                               |
     |                               | -- (review PR) ------------> |
     |                               | <-- (approve PR) ------------|
     |                               |                               |
     |-- git merge ke main --------> |                               |
     |                               |                               |
     |                               | -- git pull origin main ---> |
```

**Perintah dasar untuk kontribusi ke repository organisasi:**

```bash
# Clone repository organisasi
git clone https://github.com/nama-org/nama-repo.git

# Masuk ke folder
cd nama-repo

# Buat branch baru untuk fitur/perbaikan
git checkout -b feature/nama-fitur

# Edit file, lalu add dan commit
git add .
git commit -m "feat: tambah fitur X"

# Push branch ke GitHub
git push origin feature/nama-fitur

# Buat Pull Request di GitHub untuk di-review sebelum merge ke main
```

---

## Ringkasan Perintah

| Perintah | Fungsi |
|----------|--------|
| `git remote -v` | Lihat daftar remote |
| `git remote add origin <url>` | Tambah remote baru |
| `git remote remove origin` | Hapus remote |
| `git push origin main` | Kirim perubahan ke GitHub |
| `git fetch origin main` | Unduh perubahan tanpa merge |
| `git pull origin main` | Unduh dan merge perubahan |
| `git clone <url>` | Salin repository dari GitHub ke lokal |
