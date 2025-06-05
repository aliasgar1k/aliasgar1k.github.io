---
title: Install Windows 11 On VMware Workstation
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Homelabs
tags:
  - windows
  - vmware
image: preview-image.png
media_subpath: /assets/img/Homelabs/2025-05-31-install-windows-11-on-vmware-workstation/
---

The following are the steps to install Windows 11 on VMware Workstation Player

## Step 1: Download & Install VMware Workstation

To install Windows 11 on VMware Workstation, you should first go to download & install the [VMware Workstation Player](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html) on your PC.

## Step 2: Download Windows 11 ISO

Now download [ISO for windows 11](https://www.microsoft.com/software-download/windows11).

Please check your internet connection, choose a safe location to store the ISO file, and wait patiently while it’s downloading.

## Step 3: Create a Windows 11 Virtual Machine

* Run VMware Workstation
* Select File > **New Virtual Machine** from the main menu of the program (you can also press **Ctrl+N** or select **Create a New Virtual Machine**)

![](2025-05-31-install-windows-11-on-vmware-workstation-2.png)

* Choose **Typical (recommended)** and click **Next**.

![](2025-05-31-install-windows-11-on-vmware-workstation-3.png)

* Choose **Installer disc image file (iso)** and click **Next**.

![](2025-05-31-install-windows-11-on-vmware-workstation-4.png)

* Choose **Microsoft Windows**, select **Windows 10 x64**, and click **Next**.

![](2025-05-31-install-windows-11-on-vmware-workstation-5.png)

* Type a **Virtual machine name** (e.g. Windows 11) and click **Next**.

![](2025-05-31-install-windows-11-on-vmware-workstation-6.png)

* Allocate some disk space to be used by the virtual machine. Type **64 GB** or more after **Maximum disk size**.
* Choose **Store virtual disk as a single file** and click **Next**.

![](2025-05-31-install-windows-11-on-vmware-workstation-7.png)

![](2025-05-31-install-windows-11-on-vmware-workstation-8.png)

## Step 4: Add Trusted Platform Module

1. Go to **Edit virtual machine settings**.
2. Go to **Options** -> **Access Control**, click **Encypt** and then **OK**.

![](2025-05-31-install-windows-11-on-vmware-workstation-9.png)

1. Now switch to **Hardware** -> **Add**, select the **Trusted Platform Module** and then click **Finish**.

## Step 5: Install Windows 11 on Virtual MachineSteps to install Windows 11 on emulator

1. Navigate to the created Windows 11 virtual machine.
2. Click **Power on this virtual machine**.
3. Select **English** for Language to install in the Windows Setup window.
4. Click **Next**.
5. Click **Install Now**.
6. Click **I don’t have a product key** at the bottom.
7. Select the operating system you want to install: Windows 11 Home, Windows 11 Education, Windows 11 Pro, etc.
8. Click **Next**.
9. Check **I accept the Microsoft Software License Terms** and click **Next**.
10. Select **Custom: Install Windows only (advanced)**.
11. Click **New**, allocate some disk space, and click **Apply**.
12. Select the new partition and click **Next**.
13. Wait for the installation to complete.

![](2025-05-31-install-windows-11-on-vmware-workstation-10.png)

## Bonus Tip

some of you might not have proper internet connection while installing, but as per Microsoft will windows 11 can only be installed while being online.

but don’t worry we have a walk around

**Step 1:** `shift + F10` (this will start a command prompt windows)\
**Step 2:** `OOBE\BYPASSNRO` (type in following command then press ENTER)

it will restart the virtual machine and now go through the process of setup and this time you will have a new **“I Don’t have internet”** option.
