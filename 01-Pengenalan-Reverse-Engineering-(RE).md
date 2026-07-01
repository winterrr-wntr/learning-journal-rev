#01. Pengenalan Reverse Engineering (RE) & Metodologi Analisis

## Apa Itu Reverse Engineering?

**Reverse Engineering (RE)** adalah proses mempelajari bagaimana sebuah program bekerja tanpa memiliki kode sumber (*source code*) yang dibuat oleh pengembangnya. Ibarat membongkar sebuah jam tangan untuk mengetahui bagaimana setiap roda giginya bekerja, reverse engineering bertujuan memahami isi dan cara kerja sebuah aplikasi dari hasil akhirnya, yaitu file program yang sudah jadi.

Keahlian ini banyak digunakan dalam dunia cyber security, pengembangan perangkat lunak, hingga penelitian. Beberapa tujuan utamanya antara lain:

* **Menganalisis malware.** Dengan membedah malware, kita dapat mengetahui bagaimana cara malware tersebut bekerja, apa yang dilakukannya pada komputer korban, dan bagaimana cara menghentikannya.
* **Menguji sistem perlindungan perangkat lunak.** Banyak aplikasi menggunakan mekanisme perlindungan seperti lisensi atau *Digital Rights Management* (DRM). Reverse engineering membantu memahami bagaimana sistem tersebut bekerja sehingga pengembang dapat memperbaiki kelemahannya.
* **Memahami teknik obfuscation.** Beberapa program sengaja dibuat sulit dibaca menggunakan teknik yang disebut *obfuscation*. Reverse engineering digunakan untuk mengembalikan logika program agar lebih mudah dipahami.

Singkatnya, reverse engineering bukan hanya tentang "membongkar program", tetapi juga tentang memahami logika, perilaku, dan cara sebuah aplikasi berinteraksi dengan sistem.

---

# Metodologi Analisis Biner

Saat praktik, reverse engineering tidak dilakukan secara asal. Ada beberapa tahapan yang biasanya dilakukan agar proses analisis menjadi lebih aman, terstruktur, dan menghasilkan informasi yang lengkap.

## 1. Menyiapkan Lingkungan yang Aman (Controlled Lab)

Langkah pertama adalah menyiapkan lingkungan khusus untuk analisis, biasanya menggunakan **Virtual Machine (VM)** atau **sandbox**.

Hal ini penting karena kita belum mengetahui apakah program yang akan dianalisis aman atau justru berbahaya. Jika ternyata program tersebut adalah malware, dampaknya hanya terjadi di lingkungan virtual dan tidak merusak komputer utama.

> Anggap saja seperti menguji bahan kimia di laboratorium. Pengujian dilakukan di tempat yang aman agar tidak membahayakan lingkungan sekitar.

---

## 2. Analisis Statis (Static Analysis)

Setelah lingkungan siap, langkah berikutnya adalah mempelajari program **tanpa menjalankannya**.

Pada tahap ini, kita mencoba mengumpulkan informasi sebanyak mungkin, misalnya:

* struktur file program,
* teks atau *string* yang tersimpan di dalamnya,
* fungsi-fungsi yang digunakan,
* serta alur logika program.

Salah satu alat yang sering digunakan adalah **IDA Pro**, yaitu *disassembler* yang mengubah file biner menjadi instruksi **assembly** sehingga lebih mudah dipelajari.

Karena program belum dijalankan, tahap ini relatif aman dan sering digunakan untuk membangun gambaran awal mengenai cara kerja aplikasi.

---

## 3. Analisis Perilaku (Behavioral Analysis)

Berbeda dengan analisis statis, pada tahap ini program mulai **dijalankan**.

Tujuannya adalah melihat apa saja yang dilakukan program saat berjalan, misalnya:

* membuat atau mengubah file,
* mengubah pengaturan sistem,
* membuat koneksi ke internet,
* atau berkomunikasi dengan server tertentu.

Beberapa alat yang umum digunakan antara lain:

* **Process Monitor (ProcMon)** untuk memantau aktivitas file dan sistem.
* **RegShot** untuk membandingkan perubahan pada Windows Registry sebelum dan sesudah program dijalankan.
* **Wireshark** untuk menangkap lalu lintas jaringan.
* **FakeNet-NG** untuk mensimulasikan koneksi internet sehingga komunikasi malware dapat diamati tanpa benar-benar terhubung ke server aslinya.

Tahap ini membantu kita memahami perilaku nyata dari program, bukan hanya membaca kodenya.

---

## 4. Analisis Dinamis (Dynamic Analysis)

Jika analisis perilaku hanya mengamati apa yang dilakukan program, maka analisis dinamis memungkinkan kita **mengendalikan jalannya program**.

Hal ini dilakukan menggunakan **debugger**, yaitu alat yang dapat menghentikan program pada titik tertentu sehingga kita bisa melihat apa yang sedang terjadi.

Debugger yang sering digunakan adalah **x32dbg** dan **x64dbg**.

Dengan debugger, kita dapat:

* menjalankan program satu instruksi demi satu instruksi,
* memasang **breakpoint** (titik henti),
* melihat isi memori,
* memeriksa nilai variabel,
* dan memahami logika program secara lebih mendalam.

Teknik ini sangat berguna ketika sebuah program memiliki logika yang rumit atau sengaja dibuat sulit dipahami.

---

## 5. Unpacking dan Memory Forensics

Beberapa program, terutama malware, dikemas menggunakan **packer**.

Packer dapat diibaratkan seperti file ZIP, tetapi untuk program. Tujuannya adalah mengecilkan ukuran file atau menyembunyikan isi program agar lebih sulit dianalisis.

Karena itu, sebelum melakukan analisis lebih lanjut, kita sering kali harus membuka lapisan pelindung tersebut (*unpacking*).

Beberapa alat yang umum digunakan adalah:

* **Detect It Easy (DIE)** untuk mendeteksi apakah sebuah program menggunakan packer.
* **Scylla** atau **OllyDumpEx** untuk mengambil (*dump*) program setelah berhasil terbuka di memori.
* Debugger digunakan untuk menemukan **Original Entry Point (OEP)**, yaitu titik awal eksekusi program setelah proses unpacking selesai.

Setelah proses ini berhasil dilakukan, isi program biasanya menjadi jauh lebih mudah dianalisis menggunakan teknik analisis statis maupun dinamis.

---

Reverse engineering bukan sekadar melihat isi sebuah program, melainkan proses memahami bagaimana program tersebut bekerja dari berbagai sudut pandang. Mulai dari mempelajari struktur file tanpa menjalankannya, mengamati perilakunya saat dieksekusi, hingga mengikuti setiap instruksi program menggunakan debugger.

Dengan mengikuti tahapan yang terstruktur, proses analisis menjadi lebih aman, lebih sistematis, dan memberikan pemahaman yang jauh lebih mendalam mengenai cara kerja sebuah aplikasi.
