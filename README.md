# Integrated Network Infrastructure with Windows AD and Linux

This setup demonstrates a flexible and well-rounded network infrastructure that brings together Windows Server technologies, Linux systems, email services, and security monitoring.

## [Key Components Include:](pplx://action/followup)

- **[Windows Server with Active Directory (AD)](pplx://action/followup):** Centralized authentication and directory services  
- **[DNS and DHCP](pplx://action/followup):** Integrated for name resolution and IP address management  
- **[Client Connectivity](pplx://action/followup):** Windows 11 and Linux systems connected to the AD domain  
- **[Postfix Email Integration](pplx://action/followup):** Configured between Linux and an Ubuntu mail server for efficient email routing  
- **[Wazuh Security Monitoring](pplx://action/followup):** Collects and analyzes logs from both Windows and Linux agents, enhancing security visibility  



## Overview

| Operating System | Specs | Storage (minimum) | IP Address | Function |
|------------------|-------|-------------------|------------|----------|
| Windows Server 2025 | 2 CPU / 4096 MB | 50 GBs | 10.0.0.5 | Domain Controller (DNS, DHCP, SSO) |
| Windows 11 Enterprise | 2 CPU / 4096 MB | 80 GBs | 10.0.0.100 or (dynamic) | Windows Workstation |
| Ubuntu 22.04 Desktop | 1 CPU / 2048 MB | 80 GBs | 10.0.0.101 or (dynamic) | Linux Desktop Workstation |
| Ubuntu 22.04 Desktop | 2 CPU / 4096 MB | 80 GBs | 10.0.0.10 | Dedicated Security Server |
| Ubuntu Server 22.04 | 1 CPU / 2048 MB | 25 GBs | 10.0.0.8 | SMTP Relay Server |




## Network Topology
![image](https://github.com/user-attachments/assets/e1a2923c-24f7-4737-b654-73736be946a9)

## [Tools](pplx://action/followup)

- **[Microsoft Active Directory](pplx://action/followup)**: A directory service used for managing and organizing network resources, users, and permissions in a Windows environment.

- **[Wazuh](pplx://action/followup)**: An open-source security monitoring platform that provides intrusion detection, log analysis, vulnerability detection, and compliance reporting.

- **[Postfix](pplx://action/followup)**: A popular open-source mail transfer agent (MTA) used for sending and receiving email on Unix-like operating systems.



 # VirtualBox Installation and VM Setup


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

#### [Configure Administrator Account](pplx://action/followup)
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

![image](https://github.com/user-attachments/assets/c9bc2931-95cd-4e1e-bb54-188c2b7ad42a)


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


![image](https://github.com/user-attachments/assets/76c46aa0-110c-4a41-871a-510d418250db)

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


![image](https://github.com/user-attachments/assets/b8c86489-0d6c-4bf2-b128-16338cc4dd65)

![image](https://github.com/user-attachments/assets/f3ce2b83-042e-4b66-987e-fb261022bd6d)

---

### [Setup DNS for Internet Access](pplx://action/followup)
1. Open **Server Manager** → Go to DNS → Right-click on your server → Select **DNS Manager**.
2. In DNS Manager, right-click your domain and select **Properties**.
3. Go to the Forwarders tab → Click Edit.
4. Add `8.8.8.8` (Google's public DNS) → Click OK.
5. Open PowerShell and test with:



ping google.com


nslookup corp.project-x-dc.com

![image](https://github.com/user-attachments/assets/678c8a52-07e7-413c-a040-02cb3b31a2e0)

![image](https://github.com/user-attachments/assets/cdb4507f-11ed-4cb6-b218-be43525be30b)



![image](https://github.com/user-attachments/assets/ff9afbdf-5ef6-419d-b395-831c2c3fcf5b)

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


![image](https://github.com/user-attachments/assets/34336187-e2a1-4a9a-af9b-0b23fa497395)

---

### [Add User Accounts in Active Directory](pplx://action/followup)
1. Navigate to Server Manager → Tools → Active Directory Users and Computers.
2. Go to Users folder → Right-click and select New → User.
3. Enter user information as needed:
4. Check "User cannot change password" option, then proceed with defaults.

![image](https://github.com/user-attachments/assets/1627103f-9469-4b7e-bb85-a0dee6f6cc3f)


---


# Windows 11 Enterprise Setup and Domain Connection 

## [Setup Windows 11 Enterprise](pplx://action/followup)

### [Disk Partitioning](pplx://action/followup)
1. Select **Next** → **Install Windows 11** → Check the box → **Next**  
2. Select **Disk 0 Unallocated Space** → **Create Partition**  
   - Use default "Size in MB" setting → **Apply**  
   - Wait for three partitions to appear  
3. Select **Disk 0 Partition 3** (largest free space) → **Install**  
4. Wait for installation to complete (VM will restart automatically)

### [Bypass Microsoft Account](pplx://action/followup)
1. Change network setting to **Host-only Adapter** (disables internet)  
2. Open Command Prompt with **Shift + F10**  

       oobe\bypassnro


3. Restart VM and select **I don't have Internet** during setup  

### [Local Account Configuration](pplx://action/followup)
1. Set temporary account name (will change after domain join)  
2. Create password:  
- Enter password twice  
- Set 3 security questions  
3. Disable **all Privacy Settings** toggles  
4. Complete setup to reach login screen  

### [Network Restoration](pplx://action/followup)
1. Change network back to **NAT Network** → **project-x-network**  

![image](https://github.com/user-attachments/assets/b1db876e-c84a-4fd9-8304-35bea3100609)


---

## [Connect Windows 11 Enterprise to Domain Controller  ](pplx://action/followup)
**[Prerequisite](pplx://action/followup):** Ensure Windows Server 2025 (`project-x-dc`) is running  

### [Network Configuration](pplx://action/followup)
1. Open **Control Panel** (Windows+X) → **Network and Sharing Center**  
2. Right-click **Ethernet** → **Properties** → **IPv4 Properties**  
- Static IP Configuration:  
  ```
  IP address: 10.0.0.100  
  Subnet mask: 255.255.255.0  
  Default gateway: 10.0.0.1  
  Preferred DNS: 10.0.0.5  
  ```

![image](https://github.com/user-attachments/assets/44fe6110-b244-4dcb-8b6c-9ec733b293a1)


### [Domain Join Process](pplx://action/followup)
1. Search **"Change workgroup name"** → **Change**  
2. Enter domain credentials:  



![image](https://github.com/user-attachments/assets/31c16bda-c9cc-449c-aae2-fb29d362566b)

![image](https://github.com/user-attachments/assets/9d7a9986-31ff-4d33-b30d-50ad0fb71d87)

3. Restart VM when prompted  

### [Domain Login](pplx://action/followup)
1. At login screen: Select **Other User**  
2. Enter credentials:  

![image](https://github.com/user-attachments/assets/a7a44493-8939-4626-a2eb-e049ef7054ae)



 
# Ubuntu Setup and Active Directory Integration 

## [Install Ubuntu](pplx://action/followup)
1. Boot Ubuntu ISO and select **Install Ubuntu**
2. Configure settings:
   - Keyboard layout: Default
   - Installation type: **Erase Disk and Install Ubuntu**
   - Region: Select your location
3. Complete installation:
   - Enter credentials
   - Remove installation medium after restart
   - Disable **Location Services** in post-install wizard

![image](https://github.com/user-attachments/assets/f9a80361-f7ac-4834-bca4-4823959a678c)


![image](https://github.com/user-attachments/assets/0a487ab9-5e1c-451f-99b5-b29605c4b635)


![image](https://github.com/user-attachments/assets/0e2caa3e-3d1f-4c52-b38f-8791efaa8e72)

## [Network Configuration](pplx://action/followup)
1. Navigate to **Settings** → **Network**
2. Create "Linux AD" wired connection:

IPv4 Settings:

    Address: 10.0.0.101

    Netmask: 255.255.255.0

    Gateway: 10.0.0.1

    DNS: 10.0.0.5

![image](https://github.com/user-attachments/assets/f78a0871-031f-455e-947e-59ce935cc560)


## [Connect to Active Directory via Samba Winbind](pplx://action/followup)


sudo apt update

sudo apt -y install winbind libpam-winbind libnss-winbind krb5-config samba-dsdb-modules samba-vfs-modules

![image](https://github.com/user-attachments/assets/e8387e95-4bdc-460b-8af9-0d36cfa070d0)

![image](https://github.com/user-attachments/assets/74daaff1-23d4-4402-bba4-1fbb7f9bf0a6)

![image](https://github.com/user-attachments/assets/7b9dd4fc-091a-4eca-be77-879d3309e922)

### [Configure Samba](pplx://action/followup)
1. Move the smb.conf.org file:

sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.org


2. Edit new config:

sudo nano /etc/samba/smb.conf


Replace realm and workgroup with the following:

[global]

kerberos method = secrets and keytab

realm = CORP.PROJECT-X-DC.COM

workgroup = CORP

security = ads

template shell = /bin/bash

winbind enum groups = Yes

winbind enum users = Yes

winbind separator = +

idmap config * : rangesize = 1000000

idmap config * : range = 1000000-19999999

idmap config * : backend = autorid




### [System Configuration](pplx://action/followup)
1. Update name service switch:

sudo nano /etc/nsswitch.conf


Ensure contains:

passwd: files systemd winbind
group: files systemd winbind

![image](https://github.com/user-attachments/assets/b7c1d4a1-2704-491c-b56b-cf800ef16232)

2. Enable home directory creation:

sudo pam-auth-update

Scroll down up to the point where it states:” Create home directory on login“. Use the space
bar to select, tab to “OK” and hit enter.

![image](https://github.com/user-attachments/assets/82f96941-8d6f-4e6a-b255-01748ebf6600)

### [Change DNS settings to refer to AD](pplx://action/followup)

sudo nano /etc/resolv.conf

![image](https://github.com/user-attachments/assets/7e87dc8b-10a5-4585-9454-2dcfd80842ea)

### [Domain Join](pplx://action/followup)
Join the domain with Administrator

    sudo net ads join -U Administrator

Restart Winbind
    
    sudo systemctl restart winbind


## [Verify Configuration](pplx://action/followup)
Get Active Directory services information listing

    net ads info

List all available users

    wbinfo -u # List AD users


![image](https://github.com/user-attachments/assets/07a7f1f8-9ffa-48d2-af62-708510691fb1)



## [Create AD User (Windows Side)](pplx://action/followup)
1. On Domain Controller:
   - Open **Active Directory Users and Computers**
   - Create user `janed@corp.project-x-dc.com` 

![image](https://github.com/user-attachments/assets/ffe9fa22-e94a-46cc-bfe6-0b55caa4f7bd)

## [Final Checks](pplx://action/followup)
Clear the winbind cache by restarting the service, then see the changes reflected.

    sudo systemctl restart winbind
    wbinfo -u 
    sudo login 

Going back to the Server Manager, you should see “LINUX-CLIENT” under the “Computers”
folder.

![image](https://github.com/user-attachments/assets/25063361-df95-43cb-a71f-c25c8afd0e32)






# Ubuntu Security Server Setup and Active Directory Integration

## 1. Install Ubuntu Server
1. Boot ISO and select **Ubuntu Server**
2. Configure:
   - Network: Default settings 
   - Storage: **Use entire disk** 
   - Hostname: `email-svr` 
   - Username: `email-svt`
   
3. Skip Ubuntu Pro → Enable **OpenSSH Server**
4. Complete installation and reboot



## 2. Install Samba Winbind Packages

    sudo apt update

    sudo apt -y install winbind libpam-winbind libnss-winbind
    krb5-config krb5-user samba-dsdb-modules samba-vfs-modules


**Kerberos Prompts:**
 
 Add CORP.PROJECT-X-DC.COM for the two Kerberos Authentication pages.


## 3. Configure Samba
Move the smb.conf.org file:

    sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.org
    sudo nano /etc/samba/smb.conf

Replace realm and workgroup with the following:

[global]
kerberos method = secrets and keytab

realm = CORP.PROJECT-X-DC.COM

workgroup = CORP

security = ads

template shell = /bin/bash

winbind enum groups = Yes

winbind enum users = Yes

winbind separator = +

idmap config * : rangesize = 1000000

idmap config * : range = 1000000-19999999

idmap config * : backend = autorid



## 4. System Configuration
Confirm passwd and group have winbind set as a value. 

    sudo nano /etc/nsswitch.conf 


![image](https://github.com/user-attachments/assets/81a7eec7-08ce-4647-b976-63905ab1b343)

Issue the following command 
     
    sudo pam-auth-update 

Scroll down and select "Create home directory on login".

## 5. DNS Settings

Change DNS settings to refer to AD.

    sudo nano /etc/resolv.conf

![image](https://github.com/user-attachments/assets/7d55c04b-ed5f-45d7-bd75-426cc207bd78)

Add corp.project-x-dc.com to /etc/hosts:

    sudo nano /etc/hosts

![image](https://github.com/user-attachments/assets/82eb6192-24ac-4212-b95a-7f97682d79c1)

Change Network settings from Bridged to NAT Network .

## 6. Join Active Directory Domain

    sudo net ads join -U Administrator

Restart Winbind
    
    sudo systemctl restart winbind



## 7. Verify Integration

    net ads info 

    wbinfo -u 


## 8. Create AD Account (Windows Side)

1. Navigate to Server Manager → Tools → Active Directory Users and Computers.
2. Go to Users folder → Right-click and select New → User.
3. Enter user information as needed:

![image](https://github.com/user-attachments/assets/d96581a4-e6c1-4da4-9129-8ab5647c670a)



## 8. Final Verification
Clear the winbind cache by restarting the service, then see the changes reflected.

    sudo systemctl restart winbind
    wbinfo -u 

Login as email-svr (CORP+email-svr)

    sudo login 

Issue an id command to view status

    id 

![image](https://github.com/user-attachments/assets/1adebffb-8c52-489b-92dc-e01b2fb8248f)





# Postfix Email Server Configuration Guide

## 1. Install Postfix

sudo apt update
sudo DEBIAN_PRIORITY=low apt install postfix



**Installation Prompts Configuration:**
- General type: `Internet Site`  
- System mail name: `email-svr`  
- Root/postmaster recipient: `email-svr `
- Other destinations to accept mail for: 'Default settings'
- Force synchronous updates on mail queue: 'No'
- Mailbox size limit: `0`  
- Local address extension: `+`  
- Protocols: `All`

When prompted to restart services, accept the defaults, choose “OK”

## 2. Configure Postfix
Set the location for the Ubuntu user’s mailbox.

    sudo postconf -e 'home_mailbox=Maildir/'

Set the location of the virtual_alias_maps table, which maps arbitrary email accounts to
Linux system accounts.

    sudo postconf -e 'virtual_alias_maps=hash:/etc/postfix/virtual'



**Create Virtual Alias Map:**

Next, create the virtual file, then begin mapping email accounts to user accounts to
Linux system

    sudo nano /etc/postfix/virtual

![image](https://github.com/user-attachments/assets/21ad9815-dfa2-48f5-a217-7211cbb75485)



Enter any email address you would like accept:

    contact@corp.project-x-dc.com email-svr
    admin@corp.project-x-dc.com email-svr
    email-svr@corp.project-x-dc.com email-svr



Apply changes:

    sudo postmap /etc/postfix/virtual
    sudo systemctl restart postfix

Allow connections to the service Postfix with UFW:

    sudo ufw allow Postfix
    sudo ufw enable


## 3. Configure Email Client (s-nail)

echo 'export MAIL=~/Maildir' | sudo tee -a /etc/bash.bashrc | sudo tee -a /etc/profile.d/mail.sh

Supply variable into the current session with:

    source /etc/profile.d/mail.sh

Install s-nail email client:

    sudo apt install s-nail



**Edit s-nail Config:**

sudo nano /etc/s-nail.rc

Add:

set emptystart
set folder=Maildir
set record=+sent

![image](https://github.com/user-attachments/assets/31ac076f-12fd-4293-8b89-212cb6d27cb1)


## 4. Network Configuration
**Set Hostname:**

sudo nano /etc/hostname

Delete all contents in the file, then add:

    smtp.corp.project-x-dc.com



**Static IP Configuration:**

sudo nano /etc/netplan/01-netcfg.yaml


   network:
   
       ethernets:
       
               enp0s3:
               
                   dhcp4: false
                   
                   addresses: 
                   
                     - 10.0.0.8/24
                     
                   gateway4: 10.0.0.1
                   
                   nameservers:
                   
                   addresses: 
                   
                     - 10.0.0.5
                     
                     - 8.8.8.8
                     
     version: 2

![image](https://github.com/user-attachments/assets/873fe28c-2030-4990-a646-a0bb9f08071c)




    sudo netplan apply

    reboot



## 5. Final Postfix Configuration

    sudo nano /etc/postfix/main.cf


Add/Verify:

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination

myhostname = smtp.corp.project-x-dc.com

mydomain = corp.project-x-dc.com

alias_maps = hash:/etc/aliases

alias_database = hash:/etc/aliases

mydestination = $myhostname, localhost.$mydomain, localhost

relayhost =

mynetworks = 127.0.0.0/8 10.0.0.0/24 [::ffff:127.0.0.0]/104 [::1]/128

mailbox_size_limit = 0

recipient_delimiter = +

inet_interfaces = all

inet_protocols = all

home_mailbox = Maildir/

virtual_alias_maps = hash:/etc/postfix/virtual.

![image](https://github.com/user-attachments/assets/43ebcf71-5682-4d8f-ba5b-a6862e63e575)

## 6. DNS Configuration (Windows AD)
1. Open **DNS Manager** on Domain Controller  
2. Create new A record in `corp.project-x-dc.com` zone:  
   - Name: `smtp`  
   - IP: `10.0.0.8`

![image](https://github.com/user-attachments/assets/269eb4fc-3edc-457d-b192-8d334c7ce811)

## 7. Test Email Functionality
**Initialize Maildir:**

mkdir /home/email-svr/Maildir
echo 'init' | s-nail -s 'init' -Snorecord email-svr

![image](https://github.com/user-attachments/assets/c9d25e36-678d-453c-8df6-1485da93ba32)


**Send Test Email:**

    nano ~/test_message # Add "This is a test!"

    cat ~/test_message | s-nail -s 'Test' -r janed@corp.project-x-dc.com email-svr@email-svr


**Verify Delivery:**

    s-nail





# Wazuh Security Monitoring Setup Guide

## 1. Install Wazuh Server on Security Box (secbox)
Sign into sec-user@secbox:

    sudo su sec-user

Install cURL if it isn’t installed already:

    sudo apt install curl

Issue the following command to start the Wazuh installation wizard    
    
    curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i







## 2. Access Wazuh Dashboard
1. Navigate to `https://localhost` in browser  
2. Accept security risk 
3. Login with generated credentials  


---

## Deploy Wazuh Agents

### Windows [project-x-win-client]

-Go to “Server Management” -> “Endpoint Summary”.

-Choose “Deploy new agent”.

-Select Windows MSI.

-Server Address: 10.0.0.10.

-Assign an agent name: project-x-win-client

-Groups: default

-Copy and run the command

-Open new Powershell Session -> Right-click “Run as Administrator”

-Right-click to paste the command, then run the command:

    Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi - OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='10.0.0.10' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='project-x-win-client'

Start the Wazuh Agent:
    
    NET START WAZUH

![image](https://github.com/user-attachments/assets/a7ded300-5e59-4649-ad91-4aaf3e3c3b9e)


### Domain Controller ([project-x-dc])

Repeat the above steps. For the WAZUH_AGENT_NAME, change to project-x-dc.

Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi -OutFile $env:tmp\wazuh-agent
msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='10.0.0.10' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='project-x-dc'



**Common Post-Install Steps:**  



## Linux Client ([project-x-linux-client])

### Agent Installation
1. **From Wazuh Dashboard**:
   - Navigate to `Server Management` → `Endpoint Summary`
   - Select `Deploy new agent`
   - Configuration:
     - OS Type: `DEB amd64`  
     - Server Address: `10.0.0.10`  
     - Agent Name: `project-x-linux-client`  
     - Groups: `default`

2. **Terminal Commands**:

     `sudo wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.2-1_amd64.deb && sudo WAZUH_MANAGER='10.0.0.10' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='project-x-linux-client' dpkg -i ./wazuh-agent_4.9.2-1_amd64.deb`


3. **Enable Service**:

   `sudo systemctl daemon-reload
    sudo systemctl enable wazuh-agent
    sudo systemctl start wazuh-agent`



## Create Agent Groups

### Linux Group
1. Navigate to `Server Management` → `Endpoint Groups`
2. Select `Add new group`
3. Name: **Linux**

### Windows Group
1. Navigate to `Server Management` → `Endpoint Groups`
2. Select `Add new group`
3. Name: **Windows**

### Assign Agents to Groups
1. For `project-x-dc` and `project-x-win-client`:
- Edit agent → Add to **Windows** group
2. For `project-x-linux-client`:
- Edit agent → Add to **Linux** group



## Custom Log Collection Configuration

### Windows Group Configuration

These configurations enables Wazuh to collect and monitor Windows event logs from Security which captures security-related events, such login attempts and user account changes. They also collect logs related to applications, including crashes and significant errors.

 <agent_config>
  
   <!-- Shared agent configuration here -->
   
   <localfile>
    
     <location>Security</location>
     
     <log_format>eventchannel</log_format> #specifies that Wazuh will utilize the Windows Event Channel format for detailed monitoring and analysis. 
     
   </localfile>
   
   <localfile>
    
     <location>Application</location>
     
     <log_format>eventchannel</log_format>
     
   </localfile>

![image](https://github.com/user-attachments/assets/f4a4025b-3305-46b0-860f-0efacc6e0b6a)

  Linux Group Configuration

These logs allow Wazuh to detect potential security threats, unauthorized access attempts, and system changes on Linux systems
  
<agent_config>
  <localfile>
   
    <log_format>syslog</log_format>
    
    <location>/var/log/auth.log</location>
    
  </localfile>
  
  <localfile>
   
    <log_format>syslog</log_format>
    
    <location>/var/log/secure</location>
    
  </localfile>
  
  <localfile>
   
    <log_format>audit</log_format>
    
    <location>/var/log/audit/audit.log</location>
    
  </localfile>
  
</agent_config>


![image](https://github.com/user-attachments/assets/4d84a4c9-9691-4598-9977-6d0f1d906023)


## Future Plans

In the future, I aim to enhance this setup with the following features:

- **Implement File Integrity Monitoring (FIM)**: To detect unauthorized changes to critical files.
- **Set Up Behavioral Analysis**: To identify unusual patterns that may indicate potential security threats.
- **Integrate Cloud Services**: To extend security monitoring to cloud workloads and containers.
- **Create Custom Dashboards**: For visualizing security events and trends across the network.


