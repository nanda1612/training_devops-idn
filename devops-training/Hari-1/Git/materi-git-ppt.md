# Materi Git

---

## Slide 1: Dasar Git

```bash
git config --global user.name "Nama"
git config --global user.email "email@kamu.com"
git init
git status
git add .
git commit -m "Pesan commit"
```

**Status File:** Untracked → Modified → Staged → Committed

- `.gitignore` — abaikan file seperti `.env`, `node_modules/`
- `git mv` — rename/pindah file dengan benar
- `git rm --cached` — stop tracking file tanpa menghapusnya

---

## Slide 2: Branch, Merge & Tag

```bash
git branch <nama>            # buat branch
git switch <nama>            # pindah branch
git merge <nama-branch>      # gabungkan ke branch aktif
git diff dev main            # lihat perbedaan antar branch
git restore --staged <file>  # batalkan git add
git revert <commit-hash>     # batalkan commit dengan aman
git tag -a v1.0.0 -m "Rilis" # buat tag rilis
```

- Konflik → selesaikan manual, hapus marker `<<<<<<<`, lalu commit
- `revert` lebih aman dari `reset` karena tidak mengubah riwayat
