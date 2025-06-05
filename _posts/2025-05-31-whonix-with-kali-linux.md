---
title: Whonix With Kali Linux
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Homelabs
tags:
  - kali-linux
  - whonix
  - virtual-box
image: preview-image.png
media_subpath: /assets/img/Homelabs/2025-05-31-whonix-with-kali-linux/
---

## Whonix ™ – Overview:

Whonix ™ is an anonymous operating system that runs like an app and routes all Internet traffic through the Tor anonymity network. It offers privacy protection and anonymity online and is available for all major operating systems. Whonix ™ is a free and open-source desktop operating system (OS) that is specifically designed for advanced security and privacy. Based on the Tor anonymity network, Kicksecure ™Whonix ™ defeats common attacks while maintaining usability. Online anonymity and censorship circumvention is attainable via fail-safe, automatic and desktop-wide use of the Tor network, meaning all connections are forced through Tor or blocked.

## Whonix Architecture:

* Workstation
* Gateway

The gateway is responsible for directing the traffic through TOR network, and workstation which as you guess is the OS of Whonix, so we can replace the workstation with Kali Linux, and thus all the traffic will route through the WHONIX gateway instead of your own ISP gateway (the ISP will see the encrypted traffic only). Let’s replace the Whonix Workstation with Kali Linux and connect it with the gateway to make it anonymous.

## Things you will need:

* Virtual box: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
* Kali Linux: [https://www.kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)
* Whonix Gateway: [https://www.whonix.org/wiki/Download](https://www.whonix.org/wiki/Download)

## whonix installation:

### STEP 1: Importing Whonix to Virtual Box

* double click on whonix.ova file
* it will pop a menu showing all predefined configurations for virtual machine, here you can make change as you want.
* but for now defaults are all good
* import and agree to all the agreements for both VMs.

![](2025-05-31-whonix-with-kali-linux-2.png)

![](2025-05-31-whonix-with-kali-linux-3.png)

![](2025-05-31-whonix-with-kali-linux-4.png)

![](2025-05-31-whonix-with-kali-linux-5.png)

### STEP 2: Setup Whonix

* Once you have imported, start the newly imported whonix-gateway-XFCE and wait till it loads all the components.
* Now you will have a whonix setup wizard.
* here read the terms and click on understand then click next to move forward.
* and same for the next agreement: **read > understand > next**
* lastly click on finish to **Finish** the setup.

![](2025-05-31-whonix-with-kali-linux-6.png)

![](2025-05-31-whonix-with-kali-linux-7.png)

* now you will be promoted with anon connection wizard
* here select connect or configure if you have your own configuration in mind for bridges and local proxy.
* **Connect > Next > Next**

![](2025-05-31-whonix-with-kali-linux-8.png)

### STEP 3: Changing Whonix password and Checking Tor Connetction

* At this point it is setting up tor configuration for you.
* once done you will have to click finish to complete our setup.
* once done you will see a pop which will make some system check and connect to tor network for us.
* after this is done
* we have to change our user password, which is by default **changeme**.

```bash
passwd
changeme
(enter a new password)
(enter the same password again)
```

* here you now check if tor is working correctly by opening start menu and starting tor control panel
* here you will see it tor is active or not, logs and also you change a new identity.

![](2025-05-31-whonix-with-kali-linux-9.png)

* now from here all we need is our whonix gateway IP address. for this use command “`ip a`“
* here eth1 is the IP address we need.

![](2025-05-31-whonix-with-kali-linux-10.png)

## Kali-linux setup:

### STEP 1: Importing Kali-linux in Virtual box

* now you will need to install kali linux from ova file
* double click on on **kali-linux.ova > import > agree (done)**
* but before starting it we will click on kali linux machine and open settings
* here select networks settings and in adapter 1 select
* attached to: internal network
* and name: whonix
* now click “ok” and close settings

![](2025-05-31-whonix-with-kali-linux-11.png)

### STEP 2: Manually Configuring Network settings

* we can now start Kali Linux. once started you will need to make some changes in network settings
* from **settings > network > wired**
* here in ipv4 select method as manual
* and in addresses fill the following.

![](2025-05-31-whonix-with-kali-linux-12.png)

* here gateway will be same as our whonix IP address for eth1.
* and also same for DNS.
* now click on **Apply** and we are DONE….
* now your network is routed through whonix which is using tor making you anonymous.
* you can check this using by going to [https://check.torproject.org/](https://check.torproject.org/).
