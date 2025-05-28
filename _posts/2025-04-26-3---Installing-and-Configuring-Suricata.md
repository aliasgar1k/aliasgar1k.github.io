---
title: 2025-04-26-3---Installing-and-Configuring-Suricata
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation Upgrade 1
tags:
  - splunk-siem-implementation-Upgrade-1
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation-Upgrade-1/2025-04-26-3---Installing-and-Configuring-Suricata/
---


---
title: Installing and Configuring Suricata
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation Upgrade 1
tags:
  - splunk-siem-implementation-Upgrade-1
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation-Upgrade-1/Installing and Configuring Suricata/
---


Once the pfSense is set as firewall/router in the network, next we’ll walk through the process of building a **transparent IDS sensor** using **Suricata** on **Ubuntu Linux**. Suricata will monitor network traffic by acting as a bridge between your firewall (pfSense) and our internal Machines. This approach allows us to inspect all traffic without introducing latency or requiring readdressing — perfect for a stealthy intrusion detection setup. 

We'll also send Suricata logs to **Splunk** for centralized visibility and correlation with other logs.

## Overview

Our goal is to deploy an Ubuntu machine as a **bridge** between your **pfSense firewall** and the **internal LAN (clients/SOC)**. This machine will use **two physical NICs**:

- `ens33`: connected to pfSense LAN 
- `ens37`: connected to the internal LAN (let's say Suricata LAN)

Ubuntu will act as a transparent layer in between — forwarding traffic across the two interfaces while Suricata observes every packet. Deciding if to alert and/or block on certain threat if encountered. 

## 1. Installing Ubuntu with Two NICs


We will begin by installing a **Ubuntu** inside a virtual machine using your preferred hypervisor (VirtualBox, VMware, etc.). During the setup, attach **two virtual NICs** to the VM:

- **NIC 1**  → connect this to the **pfSense LAN segment**
- **NIC 2** → connect this to the **Suricata LAN segment** (your internal network)

![](2025-04-26-3---Installing-and-Configuring-Suricata-1.png)

Once Ubuntu is installed, log in and ensure both interfaces are recognized:

```
ip a
```

At this stage, both NICs may be unconfigured, that’s fine. We’ll set them up as a bridge in the later step.

![](2025-04-26-3---Installing-and-Configuring-Suricata-2.png)

## 2. Installing Suricata

We’ll now install Suricata, the open-source intrusion detection engine that will inspect traffic passing through our bridge.

First, update packages and install Suricata along with its rule updater:

```
sudo apt update && sudo apt upgrade -y
sudo apt install suricata suricata-update -y
```

Then update Suricata's detection rules:

```
sudo suricata-update
```

(Optional) To verify the installation is working:

```
sudo suricata -T -c /etc/suricata/suricata.yaml
```

![](2025-04-26-3---Installing-and-Configuring-Suricata-3.png)

This runs a test against the current configuration file.

Enable Suricata to start on boot:

```
sudo systemctl enable suricata
```

## 3. Enabling IP Packet Forwarding

Since we’re building a bridge that passes traffic from one NIC to the other, we need to ensure the OS allows packet forwarding.

Edit the system config:

```
sudo nano /etc/sysctl.conf
```

Uncomment or add this line:

```
net.ipv4.ip_forward=1
```

![](2025-04-26-3---Installing-and-Configuring-Suricata-4.png)

Apply the setting:

```
sudo sysctl -p
```

This step enables Ubuntu to forward IP packets between interfaces, an essential requirement for bridging.

## 4. Creating the Bridge Network

Now we’ll configure the two NICs (`ens33` and `ens37`) to operate as a single bridge interface `br0`. This bridge will forward traffic at Layer 2 (Ethernet) and allow Suricata to monitor the flow in both directions.

Edit the Netplan config:

```
sudo nano /etc/netplan/01-network-manager-all.yaml
```

Add the following:

```
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
    ens37:
      dhcp4: no
  bridges:
    br0:
      interfaces: [ens33, ens37]
      dhcp4: yes
```

![](2025-04-26-3---Installing-and-Configuring-Suricata-5.png)

What this does:

- **ens33 and ens37** are added to a bridge called `br0`
- **br0** will obtain its IP via DHCP from pfSense
- The individual interfaces won’t have IPs (they're just physical members of the bridge)

**Note**: The Suricata bridge does not require a static IP unless you want it for SSH or management. But we would take it, as if just in case.

```
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
    ens37:
      dhcp4: no
  bridges:
    br0:
      interfaces: [ens33, ens37]
      addresses:
        - 192.168.1.200/24      # Replace with desired static IP
      gateway4: 192.168.1.110   # Replace with your pfSense IP (default gateway)
      nameservers:
        addresses:
          - 192.168.1.110       # DNS server (usually pfSense)
          - 8.8.8.8             # Optional secondary DNS (Google)
```

Change the file permissions to read only for root.

```
sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
```

Apply the changes:

```
sudo netplan apply
```

If needed, restart NetworkManager

```
sudo systemctl restart NetworkManager
```

Confirm the setup:

```
ip a
```

You should now see `br0` with an IP, while `ens33` and `ens37` have no IPs.

![](2025-04-26-3---Installing-and-Configuring-Suricata-6.png)

## 5. Configuring Suricata for Bridged Operation

Once the bridge is up and running, now let tell Suricata to monitor the bridge interface (`br0`) instead of a specific NIC.

Edit the Suricata configuration:

```
sudo nano /etc/suricata/suricata.yaml
```

Find the `af-packet` section and replace it with:

```
af-packet:
  - interface: br0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
    use-mmap: yes
    tpacket-v3: yes
```

![](2025-04-26-3---Installing-and-Configuring-Suricata-7.png)

Explanation:

- `af-packet` is a high-performance packet capture method
- We're telling Suricata to monitor the `br0` interface
- `cluster-flow` improves multi-threading by keeping flows on the same thread

Restart Suricata to apply the config:

```
sudo systemctl restart suricata
```

## 6. Verifying the Setup

Check if Suricata is running:

```
sudo systemctl status suricata
```

![](2025-04-26-3---Installing-and-Configuring-Suricata-8.png)

Install bridge tools (optional, for debugging):

```
sudo apt install bridge-utils -y
brctl show
```

You should see a bridge `br0` with `ens33` and `ens37` as members.

![](2025-04-26-3---Installing-and-Configuring-Suricata-9.png)

To see if Suricata is receiving traffic, monitor the event log:

```
tail -f /var/log/suricata/eve.json | jq

		OR

tail -f -n 1 /var/log/suricata/eve.json | jq
```

If traffic is flowing, you’ll see logs with alerts, flows, and HTTP headers.

![](2025-04-26-3---Installing-and-Configuring-Suricata-10.png)

## 7. Sending Suricata Logs to Splunk

To centralize monitoring, we’ll send Suricata logs to Splunk using the **Universal Forwarder**.
Download the Splunk universal forwarder from Splunk website and install the forwarder using following commands:

```
# Install Splunk UF using dpkg
sudo dpkg -i splunkforwarder.deb

# Start the Splunk UF, accepting license (create a Administrator login for UF)
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

![](2025-04-26-3---Installing-and-Configuring-Suricata-11.png)

Create a index on Splunk Enterprise for Suricata logs.

![](2025-04-26-3---Installing-and-Configuring-Suricata-12.png)

Add the Suricata log as a monitored source:

```  
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/suricata/eve.json -index suricata_logs -sourcetype json
```

Forward logs to your Splunk indexer:

```
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.1.120:9997
```

Enable the forwarder to start at boot:

```
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

Now, you can search and correlate Suricata logs inside your Splunk dashboard.

![](2025-04-26-3---Installing-and-Configuring-Suricata-13.png)

## 8. Rerouting Traffic Through Suricata

Now that our Suricata bridge is active and monitoring traffic, it’s time to **redirect client and SOC machine traffic** to flow _through_ it — without making any major changes to their configuration.

### What Are We Changing?

During the earlier **pfSense setup**, you likely:

- Set **static IPs** on your client and SOC machines
- Assigned their **network adapters (NICs)** to the **pfSense LAN segment**

Now, we’ll make a **simple switch**:

- Change each machine’s **NIC assignment** from the **pfSense LAN segment** to the **Suricata LAN segment** (the interface connected to `ens37` on the Ubuntu bridge)
- **Leave the static IPs as-is** (since we’re still using the same subnet: `192.168.1.0/24`)


![](2025-04-26-3---Installing-and-Configuring-Suricata-14.png)

This works because:

- Clients still believe they’re communicating directly with pfSense (e.g., default gateway `192.168.1.110`)
- But in reality, their traffic now passes through the Suricata bridge first
- Suricata logs, analyzes, and alerts on every packet flowing between the clients and pfSense

**Note:** Suricata is completely transparent in this setup — no NAT, no routing changes. All Layer 2 bridging. This is a powerful technique for passive inspection without disrupting existing infrastructure.


## Confirm Connectivity & View Logs in Splunk

Now that traffic is flowing through Suricata, let’s verify everything is functioning correctly.

### 1. Test Network Connectivity

On each SOC/client machine:

- Ping the pfSense gateway (`192.168.1.110` and `pfsense.localdomain`)
- Perform DNS lookups or try accessing the internet (if allowed)
	- ping `8.8.8.8` and `www.google.com`

If all works, the bridge is forwarding traffic correctly.

### 2. Generate a Test Alert in Suricata

Run a known test signature such as the **ET Open Rule for EICAR test string** or use tools like `curl` to trigger common alerts:

```
curl http://testmyids.com
```

![](2025-04-26-3---Installing-and-Configuring-Suricata-15.png)

This domain is designed to trigger IDS/IPS rules.

### 3. Check for Logs in Splunk

On your Splunk instance:

- Search the `suricata` index
- Look for entries in the `eve.json` log file forwarded via the Universal Forwarder

Use a basic Splunk search like:

```
index=suricata alert
```

You should now see alert events with details on source/destination IPs, ports, and rule matches.

![](2025-04-26-3---Installing-and-Configuring-Suricata-16.png)

With connectivity verified and alerts flowing into Splunk, your IDS bridge is fully operational and ready for deeper tuning and visibility enhancements.

## Conclusion

With the Suricata installation and bridge configuration complete, our IDS sensor is now live in the network path — silently monitoring traffic between internal systems and the pfSense firewall. Acting as a transparent bridge, it inspects every packet in real time without disrupting connectivity or requiring changes to IP addressing. This forms a critical layer of visibility in our lab environment, allowing us to detect and log suspicious activity as it happens.

In the next step, we’ll introduce an **Endpoint Detection and Response (EDR)** solution using **OSSEC** on our client machines. This will give us deeper insight into host-level activity — such as unauthorized file changes, privilege escalations, and suspicious processes — helping to correlate network-based alerts with endpoint behavior and move toward a more complete and responsive security monitoring setup.

