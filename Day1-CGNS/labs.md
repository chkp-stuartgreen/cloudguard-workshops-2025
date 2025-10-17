### Azure - Management Lab

All of Check Points gateway architectures require a management deployment to control them. This can be a physical Smart-1 appliance, Smart-1 Cloud (SaaS management) or a management VM in public or private cloud.

For this lab, it will be easier to deploy a management VM inside a dedicated vNET. 
You will need your sandbox credentials to log in to Azure and create this.

1. Deployment of a Check Point Management Server R81.20/R82 1. Log in to the Azure Portal at https://portal.azure.com using the credentials provided. 
2. Select Create a resource. 
3. In the search box enter Check Point and select the CloudGuard Network Security – Firewall & Threat
4. Then select from the dropdown list the Check Point Security Management 
5. Select Create 
6. Enter the following details for creating the image

| **Selection**  | **Choice**                                   | **Notes**                                                                                            |
| -------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Subscription   | Choose a Subscription                        | Select the subscription to deploy under. In the lab, there is only one option                        |
| Resource Group | **Use Existing:**<br><br>**ODL-<*>_-<_>-01** | Used to identify resources of similar life-cycle                                                     |
| Region         | **Canada/US**                                | Choose a local region.<br><br>(West US 2, **East US**, Canada Central, Central US, South Central US) |
| Server Name    | CPSmartCenter                                | Name of your NVA                                                                                     |
| Password       | Checkpoint123!                               | Unique password for Check Point.                                                                     |


7.    Select **Next: Check Point Security Management Server Settings >**
8.    Enter the following details

| **Selection**                                     | **Choice**                 | **Notes**                                         |
| ------------------------------------------------- | -------------------------- | ------------------------------------------------- |
| Check Point Version                               | **R82 or R81.20**          | The version you would like to deploy              |
| License Type                                      | **Bring your Own License** |                                                   |
| Virtual Machine Size                              | **Ds3v2**                  | Choosing the default size the template will fail. |
| Allow SmartConsole connection from these networks | **0.0.0.0/0**              | Allows connection to the Smart Center.            |
|                                                   |                            |                                                   |
| Installation Type                                 | **Default** (Management)   |                                                   |
| Default Shell                                     | /etc/cli.sh or /bin/bash   |                                                   |
| Allowed GUI Clients                               | 0.0.0.0/0                  | IP for GUI client access                          |
9.    Select **Next: CloudGuard Advance Settings >**


| **Selection**                     | **Choice**                           | **Notes**             |
| --------------------------------- | ------------------------------------ | --------------------- |
| Installation Type                 | **Default** (Management)             |                       |
| Default Shell                     | /etc/cli.sh or /bin/bash             |                       |
| Enable Maintenance Mode           | **No**                               |                       |
| Maintenance Mode Password         | Leave Blank                          |                       |
| Confirm Password                  | Leave Blank                          |                       |
| Bootstrap script                  | Leave blank                          | Bootstrap script      |
| VM Disk type                      | **Default** (Premium)                | VM Disk type          |
| Accept Management API             | **Default** (Management Server only) | Accept Management API |
| Automatic Update                  | **Default** (Yes)                    | Automatic Update      |
| Create a System Assigned Identity | **Default** (Yes)                    |                       |
| Addition disk space               | **Default** (0)                      |                       |

10.    Select **Next: Network settings >**
11.    In the Virtual Network, Select **Edit** **virtual network**
12.    Configure the vNet as follows:
	- Name: MGMT-vNet
	- Modify the address space
		- Address range: 10.1.0.0/24
		NOTES: This should update the vNet and Subnet.
	- **Save**

13.    In the **Network Secure Groups** select **Create New**
14.    **Name** – Leave Default
15.    Select **Review + Create**
16.    The Purchase Agreement will now appear. Approve the Terms and Select **Create**
17.    From the Windows PC open up Smart Console to your newly deployed Check Point Management station. 

NOTES: It may take up to 15 minutes for the deployment and CPM to complete this process.

Access details:

- IP: The public IP
- Username:      admin
- Password:      Checkpoint123!

---
