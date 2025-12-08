# TM03 - Instalasi Oracle Database 23ai

Dokumen ini berisi panduan teknis untuk melakukan instalasi **Oracle Database 23ai (Free Edition)** di atas sistem operasi Oracle Linux 8.8 yang telah disiapkan pada pertemuan sebelumnya.

## Daftar Isi

  - [Persiapan Awal](#persiapan-awal)
  - [1. Instalasi Dependensi](#1-instalasi-dependensi)
  - [2. Download dan Instalasi Database](#2-download-dan-instalasi-database)
  - [3. Konfigurasi Database](#3-konfigurasi-database)
  - [4. Konfigurasi Environment](#4-konfigurasi-environment)
  - [5. Finalisasi dan Pengujian](#5-finalisasi-dan-pengujian)

## Persiapan Awal

Sebelum memulai proses instalasi, lakukan langkah-langkah pengamanan berikut pada VirtualBox:

1.  **Snapshot:**
      * Buka Oracle VirtualBox Manager.
      * Pilih VM **Oracle 8.8** (pastikan posisi *Powered Off*).
      * Buat Snapshot baru dengan mengklik **Take**.
      * Beri nama: `Instalasi Oracle 23ai`.
      * Hal ini bertujuan untuk menyimpan titik pemulihan jika terjadi kesalahan saat instalasi.
2.  **Start VM:**
      * Jalankan Virtual Machine.
      * Pastikan koneksi internet pada VM berstatus **Connected** (Wired) agar bisa mengunduh paket.

## 1. Instalasi Dependensi

Langkah ini bertujuan menyiapkan paket-paket pendukung yang dibutuhkan Oracle Database.

1.  Buka **Activities** > **Terminal**.
2.  Masuk sebagai **Super User (Root)**:
    ```bash
    su
    ```
    (Masukkan password root yang telah dibuat sebelumnya).
3.  Jalankan perintah instalasi *pre-install package*:
    ```bash
    dnf -y install oracle-database-preinstall-23ai
    ```
4.  Tunggu hingga proses selesai (*Complete*).

## 2. Download dan Instalasi Database

1.  **Download Installer:**
      * Buka browser Firefox di dalam VM.
      * Kunjungi tautan: `https://www.oracle.com/database/free/get-started/`.
      * Pilih **Oracle Linux 8 (OL8)**.
      * Klik link download pada file: `oracle-database-free-23ai-23.9-1.el8.x86_64.rpm`.
2.  **Instalasi RPM:**
      * Kembali ke Terminal (sebagai root).
      * Pindah ke direktori Downloads:
        ```bash
        cd /home/ocean/Downloads
        ```
      * Jalankan perintah instalasi:
        ```bash
        dnf install -y oracle-database-free*
        ```

## 3. Konfigurasi Database

Setelah paket terinstal, lakukan konfigurasi awal database untuk mengatur password dan listener.

1.  Jalankan skrip konfigurasi:
    ```bash
    /etc/init.d/oracle-free-23ai configure
    ```
2.  **Set Password:**
      * Sistem akan meminta password untuk akun administratif (`SYS`, `SYSTEM`, `PDBADMIN`).
      * Masukkan password yang memenuhi syarat (minimal 8 karakter, mengandung huruf besar, huruf kecil, dan angka).
3.  Tunggu proses konfigurasi hingga muncul pesan:
      * `Database creation complete.`
      * `100% complete.`

## 4. Konfigurasi Environment

Langkah ini diperlukan agar perintah Oracle dapat dikenali oleh sistem secara global.

1.  Edit file profile sistem menggunakan teks editor `vi`:
    ```bash
    vi /etc/profile
    ```
2.  Tekan tombol `i` pada keyboard untuk masuk ke mode **Insert**.
3.  Geser ke baris paling bawah, lalu tambahkan konfigurasi berikut:
    ```bash
    export ORACLE_SID=FREE
    export ORACLE_PDB_SID=FREEPDB1
    export ORACLE_ASK=NO
    . /opt/oracle/product/23ai/dbhomeFree/bin/oraenvi
    ```
4.  Simpan dan keluar dengan menekan tombol `Esc`, lalu ketik `:wq` dan tekan **Enter**.

## 5. Finalisasi dan Pengujian

### Mengaktifkan Service dan Ganti Password User Oracle

1.  Aktifkan service agar database menyala otomatis saat booting:
    ```bash
    /usr/lib/systemd/systemd-sysv-install enable oracle-free-23ai
    ```
2.  Ganti password untuk user `oracle` (user ini otomatis dibuat saat instalasi):
    ```bash
    passwd oracle
    ```
3.  Tutup terminal dan **Restart** Virtual Machine.

### Pengujian Koneksi (Login sebagai User Oracle)

**Penting:** Setelah restart, jangan login sebagai user biasa (contoh: `ocean`). Anda harus login sebagai user `oracle`.

1.  **Login ke Desktop:**
      * **Opsi A (Default):** Jika user `oracle` sudah muncul di layar, klik user tersebut.
      * **Opsi B (Jika Tidak Muncul):** Klik **Not listed?**, lalu masukkan username: `oracle`.
2.  Masukkan password yang baru saja Anda buat di langkah sebelumnya (saat perintah `passwd oracle`).
3.  Lakukan konfigurasi awal (*Initial Setup*) dengan menekan **Next** hingga selesai.
4.  Buka **Terminal**, lalu ketik:
    ```bash
    oraenv
    ```
    (Jika muncul prompt `ORACLE_SID = [FREE] ?`, tekan **Enter**).
5.  Masuk ke SQL\*Plus:
    ```bash
    sqlplus sys as sysdba
    ```
    (Masukkan password database `SYS` yang dibuat saat instalasi/konfigurasi di Tahap 3).
6.  Jika berhasil masuk ke `SQL>`, coba jalankan perintah dasar:
      * `startup` (untuk menyalakan database).
      * `show con_name` (untuk melihat container aktif).
      * `shutdown` (untuk mematikan database).

## Sumber Daya Tambahan

Untuk visualisasi lanjutan, Anda dapat menemukan file PDF di folder [`docs`](./docs).

-----

<br>

*Dibuat untuk keperluan Praktikum Mata Kuliah **Oracle Database Administrator** di Fakultas Teknologi Informasi, Universitas Tarumanagara.*