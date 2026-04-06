# 🚀 Praktikum RPL — [Kelas]-[Nomor Kelompok]

> Repositori resmi kelompok untuk mata kuliah **Rekayasa Perangkat Lunak**  
> Semester Genap 2024/2025 — Universitas [Nama Universitas]

---

## 📋 Daftar Isi

- [Tentang Repositori](#-tentang-repositori)
- [Anggota Tim](#-anggota-tim)
- [Struktur Folder](#-struktur-folder)
- [Topik Proyek](#-topik-proyek)
- [Teknologi yang Digunakan](#-teknologi-yang-digunakan)
- [Cara Berkontribusi](#-cara-berkontribusi)
- [Workflow Git](#-workflow-git)
- [Standar Commit Message](#-standar-commit-message)
- [Progress & Status](#-progress--status)
- [Kontak Tim](#-kontak-tim)

---

## 📌 Tentang Repositori

Repositori ini digunakan sebagai wadah kolaborasi tim dalam pengerjaan praktikum Rekayasa Perangkat Lunak. Seluruh dokumentasi, kode sumber, dan artefak pengujian dikelola di sini menggunakan **Git Feature Branch Workflow**.

| Atribut        | Detail                          |
|----------------|---------------------------------|
| 🏫 Kelas       | [Kelas A / B]                   |
| 👥 Kelompok    | [Nomor Kelompok]                |
| 📅 Semester    | Genap 2024/2025                 |
| 🌿 Branch Utama| `dev`                           |

---

## 👥 Anggota Tim

| No | Nama Lengkap | NIM | Peran | GitHub |
|----|--------------|-----|-------|--------|
| 1  | [Nama Anggota 1] | [NIM] | Ketua / Project Manager | [@username](https://github.com/username) |
| 2  | [Nama Anggota 2] | [NIM] | Backend Developer | [@username](https://github.com/username) |
| 3  | [Nama Anggota 3] | [NIM] | Frontend Developer | [@username](https://github.com/username) |
| 4  | [Nama Anggota 4] | [NIM] | QA / Dokumentasi | [@username](https://github.com/username) |

---

## 📁 Struktur Folder

```
praktikum-rpl-[kelas]-[nomor]/
│
├── docs/                        # Dokumentasi proyek
│   ├── team-contract.md         # Kontrak tim
│   ├── requirements/            # Dokumen kebutuhan sistem
│   └── diagrams/                # Diagram UML, ERD, dll.
│
├── src/                         # Kode sumber aplikasi
│   ├── main/                    # Kode utama
│   └── utils/                   # Fungsi utilitas
│
├── tests/                       # File pengujian
│   ├── unit/                    # Unit test
│   └── integration/             # Integration test
│
├── .gitignore                   # File yang diabaikan Git
└── README.md                    # Dokumentasi utama (file ini)
```

---

## 💡 Topik Proyek

> **Status:** 🔄 PENDING / ✅ DISETUJUI *(sesuaikan)*

### Judul Sementara
> **[Nama Aplikasi / Sistem]**

### Deskripsi Singkat
> *[Tuliskan deskripsi singkat tentang studi kasus atau sistem yang akan dibangun, misalnya: "Sistem manajemen peminjaman buku perpustakaan berbasis web untuk memudahkan pengelolaan koleksi dan transaksi peminjaman."]*

| Atribut         | Detail                          |
|-----------------|---------------------------------|
| 🗂️ Domain       | [Pendidikan / Kesehatan / UMKM / dll.] |
| 👤 Aktor Utama  | [Admin, User, dll.]             |
| ⭐ Fitur Inti   | [Fitur 1, Fitur 2, Fitur 3]     |

---

## 🛠️ Teknologi yang Digunakan

> *(Sesuaikan dengan stack teknologi kelompok)*

| Kategori     | Teknologi       |
|--------------|-----------------|
| Backend      | -               |
| Frontend     | -               |
| Database     | -               |
| Version Control | Git & GitHub |
| Editor       | VS Code         |

---

## 🤝 Cara Berkontribusi

Semua anggota tim wajib mengikuti alur kerja berikut sebelum membuat perubahan pada repositori:

1. **Pastikan branch `dev` sudah terbaru**
   ```bash
   git checkout dev
   git pull origin dev
   ```

2. **Buat branch baru untuk fitur/tugas**
   ```bash
   git checkout -b feature/nama-fitur
   ```

3. **Lakukan perubahan, lalu commit**
   ```bash
   git add .
   git commit -m "feat: deskripsi perubahan singkat"
   ```

4. **Push branch ke GitHub**
   ```bash
   git push origin feature/nama-fitur
   ```

5. **Buat Pull Request ke branch `dev`** melalui GitHub  
   → Minta minimal **1 anggota lain** untuk melakukan review  
   → Merge setelah disetujui

---

## 🌿 Workflow Git

Repositori ini menggunakan **Feature Branch Workflow**:

```
main
 └── dev  ← branch default & utama pengembangan
      ├── feature/add-[nama]-info
      ├── feature/nama-fitur-lain
      └── fix/perbaikan-bug
```

- Branch `main` → versi stabil / rilis
- Branch `dev` → integrasi pengembangan aktif
- Branch `feature/*` → pengerjaan fitur per anggota

---

## ✍️ Standar Commit Message

Gunakan format berikut untuk semua commit:

```
<type>: <deskripsi singkat dalam bahasa Indonesia atau Inggris>
```

| Type       | Digunakan untuk                            |
|------------|--------------------------------------------|
| `feat`     | Menambahkan fitur baru                     |
| `fix`      | Memperbaiki bug                            |
| `docs`     | Perubahan dokumentasi                      |
| `style`    | Perubahan format/tampilan (bukan logika)   |
| `refactor` | Refactoring kode tanpa mengubah fungsi     |
| `test`     | Menambah atau mengubah test                |
| `chore`    | Update konfigurasi, dependency, dll.       |

**Contoh:**
```
feat: tambah halaman login pengguna
fix: perbaiki validasi input form registrasi
docs: update README dengan instruksi setup
```

---

## 📊 Progress & Status

### Praktikum 1 — Setup & Git Workflow

| Tugas                                      | Status |
|--------------------------------------------|--------|
| Kelompok terbentuk dan ketua ditentukan    | ✅ / 🔄 |
| Kontrak tim ditulis (`team-contract.md`)   | ✅ / 🔄 |
| Repositori GitHub dibuat                   | ✅ / 🔄 |
| Branch `dev` dibuat dan dijadikan default  | ✅ / 🔄 |
| Setiap anggota merge minimal 1 Pull Request| ✅ / 🔄 |
| Topik diregistrasi di Google Sheet         | ✅ / 🔄 |
| Issue `[SUBMISSION] P1 Evidence` dibuat    | ✅ / 🔄 |

> Keterangan: ✅ Selesai &nbsp;|&nbsp; 🔄 Dalam Proses &nbsp;|&nbsp; ❌ Belum Dimulai

---

## 📞 Kontak Tim

| Nama | Email / WhatsApp |
|------|-----------------|
| [Nama 1] | [email@example.com] |
| [Nama 2] | [email@example.com] |
| [Nama 3] | [email@example.com] |
| [Nama 4] | [email@example.com] |

---
