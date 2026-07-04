# Fundamental Arsitektur x86/x64 dan Memori Stack

## Pendahuluan

Ketika membuat sebuah program menggunakan bahasa seperti C, C++, atau Java, kita biasanya hanya melihat kode yang mudah dipahami manusia. Namun, komputer tidak menjalankan kode tersebut secara langsung. Sebelum dieksekusi oleh CPU, kode akan diterjemahkan menjadi **bahasa mesin (machine code)** yang terdiri dari instruksi-instruksi sederhana.

Dalam **Reverse Engineering**, kita bekerja pada level tersebut. Kita mencoba memahami bagaimana CPU menjalankan setiap instruksi, bagaimana data disimpan, dan bagaimana sebuah fungsi dipanggil. Oleh karena itu, memahami **register**, **stack**, dan perbedaan antara arsitektur **x86 (32-bit)** dan **x64 (64-bit)** merupakan fondasi yang sangat penting.

---

# Apa itu Register?

**Register** adalah tempat penyimpanan data berukuran kecil yang berada langsung di dalam CPU. Karena letaknya sangat dekat dengan prosesor, register memiliki kecepatan yang jauh lebih tinggi dibandingkan RAM.

CPU menggunakan register untuk menyimpan data sementara, hasil perhitungan, alamat memori, hingga informasi mengenai instruksi yang sedang diproses.

Pada arsitektur **x64**, terdapat **16 register umum (General Purpose Register)**. Sebagai pemula, Anda tidak perlu menghafal semuanya. Cukup pahami tiga register berikut karena hampir selalu muncul saat melakukan analisis menggunakan debugger.

| Register      | Fungsi                                                                                    | Analogi Sederhana                                                                        |
| ------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **RIP / EIP** | Menyimpan alamat instruksi berikutnya yang akan dijalankan CPU.                           | Seperti penanda halaman pada buku yang menunjukkan halaman berikutnya yang harus dibaca. |
| **RSP / ESP** | Menunjukkan posisi paling atas dari stack.                                                | Seperti jari yang menunjuk piring paling atas pada sebuah tumpukan piring.               |
| **RBP / EBP** | Menjadi titik acuan (base) untuk mengakses data pada stack selama sebuah fungsi berjalan. | Seperti titik nol pada penggaris yang digunakan sebagai patokan pengukuran.              |

> **Catatan:** Huruf **E** digunakan pada register 32-bit (*Extended Register*), sedangkan huruf **R** digunakan pada register 64-bit.

---

# Perbedaan Arsitektur x86 dan x64

Sebelum melakukan reverse engineering, penting untuk mengetahui apakah program yang dianalisis merupakan aplikasi **32-bit (x86)** atau **64-bit (x64)**. Perbedaannya bukan hanya pada ukuran data, tetapi juga memengaruhi cara CPU menjalankan instruksi.

## 1. Jumlah dan Ukuran Register

Pada arsitektur **x86**, CPU hanya memiliki **8 register umum**.

Contohnya:

* EAX
* EBX
* ECX
* EDX
* ESI
* EDI
* ESP
* EBP

Sedangkan pada **x64**, ukuran register diperbesar menjadi 64-bit dan namanya berubah menggunakan awalan **R**.

Contohnya:

* RAX
* RBX
* RCX
* RDX
* RSI
* RDI
* RSP
* RBP

Selain itu, x64 juga menambahkan **8 register baru**, yaitu:

```
R8  R9  R10  R11  R12  R13  R14  R15
```

Sehingga total terdapat **16 General Purpose Register**, yang membuat CPU dapat menyimpan lebih banyak data tanpa harus sering mengakses RAM.

---

## 2. Cara Mengirim Argumen ke Fungsi (Calling Convention)

Perbedaan paling terasa saat melakukan reverse engineering adalah bagaimana sebuah fungsi menerima parameter.

### Pada x86 (32-bit)

Hampir semua parameter dikirim melalui **stack**.

Misalnya terdapat fungsi:

```c
sum(10, 20);
```

Secara sederhana, assembly akan terlihat seperti:

```asm
push 20
push 10
call sum
```

Artinya, setiap parameter dimasukkan terlebih dahulu ke stack sebelum fungsi dipanggil.

---

### Pada x64 (Windows)

Empat parameter pertama **tidak lagi dimasukkan ke stack**, melainkan langsung dikirim melalui register.

| Parameter   | Register |
| ----------- | -------- |
| Parameter 1 | RCX      |
| Parameter 2 | RDX      |
| Parameter 3 | R8       |
| Parameter 4 | R9       |

Jika jumlah parameter lebih dari empat, barulah sisanya disimpan ke stack.

Pendekatan ini membuat proses pemanggilan fungsi menjadi lebih cepat karena CPU tidak perlu terlalu sering mengakses memori.

> **Catatan:** Aturan di atas berlaku untuk **Windows x64**. Pada sistem Linux (System V AMD64 ABI), urutan register berbeda, yaitu **RDI, RSI, RDX, RCX, R8, dan R9**.

---

## 3. Return Value

Walaupun cara mengirim parameter berbeda, baik x86 maupun x64 memiliki kesamaan.

Nilai yang dikembalikan oleh sebuah fungsi (*return value*) biasanya disimpan pada:

| Arsitektur | Return Register |
| ---------- | --------------- |
| x86        | EAX             |
| x64        | RAX             |

Karena itu, saat melakukan reverse engineering, register EAX atau RAX sering menjadi tempat pertama yang diperiksa setelah sebuah fungsi selesai dipanggil.

---

## 4. Pengalamatan Memori (RIP-Relative Addressing)

Salah satu fitur penting yang hanya dimiliki arsitektur x64 adalah **RIP-relative addressing**.

Alih-alih menggunakan alamat memori yang tetap (*absolute address*), program dapat mengakses data berdasarkan jaraknya dari posisi instruksi yang sedang dijalankan (RIP).

Teknik ini membuat program menjadi lebih fleksibel karena dapat dijalankan di alamat memori yang berbeda tanpa perlu mengubah isi kodenya. Fitur ini juga mendukung mekanisme keamanan modern seperti **Address Space Layout Randomization (ASLR)**.

---

# Mengenal Stack

**Stack** adalah area memori yang digunakan program untuk menyimpan data sementara selama proses eksekusi.

Beberapa hal yang biasanya disimpan di dalam stack adalah:

* Variabel lokal.
* Parameter fungsi.
* Return address.
* Data sementara selama fungsi berjalan.

Stack menggunakan prinsip **LIFO (Last In, First Out)**, yaitu **data yang terakhir masuk akan menjadi data pertama yang keluar**.

---

## Analogi Stack

Bayangkan sebuah **tumpukan piring**.

Ketika menambahkan piring baru, kita selalu meletakkannya di bagian paling atas.

Begitu juga ketika mengambil piring, kita mengambil piring yang berada di atas terlebih dahulu.

Stack bekerja dengan cara yang sama.

```
+-----------+
| Data C    | ← Paling atas
+-----------+
| Data B    |
+-----------+
| Data A    |
+-----------+
```

---

## Stack Bertumbuh ke Bawah

Hal yang sering membingungkan pemula adalah arah pertumbuhan stack.

Pada arsitektur x86 maupun x64, stack justru **bertumbuh ke bawah**, yaitu menuju alamat memori yang nilainya semakin kecil.

```
Alamat Tinggi
-----------------------
Data A
Data B
Data C  ← RSP
-----------------------
Alamat Rendah
```

Artinya, setiap kali ada data baru yang dimasukkan ke stack, nilai **RSP** akan berkurang.

---

# Instruksi Dasar Stack

CPU menggunakan beberapa instruksi penting untuk mengelola stack.

| Instruksi | Fungsi                                                                 |
| --------- | ---------------------------------------------------------------------- |
| **PUSH**  | Menambahkan data ke bagian paling atas stack.                          |
| **POP**   | Mengambil data dari bagian paling atas stack.                          |
| **CALL**  | Memanggil fungsi dan menyimpan return address ke stack.                |
| **RET**   | Mengambil return address dari stack lalu kembali ke fungsi sebelumnya. |

---

## PUSH

Instruksi **PUSH** digunakan untuk memasukkan data ke stack.

Misalnya:

```asm
push rax
```

CPU akan:

1. Mengurangi nilai RSP.
2. Menyimpan isi register RAX ke alamat yang ditunjuk oleh RSP.

---

## POP

Instruksi **POP** digunakan untuk mengambil data dari stack.

Contohnya:

```asm
pop rax
```

CPU akan:

1. Mengambil data dari alamat yang ditunjuk RSP.
2. Menyimpannya ke RAX.
3. Menambah kembali nilai RSP.

---

## CALL dan RET

Salah satu fungsi terpenting stack adalah membantu CPU mengingat "jalan pulang".

Misalnya terdapat alur program seperti berikut.

```
main()
   │
   ▼
login()
   │
   ▼
checkPassword()
```

Ketika `main()` memanggil `login()`, CPU menjalankan instruksi **CALL**.

Instruksi ini akan:

1. Menyimpan alamat instruksi berikutnya (*return address*) ke dalam stack.
2. Melompat ke fungsi `login()`.

Ketika `login()` selesai, instruksi **RET** akan dijalankan.

RET mengambil return address dari stack, memasukkannya kembali ke register **RIP**, kemudian CPU melanjutkan eksekusi tepat dari lokasi sebelum fungsi dipanggil.

Tanpa mekanisme ini, CPU tidak akan mengetahui ke mana ia harus kembali setelah sebuah fungsi selesai dijalankan.

---

# Mengapa Semua Ini Penting?

Hampir seluruh proses reverse engineering melibatkan register dan stack.

Saat menggunakan debugger seperti **x64dbg**, Anda akan terus melihat register berubah, isi stack bertambah atau berkurang, serta instruksi seperti **PUSH**, **POP**, **CALL**, dan **RET** muncul hampir di setiap fungsi.

Selain itu, banyak teknik eksploitasi keamanan memanfaatkan kelemahan pada stack.

Contoh yang paling terkenal adalah **Buffer Overflow**, yaitu kondisi ketika sebuah program menerima data melebihi kapasitas yang disediakan. Jika program tidak memiliki mekanisme perlindungan yang baik, data tersebut dapat menimpa **return address** yang berada di stack.

Akibatnya, ketika instruksi **RET** dijalankan, CPU tidak lagi kembali ke alamat yang seharusnya, melainkan ke alamat yang telah dimanipulasi oleh penyerang. Teknik inilah yang menjadi dasar dari berbagai eksploitasi klasik dalam keamanan siber.

---

# Ringkasan

| Konsep         | Inti Pembahasan                                                                                       |
| -------------- | ----------------------------------------------------------------------------------------------------- |
| **Register**   | Tempat penyimpanan data tercepat yang berada di dalam CPU.                                            |
| **RIP/EIP**    | Menunjukkan instruksi berikutnya yang akan dijalankan.                                                |
| **RSP/ESP**    | Menunjukkan posisi paling atas stack.                                                                 |
| **RBP/EBP**    | Menjadi titik acuan selama fungsi berjalan.                                                           |
| **Stack**      | Area memori yang menyimpan data sementara dan mengatur pemanggilan fungsi.                            |
| **PUSH**       | Memasukkan data ke stack.                                                                             |
| **POP**        | Mengambil data dari stack.                                                                            |
| **CALL**       | Memanggil fungsi dan menyimpan alamat kembali.                                                        |
| **RET**        | Mengembalikan eksekusi ke fungsi sebelumnya.                                                          |
| **x86 vs x64** | Berbeda pada ukuran register, jumlah register, calling convention, dan mekanisme pengalamatan memori. |
