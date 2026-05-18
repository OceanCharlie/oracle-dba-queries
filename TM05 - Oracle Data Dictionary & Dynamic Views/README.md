# TM05 - Oracle Data Dictionary & Dynamic Performance Views

This document covers the fundamentals of the **Oracle Data Dictionary** and **Dynamic Performance Views (V$)**, which are essential tools for any DBA to monitor and manage an Oracle Database environment.

## Table of Contents

- [Learning Objectives](#learning-objectives)
- [Connecting to Oracle Database](#0-connecting-to-oracle-database)
- [1. Oracle Data Dictionary Concept](#1-oracle-data-dictionary-concept)
- [2. Common Users vs Local Users](#2-common-users-vs-local-users)
- [3. CDB\_ vs DBA\_ Views in Different Containers](#3-cdb_-vs-dba_-views-in-different-containers)
- [4. Querying Data Files](#4-querying-data-files)
- [5. Querying Temp Files](#5-querying-temp-files)
- [6. V$DATABASE, V$VERSION & V$INSTANCE](#6-vdatabase-vversion--vinstance)
- [7. V$CONTAINERS & CDB\_PDBS](#7-vcontainers--cdb_pdbs)
- [8. V$DATAFILE](#8-vdatafile)
- [9. Tablespace Views](#9-tablespace-views)
- [10. V$LOGFILE & V$CONTROLFILE](#10-vlogfile--vcontrolfile)
- [11. Pluggable Database Save State](#11-pluggable-database-save-state)

## Learning Objectives

After completing this module, students are expected to be able to:

* Connect to Oracle Database using SQL\*Plus as SYSDBA.
* Understand the Oracle Data Dictionary structure (CDB\_, DBA\_, ALL\_ views).
* Differentiate between Common Users and Local Users in a CDB architecture.
* Use Dynamic Performance Views (V$) to query database component information.
* Query data files, temp files, tablespaces, redo logs, and control files.
* Manage the state persistence of a Pluggable Database (PDB).

---

## 0. Connecting to Oracle Database

Before running any queries, you must connect to the database via SQL\*Plus from the Oracle Linux terminal.

> **Note:** Make sure you are logged in as the `oracle` user (as configured in TM03).

```bash
sqlplus
```

When prompted, enter the following credentials:

```
Enter user-name: sys as sysdba
Enter password: <your_SYS_password>
```

Once connected, you will see the `SQL>` prompt, indicating you are inside the **CDB$ROOT** container.

---

## 1. Oracle Data Dictionary Concept

The Oracle Data Dictionary is a collection of database tables and views that store **metadata** about every object in the database. These views come in three main prefixes:

| Prefix | Scope | Description |
| :--- | :--- | :--- |
| **CDB\_** | Entire CDB | Shows data from all containers (CDB + all PDBs). Only visible from CDB$ROOT. |
| **DBA\_** | Current Container | Shows all objects in the currently connected container. |
| **ALL\_** | Current Container | Shows objects the current user has access to. |
| **USER\_** | Current Container | Shows objects owned by the current user. |

### Checking the Current Container

```sql
SHOW CON_NAME;
```

### Opening a Pluggable Database

```sql
-- Make sure the PDB is open before querying CDB_ views
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Querying Tables Across All Containers

```sql
-- View tables from all containers (CDB + PDBs)
SELECT owner, table_name, con_id FROM cdb_tables ORDER BY 1, 2;
```

> **Key:** The `CON_ID` column identifies which container the data belongs to.  
> `CON_ID = 1` → CDB$ROOT | `CON_ID > 1` → PDB

### Formatting Output Columns

```sql
COL owner FORMAT a30;
COL table_name FORMAT a30;
```

### Observing the Effect of Closing a PDB

```sql
-- Close the PDB
ALTER PLUGGABLE DATABASE FREEPDB1 CLOSE;

-- Query again — PDB data disappears
SELECT owner, table_name, con_id FROM cdb_tables ORDER BY 1, 2;

-- Re-open the PDB
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Counting Tables Per Container

```sql
-- Count of tables grouped by container
SELECT con_id, COUNT(table_name) FROM cdb_tables GROUP BY con_id;

-- Count of tables visible from the current container using DBA_ view
SELECT COUNT(table_name) FROM dba_tables;
```

### Switching Between Containers

```sql
-- Switch to PDB
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

SELECT COUNT(table_name) FROM dba_tables;

-- Switch back to CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

SELECT COUNT(table_name) FROM dba_tables;
```

> **Observation:** The row count changes depending on which container you are connected to.

---

## 2. Common Users vs Local Users

In a **Multitenant** (CDB) architecture, there are two types of users:

| Type | Prefix | Exists In | Description |
| :--- | :--- | :--- | :--- |
| **Common User** | `C##` | CDB$ROOT + all PDBs | Created in CDB$ROOT, accessible across all containers. |
| **Local User** | *(none)* | One PDB only | Created inside a specific PDB, only exists there. |

### Checking PDB Status

```sql
-- Check which PDBs exist and their status
SELECT con_id, name, open_mode FROM v$pdbs;

-- Open all PDBs if needed
ALTER PLUGGABLE DATABASE ALL OPEN;
```

### Querying All Users (Common + Local)

```sql
COL username FORMAT a30;

-- View all users with their type and container
SELECT username, common, con_id FROM cdb_users ORDER BY username;

-- View only Common Users
SELECT DISTINCT(username) FROM cdb_users WHERE common = 'YES';

-- View only Local Users
SELECT username, common, con_id FROM cdb_users WHERE common = 'NO' ORDER BY username;
```

### Checking the Common User Prefix

```sql
SHOW PARAMETER common_user_prefix;
```

> The default prefix is `C##`. Any user created in CDB$ROOT without this prefix will fail.

### Creating Users

```sql
-- Attempt to create a local user in CDB$ROOT (will fail)
CREATE USER t1 IDENTIFIED BY t1;

-- Create a Common User in CDB$ROOT (must use C## prefix)
CREATE USER C##t1 IDENTIFIED BY welcome;
```

### Creating a Local User Inside a PDB

```sql
-- Switch to the PDB first
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

-- Create a local user (no C## prefix needed inside PDB)
CREATE USER charlie IDENTIFIED BY charlie;

-- Verify the local user was created
SELECT username, common, con_id FROM cdb_users WHERE common = 'NO' ORDER BY username;
```

---

## 3. CDB\_ vs DBA\_ Views in Different Containers

This section demonstrates how `CDB_` and `DBA_` views behave differently depending on which container you are connected to.

```sql
-- Start from CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

-- Count all tables across all open containers
SELECT COUNT(1) FROM cdb_tables;
-- Result: shows tables from CDB$ROOT + FREEPDB1

-- Now close the PDB
ALTER PLUGGABLE DATABASE FREEPDB1 CLOSE;

-- Count again — PDB data is excluded
SELECT COUNT(1) FROM cdb_tables;

-- Switch inside the PDB
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

-- Inside PDB, CDB_ views only show data from this PDB
SELECT COUNT(1) FROM cdb_tables;

-- Return to CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SELECT COUNT(1) FROM cdb_tables;
-- Result: higher number again (includes all containers)
```

> **Key Takeaway:** When inside a PDB, `CDB_` views are scoped to that PDB only. Full cross-container visibility is only available from `CDB$ROOT`.

---

## 4. Querying Data Files

Data files store the **actual data** in the database: tables, rows, indexes, procedures, views, and all user/application data. If data files are lost, the database is lost.

```sql
-- Make sure you are in CDB$ROOT
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

-- Check PDB status and open if needed
SELECT name, open_mode, con_id FROM v$pdbs;
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Querying with CDB\_ View (includes CON_ID)

```sql
COL file_name FORMAT a60;
COL tablespace_name FORMAT a20;

SELECT file_name, file_id, tablespace_name, con_id FROM cdb_data_files;
```

### Querying with DBA\_ View (current container only, no CON_ID)

```sql
SELECT file_name, file_id, tablespace_name FROM dba_data_files;
```

### Switching to PDB and Comparing Results

```sql
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

-- Inside PDB, DBA_ and CDB_ return the same result (scoped to this PDB)
SELECT file_name, file_id, tablespace_name FROM dba_data_files;
SELECT file_name, file_id, tablespace_name FROM cdb_data_files;
```

> **Note:** `ALL_DATA_FILES` does not exist — it will throw an error:  
> `ORA-00942: table or view does not exist`

---

## 5. Querying Temp Files

Temp files belong to **Temporary Tablespaces** and are used for special operations such as:
- Sorting large query results that do not fit in RAM (PGA)
- Hash join operations in SQL

Temp files do not store permanent data; they are used and cleared per session.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SELECT name, open_mode, con_id FROM v$pdbs;
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

### Querying Temp Files

```sql
-- CDB_ view (cross-container, includes CON_ID)
SELECT file_name, file_id, tablespace_name, con_id FROM cdb_temp_files;

-- DBA_ view (current container only)
SELECT file_name, file_id, tablespace_name FROM dba_temp_files;

-- Switch to PDB and compare
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

SELECT file_name, file_id, tablespace_name FROM dba_temp_files;
```

---

## 6. V$DATABASE, V$VERSION & V$INSTANCE

**Dynamic Performance Views (V$)** are special views that display real-time information about the running database instance. They are sourced from the **Control File** and instance memory — not the data dictionary.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;
```

### V$DATABASE — Database Information

```sql
-- Shows database name, CDB status, container ID, and open mode
SELECT name, cdb, con_id, open_mode FROM v$database;

-- Shows the DB_NAME parameter
SHOW PARAMETER db_name;
```

### V$VERSION — Oracle Version

```sql
-- Shows the Oracle Database version banner
SELECT banner FROM v$version;
```

### V$INSTANCE — Instance Information

```sql
-- Shows detailed information about the current running instance
SELECT * FROM v$instance;

-- Shows the instance name parameter
SHOW PARAMETER instance_name;
```

---

## 7. V$CONTAINERS & CDB\_PDBS

These views display information about all containers (CDB + PDBs) in the current instance.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;
```

### V$CONTAINERS — All Containers in the Instance

```sql
-- From CDB$ROOT: shows all containers
SELECT con_id, name, open_mode FROM v$containers;

-- From inside a PDB: only shows the current PDB
ALTER SESSION SET CONTAINER = FREEPDB1;
SELECT con_id, name, open_mode FROM v$containers;

ALTER SESSION SET CONTAINER = CDB$ROOT;
```

### CDB\_PDBS — PDB Status Information

```sql
COL pdb_name FORMAT a20;

SELECT pdb_id, pdb_name, status FROM cdb_pdbs;
```

**PDB Status Values:**

| Status | Description |
| :--- | :--- |
| `NEW` | PDB has never been opened since creation. |
| `NORMAL` | PDB is ready to use. |
| `UNPLUGGED` | PDB has been unplugged. Only `DROP PLUGGABLE DATABASE` is allowed. |
| `RELOCATING` | PDB is being relocated to a different CDB. |
| `RELOCATED` | PDB has been relocated to another CDB. |

```sql
-- Inside the PDB, use DBA_PDBS instead
ALTER SESSION SET CONTAINER = FREEPDB1;

SHOW CON_NAME;

SELECT pdb_id, pdb_name, status FROM dba_pdbs;
```

---

## 8. V$DATAFILE

`V$DATAFILE` is a Dynamic Performance View that shows data file information directly from the **Control File**. Unlike `CDB_DATA_FILES` (which requires the database to be open), some V$ views are available even when the PDB is not fully open.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SELECT name, open_mode, con_id FROM v$pdbs;
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;

-- Query from DBA_ view (requires PDB to be open)
SELECT file_name, file_id, tablespace_name, con_id FROM cdb_data_files;

-- Query from V$ view (available even when PDB is MOUNTED/CLOSED)
SELECT file#, name, ts#, con_id FROM v$datafile ORDER BY con_id;
```

### Observing the Difference When PDB is Closed

```sql
ALTER PLUGGABLE DATABASE FREEPDB1 CLOSE;

-- V$DATAFILE still shows data (reads from control file)
SELECT file#, name, ts#, con_id FROM v$datafile ORDER BY con_id;

-- CDB_DATA_FILES does NOT show closed PDB data
SELECT file_name, file_id, tablespace_name, con_id FROM cdb_data_files;

-- Re-open PDB
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;
```

```sql
-- Inside PDB
ALTER SESSION SET CONTAINER = FREEPDB1;

SELECT file#, name, ts#, con_id FROM v$datafile ORDER BY con_id;
SELECT file_name, file_id, tablespace_name FROM dba_data_files;

-- Quick PDB status check
SHOW PDBS;
```

---

## 9. Tablespace Views

Tablespaces are logical storage containers in Oracle. Each tablespace maps to one or more data files.

**Default Tablespaces in Oracle Database:**

| Tablespace | Purpose |
| :--- | :--- |
| `SYSTEM` | Stores the data dictionary |
| `SYSAUX` | Auxiliary support for SYSTEM |
| `UNDO` | Stores undo data for rollback |
| `TEMP` | Temporary operations (sorting, hash joins) |
| `USERS` | Default tablespace for user objects |

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;

-- Cross-container tablespace view
SELECT tablespace_name, block_size, status, contents, con_id FROM cdb_tablespaces;

-- Dynamic performance view for tablespaces
SELECT * FROM v$tablespace;

-- Switch to PDB and compare
ALTER SESSION SET CONTAINER = FREEPDB1;

SELECT tablespace_name, block_size, status, contents, con_id FROM cdb_tablespaces;
SELECT tablespace_name, block_size, status, contents, con_id FROM dba_tablespaces;

SELECT * FROM v$tablespace;
```

---

## 10. V$LOGFILE & V$CONTROLFILE

These two file types are **instance-level** — they exist for the entire CDB, not for individual PDBs.

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SHOW CON_NAME;

ALTER PLUGGABLE DATABASE ALL OPEN;

SELECT con_id, name, open_mode FROM v$containers;
```

### V$LOGFILE — Redo Log Files

Redo log files record every change made to the database to protect against data loss after an instance failure. Oracle requires a minimum of **2 redo log groups** — one is always available for writing while the other is being archived.

```sql
COL member FORMAT a50;

SELECT * FROM v$logfile;
```

> **Important:** Redo log files belong to the entire instance, not to a specific container.

### V$CONTROLFILE — Control Files

Control files store metadata about data files and redo log files (names, locations, statuses). They are required to open the database. Loss of all control files means loss of the database.

```sql
SELECT * FROM v$controlfile;
```

> **Important:** Control files also exist at the instance level, not per container.

---

## 11. Pluggable Database Save State

By default, when the CDB is restarted, all PDBs return to a **MOUNTED** state (not open). The `SAVE STATE` command instructs Oracle to automatically re-open the PDB after a CDB restart.

### Demonstrating the Problem (Without Save State)

```sql
COLUMN name FORMAT a30;

-- Check current PDB state
SELECT name, open_mode FROM v$pdbs;

-- Restart the database
SHUTDOWN IMMEDIATE;
STARTUP;

-- After restart, PDB is in MOUNTED state (not open)
SELECT name, open_mode FROM v$pdbs;
```

### Applying Save State

```sql
-- Open the PDB manually
ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;

-- Save the open state so it persists after restart
ALTER PLUGGABLE DATABASE FREEPDB1 SAVE STATE;

-- Restart to verify
SHUTDOWN IMMEDIATE;
STARTUP;

-- PDB is now automatically OPEN after restart
SELECT name, open_mode FROM v$pdbs;
```

> **Best Practice:** Always run `SAVE STATE` after opening a PDB in a production environment to ensure it reopens automatically after planned or unplanned CDB restarts.

---

<br>

*Created for the **Oracle Database Administrator** Course Practicum at the Faculty of Information Technology, Tarumanagara University.*
