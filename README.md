# VPC Mini Lab
### URLS Needed in this Mini Lab
- [IBM Cloud Portal](http://cloud.ibm.com)
- [This Lab Repo](https://github.com/hslmith/vpc-mini-lab)



## Login to IBM Cloud Portal

Decide on a Region to create this demo and keep a mental note: US South (Dallas) or US East Washington DC<br>
I have chosen **Dallas us-south** for this lab 


## Navigate to VPC
1. Click top left hamburger menu  
2. Scroll down to **VPC Infrastructure**
3. Click the Pin next to VPC Infrastructure to pin the menu option to the top.
4. Choose **Overview** within the **VPC Infrastructure** menu. This will take you into the VPC landing page.


## Create a SSH Key
>Before you can add a key in the IBM Cloud console, you must make your SSH key available. Your SSH key must be an RSA key with a key size of either 2048 bits or 4096 bits.


>### {Optional}
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
> cat /Users/<USERNAME>/.ssh/id_rsa.pub'
> ```
>Replacing the username on your machine with USERNAME. <br>
>Copy this text and use it in the directions that follow.
>
> Windows: Use [PuttyGen](https://www.ssh.com/academy/ssh/putty/windows/puttygen)<br>
>### {End Optional}

Copy the [public key](pubkey_rsa) from the github repo.

1. Expand the Compute Menu and select ‘SSH Keys’
2. Choose your region that you want to complete this lab in from the drop down in the header.
3. Click blue “Create +” 
4. SSH Keys are stored in the region. Make sure it is set to the region you started with.
	
>Name: minilab<br>
>Resource Group: Default<br>
>Public Key: Paste the public key you either created in the optional section above(id_rsa.pub) or copy and paste (pubkey_rsa) from github<br>

<br>

## Create VPC

1. Choose [VPCs](https://cloud.ibm.com/vpc-ext/network/vpcs) within VPC Infrastructure Menu
2. Choose your region that you want to complete this lab in from the drop down in the header.
3. Click the blue "Create +" button.

>***Name:*** vpc-<replace with east or south>-qbr-demo (ex. vpc-south-qbr-demo)<br>
>***Resource Group:*** Default<br>
>All other values and checkbox defaults are ok<br>

>Note: you should have three default prefixes and subnets already created for you<br>
>Click ***Create Virtual Private Cloud***<br><br>


## Edit Security Group
The last operation should return to your list of VPCs in the region you have been working with.<br>
1. Enter your VPC (vpc-east-qbr-demo) via the link in the list
2. Open Security Group for port 80
	1. In the top right tile, locate ‘Default Security Group’ section and enter the security group via the link.
 	2. In the SG window, click the ‘Manage Rules’ in the bottom left tile. Alternatively, you can navigate to the rules using rules tab across the top.
	3. Create an Inbound Rule to allow port 80.
		1. Click ‘Create +’ in the ‘Inbound Rules’ section.
		>***Protocol:*** TCP<br>
		>***Port:*** port Range<br>
		>***Port Min:*** 80<br>
		>***Port Max:*** 80<br>
		>***Source Type:*** Any<br>

		2. Click Create




## Add Public Gateway to VPC subnet
1. Navigate back to your VPC.
	1. Expand ‘Network Section’
	2. Choose VPC’s
	3. Enter your VPC via the HyperLink
2. Scroll down to the ‘subnets’ section
3. Enter the subnet for Availability Zone 1 (ex. Washington DC 1) via link in list
4. In the bottom right-hand corner, turn on the ‘Public Gateway’ tick box to Attached
5. Click Attach on modal window.

## Create Virtual Machine in VPC we just created and configured
1. If you are no longer on the VPC detail, navigate to your VPC details page using the VPCs menu option.
2. Scroll down to the subnet section and enter the subnet for zone 1.
3. While in the VPC window, click the ‘Attached Resources’ tab across top
4. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	
	Type: Intel
	Hosting Type: Public
	The correct location should be selected but if not, choose the same Location as you VPC that you created above

	>***Name:*** in-<selected region>-qbr-demo01<br>
	>***Resource Group:*** Default<br>
	>***Operating System:*** Ubuntu 20.04<br>
	>***Profile: Default Profile is OK<br>
	>Choose the SSH Key created in previous step<br>
	>***User Data:*** Copy and paste the the ***instance-user-data*** file provided in github

	![-](/assets/images/sc-github.png)
		
	>Scroll down to Networking Section<br>
	>Select you VPC you created in earlier step<br>
	><br>
	>Double check your parameters<br>
	>Click ‘Create Virtual Server’<br>
	
This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes.
<br>

### Break: 90 seconds

<br>

4. Click the refresh icon <img src="/assets/images/refresh_icon.png" width="40" height="40"> (top right in the header of the instance list.<br>
Your Instance should be running now.
5. Enter your instance via the hyper link in the link 
	>Note: If you added console permissions to your user account, you can access the vnc or serial console from the actions menu in the top right hand corner.

## Add a floating IP
1. Scroll down to the ‘Network Interfaces’ section at the bottom of the screen.
2. Click the ‘edit’ pencil icon on the line of your network interface.
3. In the slide out window you will see a option for ‘Floating IP Address’.  In that drop down, select ‘Reserve a New Floating IP’
4. Click ‘Save’
5. Within about 10 seconds, you should see a publicly accessible IP address appear in your Network Interface line.  Click the IP which will copy it.
6. Paste the IP in any available web browser. You should have a functioning website.

<br>



# Optional 
### If that was a breeze, lets increase the avability of our website by adding a second zone and a loab balancer.

## Navigate to VPC
1. Click VPC's in the menu 


## Add Public Gateway to VPC subnet
1. Enter your VPC via the HyperLink
2. Scroll down to the ‘subnets’ section
3. Enter the subnet for ==Availability Zone 2 (ex. Washington DC 2)== via link in list
4. In the bottom right-hand corner, turn on the ‘Public Gateway’ tick box to Attached
5. Click Attach on modal window.

## Create Virtual Mahcine in  Zone 2 
1. If you are no longer on the VPC detatil, navigate to your VPC using the VPCs menu option.
2. While in the VPC window, click the ‘Attached Resources’ tab across top
3. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	```
	Choose ‘Intel’ type
	Choose ‘Public’ Hosting Type
	The correct location should be selected but if not, choose the same Location as you VPC that you created above

	Name your instance **in-south-qbr-demo02**
	Resource Group: Default
	Operating System: Ubuntu 20.04
	Profile: Default is OK
	Choose the SSH Key created above
	User Data: Copy and paster the the ***instance-user-data*** file provided in github
		
	Scroll down to Networking Section
	Select you VPC you created in earlier step

	Double check your parameters
	Click ‘Create Virtual Server’
	```
This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes.
<br>

### Break: 90 seconds

<br>

4. Click the refresh icon <img src="/assets/images/refresh_icon.png" width="40" height="40"> (top right in the header of the instance list.<br>
Your Instance should be running now.


## Create an Application Load Balancer 
1. Clike the Laod Balancers Menu Option
2. Click the Blue "Create +" button to create a new load balancer instance
3. Use the following options
	```
	Region: Selected Region for Lab
	Name: "alb-south-qbr-lab"
	Resource Group: Default
	Load Balancer: Application Load Balancer
	Virtual Private Cloud: Your Lab VPC (ex. vpc-south-qbr-demo)
	Type: Public
	Subnets: Choose the two subnets in Zone 1 and Zone 2
	```
4. Create Backend Pool
	```
	Click Black "Create +" Button
	Name: pool-qbr-lab
	Protocol: HTTP
	Session Stickiness: none
	Proxy Protocol: Disabled
	Method: Round Robin
	Health Check Options: Leave all default
	Click Create 
	```
5. Now that your pool is created, click the new Attach Server Link
6. Under the VPC Devices Tab, click the subnets that correspond to your VPC
7. Add your two instnaces to the Back-end Pool by ticking the checkboxes and clicking "Configure port and weight"
8. Use port 80 for both instnances and click attach.  This will take you back to the ALB form
9. Create Front End Listener
	```
	Click Black "Create Listener +" Button and configre the following
	Pool: pool-qbr-lab
	Protocol: HTTP
	Listener Port: 80
	Click Create
	```
10. Click the "Create Load Balancer" button.  This will take you back to your list of Load Balancers and you should see your creating.

This will take a couple of minutes to create

### Break: 120 seconds

11. Click the refresh icon <img src="/assets/images/refresh_icon.png" width="40" height="40"> (top right in the header of the LB list occasionally until the Status is **Active** .<br>
In the list, you should see a hostname. that will looks something like ########-us-@@@@.lb.appdomain.cloud.  Copy that host name and paste it in any available broswer.  

## Congratulations, You should see your fully load blanacer website across two zones.



# Destroy your Items
FIP
Enter Instance Page	
Click the ‘edit’ pencil icon on the line of your network interface.
Select ‘Unbind Current Floating IP’, Click Save
Instance
	Within instance Page, choose Actions menu in the top right and select ‘Delete.
 














