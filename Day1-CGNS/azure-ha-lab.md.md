---
created_at: 30-10-2025
modified_at: 30-10-2025
---
### Azure - HA lab

Note: The lab environment created is a sandbox environment and apart from some IAM restrictions, you're free to create whichever Check Point solution you wish.
If you're attending a Check Point led session, your instructor will suggest which architectures to deploy (typically High Availability and an Autoscaling option). If you are newer to public cloud platforms and Check Point, it might be beneficial to deploy a single gateway solution which is a little easier to get to grips with. 

**Lab goal**: Create a Check Point highly available solution in your Azure sandbox environment. 

[Latest updates for Azure templates - SK110194](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk110194)

Please refer to this diagram if you get lost at any point. The IP addresses and subnet definitions are used throughout the lab guide - but don't worry if you make a mistake and use the wrong address. Just make sure you keep a note of what you're using and that the vNETs always use a larger range / mask than the subnets within them. 
It will make the lab a lot easier if you choose **unique** ranges for each of your vNETs.


![[azure-ha-topology.png]]

** Pre-requisites **
- Check Point Management station (this session is based around having a self-managed server, though Smart-1 Cloud can also be used. Ask your instructor). There is a guide for deploying this in the Day 1 docs.
- Azure Portal Access.
- Windows PC Smart Console and SSH client
- Windows PC must have unrestricted external access
	- If you have restrictions on your laptop that prevent either of these, you can probably get by using CloudShell for SSH / CLI access and a Windows VM with RDP access.

**Module 1- Creating a CloudGuard High Availability Cluster**

In this Module, we will be deploying a Check Point CloudGuard High Availability Cluster into a new Resource Group which will be managed by a Check Point Smart Center Server

[sk110194](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk110194)- Deploying a Check Point Cluster in Microsoft Azure

**Step #1 Deploying CloudGuard High Availability Cluster in Azure R81.20/R82**

1. Log in to the Azure Portal at [https://portal.azure.com](https://portal.azure.com) using the credentials provided.
2. Select **Create a resource.**
3. In the search box enter **Check Point** and select the **CloudGuard Network Security – Firewall & Threat Prevention** 
4. Then select from the dropdown list the **CloudGuard High Availability**
5. Select **Create**
6. Complete the following Basic details

|**Selection**|**Choice**|**Notes**|
|---|---|---|
|Subscription|Choose a Subscription|Select the subscription to deploy under. In the lab, there is only one option|
|Resource Group|**Use Existing:**<br><br>**ODL-<*>-<*>-02**|Used to identify resources of similar life-cycle|
|Region|East US (Or region of choice)|Choose a local region.<br><br>(West US 2, East US, Canada Central, Central US, South Central US)|
|Cluster Object Name|CPCluster|Name of your Cluster|
|Authentication Type|**Default** (Password)||
|Password|Checkpoint123!|Unique password for Check Point.|

7. Select **Next: Check Point Cluster Object settings >**

|**Selection**|**Choice**|**Notes**|
|---|---|---|
|Check Point Version|R82/R81.20|The version you would like to deploy|
|License Type|**Bring your Own License**||
|Virtual Machine Size|**Default** (Standard D4ds v5)||
|Default Shell|/etc/cli.sh or /bin/bash||
|SIC Key|Checkpoint123||
|Confirm SIC key|Checkpoint123||
|Quick connect to Smart-1 Cloud|**No**||
|Enable Maintenance Mode|**No**||
|Created a System Assigned Identity|**Default** (Yes)||
|Availability options|**Default** (Availability Set)||
|Bootstrap script|Leave blank||
|Automatic Download and updates|**Default** (Yes)||
|VM Disk type|**Default** (Premium)||
|Addition disk space|**Default** (0)||
|Enable CloudGuard Metrics|**Default** (Yes)||
|Deploy the Load Balancer with a floating IP|**Default** (No)||
|User public IP prefix|**Yes**||
|Create a public IP prefix|**Yes**||
|Quick connect to Smart-1 Cloud|**No**||

8. Select **Next: CloudGuard Advance Settings >**

|**Selection**|**Choice**|**Notes**|
|---|---|---|
|Default shell for admin user|**/etc/cli.sh** or **/bin/bash**||
|Enable Maintenance Mode|Leave blank||
|Confirm Password|Leave blank||
|Created a System Assigned Identity|**Default** (Yes)||
|Availability options|**Default** (Availability Set)||
|Bootstrap script|Leave blank||
|Automatic Download and updates|**Default** (Yes)||
|VM Disk type|**Default** (Premium)||
|Addition disk space|**Default** (0)||
|Enable CloudGuard Metrics|**Default** (Yes)||
|Deploy the Load Balancer with a floating IP|**Default** (No)||
|User public IP prefix|**No**||

9.    Select **Next: Network settings >**
10. In the Network settings, Select **Create New** for Virtual Network
11. In the Create Virtual network, configure it as follows:
12. Network settings configure the following:
- Virtual Network:
	- Name: **Cluster-vnet**
- Subnets:
	- Edit the vNet range to 10.6.0.0/16.
	- Frontend- 10.6.0.0/24
	- Backend- 10.6.1.0/24
- Save

10. Number of Virtual IPs (VIP): **1**
11. In the **Network Security Groups** select **Create New**
12. Leave the name as the **default**
13. Select **Review + Create**
14. The Purchase Agreement will now appear. Approve the Terms and Select **Create**

 

**Step 2 Configuring Cluster objects in Smart Console:**

1. Open up Smart Console to your Smart Center / Security Management Server
2. Create a new Cluster object using **Classic Mode** with the following details

- Name
	- Azure-Cluster
- Cluster IP
	- Cluster IPv4 public address

You can find the cluster IP address in the Azure portal when you select the Active cluster member's primary NIC > IP configuration > "cluster-VIP".

3.    Add each of the two members to the Cluster called **Azure-FW-<1-2>**. Use the Public IP for each of the objects. The Sic key will be **Checkpoint123**
4.    Update the Topology of the Cluster under **Network Management** as follows
- Click the **Get Interface Topology -> Get Interface with Topology**
- eth0
	- **Cluster+Primary Sync**
	- The Private VIP address is: 10.X.X.6. Note - You can find the cluster private VIP address in the Azure portal when you select the Active member primary NIC > IP configuration > cluster-vip
	- Disable **Anti-Spoofing**
	- Leads to -> Override -> **Internet** **(External)**

- eth1
	- **Sync**
	- Disable **Anti-Spoofing**

![[cluster-topology.png]]

5. Select **OK** to finish the configuration.
6. **Publish** the session.
7.    Modify the Security policy to **Any, Any, Accept**
8.    **Publish** and **Install Policy**.

**Step 3 Testing Cluster Failover:**

In this section, we will test basic cluster failover to ensure.

- SSH to each of the Cluster member's Public IP address and log in
- Confirm which unit is Active and Backup. 

1. This can be done by running the command:

	`cphaprob state`

Each unit should show its status and the status of the other member

2. Using the cluster configuration test script on each cluster member we will be able to confirm if each cluster member can communicate properly to the Azure API and able to make the necessary changes to the environment on a failover. This can be done by running the command:
	`$FWDIR/scripts/azure_ha_test.py`

Output example:

![[cluster-state.png]]

**Module 2- Configuring East/West Traffic (Spoke vNet)**

In this section, you will learn how to properly configure East-West inspection in a cluster environment.

-       Create vNet Spoke
-       Deploy Workload
-       Create vNet Peering
-       East-West UDR
-       Policy East/West Inspection

## Step 1: Deploy the Spoke vNets

1.    Log in to the Azure Portal at [https://portal.azure.com](https://portal.azure.com) using the credentials provided.
2.    Select **Create a resource.**
3.    In the search box enter **Virtual Network** and press **Enter**.
4. Select the **Virtual Network** option.
5. Select **Create**.
6. Configure the Spoke with the following:


| **Selection**        | **Choice**                                                   |
| -------------------- | ------------------------------------------------------------ |
| Subscription         | **Leave default**                                            |
| Resource Group:      | **Use Existing: ODL-<*>-<*>-04**                             |
| Virtual Network Name | **Spoke1-vnet**                                              |
| Region               | **Choose the same Region as your Cluster is deployed under** |

7. Select **Next: IP Addresses**.
8. Configure the IP Address with the following:
	**Delete the default IPv4 address space**
	New address: **10.10.0.0/16**
9. Select **add subnet**.
10. Use the following to create the new subnet.
	Subnet name: **Spoke-1-subnet**
	Subnet address range: **10.10.0.0/24**

11. Select **Add**.
12. Select **Review + create**
13. Select **Create.**
14. Repeat the steps to create Spoke 2 vNet and Spoke 2 subnet.
	Resource Group name: **Use Existing: ODL-<*>-<*>-04**
	Virtual Network Name: **Spoke2-vnet**
	Region: **Same region as the Cluster**
	vNet address range: **10.11.0.0/16**
	Subnet name: **Spoke-2-subnet**
	Subnet address range: **10.11.0.0/24**

## Step 2: Create the web servers for each spoke

In this section of the lab, you will be creating a virtual machine in each spoke.

1. Select **Create a resource.**
2. In the search box enter **Nginx on CentOS 8.2** and press **Enter**.
3. Select **Nginx on CentOS 8.2 (If not available select another NGIX image)**
4. Select **Create**
5. Fill in the following details:

|**Selection**|**Choice**|**Notes**|
|---|---|---|
|Subscription|Choose a Subscription|Select the subscription to deploy under. In the lab, there is only one option|
|Resource Group|**ODL-<*>-<*>-04**|The same Spoke that was created earlier on|
|Virtual Machine Name|Spoke-1-Workload or<br><br>Spoke-2-Workload|Name of your the Virtual Machine|
|Region|Same reason as you created the vNet for Spoke1||
|Availability Options|**No infrastructure redundancy required**||
|Security Type|**Default** (Standard)||
|Image|**Default**||
|VM Architecture|**Default**||
|Size|**B1s**||
|Authentication Type|**Password**||
|Username|**fwadmin**||
|Password|**Checkpoint123!**||

6. Select **Next: Disk >** Leave everything as default
7. Select **Next: Network settings >**
8. In the Network settings, Select the following details:

|**Selection**|**Choice**|
|---|---|
|Virtual Network|**Spoke1-vnet** or **Spoke2-vnet**|
|Subnet|**Spoke-1-subnet** or **Spoke-2-subnet**|
|Public IP|Select **None**|
|NIC network security groups|**Default** (Advanced)|
|Configure network security group|**Default** (New)|
|Delete public IP and NIC when VM is deleted|**Default** (Not Selected)|
|Enable accelerated network|**Default** (Not Selected)|
|Place this VM behind an existing Load Balancer|**Default** (Not Selected)|

9.    Select **Next: Management >** Leave everything as default
10. Select **Next: Monitoring >**
11. Ensure the option for **Boot diagnostics -> Enable with managed storage account** is selected.
12. Select **Review +** **Create**.
13. Approve the Terms and Select **Create**
14. **Repeat for Spoke2**

## Step 3: Create vNet Peer for each Spoke to the Check Point Cluster vNet

1. Go to the cluster resource group of the Check Point HA Cluster
	Example: ODL-<*>-<*>-02
2. Select the cluster vNet
	Example: Cluster-vnet
3. Select **Peerings** from the vNet Settings
4. Select **Add**.
5. Configure the two peerings as follows:
	- Peering link name: **HA-to-Spoke1**
	- Select the first two peering options
		- **Allow `VNet name` to access `peer VNet name`**
		- **Allow `VNet name` to receive forwarded traffic from `peer VNet name`**

	- Remote Peering Link name: **Spoke1-to-HA**
	- Virtual Network: **Spoke1**
	- Select the first two peering options
		- **Allow `VNet name` to access `peer VNet name`**
		- **Allow `VNet name` to receive forwarded traffic from `peer VNet name`**
6. Select **Add**.
7. Repeat steps to create peering with Spoke2

## Step 4: Create the Routes in Azure

1. Go to the ODL-<*>-<*>-04 Resource Group.
2. Select **Create**.
3. In the search bar, type the route **table,** and press **Enter**.
4. Select the **Route table**.
5. Select **Create**.
6. Use the following to create the route table:
	Region: **Select the same region as the Spoke1 resources**
	Name: **Spoke1-Route-Table**

7. Select **Review + create**.
8. Select **Create**.
9. Select **Go to resource** when deployment is completed.
10. Select **Subnets** on the left.
11. Select **Associate**.
12. Configure the Subnet:
	Virtual Network: **Spoke1**
	Subnet: **Spoke-1-subnet**
13. Select **Ok**.
14. Select **Routes** on the left.
15. Add the following route:
	Route name: **East-West**
	Address prefix: **10.10.0.0/24**
	Next hop type: **Virtual Appliance**
	Next hop address: **10.6.1.4** (IP of the internal load balancer for the cluster)

16. Select **OK**.
17. Select **Add** to add a default Route.
18. Configure as follows:
	Route name: **Default**
	Address prefix: **0.0.0.0/0**
	Next hop type: **Virtual Appliance**
	Next hop address: **10.6.1.4**

19. Select **OK**.
20. Repeat the steps to add the routes for Spoke 2 vNet.

	Route name: **East-West**
	Address prefix: **10.11.0.0/24**
	Next hop type: **Virtual Appliance**
	Next hop address: **10.6.1.4** (IP of the Internal Load Balancer for the Cluster)
	Route name: **Default**
	Address prefix: **0.0.0.0/0**
	Next hop type: **Virtual Appliance**
	Next hop address: **10.6.1.4**

21. Select **OK**.

## Step 5: Create the Cluster Policy for East/West Traffic

1.    Log into the Smart Center Server using the R81.X/R82 SmartConsole.
2.    Create Network objects for Spoke1 and Spoke2 subnets.
3.    On the NAT section, select **hide behind gateway**.
4.    Go to the NAT section on both Spoke1 and Spoke2.
5.    Configure as follows:

	**Enable Add automatic address translation rules**
	Translation method: **Hide**
	Install on gateway: **Azure-HA**

6.    Go to the **Security Policy** section.
7.    Make sure you are on the **Standard Policy**.
8.    Create a rule at the top:

	Source: **Spoke1 and Spoke2 networks**
	Action: **Accept**
	Track: **Log**

9.    Create another rule at the top:

	Source: **Spoke1 and Spoke2 networks**
	Destination: **Spoke1 and Spoke2 networks**
	Action: **Accept**
	Track: **Log**

10. Go to the **NAT policy**.
11. Create a rule at the top to create a no NAT rule for the Spoke networks going to the other Spoke networks.
12. **Publish** the session.
13. **Install** **Standard** **Policy** only to HA Cluster.

## Step 6: Testing East/West Traffic

1. Go to the Spoke-1-Workload Virtual Machine.
2. Select **Serial Console** under the **Help** section of the Virtual Machine.
3. Select **Start** under the Serial Console section if the VM is not started and **Refresh**
4. Login to the session:
	Username: **fwadmin**
	Password: **Checkpoint123!**
5. Ping Spoke-2-Workload server – 10.11.0.4
6. Check the logs in SmartConsole.

## End of Lab