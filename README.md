# MST-Project

Creating Virtual Machines and Installing Windows 

1.0 Creating VMs in VMWare Workstation Pro
Created my four VMs
Two VMs from Server 2016 ISO, named them Server Core and Server1
Two VMs from Windows 10 ISO, named them Admin Client and Client 1

2.0 Installing Windows on Your VMs
I mounted the Windows ISO on each server make sure to remove floppy disk because it was giving errors.

3.0 initial Configuration
Renamed and Configured from Prelab for Server1, Admin client, and Client1
Added DNS for the server1 to add a static ip address

4.0 Install VMWare Tools on each Guest OS 
Login to every VM except Servercore and installed VMWare tools
Had issues at first but I just removed floppy disk and SATA
you can install above the tab on the top right install vm tools

5.0 Download and Install the Remote Server Administration Tools (RSAT) and Windows Admin Center on your Adminclient VM

Downloaded the tool on my admin client done this before having it join the domain for next lab because it makes it easier to connect to the internet .
Downloaded WindowsTH-KB2693643-x64.msu 
While connected to the internet I downloaded Windows Admin Center 
Windows Admin Center will not run on internet browser without update or just download alternatives like FF or Chrome
Confirm RSAT was downloaded by typing in windows search bar Administrative tools

6.0 TCP/IP Manual Config
Change all VMS Network adapter settings to Bridged.
Lets you communicate between the VMs but you will not be able to use internet on the VMs



Active Directory and Remote Administration

1.0 Creating your Domain
Installed Active Directory on Server1 , by opening 
server manager, clicked on Add Roles and Features from the Dashboard . Checkbox for Active Directory Domain Services with DNS at the same time but it worked for me without it because previous lab needed DNS.
You can confirm if its installed by repeating steps.
On the server Manager you will see warning label yellow triangle which will help promote server1 to a domain controller
Click the option to Add a New Forest, enter the my domain name ako7.com
Leave global Catalog checked
P@ssw0rd assigned. 
Default settings in additional options , in paths as well , ignore pre req, install. Takes a couple of minutes.
Now when we login as Administrator im logging as Adminstrator of the new Domain

2.0 Clients Join the Domain
Change my TCP/IP settings so both my windows 10 client are pointing towards my Server1 for Preferred DNS settings.
Which was 10.0.238.10
Have both my clients join the domain by going to system info from windows start button , click change and add new domain ako7.com and login with ako7.com\administrator

3.0 Initial Configuration of ServerCore 
Used PowerShell to rename my server , configure static IP address, and join the domain , to join Powershell just open CMD line and type Powershell
Boot up Server Core and login as the administrator 
cmd prompt to Powershell  PS:C\Users\Adminstrators>

3.1.1 Set a static IP address
I use the command Get-NetIPinterface
output would show a column with IfIndex , beside Ethernet0 remember the # which was 2.
Ran the following cmdlets to set the static IP address

New-NetIPaddress -InterfaceIndex 2 -IPAddress 10.0.238.30 -PrefixLength 24 -DefaultGateway 10.0.238.1

.InterfaceIndex is the value of IfIndex from step 2.

· IPAddress is the static IP address you want to set.

· PrefixLength is the prefix length (another form of subnet mask) for the IP address you are setting. (For our example, 24 = 255.255.255.0)

· DefaultGateway is the IP address to the default gateway.

3.1.2 Configure Preferred DNS server
Used command to set DNS client server address

Set-DNSClientServerAddress –InterfaceIndex 2 -Server Addresses 10.0.238.10
· InterfaceIndex is the value of IfIndex from step 2.
· ServerAddresses is the IP address of your DNS server (Server1)

3.1.3 Join a domain
Used command to join a computer to a domain
Run Add-Computers prompted to login for both credentials to join the domain 
had some issues with this just kept attempting with different logins iirc it was sc login 
decided to skip next step and go to socnfig.cmd and change my workgroup to domain type.
Restart computer after Restart-computer 

3.1.4 Rename your ServerCore Server
Used following command to rename local computer to SC-ako7sc
Rename-Computer -NewName “SC-ako7sc” -DomainCredential ako7.com\Administrator -Restart
i technically already renamed my SC from Lab 1, but still run this command to get error code renamed already.
After all of this verify all the configs done so far by ipconfig /all

3.2 Using SCONFIG.CMD to Configure ServerCore
Went into sconfig and change workstation from workgroup to domain this somehow helped fixed my problem in step 3.1.3.
Enabled Remote administration , set windows update setting to manual , enable remote desktop option 2 less secure
change time by going to option 9 and then exiting config mode.

4.0 Connect Server Manager to ServerCore Server
Go on Windows 10 Admin client while logged in as Domain Administrator , go to server manager 
Had issues with this kept giving us errors along the line with  add servers active directory not showing
added it through the tab next to active directory which was also wrong but it gave me a new error so I removed it and attempted again after restarting PC and it finally showed it self.
Also add server1 .

4.1 Configure ServerCore to allow Remote Administration
On my admin client go to server manager that was open right click on my senecaid ako7 and select Powershell, essentially the same as opening Powershell prompt on my ServerCore. Enter commands to configure my firewall settings to allow more remote administration.  To confirm commands worked go to server manager and right click my SC and click management.

Import-Module NetSecurity

Get-NetFirewallRule -DisplayName *DCOM-In* | Set-NetFirewallRule -Enabled True

Get-NetFirewallRule -DisplayGroup "Remote Event*" | Set-NetFirewallRule -Enabled True


5.0 Using Windows Admin Center for Remote Administration
Using Admin Client VM, open windows admin center by double clicking icon on desktop or type searchbar, wasn't working on Internet explorer so I copy pasted link to Firefox.
Tool opens and add , servers, and select Search Active Directory , put ServerCore name in the field SC-ako7sc and add.
Same process applies to Server1
Now you can see overview of the server along with tools.



User and Group Management in a Domain



1.0 Creating Organizational Units
On Server1 , Go to Server Manager, From the Tools menu top right, select the tool called, Active Directory Users and Computers (ADUC)
When ADUC is open go right click your domain name (ako7.com) and click New Organizational Units
Created 3 different OU's , Toronto, Montreal and Vancouver.

2.0 Creating Global Groups in the Domain
For each OU, we create Global Groups for each of them , all of the groups should have the setting default which is global security.
Starting off with Toronto OU we create the 4 Groups , T_SalesRep, T_Marketing, T_HRSupport , T_Executives.
And then for Montreal and Vancouver just do the same thing but replace first character of with corresponding letter M_, V_.


3.0 Creating Domain Local Groups
Creating Domain local groups through powershell, I had a bit of difficulties with the commands , specifically the more complex ones. I just went with basic one. Review with professor if possible on how to get the complex ones to work with pathing. 

3.1 Using PowerShell on Server1 to create Domain Local Groups
The command was New-ADGroup, we need to create Marketing_Read, and Marketing_FC. 
Use some of the following options with your PowerShell command: 
•	-SamAccountName
•	-GroupCategory
•	-GroupScope
•	-Path
•	-Description
· What do you need to add to the command to put the Domain Local Group into an OU when using PowerShell? Path – “OU=Toronto” as an example.
EDIT: The other command would be 
New-ADGroup -SamAccountName "Marketing_Read" -GroupCategory "Security" -GroupScope "DomainLocal" -Path "OU=Toronto,DC=ako7,DC=com" -Description "Marketing group with read-only access."

I stuck with just using basic one which is 
New-ADGroup -Name “Marketing_FC” -GroupCategory Security -Groupscope 1
New-ADGroup -Name “Marketing_Read” -GroupCategory Security -Groupscope 1
3.2 Using RSAT on AdminClient to Create Domain Local Groups on Server1
Go to AdminClient VM and I open Windows Server Manager and right click my S1-ako7 from all servers menu. Right click and select Active Directory Users and Computers and it should pull up the same thing you see on your Server1. Using this I proceeded to create the domain local groups: 
•	HR_Read
•	HR_FC
•	SalesFiles_FC

3.3 Using RSAT on AdminClient to Add Global Groups to the Domain Local Groups
While still in the ADUC from previous step find the recently created group SalesFiles_FC
double click and youll be in properties under members cllick add, go to advanced and go to find now and select the global groups we created for Toronto, Montreal, and Vancouver's Sales rep should be T_SalesRep ,M_, etc.
After that is done and applied, add Toronto's , Montreal's , and Vancouver's Marketing groups to Marketing_FC.
Lastly same thing with Toronto's, Montreal's, and Vancouver's Executives global groups to SalesFiles_FC.

4.0 Creating Domain User Accounts
· What is the naming convention you decided on?  Initial of first name and complete last name

4.1 Create Users One at a Time with Active Directory Users and Computers
Go to the Toronto OU you created in the very beginning, create the following Users and assign them to members of the following.
•	Fred Flintstone – Member Of: T_SalesReps
•	Barney Rubble – Member Of: T_SalesReps
•	Bam-Bam Rubble – Member Of: T_Marketing
•	Wilma Flintstone – Member Of: T_HRSupport
•	Mr. Slate – Member Of: T_SalesReps and T_Executives
To add them to the specific group just go to either the user by double clicking and adding them there or go to the group and assign members there. When you're creating the users you should be prompted with a password just use default which is P@ssw0rd, and make sure to have the checkbox upon login user must change password. In the lab it wants you to also manually configure at least 2 users from the 5 Create job titles, phone numbers and addresses for users in the Toronto OU, this is to show how hard it is manually.

4.2 Creating Users Accounts with PowerShell
Go to your Server1 Powershell and create 2 users under Toronto OU. 
	· Pebbles Flintstone
	· Great Gazoo
Like previously from powershell creating I had a bit of issues but I may have figured out how to use the pathway command so maybe try if you want to redo step 3.1 but I left it should be fine either way.

New-ADUser -Name "Great Gazoo" -GivenName "Great" -Surname "Gazoo" -SamAccountName "GGazoo" -UserPrincipalName "GGazoo@ako7.com" -Path "OU=Toronto,DC=ako7,DC=com" -AccountPassword(Read-Host -AsSecureString "Input Password") -Enabled $true

New-ADUser -Name "Pebbles Flintstone" -GivenName "Pebbles" -Surname "Flintstone" -SamAccountName "PFlintstone" -UserPrincipalName "PFlintstone@ako7.com" -Path "OU=Toronto,DC=ako7,DC=com" -AccountPassword(Read-Host -AsSecureString "Input Password") -Enabled $true

To confirm it worked you can use the command , Get-ADUser -Identity username 
this will show user info.

4.3 Creating User Account Templates

To create a user account template for the Executives in the Vancouver OU, follow these steps:

1. On Server1, open Active Directory Users and Computers.
2. Create a new user account with the following details:
   - User Logon Name: _Executives_Template (starting with an underscore character)
   - First Name: (Leave it blank or use any placeholder name)
   - Last Name: (Leave it blank or use any placeholder name)
   - User logon name (pre-Windows 2000): (Leave it blank or use any placeholder name)
   - Password: Set it to the default password.
   - Account options: Disable the account.

3. Once the account is created, right-click on the template account and select "Properties".
4. In the properties window, navigate to at least two tabs and configure the desired properties (e.g., General, Address, Telephones, Organization, etc.) based on your requirements.
5. Next, go to the "Member Of" tab and click "Add".
6. Add the V_Executives and V_SalesReps global groups to make the template account a member of these groups.
7. Click "Apply" and then "OK" to save the changes.

After completing these steps, you will have created a user account template named \_Executives_Template with the specified settings and group memberships. The template account will appear at the top of the OU list because it begins with the underscore character "_".

4.4 Using Templates to Create User Accounts

1. Using the template you created in the previous section, create three new Executive user accounts in the Vancouver OU with the following names:
   - Luke Perry
   - Jason Priestly
   - Shannon Doherty
To do this go to the template we have created _Executives_Template should be located in the OU, right click the template and copy and it should direct you to the prompt of creating the user and the default settings you set before like address and city. You can verify that the properties you built into the template were copied to the new user accounts by just checking properties of the new users.

5.0 Using the Built-In Groups to Assign User Rights
1. Create the following 5 user accounts on your domain: 
2. Just like our previous steps we go to our OU and right click the Corresponding one, Vancouver, Toronto and Montreal.
	· Joe Admin in the Toronto OU
	· Jane Admin in the Montreal OU
	· Bob Admin in the Vancouver OU
	· Hani Admin in your domain, but not in an OU.
	· Marg Admin in your domain, but not in an OU.
3. Add Joe, Jane and Bob to the Account Operators group and the Backup Operators group.
4. Add Hani and Marg to the Domain Admins group. You can do this by just going to members of and instead of find now it should be just check now you can do it at the same time Account Operators : Backup operators and find now.


 
5.1 Testing User Rights
5. Test the following and record whether the login worked or did not work and if it works as you expected: 
6. We can test this by just logging in with our email, example Fred Flintstone for mine would be Fflinstone@ako7.com , password is the default P@ssw0rd.
Task	Successful or 	Works as expected or not? Notes.
	Unsuccessful?
Login to Client1 as Fred Flintstone.	                Successful 	                 
Login to Server1 as Fred Flintstone.	                Unsuccesful 	                 
Login to Client1 as Bob Admin.	                 Successful	                 
Login to Server1 as Bob Admin.	                Successful 	                 
Login to Server1 as Hani Admin.	                Successful 	                 




![image](https://github.com/koalex3500/MST-Project/assets/135271232/aefd6aa0-4b45-4a0f-9c98-0238dc794c0b)

