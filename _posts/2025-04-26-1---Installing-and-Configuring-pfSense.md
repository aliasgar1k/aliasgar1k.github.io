---
title: 1 - Installing and Configuring pfSense
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation Upgrade 1
tags:
  - splunk-siem-implementation-Upgrade-1
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation-Upgrade-1/2025-04-26-1---Installing-and-Configuring-pfSense/
---

In this guide, you'll learn how to download, install, and configure pfSense in a virtualized environment. We'll cover each step in detail, making it ideal for beginners and lab builders. You’ll also find prompts to watch for during setup, and room to insert helpful screenshots.

## Step 1: Download the pfSense ISO Image

Start by getting the installation image:
1. Go to the official pfSense website: [https://www.pfsense.org/download/](https://www.pfsense.org/download/)
2. Click the **Download** button.
3. Select the following options:
    - **Platform:** AMD64 (64-bit)
    - **Installer:** ISO Installer
    - **Console:** VGA
4. Click **Add to Cart** and proceed to **Checkout**.
5. You'll need a **Netgate account**. Create one or log in.
6. Complete the free checkout process and download the ISO.

![](2025-04-26-1---Installing-and-Configuring-pfSense-1.png)

## Step 2: Create a Virtual Machine for pfSense

Next, create a new virtual machine using your preferred hypervisor (VirtualBox, VMware, etc.). Here are some suggested VM specifications:

| Setting   | Value                          |
| --------- | ------------------------------ |
| **RAM**   | 2 GB                           |
| **CPU**   | 2 cores                        |
| **Disk**  | 10 GB                          |
| **NIC 1** | NAT                            |
| **NIC 2** | Internal Network / LAN Segment |

1. Create a new VM and attach the downloaded pfSense ISO as the boot medium.
2. Ensure the VM has **two network adapters**: 
    - Adapter 1 (WAN): Set to NAT.
    - Adapter 2 (LAN): Set to “Internal Network” (e.g., `LAN Segment`).

![](2025-04-26-1---Installing-and-Configuring-pfSense-2.png)

## Step 3: Boot and Install pfSense

With the pfSense ISO mounted and VM ready, start the virtual machine to begin installation.

1. Accept the copyright and distribution notice.
2. At the welcome screen, select **Install** and press **OK**.
3. Proceed with **network installation**.

![](2025-04-26-1---Installing-and-Configuring-pfSense-3.png)

5. Select the **WAN interface** (usually the first one, depending on your NAT setup).  
6. Press **OK**, then select **Continue** and confirm with **OK**.

![](2025-04-26-1---Installing-and-Configuring-pfSense-4.png)

7. Choose the **LAN interface** (typically the same as your LAN segment).  
8. Press **OK**, then click **Continue** and confirm again with **OK**.

![](2025-04-26-1---Installing-and-Configuring-pfSense-5.png)


9. Confirm the selected interfaces and click **Continue**.  
10. pfSense will check for **network connectivity**.

![](2025-04-26-1---Installing-and-Configuring-pfSense-6.png)

11. When prompted for pfSense Plus, choose **Install CE**.  
12. Accept default **filesystem and partition scheme** — select **Continue** and press **OK**.  
13. Select **“stripe – no redundancy”** as the device type.  
14. Choose the installation **disk** and press **OK**.  
15. Confirm with **YES** to format the disk.

![](2025-04-26-1---Installing-and-Configuring-pfSense-7.png)

16. Select **Current Stable Release** as the version and press **OK**.  
17. Wait for the installation to complete.  
18. On the post-install screen, press **OK**.  
19. Select **Reboot** to finish installation.  
20. On reboot, either press **1** or wait for **autoboot**.

![](2025-04-26-1---Installing-and-Configuring-pfSense-8.png)

21. You will land at the **pfSense Console Menu**, confirming successful installation.

![](2025-04-26-1---Installing-and-Configuring-pfSense-9.png)

## Step 4: Initial Setup via Console

After reboot, the system will assign WAN and LAN interfaces.

![](2025-04-26-1---Installing-and-Configuring-pfSense-10.png)

**Reassign to static IP address on WAN**
From the pfSense console menu, Choose option `2` (Set interface IP):

- Select interface: `WAN`
- Enter new LAN IP address: `192.168.125.80`
- Subnet mask bits: `24`
- Upstream gateway: `192.168.125.2`
- set same address as default gateway: `y`
- configure IPv6 via DHCP: `y` (as we don't need it, let it configure for itself)
- Enable DHCP on WAN? `y` (To allow to take IP address incase our is unavailable)
- Enter start address: `192.168.125.1`
- Enter end address: `192.168.125.254`

![](2025-04-26-1---Installing-and-Configuring-pfSense-11.png)

**Reassign to static IP address on LAN**
From the pfSense console menu, Choose option `2` (Set interface IP):

- Select interface: `LAN`
- configure IPv4 via DHCP: `n`
- Enter new LAN IP address: `192.168.1.110`
- Subnet mask bits: `24`
- Upstream gateway: leave **blank**
- configure IPv6 via DHCP: `y` (as we don't need it, let it configure for itself)
- Enable DHCP on LAN? `y`
- Enter start address: `192.168.1.120`
- Enter end address: `192.168.1.200`

![](2025-04-26-1---Installing-and-Configuring-pfSense-12.png)

## Step 5: Configure SOC Machine to Access pfSense

On your SOC or test machine:

1. Change network adapter to **LAN Segment**.    
2. Set a static IP:

```
IP Address:      192.168.1.120
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.110
DNS Server:      192.168.1.110
```

![](2025-04-26-1---Installing-and-Configuring-pfSense-13.png)

3. Open a browser and go to: `https://192.168.1.110` (accept the risk and continue if warned for self-signed certificate.)

![](2025-04-26-1---Installing-and-Configuring-pfSense-14.png)

Login with:

- Username: `admin`
- Password: `pfsense`

## Step 6: Setup Wizard Configuration

After login, the setup wizard launches automatically. Run along with the settings as defined below:
(if some fields are not defined or left blank in settings, Thus mean it is good by default.)

![](2025-04-26-1---Installing-and-Configuring-pfSense-15.png)
![](2025-04-26-1---Installing-and-Configuring-pfSense-16.png)

**General Information**

- Hostname: `pfsense`
- Domain: `localdomain`
- Primary DNS Server: `192.168.125.2`
- Check the box to **override DNS**

![](2025-04-26-1---Installing-and-Configuring-pfSense-17.png)

**Time Server Information**

- Leave the defaults.

![](2025-04-26-1---Installing-and-Configuring-pfSense-18.png)

**Configure WAN Interface**

- Selected Type: `static` (Set WAN interface (your NAT interface) for static IP address)

![](2025-04-26-1---Installing-and-Configuring-pfSense-19.png)

- IP Address: `192.168.125.80`
- Subnet Mask: `24`
- Upstream Gateway: `192.168.125.2`

![](2025-04-26-1---Installing-and-Configuring-pfSense-20.png)

- Uncheck the checkbox for **Block RFC1918 private networks**

> **Block RFC1918 Private Networks**
> - **RFC1918** IP ranges = private networks (like `10.x.x.x`, `172.16.x.x`, `192.168.x.x`).
> - If **enabled**, pfSense **blocks traffic** from private IP addresses coming into WAN.
> - **Normally**, your WAN (internet side) should _not_ have private IPs — only public internet IPs.
> - **BUT in your case**, your WAN _is_ on a **private network** (VMware NAT: `192.168.125.x`).
> - **So you should UNCHECK this. ❌** Otherwise, pfSense will block VMware NAT traffic.
{: .prompt-info }

![](2025-04-26-1---Installing-and-Configuring-pfSense-21.png)

- Check the checkbox for **Block Bogon Networks**

> **Block Bogon Networks**
> - **Bogon networks** = IP ranges that are unassigned, reserved, or should never be seen publicly.
> - Examples: `0.0.0.0/8`, `100.64.0.0/10`, parts of `240.0.0.0/4`
> - Blocking bogon networks **protects you** from strange/bad traffic.
> - **This is safe to keep enabled.** ✅
{: .prompt-info }

![](2025-04-26-1---Installing-and-Configuring-pfSense-22.png)


**Configure LAN Interface**

- LAN IP Address: `192.168.1.110`
- Subnet: `24`

![](2025-04-26-1---Installing-and-Configuring-pfSense-23.png)

**Set Admin WebGUI Password**

- Set a new password for the `admin` user. (too keep it simple, use same password i.e. `pfsense`)

![](2025-04-26-1---Installing-and-Configuring-pfSense-24.png)

**Reload configuration**

- click on "Reload" button to reload the pfsense with new configurations.

![](2025-04-26-1---Installing-and-Configuring-pfSense-25.png)

**Wizard completed**

- Finish the setup and you will be redirected to pfsense dashboard.

![](2025-04-26-1---Installing-and-Configuring-pfSense-26.png)

![](2025-04-26-1---Installing-and-Configuring-pfSense-27.png)


## Step 7: Configure Firewall Rule (If Needed)

On PfSense's Default Behavior:

- On the **LAN interface** ➔ pfSense allows traffic by default, allowing machine to communicate internally.
- On the **WAN interface** ➔ pfSense **blocks ALL incoming traffic** by default — for security.  

That means: nothing from the **outside** (internet or NAT network) can reach inside **unless you allow it.** meaning no internet connection for now.

but in newer versions of pfsense by default pfsense creates a rule to allow LAN segment to any, allowing it to communicate internally as well as externally. thus we can reach internet.

![](2025-04-26-1---Installing-and-Configuring-pfSense-28.png)

**Note:** If your pfSense version does not create a default LAN-to-any rule, you must add it manually as shown below.

If there's no default "allow all" rule, add one by navigating to **Firewall > Rules > LAN**.

- Action: `Pass`
- Disabled: `Unchecked`
- Interface: `LAN`
- Address Family: `IPv4`
- Protocol: `Any`
- Source: `LAN net`
- Destination: `Any`
- Description: `Default allow LAN to any rule`
- Log: `Unchecked`
- Advanced Options: `Default`

![](2025-04-26-1---Installing-and-Configuring-pfSense-29.png)

## Step 8: Configure and Test Client Machines

Configure additional machines on the pfSense LAN segment as such with static IP address:

| Device         | IP Address    | Subnet Mask   | Default Gateway | DNS           |
| -------------- | ------------- | ------------- | --------------- | ------------- |
| SOC Machine    | 192.168.1.120 | 255.255.255.0 | 192.168.1.110   | 192.168.1.110 |
| Windows Client | 192.168.1.130 | 255.255.255.0 | 192.168.1.110   | 192.168.1.110 |
| Ubuntu Client  | 192.168.1.140 | 255.255.255.0 | 192.168.1.110   | 192.168.1.110 |

**Verify Connectivity**
From each machine verify the connectivity by checking the following routes:

- ✅ Ping `192.168.1.110` (pfSense)
- ✅ Open `https://192.168.1.110` (web interface)
- ✅ Open `https://pfsense.localdomain` (web interface)
- ✅ Ping external IP: `8.8.8.8`
- ✅ Ping external IP: `www.google.com`

## Conclusion

With the installation and initial configuration complete, pfSense is now up and running as the firewall and router for our network. It forms the first line of defense, managing and securing traffic between internal systems and the outside world. In the next step, we’ll extend this setup by introducing an IDS/IPS machine to monitor live traffic, detect threats, and actively prevent malicious activity, building toward a more realistic and practical security lab environment.

