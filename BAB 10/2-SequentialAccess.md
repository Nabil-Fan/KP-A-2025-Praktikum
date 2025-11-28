# 10.2 - Sequential Access

## Pengertian Sequential Access

Sequential access adalah cara mengakses file dimana data dibaca atau ditulis secara berurutan dari awal hingga akhir. File dianggap sebagai kumpulan karakter/bytes yang panjangnya tidak diketahui di awal.

File plaintext (teks biasa) biasanya dioperasikan dengan sequential access. Materi ini akan fokus pada operasi file plaintext.

## Operasi Membaca File

Untuk membaca file plaintext, kita bisa menggunakan beberapa fungsi dari header `stdio.h`:

### Menggunakan `fscanf()`

Fungsi `fscanf()` bekerja mirip dengan `scanf()`, tetapi membaca dari file.

**Contoh:** File `biodata.txt` berisi:
```text
Michael_Raditya
Informatika
2020
```

**Kode Program:**
```c
#include <stdio.h>

int main() {
    FILE *f;
    
    printf("Memuat dari biodata.txt ...\n");
    f = fopen("biodata.txt", "r");
    
    if (f != NULL) {
        char nama[100];
        char prodi[50];
        int angkatan;

        // Membaca data dari file
        fscanf(f, "%s", nama);
        fscanf(f, "%s", prodi);
        fscanf(f, "%d", &angkatan);

        // Menampilkan data
        printf("Nama : %s\n", nama);
        printf("Prodi : %s\n", prodi);
        printf("Angkatan : %d\n", angkatan);

        fclose(f);
    } else {
        printf("Tidak dapat membaca file biodata.txt\n");
    }
    
    return 0;
}
```

**Output:**
```text
Memuat dari biodata.txt ...
Nama : Michael_Raditya
Prodi : Informatika
Angkatan : 2020
```

## Operasi Menulis File

Untuk menulis file plaintext, kita bisa menggunakan beberapa fungsi dari `stdio.h`:

### Menggunakan `fprintf()`

Fungsi `fprintf()` bekerja mirip dengan `printf()`, tetapi menulis ke file.

**Kode Program:**
```c
#include <stdio.h>

int main() {
    FILE *f;
    char nama[100];
    char prodi[50];
    int angkatan;

    // Input dari user
    printf("Masukkan nama : ");
    scanf("%s", nama);
    
    printf("Masukkan prodi : ");
    scanf("%s", prodi);
    
    printf("Masukkan angkatan : ");
    scanf("%d", &angkatan);

    // Menulis ke file
    printf("Menyimpan ke biodata.txt ...\n");
    f = fopen("biodata.txt", "w");
    
    if (f != NULL) {
        fprintf(f, "%s\n", nama);
        fprintf(f, "%s\n", prodi);
        fprintf(f, "%d\n", angkatan);
        
        fclose(f);
        printf("Sukses!\n");
    } else {
        printf("Tidak dapat menulis ke file biodata.txt\n");
    }
    
    return 0;
}
```

**Contoh Output Program:**
```text
Masukkan nama : Adriel_Alfeus
Masukkan prodi : Informatika
Masukkan angkatan : 2020
Menyimpan ke biodata.txt ...
Sukses!
```

**Isi file `biodata.txt` setelah dijalankan:**
```text
Adriel_Alfeus
Informatika
2020
```

## Fungsi Rewind

Fungsi `rewind()` digunakan untuk mengembalikan kursor file ke posisi awal.

**Deklarasi:**
```c
void rewind(FILE *f);
```

**Contoh Penggunaan:**
```c
FILE *f;

fprintf(f, "Lorem ipsum dolor\n");
rewind(f);  // Kursor kembali ke awal file
fprintf(f, "sit amet\n");
```

---

**Catatan:** Pastikan selalu menutup file dengan `fclose()` setelah selesai menggunakannya untuk menghindari kebocoran memori dan memastikan data tersimpan dengan benar.
