<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Configuring On-premises Active Directory within Azure VMs</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Deployment and Configuration Steps</h2>
<img src="https://i.imgur.com/4wQcP0m.png" height="70%" width="70%" alt="Create DC1 and Resoruce Group"/>
<p>
We will be creating 2 virtual machines, one will be the Domain Controller and the other will be a computer within a domain.

- First, we will be creating the first virtual machine which will be the Domain Controller
    - Resource Group: Create New > AD-Lab
    - Name: DC-1
    - Region: Any Region Close to You (East US)
    - Image: Windows Server 2022
    - Size: E2s_v3 - 2 vcpus, 16 GiB
    - Username: labuser
    - Password: Password123!
    - Use Existing Windows Server License
- Confirm These Settings > Review + Create
</p>

<img src="https://i.imgur.com/HN4yh7O.png" height="70%" width="70%" alt="Setting DC-1 Private IP to Static"/>
<p>
After creating our virtual machine, we will now set the DC-1 VM's private ip to static.

- Select the DC-1 Virtual Machine > Networking > Network Interface [In my case it was [dc-1493]
    - IP Configurations > ipconfig1
    - Change Assignment from Dynamic to Static
- Confirm These Settings and Save
</p>

<img src="https://i.imgur.com/98K4Usf.png" height="70%" width="70%" alt="Creating Client-1 VM"/>
<p>
After configuring our Domain Controller, now we will be creating our host computer.

- Virtual Machines > Create > Azure Virtual Machine. Create a Virtual Machine with these settings
    - Resource Group: AD-Lab
    - Name: Client-1
    - Image: Windows 10 Pro
    - Username: labuser
    - Password: Password123!
    - Confirm I have an eligible Windows 10/11 license
- Confirm These Settings > Review + Create
</p>
       
<img src="https://i.imgur.com/U5p8NMm.png" height="70%" width="70%" alt="Test Connectivity"/>
<p>
After configuring our Virtual Machines, we will now test the connectivity between Client-1 and DC-1

- Login to Client-1 using Remote Desktop
    - Open Up Command Line
    - Use the "ping -t <dc-1 private-ip>" command. In this case its "ping -t 10.0.0.4"
- After following these directions, you will notice that these requests are timing out. This is because our DC-1 Virtual Machine is blocking ICMP messages.
</p>

<img src="https://i.imgur.com/etryl6p.png" height="70%" width="70%" alt="Enable ICMP on DC-1"/>
<p>
Now, we will need to navigate into DC-1's firewall and enable ICMP messages

- Login to DC-1 using Remote Desktop
    - Open Up The Search Bar > Windows Defender Firewall with Advanced Security
    - Inbound Rules > Sort By Protocol > Enable The ICMPv4 Rules
<img src="https://i.imgur.com/0mK2Rq5.png" height="70%" width="70%" alt="Successful Requests"/>

- After following these directions, you will notice that the requests sent by Client-1 are now getting replies from DC-1. Cancel the ping requests from Client-1 and move on to the next steps.
</p>

<img src="https://i.imgur.com/hyrLNYU.png" height="70%" width="70%" alt="Installing Active Directory"/>
<p>
Now that we have confirmed the connectivity between Client-1 and DC-1, we will now install Active Directory

- Open up DC-1 > Server Manager > Add Roles and Features
    - From the Server Roles Tab, Select "Active Directory Domain Services" and Press "Add Features
- After that, keep clicking on the Next button until you reach the Confirmation tab where you will hit Install
</p>

<img src="https://i.imgur.com/LRCR7Uy.png" height="70%" width="70%" alt="Configuring Domain Controller"/>
<p>
After the the installation has finished and you've received a "Installation succeeded on DC-1" message, we will now configure the domain controller.

- In Server Manager, Select the Flag at the top right with a yellow warning icon. Click "Promote this server to a domain controller".
    - Add a new forest > Root domain name: mydomain.com and hit Next
    - DSRM Password: Password1
- After that, keep clicking on the Next button and click Install. You will then be signed out of DC-1
</p>

<img src="https://i.imgur.com/LRCR7Uy.png" height="70%" width="70%" alt="Configuring Domain Controller"/>
<p>
After the the installation has finished and you've received a "Installation succeeded on DC-1" message, we will now configure the domain controller.

- In Server Manager, Select the Flag at the top right with a yellow warning icon. Click "Promote this server to a domain controller".
    - Add a new forest > Root domain name: mydomain.com and hit Next
    - DSRM Password: Password1
- After that, keep clicking on the Next button and click Install. You will then be signed out of DC-1 and your Remote Desktop connection will close.
</p>

<img src="https://i.imgur.com/evFUcC6.png" height="70%" width="70%" alt="Loging Into DC-1 Again"/>
<p>
After the the installation has finished, our previous credentials will not work anymore. In order to login again, use the mydomain.com\labuser credentials.
</p>

<img src="https://i.imgur.com/vYiCUqL.png" height="70%" width="70%" alt="Creating Organizational Units in Active Directory"/>
<p>
Now, we will be creating accounts, Organizational Units, and Provision Permissions in Active Directory

- In Server Manager > Top Right Corner (Tools) > Active Directory Users and Computers
    - Right-Click mydomain.com > New > Organizational Unit > Create 2 Organizational Units
        - _EMPLOYEES
        - _ADMINS
        
<img src="https://i.imgur.com/QXgDxML.png" height="70%" width="70%" alt="Creating Users in Active Directory"/>
<p>

- After that, Go to Right-click _ADMINS Organizational Unit > New > User
    - First Name: Jane
    - Last Name: Doe
    - Login: jane_admin
    - Hit Next > Password: Password1 > Uncheck User must change password > Check Password never expires and complete configuration
</p>

<img src="https://i.imgur.com/eGspDZ0.png" height="70%" width="70%" alt="Provisioning Permissions in Active Directory"/>
We will now give Jane Doe Domain Admin permissions in Active Directory

<p>

- Find Jane Doe from the _ADMINS Organizational Unit and Double Click to Open Up Their Properties
    - Member Of Tab > Add... > Type "Domain Admins" and hit "Check Names" > OK > Apply
    - After this, log out of DC-1 as "labuser" and log back in as mydomain.com\jane_admin
</p>

<img src="https://i.imgur.com/djBRabu.png" height="70%" width="70%" alt="Fixing DNS Settings"/>
<p>
From the Azure portal, we will need to Client-1's DNS to DC-1's private IP address

- Open up Client-1 from the Azure portal > Networking tab > Client-1's Network Interface
    - Select Client-1's DNS servers settings > Custom > Enter DC-1's private IP address into the DNS Server + Hit Save
- Once we've set up the DNS settings, restart Client-1 from Azure
</p>

<img src="https://i.imgur.com/9gaKVYc.png" height="70%" width="70%" alt="Adding Client-1 to mydomain.com"/>
<p>
Now, we will be configuring Client-1 such that it will be a part of the mydomain.com domain.

- Log into Client-1 using the labuser credentials
    - Right-click the windows icon from the bottom left and hit "System"
    - Select "Rename this PC" on the left-hand side
    - Click "Change..." next to "rename this computer or change its domain" > Member of... Domain: mydomain.com
    - Username: mydomain.com\jane_admin
    - Password: Password1
- Afterward, you will be asked to restart the computer. Restart the computer and login with mydomain.com\jane_admin and notice that you are able to login with those credentials
</p>

<img src="https://i.imgur.com/eMXicfq.png" height="70%" width="70%" alt="Allowing Domain Users to RDP to Client-1"/>
<p>
We will now configure Client-1 such that any domain user can log into the computer.

- Log into Client-1 using the mydomain.com\jane_admin credentials
    - Right-click the windows icon from the bottom left and hit "System"
    - Select "Remote desktop" on the left-hand side
    - Click "Select users that can remotely access this PC" > Add... > Domain Users, Check Names, OK
    - Username: mydomain.com\jane_admin
    - Password: Password1
- Afterward, you will be asked to restart the computer. Restart the computer and login with mydomain.com\jane_admin and notice that you are able to login with those credentials
</p>

<img src="https://i.imgur.com/8YXgzdC.png" height="70%" width="70%" alt="Creating Dummy Users"/>
<p>
After all these configuration steps, we will now create a lot of dummy users using PowerShell and use one of their credentials to log into Client-1

- First, if you are still logged into Client-1 with jane_admin's credentials, then log out. Navigate back into DC-1 and follow the next steps
    - Search "PowerShell ISE" in the search box and open as Administrator
    - Click on the "New Script" icon which should be directly underneath "File" and beside the Folder icon
    - Copy the code [here](https://github.com/vmp576/azure-active-directory/blob/main/Generate-Names-Create-Users.ps1)
    - Click on the Green play button or press F5
</p>
 
<img src="https://i.imgur.com/VTEb5ev.png" height="70%" width="70%" alt="Copying Dummy User Name"/>
<p>

- After this, we will copy one of the created usernames and log into Client-1 with the created account
    - Type "Active Directory Users and Computers" in the search bar > open up mydomain.com > open up the _EMPLOYEES Organizational Unit
    - Copy one of the usernames. In this case, I used "baba.waj"
    - Log into Client-1 using mydomain.com\username and password of Password1
<img src="https://i.imgur.com/ROokFfk.png" height="70%" width="70%" alt="Logging Into Client-1 With Dummy User"/>
</p>
