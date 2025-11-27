# File Processing dalam Pemrograman C

## 1. Konsep Dasar File Processing

**File Processing** adalah teknik untuk menyimpan dan membaca data dari media penyimpanan (harddisk, flashdisk, dll).

### Contoh Aplikasi:
* Save game progress
* File konfigurasi program
* Database sederhana
* Laporan dan log system

---

## 2. Jenis-Jenis File

### A. File Plaintext (Text Files)
* **Ciri**: Dapat dibaca manusia dengan text editor
* **Format**: ASCII/UTF-8 encoding
* **Contoh ekstensi**: `.txt`, `.csv`, `.html`, `.c`, `.cpp`

### B. File Binary (Binary Files)
* **Ciri**: Hanya bisa dibaca program tertentu
* **Format**: Data biner mentah
* **Contoh ekstensi**: `.exe`, `.jpg`, `.dat`, `.mp3`

---

## 3. Struktur FILE dalam C
```c
// FILE adalah struktur yang sudah didefinisikan dalam stdio.h
// Kita menggunakan pointer ke struktur tersebut: FILE*
```

> **TIDAK PERLU** mendefinisikan ulang struktur FILE karena sudah built-in.

---

## 4. Fungsi Dasar File Processing

### A. Membuka File - `fopen()`
```c
FILE* fopen(const char* filename, const char* mode);
```

**Parameter:**
* `filename`: Nama file (contoh: "data.txt")
* `mode`: Mode akses (contoh: "r", "w", "a")

**Return Value:**
* **Berhasil**: Pointer ke FILE
* **Gagal**: NULL

### B. Menutup File - `fclose()`
```c
int fclose(FILE* stream);
```

**Parameter:**
* `stream`: Pointer file yang akan ditutup
