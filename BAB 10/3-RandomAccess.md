# 9.3 - Random Access

**Random access** menyimpulkan bahwa file direpresentasikan sebagai kumpulan bytes yang panjangnya dapat diketahui dan bisa digambarkan sebagai kumpulan blok biner dengan tiap blok memiliki ukuran tertentu dan dapat disimpan dalam suatu variabel atau obyek bertipe struct. File berjenis **binary** umumnya dioperasikan secara random access. Dengan demikian, pembahasan seputar topik ini hanya seputar file berjenis binary saja.

## Operasi Penulisan (Write)

Dalam menulis ke file berjenis binary, dapat menggunakan fungsi `fwrite` yang terdeklarasi sebagai berikut:

```c
/* Tipe data size_t merupakan type alias dari satu bilangan bulat unsigned */
size_t fwrite(const void *src, size_t blocksize, size_t n, FILE *f);
```

Fungsi tersebut menulis blok yang diambil dari pointer `src` sebanyak `n` yang tiap blok berukuran `size` menuju file `f`. Akan me-return suatu angka yang menyatakan jumlah blok yang berhasil ditulis (jika sukses, nilainya seharusnya sama dengan parameter `n`)

Perhatikan contoh berikut:

```c
FILE *f;
char nama[100];
char prodi[50];
int angkatan;

printf("Masukkan nama : ");
scanf("%s", nama);

printf("Masukkan prodi : ");
scanf("%s", prodi);

printf("Masukkan angkatan : ");
scanf("%d", &angkatan);

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

/*
Output:

Masukkan nama : Michael_Raditya
Masukkan prodi : Informatika
Masukkan angkatan : 2020
Menyimpan ke biodata.bin ...
Sukses!
*/
```

File **biodata.bin** setelahnya berisi tiga blok yang masing-masing merepresentasikan nama, prodi, dan angkatan pada program tersebut dalam bentuk binary dan dapat dibaca lagi dengan cara memasukkan data tiap blok ke variabel yang memiliki tipe (atau ukuran) yang sama.

Cobalah untuk membuka file **biodata.bin** menggunakan notepad. Apa yang akan anda lihat? Data menjadi tidak dapat dibaca oleh manusia karena hanya bisa dibaca oleh program yang mengelola data tersebut.

Berapa ukuran file dari **biodata.bin**? Apakah 154 bytes? Kemudian cobalah untuk mengganti data yang anda inputkan pada console dan lihatlah ukuran dari **biodata.bin** lagi. Apakah masih sama? Hal itu dikarenakan tiap blok disimpan dengan ukuran yang tetap (fixed) yang mana blok nama disimpan dengan ukuran 100 bytes tanpa memedulikan seberapa panjang string yang dimasukkan, blok prodi disimpan dengan ukuran 50 bytes dan tanpa memedulikan seberapa panjang string yang dimasukkan juga, dan blok angkatan disimpan dengan ukuran 4 bytes sehingga total adalah 154 bytes. Dengan demikian, perbedaan nilai pada data yang disimpan dalam file yang memiliki layout yang sama selalu menghasilkan ukuran file yang sama juga. Hal ini berbeda dengan sequential access (plaintext) di mana ukuran file bergantung pada representasi suatu variable dalam bentuk string (misal jika jumlah digit suatu angka adalah sebanyak 6 digit, maka angka tersebut membutuhkan ruang 6 bytes pada file plaintext, dan jika panjang suatu string adalah sebanyak 16 karakter, maka string tersebut membutuhkan ruang 16 bytes pada file plaintext) dan dengan demikian file plaintext tidak menjamin kesamaan ukuran file untuk berbagai kemungkinan data.

## Operasi Pembacaan (Read)

Dalam membaca file berjenis binary, dapat menggunakan fungsi `fread` yang terdeklarasi sebagai berikut:

```c
/* Tipe data size_t merupakan type alias dari satu bilangan bulat unsigned */
size_t fread(void *dest, size_t blocksize, size_t n, FILE *f);
```

Fungsi tersebut membaca blok sebanyak `n` yang tiap blok berukuran `size` dalam file `f` dan kemudian hasilnya disimpan pada pointer `dest`. Akan me-return suatu angka yang menyatakan jumlah blok yang berhasil dibaca (jika sukses, nilainya seharusnya sama dengan parameter `n`)

Perhatikan contoh berikut:

Asumsikan **biodata.bin** berisi tiga blok yang masing-masing merepresentasikan nama, prodi, dan angkatan dalam bentuk binary.

```c
FILE *f;

printf("Memuat dari biodata.bin ...\n");
f = fopen("biodata.bin", "rb");
if (f != NULL) {
    char nama[100];
    char prodi[50];
    int angkatan;

    fread(nama, 100 * sizeof(char), 1, f);
    fread(prodi, 50 * sizeof(char), 1, f);
    fread(&angkatan, sizeof(int), 1, f);
    fclose(f);

    printf("Nama : %s\n", nama);
    printf("Prodi : %s\n", prodi);
    printf("Angkatan : %d\n", angkatan);
} else {
    printf("Tidak dapat memuat file biodata.bin\n");
}

/*
Output:

Memuat dari biodata.bin ...
Nama : Michael_Raditya
Prodi : Informatika
Angkatan : 2020
*/
```

## Baca-Tulis Array (Fixed Size) Dalam File

Suatu array (fixed size) dapat dibaca/ditulis dengan mudah dalam file jika menggunakan mode random-access (binary) dengan block size adalah ukuran tiap-tiap elemen array. Perhatikan contoh di bawah

Penulisan (write):

```c
FILE *f;
int price_list[5] = {750, 1500, 2500, 3100, 4750};

/* ... */

fwrite(price_list, sizeof(int), 5, f);
```

Pembacaan (read):

```c
FILE *f;
int price_list[5];

/* ... */

fread(price_list, sizeof(int), 5, f);
```

## Baca-Tulis Array (Dynamic Size) Dalam File

Suatu array (dynamic size) juga dapat dibaca/ditulis dengan mudah dalam file jika menggunakan mode random-access (binary) dengan block size adalah ukuran tiap-tiap elemen array. Perhatikan contoh di bawah

Penulisan (write):

```c
#include <stdio.h>

int main() {
    FILE *f;
    int price_list[100];
    int item_count, i;

    f = fopen("harga.bin", "wb");
    if (f == NULL) {
        printf("Gagal membuka file!\n");
        return 0;
    }

    printf("Masukkan jumlah barang: ");
    scanf("%d", &item_count);

    for (i = 0; i < item_count; i++) {
        printf("Harga barang ke-%d: ", i+1);
        scanf("%d", &price_list[i]);
    }

    fwrite(&item_count, sizeof(int), 1, f);
    fwrite(price_list, sizeof(int), item_count, f);

    fclose(f);
    return 0;
}

```

Pembacaan (read):

```c
#include <stdio.h>

int main() {
    FILE *f;
    int price_list[100];
    int item_count, i;

    f = fopen("harga.bin", "rb");
    if (f == NULL) {
        printf("File tidak ditemukan!\n");
        return 0;
    }

    fread(&item_count, sizeof(int), 1, f);
    fread(price_list, sizeof(int), item_count, f);

    fclose(f);

    printf("Jumlah barang: %d\n", item_count);
    for (i = 0; i < item_count; i++) {
        printf("Harga barang ke-%d: %d\n", i+1, price_list[i]);
    }

    return 0;
}

```

## Baca-Tulis Obyek Bertipe Struct Dalam File

Suatu obyek bertipe struct juga dapat dibaca/tulis dengan mudah dalam file jika menggunakan mode random-access (binary) dengan block size adalah ukuran struct obyek tersebut. Perhatikan contoh di bawah

Asumsikan `struct Weapon` terdefinisi sebagai:

```c
struct Weapon {
    char name[50];
    int price;
    int damage;
    int rounds;
};
```

Penulisan (write):

```c
#include <stdio.h>
#include <string.h>

struct Weapon {
    char name[50];
    int price;
    int damage;
    int rounds;
};

int main() {
    FILE *f;
    struct Weapon deagle;

    // isi data
    strcpy(deagle.name, "Desert Eagle");
    deagle.price = 750;
    deagle.damage = 35;
    deagle.rounds = 7;

    // buka file
    f = fopen("weapon.bin", "wb");
    if (f == NULL) {
        printf("Gagal membuka file!\n");
        return 1;
    }

    // tulis data struct ke file
    fwrite(&deagle, sizeof(struct Weapon), 1, f);

    fclose(f);

    printf("Data weapon berhasil disimpan ke weapon.bin\n");
    return 0;
}

```

Pembacaan (read):

```c
#include <stdio.h>

struct Weapon {
    char name[50];
    int price;
    int damage;
    int rounds;
};

void print_weapon(struct Weapon *w) {
    printf("Nama Weapon  : %s\n", w->name);
    printf("Harga        : %d\n", w->price);
    printf("Damage       : %d\n", w->damage);
    printf("Rounds/Mag   : %d\n", w->rounds);
}

int main() {
    FILE *f;
    struct Weapon deagle;

    // buka file
    f = fopen("weapon.bin", "rb");
    if (f == NULL) {
        printf("File weapon.bin tidak ditemukan!\n");
        return 1;
    }

    // baca struct dari file
    fread(&deagle, sizeof(struct Weapon), 1, f);

    fclose(f);

    // tampilkan data
    print_weapon(&deagle);

    return 0;
}

```

## File Seeking

Kursor yang merujuk ke lokasi target operasi pembacaan/penulisan berikutnya dalam file random-access (binary) dapat dipindah-pindah menggunakan fungsi **fseek()** pada **stdio.h** dengan deklarasi sebagai berikut:

```c
int fseek(FILE *f, long int offset, int origin);
```

Fungsi tersebut mengatur kursor pada file `f` supaya berjarak sebesar `offset` bytes relatif terhadap `origin`, dengan `origin` adalah salah satu dari:

- `SEEK_SET` : lokasi kursor pada awal file
- `SEEK_CUR` : lokasi kursor saat ini
- `SEEK_END` : lokasi kursor pada akhir file

Sebagai contoh:

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
    f = fopen("data.bin", "wb+");
    if (f == NULL) {
        printf("File gagal dibuka!\n");
        return 1;
    }
    fwrite(price_list, sizeof(int), 5, f);
    fseek(f, 1 * sizeof(int), SEEK_SET);
    fwrite(&new_price, sizeof(int), 1, f);
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
