<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Azure Active Directory Test Lab</h1>


<h2>Project Summary</h2>
The purpose of this lab is to walk the user through the process of creating two VM's within Microsoft Azure and then configuring them to work with Active Directory.
One of the VM's will work as the domain controller that manages all the network devices while the other will be a client server that non-admin users have access to.
After setting up both VM's to work with Active Directory, a few tests will be run such as unlocking user accounts, reseting user passwords, and disabling/enabling user accounts <br />

<h2>Environments and Technologies Used</h2>

Languages Used: Powershell

Environments Used: Azure, Windows 10(22H2), Windows Server 2022

Technology/Applications/Services: Azure Virtual Machines, Remote Desktop, Active Directory Domain Services



<h2>High-Level Deployment and Configuration Steps</h2>
<h3>Azure VM Setup</h3>
- Create Resource Group and Vnet <br />
- Create Domain Controller VM <br />
- Create Client VM <br />
- Configuration Steps / Testing <br />
<h3>Active Directory Deployment</h3>
- Configuring a domain on the domain controller VM <br />
- Creating a domain admin <br />
- Join the Client VM to the domain <br />
- Create a normal client user <br />
<h3>Final Configurations + Testing</h3>
- Configuring Lockout Group Policy <br />
- Lockout Policy Testing <br />
- Reseting User Passwords <br />
- Enabling/Disabling User Accounts <br /> <br />

<h2>Azure VM Setup: Creating Resource Group and VNet</h2>

![image](https://github.com/user-attachments/assets/aa89f2bf-e166-4c60-b116-f98d482fa804)

<p>
To start our walkthrough, we will create a resource group that will hold our virtual network with a Domain Controller Server and a Client Server. You can name the resource group anything. I will name mine "Active_Directory_Walkthrough" for the sake of the tutorial. In my case, I am setting the region as Canada Central. Leave everything else as default values.
</p>
<br />

<h2>Azure VM Setup: Creating Domain Controler</h2>

![image](https://github.com/user-attachments/assets/4035f787-0d2d-4eb5-bf60-328e266e709c)

<p>
When creating your Vnet, give it a name, and make sure it is in the same region as your recource group. Other than that, leave all other settings as default. Next, we can create our domain controller. Make sure the following properties are used for this machine:

**Resource group**: The one we created earlier

**Virtual Machine name**: Whatever you want

**Region**: Same one as before

**Image**: Windows Server 2022 Datacenter: Azure Edition - x64 Gen2

**Size**: anything with 2 vcpu's

**Username and Password**: Anything you want (just make sure you log it in a notepad)

**Networking > Virtual network**: Network we createed before

After you are ready, click review + create and then Create.
</p>
<br />

<h2>Azure VM Setup: Creating Client</h2>

![image](https://github.com/user-attachments/assets/a30f4ac2-bd7a-4733-bd82-14bd6ddb43c7)

<p>
Now, we need to create the Client VM. Use the exact same options as in the Domain Controller aside from the image being Windows 10 Pro.
</p>
<br />

<h2>Azure VM Setup: Configuration Steps for VM's</h2>

![image](https://github.com/user-attachments/assets/2b884234-117f-4d17-9269-c62c0b3769ab)

<p>
In this walkthrough, we are going to configure the domain controller vm to act as a DNS server for the client VM. To do this, we have to set up a few things now, and then some later on. Let's change the domain controller's IP to be static rather than dynamic so that the client server will always be able to reliably find the dns at it's specified static address. To do this, we can click on the domain controller vm from the Virtual Machines window. Then click Networking > Network settings > blue link corresponding to the VM's Network Interface Card (NIC). On the next page, click the blue ipconfig and change the Private IP address settings Allocation from Dynamic to Static and click Save.
</p>
<br />

![image](https://github.com/user-attachments/assets/130c389d-5e8a-4bf9-b177-22d9f2bb651a)

<p>
Next, we need to ensure that the client VM will see the domain controller as a DNS server. Take note of the domain controller's private IP and then go back to the Virtual Machines page and click on the client VM. Then go to Networking > Network settings > blue link corresponding to the VM's Network Interface Card (NIC). Now, on the left side of the screen, click DNS servers. Now, just change the DNS servers from "Inherit from the virtual network" to "Custom" and put down the private IP of the domain controller we took note of earlier. CLick Save at the top.
</p>
<br />

<p>
Now, to apply the changes, restart the client vm:
</p>
<br />

![image](https://github.com/user-attachments/assets/5dcf37ee-3259-4bf9-9249-b3d0d975f704)

![image](https://github.com/user-attachments/assets/ba3dbb06-76b4-41b0-8b19-f8c11dd1d9b0)

<p>
We will need to disable the firewall in the domain controller for the purposes of this lab. To do so, remote into the domain controller. Then, Use the Run program and enter "wf.msc" to access the Windows Firewall. Next, right click "Windows Defender Firewall with Advanced Security on Local Computer" > Properties > Under all tabs, make sure the firewall state is set to Off > Apply > Ok.
</p>
<br />

![image](https://github.com/user-attachments/assets/6e71dc40-9bc5-4ae0-ac84-d2d3d5ad7b7a)

![image](https://github.com/user-attachments/assets/dd8721ab-0083-427a-b2a6-e6130c1b44a4)

<p>
To finish the first part of this walkthrough, we need to remote into the client VM and attempt to ping the domain controller to make sure the setup is working as expected. After opening the connection, open up the command prompt and ping the private IP address of the domain controller. Then, run the command ipconfig /all to check the DNS server for the client VM. The DNS server should be listed with the private IP of our domain controller:
</p>
<br />

![image](https://github.com/user-attachments/assets/8e868f12-bef7-4cd7-ba62-bbec75111bbd)




<h2>Active Directory Deployment: Configuring Domain</h2>

![image](https://github.com/user-attachments/assets/d2956577-6682-466e-90ae-3db84d73d98c)

<p>
To start, remote into both servers we created from the last portion of the walkthrough. We will first install active directory domain services on the domain controller VM. To do this, from the start menu, find the server manager > Add roles and features. Keep clicking next until the server roles menu appears. Check the box that says "Active Directory Domain Services" then Add Features. Then go through the rest of the installation leaving everything as default and click Install.
</p>
<br />

![image](https://github.com/user-attachments/assets/d2bf81d3-27d5-4909-a1b8-986497949847)

<p>
Next, click on the flag in the top right and then click "Promote this server to a domain controller" > Add a new forest > Root domain name = mydomain.com > Next > Choose a password for the DSRM (this is not going to be used but is required for setting up AD. Then click Next and uncheck Create DNS delegation. Leave everything else as default values and click Install at the end. After the installation completes, the server should automatically restart itself. 
</p>
<br />

![image](https://github.com/user-attachments/assets/d800875e-9d6d-434c-aedd-0c57186c9039)


<p>
After the restart, get back into the server with remote desktop. Doing this will require us to enter the domain name\user in the username feild as the domain controller now requires a domain source. 
</p>
<br />

<h2>Active Directory Deployment: Creating A Domain Admin</h2>

![image](https://github.com/user-attachments/assets/bc252d6d-dcbb-4ffc-b063-2dab32e72f51)

<p>
When you are logged into the VM again, click on the start menu > Windows Administrative Tools > Active Directory Users and Computers. We are going to be creating a domain admin. Before we can do this, lets create two Organizational Units (OU's) called _EMPLOYEES and _ADMINS . Right click on mydomain.com > New > Organizational Unit. 
</p>
<br />

![image](https://github.com/user-attachments/assets/07e2e20f-3c04-4c6b-aa1f-de241a3883d7)

<p>
Now, we are going to create the admin user. From within the ADMIN OU, right click > New > User. Fill out the information with whatever you want, but uncheck the "User must change password at next login" box. Also make sure to take note of the credentials you use because we will be using this user from now on to access the VM.
</p>
<br />

![image](https://github.com/user-attachments/assets/732e513b-460a-4f39-9645-47c2d145ac32)

<p>
Next, to make this user an admin, we need to add them to a security group. To do so, right click the user > Member Of > Add. Type "domain admins" in the box and click Chekc Names > Ok > Apply > Ok.
</p>
<br />

<h2>Active Directory Deployment: Adding Client to the Domain</h2>

![image](https://github.com/user-attachments/assets/51472794-8aec-4fe8-a39f-97ddece15661)

<p>
Now, log out of the machine and reconnect using the credentials of the user we just created. Also, use remote desktop to access the client VM as we are going to join it to the domain we configured. To do this, from within the client VM, right click start > System > Rename this PC (advanced) > Computer Name > Change. Switch Member of to Domain, and enter the domain name in the box (mydomain.com) then click ok. It should ask for credentials of an admin to make this change. We can use the admin credentials for the admin user we created before. This should result in a message that says welcome to the domain and then ask you to restart your sever. 
</p>
<br />

![image](https://github.com/user-attachments/assets/a9a8dfe8-8724-4ad5-9b87-44640f089153)

![image](https://github.com/user-attachments/assets/1d3a223b-8c3e-44d1-9a8b-2d4664b5550c)


<p>
To confirm that the client server is on the domain, we can open the Active Directory Users and Computers > mydomain.com > Computers. The client server should now appear in this list. Now, let's create a new OU called _CLIENTS and drag the client computer in Computers into the _CLIENTS OU. 
</p>
<br />

![image](https://github.com/user-attachments/assets/69677af1-cc7a-434f-8da2-a9f75331b256)


<p>
Next, login to the client server using the admin user credentials. We are able to do this because the client server was added to the domain. We want to configure the CLient VM so that any domain user can use their credentials to access the VM. Right click start > System > remote desktop > Select users that can remotely access this PC > Add > Type "domain users" in the box and click Check Names > Ok > Ok. 
</p>
<br />

<h2>Active Directory Deployment: Creating A Non-Admin Employee</h2>

<p>
From within the domain controller, let's now create an employee in the Active Directory Users and Computers window that won't be admins. Click on the _EMPLOYEES OU > Right click > New > User. Use whatever credentials you want. Now try to login to the client VM using the credentials of the employee you just made. 
</p>
<br />

![image](https://github.com/user-attachments/assets/dd726342-2d8e-4d02-810e-d3f67015cb62)




<h2>Final Configurations + Testing: Configuring Lockout Group Policy</h2>

![image](https://github.com/user-attachments/assets/553278c7-2301-4d54-bdde-132faf7929f1)


<p>
We will first need to make a configuration change that will lock users out of their account if they fail too many password attempts. To do this, we will need to access the Group Policy Management Console. We can get their by right clicking start > Run and typing gpmc.msc > Ok. Expand the tabs on the left until you find the Default Domain Policy under Forest > Domains > mydomain.com. Right click it and then Edit. Now, find the Account lockout policy through Policies > Windows Settings > Security Settings > Account Policies. 
</p>
<br />

![image](https://github.com/user-attachments/assets/949da7b0-849d-4d48-a05c-94824c96e153)

<p>
Edit the values on the right side of the window to match the following:

Account lockout duration : 30 minutes

Account lockout threshold: 5 invalid logon attempts

Reset account lockout counter after: 10 minutes
</p>
<br />

<h2>Final Configurations + Testing: Lockout Policy Testing</h2>

![image](https://github.com/user-attachments/assets/c79ce139-2dfb-477a-9ec8-4edcaaf98329)

<p>
In order for the changes we made to take effect, we need to log into the client VM using admin credentials anf then open the command prompt and run gpupdate /force. Once that has finished, we can logout of the vm and attempt to log back in as a regular user but purposfully use the wrong password over and over again until we are locked out. You should eventually get a message that looks like this: 
</p>
<br />

![image](https://github.com/user-attachments/assets/a301283e-ca63-48d3-866c-a36ece925191)


<p>
Now, in order to unlock the account, we can go back to the domain controller vm and unlock the users account from within the Active Directory Users and Computers Window. Double click the user and under the Account tab, check the box that says "Unlock account" > Apply > Ok. When we try to log back into the client vm with the user account, we should be able to log in again. 
</p>
<br />

<h2>Final Configurations + Testing: Reseting User Passwords</h2>

![image](https://github.com/user-attachments/assets/79656941-fb05-49b9-9459-967544575493)


<p>
The user's password can also be reset if they forget it by right clicking the user and selecting reset password. Try changing the user's password and re-logging back in with the new password now.
</p>
<br />

<h2>Final Configurations + Testing: Enabling/Disabling User Accounts</h2>

<p>
Another feature of admins is the ability to enable and disable user accounts. This can be done by simply right clicking the user and selecting enable/disable account. This feature is usually used in a case where someone retires from the company or is fired. Now, try to login to the non-admin user account after disabling the account. You should see a message that says the account is disabled:
</p>
<br />

![image](https://github.com/user-attachments/assets/7cfbbbd7-aae9-43e6-9dd6-51fe9d602530)

<p>
Lastly, as an admin in the domain controller VM, re-enable the account again and try to log back in as the user. 
</p>
<br />

<p>
Congratulations! You have finished the walkthrough for setting up Active Directory in an Azure Virtual Enviornment.
</p>
<br />



