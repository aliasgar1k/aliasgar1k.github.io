---
title: Encrypted Persistence Live USB
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Homelabs
tags:
  - live-usb
  - encrypted
  - persistant
  - lsblk
  - cryptsetup
  - kali-linux
image: preview-image.png
media_subpath: /assets/img/Homelabs/2025-05-31-encrypted-persistence-live-usb/
---

The term “**Live USB Persistence**” refers to the fact that the files and data that we create in USB as a result of our work there, they are **stored** on the USB drive rather than the Hard drive, and by using Encryption in the Live USB, the Stored Data will be encrypted. As of this, we don’t need to create any partitions on our hard drive in order to boot into Kali Linux, unlike in dual booting systems.

## Advantages of Live USB Persistence:

* It’s Portable.
* Easy to test new tools and distros.
* Saves your work on USB in an encrypted form.

## Steps to Create a Kali Live Persistent USB:

**Step 1:** Navigate to the Kali download page in your Browser and Download [Kali Linux](https://www.kali.org/get-kali/#kali-live) live Installer as of your system architecture and needs.&#x20;

![](2025-05-31-encrypted-persistence-live-usb-2.png)

**Step 2:** Now to Flash Kali Image in USB, we require a tool i.e., [**Rufus**](https://rufus.ie/en/)**.** Navigate to the link and download it.

**Step 3:** Insert your USB into the USB port and Open Rufus Software.

**Step 4:** Select your USB device in Rufus.

![](2025-05-31-encrypted-persistence-live-usb-3.png)

**Step 5:** Select the Downloaded Kali Linux live Installer Image by Clicking on Select in Boot Selection.

![](2025-05-31-encrypted-persistence-live-usb-4.png)

**Step 6:** Now select the Persistent Partition Size as of your USB size.

![](2025-05-31-encrypted-persistence-live-usb-5.png)

**Step 7:** Then press the Start button. You will be informed that all data on the USB will be deleted; if you agree, click OK.

**Note: Your USB will now be formatted and flashed with Kali Linux live.**&#x20;

**Step 8:** Shut down your computer, then turn it back on, then go to the Boot Menu and choose Kali.

**Step 9:** When the Kali Boot menu loads, select **Live system with USB Encrypted persistence.** It will start Kali for the first time without requesting that you create a password or login system.

![](2025-05-31-encrypted-persistence-live-usb-6.png)

**Step 10:** When Kali loads the desktop, open the terminal with **root** permission.

**Step 11:** Check for your USB block for further process by following command.

```
lsblk
```

In my case, it’s in the third block, i.e. `/dev/sdc2`.

![](2025-05-31-encrypted-persistence-live-usb-7.png)

**Step 12:** We’re going to use LUKs Encryption to encrypt the USB. Commands listed below.

```
cryptsetup –verbose –verify-passphrase luksFormat /dev/sdc2
cryptsetup luksOpen /dev/sdb3 kali_usb
```

![](2025-05-31-encrypted-persistence-live-usb-8.png)

![](2025-05-31-encrypted-persistence-live-usb-9.png)

whereas **cryptsetup** is used to conveniently set up dm-crypt managed device-mapper mappings; first flag **–verbose** print messages for current action; the other flag **–verify-passphrase** asks you to enter your password twice; **luks** is an extension of cryptsetup for disk encryption and the last argument is the **USB Block**. Keep in mind the password you choose.

**Step 13:** Now we need to create an ext4 file system, namely “persistence”.

```
mkfs.ext4 -L persistence /dev/mapper/kali_usb
e2label /dev/mapper/kali_usb persistence
```

![](2025-05-31-encrypted-persistence-live-usb-10.png)

![](2025-05-31-encrypted-persistence-live-usb-11.png)

**Step 14:** To mount our encrypted disc, we must first build a mount point, after which we will create a persistence.conf file and unmount the encrypted partition. See the commands below for the desired.

```
mkdir -p /mnt/kali_usb/
mount /dev/mapper/kali_usb /mnt/kali_usb
echo “/ union” > /mnt/kali_usb/persistence.conf
umount /dev/mapper/kali_usb
```

**Step 15:** The channel to our encrypted persistent partition needs to be shut off last now.

```
cryptsetup luksClose /dev/mapper/kali_usb
```

**Step 16:** Now our USB is ready to boot with Encrypted Persistence Storage. Shut down the computer, start it again and enter to boot menu, and select Kali.

**Step 17:** Now from the Kali Boot menu, select **Live system with USB Encrypted persistence.**

**Step 18:** Due to encryption, it will go slow. At the initial time, it will ask for the password that we created, Enter the password, and it will take you to the Desktop if the password is correct.

![](2025-05-31-encrypted-persistence-live-usb-12.png)
