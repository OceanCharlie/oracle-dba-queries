# TM02 - Instalasi Oracle Linux 8.8

Dokumen ini berisi panduan langkah demi langkah untuk menginstal sistem operasi **Oracle Linux 8.8** pada Oracle VirtualBox, serta konfigurasi pasca-instalasi yang diperlukan untuk lingkungan praktikum.

## Daftar Isi
- [Persiapan dan Download](#persiapan-dan-download)
- [1. Konfigurasi Virtual Machine](#1-konfigurasi-virtual-machine)
- [2. Proses Instalasi Oracle Linux](#2-proses-instalasi-oracle-linux)
- [3. Konfigurasi Pasca Instalasi](#3-konfigurasi-pasca-instalasi)
- [Troubleshooting](#troubleshooting)

## Persiapan dan Download
Sebelum memulai, unduh file ISO Oracle Linux versi 8.8 melalui tautan resmi berikut:

* **URL Download:** https://yum.oracle.com/oracle-linux-isos.html
* **Versi yang dipilih:** Oracle Linux 8.8 (x86_64) - **Full ISO**.
    * *Catatan:* Jika menggunakan Mac dengan chip Apple Silicon (M1/M2/M3), pilih versi ARM.

## 1. Konfigurasi Virtual Machine
Ikuti langkah berikut untuk membuat mesin virtual baru di VirtualBox:

1.  **Buat VM Baru:**
    * Klik **New** pada VirtualBox Manager.
    * **Name:** `Oracle 8.8`
    * **ISO Image:** Pilih file ISO Oracle Linux 8.8 yang telah diunduh.
    * **Type:** Linux
    * **Version:** Oracle Linux (64-bit)
2.  **Hardware (Disarankan):**
    * **Base Memory (RAM):** 4096 MB (4 GB).
    * **Processors (CPU):** Minimal 1 CPU (disarankan disesuaikan dengan spesifikasi laptop).
3.  **Hard Disk:**
    * **Size:** 40.00 GB.
    * Pilih **Create a Virtual Hard Disk Now**.
4.  **Snapshot Awal (Penting):**
    * Sebelum menjalankan VM, buat Snapshot untuk menyimpan *state* awal.
    * Klik menu burger (sebelah kanan tombol Start) > **Snapshots** > **Take**.
    * Beri nama: `Snapshot 1`.

## 2. Proses Instalasi Oracle Linux
Jalankan VM (Start) dan ikuti langkah instalasi berikut:

1.  **Boot Menu:** Pilih **Install Oracle Linux 8.8.0**.
2.  **Bahasa:** Pilih **English (United States)** > Continue.
3.  **Installation Summary:**
    Atur konfigurasi berikut pada menu utama:
    * **Time & Date:** Pilih Region `Asia` dan City `Jakarta`.
    * **Software Selection:** Pilih **Server with GUI** (Pastikan ini terpilih agar memiliki tampilan desktop).
    * **Installation Destination:**
        * Pilih `ATA VBOX HARDDISK` (40 GiB).
        * Storage Configuration: **Automatic**.
        * Klik **Done**.
    * **Root Password:**
        * Set password menjadi: `oracle` (Disarankan agar seragam).
        * Klik **Done** dua kali untuk konfirmasi (karena password dianggap *weak*).
    * **User Creation:**
        * Buat user baru (contoh: `ocean`).
        * Centang "Require a password to use this account".
        * Isi password dan konfirmasi.
4.  **Memulai Instalasi:**
    * Klik **Begin Installation**.
    * Tunggu proses selesai, lalu klik **Reboot System**.
5.  **Licensing (Initial Setup):**
    * Setelah restart, akan muncul menu *Initial Setup*.
    * Klik **License Information**.
    * Centang **I accept the license agreement** > Done.
    * Klik **Finish Configuration**.

## 3. Konfigurasi Pasca Instalasi
Setelah berhasil login ke desktop Oracle Linux, lakukan langkah berikut untuk mengaktifkan fitur layar penuh dan *copy-paste*.

### A. Koneksi Internet
Pastikan VM terhubung ke internet:
1.  Klik ikon status di pojok kanan atas.
2.  Klik **Wired Off** > **Connect**.

### B. Instalasi Dependensi (Terminal)
1.  Buka **Activities** (pojok kiri atas) > cari **Terminal**.
2.  Masuk sebagai Super User (Root):
    ```bash
    su
    ```
    (Masukkan password root: `oracle`)
3.  Jalankan perintah berikut untuk menginstal paket kernel dan development tools:
    ```bash
    dnf install kernel-uek-devel-$(uname -r) gcc binutils automake make perl bzip2 elfutils-libelf-devel
    ```
4.  Ketik `y` jika diminta konfirmasi. Tunggu hingga proses *Complete!*.

### C. Install Guest Additions
Fitur ini diperlukan agar bisa *copy-paste* antara Windows/Mac dan VirtualBox.
1.  Pada menu bar VirtualBox (di atas jendela VM), pilih **Devices** > **Insert Guest Additions CD image...**.
2.  Akan muncul *pop-up* di dalam Oracle Linux. Klik **Run**.
3.  Masukkan password user jika diminta, lalu klik **Authenticate**.
4.  Tunggu terminal memproses hingga selesai (tekan Enter jika diminta menutup window).
5.  **Restart VM:** Klik ikon kanan atas > Power Off/Log Out > **Restart**.

### D. Mengaktifkan Shared Clipboard
1.  Setelah VM menyala kembali, pilih menu **Devices** > **Shared Clipboard**.
2.  Pilih opsi **Bidirectional**.
3.  **Pengujian:** Coba salin teks dari Notepad di Windows dan *paste* ke Terminal Linux (atau sebaliknya).
4.  **Snapshot Akhir:** Matikan VM, lalu buat Snapshot baru dengan nama `Setelah instalasi Oracle 8.8`.

## Troubleshooting

### Masalah 1: Guest Additions Tidak Muncul Otomatis
Jika setelah klik "Insert Guest Additions" tidak ada reaksi:
1.  Buka menu **Activities** > **Files**.
2.  Lihat di *sidebar* kiri, cari disk bernama **VBox_GAs_...**.
3.  Klik tombol **Run Software** di pojok kanan atas jendela Files.
4.  Klik **Run** pada dialog konfirmasi.

### Masalah 2: Tombol Host Key (Right Ctrl) Tidak Berfungsi
Jika laptop Anda tidak memiliki tombol "Right Ctrl" (Ctrl Kanan) untuk melepaskan kursor mouse dari VM:
1.  Pada VirtualBox Manager utama, tekan `Ctrl + G` (Preferences).
2.  Pilih menu **Input** > tab **Virtual Machine**.
3.  Cari baris **Host Key Combo**.
4.  Klik pada kolom *Shortcut*, lalu tekan tombol pengganti yang diinginkan di keyboard Anda (contoh: tombol `Shift` Kanan atau `Alt` Kanan).
5.  Klik **OK**.

## Sumber Daya Tambahan
Untuk visualisasi lanjutan, Anda dapat menemukan file PDF di folder [`docs`](./docs).

---
<br>

*Dibuat untuk keperluan Praktikum Mata Kuliah **Oracle Database Administrator** di Fakultas Teknologi Informasi, Universitas Tarumanagara.*