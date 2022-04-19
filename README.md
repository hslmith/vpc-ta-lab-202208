# VPC Mini Lab
### URLS Needed in this Mini Lab
- [IBM Cloud Portal] (http://cloud.ibm.com)
- [This Lab Repo] (https://github.com/hslmith/vpc-mini-lab)



## Login to IBM Cloud Portal

Decide on a Region to create this demo: US South (Dallas) or US East Washington DC<br>
I have chosen **Dallas us-south** for this lab 


>{Optional}
>### Check IAM Permissions
> - Add Console Administrator
>
>{End Optional}



## Navigate to VPC
1. Click top left hamburger menu  
2. Scroll down to **VPC Infrastructure**
3. Click the Pin next to VPC Infrastructure to pin the menu otion to the top.
4. Choose **Overview** within the **VPC Infrastructure** menu. This will take you into the VPC landing page.


## Create a SSH Key
>Before you can add a key in the IBM Cloud console, you must make your SSH key available. Your SSH key must be an RSA key with a key size of either 2048 bits or 4096 bits.


>{Optional}
>Mac or Linux:
>In a Terminal Window run the following command
> ```
> ssh-keygen -b 4096 -C “lab"
> ```
>	Press Enter for location;
>	Leave passphrase blank and press enter;
>	Press Enter for confirmation of passphrase;
>The key will be saved to /Users/USERNAME/.ssh/id_rsa.pub
>
>To view the public key run the following command
> ```
> 'cat /Users/<USERNAME>/.ssh/id_rsa.pub'
> ```
>Replacing the username on your machine with USERNAME
> Copy this text and use it in the directions that follow.
>
> Windows: Use [PuttyGen] (https://www.ssh.com/academy/ssh/putty/windows/puttygen)<br>
>{End Optional}

Copy the [public key] (pubkey_rsa) from the github repo.

Expand the Compute Menu and select ‘SSH Keys’
Choose your region that you want to complete this lab in from the drop down in the header.
Click blue “Create +” 
SSH Keys are stored in the region. Make sure is set to the region you started with.
Name: MiniLab
Resource Group: Default
Public Key: Paste the public key you either created (id_rsa.pub) or copied from github










Create VPC

Choose VPC’s within VPC Infrastructure
	Choose Region (Dallas or Washington DC)
	Name of VPC: vpc-<region>-qbr-demo (ex. vpc-south-qbr-demo)
	Resource Group: Default
	All other values and checkbox defaults are ok 

Note: you should have three default prefixes and subnets already created for you


Enter your VPC (vpc-east-qbr-demo) via the link in the list

Open Security Group for port 80
In the top right tile, locate ‘Default Security Group’ section and enter the security group via the link.
In the SG window, click the ‘Manage Rules’ in the bottom left tile.
Alternatively, you can navigate to the rules using rules tab across the top.
We need to create an Inbound Rule to allow port 80.
Click ‘Create +’ in the ‘Inbound Rules’ section.
Protocol: TCP
Port: port Range
Port Min: 80
Port Max: 80
Source Type: Any
Click Create




Add Public Gateway to VPC
Navigate back to your VPC.
	Expand ‘Network Section’
	Choose VPC’s
	Enter your VPC via the HyperLink

Scroll down to the ‘subnets’ section
Enter the subnet for Availability Zone 1 (ex. Washington DC 1) via link in list
In the bottom right-hand corner, turn on the ‘Public Gateway’ tick box to Attached
Click Attach on modal window.

Click the ‘Attached Resources’ tab across top
Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section

Choose ‘Intel’ type
Choose ‘Public’ Hosting Type
The correct location should be selected but if not, choose the same Location as you VPC that you created above

Name your instance ‘in-south-qbr-demo01’
Resource Group: Default
Operating System: Ubuntu 20.04
Profile: Default is OK
Choose the SSH Key created above
User Data: Use the user data file provided in github
	
	Scroll down to Networking Section
	Select you VPC you created in earlier step

Double check your parameters
	Click ‘Create Virtual Server’

This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes. 
Break: 90 seconds
Click the refresh icon <icon> (top right in the header of the instance list.
You Instance should be running now.
Enter your instance via the hyper link in the link 
If you added console permissions to your user account, you can access the vnc or serial console from the actions menu in the top right hand corner.

Add a floating IP
Scroll down to the ‘Network Interfaces’ section at the bottom of the screen.
Click the ‘edit’ pencil icon on the line of your network interface.
In the slide out window you will see a option for ‘Floating IP Address’.  In that drop down, select ‘Reserve a New Floating IP’, Click ‘Save’
Within about 10 seconds, you should see a publicly accessible IP address appear in your Network Interface line.  Click the IP which will copy it.


Destroy Items
FIP
Enter Instance Page	
Click the ‘edit’ pencil icon on the line of your network interface.
Select ‘Unbind Current Floating IP’, Click Save
Instance
	Within instance Page, choose Actions menu in the top right and select ‘Delete.
 














