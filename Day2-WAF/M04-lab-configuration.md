# M04 Lab WAF Configuration

## Onboarding Web Assets to the Infinity Portal

### Task 1: Create Web Application and API Assets

In the lab prerequisites, we deployed three Docker containers, including Juice Shop, DVWA, and Vampi API. Now, we will add these assets to the Infinity Portal, configuring them in Prevent and Detect modes.

1. Go to the Infinity Portal and, under the CloudGuard section, select "WAF - Web Application & API".

2. Under Policy, click on Assets, then select "New Asset" → "Web Application". 

3. Configure 6 new web application assets as per below table

| Asset Name         | Public URL                     | Private IP URL                     | Web Application Best Practice            | API Protection Practice | 
|--------------------|--------------------------------|------------------------------------|---------------------------|------------------------------------|
| juiceshop-unsecured| http://juiceshop-unsecured.app | http://172.16.0.x:3001            | Learn / Detect            |Disabled| 
| juiceshop-secured  | http://juiceshop-secured.app   | http://172.16.0.x:3001            | Prevent                   |Disabled| 
| dvwa-unsecured     | http://dvwa-unsecured.app      | http://172.16.0.x:3000            | Learn / Detect            |Disabled| 
| dvwa-secured       | http://dvwa-secured.app        | http://172.16.0.x:3000            | Prevent                   |Disabled| 
| vampi-unsecured    | http://vampi-unsecured.app     | http://172.16.0.x:3002            | Disabled            |Learn / Detect| 
| vampi-secured      | http://vampi-secured.app       | http://172.16.0.x:3002            | Disabled                   |Prevent| 

The following configuration parameters are the same for all assets. Leave everything else default:

| Configuration | Value / Action                                |
|--------------------|-----------------------------------------------|
| Profile            | azure-waf-gateways                  |
| Learning Engine identifies HTTP requests according to          | Source IP                  |                                   |

4. Go to C:\Windows\System32\drivers\etc on your PC.

5. Open the hosts file with Notepad editor. You may need to run Notepad as administrator to have the necessary permissions.

6. Modify the /etc/hosts file on the PC used for testing and add the URL addresses using the public IP of the CloudGuard WAF instance on AWS. Do not use the example IP below but find the public IP of CloudGuard WAF in the Azure Console. After adding the entries, save the file and exit the editor.

```
34.236.235.173 juiceshop-secured.app
34.236.235.173 juiceshop-unsecured.app
34.236.235.173 dvwa-secured.app
34.236.235.173 dvwa-unsecured.app
34.236.235.173 vampi-demo.app
```

7. After adding the URLs to the hosts file, try to access the following websites:

- http://juiceshop-unsecured.app
- http://juiceshop-secured.app
- http://dvwa-unsecured.app
- http://dvwa-secured.app

These should display the lab web pages as expected.
