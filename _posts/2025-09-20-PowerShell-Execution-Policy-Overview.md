---
title: PowerShell Execution Policy Overview
date: 2025-09-20 00:00:00 +/-TTTT
categories:
  - Others
tags:
  - powershell
  - gpo
  - get-executionpolicy
  - set-executionpolicy
  - gpedit
  - gpupdate
  - get-history
  - get-psreadlineoption
image: preview-image.png
media_subpath: /assets/img/Others/2025-09-20-PowerShell-Execution-Policy-Overview/
---


## Introduction

As a long-time Visual Studio Code (VS Code) user, I often encounter a frustrating issue: when I open VS Code, it fails to start the Python environment automatically. Instead, it throws an error message like the one shown below:

![](2025-09-20-PowerShell-Execution-Policy-Overview-1.png)

This error reminded me of similar problems I’ve faced while working with PowerShell scripts, especially in penetration testing (Pentest) or Capture The Flag (CTF) scenarios. I've also encountered environments where PowerShell is completely restricted. This means not only are you unable to run scripts, but you also can't open the PowerShell prompt or even run basic commands.

After some investigation, I discovered that this issue stems from a security feature called the **PowerShell Execution Policy**. Let’s dive into what it is, how it works, and how you can resolve such issues.

## What is PowerShell Execution Policy?

The **Execution Policy** in PowerShell is a security feature designed to prevent the execution of potentially harmful scripts. It acts as a safeguard, especially in corporate environments, against attackers who may try to run malicious scripts.

Enterprises often use **Group Policy** and **AppLocker** to control the execution of scripts on systems, ensuring that unauthorized or unsafe scripts are not run. In some cases, organizations may even completely restrict PowerShell access for users.

### Default Execution Policy

On Windows client machines, the default execution policy is set to **Restricted**, which prevents all scripts from running. Here’s a quick overview of the various execution policies:

- **Unrestricted**: Allows all scripts to run, but will prompt you before executing unsigned scripts downloaded from the internet.
- **RemoteSigned**: Requires that scripts downloaded from the internet be signed by a trusted publisher. Local scripts can run without a signature.
- **AllSigned**: Requires all scripts (even local ones) to be signed by a trusted publisher.
- **Restricted**: No scripts can run. This is the default setting for Windows clients.
- **Default**: The default execution policy for the system. For clients, this is **Restricted**, and for servers, it is **RemoteSigned**.
- **Bypass**: Allows all scripts to run without any warnings or prompts.
- **Undefined**: Indicates that no execution policy has been explicitly set. If no policy is set across all scopes, the effective policy defaults to **Restricted** on Windows clients.

## How to Check Current Execution Policy

To check your current execution policy, run the following command in PowerShell:

```powershell
Get-ExecutionPolicy
```

To view all execution policies for different scopes, run:

```powershell
Get-ExecutionPolicy -List
```

![](2025-09-20-PowerShell-Execution-Policy-Overview-2.png)

### Different Execution Policy Scopes

PowerShell execution policies can be applied at different scopes. Here's a breakdown of each:

- **MachinePolicy**: Set by Group Policy for all users on the computer. This policy overrides all other scopes.
- **UserPolicy**: Set by Group Policy for the current user. It overrides **Process**, **LocalMachine**, and **CurrentUser** but is overridden by **MachinePolicy**.
- **Process**: Applies only to the current PowerShell session. It is temporary and will revert once the session ends.
- **LocalMachine**: The default scope. Affects all users on the computer and requires administrative privileges to modify.
- **CurrentUser**: Affects only the current user and is stored in the registry under **HKEY_CURRENT_USER**.

## Changing the Execution Policy

### Changing the Execution Policy for a Single PowerShell Session

If you just want to bypass the policy for a single session, you can use the following command:

```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
```

This change will only affect the current session and will be lost once the session is closed.

#### Resetting Execution Policy to Default

To reset the execution policy to its default setting, use the following command:

```powershell
Set-ExecutionPolicy -Scope CurrentUser Default
```

### Changing PowerShell Execution Policy via Group Policy Editor

A more convenient way to change the execution policy is through the **Group Policy Editor**.

#### Steps to Configure PowerShell Execution Policy

1. **Open the Local Group Policy Editor**:
    
    - Press `Windows + R` to open the **Run** dialog, type `gpedit.msc`, and press **Enter**.
    
![](2025-09-20-PowerShell-Execution-Policy-Overview-3.png)
![](2025-09-20-PowerShell-Execution-Policy-Overview-4.png)

2. **Navigate to PowerShell Execution Policy Settings**:
    
    - Go to:
        - **Computer Configuration** or **User Configuration** (depending on your needs).
        - **Administrative Templates** > **Windows Components** > **Windows PowerShell**.
    
![](2025-09-20-PowerShell-Execution-Policy-Overview-5.png)
    
3. **Configure the Execution Policy**:
    
    - Double-click on **Turn on Script Execution**.
    - Select **Enabled** and choose the desired execution policy from the dropdown:
        - **Allow only signed scripts**: equivalent to **AllSigned** mode.
        - **Allow local scripts and remote signed scripts**: equivalent to **RemoteSigned** mode.
        - **Allow all scripts**: equivalent to **Unrestricted** mode.
    
    You can also select **Disabled** to completely block script execution.
    
![](2025-09-20-PowerShell-Execution-Policy-Overview-6.png)


4. **Apply and Refresh Group Policy**:

    - Click **OK** to apply your changes.
    - Windows automatically refreshes Group Policy settings at regular intervals (default: 90 minutes). To apply immediately, run:

```powershell
gpupdate /force
```

![](2025-09-20-PowerShell-Execution-Policy-Overview-7.png)

5. **Verify the PowerShell Execution Policy**:
    
    - Open **PowerShell** and run:

```powershell
Get-ExecutionPolicy -List
```

#### Reverting Changes

If you want to revert any changes, simply go back to the **Group Policy Editor**, and set the policy to **Not Configured** or **Disabled**.

![](2025-09-20-PowerShell-Execution-Policy-Overview-8.png)

### Managing Execution Policies in Enterprises

For organizations with **thousands of computers** (enterprise environments), managing execution policies (and other settings) across the network is done at scale using **Active Directory Group Policy** (GPO) rather than configuring individual systems manually. The process becomes much more centralized and automated, leveraging Group Policy Objects (GPOs) that apply to a large set of computers and users (simply an Oragnizational Unit (OU)).

In large organizations, Group Policy is the primary tool for managing system-wide settings across a domain. This is done using **Group Policy Management Console (GPMC)**. You can configure policies for computers and users centrally, so you don't need to visit each system individually.

#### Steps for Configuring Execution Policy in an Organization

1. **Open Group Policy Management Console (GPMC)**:
    
    - Go to: **Start Menu > Administrative Tools > Group Policy Management**.
    - Another way if from the **Server Manager > Tools > Group Policy Management**.
    
![](2025-09-20-PowerShell-Execution-Policy-Overview-9.png)
    
2. **Create or Edit a GPO**:
    
    - Right-click **Group Policy Objects** and select **New** to create a new GPO.
    - Name the GPO (e.g., **PowerShell Execution Policy**).
    
![](2025-09-20-PowerShell-Execution-Policy-Overview-10.png)
    
1. **Open the Execution Policy Settings**:

	- Right Click the newly created **Group Policy Object** and select **Edit**. This will open the **Group Policy Management Editor**.

![](2025-09-20-PowerShell-Execution-Policy-Overview-11.png)

- In **Group Policy Management Editor** and go to:

```
Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options
```

- Alternatively, for more granular control, you can configure execution policies through: 

```
Computer Configuration > Policies > Administrative Templates > Windows Components > Windows PowerShell > Turn on Script Execution
```

![](2025-09-20-PowerShell-Execution-Policy-Overview-12.png)

- Enable the policy and select the desired execution level.

4. **Configure the Policy**:
	- Double Click **Turn on Script Execution** Setting.
	- Choose **Enable** checkbox to enable the settings.
    - Select the policy to one of the available execution levels (Restricted, AllSigned, RemoteSigned, Unrestricted, Bypass, etc.).
    - Click **Apply** and **OK** to continue.

![](2025-09-20-PowerShell-Execution-Policy-Overview-13.png)

![](2025-09-20-PowerShell-Execution-Policy-Overview-14.png)

5. **Link the GPO to Organizational Units (OUs)**:
    
	- Once the GPO is configured, it must be **linked** to the appropriate OUs that contain the computers and/or users you want the policy to apply to.
    - Right-click on the OU where your target machines or users reside and select **Link an Existing GPO**. Choose the GPO you just created.

![](2025-09-20-PowerShell-Execution-Policy-Overview-15.png)
![](2025-09-20-PowerShell-Execution-Policy-Overview-16.png)
![](2025-09-20-PowerShell-Execution-Policy-Overview-17.png)

6. **Force Group Policy Update**:
    
    - On client machines, run:

```powershell
gpupdate /force
```

![](2025-09-20-PowerShell-Execution-Policy-Overview-18.png)

- Restart the machine to apply the changes.

![](2025-09-20-PowerShell-Execution-Policy-Overview-19.png)

## Additional Restrictions for PowerShell Access

In some organizations, PowerShell itself is completely disabled using more advanced methods like **AppLocker** or **Windows Defender Application Control (WDAC)**. These tools provide granular control over which binaries can be executed on the system.

- **AppLocker** allows administrators to control which applications can run based on rules.

![](2025-09-20-PowerShell-Execution-Policy-Overview-20.png)

- **WDAC** is a more advanced feature used to lock down system configurations and restrict app execution.

![](2025-09-20-PowerShell-Execution-Policy-Overview-21.png)

For larger enterprises, administrators may use tools such as **System Center Configuration Manager (SCCM)** or **Microsoft Intune** for even more control and flexibility over large-scale settings deployment. These tools allow for more granular management across desktops, laptops, and servers within an organization. 

**SCCM (System Center Configuration Manager)**: now known as [Microsoft Endpoint Configuration Manager](https://www.google.com/search?client=firefox-b-d&sa=X&sca_esv=53bbb5633a515c6e&biw=1536&bih=731&q=Microsoft+Endpoint+Configuration+Manager&ved=2ahUKEwjmx6yDxZuPAxXOSWwGHYuYKRYQxccNegQIJBAB&mstk=AUtExfB4NlVRxHfV1A9Dk2_LGFbY52Xkd_P5tP3QCDR8nN5YxrNOP9nSosz174LQOIfRXTf6XlfmvzdOgHyI_uU80A2bcgFRlIWM89DwGxKM9ExVm9y5iIAPvCiCJyafJGlW2H8&csui=3), is a systems management software developed by Microsoft. It enables IT administrators to manage, deploy, and secure devices, deploy GPOs, scripts and applications to devices across an organization's network. It also provides reporting and monitoring capabilities. 

![](2025-09-20-PowerShell-Execution-Policy-Overview-22.png)

**Microsoft Intune**: If your organization is using modern management and has devices enrolled in Intune (especially for cloud-managed devices), you can apply configuration profiles to control execution policies across your devices.

![](2025-09-20-PowerShell-Execution-Policy-Overview-23.png)

## PowerShell Logging and Auditing

Even if you bypass execution policies, remember that everything you do in PowerShell is logged for auditing purposes.

PowerShell handles command history in a nuanced way, and the extent of logging depends on the specific configuration and version in use.

**Session History**:

- By default, PowerShell maintains a session history that tracks commands executed within a single PowerShell session. This history is stored in memory and is typically lost when the session is closed. The number of entries in this history is controlled by the `$MaximumHistoryCount` preference variable.
- The following cmdlet can be used to view this session-specific history. 

```powershell
Get-History
```

**Persistent History (via PSReadLine)**:

- Modern versions of PowerShell, particularly when using the PSReadLine module (which is included by default in Windows 10 and later), provide a persistent command history.
- This history is saved to a file and persists across PowerShell sessions, allowing you to recall commands from previous sessions.
- The default powershell history file location is:

```
C:\Users\username\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

If in some case your its not in the intended place, you can always check the file location using following command:

```powershell
(Get-PSReadlineOption).HistorySavePath
```

**Logging for Auditing and Forensics:**

- While PSReadLine provides a persistent history for user convenience, for more comprehensive logging and security auditing purposes, PowerShell offers features like Script Block Logging and Module Logging.

- **Script Block Logging**: records blocks of code as they are executed by the PowerShell engine, capturing the full content of scripts and commands. This is often enabled by default. 

- **Module Logging**: records portions of scripts and de-obfuscated code, though it is typically disabled by default.

- These logging features write events to the Windows Event Log, which can be viewed in Event Viewer or accessed using cmdlets like `Get-WinEvent` or `Get-EventLog`.

In addition to these built-in logging methods, PowerShell activity can also be tracked through **Audit Policy** settings and **SIEM (Security Information and Event Management)** tools. Enabling audit policies, such as **Logon/Logoff** and **Privilege Use**, can provide crucial information about who executed PowerShell commands and when, even if Script Block logging and Module logging is disabled. **Sysmon** can also capture low-level details about PowerShell activity, including process creation, network connections, and file modifications, regardless of PowerShell's internal logging settings. Even if PowerShell logging and audit policies are turned off, SIEM tools like **Splunk** or **Elastic Stack** can still aggregate and analyze data from various sources, such as **Windows Event Logs**, **Sysmon**, and **Windows Defender ATP**. These tools can collect details like process execution, command-line arguments, and other system activities, helping identify suspicious behavior and enabling incident response teams to detect and investigate malicious activity.

## Conclusion

In summary, while PowerShell’s default session history is not persistent, modern versions with PSReadLine allow for a persistent command history, ensuring convenience for users. However, more advanced logging features like Script Block Logging and Module Logging are enabled by default in many PowerShell environments for security and auditing purposes, capturing all executed commands and script content.

While bypassing execution policies can be an effective workaround to resolve certain issues, it's crucial to remember that everything you do in PowerShell is logged. These logs can be crucial for auditing or forensic analysis, especially in sensitive or security-conscious environments. Always execute commands with caution, as actions taken may be recorded and later reviewed.

**References and Supplementary Readings**:

- [Set PowerShell Execution Policy with Group Policy in Windows Server 2022](https://www.youtube.com/watch?v=X-ilgIG1Xao)
- [15-ways-to-bypass-the-powershell-execution-policy](https://www.netspi.com/blog/technical-blog/network-pentesting/15-ways-to-bypass-the-powershell-execution-policy/)
- [Windows Powershell & cmd pentesting commands Cheat Sheet](https://glasgowned.github.io/PenTesting/Windows/Powershell/)
- [Windows PowerShell Pivoting and General Pentesting Tricks](https://agrohacksstuff.io/posts/windows-powershell-pivoting-and-general-pentesting-tricks/)
