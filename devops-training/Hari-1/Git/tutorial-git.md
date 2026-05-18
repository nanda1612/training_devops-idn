# Tutorial Git: Version Control System

> **Peserta:** DevOps Pemula | **Prasyarat:** `git --version`

```
Working Directory  →  Staging Area  →  Repository
   (edit file)         (git add)        (git commit)
```

---

## 1. Konfigurasi Awal

```bash
git config --global user.name "Nama Anda"
git config --global user.email "email@domain.com"
git config --global --list        # verifikasi
```

---

## 2. Membuat Repository

```bash
git init nama-proyek    # repo baru
git init                # folder yang sudah ada
ls -la                  # verifikasi: cek folder .git
```

---

## 3. Mengelola File

### Status & Staging

```bash
git status              # cek kondisi file
git add nama-file.txt   # tambah file tertentu
git add .               # tambah semua file
git commit -m "pesan"   # simpan ke repository
```

| Status | Artinya |
|--------|---------|
| `Untracked` | File baru, belum dikenal Git |
| `Modified` | File sudah diubah |
| `Staged` | Siap di-commit |

### Operasi File

```bash
git mv file-lama.txt file-baru.txt    # rename
git mv file.txt folder/file.txt       # pindah
git rm file.txt                       # hapus dari repo & disk
git rm --cached file.txt              # hapus dari repo saja
```

### Mengabaikan File (.gitignore)

```
node_modules/
.env
dist/
*.key
```

---

## 4. Membatalkan Perubahan

```bash
git restore --staged file.txt   # batalkan git add
git reset file.txt              # buang perubahan di working dir

git log --oneline               # lihat commit-id
git revert -n <commit-id>       # siapkan revert
git revert --continue           # selesaikan revert
```

> `revert` aman untuk branch yang sudah di-share — membuat commit baru, tidak menghapus riwayat.

---

## 5. Branching dan Merging

```
main  ──●──────────●── (merge)
          \       /
feature    ●──●──●
```

```bash
git branch                      # lihat daftar branch
git switch -c nama-fitur        # buat & pindah branch
git switch main                 # pindah branch

git diff dev main               # lihat perbedaan antar branch
git merge dev                   # merge dev ke branch aktif

git branch -d nama-fitur        # hapus branch
```

### Menyelesaikan Konflik

1. Edit file — hapus marker `<<<<<<<`, `=======`, `>>>>>>>`, pilih kode yang benar
2. `git add file.txt`
3. `git commit -m "Resolve konflik"`

---

## 6. Tagging

```bash
git tag                                         # lihat tag
git tag -a v1.0.0 -m "Rilis v1.0.0" <commit-id>  # buat tag
git tag -d v1.0.0                               # hapus tag
```

---

## 7. Latihan Praktik

**Latihan 1 — Commit pertama:**
```bash
git init latihan-git && cd latihan-git
echo "# Latihan Git" > README.md
git add README.md
git commit -m "Inisialisasi repository"
git log --oneline
```

**Latihan 2 — Branching:**
```bash
git switch -c fitur-login
echo "Form login" > login.html
git add login.html && git commit -m "Tambah halaman login"
git switch main && git merge fitur-login
git branch -d fitur-login
```

**Latihan 3 — Konflik:**
```bash
git switch -c branch-a
echo "Versi A" > config.txt && git add config.txt && git commit -m "Edit branch A"
git switch main
echo "Versi Main" > config.txt && git add config.txt && git commit -m "Edit main"
git merge branch-a    # selesaikan konflik secara manual
```

---

## Ringkasan Perintah

| Perintah | Fungsi |
|----------|--------|
| `git config --global user.name` | Set nama pengguna |
| `git init` | Inisialisasi repository |
| `git status` | Cek status file |
| `git add` / `git commit` | Stage & simpan perubahan |
| `git log --oneline` | Lihat riwayat commit |
| `git switch -c <nama>` | Buat & pindah branch |
| `git merge <branch>` | Gabungkan branch |
| `git revert -n <id>` | Batalkan commit |
| `git tag -a <versi>` | Buat tag versi |
