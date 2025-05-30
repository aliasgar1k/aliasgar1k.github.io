---
title: Set a Static IP Address on Windows
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Others
tags:
  - static-ip
image: preview-image.png
media_subpath: /assets/img/Others/2025-05-26-Set-a-Static-IP-Address-on-Windows/
---
  
Configuring a static IP address on your Windows machine can be useful for a variety of reasons, such as setting up a server, ensuring a fixed IP for your network devices, or troubleshooting network issues. This guide will walk you through the process of setting a static IP using both the **Graphical User Interface (GUI)** and the **Command Line**.
## Method 1: Set Static IP Using the GUI
  
### Step 1: Open Network Connections
  
Press the **Windows + R** keys to open the Run dialog. Type the following command and click **OK**:  
  
```
ncpa.cpl
```

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-1.png)

This will open the **Network Connections** window, where you can manage all your network adapters.
  
### Step 2: Access Network Adapter Properties
  
In the **Network Connections** window, locate the network adapter you're using (such as **Ethernet** or **Wi-Fi**). Right-click on the adapter and select **Properties** from the context menu.
  
![](2025-05-26-Set-a-Static-IP-Address-on-Windows-2.png)

### Step 3: Configure IPv4 Settings
  
In the **Properties** dialog, scroll down to **Internet Protocol Version 4 (TCP/IPv4)**, select it, and click on the **Properties** button.

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-3.png)

### Step 4: Set a Static IP Address
  
In the **Internet Protocol Version 4 (TCP/IPv4) Properties** window, follow these steps:

**Change IP Address Settings:**

- Select **Use the following IP address** and enter your desired static IP details.

Example configuration:

```
IP Address: 192.168.125.20  
Subnet Mask: 255.255.255.0    
Default Gateway: 192.168.125.2
```

**Configure DNS Settings:**

- Select **Use the following DNS server addresses** and enter the DNS server addresses.

Example DNS configuration:

```
Preferred DNS Server: 192.168.125.2
Alternate DNS Server: (Leave blank or specify an alternative.)
```

Your settings should look like this:  

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-4.png)

### Step 5: Apply Settings
  
Once you've entered all the required information, click **OK** to save your changes. Then, click **Close** on the **Ethernet Properties** window to exit.

### Step 6: Verify the Settings
  
To confirm that your settings have been applied correctly, open **Command Prompt** and type the following command:
  
```
ipconfig /all
```

This will display your current network configurations, and you should see the static IP address you just configured listed under your network adapter.

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-5.png)

## Method 2: Set Static IP Using Command Line

If you prefer to use the **Command Line** (CMD or PowerShell), you can also set a static IP address using `netsh`. Here’s how:

### Step 1: Open Command Prompt as Administrator

Press **Win + X**, and choose **Command Prompt (Admin)** or **Windows Terminal (Admin)**.
Run it as an Administrator.

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-6.png)

### Step 2: List Available Network Interfaces

In the command prompt, type the following command to list all available network interfaces:

```
netsh interface ipv4 show interfaces
```

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-7.png)

Note the name of the interface you want to configure (e.g., `"Ethernet"` or `"Wi-Fi"`).

### Step 3: Set Static IP Address

Use the following command to set a static IP address:

```
netsh interface ipv4 set address name="Ethernet0" static 192.168.125.15 255.255.255.0 192.168.125.2
```

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-8.png)

Replace `"Ethernet"` with the name of your network interface. Here:

- `192.168.125.15` is the static IP you want to assign.
- `255.255.255.0` is your subnet mask.
- `192.168.125.2` is your default gateway.

### Step 4: Set DNS Server

To configure DNS servers, use these commands:

```
netsh interface ipv4 set dns name="Ethernet0" static 192.168.125.2
netsh interface ipv4 add dns name="Ethernet0" 8.8.8.8 index=2
```

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-9.png)

The first command sets your **preferred DNS server**, and the second command adds an **alternate DNS server** (Google’s public DNS in this case). Alternatively you can also choose to set different DNS server as per the need of your network environment.

### Step 5: Verify Configuration

After applying the changes, verify that the static IP has been correctly assigned by running:

```
ipconfig /all
```

![](2025-05-26-Set-a-Static-IP-Address-on-Windows-10.png)

Check that your IP, gateway, and DNS are correctly set.

## Conclusion

Setting a static IP address on Windows is a straightforward process. Whether you prefer the graphical interface or the command line, both methods give you full control over your network settings. By following this guide, you can easily configure your machine to use a fixed IP address, making it more stable and reliable for tasks like setting up servers or ensuring your devices are always accessible on the same IP.

