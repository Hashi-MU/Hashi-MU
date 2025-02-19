# Cybersecurity-Homelab-and-

# Cybersecurity Homelab

## Overview

## Network Topology
![image](https://github.com/user-attachments/assets/063666f2-dcaf-4e7d-aec3-8396a018ac47)


 # <a href="https://github.com/Hashi-MU/VirtualBox-Installation-and-VM-Setup/edit/main/README.md">VirtualBox Installation and VM Setup</a>


## Installing VirtualBox

1. Navigate to [VirtualBox.org](https://www.virtualbox.org/).
2. Select the Host operating system you are currently working with.
3. Select all system defaults designated in the installation wizard, unless you would like to customize.



## Provisioning A New NAT Network

1. Navigate to File > Tools > Network Manager
2. Select NAT Networks > "Create"
3. Name the NatNetwork "project-x-network" and choose an IPv4 prefix
4. Select "Apply" to save changes




## Provisioning A Virtual Machine

1. In VirtualBox, go to Machine > New
2. Enter VM details:
   - Name
   - Type: Microsoft Windows
   - Version: Windows 2022 (64-bit)
   - RAM: Minimum 4 GB (4096 MB)
   - CPU: Minimum 2
   - Enable EFI for Windows
   - Virtual Hard disk space: Minimum 50 GB
3. Review specifications and select "Finish"
4. In VM settings:
   - "Storage" tab: Add ISO to the optical drive
   - "Network" tab: Select "NAT Network" and choose "project-x-network"
5. Start the VM and follow the OS installation wizard


![image](https://github.com/user-attachments/assets/aab9d64c-bd5b-42ce-bd13-aa351c504a0e)









## [Windows Server 2025 Installation and Configuration Steps](pplx://action/followup)

#### [Install Windows Server 2025](pplx://action/followup)
1. Select **Next** → **"Install Windows 11"** → Check the box → **Next**.
2. Select **Desktop Experience**.
3. Accept Microsoft’s End User License Agreement (EULA) → **Next**.
4. Select **Disk 0 Unallocated Space** → **Create Partition**. Use the default **Size in MB** setting → **Apply**.
   - Wait for three partitions to show up.
5. Select **Disk 0 Partition 3** (with the largest free space) → **Install**.
   - Wait for Windows Server 2025 to fully install. The VM should restart.

---

#### [Step 2: Configure Administrator Account](pplx://action/followup)
1. Set a password for the default Administrator account: 
2. The login screen will appear.
3. Navigate to the top of VirtualBox: **Input** → **Keyboard** → **Insert Ctrl-Alt-Del** to open the login prompt.
4. Choose **Required only** for sending diagnostic data to Microsoft.
5. After signing in, you should see the **Server Manager** window. Exit out of the dialog box to try Azure Arc.

---

#### [Disable Default Logoff](pplx://action/followup)
1. Search for **Settings** in the Search bar → **System** → **Power**.
2. Select the toggle under **Screen timeout** → Set to **Never**.

---

#### [Disable CTRL + ALT + DEL Requirement](pplx://action/followup)
1. Search for **Local Security Policy**.
2. Navigate through the following folder tree:
   - Look for **Interactive logon…**
3. Toggle from Disabled to Enabled → Select **Apply** → **OK**.

---

### [Assign Static IP Address](pplx://action/followup)
1. Ensure Windows Server 2025 (`project-x-dc`) is running.
2. Open Control Panel (Shortcut: `Windows+X`).
3. Select **Network and Sharing Center** → **Change adapter settings**.
4. Right-click on the computer icon named "Ethernet" → Select **Properties**.
5. In the new dialog box, select **Internet Protocol Version 4 (TCP/IPv4)** → Click on **Properties**.
6. Set a static IP address:
   - IP address: `10.0.0.05`
   - Subnet mask: `255.255.255.0`
   - Default gateway: `10.0.0.1`
7. Click **OK**.

---

### [Promote Active Directory to a Domain Controller](pplx://action/followup)
1. Go back to **Server Manager** → Select **Add roles and features**.
2. Click **Next** for the first three boxes.
3. Select:
   - Active Directory Domain Services
   - DHCP Server
   - DNS Server
   - File and Storage Services
   - Web Server (IIS)
4. Leave defaults and click **Next**, then proceed until you reach the Confirmation tab → Click **Install**.
5. Close the dialog box while features are installed.
6. Once installation is complete, a notification will appear in Server Manager:
   - Click on it and select "Promote this server to a domain."
7. Choose:
   - Add a new forest → Enter root domain name: `corp.project-x-dc.com`.
8. Leave default options; set Directory Services Restore Mode (DSRM) password → Click Next.
9. Skip "Create DNS delegation" box → Click Next through all defaults until reaching the check screen.
10. Allow checks to complete, then click Install and let the server restart.

---

### [Verify Domain Controller Setup](pplx://action/followup)
1. Log in under `CORP\Administrator`.
2. Open PowerShell and type:
Get-ADDomainController


---

### [Setup DNS for Internet Access](pplx://action/followup)
1. Open **Server Manager** → Go to DNS → Right-click on your server → Select **DNS Manager**.
2. In DNS Manager, right-click your domain and select **Properties**.
3. Go to the Forwarders tab → Click Edit.
4. Add `8.8.8.8` (Google's public DNS) → Click OK.
5. Open PowerShell and test with:

ping google.com


nslookup corp.project-x-dc.com


---

### [Setup DHCP](pplx://action/followup)
1. Navigate to DHCP in Server Manager → Open DHCP Manager.
2. Go to IPv4 and create a new scope:
- Scope Name: `project-x-scope`
- Start IP address: `10.0.0.100`
- End IP address: `10.0.0.200`
- Subnet mask: `255.255.255.0`
3. Use defaults for exclusions, lease expiration, etc., and set Router IP as `10.0.0.1`.
4. Complete all dialog boxes until finished.

---

### [Add User Accounts in Active Directory](pplx://action/followup)
1. Navigate to Server Manager → Tools → Active Directory Users and Computers.
2. Go to Users folder → Right-click and select New → User.
3. Enter user information as needed:
4. Check "User cannot change password" option, then proceed with defaults.

---

### [Success!](pplx://action/followup)
You have now completed installing, configuring, and customizing your Windows Server 2025 environment with Active Directory, DNS, DHCP, and user accounts!
