<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

1. Setup Resources in Azure
2. Ensure Connectivity Between Client and Domain Controller
3. Install Active Directory
4. Create Admin/User Accounts in AD
5. Join Client to Domain
6. Setup Remote Desktop
7. Create Additional Users

<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/yeMkPdF.png" height="80%" width="80%" alt=""/>
</p>

<p>
1.) First, we will create two Azure VMs. I recommend using 2 vCPUs for each VM. Create the Domain Controller VM (named DC-1) with the Windows Server 2022 image. Set DC-1's NIC Private IP address from dynamic to static. Create another VM with the Windows 10 image, and name it Client-1. You can check the topology in Network Watcher to ensure both VMs are in the same vnet. If you are having trouble viewing the topology, you may need to move the NetworkWatcherRG resource group into the same resource group as your VMs.
</p>
<br />

<p>
<img src="https://i.imgur.com/aSIIMfM.png" height="80%" width="80%" alt=""/>
</p>

<p>
2.) Login to Client-1 with Remote Desktop and ping DC-1's private IP address using the command <i>ping -t [ip address]</i>. This will issue a perpetual ping. You should notice the ping fails. From there, login to the Domain Controller (DC-1) and enable ICMPv4 on the local windows firewall by selecting Inbound Rules (hint: search 'wf.msc' on the start screen to access Windows Firewall). From the list of Inbound Rules, enable both 'Core Networking Diagnostic' rules with ICMPv4 protocol. Check back on Client-1 to see the ping succeed. Ctrl+C will stop the ping.
</p>
<br />

<p>
<img src="https://i.imgur.com/v55ZAgO.png" height="80%" width="80%" alt=""/>
</p>
<p>
3.) From the DC-1 Server Manager screen, select Manage, and navigate to the 'Add Roles and Features Wizard.' After installation is complete, select the hazard flag icon at the top right of the screen, and promote the server as a DC. Add a new forest, and give your domain a name (using mydomain.com in this scenario). Feel free to use any password, but take note of it for later. Continue through rest of the installation with default settings. Restart DC-1 and log back in as user: mydomain.com\labuser
</p>
<br />

<p>
<img src="https://i.imgur.com/X5LhMEs.png" height="80%" width="80%" alt=""/>
</p>
<p>
4.) After rebooting DC-1, open Active Directory Users and Computers (ADUC) from Tools at the top right of your Server Manager screen. Right click 'mydomain.com' and create two New Organizational Units. Title the first _EMPLOYEES and the second _ADMINS. Create a new User named Jane Doe with the username 'jane_admin' and assign them the same password as you used earlier when creating your domain. Be sure to uncheck any of the boxes on the Password window. Drag Jane Doe to the _ADMINS folder. Right-click Jane Doe and select Properties -> Member Of -> Add -> Type 'domain' -> Check Names -> Select 'Domain Admins' -> Ok -> Apply -> Ok. Log out of DC-1 and log back in as 'mydomain.com\jane_admin' (use this as your admin account from now on).
</p>
<br />

<p>
<img src="https://i.imgur.com/xMuU6gc.png" height="80%" width="80%" alt=""/>
</p>
<p>
5.) From the Azure portal, set Client-1's DNS settings to the DC's private IP address by selecting Virtual Machines -> Client-1 -> Networking -> Network interface: client-1487 (may be different in your case) -> DNS servers -> Custom -> <i>DC-1 private IP</i> -> Save. Still on Azure portal, restart the Client-1 VM from the Overview tab. Once complete, login to Client-1 with Remote Desktop as the original local admin (labuser) and join it to the domain (computer will restart). To do so, right-click the start menu -> System -> Rename this PC (advanced) [from Related Settings] -> Change -> Select 'Domain' under 'Member of' tab and type your domain name -> Ok -> Use jane_admin username and password when prompted -> Click Ok when prompted and let computer restart. Log back into DC-1 and verify Client-1 shows up in the Active Directory Users and Computers inside the 'Computers' container on the root of the domain. Create a new Organizational Unit named '_CLIENTS' and drag Client-1 to it.                                     
<br />

<p>
<img src="https://i.imgur.com/KlqLWKq.png" height="80%" width="80%" alt=""/>
</p>
<p>
6.) Login to Client-1 as mydomain.com\jane_admin and open system properties by right-clicking the start menu and selecting System -> Remote Desktop [Related Settings] -> 'Select users that can remotely access this PC' -> Add -> type 'Domain Users' -> Check Names -> OK. You can now login to Client-1 as a normal, non-admin user. You would normally do this with Group Policy that allows you to change multiple systems at once.
</p>
</p>
<br />

<p>
<img src="https://i.imgur.com/xyo7SDK.png" height="80%" width="80%" alt=""/>
</p>
<p>
7.) Login to DC-1 as jane_admin, and open Powershell_ISE as an administrator (right-click and select 'Run as administrator'). Create a new file and paste the contents of the [Script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1) Feel free to observe the script. You may notice the amount of users being created and the password used for each one (Password1). Run the script, and once it is finished, you can open ADUC and observe the accounts in the 'User' Organizational Unit. From here, you can select one of the new users and attempt to login to Client-1.
</p>
<br />
