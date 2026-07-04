# 03. Representasi Data, Heksadesimal, dan Endianness

## Pendahuluan

Saat pertama kali membuka sebuah program menggunakan debugger seperti **x64dbg** atau disassembler seperti **Ghidra**, kemungkinan besar kamu tidak akan melihat tulisan yang mudah dipahami. Yang muncul justru angka dan huruf seperti ini:

```text
48 89 E5
FC DE 21 43
FF D0
```

Kalau baru belajar, tampilannya memang terlihat membingungkan. Tapi tenang, angka-angka itu sebenarnya hanya cara komputer menyimpan dan menampilkan data.

Di entri ini kita akan membahas dua hal yang paling sering ditemui saat melakukan reverse engineering, yaitu **heksadesimal** dan **endianness**.

---

# Apa Itu Heksadesimal?

Komputer sebenarnya hanya mengenal **0** dan **1** (biner). Masalahnya, kalau semua data ditampilkan dalam bentuk biner, tampilannya akan sangat panjang dan sulit dibaca.

Sebagai gantinya, hampir semua tools reverse engineering menggunakan **heksadesimal (hex)** karena jauh lebih ringkas.

Misalnya:

| Desimal | Heksadesimal |
| ------: | -----------: |
|      10 |            A |
|      11 |            B |
|      12 |            C |
|      13 |            D |
|      14 |            E |
|      15 |            F |

Jadi setelah angka **9**, urutannya bukan **10**, tetapi **A**, lalu **B**, **C**, hingga **F**.

Biasanya angka heksadesimal ditulis seperti berikut:

```text
0xDEADBEEF
```

atau

```text
DEADBEEFh
```

Keduanya memiliki arti yang sama.

---

# Kenapa Reverse Engineering Menggunakan Heksadesimal?

Alasannya sederhana: **lebih mudah dibaca.**

Misalnya huruf **H**.

Kalau ditulis dalam biner:

```text
01001000
```

Sedangkan dalam heksadesimal cukup:

```text
48
```

Jauh lebih pendek, bukan?

Karena itu, hampir semua debugger dan disassembler akan menampilkan isi memori dalam bentuk heksadesimal.

Sebagai contoh, kata:

```text
Hello
```

Di dalam memori akan terlihat seperti:

```text
48 65 6C 6C 6F
```

Setiap dua digit heksadesimal mewakili **1 byte**.

---

# Apa Itu Endianness?

Setelah memahami heksadesimal, ada satu konsep lagi yang wajib dipahami, yaitu **Endianness**.

Sederhananya, Endianness adalah **cara komputer menyusun data di dalam memori**.

Misalnya kita punya angka:

```text
0x12345678
```

Pertanyaannya, saat angka ini disimpan ke memori, urutannya bagaimana?

Nah, jawabannya tergantung jenis Endianness yang digunakan.

---

## Big-Endian

Pada **Big-Endian**, urutannya sama seperti kita membaca angka.

```text
12 34 56 78
```

Tidak ada yang berubah.

---

## Little-Endian

Sedangkan pada **Little-Endian**, urutan setiap byte dibalik.

```text
78 56 34 12
```

Yang dibalik **bukan setiap angka**, tetapi setiap **byte** (dua digit heksadesimal).

Ini adalah bagian yang paling sering membuat pemula bingung.

---

# x86 dan x64 Menggunakan Little-Endian

Kabar baiknya, kamu tidak perlu menghafal banyak aturan.

Cukup ingat satu hal:

> **Semua komputer modern dengan arsitektur x86 dan x64 menggunakan Little-Endian.**

Artinya, setiap kali melihat isi memori di debugger, kemungkinan besar urutannya memang terlihat "terbalik".

---

# Contoh

Misalnya ada sebuah alamat:

```text
0x4321DEFC
```

Di kepala kita, kita membacanya seperti ini:

```text
43 21 DE FC
```

Tetapi ketika disimpan ke memori, urutannya berubah menjadi:

```text
FC DE 21 43
```

Kalau pertama kali melihat ini, jangan panik. Nilainya **tetap sama**, hanya cara penyimpanannya yang berbeda.

---

# Kenapa Ini Penting?

Saat melakukan reverse engineering, kita hampir selalu berurusan dengan alamat memori.

Misalnya kita ingin menulis alamat:

```text
0x4321DEFC
```

Kalau kita menulis:

```text
43 21 DE FC
```

hasilnya justru salah.

Yang benar adalah:

```text
FC DE 21 43
```

Karena prosesor x86/x64 membacanya menggunakan format Little-Endian.

Hal ini juga sering muncul saat mempelajari **Buffer Overflow**, **ROP**, maupun teknik eksploitasi lainnya. Banyak exploit gagal hanya karena urutan byte yang dimasukkan salah.

---

# Ringkasan

| Konsep            | Intinya                                                         |
| ----------------- | --------------------------------------------------------------- |
| **Heksadesimal**  | Cara yang lebih ringkas untuk menampilkan data dibanding biner. |
| **1 Byte**        | Ditulis menggunakan dua digit heksadesimal.                     |
| **Big-Endian**    | Data disimpan sesuai urutan normal.                             |
| **Little-Endian** | Data disimpan dengan urutan byte yang dibalik.                  |
| **x86/x64**       | Menggunakan Little-Endian.                                      |

---