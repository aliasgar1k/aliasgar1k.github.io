---
title: Active Directory Home-Lab Setup
date: 2025-04-26 00:00:00 +/-TTTT
categories:
  - Homelabs
tags:
  - active-directory
image: preview-image.png
media_subpath: /assets/img/Homelabs/2025-05-31-active-directory-home-lab-setup/
---

## Introduction to Active Directory

The role of a directory is to store information about the objects present within it, but the Active Directory not only stores data but also provides it to the Network Administrators and the users of that particular domain whenever it is requested. It generally stores important information about the users like their names, passwords, contact information, etc and provides it to other users with authority in the same network to make use of the available information.

It stores data in a structured form hierarchically. It can have upright security with logon authentications and by having access control over the objects present in the Active Directory. For easy management one can also implement policy-based administration.

## Lab Requirements

* Virtual Machine (VMware Work Station/Player)
* Windows Server 2016
* Windows 10 Pro Operating System (Client)

## Configuring Windows Server 2016

Power on your VMware, and let’s begin with the installation by creating a new Virtual machine from the File option. Here you will personalise your Windows system by providing it with your username and the password that you want to set. Then click on Next to proceed.

![](2025-05-31-active-directory-home-lab-setup-1.png)

Now go to Virtual Machine settings and click on Network Adapter settings and make sure that there is a bridged connection where the host system’s physical network connection will be replicated. Let’s close this and move ahead.

![](2025-05-31-active-directory-home-lab-setup-2.png)

Here you see that the setup did not proceed, therefore let’s go back and fix this error from occurring.

![](2025-05-31-active-directory-home-lab-setup-3.png)

We will go back to Virtual Machine settings and click on Floppy, there under the connection option and choose the Use floppy image file option to make it work like a charm and proceed.

![](2025-05-31-active-directory-home-lab-setup-4.png)

Now you will select the operating system to install from the four options given below. Here we use Standard Edition with the GUI to have a better user-interface. The Desktop Edition provides much better features as compared to the server-core as it has very limited functions. Click on next to proceed.

![](2025-05-31-active-directory-home-lab-setup-5.png)

The operating system should start installing and under the customize setting option enter the password you want to put for the default administrator account.

![](2025-05-31-active-directory-home-lab-setup-6.png)

Now you see that your server is installed and ready to use and can find all the basic details on the server under the system option of the control panel.

![](2025-05-31-active-directory-home-lab-setup-7.png)

## Installing AD DS

Now let us open the system properties from the ‘Local Server’ option and let us make changes to the domain name.

![](2025-05-31-active-directory-home-lab-setup-8.png)

Let’s keep the computer name as DC 1 and make it the member of the workgroup with the name ‘WorkGroup’. On finishing this, click on ‘OK’ to proceed.

![](2025-05-31-active-directory-home-lab-setup-9.png)

Come back to the dashboard and now let’s begin with configuring the Active Directory role. Click on the Manage option at the top of the Dashboard. Then click on ‘Add roles and features.

![](2025-05-31-active-directory-home-lab-setup-10.png)

You see the installation wizard before you and click on ‘next’ to proceed.

![](2025-05-31-active-directory-home-lab-setup-11.png)

Then select Role-based or feature-based installation as it allows you to manually configure all the prefered roles at your convenience.

![](2025-05-31-active-directory-home-lab-setup-12.png)

Choose the server you have created from the server pool that is available before you.

Now choose the server roles you want to add. Here we require Active Directory Domain Services. We check that option and click next to proceed.

![](2025-05-31-active-directory-home-lab-setup-13.png)

In features installation Choose Group Policy Management. It is a management feature in Windows that allows you to control multiple users and computer configurations present in an Active Directory environment. Click on Next to proceed.

Now let us confirm the selections you have made for the installation of the Active Directory Domain Server and proceed.

![](2025-05-31-active-directory-home-lab-setup-14.png)

Let us wait for the installation to complete and close the window when it is ready.

![](2025-05-31-active-directory-home-lab-setup-15.png)

## Network Configurations

Enable the ethernet connection and click on Properties. Double click on Internet Protocol Version TCP/IPv4.

![](2025-05-31-active-directory-home-lab-setup-16.png)

Now assign the Static IP address and the subnet mask will be automatically be assigned. Also, assign the default gateway. Then assign DNS Server address.

![](2025-05-31-active-directory-home-lab-setup-17.png)

## Post-Deployment Configurations

Once the AD DS feature installation is completed you see a flag notification, so let us move on to the configurations that are required in the post-deployment phase. Click on Promote this server to a domain controller to proceed.

![](2025-05-31-active-directory-home-lab-setup-18.png)

In Deployment Configuration let’s create a new forest with the root domain name as ignite.local. A forest in the Active Directory is of the highest level of organisation. Each forest has the potential to share a single database, a global address list and security boundaries. Therefore, by default one use or even for that matter an administrator belonging to one forest cannot make us of another forest.

![](2025-05-31-active-directory-home-lab-setup-19.png)

Now let’s configure the domain controller capabilities by checking the first two boxes which allow DNS server and Global Catalog. Also, enter the Directory Services Restore Mode password which is a safe mode booting method for windows server domain controllers. The Domain functional level will depend on the forest functional level. Click on Next to proceed.

![](2025-05-31-active-directory-home-lab-setup-20.png)

You can skip this option and click on Next.

![](2025-05-31-active-directory-home-lab-setup-21.png)

In the additional option, you can verify your NetBIOS name as entered prior and proceed.

![](2025-05-31-active-directory-home-lab-setup-22.png)

Mention the path for creating AD DS database, log files and SYSVOL storage and proceed.

![](2025-05-31-active-directory-home-lab-setup-23.png)

Check all the specifications that you have set are correct and Install the configuration. On finishing the installation, the server will reboot itself and ask you to login again.

![](2025-05-31-active-directory-home-lab-setup-24.png)

## Create OU and user

Now let us proceed to create users in our Active Directory by clicking on **Tools/Active Directory Users and Computers.** It will open a new window; click on the domain name you have created and then click on **New/Organisational Unit**.

![](2025-05-31-active-directory-home-lab-setup-25.png)

A new window will appear for creating a new object. You can name it as per your requirement and proceed.

![](2025-05-31-active-directory-home-lab-setup-26.png)

A window to create a new object which is a user will appear. Enter all the required details of the user and proceed.

![](2025-05-31-active-directory-home-lab-setup-27.png)

Enter the password for the newly created user and then proceed ahead. Voila! Your user has been created.

![](2025-05-31-active-directory-home-lab-setup-28.png)

Subsequently you can create multiple users under an organisational unit.

![](2025-05-31-active-directory-home-lab-setup-29.png)

## Add Client to the Domain

Here in the Windows 10 system before connecting it to the domain, we have to set a Static IP for the system and mention the IP address of the Domain Controller in the DNS server address.

![](2025-05-31-active-directory-home-lab-setup-30.png)

Go to the control panel and check the basic information of your system and change the computer name settings.

![](2025-05-31-active-directory-home-lab-setup-31.png)

Now click on the change option to join the domain.

![](2025-05-31-active-directory-home-lab-setup-32.png)

It will display your computer name and click on domain under the member and you will be prompted to enter the username and password of the domain changes that you are making.

![](2025-05-31-active-directory-home-lab-setup-33.png)

Once, you are done with this, restart your system and you can login with your username and password to sign in under the domain that you had previously created.

![](2025-05-31-active-directory-home-lab-setup-34.png)

After logging in you can open the command prompt and go too the directory in which your user is present. Make use of the net user command and mention the user’s name with domain. You will get details about the user

```powershell
net user yashika /domain
```

![](2025-05-31-active-directory-home-lab-setup-35.png)

Hence here your Active Directory Pentesting Lab is setup and ready to use. Happy Pentesting!

## Bonus:

### Allowing Network Discovery

1. **Open the Control Panel** - Click the Start button, type control, and press Enter.
2. In the Control Panel, go to **Network and Internet** > **Network and Sharing Center**.
3. Click on the **Change advanced sharing setting** link on the left.

![](2025-05-31-active-directory-home-lab-setup-36.png)

4. Under the Private network section, **Turn on network discovery**.

![](2025-05-31-active-directory-home-lab-setup-37.png)

5. Click Save Changes.

After enabling Network Discovery, you should see the shared computers and devices on your network.

### Disabling Windows Defender

* Open Group Policy Management on DC and under our newly created domain right click and choose "Create a GPO in this domain, and Link it here..." to create a new group policy for the whole domain at once.

![](2025-05-31-active-directory-home-lab-setup-38.png)

* Name the GPO as "Disable Windows Defender" and click "OK".

![](2025-05-31-active-directory-home-lab-setup-39.png)

* Now open the dropdown list under the domain and right click on our new GPO and click on "Edit...".&#x20;

![](2025-05-31-active-directory-home-lab-setup-40.png)

* One we click edit it will open GPM Editor, from here in drop down list and go to&#x20;

**Computer Configuration** > **Policies** > **Administrative** **Templates** > **Windows** **Components** > **Windows** **Defender Antivirus.**

![](2025-05-31-active-directory-home-lab-setup-41.png)

* click on “Windows Defender Antivirus” and then double click “Turn off Windows Defender Antivirus"

![](2025-05-31-active-directory-home-lab-setup-42.png)

* In the pop up prompt check "Enabled" and then click on "Apply" and "OK" to complete the action.

![](2025-05-31-active-directory-home-lab-setup-43.png)

* Now go back to our created GPO in Group Policy Management window and click on our policy and check "Enforced" in Scope panel to enforced the policy through out the entire Domain.

![](2025-05-31-active-directory-home-lab-setup-44.png)

### Create Users using PowerShell

As the main purpose of this simulation is to create an environment like fortune 500. So, for these we will be creating atleat a 1000 users with the help of PowerShell scripting.

#### Script 1:

The purpose of below script is to generate random firstname and lastname and create a user with with `Password1`.

```powershell
 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}
```

#### Script 2:

The purpose of below script is to get names form a file and create users with `Password1`.

```powershell
# ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
# ------------------------------------------------------ #

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```
