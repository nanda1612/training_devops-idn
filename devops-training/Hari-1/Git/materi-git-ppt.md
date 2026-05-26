# Materi Git

---

## Slide 1: Dasar Git

**Konfigurasi & Repository**
```bash
git config --global user.name "Nama"
git config --global user.email "email@kamu.com"
git init
```

**Status & Menyimpan Perubahan**
```bash
git status
git add .
git commit -m "Pesan commit"
```

**Status File:**
- `Untracked` → belum dikenal Git
- `Modified` → sudah diubah, belum disimpan
- `Staged` → siap di-commit

**Tips:**
- Buat `.gitignore` untuk mengabaikan file seperti `.env`, `node_modules/`
- Gunakan `git mv` saat rename/pindah file
- `git rm --cached` untuk stop tracking file tanpa menghapusnya

---

## Slide 2: Branch, Merge & Tag

**Branching**
```bash
git branch <nama>     # buat branch
git switch <nama>     # pindah branch
```

**Merge & Cek Perbedaan**
```bash
git switch main
git merge <nama-branch>
git diff dev main
```

**Membatalkan Perubahan**
```bash
git restore --staged <file>   # batalkan git add
git reset <file>              # buang perubahan (permanen!)
git revert <commit-hash>      # batalkan commit dengan aman
```

**Tagging (penanda rilis)**
```bash
git tag -a v1.0.0 -m "Rilis versi 1.0.0"
```

> Konflik terjadi saat dua branch ubah baris yang sama — selesaikan manual, hapus marker `<<<<<<<`, lalu commit.
