# Pengenalan Pointer dalam Bahasa C

## Apa itu Pointer
**Pointer** adalah variabel khusus yang menyimpan **alamat memori** dari variabel lain.  

## Mengapa Pointer Penting
Pointer memungkinkan kita **mengubah nilai variabel** dari **luar scope** tempat variabel tersebut dibuat.

Contoh tanpa pointer:

```c
#include <stdio.h>

void kurangi_health(int jumlah);

int main() {
    int health = 100;
    printf("Health awal: %d\n", health);
    
    kurangi_health(20); // Ingin mengurangi health
    
    printf("Health akhir: %d\n", health);
    return 0;
}

void kurangi_health(int jumlah) {
    health = health - jumlah; 
}
```

## Cara Membuat Pointer

### Deklarasi Pointer
```c
// Format: tipe_data *nama_variabel;
int *health_ptr;        // Pointer ke integer
float *speed_ptr;       // Pointer ke float
char *nama_ptr;         // Pointer ke karakter
```

### Penulisan Alternatif
```c
int* health_ptr;  // Sama saja dengan int *health_ptr
```

### Deklarasi Banyak Pointer
```c
// BENAR: kedua variabel adalah pointer
int *health_ptr, *ammo_ptr;

// SALAH: hanya health_ptr yang pointer, ammo_ptr biasa
int* health_ptr, ammo_ptr;
```

## Menggunakan Pointer

### Inisialisasi Pointer
```c
int health = 100;
int *health_ptr;           // Deklarasi pointer
health_ptr = &health;      // Pointer menunjuk ke health
// &health artinya "alamat dari health"
```

## Pointer NULL
Pointer NULL adalah pointer yang tidak menunjuk ke alamat mana pun.

```c
#include <stddef.h>  // Untuk NULL

int health = 100;
int *health_ptr;
int pilihan;

printf("Masukkan 1 untuk bertarung, 0 untuk keluar: ");
scanf("%d", &pilihan);

if (pilihan == 1) {
    health_ptr = &health;  // Pointer menunjuk ke health
} else {
    health_ptr = NULL;     // Pointer tidak menunjuk ke mana-mana
}

if (health_ptr != NULL) {
    printf("Anda memilih bertarung\n");
    *health_ptr = 85;      // Ubah health menjadi 85
} else {
    printf("Anda memilih keluar\n");
}

printf("Health akhir: %d\n", health);
```

## Pointer Read-Only (Constant Pointer)
Pointer yang hanya bisa membaca, tidak bisa mengubah nilai.

```c
int health = 100;
const int *health_ptr = &health;  // Pointer read-only

// BISA: Membaca nilai
printf("Health: %d\n", *health_ptr);

// ERROR: Tidak bisa mengubah nilai
*health_ptr = 50;  // Akan error karena read-only
```

## Ringkasan Simbol Penting

| Simbol | Arti | Contoh | Penjelasan |
|--------|------|---------|------------|
| `*` (deklarasi) | Membuat pointer | `int *ptr;` | ptr adalah pointer ke integer |
| `&` | Mengambil alamat | `ptr = &health;` | ptr menyimpan alamat health |
| `*` (penggunaan) | Mengakses nilai | `printf("%d", *ptr);` | Menampilkan nilai yang ditunjuk ptr |

## Kesimpulan
Pointer sangat berguna untuk memodifikasi variabel dari fungsi lain (**Pass by Reference**), serta menjadi dasar dalam pengelolaan memori dinamis, array, dan struktur data kompleks seperti **linked list** dan **tree**.

[Pass By Reference >>](2-PassByRef.md)
