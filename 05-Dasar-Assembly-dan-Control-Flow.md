# 05. Bahasa Assembly x86/x64, Control Flow, dan Calling Conventions

Pada minggu sebelumnya saya mempelajari berbagai tools yang digunakan dalam reverse engineering, seperti **Detect It Easy (DIE)**, **Ghidra**, dan **x64dbg**. Setelah memahami cara menggunakan tools tersebut, kini saatnya mulai membaca bahasa yang dipahami langsung oleh CPU, yaitu **Assembly**.

Awalnya Assembly terlihat sangat rumit karena hanya berisi instruksi-instruksi pendek seperti `mov`, `cmp`, `jmp`, atau `call`. Namun setelah dipelajari, saya mulai menyadari bahwa instruksi-instruksi tersebut sebenarnya hanyalah "potongan kecil" yang jika digabungkan akan membentuk logika sebuah program, seperti percabangan (*if-else*), perulangan (*loop*), maupun pemanggilan fungsi.

---

# Mengenal Instruksi Assembly Dasar

Assembly terdiri dari banyak instruksi, tetapi ada beberapa yang hampir selalu muncul saat melakukan reverse engineering.

## MOV (Move)

Instruksi **MOV** digunakan untuk menyalin data dari satu tempat ke tempat lain.

Contohnya:

```asm
mov rax, rbx
```

Artinya, isi register **RBX** disalin ke **RAX**.

Perlu diingat, instruksi ini **tidak memindahkan** data, melainkan hanya **menyalinnya**. Nilai pada register sumber tetap tidak berubah.

---

## Mengakses Memori

Selain memindahkan data antar register, Assembly juga dapat mengambil data dari memori.

Misalnya:

```asm
mov eax, [rbx]
```

Instruksi tersebut berarti CPU mengambil data yang berada pada alamat memori yang ditunjuk oleh **RBX**, kemudian menyimpannya ke register **EAX**.

Kadang kita juga akan menemukan bentuk seperti:

```asm
mov eax, [rbx + rcx*4]
```

Awalnya terlihat rumit, tetapi sebenarnya instruksi ini hanya menghitung alamat memori menggunakan rumus:

```text
Alamat = Base + (Index × Ukuran Data)
```

Format seperti ini sering digunakan ketika program sedang mengakses elemen di dalam sebuah **array**.

---

## LEA (Load Effective Address)

Instruksi **LEA** adalah salah satu instruksi yang sering membuat pemula bingung.

Misalnya:

```asm
lea rax, [rbx+8]
```

Sekilas terlihat seperti membaca isi memori karena menggunakan tanda kurung siku `[]`.

Padahal sebenarnya **LEA tidak mengambil data dari memori**.

Instruksi ini hanya menghitung alamatnya saja, lalu menyimpan hasil perhitungannya ke register tujuan.

Cara paling mudah mengingatnya adalah:

* **MOV** → mengambil isi memori.
* **LEA** → mengambil alamat memorinya.

---

# Memahami Control Flow

Pada bahasa pemrograman seperti C atau Python, kita mengenal:

* `if`
* `else`
* `while`
* `for`
* `switch`

Namun di Assembly, semua struktur tersebut sebenarnya tidak ada.

CPU hanya mengenal dua hal:

1. membandingkan nilai,
2. melompat ke alamat tertentu.

Dari dua operasi sederhana inilah seluruh logika program dibangun.

---

## CMP (Compare)

Instruksi **CMP** digunakan untuk membandingkan dua nilai.

Contohnya:

```asm
cmp eax, ebx
```

Yang menarik, **CMP tidak menyimpan hasil perbandingan**.

Instruksi ini hanya mengubah kondisi pada register **EFLAGS**.

Beberapa flag yang sering muncul adalah:

| Flag                | Fungsi                                   |
| ------------------- | ---------------------------------------- |
| **ZF (Zero Flag)**  | Bernilai 1 jika hasil perbandingan sama. |
| **SF (Sign Flag)**  | Menunjukkan hasil bernilai negatif.      |
| **CF (Carry Flag)** | Digunakan pada operasi unsigned.         |

Flag-flag inilah yang nantinya menentukan apakah program akan berpindah ke jalur tertentu atau tidak.

---

## Jump (JMP)

Setelah melakukan perbandingan menggunakan **CMP**, biasanya akan muncul instruksi **Jump**.

Beberapa yang paling sering ditemui adalah:

| Instruksi | Arti                                  |
| --------- | ------------------------------------- |
| **JMP**   | Selalu melompat.                      |
| **JZ**    | Melompat jika ZF = 1 (hasil sama).    |
| **JNZ**   | Melompat jika ZF = 0 (hasil berbeda). |
| **JL**    | Melompat jika lebih kecil.            |
| **JG**    | Melompat jika lebih besar.            |

Sederhananya:

```text
CMP
   │
   ▼
Apakah syarat terpenuhi?
        │
   Ya ─────► Jump
        │
Tidak
        ▼
Lanjut ke instruksi berikutnya
```

Dari kombinasi **CMP** dan **Jump**, compiler dapat membentuk berbagai struktur seperti `if`, `else`, maupun `while`.

---

## Switch-Case dan Jump Table

Jika sebuah program memiliki banyak pilihan, misalnya:

```c
switch(menu)
```

Compiler biasanya tidak membuat banyak instruksi `CMP` satu per satu.

Sebagai gantinya, compiler membuat sebuah **Jump Table**.

Jump Table dapat dibayangkan sebagai **daftar alamat tujuan**.

Ketika nilai `menu` diketahui, CPU cukup mengambil alamat yang sesuai dari tabel tersebut, lalu langsung melompat ke sana.

Teknik ini jauh lebih cepat dibandingkan melakukan banyak proses perbandingan.

---

# Calling Convention

Ketika sebuah fungsi dipanggil, muncul pertanyaan:

> Bagaimana cara fungsi tersebut menerima parameter?

Jawabannya diatur oleh sesuatu yang disebut **Calling Convention**.

Calling Convention adalah aturan yang menentukan:

* parameter dikirim lewat mana,
* siapa yang membersihkan stack,
* dan bagaimana nilai hasil (*return value*) dikembalikan.

---

## Windows x64

Pada Windows x64, empat parameter pertama dikirim melalui register.

| Parameter   | Register |
| ----------- | -------- |
| Parameter 1 | RCX      |
| Parameter 2 | RDX      |
| Parameter 3 | R8       |
| Parameter 4 | R9       |

Jika jumlah parameter lebih dari empat, sisanya disimpan di stack.

---

## Linux (System V ABI)

Linux memiliki aturan yang sedikit berbeda.

Enam parameter pertama dikirim melalui register berikut.

| Parameter   | Register |
| ----------- | -------- |
| Parameter 1 | RDI      |
| Parameter 2 | RSI      |
| Parameter 3 | RDX      |
| Parameter 4 | RCX      |
| Parameter 5 | R8       |
| Parameter 6 | R9       |

Parameter berikutnya baru disimpan di stack.

Karena itu, penting untuk mengetahui apakah program yang sedang dianalisis dibuat untuk Windows atau Linux. Jika salah menggunakan calling convention, kita bisa salah membaca parameter sebuah fungsi.

---

# Shadow Space pada Windows x64

Saat menggunakan **x64dbg**, saya sering melihat instruksi seperti:

```asm
sub rsp, 40h
```

Awalnya saya bingung mengapa hampir semua fungsi memiliki instruksi tersebut.

Setelah dipelajari, ternyata Windows memiliki aturan yang disebut **Shadow Space**.

Sebelum sebuah fungsi dipanggil, program harus menyediakan ruang kosong di stack sebesar **32 byte**.

Ruang ini nantinya dapat digunakan oleh fungsi yang dipanggil jika ingin menyimpan isi register seperti **RCX**, **RDX**, **R8**, atau **R9** ke dalam memori.

Karena itulah instruksi seperti `sub rsp, 40h` sering muncul sebelum `call`.

---

# Stack Alignment

Selain Shadow Space, Windows x64 juga memiliki aturan lain, yaitu **Stack Alignment**.

Sebelum instruksi `CALL` dijalankan, nilai **RSP** harus berada pada kelipatan **16 byte**.

Aturan ini dibuat agar prosesor dapat menjalankan instruksi tertentu dengan lebih efisien dan menghindari kesalahan saat program berjalan.

Walaupun terlihat sebagai detail kecil, aturan ini hampir selalu muncul ketika menganalisis binary Windows.

---

# Ringkasan

| Konsep                 | Inti Pembahasan                                                      |
| ---------------------- | -------------------------------------------------------------------- |
| **MOV**                | Menyalin data antar register atau antara register dan memori.        |
| **LEA**                | Menghitung alamat memori tanpa mengambil isinya.                     |
| **CMP**                | Membandingkan dua nilai dan mengubah kondisi pada EFLAGS.            |
| **JMP / JCC**          | Mengatur alur program berdasarkan kondisi tertentu.                  |
| **Jump Table**         | Tabel alamat yang digunakan untuk mengoptimalkan `switch-case`.      |
| **Calling Convention** | Aturan pengiriman parameter antar fungsi.                            |
| **Shadow Space**       | Ruang 32 byte yang wajib disediakan sebelum `CALL` pada Windows x64. |
| **Stack Alignment**    | Penyelarasan stack ke kelipatan 16 byte sebelum pemanggilan fungsi.  |