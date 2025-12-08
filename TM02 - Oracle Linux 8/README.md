# TM02 - Oracle Linux 8.8 Installation Guide

This document provides a step-by-step guide for installing **Oracle Linux 8.8** on Oracle VirtualBox, along with essential post-installation configurations required for the lab environment.

## Table of Contents

* [Preparation and Downloads](#preparation-and-downloads)
* [1. Virtual Machine Configuration](#1-virtual-machine-configuration)
* [2. Oracle Linux Installation Process](#2-oracle-linux-installation-process)
* [3. Post-Installation Configuration](#3-post-installation-configuration)
* [Troubleshooting](#troubleshooting)

## Preparation and Downloads

Before starting, download the Oracle Linux ISO file from the official Oracle site:

* **Download URL:** [https://yum.oracle.com/oracle-linux-isos.html](https://yum.oracle.com/oracle-linux-isos.html)
* **Version Selected:** Oracle Linux 8.8 (x86_64) — **Full ISO**

  * *Note:* For Mac devices with Apple Silicon (M1/M2/M3), select the ARM version.

## 1. Virtual Machine Configuration

Follow these steps to create a new virtual machine in VirtualBox:

1. **Create a New VM**

   * Click **New** in VirtualBox Manager.
   * **Name:** `Oracle 8.8`
   * **ISO Image:** Select the downloaded Oracle Linux 8.8 ISO.
   * **Type:** Linux
   * **Version:** Oracle Linux (64-bit)

2. **Hardware (Recommended)**

   * **Base Memory (RAM):** 4096 MB (4 GB)
   * **Processors (CPU):** Minimum 1 CPU (adjust to your laptop’s specs)

3. **Hard Disk**

   * **Size:** 40.00 GB
   * Choose **Create a Virtual Hard Disk Now**

4. **Initial Snapshot (Important)**

   * Before starting the VM, create a snapshot.
   * Click the menu button (next to Start) > **Snapshots** > **Take**.
   * Name it: `Snapshot 1`.

## 2. Oracle Linux Installation Process

Start the VM and follow these installation steps:

1. Select **Install Oracle Linux 8.8.0** from the boot menu.

2. Choose **English (United States)** > Continue.

3. Configure the following under **Installation Summary**:

   * **Time & Date:**
     Region: `Asia`
     City: `Jakarta`

   * **Software Selection:**
     Choose **Server with GUI**

   * **Installation Destination:**

     * Select `ATA VBOX HARDDISK` (40 GiB)
     * Storage Configuration: **Automatic**
     * Click **Done**

   * **Root Password:**

     * Set password: `oracle`
     * Click **Done** twice to confirm (weak password warning)

   * **User Creation:**

     * Create a user (example: `ocean`)
     * Enable *Require a password to use this account*
     * Set and confirm the password

4. Click **Begin Installation**
   After it finishes, click **Reboot System**.

5. **Licensing (Initial Setup)**

   * Open **License Information**
   * Check **I accept the license agreement** > Done
   * Click **Finish Configuration**

## 3. Post-Installation Configuration

### A. Internet Connection

Ensure the VM is connected:

1. Click the network icon (top-right).
2. Click **Wired Off** > **Connect**.

### B. Installing Dependencies (Terminal)

1. Open **Activities** > search **Terminal**.
2. Switch to root:

   ```bash
   su
   ```
3. Install kernel packages and development tools:

   ```bash
   dnf install kernel-uek-devel-$(uname -r) gcc binutils automake make perl bzip2 elfutils-libelf-devel
   ```
4. Type `y` to confirm.

### C. Install Guest Additions

1. Go to **Devices** > **Insert Guest Additions CD image...**
2. Click **Run** when the popup appears.
3. Enter your user password if requested.
4. Wait for the installation to finish.
5. Restart the VM.

### D. Enable Shared Clipboard

1. Go to **Devices** > **Shared Clipboard**
2. Select **Bidirectional**
3. Test copy–paste between host and VM
4. Create a final snapshot:
   `After Oracle 8.8 Installation`

## Troubleshooting

### Issue 1: Guest Additions Does Not Launch

1. Open **Activities** > **Files**
2. Look for **VBox_GAs_...** on the left panel
3. Click **Run Software**
4. Click **Run**

### Issue 2: Host Key (Right Ctrl) Not Working

1. In VirtualBox Manager, press `Ctrl + G`
2. Go to **Input** > **Virtual Machine**
3. Change the **Host Key Combo** to another key (e.g., Right Shift)
4. Click **OK**

## Additional Resources
For advanced visualization, you can find PDF files in the [`docs`](./docs) folder.

---
<br>

*Created for the **Oracle Database Administrator** Course Lab Session at the Faculty of Information Technology, Tarumanagara University.*