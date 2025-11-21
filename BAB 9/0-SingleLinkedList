[<< Silabus](../silabus.md)

# 8.1 - Linked List

## Introduction

Pada dasarnya, linked list berfungsi sebagai array yang dapat tumbuh dan menyusut sesuai kebutuhan, dari titik mana pun dalam array. Linked list memiliki beberapa keunggulan dibandingkan array:
1. **Item dapat ditambahkan atau dihapus dari tengah daftar**
2. **Tidak perlu mendefinisikan ukuran awal**

Namun, linked list juga memiliki beberapa kelemahan, seperti:
1. **Tidak ada akses "acak"**, tidak mungkin mencapai item ke-n dalam array tanpa melakukan iterasi terlebih dahulu semua item hingga item itu. Artinya kita harus mulai dari awal daftar dan menghitung berapa kali kita maju dalam list/daftar sampai kita sampai ke item yang diinginkan.
2. **Alokasi memori dinamis dan penunjuk diperlukan**, yang mempersulit kode dan meningkatkan risiko kebocoran memori dan kesalahan segmen.
3. **Linked list memiliki overhead over array yang jauh lebih besar**, karena item daftar tertaut dialokasikan secara dinamis (yang kurang efisien dalam penggunaan memori) dan setiap item dalam daftar juga harus menyimpan pointer tambahan.

## Apa itu Linked List?

Linked list adalah **sekumpulan node yang dialokasikan secara dinamis**, disusun sedemikian rupa sehingga setiap node berisi satu value dan satu pointer. **Pointer akan selalu menunjuk ke anggota list berikutnya**. Jika **pointer adalah NULL**, maka itu adalah **node terakhir dalam list**. 

Linked list disimpan menggunakan variabel pointer lokal yang menunjuk ke item pertama list. Jika **pointer adalah NULL**, maka **list tersebut dianggap kosong**.

Mari kita deklarasikan node linked listnya terlebih dahulu:

```c

typedef struct node {
    int val;
    struct node *next;
} node_t;

```

Sekarang kita bisa menggunakan node. Mari kita buat variabel lokal yang menunjuk ke item pertama dari daftar (disebut `head`).

```c

node_t *head = NULL;
head = (node_t *) malloc(sizeof(node_t));
if (head == NULL) {
    return 1;
}

head->val = 1;
head->next = NULL;

```

Nah, kita baru saja membuat variabel pertama dalam daftar. Kita harus menetapkan nilainya, dan item berikutnya menjadi kosong, jika kita mau untuk menyelesaikan mengisi daftar. Perhatikan bahwa kita harus selalu memeriksa apakah malloc mengembalikan nilai NULL atau tidak.

Untuk menambahkan variabel ke akhir daftar, kita hanya dapat melanjutkan maju ke pointer berikutnya:

```c
node_t *head = NULL;
head = (node_t *) malloc(sizeof(node_t));
head->val = 1;
head->next = (node_t *) malloc(sizeof(node_t));
head->next->val = 2;
head->next->next = NULL;
```

Hal ini bisa terus berlanjut, namun yang harus kita lakukan sebenarnya adalah maju ke item terakhir dalam daftar, hingga next variabel akan menjadi NULL.


## Iterasi List

Sekarang, mari kita coba buat fungsi yang mencetak semua item dalam list. Untuk melakukan ini, kita perlu menggunakan `current ` pointer yang akan melacak node yang sedang kita cetak. Setelah mencetak node, kita atur `current` pointer-nya ke node berikutnya, dan cetak lagi sampai mencapai akhir list (node berikutnya adalah NULL).

```c

void print_list(node_t *head) {
    node_t *current = head;

    while (current != NULL) {
        printf("%d\n", current->val);
        current = current->next;
    }
}

```

## Menambahkan Item ke Akhir List

Untuk melakukan iterasi semua anggot linked list, kita gunakan pointer yang dipanggil oleh `current`. Kita atur untuk memulai dari head (kepala) dan kemudian di tiap step, kita majukan pointer ke item berikutnya dalam list, sampai kita mencapai item terakhir.

```c

void push(node_t *head, int val) {
    node_t *current = head;
    while (current->next != NULL) {
        current = current->next;
    }

    /* sekarang kita bisa tambahkan variabel baru */
    current->next = (node_t *) malloc(sizeof(node_t));
    current->next->val = val;
    current->next->next = NULL;
}

```

## Menambahkan Item ke Awal List

Untuk menambahkan item ke awal list, kita perlu melakukan hal-hal berikut:
1. **Buat item baru dan tetapkan nilainya**
2. **Tautkan item baru untuk menunjuk ke head (kepala) daftar**
3. **Tetapkan head (kepala) daftar untuk menjadi item baru**

Hal ini secara efektif akan membuat head (kepala) baru ke list dengan nilai baru, dan menjaga sisa list yang terkait dengannya. 

Karena kita menggunakan fungsi untuk melakukan operasi ini, kita dapat memodifikasi variabel head (kepala). Untuk melakukan hal itu, ktia bisa beri pointer ke variabel pointer (**hal ini disebut double pointer**) sehingga kita dapat memodifikasi pointer itu sendiri. 

```c

void push(node_t ** head, int val) {
    node_t *new_node;
    new_node = (node_t *) malloc (sizeof(node_t));

    new_node->val = val;
    new_node->next = *head;
    *head = new_node;
}

```

## Menghapus Item Pertama

Untuk memunculkan suatu variabel, kita perlu melakukan tindakan ini:
1. **Ambil item berikutnya yang ditunjuk head dan simpan**
2. **Free item head**
3. **Atur head menjadi item berikutnya yang kita simpan di samping**

Berikut kodenya:

```c

int pop(node_t **head) {
    int retval = -1;
    node_t *next_node = NULL;

    if (*head == NULL) {
        return -1;
    }

    next_node = (*head)->next;
    retval = (*head)->val;
    free(*head);
    *head = next_node;

    return retval;
}

```

## Menghapus Item Terakhir dari Daftar

Menghapus item terakhir dari daftar sangat mirip dengan menambahkannya ke akhir daftar, **tetapi dengan satu pengecualian besar**, karena kita harus mengubah satu item sebelum item terakhir. Kita sebenarnya harus melihat dua item ke depan dan melihat apakah item berikutnya adalah item terakhir yang ada dalam list.

```c

int remove_last(node_t **head) {
    int retval = 0;

    if (*head == NULL) {
        return -1;
    }

    //kalau cuman ada satu node
    if ((*head)->next == NULL) {
        retval = (*head)->val;
        free(*head);
        *head = NULL;
        return retval;
    }

    //akses ke elemen kedua sebelum terakhir
    node_t *current = *head;
    while (current->next->next != NULL) {
        current = current->next;
    }

    retval = current->next->val;
    free(current->next);
    current->next = NULL;

    return retval;
}


```

## Menghapus Item Tertentu

Untuk menghapus item tertentu dari list, baik berdasarkan indeksnya dari awal daftar maupun berdasarkan nilanya, kita perlu memeriksa semua item, lalu melihat ke depan untuk mengetahui apakah kita telah mencapai node sebelumnya dari item yang akan kita hapus. Ini karena kita perlu mengubah lokasi ke tempat titik node sebelumnya juga.

Berikut adalah algoritmanya:
1. Iterasi ke node sebelum node yang ingin kita hapus
2. Simpan node yang ingin kita hapus dalam pointer sementara
3. Atur pointer berikutnya dari node sebelumnya untuk menunjuk ke node setelah node yang ingin kita hapus
4. Hapus node menggunakan pointer sementara

```c

int remove_by_index(node_t **head, int n) {
    int i = 0;
    int retval = -1;
    node_t *current = *head;
    node_t *temp_node = NULL;

    if (n == 0) {
        return pop(head);
    }

    for (i = 0; i < n-1; i++) {
        if (current->next == NULL) {
            return -1;
        }
        current = current->next;
    }

    if (current->next == NULL) {
        return -1;
    }

    temp_node = current->next;
    retval = temp_node->val;
    current->next = temp_node->next;
    free(temp_node);

    return retval;

}

```

[Double Linked List >>](2-DoubleLinkedList.md)
