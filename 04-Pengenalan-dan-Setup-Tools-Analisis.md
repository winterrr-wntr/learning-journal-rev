# 04. Pengenalan dan Setup Tools Analisis (DIE, Ghidra, dan x64dbg)

Setelah mempelajari dasar-dasar **register**, **stack**, **heksadesimal**, dan **endianness**, langkah berikutnya adalah mengenal alat-alat yang akan digunakan selama proses reverse engineering.

Pada minggu ini, saya fokus menyiapkan **environment** sekaligus mempelajari tiga tools yang paling sering digunakan dalam reverse engineering, yaitu **Detect It Easy (DIE)**, **Ghidra**, dan **x64dbg**.

---

# Kategori Tools Reverse Engineering

Tidak semua tools memiliki fungsi yang sama. Dalam praktiknya, tools reverse engineering biasanya dibagi menjadi beberapa kategori sesuai dengan tahap analisis yang dilakukan.

## 1. Identifier / Triage Tools

Tahap pertama sebelum menganalisis sebuah program adalah mencari tahu informasi dasarnya.

Beberapa pertanyaan yang biasanya ingin dijawab adalah:

* Program ini dibuat untuk arsitektur 32-bit atau 64-bit?
* Compiler apa yang digunakan?
* Apakah program ini menggunakan packer?
* Apakah file ini memiliki proteksi tertentu?

Untuk menjawab pertanyaan tersebut, kita menggunakan **Identifier** atau **Triage Tools**.

### Contoh Tools

* Detect It Easy (DIE)
* Exeinfo PE
* Bytehist

Dari ketiga tools tersebut, saya menggunakan **Detect It Easy (DIE)**.

---

## Detect It Easy (DIE)

**Detect It Easy (DIE)** adalah tool yang digunakan untuk mengenali karakteristik sebuah file executable.

Dengan DIE, kita dapat mengetahui berbagai informasi penting, seperti:

* jenis file (PE, ELF, dan lain-lain),
* arsitektur program (32-bit atau 64-bit),
* compiler yang digunakan,
* apakah file menggunakan packer,
* hingga protector yang digunakan.

Informasi ini sangat membantu sebelum analisis dilakukan.

Misalnya, jika sebuah program ternyata menggunakan **UPX** atau packer lainnya, maka proses analisis biasanya menjadi lebih sulit karena isi program masih "terbungkus".

Dalam kondisi seperti ini, langkah pertama yang harus dilakukan adalah **unpacking**, sehingga kode aslinya dapat dianalisis dengan lebih mudah.

---

# 2. Disassembler dan Decompiler (Static Analysis)

Setelah mengetahui informasi dasar sebuah program, tahap berikutnya adalah melihat isi program tersebut tanpa menjalankannya.

Proses ini disebut **Static Analysis**.

Tool yang digunakan pada tahap ini disebut **Disassembler** atau **Decompiler**.

### Contoh Tools

* Ghidra
* IDA Pro

Pada jurnal ini saya menggunakan **Ghidra** karena bersifat gratis dan memiliki fitur yang sangat lengkap.

---

## Ghidra

**Ghidra** adalah framework reverse engineering yang dikembangkan oleh **National Security Agency (NSA)**.

Fungsi utama Ghidra adalah mengubah file biner menjadi instruksi **assembly**, kemudian mencoba menerjemahkannya menjadi **pseudo-code** yang bentuknya mirip bahasa C.

Hal ini membuat program jauh lebih mudah dipahami dibanding hanya melihat kumpulan opcode.

Beberapa fitur yang menurut saya paling membantu adalah:

* **Decompiler**, untuk melihat pseudo-code yang lebih mudah dibaca.
* **Listing View**, untuk melihat instruksi assembly.
* **Function List**, untuk melihat semua fungsi yang ada di dalam program.
* **Strings**, untuk mencari teks yang tersimpan di dalam executable.
* **Control Flow Graph (CFG)**, untuk melihat alur logika program dalam bentuk diagram.

Dengan Ghidra, kita dapat memahami bagaimana sebuah program bekerja tanpa perlu menjalankannya terlebih dahulu.

---

# 3. Debugger (Dynamic Analysis)

Jika Ghidra digunakan untuk melihat program dalam keadaan "diam", maka debugger digunakan untuk melihat program **saat sedang berjalan**.

Tahap ini disebut **Dynamic Analysis**.

Debugger memberikan kendali penuh terhadap jalannya program sehingga kita dapat menghentikan, menjalankan, atau mengeksekusi instruksi satu per satu.

### Contoh Tools

* x64dbg
* x32dbg
* GDB (Linux)

Pada pembelajaran kali ini saya menggunakan **x64dbg**.

---

## x64dbg

**x64dbg** adalah debugger yang dirancang untuk menganalisis aplikasi Windows.

Saat menggunakan x64dbg, kita dapat melihat banyak informasi penting, seperti:

* isi register,
* isi stack,
* isi memori,
* instruksi assembly,
* hingga nilai flag CPU.

Debugger juga memungkinkan kita menghentikan program pada titik tertentu agar kita bisa mengamati apa yang sedang terjadi.

---

# Shortcut Dasar x64dbg

Berikut beberapa shortcut yang paling sering digunakan selama proses debugging.

| Tombol | Fungsi                                                                |
| ------ | --------------------------------------------------------------------- |
| **F2** | Menambahkan atau menghapus breakpoint.                                |
| **F7** | Menjalankan satu instruksi dan masuk ke dalam fungsi (*Step Into*).   |
| **F8** | Menjalankan satu instruksi tanpa masuk ke fungsi (*Step Over*).       |
| **F9** | Menjalankan program hingga breakpoint berikutnya atau sampai selesai. |

Keempat shortcut ini merupakan dasar yang hampir selalu digunakan saat melakukan debugging.

---

# Breakpoint

Salah satu fitur paling penting pada debugger adalah **Breakpoint**.

Breakpoint dapat diibaratkan seperti tombol **pause** pada sebuah video.

Saat program mencapai titik tersebut, eksekusi akan berhenti sementara sehingga kita dapat melihat:

* isi register,
* kondisi stack,
* isi memori,
* maupun instruksi yang akan dijalankan berikutnya.

Tanpa breakpoint, program akan berjalan terlalu cepat sehingga proses analisis menjadi jauh lebih sulit.

---

# Mengubah Nilai Saat Debugging

Selain mengamati program, debugger juga memungkinkan kita melakukan perubahan secara langsung.

Sebagai contoh, kita dapat:

* mengubah isi register,
* mengubah nilai di memori,
* bahkan mengubah **CPU Flag**, seperti **Zero Flag (ZF)**.

Teknik ini sering digunakan untuk memahami bagaimana sebuah kondisi memengaruhi jalannya program.

Misalnya, jika sebuah percabangan bergantung pada nilai **ZF**, kita dapat mengubah nilainya untuk melihat apakah program akan mengambil jalur yang berbeda.

---

# Alur Penggunaan Tools

Dalam praktiknya, proses analisis biasanya dilakukan dengan urutan seperti berikut.

```text
File Program
      │
      ▼
Detect It Easy (DIE)
(Melihat informasi awal)
      │
      ▼
Ghidra
(Memahami isi program)
      │
      ▼
x64dbg
(Melihat program saat berjalan)
```

Urutan ini tidak selalu harus diikuti, tetapi cukup umum digunakan ketika mulai menganalisis sebuah executable.

---

# Ringkasan

| Tool                     | Fungsi Utama                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------ |
| **Detect It Easy (DIE)** | Mengenali informasi dasar file executable, seperti arsitektur, compiler, dan packer.             |
| **Ghidra**               | Membongkar executable menjadi assembly dan pseudo-code untuk analisis statis.                    |
| **x64dbg**               | Menjalankan program sambil mengamati register, memori, stack, dan alur eksekusi secara langsung. |

---
