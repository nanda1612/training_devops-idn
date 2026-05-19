# Penjelasan Materi Git

---

## 1. Konfigurasi Awal

Sebelum menggunakan Git, kita harus memberitahu Git siapa kita. Identitas ini akan tercatat di setiap commit yang dibuat, sehingga anggota tim bisa mengetahui siapa yang melakukan perubahan.

`--global` artinya konfigurasi ini berlaku untuk semua repository di komputer. Jika hanya ingin berlaku di satu project, gunakan `--local` di dalam folder repository tersebut.

File konfigurasi disimpan di `~/.gitconfig` dan bisa dilihat kapan saja dengan `git config --global --list`.

---

## 2. Membuat Repository

Repository adalah tempat Git menyimpan seluruh riwayat perubahan project. Ketika kita menjalankan `git init`, Git membuat folder tersembunyi bernama `.git` di dalam project kita.

Folder `.git` inilah yang menjadi "otak" dari repository — berisi semua snapshot, log commit, konfigurasi, dan informasi branch. **Jangan pernah menghapus folder `.git`** karena seluruh riwayat project akan hilang.

`git init` bisa dijalankan di folder kosong (project baru) maupun folder yang sudah berisi file (project lama yang baru mau di-track).

---

## 3. Mengelola File

### Status File

Git membagi kondisi file ke dalam tiga status:

- **Untracked** — file baru yang belum pernah dikenal Git. Git tidak memantau file ini sama sekali.
- **Modified** — file yang sudah dikenal Git namun baru saja diubah. Perubahan ini belum "disimpan" ke repository.
- **Staged** — file yang sudah disiapkan untuk di-commit. File ini ada di *staging area* (disebut juga *index*).

`git status` adalah perintah yang paling sering digunakan. Selalu jalankan ini sebelum `git add` maupun `git commit` untuk memastikan file yang akan disimpan sudah benar.

### Staging dan Commit

Git menggunakan dua langkah untuk menyimpan perubahan: **add** lalu **commit**. Ini disengaja agar kita bisa memilih perubahan mana yang ingin digabungkan dalam satu commit, meskipun ada banyak file yang diubah sekaligus.

`git add .` menambahkan semua file yang berubah sekaligus. Ini praktis, tapi hati-hati — pastikan tidak ada file sensitif (seperti `.env` atau password) yang ikut ter-add.

Pesan commit sebaiknya singkat dan jelas menjelaskan **apa** yang berubah. Gunakan gaya imperatif: "Tambah fitur login", "Fix bug validasi", bukan "Saya menambahkan..." atau "Sudah fix...".

### Mengabaikan File (.gitignore)

Tidak semua file perlu di-track oleh Git. File hasil build (`dist/`, `*.class`), dependency (`node_modules/`, `vendor/`), dan file rahasia (`.env`, `*.key`) sebaiknya tidak masuk ke repository.

`.gitignore` berisi daftar pola nama file atau folder yang akan diabaikan Git secara otomatis. File ini sendiri harus di-commit agar berlaku untuk semua anggota tim.

### Operasi File

Saat merename atau memindahkan file, gunakan `git mv` bukan `mv` biasa. Jika menggunakan `mv` biasa, Git akan menganggap file lama dihapus dan ada file baru yang belum di-track — Git tidak tahu itu adalah file yang sama yang dipindahkan.

`git rm --cached` berguna untuk berhenti men-track file tanpa menghapusnya dari komputer. Kasus umum: file `.env` yang terlanjur di-commit dan ingin dikeluarkan dari repository.

---

## 4. Membatalkan Perubahan

### Unstage (Batalkan git add)

`git restore --staged` memindahkan file kembali dari staging area ke working directory. File tidak berubah — hanya statusnya yang kembali dari *staged* menjadi *modified*.

### Buang Perubahan di Working Directory

`git reset` membuang perubahan pada file yang belum di-stage. Gunakan dengan hati-hati karena perubahan yang dibuang **tidak bisa dikembalikan**.

### Revert Commit

`git revert` adalah cara aman untuk membatalkan commit yang sudah ada. Berbeda dengan menghapus commit, `revert` membuat **commit baru** yang isinya adalah kebalikan dari commit yang ingin dibatalkan.

Ini penting ketika bekerja dalam tim — kita tidak mengubah riwayat yang sudah ada, sehingga anggota tim lain tidak terdampak. Flag `-n` (no-commit) membuat Git menyiapkan perubahan revert tanpa langsung commit, sehingga kita bisa memeriksa hasilnya terlebih dahulu sebelum `git revert --continue`.

---

## 5. Branching dan Merging

### Apa itu Branch?

Branch adalah "jalur" pengembangan yang terpisah dari jalur utama. Bayangkan seperti salinan project yang bisa kita ubah bebas tanpa merusak versi aslinya.

Branch utama biasanya bernama `main` (atau `master` di repository lama). Saat mengerjakan fitur baru atau memperbaiki bug, kita membuat branch baru, bekerja di sana, lalu menggabungkannya kembali ke `main` setelah selesai.

Manfaat branch:
- Fitur yang belum selesai tidak mengganggu kode yang sudah stabil di `main`
- Beberapa orang bisa bekerja pada fitur berbeda secara bersamaan
- Mudah membandingkan perbedaan antar versi sebelum digabungkan

### Merge

`git merge` menggabungkan perubahan dari satu branch ke branch lain. Sebelum merge, pastikan kita sudah berpindah ke branch tujuan (misalnya `main`) dengan `git switch main`.

### Konflik

Konflik terjadi ketika dua branch mengubah **baris yang sama** pada file yang sama. Git tidak bisa otomatis memilih versi mana yang benar, sehingga kita harus menyelesaikannya secara manual.

Git akan menandai bagian yang konflik dengan marker seperti ini:

```
<<<<<<< HEAD
kode dari branch aktif (main)
=======
kode dari branch yang di-merge (dev)
>>>>>>> dev
```

Tugas kita adalah memilih atau menggabungkan kedua versi tersebut, lalu menghapus semua marker (`<<<<<<<`, `=======`, `>>>>>>>`), kemudian commit hasilnya.

### git diff

`git diff dev main` menampilkan perbedaan antara dua branch sebelum di-merge. Berguna untuk mereview perubahan dan memastikan tidak ada yang terlewat sebelum menggabungkan branch.

---

## 6. Tagging

Tag adalah penanda permanen pada commit tertentu. Berbeda dengan branch yang bisa bergerak (setiap commit baru memajukan posisi branch), tag tetap menunjuk ke satu commit yang sama selamanya.

Tag paling umum digunakan untuk menandai **versi rilis**: `v1.0.0`, `v2.3.1`, dan seterusnya. Dengan tag, kita bisa dengan mudah kembali ke kondisi kode saat versi tertentu dirilis.

Flag `-a` membuat *annotated tag* yang menyimpan nama pembuat, tanggal, dan pesan — lebih lengkap dibanding *lightweight tag* (tanpa `-a`). Untuk rilis resmi, selalu gunakan annotated tag.
