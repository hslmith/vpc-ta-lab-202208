# VPC Tech Academy Lab

The following multi-zone architecture will be used

99999999mage






### URLS Needed in this Mini Lab
- [IBM Cloud Portal](http://cloud.ibm.com)
- [This Lab Repo](https://github.com/hslmith/vpc-mini-lab)


### Tools Needed for this Lab
- A SSH clinet (Terminal or Putty)
- [Install jq](https://stedolan.github.io/jq/download/) (a lightweight json command line processor)
- [Install the IBM CLOUD CLI](https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli#install-ibmcloud-cli)
- Install the VPC infrastrcture Plugin
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


## Create a SSH Key in the IBM Cloud Portal
>Before you can add a key in the IBM Cloud console, you must make your SSH key available. Your SSH key must be an RSA key with a key size of either 2048 bits or 4096 bits.


Copy the [public key](pubkey_rsa) from the github repo.

1. Expand the Compute Menu and select ‘SSH Keys’
2. Choose your region that you want to complete this lab in from the drop down in the header.
3. Click blue “Create +” 
4. SSH Keys are stored at the regional level. Confirm you have the correct region selected. Make sure it is set to the region you started with.
	
>Name: tavpclab<br>
>Resource Group: Default<br>
>Public Key: Paste the public key you either created in the optional section above(id_rsa.pub) or copy and paste (pubkey_rsa) from github<br>
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
8. Change the location to Washingon DC 1
9. Click Create
10. Click Create again
7. Enter 192.168.50.0/24 in the IP Range field
8. Change the location to Washingon DC 3
9. Click Create
10. Go back to the overview tab, scoll down and click to create 2 subnets in your shiny new VPC.

>Location: Washington DC 1
>Name: sn-vnf-ta-transit
>Resource Group: Default
>Virtual Private Cloud: your transit vpc for this lab
>IP Range Selection: Choose 128 in the Number of adddress drop down (this should create a subnet of 10.100.200.0/25)
>attach a public gateway for update
>Click Create Subnet
>
>
>Location: Washington DC 3
>Name: sn-client-ta-demo
>Resource Group: Default
>Virtual Private Cloud: your transit vpc for this lab
>IP Range Selection: Choose 128 in the Number of adddress drop down (this should create a subnet of 192.168.50.0/25)
>attach a public gateway for update
>Click Create Subnet
>
>
# Create Windows Client in transit VPC
1. You should be at you list of subnets in the region, click the subnet you just created in zone 1
2. While in the subnet window, click the ‘Attached Resources’ tab across top
4. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	
	Type: Intel
	Hosting Type: Public
	The correct location should be selected but if not, choose the same Location as you VPC that you created above (Zone 3)

	>***Name:*** in-<em>selected region</em>in-ta-client-demo01<br>
	>***Resource Group:*** Default<br>
	>***Operating System:*** Windows Server 2022 <br>
	>***Profile:*** mx2-2x16<br>

	![-](/assets/images/sc-profiles-memory.png)<br>
	
	>
	>***SSH keys:*** tavpclab<br>


	<br><br><br>
	![-](/assets/images/sc-github.png)
		
	>Scroll down to Networking Section<br>
	>Select you VPC you created in earlier step<br>
	>Double check your parameters<br>
	>Click ‘Create Virtual Server’<br>
	
This will take you back to the ‘Virtual Server’ list and you should see your instance in a starting state.  This will take a couple of minutes.


## Create VNF in transit VPC
1. You should be at you list of subnets in the region, click the subnet you just created in zone 1
2. While in the subnet window, click the ‘Attached Resources’ tab across top
4. Choose the blue ‘Create +’ button in the top right of the ‘Attached Instances’ section
	
	Type: Intel
	Hosting Type: Public
	The correct location should be selected but if not, choose the same Location as you VPC that you created above

	>***Name:*** in-<em>selected region</em>-ta-vnf-demo01<br>
	>***Resource Group:*** Default<br>
	>***Operating System:*** CeentOS-7<br>
	>***Profile:*** bx2-2x8<br>

	![-](/assets/images/sc-profiles-memory.png)<br>
	
	>
	>***SSH keys:*** tavpclab<br>


	<br><br><br>
	![-](/assets/images/sc-github.png)
		
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
5. Enter your instance via the hyper link in the link 
	>Note: If you added console permissions to your user account, you can access the vnc or serial console from the actions menu in the top right hand corner.

## Add a floating IP to your client (and to VNF for admin purposes)
1. Scroll down to the ‘Network Interfaces’ section at the bottom of the screen.
2. Click the ‘edit’ pencil icon on the line of your network interface.<br>
![-](/assets/images/sc-pencil-nic.png)
3. In the slide out window you will see a option for ‘Floating IP Address’.  In that drop down, select ‘Reserve a New Floating IP’
4. Click ‘Save’
5. Within about 10 seconds, you should see a publicly accessible IP address appear in your Network Interface line.  Click the IP which will copy it.
6. Paste the IP in any available web browser. You should have a functioning website.

### Add new Security group just for RDP and outbound
1. Click the Securuty Group Menu Option on the left
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
>Dsetination: Any
>
>4. Now that you created a SG that will allow RDP to your windows client, enter into the details of the SG and click the attached resouces tab.
>5. Under the attached interfaces section, click Edit Interfaces
>6. Choose eth0 of your Windows client instance
<br>

### Decrypt your Windows Server Instance password  








d

e
e

e





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
	1. In the top right tile, locate ‘Default Security Group’ section and enter the security group via the link.
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
	1. Expand ‘Network Section’
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
	>***SSH keys:*** minilab<br>
	>***User Data:*** Copy and paste the the [user_data](instance-user-data) file provided in github


	<br><br><br>
	![-](/assets/images/sc-github.png)
		
	>Scroll down to Networking Section<br>
	>Select you VPC you created in earlier step<br>
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
2. Click the ‘edit’ pencil icon on the line of your network interface.<br>
![-](/assets/images/sc-pencil-nic.png)
3. In the slide out window you will see a option for ‘Floating IP Address’.  In that drop down, select ‘Reserve a New Floating IP’
4. Click ‘Save’
5. Within about 10 seconds, you should see a publicly accessible IP address appear in your Network Interface line.  Click the IP which will copy it.
6. Paste the IP in any available web browser. You should have a functioning website.

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
	>***Resource Group:*** Default
	>***Operating System:*** Ubuntu 20.04
	>***Profile:*** Click 'view all profiles', select memory and choose 'mx2-2x16'<br>
	>***SSH keys:*** minilab<br>
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


## Create an Private Network Load Balancer for your instnaces
1. Click the Load Balancers Menu Option
2. Click the Blue "Create +" button to create a new load balancer instance
3. Use the following options
	
	>***Region:*** Selected Region for Lab<br>
	>***Name:*** "nlb-us-east-ta-pri-demo"<br>
	>***Resource Group:*** Default<br>
	>***Load Balancer:*** Network Load Balancer<br>
	>***Virtual Private Cloud:*** Your Lab VPC (ex. vpc-us-east-ta-work1-demo)<br>
	>***Type:*** Private<br>
	>***Subnets:*** Choose the two subnets in Zone 1 and Zone 2<br>
	
4. Create Backend Pool
	
	>Click Black "Create +" Button<br>
	>***Name:*** pool-ta-work-demo<br>
	>***Protocol:*** HTTP<br>
	>***Session Stickiness:*** none<br>
	>***Proxy Protocol:*** Disabled<br>
	>***Method:*** Round Robin<br>
	>***Health Check Options:*** Leave all default<br>
	>Click Create <br>
	
5. Now that your pool is created, click the new Attach Server Link
6. Under the VPC Devices Tab, click the subnets that correspond to your VPC
7. Add your two instnaces to the Back-end Pool by ticking the checkboxes and clicking "Configure port"
8. Use port 80 for both instnances and click attach.  This will take you back to the ALB form
9. Create Front End Listener
	
	>Click Black "Create Listener +" Button and configre the following<br>
	>***Pool:*** pool-ta-work-demo<br>
	>***Protocol:*** TCP<br>
	>***Listener Port:*** 80<br>
	>Click Create<br>

10. Click the "Create Load Balancer" button.  This will take you back to your list of Load Balancers and you should see your creating.

This will take a couple of minutes to create


## Enable packet forwarding on CENTOS 7
login as root using the ssh you added to the portal above and issue the following command
~~~
sysctl -w net.ipv4.ip_forward=1
~~~

## Edit Routing tables
1. Navigate to routing tables
2. Create new routing table 
3. Name: ingress-client
4. VPC: your transit VPC
5. Traffic: Ingress
6. AIngress Properties: VPC Zone


## Deploy IBM CLoud Service
### Deploy Transit Gateway vi CLI
1. Run the following command to see a list of possible locations
~~~
bmcloud tg locations
~~~
2. Using the appropiate location create a transit gateway using the following command
~~~
TECHLAB_TGW_ID=$(ibmcloud tg gwc --name tgw-ta-demo --location us-east --output json | jq -r .id)
~~~
Retrieve the CRN's of your VPC to add as connections
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
2. Create a new zone in your newly created instnace 
~~~
DNS_ZONE_ID=$(ibmcloud dns zone-create techlab.com -i $DNS_INSTANCE --output json | jq -r .id)
~~~
3. to Add Permitted Netowrks to your zone.  We first need to find the crn of your exising VPC's. Run the following command and replace the name of your vpc with your vpc.
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


5. Finally add a 'CNAME' record for your network load balancer instance. Retrieve the loadbalancer hostname (replace the name of the load balancer with your)
~~~
NLB_HOSTNAME=$(ibmcloud is load-balancer nlb-us-east-ta-pri-demo --output json | jq -r .hostname)
~~~
add a 'CNAME' record to your zone 
~~~
ibmcloud dns resource-record-create $DNS_ZONE_ID --type CNAME --name www --cname $NLB_HOSTNAME -i $DNS_INSTANCE
~~~




# Destroy your Items

 














