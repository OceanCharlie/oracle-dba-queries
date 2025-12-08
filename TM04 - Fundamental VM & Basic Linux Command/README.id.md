# TM04 - Fundamental VM & Basic Linux Commands

Dokumen ini berisi rangkuman materi dan panduan praktikum mengenai dasar-dasar **Virtual Machine (VM)** serta perintah dasar sistem operasi **Linux**.

## Daftar Isi

  - [Tujuan Pembelajaran](#tujuan-pembelajaran)
  - [Pengenalan Virtual Machine & Linux](#1-pengenalan-virtual-machine--linux)
  - [Navigasi Dasar (Navigation)](#2-navigasi-dasar-navigation)
  - [Manajemen File & Folder](#3-manajemen-file--folder)
  - [Manajemen User & Permission](#4-manajemen-user--permission)
  - [Sumber Daya Tambahan](#sumber-daya-tambahan)

## Tujuan Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa diharapkan mampu:
* Memahami fundamental Virtual Machine.
* Mengenal command dasar Linux.
* Membuat, mengelola, dan menghapus file/direktori.
* Memahami konsep *permission* file.

---

## 1. Pengenalan Virtual Machine & Linux

### Apa itu Virtual Machine?

Virtual Machine (VM) adalah lingkungan virtual yang mengemulasikan komputer fisik di dalam komputer utama. Hal ini memungkinkan pengguna menjalankan sistem operasi lain secara aman, fleksibel, dan terisolasi tanpa mengganggu sistem utama.

**Mengapa menggunakan VM?**
* **Keamanan:** Linux berjalan terpisah dari sistem utama, sehingga kesalahan tidak merusak laptop.
* **Fleksibilitas:** Memungkinkan eksperimen tanpa risiko dan menjalankan multi-OS bersamaan.
* **Simulasi Nyata:** Menghadirkan simulasi server Linux sungguhan.

### Apa itu Linux?

Linux adalah sistem operasi (OS) *open-source* berbasis UNIX yang menghubungkan *hardware* dan *software*, dikenal stabil, aman, dan gratis.

**Perbandingan Singkat:**

| Fitur | Linux | Windows | MacOS |
| :--- | :--- | :--- | :--- |
| **Sifat** | Open-source, Gratis | Proprietary, Berbayar | Proprietary, Berbayar |
| **Fleksibilitas** | Sangat Tinggi | Terbatas | Sangat Terbatas (Ekosistem Apple) |
| **Keamanan** | Relatif aman, jarang virus | Rentan virus & malware | Sangat aman (tertutup) |
| **Biaya** | Gratis | Berbayar (Lisensi) | Mahal (Bundling Device) |

---

## 2. Navigasi Dasar (Navigation)

Berikut adalah perintah dasar untuk berpindah antar direktori dan melihat isi folder:

* **`pwd`** (*Print Working Directory*):
    Menampilkan *path* (lokasi) direktori tempat Anda bekerja saat ini.
    ```bash
    [ocean@localhost ~]$ pwd
    /home/ocean
    ```

* **`ls`** (*List*):
    Menampilkan daftar file dan folder. Output biasanya berwarna: Putih (File), Biru (Direktori), Hijau (Executable).

* **`ls -l`** (*List Long*):
    Menampilkan daftar isi direktori dengan detail lengkap (izin akses, pemilik, ukuran, tanggal).

* **`cd`** (*Change Directory*):
    Mengganti posisi kursor ke lokasi direktori lain.
    ```bash
    cd /home/ocean/Downloads
    ```

---

## 3. Manajemen File & Folder

### Membuat File

Ada beberapa cara untuk membuat file di Linux:

1.  **`touch namafile`**: Membuat file kosong baru atau mengubah *timestamp*.
2.  **`nano namafile`**: Editor teks sederhana dan *user-friendly* (Keluar: `Ctrl+X`, Save: `Ctrl+O`).
3.  **`vi namafile`**: Editor teks *powerful* namun lebih kompleks.
    * Tekan `i` untuk Insert (menulis).
    * Tekan `Esc` untuk keluar mode edit.
    * Ketik `:wq` untuk Save & Quit.

### Mengelola File

* **`cp`** (*Copy*): Menyalin file atau folder.
    ```bash
    cp Senin.txt FolderTujuan
    ```
* **`mv`** (*Move*): Memindahkan file atau me-rename file.
    ```bash
    mv Senin.txt FolderTujuan
    ```
* **`rm`** (*Remove*): Menghapus file.
    ```bash
    rm Senin.txt
    ```

### Menghapus Direktori (PENTING)

Perhatikan perbedaan kedua perintah ini agar tidak terjadi kesalahan penghapusan data:

| Perintah | Fungsi | Kondisi |
| :--- | :--- | :--- |
| **`rmdir`** | Remove Directory | Hanya bisa menghapus **direktori kosong**. |
| **`rm -r`** | Remove Recursive | Menghapus direktori **beserta seluruh isinya** (file & subfolder). |

---

## 4. Manajemen User & Permission

Linux memiliki aturan hak akses (*Permission*) untuk menentukan siapa yang boleh membaca, menulis, atau menjalankan file.

### Struktur Permission

Contoh: `drwxr-xr-x`
* Karakter pertama: `d` (directory) atau `-` (file biasa).
* 9 Karakter berikutnya dibagi 3 grup: **Owner**, **Group**, **Others**.

### Nilai Akses (Chmod)

Hak akses diwakili oleh angka:
* **r** (Read) = **4**
* **w** (Write) = **2**
* **x** (Execute) = **1**

**Tabel Total Permission:**

| Angka | Arti | Keterangan |
| :--- | :--- | :--- |
| **7** | `rwx` | Read + Write + Execute (Akses Penuh) |
| **6** | `rw-` | Read + Write |
| **5** | `r-x` | Read + Execute |
| **4** | `r--` | Read Only |
| **0** | `---` | Tidak ada akses |

### Mengubah Permission

Gunakan perintah **`chmod`** (*Change Mode*) untuk mengubah izin akses.

**Contoh:**
```bash
chmod 476 Senin.txt
```

Artinya:
  * **Owner (4)**: Read only.
  * **Group (7)**: Read + Write + Execute (Full).
  * **Others (6)**: Read + Write.

---

## Sumber Daya Tambahan

Untuk mendukung pembelajaran, Anda dapat mengakses materi pelengkap berikut:

  * **Visualisasi Tambahan:**
    File PDF materi dan panduan visual dapat ditemukan di dalam folder [`docs`](./docs).

  * **Wayground (Materi Interaktif & Kuis):**
    Kunjungi tautan berikut untuk mengakses slide presentasi (PPT) dan mengerjakan kuis latihan terkait materi ini:
    [Akses Wayground - Fundamental VM & Linux](https://wayground.com/join?gc=65948550)

-----

<br>

*Dibuat untuk keperluan Praktikum Mata Kuliah **Oracle Database Administrator** di Fakultas Teknologi Informasi, Universitas Tarumanagara.*