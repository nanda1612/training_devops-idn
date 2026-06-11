# Git: Version Control System
### *"Menyimpan dan melacak perubahan kode"*

---

## Apa itu Git?

**Git** adalah sistem *version control* terdistribusi yang dibuat oleh **Linus Torvalds** pada tahun 2005 — orang yang sama yang menciptakan kernel Linux.

Git bekerja dengan cara **merekam snapshot** dari seluruh isi project setiap kali kamu melakukan commit. Berbeda dengan sistem lama yang hanya menyimpan daftar perubahan per file, Git menyimpan kondisi penuh project di setiap titik waktu.

### Karakteristik utama Git:

| Karakteristik | Penjelasan |
|---------------|------------|
| **Terdistribusi** | Setiap developer punya salinan penuh repository di komputernya sendiri — tidak bergantung server |
| **Cepat** | Hampir semua operasi berjalan lokal, tanpa perlu koneksi internet |
| **Aman** | Setiap commit diberi hash unik (SHA-1) — data tidak bisa diubah tanpa terdeteksi |
| **Non-linear** | Mendukung ribuan branch paralel yang bisa digabungkan kapan saja |
| **Gratis & Open Source** | Bebas digunakan untuk project apapun |

### Git vs tanpa Git

```
Tanpa Git                          Dengan Git
──────────────────────────────     ──────────────────────────────────────
Satu folder, banyak file duplikat  Satu folder, semua versi tersimpan
Tidak tahu siapa yang mengubah     Setiap perubahan tercatat + nama + waktu
Sulit kerja tim — file saling      Tim bisa kerja paralel tanpa bentrok
  timpa satu sama lain             (dengan branch)
Tidak bisa balik ke versi lama     Bisa kembali ke titik mana pun
```

> Git bukan hanya alat untuk programmer. Desainer, penulis teknis, dan data scientist
> pun menggunakan Git untuk melacak perubahan pada file yang mereka kerjakan.

---

## Mengapa Kita Butuh Git?

Bayangkan kamu mengerjakan skripsi tanpa Git:

```
Skripsi.docx
Skripsi-FINAL.docx
Skripsi-FINAL-beneran.docx
Skripsi-FINAL-beneran-revisi.docx
Skripsi-FINAL-fix-fiksss.docx   ← yang mana ini?
```

**Dengan Git, cukup 1 file.** Semua riwayat perubahan tersimpan rapi di dalam repository.
Kamu bisa kembali ke versi mana pun kapan pun kamu mau.

---

## Alur Kerja Git

```
  [Kamu edit file]     [git add]      [git commit]
  Working Directory  ──────────>  Staging Area  ──────────>  Repository
       (lokal)                      (antrian)                (tersimpan)
```

> **Analogi:** Staging area itu seperti troli belanja. Kamu pilih dulu barang yang mau dibeli,
> baru bayar (commit) semuanya sekaligus.

---

## 1. Persiapan Sebelum Mulai

Sebelum melakukan konfigurasi, pastikan Git sudah terinstal di komputer kamu.

### Instalasi Git

| Sistem Operasi | Cara Instalasi |
|----------------|----------------|
| **Linux (Ubuntu/Debian)** | `sudo apt update && sudo apt install git` |
| **Linux (Fedora/RHEL)** | `sudo dnf install git` |
| **macOS** | `brew install git` atau install Xcode Command Line Tools |
| **Windows** | Download dari [git-scm.com](https://git-scm.com/download/win), jalankan installer |

### Verifikasi Instalasi

Setelah instalasi, pastikan Git dapat dijalankan:

```bash
git --version
# Contoh output: git version 2.43.0
```

Jika muncul nomor versi, Git sudah siap digunakan.

### Prasyarat Lainnya

- **Terminal / Command Prompt** — tempat menjalankan perintah Git
- **Text Editor** (opsional, tapi disarankan) — untuk menulis kode dan pesan commit.
  Rekomendasi: VS Code, Neovim, atau Nano
- **Akun Git remote** (opsional untuk tahap ini) — GitHub, GitLab, atau Bitbucket
  diperlukan nanti saat ingin menyimpan repository ke server

---

## 2. Konfigurasi Awal

Sebelum mulai, kita harus kenalkan diri kita ke Git.
Identitas ini akan tercatat di setiap commit yang kita buat.

```bash
git config --global user.name  "Nama Kamu"
git config --global user.email "email@domain.com"
```

Cek apakah konfigurasi sudah tersimpan:

```bash
git config --global --list

# Lokasi file konfigurasi disimpan di:
cat $HOME/.gitconfig
```

| Flag | Cakupan |
|------|---------|
| `--global` | Berlaku untuk semua repository di komputer |
| `--local` | Hanya berlaku di satu repository saat ini |

---

## 3. Membuat Repository

Repository adalah "gudang" tempat Git menyimpan seluruh riwayat project.

```bash
# Buat repository baru
git init nama-proyek

# Atau jadikan folder yang sudah ada sebagai repository
git init

# Verifikasi — harus ada folder .git
ls -la
```

> **Penting:** Folder `.git` adalah "otak" dari repository.
> Di dalamnya tersimpan semua snapshot, log commit, dan konfigurasi.
> **Jangan pernah hapus folder ini** — seluruh riwayat project akan hilang selamanya.

---

## 4. Mengelola File

### Status File

Git membagi kondisi file ke dalam tiga status:

```
  Untracked  →  (git add)  →  Staged  →  (git commit)  →  Committed
  (baru ada)                 (antri)                       (tersimpan)
      ↑                                                         |
      └─────────────────  (edit file)  ←  Modified  ←──────────┘
```

| Status | Artinya |
|--------|---------|
| `Untracked` | File baru, belum pernah dikenal Git |
| `Modified` | File diubah setelah commit terakhir |
| `Staged` | File siap masuk commit berikutnya |

**Selalu cek status sebelum add atau commit:**

```bash
git status
```

---

### Menambahkan & Menyimpan File

```bash
# Tambah file tertentu ke staging area
git add nama-file.txt

# Tambah SEMUA perubahan sekaligus
git add .

# Simpan ke repository dengan pesan
git commit -m "Tambah halaman login"
```

**Cara menulis pesan commit yang baik:**

```
✅ "Tambah fitur login"          ← imperatif, spesifik
✅ "Fix bug validasi form"
✅ "Update konfigurasi database"

❌ "Saya menambahkan fitur..."   ← bertele-tele
❌ "fix"                          ← tidak informatif
❌ "asd"                          ← tidak bermakna
```

---

### Mengabaikan File (.gitignore)

Tidak semua file perlu masuk repository. File berikut sebaiknya **diabaikan**:

```bash
# Isi file .gitignore
node_modules/     # dependency (besar, bisa di-install ulang)
dist/             # hasil build
.env              # variabel rahasia (password, API key)
*.key             # file kunci enkripsi
*.log             # log aplikasi
```

> **Ingat:** File `.gitignore` itu sendiri harus di-commit,
> agar aturan pengabaian berlaku untuk seluruh anggota tim.

---

### Operasi File

```bash
# Rename atau pindahkan file (Git-aware)
git mv file-lama.txt file-baru.txt
git mv file.txt folder/file.txt

# Hapus file dari repository DAN dari disk
git rm file.txt

# Hapus dari repository SAJA (file tetap ada di komputer)
git rm --cached file.txt
```

> **Kenapa harus pakai `git mv`, bukan `mv` biasa?**
> Kalau pakai `mv` biasa, Git menganggap file lama *dihapus* dan ada file baru yang belum di-track.
> Git tidak tahu bahwa itu file yang sama yang dipindahkan.

---

## 5. Membatalkan Perubahan

Tiga skenario umum saat ingin membatalkan sesuatu:

```
SKENARIO 1: Sudah git add, mau batalkan (belum commit)
──────────────────────────────────────────────────────
git restore --staged nama-file.txt
→ File kembali ke status Modified (isi file tidak berubah)


SKENARIO 2: Buang perubahan di working directory (belum di-add)
─────────────────────────────────────────────────────────────────
git reset nama-file.txt
→ HATI-HATI: perubahan hilang permanen, tidak bisa di-undo


SKENARIO 3: Batalkan commit yang sudah ada (cara aman)
────────────────────────────────────────────────────────
git log --oneline             ← cari commit-id yang mau dibatalkan
git revert -n <commit-id>     ← siapkan pembatalan
git revert --continue         ← selesaikan
→ Git membuat COMMIT BARU yang isinya kebalikan dari commit lama
→ Riwayat lama tetap ada — aman untuk dipakai dalam tim
```

---

## 6. Branching dan Merging

### Apa itu Branch?

Branch adalah jalur pengembangan yang terpisah dari jalur utama (`main`).

```
main    ──●──────────────────────────●── (setelah merge)
           \                        /
feature     ●──●──●──●──●──●──●──●
            (kerjakan fitur di sini)
```

**Manfaat branch:**
- Fitur yang belum selesai tidak mengganggu kode stabil di `main`
- Beberapa developer bisa kerja paralel di branch berbeda
- Mudah membandingkan dan mereview perubahan sebelum digabung

---

### Perintah Branch

```bash
# Lihat semua branch
git branch

# Buat branch baru
git branch nama-fitur

# Buat branch baru SEKALIGUS pindah ke sana
git switch -c nama-fitur

# Pindah branch
git switch main

# Hapus branch (setelah selesai di-merge)
git branch -d nama-fitur
```

---

### Merge: Menggabungkan Branch

```bash
# 1. Pindah dulu ke branch tujuan
git switch main

# 2. Lihat perbedaan sebelum merge (opsional tapi disarankan)
git diff dev main

# 3. Gabungkan
git merge dev
```

---

### Menyelesaikan Konflik

Konflik terjadi ketika **dua branch mengubah baris yang sama** pada file yang sama.
Git tidak bisa memilih sendiri versi mana yang benar — kita harus putuskan manual.

Git akan menandai bagian yang konflik seperti ini:

```
<<<<<<< HEAD
kode dari branch aktif (main)
=======
kode dari branch yang di-merge (dev)
>>>>>>> dev
```

**Langkah menyelesaikan konflik:**

```
1. Buka file yang konflik di editor
2. Pilih versi yang benar (atau gabungkan keduanya)
3. Hapus semua marker: <<<<<<<, =======, >>>>>>>
4. git add nama-file.txt
5. git commit -m "Resolve konflik antara main dan dev"
```

---

## 7. Tagging

Tag adalah **penanda permanen** pada commit tertentu. Berbeda dengan branch yang terus bergerak maju setiap ada commit baru, tag **selalu menunjuk ke satu commit yang sama** selamanya.

Tag paling sering digunakan untuk menandai **versi rilis**: `v1.0.0`, `v2.3.1`, dst.

```bash
# Lihat semua tag
git tag

# Buat annotated tag (direkomendasikan untuk rilis resmi)
git tag -a v1.0.0 -m "Rilis versi 1.0.0" <commit-id>

# Hapus tag
git tag -d v1.0.0
```

> **Annotated tag** (`-a`) menyimpan nama pembuat, tanggal, dan pesan.
> Lebih lengkap dibanding *lightweight tag* (tanpa `-a`).
> Untuk rilis resmi, selalu gunakan annotated tag.

---

## Ringkasan Perintah

| Perintah | Fungsi |
|----------|--------|
| `git config --global user.name` | Set nama pengguna |
| `git init` | Inisialisasi repository |
| `git status` | Cek status file |
| `git add .` | Tambah semua perubahan ke staging |
| `git commit -m "pesan"` | Simpan perubahan ke repository |
| `git log --oneline` | Lihat riwayat commit (ringkas) |
| `git mv <lama> <baru>` | Rename / pindah file |
| `git rm --cached <file>` | Stop tracking file |
| `git restore --staged <file>` | Batalkan git add |
| `git revert -n <id>` | Batalkan commit (aman) |
| `git branch <nama>` | Buat branch baru |
| `git switch -c <nama>` | Buat & pindah branch |
| `git merge <branch>` | Gabungkan branch |
| `git diff <a> <b>` | Lihat perbedaan antar branch |
| `git tag -a <versi>` | Buat tag versi rilis |

---

## Latihan Praktik

### Latihan 1 — Commit Pertama

```bash
git init latihan-git && cd latihan-git
echo "# Latihan Git" > README.md
git add README.md
git commit -m "Inisialisasi repository"
git log --oneline
```

---

### Latihan 2 — Branching & Merge

```bash
git switch -c fitur-login
echo "Form login" > login.html
git add login.html && git commit -m "Tambah halaman login"
git switch main
git merge fitur-login
git branch -d fitur-login
```

---

### Latihan 3 — Simulasi Konflik

```bash
# Buat konflik antara dua branch
git switch -c branch-a
echo "Versi A" > config.txt && git add config.txt && git commit -m "Edit di branch A"

git switch main
echo "Versi Main" > config.txt && git add config.txt && git commit -m "Edit di main"

git merge branch-a
# → Konflik! Selesaikan secara manual, lalu commit hasilnya
```

---

*Referensi: docs-git.md — Git Version Control System Command Reference*
