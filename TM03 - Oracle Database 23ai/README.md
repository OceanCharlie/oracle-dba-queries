# TM03 - Oracle Database 23ai Installation

This document provides technical guidance for installing **Oracle Database 23ai (Free Edition)** on the Oracle Linux 8.8 operating system prepared in the previous session.

## Table of Contents

- [Initial Preparation](#initial-preparation)
- [1. Dependency Installation](#1-dependency-installation)
- [2. Database Download and Installation](#2-database-download-and-installation)
- [3. Database Configuration](#3-database-configuration)
- [4. Environment Configuration](#4-environment-configuration)
- [5. Finalization and Testing](#5-finalization-and-testing)

## Initial Preparation

Before starting the installation process, perform the following security steps on VirtualBox:

1.  **Snapshot:**
    *   Open Oracle VirtualBox Manager.
    *   Select the **Oracle 8.8** VM (ensure it is *Powered Off*).
    *   Create a new Snapshot by clicking **Take**.
    *   Name it: `Oracle 23ai Installation`.
    *   This aims to save a recovery point in case of errors during installation.
2.  **Start VM:**
    *   Run the Virtual Machine.
    *   Ensure the internet connection on the VM is **Connected** (Wired) to download packages.

## 1. Dependency Installation

This step aims to prepare the supporting packages required by the Oracle Database.

1.  Open **Activities** > **Terminal**.
2.  Login as **Super User (Root)**:
    ```bash
    su
    ```
    (Enter the root password created previously).
3.  Run the *pre-install package* installation command:
    ```bash
    dnf -y install oracle-database-preinstall-23ai
    ```
4.  Wait until the process is complete (*Complete*).

## 2. Database Download and Installation

1.  **Download Installer:**
    *   Open the Firefox browser inside the VM.
    *   Visit the link: `https://www.oracle.com/database/free/get-started/`.
    *   Select **Oracle Linux 8 (OL8)**.
    *   Click the download link on the file: `oracle-database-free-23ai-23.9-1.el8.x86_64.rpm`.
2.  **RPM Installation:**
    *   Return to the Terminal (as root).
    *   Move to the Downloads directory:
        ```bash
        cd /home/ocean/Downloads
        ```
    *   Run the installation command:
        ```bash
        dnf install -y oracle-database-free*
        ```

## 3. Database Configuration

After the package is installed, perform the initial database configuration to set the password and listener.

1.  Run the configuration script:
    ```bash
    /etc/init.d/oracle-free-23ai configure
    ```
2.  **Set Password:**
    *   The system will ask for a password for administrative accounts (`SYS`, `SYSTEM`, `PDBADMIN`).
    *   Enter a password that meets the requirements (minimum 8 characters, containing uppercase, lowercase, and numbers).
3.  Wait for the configuration process until the message appears:
    *   `Database creation complete.`
    *   `100% complete.`

## 4. Environment Configuration

This step is necessary so that Oracle commands can be recognized by the system globally.

1.  Edit the system profile file using the `vi` text editor:
    ```bash
    vi /etc/profile
    ```
2.  Press the `i` key on the keyboard to enter **Insert** mode.
3.  Scroll to the bottom line, then add the following configuration:
    ```bash
    export ORACLE_SID=FREE
    export ORACLE_PDB_SID=FREEPDB1
    export ORACLE_ASK=NO
    . /opt/oracle/product/23ai/dbhomeFree/bin/oraenvi
    ```
4.  Save and exit by pressing the `Esc` key, then type `:wq` and press **Enter**.

## 5. Finalization and Testing

### Enable Service and Change Oracle User Password

1.  Enable the service so the database starts automatically at boot:
    ```bash
    /usr/lib/systemd/systemd-sysv-install enable oracle-free-23ai
    ```
2.  Change the password for the `oracle` user (this user is automatically created during installation):
    ```bash
    passwd oracle
    ```
3.  Close the terminal and **Restart** the Virtual Machine.

### Connection Testing (Login as Oracle User)

**Important:** After restarting, do not login as a regular user (example: `ocean`). You must login as the `oracle` user.

1.  **Login to Desktop:**
    *   **Option A (Default):** If the `oracle` user already appears on the screen, click that user.
    *   **Option B (If Not Listed):** Click **Not listed?**, then enter username: `oracle`.
2.  Enter the password you just created in the previous step (during the `passwd oracle` command).
3.  Perform the *Initial Setup* configuration by clicking **Next** until finished.
4.  Open **Terminal**, then type:
    ```bash
    oraenv
    ```
    (If the prompt `ORACLE_SID = [FREE] ?` appears, press **Enter**).
5.  Enter SQL\*Plus:
    ```bash
    sqlplus sys as sysdba
    ```
    (Enter the `SYS` database password created during installation/configuration in Stage 3).
6.  If successfully entered `SQL>`, try running basic commands:
    *   `startup` (to start the database).
    *   `show con_name` (to see the active container).
    *   `shutdown` (to shut down the database).

## Additional Resources

For advanced visualization, you can find PDF files in the [`docs`](./docs) folder.

-----

<br>

*Created for the **Oracle Database Administrator** Course Lab Session at the Faculty of Information Technology, Tarumanagara University.*