# 10.3 - Random Access

## Apa itu Random Access?

Random access adalah cara mengakses file dimana kita bisa membaca atau menulis data di posisi mana saja dalam file tanpa harus berurutan. File dianggap sebagai kumpulan bytes yang panjangnya diketahui dan dapat dibagi menjadi blok-blok data.

File binary (biner) biasanya dioperasikan dengan random access. File binary menyimpan data dalam format yang hanya bisa dibaca oleh komputer.

## Keuntungan Random Access

- Ukuran file tetap sama untuk layout data yang sama
- Akses data lebih cepat karena bisa langsung ke posisi tertentu
- Cocok untuk menyimpan data terstruktur seperti array dan struct

## Operasi Menulis File Binary

### Menggunakan `fwrite()`
```c
size_t fwrite(const void *src, size_t blocksize, size_t n, FILE *f);
```

**Parameter:**
- `src`: pointer ke data yang akan ditulis
- `blocksize`: ukuran setiap blok data
- `n`: jumlah blok yang akan ditulis
- `f`: pointer ke file

**Contoh Program:**
```c
#include <stdio.h>

int main() {
    FILE *f;
    char nama[100];
    char prodi[50];
    int angkatan;

    // Input data dari user
    printf("Masukkan nama : ");
    scanf("%s", nama);
    
    printf("Masukkan prodi : ");
    scanf("%s", prodi);
    
    printf("Masukkan angkatan : ");
    scanf("%d", &angkatan);

    // Menulis ke file binary
    printf("Menyimpan ke biodata.bin ...\n");
    f = fopen("biodata.bin", "wb");
    
    if (f != NULL) {
        fwrite(nama, 100 * sizeof(char), 1, f);
        fwrite(prodi, 50 * sizeof(char), 1, f);
        fwrite(&angkatan, sizeof(int), 1, f);
        
        fclose(f);
        printf("Sukses!\n");
    } else {
        printf("Tidak dapat menulis ke file biodata.bin\n");
    }
    
    return 0;
}
```

**Output:**
```text
Masukkan nama : Michael_Raditya
Masukkan prodi : Informatika
Masukkan angkatan : 2020
Menyimpan ke biodata.bin ...
Sukses!
```

**Karakteristik File Binary:**
- **Tidak bisa dibaca manusia** - Jika dibuka di text editor, tampilannya acak
- **Ukuran tetap** - File selalu berukuran 154 bytes (100 + 50 + 4 bytes)
- **Data padding** - String yang pendek tetap memakai ruang penuh

## Operasi Membaca File Binary

### Menggunakan `fread()`
```c
size_t fread(void *dest, size_t blocksize, size_t n, FILE *f);
```

**Parameter:**
- `dest`: pointer tempat menyimpan data yang dibaca
- `blocksize`: ukuran setiap blok data
- `n`: jumlah blok yang akan dibaca
- `f`: pointer ke file

**Contoh Program:**
```c
#include <stdio.h>

int main() {
    FILE *f;
    
    printf("Memuat dari biodata.bin ...\n");
    f = fopen("biodata.bin", "rb");
    
    if (f != NULL) {
        char nama[100];
        char prodi[50];
        int angkatan;

        // Membaca data dari file binary
        fread(nama, 100 * sizeof(char), 1, f);
        fread(prodi, 50 * sizeof(char), 1, f);
        fread(&angkatan, sizeof(int), 1, f);
        
        fclose(f);

        // Menampilkan data
        printf("Nama : %s\n", nama);
        printf("Prodi : %s\n", prodi);
        printf("Angkatan : %d\n", angkatan);
    } else {
        printf("Tidak dapat memuat file biodata.bin\n");
    }
    
    return 0;
}
```

**Output:**
```text
Memuat dari biodata.bin ...
Nama : Michael_Raditya
Prodi : Informatika
Angkatan : 2020
```

## File Seeking dengan `fseek()`

Fungsi `fseek()` digunakan untuk memindahkan kursor file ke posisi tertentu.
```c
int fseek(FILE *f, long int offset, int origin);
```

**Parameter `origin`:**
- `SEEK_SET` - Dari awal file
- `SEEK_CUR` - Dari posisi sekarang
- `SEEK_END` - Dari akhir file

**Contoh Penggunaan:**
```c
#include <stdio.h>

int main() {
    FILE *f;
    int price_list[5] = {750, 1500, 2500, 3100, 4750};
    int new_price = 2000;
    int buffer[5];
    
    printf("Data awal:\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", price_list[i]);
    }
    printf("\n\n");
    
    // Buka file untuk baca+tulis
    f = fopen("data.bin", "wb+");
    if (f == NULL) {
        printf("File gagal dibuka!\n");
        return 1;
    }
    
    // Tulis array ke file
    fwrite(price_list, sizeof(int), 5, f);
    
    // Pindah ke elemen ke-2 (indeks 1) dan ubah nilainya
    fseek(f, 1 * sizeof(int), SEEK_SET);
    fwrite(&new_price, sizeof(int), 1, f);
    
    // Kembali ke awal dan baca seluruh array
    fseek(f, 0, SEEK_SET);
    fread(buffer, sizeof(int), 5, f);
    
    printf("Data setelah diubah:\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", buffer[i]);
    }
    printf("\n");

    fclose(f);
    return 0;
}
```

**Output:**
```text
Data awal:
750 1500 2500 3100 4750 

Data setelah diubah:
750 2000 2500 3100 4750
```

---

**Catatan:** Random access sangat berguna untuk aplikasi yang membutuhkan akses data cepat dan efisien, seperti database sederhana atau sistem manajemen file.
