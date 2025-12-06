# TM01 - Instalasi Oracle VirtualBox

Dokumen ini memandu Anda dalam proses instalasi Oracle VM VirtualBox versi **7.2.0**. VirtualBox dibutuhkan sebagai lingkungan virtualisasi untuk menjalankan database Oracle.

## Daftar Isi
- [Prasyarat Sistem](#prasyarat-sistem)
- [Download Installer](#1-download-installer)
- [Panduan Instalasi](#2-panduan-instalasi)
- [Troubleshooting](#3-troubleshooting)


## Prasyarat Sistem
Sebelum memulai, pastikan laptop Anda memenuhi kriteria berikut untuk menghindari kegagalan instalasi:

1.  **Microsoft Visual C++ Redistributable (Wajib untuk Windows):**
    * Oracle VirtualBox membutuhkan *Microsoft Visual C++ 2019* ke atas.
    * Jika belum terinstall, unduh versi **x64** (untuk laptop 64-bit umum) di: [Microsoft Learn - Latest Supported VC Redist](https://learn.microsoft.com/id-id/cpp/windows/latest-supported-vc-redist?view=msvc-170).
    * **Penting:** Lakukan **Restart Laptop** setelah menginstal Visual C++ sebelum lanjut menginstal VirtualBox.
2.  **Ruang Penyimpanan:**
    * Dibutuhkan minimal **230MB** ruang kosong di hard drive untuk base aplikasi.

## 1. Download Installer
Unduh installer resmi sesuai sistem operasi Anda:

* **Website Resmi:** [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) 
* **Pilihan Platform:**
    * `Windows hosts`: Untuk pengguna Windows.
    * `macOS / Intel hosts`: Untuk MacBook dengan prosesor Intel.
    * `macOS / Apple Silicon hosts`: Untuk MacBook dengan chip M1/M2/M3.


## 2. Panduan Instalasi
Ikuti langkah-langkah berikut setelah file installer berhasil diunduh:

1.  **Jalankan Installer:** Buka file `Oracle VirtualBox 7.2.0 Installer`.
2.  **Setup Wizard:** Pada layar *Welcome*, klik **Next**.
3.  **License Agreement:**
    * Pilih opsi **"I accept the terms of the license agreement"**.
    * Klik **Next**.
4.  **Custom Setup:**
    * Biarkan fitur terpilih secara *default* (termasuk USB Support & Networking).
    * Klik **Next** (kecuali Anda ingin mengubah lokasi instalasi via tombol *Change*).
5.  **Konfirmasi Peringatan Network:** (Jika muncul peringatan *Network Interfaces*, klik **Yes** untuk melanjutkan).
6.  **Ready to Install:** Klik tombol **Install**.
7.  **Proses:** Tunggu hingga status bar penuh dan muncul pesan "Oracle VirtualBox 7.2.0 has been installed".
8.  **Selesai:** Centang "Start Oracle VirtualBox" lalu klik **Finish**.

## 3. Troubleshooting
Jika Anda mengalami kendala saat instalasi:

| Masalah | Solusi |
| :--- | :--- |
| **Error: Visual C++ Missing** | Muncul pesan *"Needs Microsoft Visual C++ 2019..."*.<br> **Solusi:** Lihat bagian [Prasyarat Sistem](#-prasyarat-sistem) di atas. Unduh Visual C++, install, lalu **wajib Restart laptop**. |
| **Instalasi Gagal / Rollback** | Cek koneksi internet saat mengunduh atau pastikan antivirus tidak memblokir installer. |

---
<br>

*Dibuat untuk keperluan Praktikum Mata Kuliah **Oracle Database Administrator** di Fakultas Teknologi Informasi Universitas Tarumanagara.*