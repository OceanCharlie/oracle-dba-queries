# TM04 - Fundamental VM & Basic Linux Commands

This document contains a summary of materials and practical guides regarding the basics of **Virtual Machine (VM)** and **Linux** operating system commands.

## Table of Contents

  - [Learning Objectives](#learning-objectives)
  - [Introduction to Virtual Machine & Linux](#1-introduction-to-virtual-machine--linux)
  - [Basic Navigation](#2-basic-navigation-navigation)
  - [File & Folder Management](#3-file--folder-management)
  - [User & Permission Management](#4-user--permission-management)
  - [Additional Resources](#additional-resources)

## Learning Objectives

After completing this module, students are expected to be able to:
* Understand the fundamentals of Virtual Machines.
* Recognize basic Linux commands.
* Create, manage, and delete files/directories.
* Understand file *permission* concepts.

---

## 1. Introduction to Virtual Machine & Linux

### What is a Virtual Machine?

A Virtual Machine (VM) is a virtual environment that emulates a physical computer within a main computer. This allows users to run other operating systems safely, flexibly, and in isolation without disturbing the main system.

**Why use a VM?**
* **Security:** Linux runs separately from the main system, so errors do not damage your laptop.
* **Flexibility:** Allows for risk-free experimentation and running multiple OSs simultaneously.
* **Real Simulation:** Presents a simulation of a real Linux server.

### What is Linux?

Linux is an *open-source* operating system (OS) based on UNIX that bridges *hardware* and *software*, known for being stable, secure, and free.

**Brief Comparison:**

| Feature | Linux | Windows | MacOS |
| :--- | :--- | :--- | :--- |
| **Nature** | Open-source, Free | Proprietary, Paid | Proprietary, Paid |
| **Flexibility** | Very High | Limited | Very Limited (Apple Ecosystem) |
| **Security** | Relatively secure, rare viruses | Vulnerable to viruses & malware | Very secure (closed system) |
| **Cost** | Free | Paid (License) | Expensive (Device Bundling) |

---

## 2. Basic Navigation

Here are the basic commands to move between directories and view folder contents:

* **`pwd`** (*Print Working Directory*):
    Displays the *path* (location) of the directory you are currently working in.
    ```bash
    [ocean@localhost ~]$ pwd
    /home/ocean
    ```

* **`ls`** (*List*):
    Displays a list of files and folders. Output is usually colored: White (File), Blue (Directory), Green (Executable).

* **`ls -l`** (*List Long*):
    Displays a directory listing with complete details (access permissions, owner, size, date).

* **`cd`** (*Change Directory*):
    Changes the cursor position to another directory location.
    ```bash
    cd /home/ocean/Downloads
    ```

---

## 3. File & Folder Management

### Creating Files

There are several ways to create a file in Linux:

1.  **`touch filename`**: Creates a new empty file or updates the *timestamp*.
2.  **`nano filename`**: A simple and *user-friendly* text editor (Exit: `Ctrl+X`, Save: `Ctrl+O`).
3.  **`vi filename`**: A *powerful* but more complex text editor.
    * Press `i` to Insert (write).
    * Press `Esc` to exit edit mode.
    * Type `:wq` to Save & Quit.

### Managing Files

* **`cp`** (*Copy*): Copies a file or folder.
    ```bash
    cp Monday.txt DestinationFolder
    ```
* **`mv`** (*Move*): Moves a file or renames a file.
    ```bash
    mv Monday.txt DestinationFolder
    ```
* **`rm`** (*Remove*): Deletes a file.
    ```bash
    rm Monday.txt
    ```

### Deleting Directories (IMPORTANT)

Pay attention to the difference between these two commands to avoid accidental data loss:

| Command | Function | Condition |
| :--- | :--- | :--- |
| **`rmdir`** | Remove Directory | Can only delete **empty directories**. |
| **`rm -r`** | Remove Recursive | Deletes a directory **along with all its contents** (files & subfolders). |

---

## 4. User & Permission Management

Linux has access right rules (*Permissions*) to determine who is allowed to read, write, or execute a file.

### Permission Structure

Example: `drwxr-xr-x`
* First character: `d` (directory) or `-` (regular file).
* Next 9 characters are divided into 3 groups: **Owner**, **Group**, **Others**.

### Access Values (Chmod)

Access rights are represented by numbers:
* **r** (Read) = **4**
* **w** (Write) = **2**
* **x** (Execute) = **1**

**Total Permission Table:**

| Number | Meaning | Description |
| :--- | :--- | :--- |
| **7** | `rwx` | Read + Write + Execute (Full Access) |
| **6** | `rw-` | Read + Write |
| **5** | `r-x` | Read + Execute |
| **4** | `r--` | Read Only |
| **0** | `---` | No access |

### Changing Permissions

Use the **`chmod`** (*Change Mode*) command to change access permissions.

**Example:**
```bash
chmod 476 Monday.txt
```

Meaning:
  * **Owner (4)**: Read only.
  * **Group (7)**: Read + Write + Execute (Full).
  * **Others (6)**: Read + Write.

---

## Additional Resources

To support learning, you can access the following supplementary materials:

  * **Additional Visualization:**
    PDF materials and visual guides can be found in the [`docs`](./docs) folder.

  * **Wayground (Interactive Material & Quiz):**
    Visit the following link to access presentation slides (PPT) and take practice quizzes related to this material:
    [Access Wayground - Fundamental VM & Linux](https://wayground.com/join?gc=65948550)

-----

<br>

*Created for the **Oracle Database Administrator** Course Practicum at the Faculty of Information Technology, Tarumanagara University.*
