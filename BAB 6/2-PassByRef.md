[<< Pengenalan Pointer](1-PengenalanPointer.md)

# 6.2 - Pass By Reference

## Mengapa Kita Membutuhkan Pointer di Fungsi?

Mari kita ingat kembali masalah yang muncul pada contoh sebelumnya.  
Kita ingin **mengubah nilai variabel `health` dari dalam fungsi lain**, tetapi program menghasilkan error.

```c
#include <stdio.h>

void kurangi_health(int jumlah);

int main() {
    int health = 100;

    printf("Health awal: %d\n", health);

    /* Ingin mengurangi health sebesar 20 lewat fungsi */
    kurangi_health(20);

    printf("Health akhir: %d\n", health);

    return 0;
}

void kurangi_health(int jumlah) {
    /* Masalah: variabel `health` tidak dikenali di sini karena berada di scope main() */
    health = health - jumlah;  // ERROR: variabel 'health' tidak ditemukan
}
```

Masalahnya adalah fungsi `kurangi_health()` **tidak memiliki akses langsung** ke variabel `health` yang dideklarasikan di `main()`.  
Ini terjadi karena dalam C, **setiap fungsi memiliki ruang memori (scope) sendiri**.

---

## Konsep Dasar: Pass by Value vs Pass by Reference

Dalam C, terdapat **dua cara utama** untuk mengirim data ke fungsi:

### 1. Pass by Value
- **Nilai variabel dikopi ke parameter fungsi.**
- Perubahan di dalam fungsi **tidak memengaruhi variabel asli.**

Contoh:

```c
#include <stdio.h>

void ubah_nilai(int x) {
    x = 50;  // hanya mengubah salinan
}

int main() {
    int a = 100;
    ubah_nilai(a);
    printf("Nilai a: %d\n", a);  // Output: 100 (tidak berubah)
    return 0;
}
```

> **Penjelasan:**  
> Variabel `a` dikirim **sebagai nilai salinan**, jadi fungsi `ubah_nilai()` hanya bekerja dengan kopinya. Nilai asli `a` tetap 100.

---

### 2. Pass by Reference
- Yang dikirim **bukan nilai**, tetapi **alamat memori variabel**.
- Fungsi dapat **mengakses dan mengubah nilai asli** dari variabel tersebut menggunakan pointer.

Contoh:

```c
#include <stdio.h>

void ubah_nilai(int *x) {
    *x = 50;  // mengubah nilai asli melalui pointer
}

int main() {
    int a = 100;
    ubah_nilai(&a);  // kirim alamat variabel a
    printf("Nilai a: %d\n", a);  // Output: 50 (berubah)
    return 0;
}
```

> **Penjelasan:**  
> Dengan mengirimkan **alamat memori** (`&a`) dan menggunakan pointer (`*x`), fungsi dapat mengubah nilai asli `a`.

---

## Menerapkan Pass by Reference pada Kasus Health

Sekarang kita ubah contoh sebelumnya agar dapat bekerja menggunakan **pointer sebagai parameter fungsi**.

```c
#include <stdio.h>

/* Tambahkan parameter berupa pointer */
void kurangi_health(int *health_ptr, int jumlah);

int main() {
    int health = 100;

    printf("Health awal: %d\n", health);

    /* Kirim alamat dari variabel health menggunakan operator & */
    kurangi_health(&health, 20);

    printf("Health akhir: %d\n", health);

    return 0;
}

void kurangi_health(int *health_ptr, int jumlah) {
    /* Akses nilai variabel asli melalui pointer */
    *health_ptr = *health_ptr - jumlah;
}

/*
Output:

Health awal: 100
Health akhir: 80
*/
```

### Penjelasan Langkah demi Langkah

1. `int *health_ptr` → parameter berupa **pointer ke integer**, digunakan untuk menerima alamat variabel `health`.
2. `kurangi_health(&health, 20)` → simbol `&` berarti **ambil alamat dari variabel health**.
3. Di dalam fungsi, `*health_ptr` berarti **nilai yang disimpan di alamat tersebut**, sehingga ketika kita tulis:
   ```c
   *health_ptr = *health_ptr - jumlah;
   ```
   maka kita benar-benar **mengubah nilai asli `health` di fungsi main()**.

---

## Perbandingan Singkat

| Jenis Pemanggilan | Data yang Dikirim ke Fungsi | Efek terhadap Variabel Asli | Sintaks Pemanggilan |
|--------------------|-----------------------------|------------------------------|----------------------|
| **Pass by Value** | Salinan nilai               | Tidak berubah                | `fungsi(x);` |
| **Pass by Reference** | Alamat variabel (pointer) | Berubah                      | `fungsi(&x);` |

---

## Kesimpulan
- **Pass by Value** hanya mengirim **nilai salinan**, jadi perubahan di dalam fungsi **tidak memengaruhi variabel asli**.  
- **Pass by Reference** mengirimkan **alamat memori** variabel, sehingga fungsi **bisa mengubah nilai asli**.  
- Konsep ini sangat penting dalam penggunaan **pointer** karena memungkinkan fungsi berinteraksi langsung dengan variabel di luar scope-nya.

[Dynamic Memory Allocation >>](3-DMA.md)
