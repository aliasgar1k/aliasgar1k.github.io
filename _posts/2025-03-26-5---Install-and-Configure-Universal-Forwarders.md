---
title: 5 - Install and Configure Universal Forwarders
date: 2025-03-26 00:00:00 +/-TTTT
categories:
  - Splunk SIEM Implementation
tags:
  - splunk-siem-implementation
image: ../preview-image.png
media_subpath: /assets/img/Splunk-SIEM-Implementation/2025-03-26-5---Install-and-Configure-Universal-Forwarders/
---
  
## Overview:
  
Universal Forwarders are lightweight Splunk components that collect and forward log data from client machines to a Splunk instance for indexing and analysis. You can find a detailed overview of the different types of Splunk forwarders, including Universal Forwarders and Heavy Forwarders, at this [Splunk Documentation page on Forwarders](https://docs.splunk.com/Documentation/Splunk/latest/Forwarding/Aboutforwarding).  
  
In our project, we have chosen Universal Forwarders due to their lightweight nature and efficiency in data collection. They consume minimal system resources while reliably transmitting logs from remote machines to the main Splunk server. This makes them ideal for environments where performance is crucial, such as in our SIEM setup, where we aim to monitor logs from both Windows and Linux client machines without introducing significant overhead.

## Step-by-Step Guide:

**NOTE: we would require to step all machines to static IP address to ensure reliable log collection and communication in the SIEM setup.** 

1. **Download the Universal Forwarder:**  
2. **Install the Universal Forwarder:**  
3. **Start the Universal Forwarder:**  

## Conclusion:

Configuring Universal Forwarders is essential for gathering logs from distributed client machines to a central Splunk instance. By following these straightforward steps, you can ensure that your Splunk environment is equipped to effectively monitor and analyze logs for better security insights.
