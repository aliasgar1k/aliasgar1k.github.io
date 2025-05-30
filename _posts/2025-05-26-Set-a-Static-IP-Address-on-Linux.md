---
title: Set a Static IP Address on Linux
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Others
tags:
  - static-ip
image: preview-image.png
media_subpath: /assets/img/Others/2025-05-26-Set-a-Static-IP-Address-on-Linux/
---

Setting a static IP address is an essential skill when configuring Linux for servers, virtual machines, or network-dependent applications. Unlike a dynamic IP address that changes, a static IP remains fixed, providing consistency and reliability.

In this guide, we’ll walk through various methods for setting a static IP on different Linux systems, based on their network management tools.

## Why Set a Static IP Address?

- Remote access (SSH, FTP, etc.)
- Hosting web or game servers
- Port forwarding and firewall rules
- Reducing DHCP issues on headless systems

## Prerequisites

- Sudo/root access
- Basic Linux terminal skills
- Network information:
    - Desired IP address (e.g., 192.168.1.100)
    - Subnet mask or CIDR (e.g., 255.255.255.0 or /24)
    - Gateway (e.g., 192.168.1.1)
    - DNS servers (e.g., 8.8.8.8, 8.8.4.4)

## How to Detect Your Network Manager

To set a static IP, it's important to know which network manager your Linux system is using. Follow these steps:

### Check for Netplan

First, check if your system is using **Netplan**. Run:

```bash
ls /etc/netplan/*.yaml
```

If you see files listed, inspect them to find out which network manager is being used. Run:    

```bash
grep renderer /etc/netplan/*.yaml
```

- If the output says `renderer: NetworkManager`, use **Method 2: NetworkManager**.
- If the output says `renderer: networkd`, use **Method 3: systemd-networkd**.

### If Netplan is not found, check active services

Run these commands to see which network manager service is running:

```bash
systemctl is-active NetworkManager
```

- For **NetworkManager**: If it shows **active**, use **Method 2: NetworkManager**.

```
systemctl is-active systemd-networkd
```

- For **systemd-networkd**: If **active**, use **Method 3: systemd-networkd**.

```
systemctl is-active networking
```   

- For **ifupdown**: If **active** or if `/etc/network/interfaces` exists, use **Method 4: ifupdown**.

```
systemctl is-active dhcpcd
```

- For **dhcpcd**: If **active**, or if `/etc/dhcpcd.conf` exists, use **Method 5: dhcpcd**.

Once you identify the correct network manager, follow the appropriate method to set a static IP.

## Detection Table: Network Manager by System State

Here’s a quick reference table to help you choose the correct method based on your system's output:

|**Recommended Method**|**Indicators / Command Outputs**|
|---|---|
|🔢 **Method 1: Netplan**|`ls /etc/netplan/*.yaml` → files found, <br>`grep renderer` → present|
|🔢 **Method 2: NetworkManager**|`systemctl is-active NetworkManager` → `active`<br>             OR <br>`renderer: NetworkManager`|
|🔢 **Method 3: systemd-networkd**|`systemctl is-active systemd-networkd` → `active`<br>             OR <br>`renderer: networkd`|
|🔢 **Method 4: ifupdown**|`systemctl is-active networking` → `active`<br>             OR <br>`/etc/network/interfaces` exists|
|🔢 **Method 5: dhcpcd**|`systemctl is-active dhcpcd` → `active`<br>             OR <br>`/etc/dhcpcd.conf` exists|

If this still confuses you, you can simply use my script to detect so:

```bash
#!/bin/bash

echo "=== Detecting Linux Distribution and Network Manager ==="

# Detect Linux Distribution
echo -e "\n=== Linux Distribution ==="
if [ -f /etc/os-release ]; then
    cat /etc/os-release
elif [ -f /etc/*release ]; then
    cat /etc/*release
else
    echo "Unable to detect distribution."
fi

# Detect Network Manager
echo -e "\n=== Network Manager Detection ==="

# Netplan (Ubuntu, Pop!_OS, etc.)
if ls /etc/netplan/*.yaml &>/dev/null; then
    echo "→ Netplan is in use"
    
    # Process all Netplan files and extract the renderer
    renderer=""
    for netplan_file in /etc/netplan/*.yaml; do
        file_renderer=$(grep 'renderer' "$netplan_file" | awk '{print $2}')
        if [ -n "$file_renderer" ]; then
            renderer="$file_renderer"
            break
        fi
    done

    # Based on the renderer, print the result
    if [ "$renderer" == "NetworkManager" ]; then
        echo "  Using NetworkManager via Netplan"
    elif [ "$renderer" == "networkd" ]; then
        echo "  Using systemd-networkd via Netplan"
    else
        echo "  Renderer not specified or unrecognized in Netplan."
    fi


# NetworkManager
elif systemctl is-active --quiet NetworkManager; then
    echo "→ NetworkManager is active"
    
# systemd-networkd
elif systemctl is-active --quiet systemd-networkd; then
    echo "→ systemd-networkd is active"
    
# ifupdown
elif systemctl is-active --quiet networking; then
    echo "→ ifupdown is in use (networking service)"
    
# dhcpcd (Raspberry Pi, Alpine, etc.)
elif systemctl is-active --quiet dhcpcd; then
    echo "→ dhcpcd is active"
    
else
    echo "→ No known network manager detected"
fi
```

## 🔢 Method 1: Netplan (Ubuntu 18.04+, Pop!_OS, etc.)

Netplan is the default on newer Ubuntu-based distributions.

### Step-by-step:

1. Open the Netplan config file:

```
sudo nano /etc/netplan/01-netcfg.yaml
```

2. Modify it like this:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

3. Apply the configuration:

```
sudo netplan apply
```

## 🔢 Method 2: NetworkManager (`nmcli`, `nmtui`, or GUI)

Common in Fedora, CentOS 8+, Debian desktops, and others.

### Step-by-step using `nmcli`:

1. Find your connection name:

```
nmcli con show
```

2. Set static IP:

```
nmcli con mod "Wired connection 1" ipv4.method manual \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 8.8.4.4"
```

3. Bring up the connection:

```
nmcli con up "Wired connection 1"
```

Alternative: 
Run `nmtui` and navigate to "Edit a connection" > Select interface > Set IPv4 config to manual.


## 🔢 Method 3: systemd-networkd (Arch, Debian minimal, etc.)

### Step-by-step:

1. Create a new config file:

```
sudo nano /etc/systemd/network/20-wired.network
```

2. Add your configuration:

```
[Match]
Name=enp3s0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
```

3. Enable and restart service:

```
sudo systemctl enable systemd-networkd
sudo systemctl restart systemd-networkd
```

## 🔢 Method 4: ifupdown (/etc/network/interfaces, Debian)

### Step-by-step:

1. Edit the interfaces file:

```
sudo nano /etc/network/interfaces
```

2. Add or modify this block:

```
auto enp3s0
iface enp3s0 inet static
  address 192.168.1.100
  netmask 255.255.255.0
  gateway 192.168.1.1
  dns-nameservers 8.8.8.8 8.8.4.4
```

3. Restart networking service:

```
sudo systemctl restart networking
```

## 🔢 Method 5: dhcpcd (Raspberry Pi OS, Alpine)

### Step-by-step:

1. Open dhcpcd config:

```
sudo nano /etc/dhcpcd.conf
```

2. Append the following:

```
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8 8.8.4.4
```

3. Restart the service:

```
sudo systemctl restart dhcpcd
```

## Verifying Your Static IP

```
ip a
ping 8.8.8.8
ping www.google.com
```

## Troubleshooting

- Interface name may vary (`eth0`, `enp3s0`, etc.)
- YAML syntax in Netplan is sensitive (spaces, colons)
- Don't forget `sudo` for commands
- Use `journalctl -xe` or `dmesg` for error logs

## Conclusion

Setting a static IP address in Linux is easy once you understand your distro's network management system. Whether you're working on a server, desktop, or IoT device, the methods above will help you keep your network configuration stable and reliable.
