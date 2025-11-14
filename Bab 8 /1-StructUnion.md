# Struct dan Union dalam Bahasa C

## Pendahuluan

### Masalah yang Diatasi oleh Struct

Bayangkan Anda ingin menyimpan data tentang senjata dalam game:

```c
char weapon_names[5][50] = {"Desert Eagle", "H&K MP5", "AK-47", "M4A1", "AWP Magnum"};
int weapon_prices[5] = {750, 1500, 2500, 3100, 4750};
int weapon_damages[5] = {35, 15, 25, 20, 100};
int weapon_rounds[5] = {7, 30, 30, 30, 10};
```

**Masalah:**

- Data terpisah-pisah dalam array yang berbeda
- Sulit untuk mengelompokkan data yang berkaitan
- Jika ingin menambah properti baru, harus membuat array baru
- Tidak efisien saat passing data ke fungsi

## Apa itu Struct?

Struct adalah tipe data yang memungkinkan kita menggabungkan beberapa variabel dengan tipe data yang berbeda dalam satu unit.

### Analogi Sederhana

Jika array seperti laci yang hanya bisa menyimpan benda sejenis, maka struct seperti tas yang bisa menyimpan berbagai macam benda berbeda.

## Deklarasi dan Definisi Struct

### Cara 1: Deklarasi dan Definisi Terpisah

```c
// Deklarasi struct
struct Weapon;

// Definisi struct
struct Weapon {
    char name[50];
    int price;
    int damage;
    int rounds;
};
```

### Cara 2: Deklarasi dan Definisi Sekaligus

```c
struct Weapon {
    char name[50];
    int price;
    int damage;
    int rounds;
};
```

### Cara 3: Dengan typedef (Recommended)

```c
typedef struct {
    char name[50];
    int price;
    int damage;
    int rounds;
} Weapon;
```

## Menggunakan Struct

### 1. Deklarasi Variabel Struct

```c
// Dengan struct biasa
struct Weapon deagle;
struct Weapon ak47, m4a1;

// Dengan typedef
Weapon deagle;
Weapon ak47, m4a1;
```

### 2. Inisialisasi Struct

```c
// Cara 1: Inisialisasi langsung
Weapon deagle = {"Desert Eagle", 750, 35, 7};

// Cara 2: Inisialisasi per member
Weapon deagle;
strcpy(deagle.name, "Desert Eagle");
deagle.price = 750;
deagle.damage = 35;
deagle.rounds = 7;
```

### 3. Array of Struct

```c
Weapon weapons[5] = {
    {"Desert Eagle", 750, 35, 7},
    {"H&K MP5", 1500, 15, 30},
    {"AK-47", 2500, 25, 30},
    {"M4A1", 3100, 20, 30},
    {"AWP Magnum", 4750, 100, 10}
};
```

## Mengakses Member Struct

### Menggunakan Operator Titik (.)

```c
Weapon deagle = {"Desert Eagle", 750, 35, 7};

// Membaca data
printf("Nama: %s\n", deagle.name);
printf("Harga: $%d\n", deagle.price);

// Mengubah data
deagle.damage = 45;
deagle.price = 800;
```

## Contoh Program Lengkap

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char name[50];
    int price;
    int damage;
    int rounds;
} Weapon;

void printWeapon(Weapon w) {
    printf("=== STATISTIK SENJATA ===\n");
    printf("Nama    : %s\n", w.name);
    printf("Harga   : $%d\n", w.price);
    printf("Damage  : %d%%\n", w.damage);
    printf("Peluru  : %d\n", w.rounds);
    printf("========================\n\n");
}

int main() {
    Weapon deagle = {"Desert Eagle", 750, 35, 7};
    Weapon ak47 = {"AK-47", 2500, 25, 30};
    
    printWeapon(deagle);
    printWeapon(ak47);
    
    // Upgrade senjata
    deagle.damage += 10;
    deagle.price += 100;
    
    printf("Setelah upgrade:\n");
    printWeapon(deagle);
    
    return 0;
}
```

## Pointer dan Struct

### Mengapa Menggunakan Pointer?

1. Menghemat memori (tidak menyalin seluruh struct)
2. Bisa memodifikasi data asli
3. Lebih efisien untuk struct yang besar

### Contoh dengan Pointer

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char name[50];
    int price;
    int damage;
    int rounds;
} Weapon;

// Fungsi dengan pointer (lebih efisien)
void printWeapon(const Weapon *w) {
    printf("Nama: %s\n", w->name);    // -> untuk pointer
    printf("Harga: $%d\n", w->price);
    printf("Damage: %d%%\n", w->damage);
    printf("Peluru: %d\n", w->rounds);
}

// Fungsi untuk memodifikasi data
void upgradeWeapon(Weapon *w, int damageBoost, int priceIncrease) {
    w->damage += damageBoost;
    w->price += priceIncrease;
}

int main() {
    Weapon deagle = {"Desert Eagle", 750, 35, 7};
    
    printf("Sebelum upgrade:\n");
    printWeapon(&deagle);  // Pass alamat
    
    upgradeWeapon(&deagle, 15, 200);
    
    printf("\nSetelah upgrade:\n");
    printWeapon(&deagle);
    
    return 0;
}
```

## Dynamic Memory Allocation dengan Struct

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[50];
    int price;
    int damage;
    int rounds;
} Weapon;

int main() {
    int jumlahSenjata = 3;
    Weapon *senjata = malloc(jumlahSenjata * sizeof(Weapon));
    
    if (senjata == NULL) {
        printf("Error: Memori tidak cukup!\n");
        return 1;
    }
    
    // Isi data
    strcpy(senjata[0].name, "Desert Eagle");
    senjata[0].price = 750;
    senjata[0].damage = 35;
    senjata[0].rounds = 7;
    
    strcpy(senjata[1].name, "AK-47");
    senjata[1].price = 2500;
    senjata[1].damage = 25;
    senjata[1].rounds = 30;
    
    strcpy(senjata[2].name, "AWP Magnum");
    senjata[2].price = 4750;
    senjata[2].damage = 100;
    senjata[2].rounds = 10;
    
    // Tampilkan data
    for(int i = 0; i < jumlahSenjata; i++) {
        printf("Senjata %d: %s ($%d)\n", i+1, senjata[i].name, senjata[i].price);
    }
    
    free(senjata);  // Jangan lupa free memory!
    return 0;
}
```

## Union

### Apa itu Union?

Union mirip dengan struct, tetapi semua member berbagi alamat memori yang sama. Ukuran union ditentukan oleh member terbesar.

### Perbedaan Struct vs Union

```c
// STRUCT - setiap member punya alamat sendiri
struct DataStruct {
    int angka;      // 4 bytes
    float desimal;  // 4 bytes  
    char huruf;     // 1 byte
}; // Total: 9+ bytes (dengan padding)

// UNION - semua member berbagi alamat yang sama
union DataUnion {
    int angka;      // 4 bytes
    float desimal;  // 4 bytes
    char huruf;     // 1 byte
}; // Total: 4 bytes (ukuran terbesar)
```

### Contoh Penggunaan Union

```c
#include <stdio.h>

typedef union {
    int angka;
    float desimal;
    char huruf;
} Data;

int main() {
    Data data;
    
    data.angka = 65;
    printf("Sebagai integer: %d\n", data.angka);
    printf("Sebagai float: %f\n", data.desimal); // Hasil aneh!
    printf("Sebagai char: %c\n", data.huruf);    // Karakter 'A'
    
    printf("\n");
    
    data.desimal = 3.14;
    printf("Sebagai integer: %d\n", data.angka); // Hasil aneh!
    printf("Sebagai float: %f\n", data.desimal);
    printf("Sebagai char: %c\n", data.huruf);    // Hasil aneh!
    
    return 0;
}
```

## Kesimpulan

- Struct untuk mengelompokkan data yang terkait
- Union untuk menghemat memori ketika hanya satu member yang digunakan
- Selalu gunakan pointer ketika passing struct ke fungsi
- Gunakan typedef untuk membuat kode lebih rapi
- const untuk parameter yang tidak diubah
