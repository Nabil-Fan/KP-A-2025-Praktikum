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

  ## 5. Mode Akses File
| Mode (Plaintext) | Mode (Binary) | Deskripsi                                                                                 | Jika lokasi file di komputer sudah ada                        | Jika lokasi file di komputer belum ada                                  |
| ---------------- | ------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------- |
| r                | rb            | File dibuka hanya untuk dibaca saja (read-only)                                           | return pointer obyek file tersebut                            | return `NULL`                                                           |
| w                | wb            | File dibuka hanya untuk ditulis saja (write-only)                                         | file akan ter-replace dan return pointer obyek file yang baru | file dibuat secara otomatis dan return pointer obyek file tersebut      |
| a                | ab            | File dibuka hanya untuk ditulis saja pada bagian akhir file (append-only)                 | return pointer obyek file tersebut                            | file dibuat secara otomatis dan return pointer obyek file tersebut      |
| r+               | rb+           | File dibuka untuk dibaca dan ditulis (read/write)                                         | return pointer obyek file tersebut                            | return `NULL`                                                           |
| w+               | wb+           | File dibuka untuk dibaca dan ditulis (read/write)                                         | file akan ter-replace dan return pointer obyek file yang baru | file akan dibuat secara otomatis dan return pointer obyek file tersebut |
| a+               | ab+           | File dibuka untuk dibaca (seluruh bagian file) dan ditulis (hanya pada bagian akhir file) | return pointer obyek file tersebut                            | file dibuat secara otomatis dan return pointer obyek file tersebut      |

> **Catatan**: Tambahkan `b` untuk binary mode (contoh: `"wb"`, `"rb"`)

---

## 6. Template Dasar Program File Processing
```c
#include <stdio.h>

int main() {
    // 1. Deklarasi pointer file
    FILE* file_ptr;
    
    // 2. Buka file dan simpan pointer-nya
    file_ptr = fopen("nama_file.txt", "mode_akses");
    
    // 3. CEK apakah file berhasil dibuka
    if (file_ptr == NULL) {
        printf("Error: Gagal membuka file!\n");
        return 1; // Keluar dengan error code
    }
    
    // 4. LAKUKAN OPERASI FILE di sini
    // (baca data, tulis data, dll)
    
    // 5. TUTUP file setelah selesai
    fclose(file_ptr);
    
    return 0; // Program sukses
}
```

---

## 7. Contoh Implementasi

### Contoh 1: Membuat dan Menulis File Text
```c
#include <stdio.h>

int main() {
    FILE* file = fopen("data.txt", "w");
    
    if (file == NULL) {
        printf("Gagal membuat file!\n");
        return 1;
    }
    
    // Menulis data ke file
    fprintf(file, "Nama: John Doe\n");
    fprintf(file, "Umur: 25 tahun\n");
    fprintf(file, "Kota: Jakarta\n");
    
    fclose(file);
    printf("File berhasil dibuat dan ditulis!\n");
    
    return 0;
}
```

### Contoh 2: Membaca File Text
```c
#include <stdio.h>

int main() {
    FILE* file = fopen("data.txt", "r");
    char buffer[100];
    
    if (file == NULL) {
        printf("File tidak ditemukan!\n");
        return 1;
    }
    
    // Membaca file baris per baris
    printf("Isi file data.txt:\n");
    while (fgets(buffer, sizeof(buffer), file) != NULL) {
        printf("%s", buffer);
    }
    
    fclose(file);
    return 0;
}
```

### Contoh 3: Menambah Data ke File (Append)
```c
#include <stdio.h>

int main() {
    FILE* file = fopen("data.txt", "a");
    
    if (file == NULL) {
        printf("Gagal membuka file!\n");
        return 1;
    }
    
    // Menambah data di akhir file
    fprintf(file, "Pekerjaan: Programmer\n");
    
    fclose(file);
    printf("Data berhasil ditambahkan!\n");
    
    return 0;
}
```

### Contoh 4: File Binary
```c
#include <stdio.h>

int main() {
    FILE* file = fopen("data.dat", "wb");
    int numbers[] = {10, 20, 30, 40, 50};
    
    if (file == NULL) {
        printf("Gagal membuat file binary!\n");
        return 1;
    }
    
    // Menulis data array ke file binary
    fwrite(numbers, sizeof(int), 5, file);
    
    fclose(file);
    printf("File binary berhasil dibuat!\n");
    
    return 0;
}
```
