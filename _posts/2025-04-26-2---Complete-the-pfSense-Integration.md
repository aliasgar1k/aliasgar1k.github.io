---
title: 2025-04-26-2---Complete-the-pfSense-Integration
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation Upgrade 1
tags:
  - splunk-siem-implementation-Upgrade-1
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation-Upgrade-1/2025-04-26-2---Complete-the-pfSense-Integration/
---


---
title: Complete the pfSense Integration
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation Upgrade 1
tags:
  - splunk-siem-implementation-Upgrade-1
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation-Upgrade-1/Complete the pfSense Integration/
---



In the previous guide, we covered the essentials of **Installing and Configuring pfSense**. Now that the firewall is up and running, it’s time to take the next step which focuses on integrating pfSense with **Splunk** for centralized logging, enabling access to internal services like SSH and RDP from outside the network, and simulating remote attacker behavior using Kali Linux.

## Centralizing Logs

In any enterprise network, logs are the foundation of security monitoring. They tell the story of what's happening across systems: what’s allowed, what’s blocked, who’s logging in, and whether anything suspicious is going on. But raw logs sitting on individual devices don’t help much on their own.

Thus we will be sending pfSense logs to **Splunk**, giving us a centralized platform to collect, index, and analyze logs in real time. By sending firewall and IDS logs from pfSense to Splunk, we enable detection, alerting, and forensic capabilities that are impossible without a unified view.

## Preparing Splunk to Receive Logs

Before sending any data, Splunk needs to be configured to accept it. This method is a bit different from Splunk universal forwarder. As here we setup remote sysloging on pfSense and setup a receiving port on Splunk specifying types of data that is to be send to it. Here's how to set up Splunk to receive syslog data over UDP:

### Create Dedicated Indexes

Navigate to `Settings → Indexes` and create a index named as `pfsense_logs`. 

![](2025-04-26-2---Complete-the-pfSense-Integration-1.png)

### Enable UDP Input on Port 514

To receive logs from pfSense, Splunk needs to listen for incoming syslog data over the network. This is done by configuring a **UDP data input**. Syslog typically uses **UDP port 514**, which pfSense uses by default for log forwarding. Follow these steps to configure Splunk:

**Navigate to the UDP Input Settings**
In the Splunk Web interface:

```
Settings → Data Inputs → Local Inputs → UDP → Add New
```

This section lets you define how Splunk listens for raw log data sent over the network via UDP.

**Configure the UDP Input**
On the "Add New UDP" input screen, enter the following:

|Field|Value|Explanation|
|---|---|---|
|**Port**|`514`|This is the default syslog port used by pfSense and most network devices.|
|**Source name override**|_(Leave blank)_|Allows you to override the source name. We leave it blank to let Splunk determine it automatically.|
|**Only accept connections from**|_(Leave blank)_|Restricts log sources by IP. Leaving it blank allows logs from any device, which is fine in a controlled lab.|
Click **Next** to proceed.

**Set Input Parameters**
You’ll now specify how Splunk should interpret the incoming data:

| Field              | Value                                     | Explanation                                                                                     |
| ------------------ | ----------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Set sourcetype** | `syslog`                                  | This tells Splunk to parse the logs using standard syslog formatting.                           |
| **App Context**    | `Search & Reporting`                      | Determines where in Splunk the input is managed. The default is appropriate for most use cases. |
| **Host**           | _(Leave as default or specify static IP)_ | Identifies the source system. You can let Splunk auto-detect or specify the pfSense IP.         |
| **Select Index**   | `pfsense_logs`                            | Choose the index you previously created to store pfSense logs separately from other data.       |

Finally, click **Save**.

Splunk is now listening for incoming syslog data on port 514. Once pfSense is configured to send logs to this IP and port, logs should begin appearing in the `pfsense_logs` index automatically.

![](2025-04-26-2---Complete-the-pfSense-Integration-2.png)

### Open Port on the Host Firewall

Ensure your Splunk host's firewall allows incoming traffic on **UDP 514** by setting up a firewall rule for it.

![](2025-04-26-2---Complete-the-pfSense-Integration-3.png)

## Configuring pfSense to Send Logs to Splunk

With Splunk ready to receive data, pfSense needs to be instructed to forward its logs:

- **Access pfSense GUI** and navigate to: `Status → System Logs → Settings`
- **Enable Remote Logging**: Check the checkbox for "Send log messages to remote syslog server"
- **Source Address**: Default (any)
- **IP Protocol**: IPv4
- **Remote log server**: Enter your Splunk server’s IP in the following format: `192.168.1.120[:514]`
- **Remote Syslog Contents**: System Events, Firewall Events, DNS Events, DHCP Events, General Authentication Events, VPN Events  Gateway Monitor Events.

Choose which logs to send based on relevance and if you cannot decide on it, use the below table as reference.

| Log Type                   | Send to Splunk?       | Why?                                                                                            |
| -------------------------- | --------------------- | ----------------------------------------------------------------------------------------------- |
| **System Events**          | ✅ Yes                | Logs system health, services, errors. <br>Important for monitoring device stability.            |
| **Firewall Events**        | ✅✅✅ Must          | Tracks allow/deny traffic, NAT, rules hit. <br>**Core data** for threat detection!              |
| **DNS Events**             | ✅ Highly Recommended | DNS abuse, tunneling, C2 traffic detection starts here. <br>Very useful.                        |
| **DHCP Events**            | ✅ Optional           | Good for correlating IP addresses to devices over time. <br>(Helpful but not critical.)         |
| **PPP Events**             | ❌ Skip               | Only if using WAN via PPPoE (rare today). <br>Not needed for most setups.                       |
| **Authentication Events**  | ✅ Recommended        | Tracks successful/failed logins into pfSense <br>(admin activity, possible brute-force).        |
| **Captive Portal Events**  | ❌ Skip               | Only needed if you use captive portals (like for hotels, cafés).                                |
| **VPN Events**             | ✅ Recommended        | VPN tunnels up/down, client connections. <br>very important if you use VPNs.                    |
| **Gateway Monitor Events** | ✅ Optional           | Monitors WAN link health. <br>Good for diagnosing outages but not urgent for threat monitoring. |
| **Routing Daemon Events**  | ❌ Skip               | Only if using dynamic routing protocols <br>(rare in small/medium labs).                        |
| **NTP Events**             | ❌ Skip               | Only useful if diagnosing time sync issues. <br>Otherwise unnecessary.                          |
| **Wireless Events**        | ❌ Skip               | Only if pfSense is acting as a wireless AP. <br>Otherwise irrelevant.                           |

- **Save** your settings.

![](2025-04-26-2---Complete-the-pfSense-Integration-4.png)

## Verifying Logs in Splunk

Once configured, logs should begin appearing in Splunk within seconds. To verify:

- Open the **Search** interface in Splunk.
- Run the query:
 
 ```
 index="pfsense_logs" | head 20
```

- You should see logs with fields like `host`, `sourcetype`, and `log_level`.

![](2025-04-26-2---Complete-the-pfSense-Integration-5.png)

If no logs appear, double-check that the port is open, and that the remote logging IP and port are correct in pfSense.

## Exposing Internal Services to the Outside (Safely)

For client machines using remote access, it's often necessary to expose internal services like RDP or SSH. We can do this using 2 different methods: Port Forwarding and Routing + Firewall rules.

| Use Case                                                     | Method                       | Example Command                |
| ------------------------------------------------------------ | ---------------------------- | ------------------------------ |
| Access internal server via pfSense WAN IP                    | **Port Forwarding**          | `ssh user@<pfSense_WAN_IP>`    |
| Access internal server via internal IP from external network | **Routing + Firewall Rules** | `ssh user@<Client_Machine_IP>` |

### Port Forwarding

Here’s how to set up port forwarding:

- Go to `Firewall → NAT → Port Forward`.
- Add a new rule:
    - **Interface**: WAN
    - **Protocol**: TCP
    - **Destination Port Range**: e.g., `3389` for RDP or `22` for SSH
    - **Redirect Target IP**: Internal IP of your Windows/Linux machine
    - **Redirect Target Port**: Match the service (e.g., 3389 or 22)

- Save and apply the changes.
- Create a corresponding firewall rule to **allow** this traffic.

### Routing + Firewall Rules

In production environments, this is a security risk. In labs, it's a powerful tool—but use it responsibly. But as we are trying to simulate real world environment in our project. we would use the second method of routing + firewall rules, which presents in a way that makes it files like our Kali Linux attacker machine is in victim network just as a regular employee would be using a VPN in an organization.

Here’s how to set up firewall rules:

- Open pfSense Web GUI.
- Go to **Firewall > Rules > WAN**
- Add a rule:
    - **Action**: Pass
    - **Interface**: WAN
    - **Protocol**: TCP
    - **Source**: `any` or your Kali IP/subnet
    - **Destination**: `192.168.1.130`
    - **Destination port**: `3389`

Here’s how to set up routing:

- Open Terminal in Kali Linux
- Add a route:

```
sudo ip route add 192.168.1.0/24 via 192.168.125.80
```

This tells Kali: "Send all packets destined for 192.168.1.0/24 (including 192.168.1.130) to pfSense at 192.168.125.80.”

**Note**: this route is temporary and wouldn't sustain over system reboots. for a simply soultion add it to your `.bashrc` or `.zshrc` file.

## Conclusion

By integrating pfSense with Splunk, forwarding key log types, and setting up both internal access and simulated remote routing, you've taken a major step toward a realistic lab setup. You now have a Real-time logs flowing into Splunk for visibility and analysis with Controlled external access to lab services.
