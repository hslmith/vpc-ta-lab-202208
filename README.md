# VPC Tech Academy Lab
This lab will walk you through setting up load balanced cluster of web servers in a private (workload) VPC that can only be accessed via a transit VPC and basic VNF firewall. This lab will employ a mix of IBM Cloud Portal and CLI usage to expose you to both.
You will deploy the following services by the end of this lab.

- Multiple VPC's
- VSI Virtual Machines
- Custom Subnets
- Custom Routes
- DNS Services
- Transit Gateway
- Network Load Balancer
- Floating IP
- Public Gateway

The CLI portion of this lab uses session variables that may not persist if you logout or close your terminal.

- Check for user permissions. Be sure that your user account has sufficient permissions to create and manage IBM Cloud VPC resources.
- By default, the permission needed to "enable IP Spoofing" if turned off, you will need to add it to your profile via IAM.
	+ In the MANAGE menu across the top, choose Access (IAM).
	+ Click Users in the left navigation
	+ Enter your profile by clicking on your name.
	+ Choose the 'Access' tab across the top.
	+ Click Blue 'Assign Access+' button.
	+ You want Access Policy chosen for an individual.
	+ IN Service, Search for 'VPC Infrastructure Services'
	+ Click Blue Next
	+ Resources: All Resources -> Next
	+ Under Roles and Actions, you want to choose IP Spoofing Operator, while your at it go ahead and add Console Administrator as well.
	+ Click 'Review'
	+ Then blue 'Add' button and now it should be in your cart on the right.
	+ Lastly, click Assign
	+ You will receive an error if the access policy already exists on your profile.

![-](/assets/images/iam-vpc.png)<br><br><br>




### The following multi-zone architecture will be used

![-](/assets/images/TA-LAB-TransitDemo.png)<br>





### URLS Needed in this Mini Lab
- [IBM Cloud Portal](http://cloud.ibm.com)
- [This Lab Repo](https://github.com/hslmith/vpc-mini-lab)


### Tools Needed for this Lab
- A SSH clinet (Terminal or Putty)
- [Install jq](https://stedolan.github.io/jq/download/) (a lightweight json command line processor)
- [Install the IBM CLOUD CLI](https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli#install-ibmcloud-cli)
- Install the VPC infrastructure Plugin
~~~
ibmcloud plugin install vpc-infrastructure
~~~
- Install the DNS Services Plugin
~~~
ibmcloud plugin install cloud-dns-services
~~~
- Install the transit gateway plugin
~~~
ibmcloud plugin install tg
~~~

## Login to IBM Cloud Portal

Decide on a Region to create this demo and keep a mental note: US South (Dallas) or US East Washington DC<br>
I have chosen **Washington DC us-east** for this lab 


## Navigate to VPC Infrastructure in IBM Cloud Portal
1. Click top left hamburger menu  
2. Scroll down to **VPC Infrastructure**
3. Click the Pin next to VPC Infrastructure to pin the menu option to the top.
4. Choose **Overview** within the **VPC Infrastructure** menu. This will take you into the VPC landing page.


## Add your SSH Key in the IBM Cloud Portal
We will use the key you created during the prework.  If you have not already added it to the cloud portal, please follow the directions below.

If you do not have one created,  [Create a SSH Key](https://cloud.ibm.com/docs/vpc?topic=vpc-ssh-keys#locating-ssh-keys).

1. Expand the Compute Menu and select ‘SSH Keys’
2. Choose your region that you want to complete this lab in from the drop down in the header.
3. Click blue “Create +” 
4. SSH Keys are stored at the regional level. Confirm you have the correct region selected. Make sure it is set to the region you started with.
	
>Name: tavpclab<br>
>Resource Group: Default<br>
>Public Key: Paste your public key you created (id_rsa.pub)<br>
![-](/assets/images/sshkey-create.png)

Click Create<br>




## Create your Transit VPC
A transit  or "Hub" VPC serves as a centralized point for routing network traffic to/from the "Spoke" VPCs where workloads are running.
A transit "Hub" VPC serves as a centralized point for routing network traffic to/from the "Spoke" VPCs where workloads are running

1. Choose [VPCs](https://cloud.ibm.com/vpc-ext/network/vpcs) within VPC Infrastructure Menu
2. Choose your region that you want to complete this lab in from the drop down in the header.
3. Click the blue "Create +" button.

>***Name:*** vpc-**@selected-region**-ta-transit-demo (ex. vpc-us-east-ta-transit-demo)<br>
>***Resource Group:*** Default<br>
>Uncheck "Create a default prefix for each zone" box as we will create a small one for our VNF/Firewall<br>
>All other values and checkbox defaults are ok<br>

>Click ***Create Virtual Private Cloud***<br><br>

4. Now that your VPC is create, click name in the list to view the details.
5. In the top right tile, click "Manage address prefixes"
6. Click Create
7. Enter 10.100.200.0/24 in the IP Range field
8. Change the location to Washington DC 1
9. Click Create
10. Click Create again
7. Enter 192.168.50.0/24 in the IP Range field
8. Change the location to Washington DC 3
9. Click Create
10. Go back to the overview tab, scroll down and click to create 2 subnets in your shiny new VPC.

>Location: Washington DC 1<br>
>Name: 
>~~~
>sn-vnf-ta-transit
>~~~ 
><br>
>Resource Group: Default<br>
>Virtual Private Cloud: your transit vpc for this lab<br>
>IP Range Selection: Choose 128 in the Number of adddress drop down (this should create a subnet of 10.100.200.0/25)<br>
>attach a public gateway for update<br>
>Click Create Subnet<br>
><br>
><br>
>Location: Washington DC 3<br>
>Name: 
>~~~ 
>sn-client-ta-demo
>~~~ 
><br>
>Resource Group: Default<br>
>Virtual Private Cloud: your transit vpc for this lab<br>
>IP Range Selection: Choose 128 in the Number of adddress drop down (this should create a subnet of 192.168.50.0/25)<br>
>attach a public gateway for update<br>
>Click Create Subnet<br>
><br>
>
# Create Windows Client in transit VPC
1. You should be at you list of subnets in the region, click the subnet you just created in zone 3 (sn-client-ta-demo)
2. While in the subnet window, click the ‘Attached Resources’ tab across top
4. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	
	Type: Intel
	Hosting Type: Public
	The correct location should be selected but if not, choose the same Location as you VPC that you created above (Zone 3)

	>***Name:*** in-<em>selected region</em>-in-ta-client-demo01 ex. in-us-east-ta-client-demo01<br>
	>***Resource Group:*** Default<br>
	>***Operating System:*** Windows Server 2022 <br>
	>***Profile:*** mx2-2x16<br>

	![-](/assets/images/sc-profiles-memory.png)<br>
	
	>
	>***SSH keys:*** tavpclab<br>
	>Scroll down to Networking Section<br>
	>Select you VPC you created in earlier step<br>
	>Double check your parameters<br>
	>Click ‘Create Virtual Server’<br>
	
This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes.


## Create VNF virtual machine in transit VPC
1. You should be at you list of virtual machines in the region.
2. In this list, you should also see your transit next to the instance you just created, click into the VPC.
3. While in the VPC windows, scroll down to 'Subnets in this VPC' section.
4. Click the subnet you created for zone 1 (sn-vnf-ta-transit)
2. While in the subnet window, click the ‘Attached Resources’ tab across top
4. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	
	Type: Intel
	Hosting Type: Public
	The correct location should be selected but if not, choose the same Location as you VPC that you created above

	>***Name:*** in-<em>selected region</em>-ta-vnf-demo01<br>
	>***Resource Group:*** Default<br>
	>***Operating System:*** CeentOS-7<br>
	>***Profile:*** bx2-2x8<br>

	![-](/assets/images/sc-profiles-balanced.png)<br>
	
	>
	>***SSH keys:*** tavpclab<br>
	>Scroll down to Networking Section<br>
	>Select you VPC you created in earlier step<br>
	>Edit the interface and enable "Allow IP Spoofing"
	>Double check your parameters<br>
	>Click ‘Create Virtual Server’<br>
	
This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes.
<br>

### Break: 90 seconds

<br>

4. Click the refresh icon <img src="/assets/images/refresh_icon.png" width="40" height="40"> (top right in the header of the instance list.<br>
Your Instance should be running now.


## Add a floating IP to your windows client and to CentOS VNF for admin purposes and enable IP spoofing
1. Enter your VNF instance via the hyper link in the link 
2. Scroll down to the ‘Network Interfaces’ section at the bottom of the screen.
2. Click the ‘edit’ pencil icon on the line of your network interface.<br>
![-](/assets/images/sc-pencil-nic.png)
3. In the slide out window you will see a option for 'Allow IP Spoofing, turn this ON. Below you will see ‘Floating IP Address’.  In that drop down, select ‘Reserve a New Floating IP’
4. Click ‘Save’
5. Within about 10 seconds, you should see a publicly accessible IP address appear in your Network Interface line.
6. Head back to the list of virtual instances and enter your windows client and we will ONYL ADD A FLOATING IP to the client.
7. Scroll down to the ‘Network Interfaces’ section at the bottom of the screen.
8. Click the ‘edit’ pencil icon on the line of your network interface.
9. In the slide out window you will see ‘Floating IP Address’.  In that drop down, select ‘Reserve a New Floating IP’
4. Click ‘Save’

### Add new Security group just for RDP and outbound
1. Click the Security Group Menu Option on the left
2. Click Blue Create
3. use the following values
>Location: Washington DC
>Name: sg-rdp-only
>Resource Group: Default
>VPC: vpc-us-east-ta-transit-demo
>CREATE RULE INBOUND
>PORT RANGE: 3389
>Source: Any
>
>CREATE RULE OUTBOUND
>PORT RANGE: ANY
>Destination: Any
>
>
>5. Under the 'Edit Interfaces' Section, tick the box that corresponds to your client VSI. 
>6. Click 'Create Security Group'
<br>

### Decrypt your Windows Server Instance password
1. Locate and take note of your instance id using the IBM CLOUD CLI.  Start by logging in.
~~~
ibmcloud login --sso
~~~
2. Target the correct region where you are building this lab
~~~
ibmcloud target -r us-east

~~~
2. Run the following which will list all instances in the target region
~~~
ibmcloud is instances
~~~
2. Using the instance id for your windows instance from the above output, run the following command replacing 'INSTANCE_ID with the ID found in the previous list' and the path to your private key after the '@'.  The output should provide you with a password that you can now use to login to you windows instance via RDP. NOTE: Windows users may need to use the full path. i.e. 'C:\users\labuser\ssh\privatekey'
~~~
ibmcloud is instance-initialization-values INSTANCE_ID --private-key @~/.ssh/id_rsa
~~~

3. Make a note of your windows password


## Create your Work Load VPC

1. Choose [VPCs](https://cloud.ibm.com/vpc-ext/network/vpcs) within VPC Infrastructure Menu
2. Choose your region that you are completing this lab in from the drop down in the header.
3. Click the blue "Create +" button.

>***Name:*** vpc-**selected-region**-ta-work1-demo (ex. vpc-us-east-ta-work1-demo)<br>
>***Resource Group:*** Default<br>
>All other values and checkbox defaults are ok<br>

>Note: you should have three default prefixes and subnets already created for you<br>
>Click ***Create Virtual Private Cloud***<br><br>


## Edit Security Group
The last operation should return to your list of VPCs in the region you have been working with.<br>
1. Enter your VPC (vpc-us-east-ta-work1-demo) via the link in the list
2. Open Security Group for port 80
	1. In the top left tile, locate ‘Default Security Group’ section and enter the security group via the link.
 	2. In the SG window, click the ‘Manage Rules’ in the bottom left tile. Alternatively, you can navigate to the rules using rules tab across the top.
	3. Create an Inbound Rule to allow port 80.
		1. Click ‘Create +’ in the ‘Inbound Rules’ section.
		>***Protocol:*** TCP<br>
		>***Port:*** port Range<br>
		>***Port Min:*** 80<br>
		>***Port Max:*** 80<br>
		>***Source Type:*** Any<br>
		![-](/assets/images/sc-sg-inbound-edit.png)
		2. Click Create




## Add Public Gateway to VPC subnet
1. Navigate back to your VPC.
	1. Expand ‘Network Section’ in left menu (If collapsed)
	2. Choose VPC’s
	3. Enter your VPC via the HyperLink
2. Scroll down to the ‘subnets’ section
3. Enter the subnet for Availability Zone 1 (ex. Washington DC 1) via link in list
4. In the bottom right-hand corner, turn on the ‘Public Gateway’ tick box to Attached
5. Click Attach on modal window.




## Create Workload Virtual Machine in VPC we just created and configured
1. If you are no longer on the VPC detail, navigate to your VPC details page using the VPCs menu option.
2. Scroll down to the subnet section and enter the subnet for zone 1.
3. While in the VPC window, click the ‘Attached Resources’ tab across top
4. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	
	Type: Intel
	Hosting Type: Public
	The correct location should be selected but if not, choose the same Location as you VPC that you created above

	>***Name:*** in-<em>selected region</em>in-ta-demo01<br>
	>***Resource Group:*** Default<br>
	>***Operating System:*** Ubuntu 20.04<br>
	>***Profile:*** Click 'view all profiles', select memory and choose 'mx2-2x16'<br>

	![-](/assets/images/sc-profiles-memory.png)<br>
	
	>
	>***SSH keys:*** tavpclab<br>
	>***User Data:*** Copy and paste the [user_data](instance-user-data) file provided in github


	<br><br><br>
	![-](/assets/images/sc-github.png)
		
	>Scroll down to Networking Section<br>
	>Select your workload VPC you created in earlier step ex. vpc-us-east-ta-work1-demo<br>
	>Double check your parameters<br>
	>Click ‘Create Virtual Server’<br>
	
This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes.
<br>

### Break: 90 seconds

<br>

4. Click the refresh icon <img src="/assets/images/refresh_icon.png" width="40" height="40"> (top right in the header of the instance list.<br>
Your Instance should be running now.
<br>

 Add a second zone and a load balancer.

## Navigate to VPC
1. Click VPC's in the menu 


## Add Public Gateway to VPC subnet
1. Enter your VPC via the HyperLink
2. Scroll down to the ‘subnets’ section
3. Enter the subnet for <mark>Availability Zone 2 (ex. Washington DC 2)</mark> via link in list
4. In the bottom right-hand corner, turn on the ‘Public Gateway’ tick box to Attached
5. Click Attach on modal window.

## Create Virtual Machine in  Zone 2
1. If you are no longer on the VPC detatil, navigate to your VPC using the VPCs menu option.
2. While in the VPC window, click the ‘Attached Resources’ tab across top
3. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	
	>***Architecture:*** Intel<br>
	>***Hosting Type:*** Public<br>
	>The correct location should be selected but if not, choose the same Location as you VPC that you created above
	>
	>***Name:*** in-us-east-ta-demo02<br>
	>***Resource Group:*** Default<br>
	>***Operating System:*** Ubuntu 20.04<br>
	>***Profile:*** Click 'view all profiles', select memory and choose 'mx2-2x16'<br>
	>***SSH keys:*** tavpclab<br>
	>***User Data:*** Copy and paste the the [user_data](instance-user-data) file provided in github<br><br>
	>	
	>Scroll down to Networking Section<br>
	>Select you VPC you created in earlier step<br><br>
	>
	>Double check your parameters<br>
	>Click ‘Create Virtual Server’<br>

This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes.
<br><br><br>

### Break: 90 seconds

<br>

4. Click the refresh icon <img src="/assets/images/refresh_icon.png" width="40" height="40"> (top right in the header of the instance list.<br>
Your Instance should be running now.


## Create an Private Network Load Balancer for your instances
1. Click the Load Balancers Menu Option
2. Click the Blue "Create +" button to create a new load balancer instance
3. Use the following options
	
	>***Region:*** Selected Region for Lab<br>
	>***Name:*** "nlb-us-east-ta-pri-demo"<br>
	>***Resource Group:*** Default<br>
	>***Load Balancer:*** Network Load Balancer<br>
	>***Virtual Private Cloud:*** Your Lab VPC (ex. vpc-us-east-ta-work1-demo)<br>
	>***Type:*** Private<br>
	>***Subnets:*** Choose Zone 1<br>
	
4. Create Backend Pool
	
	>Click Black "Create +" Button<br>
	>***Name:*** pool-ta-work-demo<br>
	>***Protocol:*** TCP<br>
	>***Session Stickiness:*** none<br>
	>***Proxy Protocol:*** Disabled<br>
	>***Method:*** Round Robin<br>
	>***Health Check Options:*** Leave all default<br>
	>Click Create <br>
	
5. Now that your pool is created, click the new Attach Server Link
6. Under the VPC Devices Tab, click the subnets that correspond to your VPC
7. Add your two instances to the Back-end Pool by ticking the check boxes and clicking "Configure port"
8. Use port 80 for both instances and click attach.  This will take you back to the ALB form
9. Create Front End Listener
	
	>Click Black "Create Listener +" Button and configre the following<br>
	>***Pool:*** pool-ta-work-demo<br>
	>***Protocol:*** TCP<br>
	>***Listener Port:*** 80<br>
	>Click Create<br>

10. Click the "Create Load Balancer" button.  This will take you back to your list of Load Balancers and you should see your creating.

This will take a couple of minutes to create


## Enable packet forwarding on CENTOS 7
login as root using the ssh key you added to the portal above and issue the following command
~~~
sysctl -w net.ipv4.ip_forward=1
~~~

## Edit Routing tables
1. Navigate to routing tables
2. Choose the Transit VPC from the drop down
3. Enter the default routing table via the link
4. Click Create 
5. ENter the following details
><br>
>Zone: Zone 3
>Name: vnf-egress<br>
>Destination CIDR: 10.241.0.0/16<br>
>Action: Deliver<br>
>Next Hop: The Private IP of your VNF (Most likely 10.100.200.4)<br>
><br>


## Deploy IBM CLoud Service
### Deploy Transit Gateway vi CLI
1. Run the following command to see a list of possible locations
~~~
ibmcloud tg locations
~~~
2. Using the appropiate location create a transit gateway using the following command
~~~
TECHLAB_TGW_ID=$(ibmcloud tg gwc --name tgw-ta-demo --location us-east --output json | jq -r .id)
~~~
Retrieve the CRN's of your VPCs to add as connections. You may need to replace the name of your VPC in the first two commands to match.
~~~
TRANSIT_VPC_CRN=$(ibmcloud is vpc vpc-us-east-ta-transit-demo --output json | jq -r .crn)
~~~
~~~
WORK_VPC_CRN=$(ibmcloud is vpc vpc-us-east-ta-work1-demo --output json | jq -r .crn)
~~~
~~~
TRANSIT_CONN_ID=$(ibmcloud tg cc $TECHLAB_TGW_ID --name transit_vpc --network-type vpc --network-id $TRANSIT_VPC_CRN --output json | jq -r .id)
~~~
~~~
WORK_CONN_ID=$(ibmcloud tg cc $TECHLAB_TGW_ID --name work_vpc --network-type vpc --network-id $WORK_VPC_CRN --output json | jq -r .id)
~~~

Verify you have a gateway deployed with connection to both of your VPC's.  Status should say 'attached'
~~~
ibmcloud tg connections $TECHLAB_TGW_ID
~~~


### Deploy DNS Services using CLI
1. Use the CLI to create a dns service instance by the name of 'dns-tech-lab' using the standard plan ID and assigning it to the default resource group.
~~~
DNS_INSTANCE=$(ibmcloud dns instance-create dns-tech-lab 2c8fa097-d7c2-4df2-b53e-2efb7874cdf7 --resource-group Default --output json | jq -r .guid)
~~~
2. Create a new zone in your newly created instance 
~~~
DNS_ZONE_ID=$(ibmcloud dns zone-create techlab.com -i $DNS_INSTANCE --output json | jq -r .id)
~~~
3. to Add Permitted Networks to your zone.  We first need to find the crn of your existing VPC's. Run the following command and replace the name of your vpc with your vpc.
~~~
TRANSIT_VPC_CRN=$(ibmcloud is vpc vpc-us-east-ta-transit-demo --output json | jq -r .crn)
~~~
~~~
WORK_VPC_CRN=$(ibmcloud is vpc vpc-us-east-ta-work1-demo --output json | jq -r .crn)
~~~

4. Add permitted networks to your zone
~~~
TRANSIT_PERMITTED_NET_ID=$(ibmcloud dns permitted-network-add $DNS_ZONE_ID --vpc-crn $TRANSIT_VPC_CRN --type vpc -i $DNS_INSTANCE --output json)
~~~
~~~
WORK_PERMITTED_NET_ID=$(ibmcloud dns permitted-network-add $DNS_ZONE_ID --vpc-crn $WORK_VPC_CRN --type vpc -i $DNS_INSTANCE --output json)
~~~


5. Finally add a 'CNAME' record for your network load balancer instance. Retrieve the loadbalancer host name (replace the name of the load balancer with your)
~~~
NLB_HOSTNAME=$(ibmcloud is load-balancer nlb-us-east-ta-pri-demo --output json | jq -r .hostname)
~~~
add a 'CNAME' record to your zone 
~~~
ibmcloud dns resource-record-create $DNS_ZONE_ID --type CNAME --name www --cname $NLB_HOSTNAME -i $DNS_INSTANCE
~~~


##Test your deployment
1. Using an RDP client, connect to you windows server using the password you decrypted earlier
2. Open a browser and browse to www.techlab.com



# Please Destroy your resources once complete

 














