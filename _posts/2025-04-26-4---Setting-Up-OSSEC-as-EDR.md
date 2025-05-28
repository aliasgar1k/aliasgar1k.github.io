---
title: 2025-04-26-4---Setting-Up-OSSEC-as-EDR
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation Upgrade 1
tags:
  - splunk-siem-implementation-Upgrade-1
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation-Upgrade-1/2025-04-26-4---Setting-Up-OSSEC-as-EDR/
---


---
title: Setting Up OSSEC as EDR
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation Upgrade 1
tags:
  - splunk-siem-implementation-Upgrade-1
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation-Upgrade-1/Setting Up OSSEC as EDR/
---



https://www.youtube.com/watch?v=A4RrDpcvKf8
https://www.youtube.com/watch?v=7c8xowHz0Ko&t=516s

In our previous blog series Splunk SIEM Implementation, we set up a basic security monitoring system with Splunk, Pfsense, Suricata, Sysmon, etc. Today, we’re going to take it a step further and install OSSEC, an open-source EDR (Endpoint Detection and Response) solution, to monitor our Windows and Linux clients. We will be installing the OSSEC server on the Suricata machine, and logs will be forwarded to Splunk via the Universal Forwarder (UF), just like we did with Suricata logs.

## What is an EDR?

EDR stands for **Endpoint Detection and Response**. It is a type of security solution designed to monitor, detect, and respond to suspicious activity on endpoint devices (like computers and servers). EDR tools often provide features such as real-time monitoring, advanced threat detection, and incident response automation.

## What is OSSEC?

**OSSEC** is an open-source, host-based intrusion detection system (HIDS) that functions as an EDR. It helps in log analysis, file integrity checking, rootkit detection, real-time alerting, and active response to threats. OSSEC is highly customizable and supports Windows, Linux, and other UNIX-based systems.

## Why OSSEC?

OSSEC is an excellent choice for home-lab environments for several reasons:

- **Free and Open Source**: OSSEC is completely free and can be customized to suit your specific needs.
- **Comprehensive Security Features**: It provides features like log analysis, file integrity checking, and real-time monitoring.
- **Cross-Platform Support**: OSSEC supports a wide range of platforms, including both Windows and Linux.
- **Scalability**: Ideal for environments of any size, from single machines to enterprise setups.

Now, let’s dive into the setup process.

## Step 1: Download and Install OSSEC Server

- As said earlier, we will be installing OSSEC server on Suricata machine, putting a single machine to be used for multiple purposes. Note in real world environment this server would be hosted on its own separate machine.
- So head over to Suricata Machine and download the latest stable version of OSSEC for Linux.
 
```
wget https://github.com/ossec/ossec-hids/releases/download/3.6.0/ossec-hids-3.6.0.tar.gz
```

- Once the download is complete, extract the tar file and move into its directory.

```
tar -xvzf ossec-hids-3.6.0.tar.gz
cd ossec-hids-3.6.0
```

- Before starting the installation, install dependencies:

```
sudo apt install -y build-essential libssl-dev libpcap-dev zlib1g-dev libpcre3-dev libpcre2-dev libsystemd-dev
```

- Now begin the installation by running following command:

```
sudo ./install.sh
```

- Follow the prompts to complete the installation as such: 

```
1- What kind of installation do you want (server, agent, local, hybrid or help)? server

2-  Setting up the installation environment.
	- Choose where to install the OSSEC HIDS [/var/ossec] : ENTER
	- The installation directory already exists. Should I delete it? (y/n) [y] : y

3- Configuring the OSSEC HIDS.
	3.1- Do you want e-mail notification? (y/ n) [y]: n
	3.2- Do you want to run the integrity check daemon? (y/ n) [y]: y
	3.3- Do you want to run the rootkit detection engine? (y/ n) Y
	3.4:
		- Do you want to enable active response? (y/n) [y]: y
		- Do you want to enable the firewall-drop response? (y/n) [y] : y
		- Do you want to add more IPS to the white list? (y/n)? [n]: n
		- Do you want to add more IPS to the white list? (y/n)? [n]: n
	3.5- Do you want to enable remote sys log (port 514 udp)? (y/n) [y] : y
	3.6- Setting the configuration to analyze the following logs: ENTER

ENTER
```

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-1.png)

- After installation, you’ll need to modify the `ossec.conf` file to customize your server settings. This file is located in `/var/ossec/etc/ossec.conf`. Open it for editing:

```
sudo nano /var/ossec/etc/ossec.conf
```

- Make any necessary changes for your environment (e.g., disable email notification, etc.).

```
---snip---
		<email_notification>no</email_notification>
---snip---
```

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-2.png)

- Once configuration is set, restart the OSSEC server using following command:

```
sudo /var/ossec/bin/ossec-control start
```

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-3.png)

- Verify the status:

```
sudo /var/ossec/bin/ossec-control status
```

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-4.png)

## Step 2: Add OSSEC Agent and Extract Key

Once the OSSEC server is up and running, we need to generate an agent key for each client. 
This is because on each client (Windows/Linux) where you want to install OSSEC agents, you'll need to add the client machine to the OSSEC server. This is done by adding a Name for the client, machine's IP address, and a agent ID. After which an authentication key for the agent is generated.

Run the following command to add an agent:

```
sudo /var/ossec/bin/manage_agents
```

Follow the prompts to generate the agent's configuration:

- To **Add an agent**, select `A` and provide with agent name, client IP address and agent ID.
- To **List agents**, select `L` , which will show our newly added agent.
- To **Remove an agent**, select `R`, incase you made a mistake putting in some details.
- To **Extract key for an agent**, select `E`, and give in agent ID of the agent you want to extract keys for.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-5.png)

Once an agent is added and its key is extracted (save the key for later use), you will need to install the OSSEC agent on your client machines and configure them to connect to your OSSEC server.

## Step 3: Setup OSSEC Agent on Windows Client Machine

### Download and install OSSEC Agent

- Head to the [OSSEC download page](https://www.ossec.net/download/) and download the Windows installer.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-6.png)

- Run the installer and follow the installation steps. (simple Next > I Agree > Next > Install > Next > Finish)

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-7.png)

### Setup Agent

- Once installation is complete, run the OSSEC agent manager as Administrator from the start menu.
- Now enter in the **OSSEC server IP** (the bridge NIC IP address from Suricata machine) and the **Agent Key** that was generated earlier and click "Save".

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-8.png)

- A confirmation prompt will appear on the screen, cross check your agent information and click "Ok" to continue.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-9.png)

- Now from "Manage" menu and click on "Start OSSEC" to start the agent.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-10.png)

### Configure Agent

- Once the agent is started you can view the logs from "View > View Logs" and view/change configs from "View > View config".

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-11.png)

- by default active response is disable on agents, so we need to enable it in config by specifying no under disable for active response.

```
---snip---
	<active-response>
		<disabled>no</disabled>
	</active-response>
---snip---
```

**Note**: if config is changed, restart is necessary so restart from "Manage > Restart".

### Confirm Agent Connection

- To confirm the Windows agent is properly connected to the OSSEC server, check the agent status:

```
sudo /var/ossec/bin/agent_control -lc
```

- You should see the Windows machine listed in here and showing active as such.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-12.png)

## Step 4: Setup OSSEC Agent on Linux Client Machine

### Download and install OSSEC Agent

- Installation of OSSEC agent is same as we did for OSSEC server on our suricata machine.
- So head over to Linux Client Machine and download the latest stable version of OSSEC for Linux.
 
```
wget https://github.com/ossec/ossec-hids/releases/download/3.6.0/ossec-hids-3.6.0.tar.gz
```

- Once the download is complete, extract the tar file and move into its directory.

```
tar -xvzf ossec-hids-3.6.0.tar.gz
cd ossec-hids-3.6.0
```

- Before starting the installation, install dependencies:

```
sudo apt install -y build-essential libssl-dev libpcap-dev zlib1g-dev libpcre3-dev libpcre2-dev libsystemd-dev
```

- Now begin the installation by running following command:

```
sudo ./install.sh
```

- Follow the prompts to complete the installation as such: 

```
1- What kind of installation do you want (server, agent, local, hybrid or help)? agent

2-  Setting up the installation environment.
	- Choose where to install the OSSEC HIDS [/var/ossec] : ENTER

3- Configuring the OSSEC HIDS.
	3.1- what's the IP address or hostname of the OSSEC HIDS server?: 192.168.1.103
	3.2- Do you want to run the integrity check daemon? (y/ n) [y]: y
	3.3- Do you want to run the rootkit detection engine? (y/ n) Y
	3.4- Do you want to enable active response? (y/n) [y]: y

ENTER
```

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-13.png)

### Setup Agent

- Once the agent is installed successfully, now we need to setup the agent by giving in the previous extracted key for Linux agent.
- Run the following command to import the key for agent:

```
sudo /var/ossec/bin/manage_agents
```

- Follow the prompt as such:
	- To **Import key form the server**, select `I` and provide with agent key and press ENTER.
	- To **Quit**, select `Q` , which will exit the program.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-14.png)
### Configure Agent

- Before we finalize the install and restart the OSSEC, we need to configure and set email notification to be disabled.
- you can edit the configuration file at `/var/ossec/etc/ossec.conf` using following command:

```
sudo nano /var/ossec/etc/ossec.conf
```

- Once configuration is set, restart the OSSEC server using following command:

```
sudo /var/ossec/bin/ossec-control start
```

- Verify the status:

```
sudo /var/ossec/bin/ossec-control status
```

### Confirm Agent Connection

- To confirm the Linux agent is properly connected to the OSSEC server, check the agent status:

```
sudo /var/ossec/bin/agent_control -lc
```

- You should see the Linux client machine listed in here and showing active as such.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-15.png)

## Step 5: Use Splunk UF to Send OSSEC Logs to SIEM

We want to forward the OSSEC logs to Splunk in the same way we did with Suricata logs. To do this, you’ll need to configure Splunk to monitor the `/var/ossec/logs` directory.

- Create a Index for `ossec_logs` in Splunk Enterprise to store all incoming OSSEC logs.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-16.png)

- Run the following command to add the OSSEC log directory for monitoring in Splunk UF.

```
sudo /opt/splunkforwarder/bin/splunk add monitor -source /var/ossec/logs -index ossec_logs
```

- Restart the Splunk Universal Forwarder in order to take affect of changes.

```
sudo /opt/splunkforwarder/bin/splunk restart
```

- Go to your Splunk dashboard and check the `ossec_logs` index to confirm that the OSSEC logs are being forwarded correctly.

![](2025-04-26-4---Setting-Up-OSSEC-as-EDR-17.png)

## Conclusion

By following these steps, you’ve successfully installed and configured OSSEC as an EDR on your Windows and Linux clients, with the server running on your Suricata machine. You’ve also set up Splunk to ingest OSSEC logs for centralized monitoring and analysis. This setup adds an extra layer of protection and monitoring to your homelab, making it more robust and secure.

Stay tuned for our next blog post, where we’ll dive deeper into OSSEC alerting and advanced configuration!
