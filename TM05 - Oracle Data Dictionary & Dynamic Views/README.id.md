# TM05 - Oracle Data Dictionary & Dynamic Performance Views

Dokumen ini membahas dasar-dasar **Oracle Data Dictionary** dan **Dynamic Performance Views (V$)**, yang merupakan alat penting bagi setiap DBA untuk memantau dan mengelola lingkungan Oracle Database.

## Daftar Isi

- [Tujuan Pembelajaran](#tujuan-pembelajaran)
- [Koneksi ke Oracle Database](#0-koneksi-ke-oracle-database)
- [1. Konsep Oracle Data Dictionary](#1-konsep-oracle-data-dictionary)
- [2. Common Users vs Local Users](#2-common-users-vs-local-users)
- [3. CDB\_ vs DBA\_ Views di Kontainer Berbeda](#3-cdb_-vs-dba_-views-di-kontainer-berbeda)
- [4. Query Data Files](#4-query-data-files)
- [5. Query Temp Files](#5-query-temp-files)
- [6. V$DATABASE, V$VERSION & V$INSTANCE](#6-vdatabase-vversion--vinstance)
- [7. V$CONTAINERS & CDB\_PDBS](#7-vcontainers--cdb_pdbs)
- [8. V$DATAFILE](#8-vdatafile)
- [9. Views Tablespace](#9-views-tablespace)
- [10. V$LOGFILE & V$CONTROLFILE](#10-vlogfile--vcontrolfile)
- [11. Pluggable Database Save State](#11-pluggable-database-save-state)

## Tujuan Pembelajaran

Setelah menyelesaikan modul ini, mahasiswa diharapkan mampu:

* Terhubung ke Oracle Database menggunakan SQL\*Plus sebagai SYSDBA.
* Memahami struktur Oracle Data Dictionary (views CDB\_, DBA\_, ALL\_).
* Membedakan antara Common Users dan Local Users dalam arsitektur CDB.
* Menggunakan Dynamic Performance Views (V$) untuk melihat informasi komponen database.
* Melakukan query pada data files, temp files, tablespace, redo logs, dan control files.
* Mengelola persistensi status Pluggable Database (PDB).

---

## 0. Koneksi ke Oracle Database

Sebelum menjalankan query apapun, Anda harus terhubung ke database melalui SQL\*Plus dari terminal Oracle Linux.

> **Catatan:** Pastikan Anda sudah login sebagai user `oracle` (sesuai konfigurasi di TM03).

```bash
sqlplus
```

Saat diminta, masukkan kredensial berikut:

```
Enter user-name: sys as sysdba
Enter password: <password_SYS_Anda>
```

Setelah terhubung, Anda akan melihat prompt `SQL>`, yang menandakan Anda berada di dalam kontainer **CDB$ROOT**.

---

## 1. Konsep Oracle Data Dictionary

Oracle Data Dictionary adalah kumpulan tabel dan view database yang menyimpan **metadata** tentang setiap objek di dalam database. View-view ini memiliki tiga awalan utama:

| Awalan | Cakupan | Keterangan |
| :--- | :--- | :--- |
| **CDB\_** | Seluruh CDB | Menampilkan data dari semua kontainer (CDB + semua PDB). Hanya terlihat dari CDB$ROOT. |
| **DBA\_** | Kontainer Saat Ini | Menampilkan semua objek di kontainer yang sedang digunakan. |
| **ALL\_** | Kontainer Saat Ini | Menampilkan objek yang dapat diakses oleh user saat ini. |
| **USER\_** | Kontainer Saat Ini | Menampilkan objek yang dimiliki oleh user saat ini. |

### Memeriksa Kontainer Saat Ini

```sql
SHOW CON_NAME;
```

### Membuka Pluggable Database

```sql
-- Pastikan PDB sudah terbuka sebelum query CDB_ views
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Query Tabel dari Semua Kontainer

```sql
-- Lihat tabel dari semua kontainer (CDB + PDB)
SELECT owner, table_name, con_id FROM cdb_tables ORDER BY 1, 2;
```

> **Kunci:** Kolom `CON_ID` menunjukkan kontainer mana data tersebut berasal.  
> `CON_ID = 1` → CDB$ROOT | `CON_ID > 1` → PDB

### Format Output Kolom

```sql
COL owner FORMAT a30;
COL table_name FORMAT a30;
```

### Mengamati Efek Penutupan PDB

```sql
-- Tutup PDB
ALTER PLUGGABLE DATABASE FREEPDB1 CLOSE;

-- Query ulang — data PDB menghilang
SELECT owner, table_name, con_id FROM cdb_tables ORDER BY 1, 2;

-- Buka kembali PDB
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Menghitung Tabel Per Kontainer

```sql
-- Jumlah tabel per kontainer
SELECT con_id, COUNT(table_name) FROM cdb_tables GROUP BY con_id;

-- Jumlah tabel dari kontainer saat ini menggunakan DBA_ view
SELECT COUNT(table_name) FROM dba_tables;
```

### Berpindah Antar Kontainer

```sql
-- Pindah ke PDB
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

SELECT COUNT(table_name) FROM dba_tables;

-- Kembali ke CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

SELECT COUNT(table_name) FROM dba_tables;
```

> **Pengamatan:** Jumlah baris berubah tergantung kontainer yang sedang digunakan.

---

## 2. Common Users vs Local Users

Dalam arsitektur **Multitenant** (CDB), terdapat dua jenis user:

| Tipe | Awalan | Tersedia Di | Keterangan |
| :--- | :--- | :--- | :--- |
| **Common User** | `C##` | CDB$ROOT + semua PDB | Dibuat di CDB$ROOT, dapat diakses di semua kontainer. |
| **Local User** | *(tidak ada)* | Satu PDB saja | Dibuat di dalam PDB tertentu, hanya ada di sana. |

### Memeriksa Status PDB

```sql
-- Periksa PDB yang ada beserta statusnya
SELECT con_id, name, open_mode FROM v$pdbs;

-- Buka semua PDB jika diperlukan
ALTER PLUGGABLE DATABASE ALL OPEN;
```

### Query Semua User (Common + Local)

```sql
COL username FORMAT a30;

-- Lihat semua user beserta tipe dan kontainernya
SELECT username, common, con_id FROM cdb_users ORDER BY username;

-- Lihat hanya Common Users
SELECT DISTINCT(username) FROM cdb_users WHERE common = 'YES';

-- Lihat hanya Local Users
SELECT username, common, con_id FROM cdb_users WHERE common = 'NO' ORDER BY username;
```

### Memeriksa Awalan Common User

```sql
SHOW PARAMETER common_user_prefix;
```

> Awalan default adalah `C##`. User yang dibuat di CDB$ROOT tanpa awalan ini akan gagal.

### Membuat User

```sql
-- Membuat local user di CDB$ROOT (akan gagal)
CREATE USER t1 IDENTIFIED BY t1;

-- Membuat Common User di CDB$ROOT (harus menggunakan awalan C##)
CREATE USER C##t1 IDENTIFIED BY welcome;
```

### Membuat Local User di Dalam PDB

```sql
-- Pindah ke PDB terlebih dahulu
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

-- Buat local user (tidak perlu awalan C## di dalam PDB)
CREATE USER charlie IDENTIFIED BY charlie;

-- Verifikasi local user berhasil dibuat
SELECT username, common, con_id FROM cdb_users WHERE common = 'NO' ORDER BY username;
```

---

## 3. CDB\_ vs DBA\_ Views di Kontainer Berbeda

Bagian ini menunjukkan bagaimana views `CDB_` dan `DBA_` berperilaku berbeda tergantung kontainer yang digunakan.

```sql
-- Mulai dari CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

-- Hitung semua tabel di semua kontainer yang terbuka
SELECT COUNT(1) FROM cdb_tables;
-- Hasil: menampilkan tabel dari CDB$ROOT + FREEPDB1

-- Tutup PDB
ALTER PLUGGABLE DATABASE FREEPDB1 CLOSE;

-- Hitung lagi — data PDB tidak termasuk
SELECT COUNT(1) FROM cdb_tables;

-- Masuk ke dalam PDB
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

-- Di dalam PDB, views CDB_ hanya menampilkan data dari PDB ini
SELECT COUNT(1) FROM cdb_tables;

-- Kembali ke CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SELECT COUNT(1) FROM cdb_tables;
-- Hasil: angka lebih besar (mencakup semua kontainer)
```

> **Kesimpulan Utama:** Ketika berada di dalam PDB, views `CDB_` hanya menampilkan data dari PDB tersebut. Visibilitas lintas kontainer penuh hanya tersedia dari `CDB$ROOT`.

---

## 4. Query Data Files

Data files menyimpan **data sesungguhnya** di dalam database: tabel, baris, indeks, prosedur, view, dan semua data user/aplikasi. Jika data files hilang, database pun hilang.

```sql
-- Pastikan berada di CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

-- Periksa status PDB dan buka jika perlu
SELECT name, open_mode, con_id FROM v$pdbs;
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Query dengan CDB\_ View (termasuk CON_ID)

```sql
COL file_name FORMAT a60;
COL tablespace_name FORMAT a20;

SELECT file_name, file_id, tablespace_name, con_id FROM cdb_data_files;
```

### Query dengan DBA\_ View (hanya kontainer saat ini, tanpa CON_ID)

```sql
SELECT file_name, file_id, tablespace_name FROM dba_data_files;
```

### Pindah ke PDB dan Bandingkan Hasil

```sql
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

-- Di dalam PDB, DBA_ dan CDB_ menghasilkan data yang sama (terbatas pada PDB ini)
SELECT file_name, file_id, tablespace_name FROM dba_data_files;
SELECT file_name, file_id, tablespace_name FROM cdb_data_files;
```

> **Catatan:** `ALL_DATA_FILES` tidak ada — akan menghasilkan error:  
> `ORA-00942: table or view does not exist`

---

## 5. Query Temp Files

Temp files termasuk dalam **Temporary Tablespace** dan digunakan untuk operasi khusus seperti:
- Pengurutan hasil query besar yang tidak cukup di RAM (PGA)
- Operasi hash join dalam SQL

Temp files tidak menyimpan data permanen; digunakan dan dibersihkan per sesi.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SELECT name, open_mode, con_id FROM v$pdbs;
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Query Temp Files

```sql
-- CDB_ view (lintas kontainer, termasuk CON_ID)
SELECT file_name, file_id, tablespace_name, con_id FROM cdb_temp_files;

-- DBA_ view (hanya kontainer saat ini)
SELECT file_name, file_id, tablespace_name FROM dba_temp_files;

-- Pindah ke PDB dan bandingkan
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

SELECT file_name, file_id, tablespace_name FROM dba_temp_files;
```

---

## 6. V$DATABASE, V$VERSION & V$INSTANCE

**Dynamic Performance Views (V$)** adalah view khusus yang menampilkan informasi real-time tentang instance database yang sedang berjalan. Data bersumber dari **Control File** dan memori instance — bukan dari data dictionary.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;
```

### V$DATABASE — Informasi Database

```sql
-- Menampilkan nama database, status CDB, container ID, dan open mode
SELECT name, cdb, con_id, open_mode FROM v$database;

-- Menampilkan parameter DB_NAME
SHOW PARAMETER db_name;
```

### V$VERSION — Versi Oracle

```sql
-- Menampilkan banner versi Oracle Database
SELECT banner FROM v$version;
```

### V$INSTANCE — Informasi Instance

```sql
-- Menampilkan informasi detail tentang instance yang sedang berjalan
SELECT * FROM v$instance;

-- Menampilkan parameter nama instance
SHOW PARAMETER instance_name;
```

---

## 7. V$CONTAINERS & CDB\_PDBS

View-view ini menampilkan informasi tentang semua kontainer (CDB + PDB) dalam instance saat ini.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;
```

### V$CONTAINERS — Semua Kontainer dalam Instance

```sql
-- Dari CDB$ROOT: menampilkan semua kontainer
SELECT con_id, name, open_mode FROM v$containers;

-- Dari dalam PDB: hanya menampilkan PDB saat ini
ALTER SESSION SET CONTAINER = FREEPDB1;
SELECT con_id, name, open_mode FROM v$containers;

ALTER SESSION SET CONTAINER = CDB$ROOT;
```

### CDB\_PDBS — Informasi Status PDB

```sql
COL pdb_name FORMAT a20;

SELECT pdb_id, pdb_name, status FROM cdb_pdbs;
```

**Nilai Status PDB:**

| Status | Keterangan |
| :--- | :--- |
| `NEW` | PDB belum pernah dibuka sejak dibuat. |
| `NORMAL` | PDB siap digunakan. |
| `UNPLUGGED` | PDB telah di-unplug. Hanya `DROP PLUGGABLE DATABASE` yang diizinkan. |
| `RELOCATING` | PDB sedang dipindahkan ke CDB lain. |
| `RELOCATED` | PDB telah dipindahkan ke CDB lain. |

```sql
-- Di dalam PDB, gunakan DBA_PDBS
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

SELECT pdb_id, pdb_name, status FROM dba_pdbs;
```

---

## 8. V$DATAFILE

`V$DATAFILE` adalah Dynamic Performance View yang menampilkan informasi data file langsung dari **Control File**. Tidak seperti `CDB_DATA_FILES` (yang membutuhkan database terbuka), beberapa V$ views tersedia bahkan saat PDB tidak sepenuhnya terbuka.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SELECT name, open_mode, con_id FROM v$pdbs;
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;

-- Query dari DBA_ view (membutuhkan PDB terbuka)
SELECT file_name, file_id, tablespace_name, con_id FROM cdb_data_files;

-- Query dari V$ view (tersedia bahkan saat PDB MOUNTED/CLOSED)
SELECT file#, name, ts#, con_id FROM v$datafile ORDER BY con_id;
```

### Mengamati Perbedaan Saat PDB Ditutup

```sql
ALTER PLUGGABLE DATABASE FREEPDB1 CLOSE;

-- V$DATAFILE masih menampilkan data (membaca dari control file)
SELECT file#, name, ts#, con_id FROM v$datafile ORDER BY con_id;

-- CDB_DATA_FILES TIDAK menampilkan data PDB yang ditutup
SELECT file_name, file_id, tablespace_name, con_id FROM cdb_data_files;

-- Buka kembali PDB
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

```sql
-- Di dalam PDB
ALTER SESSION SET CONTAINER = FREEPDB1;

SELECT file#, name, ts#, con_id FROM v$datafile ORDER BY con_id;
SELECT file_name, file_id, tablespace_name FROM dba_data_files;

-- Cek status PDB secara cepat
SHOW PDBS;
```

---

## 9. Views Tablespace

Tablespace adalah wadah penyimpanan logis di Oracle. Setiap tablespace memetakan ke satu atau lebih data file.

**Tablespace Default di Oracle Database:**

| Tablespace | Fungsi |
| :--- | :--- |
| `SYSTEM` | Menyimpan data dictionary |
| `SYSAUX` | Dukungan tambahan untuk SYSTEM |
| `UNDO` | Menyimpan data undo untuk rollback |
| `TEMP` | Operasi sementara (pengurutan, hash join) |
| `USERS` | Tablespace default untuk objek user |

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;

-- View tablespace lintas kontainer
SELECT tablespace_name, block_size, status, contents, con_id FROM cdb_tablespaces;

-- Dynamic performance view untuk tablespace
SELECT * FROM v$tablespace;

-- Pindah ke PDB dan bandingkan
ALTER SESSION SET CONTAINER = FREEPDB1;

SELECT tablespace_name, block_size, status, contents, con_id FROM cdb_tablespaces;
SELECT tablespace_name, block_size, status, contents, con_id FROM dba_tablespaces;

SELECT * FROM v$tablespace;
```

---

## 10. V$LOGFILE & V$CONTROLFILE

Kedua jenis file ini bersifat **instance-level** — ada untuk seluruh CDB, bukan untuk PDB individual.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

ALTER PLUGGABLE DATABASE ALL OPEN;

SELECT con_id, name, open_mode FROM v$containers;
```

### V$LOGFILE — Redo Log Files

Redo log files mencatat setiap perubahan yang dilakukan pada database untuk melindungi dari kehilangan data setelah kegagalan instance. Oracle membutuhkan minimal **2 redo log group** — satu selalu tersedia untuk penulisan sementara yang lain sedang diarsipkan.

```sql
COL member FORMAT a50;

SELECT * FROM v$logfile;
```

> **Penting:** Redo log files milik seluruh instance, bukan kontainer tertentu.

### V$CONTROLFILE — Control Files

Control files menyimpan metadata tentang data files dan redo log files (nama, lokasi, status). Diperlukan untuk membuka database. Kehilangan semua control files berarti kehilangan database.

```sql
SELECT * FROM v$controlfile;
```

> **Penting:** Control files juga ada di level instance, bukan per kontainer.

---

## 11. Pluggable Database Save State

Secara default, saat CDB di-restart, semua PDB kembali ke status **MOUNTED** (tidak terbuka). Perintah `SAVE STATE` menginstruksikan Oracle untuk membuka kembali PDB secara otomatis setelah restart CDB.

### Demonstrasi Masalah (Tanpa Save State)

```sql
COLUMN name FORMAT a30;

-- Periksa status PDB saat ini
SELECT name, open_mode FROM v$pdbs;

-- Restart database
SHUTDOWN IMMEDIATE;
STARTUP;

-- Setelah restart, PDB berada dalam status MOUNTED (tidak terbuka)
SELECT name, open_mode FROM v$pdbs;
```

### Menerapkan Save State

```sql
-- Buka PDB secara manual
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;

-- Simpan status terbuka agar bertahan setelah restart
ALTER PLUGGABLE DATABASE FREEPDB1 SAVE STATE;

-- Restart untuk verifikasi
SHUTDOWN IMMEDIATE;
STARTUP;

-- PDB sekarang otomatis OPEN setelah restart
SELECT name, open_mode FROM v$pdbs;
```

> **Best Practice:** Selalu jalankan `SAVE STATE` setelah membuka PDB di lingkungan produksi untuk memastikan PDB terbuka kembali secara otomatis setelah restart CDB yang direncanakan maupun tidak direncanakan.

---

<br>

*Dibuat untuk Praktikum Mata Kuliah **Oracle Database Administrator** di Fakultas Teknologi Informasi, Universitas Tarumanagara.*