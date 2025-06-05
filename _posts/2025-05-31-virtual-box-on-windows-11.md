---
title: Virtual Box On Windows 11
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Homelabs
tags:
  - virtual-box
  - windows 
image: preview-image.png
media_subpath: /assets/img/Homelabs/2025-05-31-virtual-box-on-windows-11/
---

**NOTE: This is note a Guide on “How to install Virtual Box on Windows 11”, but this blog is to understand and resolve virtual box issuses on compitability with windows 11. for a guide on installation please visit :** [**nerdschalk.com**](https://nerdschalk.com/how-to-install-and-use-virtualbox-on-windows-11/)

Most modern-day chipsets cpus come with a virtualization capability built into the chipset. several operating systems take advantage of that and build a virtualization capability into their os. most notably that would be versions of linux. microsoft is joining the party on that and they’ve built something a feature called hyper-v.

Hyper-v runs in the operating system and it accesses the virtualization that’s built into the cpu chipset it then allows for user to build virtual mahcines and operate these on virtualization based security.

Now when virtualbox starts up virtualbox will sense that a hyper-v hypervisor is running and it will try to establish native vms through hyper-v. the problem is hyper-v blocks virtualbox from accessing it. again if you look at the documentation for virtualbox 6.1 you’ll see that this hyper-v interfacing is considered experimental and it doesn’t really work well.

Here if we start a virtual mahcine on virtual box like windows server 2019 but it would not open. crashing or hanging in middle would not start the actual OS. showing us the screen little turtle at the very bottom right corner which tells us detail about the hypervisor system it is utilizing while running the current VM. this show that its taking use of hyper-V which inturn is a type 1 virtualization techinque. while virtual box is supposed to be using type 2 virtualization for our VMs.

![](2025-05-31-virtual-box-on-windows-11-2.png)

the first image above shows how virtual box represents the type of virtualization technique it using while hyper-V. (with a symbol of a turtle) and in second image we are seeing how virtual box will show if its utilizing type 2 virtualization. (with a symbol of letter V)

we got here if you have a windows 10 pc or laptop hyper-v was built into intuit a feature is available but it wasn’t turned on by default. if you upgraded to windows 11 from your windows 10, windows 11 upgrade would look at your configuration see that hyper-v was turned off and not turn it on. however if you buy a brand new machine which comes with windows 11 as the default operating system. microsoft has required that all vartiualization capabilites (hyper-V) turn on as a default.

## SOLUTION

* Before we start we check if virtulization is running or not
* for this go to **WIN+R > msinfo32 > System Summary**
* here as we see virtualization in running and in last line it says **“A hypervisor has been detected”**.
* thus it hyper-v is running.

![](2025-05-31-virtual-box-on-windows-11-3.png)

### Step 1: Fix boot

This is because when you do a shutdown, what happens here is the Windows will cache your current state and it will save it and then it reinstates that on next boot up.

* start menu
  * control panel
    * system and security
      * power options
        * choose what the power button does
          * choose settings that are currently unavailable
            * turn off fast startup
              * save changes

![](2025-05-31-virtual-box-on-windows-11-4.png)

### Step 2: Turn off hyper-V feature

* start menu
  * settings
    * apps
      * optional feature
        * more windows features
          * uncheck hyper-V
            * ok
              * restart now

once rebooted you can check if changes we made have actually taken place by gonig again same “optional feature > more windows features” and checking if hyper-v is unchecked.

![](2025-05-31-virtual-box-on-windows-11-5.png)

![](2025-05-31-virtual-box-on-windows-11-6.png)

### Step 3: Turning Off Memory Integrity

* start menu
  * core isolation
    * turn off memory integrity

### Step 4: Change registry settings

* win+r
  * regedit
    * Computer\HKEY\_LOCAL\_MAHCINE\SYSTEM\CurrentControlSet\Control\DeviceGuard
      * set EnableVirtualizationBasedSecurity = 0

![](2025-05-31-virtual-box-on-windows-11-7.png)

Now here we have in DeviceGuard we have Scenarios. In Scenarios we have to go every folder and turn Enabled = 0

```
DeviceGuard > Scenarios > CredentialGuard > Enabled = 0
DeviceGuard > Scenarios > HyperVisorEnforcedCodeIntegrity > Enabled = 0
DeviceGuard > Scenarios > KernelShadowStacks > Enabled = 0
DeviceGuard > Scenarios > SystemGuard > Enabled = 0
```

![](2025-05-31-virtual-box-on-windows-11-8.png)

Once completed above step do complete shutdown and then restart the PC with power button.

now our hyper-v should be turned OFF with all of its virtualization capabilites, allowing us to smoothly run VMs on vitual box on our windows 11 machine.\
to check this and confirm we can go to **WIN+R > msinfo32 > SystemSummary**\
here we will see all virtualization settings says **“not enabled”** and also no hypervisor is detected.\
thus now if you run windows server 2019 in your virtual box it will run with a letter V symbol and without any problems like lagging or hanging.

![](2025-05-31-virtual-box-on-windows-11-9.png)
