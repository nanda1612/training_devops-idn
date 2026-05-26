# Penjelasan Materi Git

## 1. Konfigurasi Awal
Sebelum menggunakan Git, daftarkan identitas agar tercatat di setiap commit.
- `--global` berlaku untuk semua repository, `--local` hanya untuk satu project.

## 2. Membuat Repository
`git init` membuat folder `.git` sebagai "otak" repository — berisi semua riwayat, log, dan konfigurasi.
**Jangan hapus folder `.git`**, seluruh riwayat project akan hilang.

## 3. Mengelola File

**Status file:**
- **Untracked** — file baru, belum dikenal Git
- **Modified** — file diubah, belum disimpan ke repo
- **Staged** — file siap di-commit

Selalu jalankan `git status` sebelum `git add` atau `git commit`.

**Staging & Commit:**
Git menyimpan perubahan dalam dua langkah: `add` lalu `commit`. Ini memungkinkan kita memilih perubahan mana yang digabung dalam satu commit. Pesan commit harus singkat dan imperatif: *"Tambah fitur login"*, bukan *"Saya menambahkan..."*.

**Mengabaikan File (.gitignore):**
File build, dependency, dan file rahasia (`.env`) sebaiknya tidak di-track. Daftarkan polanya di `.gitignore`, lalu commit file tersebut agar berlaku untuk semua anggota tim.

**Operasi File:**
Gunakan `git mv` saat rename/memindahkan file agar Git mengenali perpindahannya. Gunakan `git rm --cached` untuk berhenti men-track file tanpa menghapusnya dari komputer.

## 4. Membatalkan Perubahan
- `git restore --staged` — kembalikan file dari staged ke modified (file tidak berubah)
- `git reset` — buang perubahan di working directory (**tidak bisa dikembalikan**)
- `git revert` — batalkan commit dengan membuat **commit baru** yang aman untuk tim

## 5. Branching dan Merging
Branch adalah jalur pengembangan terpisah agar fitur yang belum selesai tidak mengganggu kode stabil di `main`.

`git merge` menggabungkan branch ke branch tujuan. **Konflik** terjadi saat dua branch mengubah baris yang sama — selesaikan manual, hapus marker (`<<<<<<<`, `=======`, `>>>>>>>`), lalu commit.

Gunakan `git diff dev main` untuk mereview perbedaan sebelum merge.

## 6. Tagging
Tag adalah penanda permanen pada commit, umumnya digunakan untuk menandai versi rilis (`v1.0.0`). Gunakan **annotated tag** (`-a`) untuk rilis resmi karena menyimpan nama pembuat, tanggal, dan pesan.
