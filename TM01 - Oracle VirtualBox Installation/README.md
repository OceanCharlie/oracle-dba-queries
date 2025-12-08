# TM01 - Oracle VirtualBox Installation

This document guides you through the installation process of Oracle VM VirtualBox version **7.2.0.** VirtualBox is required as the virtualization environment to run the Oracle database.

## Table of Contents
- [System Prerequisites](#system-prerequisites)
- [Download Installer](#1-download-installer)
- [Instalation Guide](#2-instalation-guide)
- [Troubleshooting](#3-troubleshooting)


## System Prerequisites
Before starting, ensure your laptop meets the following criteria to avoid installation failure:

1.  **Microsoft Visual C++ Redistributable (Mandatory for Windows):**
    * Oracle VirtualBox requires *Microsoft Visual C++ 2019* or higher.
    * If not installed, download the **x64** version (for standard 64-bit laptops) at: [Microsoft Learn - Latest Supported VC Redist](https://learn.microsoft.com/id-id/cpp/windows/latest-supported-vc-redist?view=msvc-170).
    * **Important:** Perform a **Laptop Restart** after installing Visual C++ before proceeding to install VirtualBox.
2.  **Storage Space:**
    * A minimum **230MB** of free hard drive space is required for the base application.

## 1. Download Installer
Download the official installer according to your operating system:

* **Official Website:** [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) 
* **Platform Options:**
    * `Windows hosts`: For Windows users.
    * `macOS / Intel hosts`: For MacBook with Intel processors.
    * `macOS / Apple Silicon hosts`: For MacBook with M1/M2/M3 chips.


## 2. Installation Guide
Follow these steps after the installer file has been successfully downloaded:

1.  **Run Installer:** Open the `Oracle VirtualBox 7.2.0 Installer`.
2.  **Setup Wizard:** On the Welcome screen, click **Next**.
3.  **License Agreement:**
    * Select the option **"I accept the terms of the license agreement"**.
    * Click **Next**.
4.  **Custom Setup:**
    * Leave the features selected by *default* (including USB Support & Networking).
    * Click **Next** (unless you wish to change the installation location via the *Change* button).
5.  **Network Warning Confirmation:** (If a *Network Interfaces* warning appears, click **Yes** to proceed).
6.  **Ready to Install:** Click the **Install** button.
7.  **Process:** Wait until the status bar is full and the message "Oracle VirtualBox 7.2.0 has been installed" appears.
8.  **Finish:** Check "Start Oracle VirtualBox" then click **Finish**.

## 3. Troubleshooting
If you encounter issues during installation:

| Issue | Solution |
| :--- | :--- |
| **Error: Visual C++ Missing** | The message *"Needs Microsoft Visual C++ 2019..."* appears.<br> **Solution:** See the [System Prerequisites](#system-prerequisites) section above. Download Visual C++, install, then **you must Restart the laptop**. |
| **Installation Failed / Rollback** | Check your internet connection during the download or ensure your antivirus is not blocking the installer. |

## Additional Resources
For advanced visualization, you can find PDF files in the [`docs`](./docs) folder.

---
<br>

*Created for the **Oracle Database Administrator** Course Lab Session at the Faculty of Information Technology, Tarumanagara University.*