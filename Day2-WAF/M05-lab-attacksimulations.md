# M05 - Lab Attack Simulations

## Introduction

In this module, you will learn how to simulate and analyze common web application and API attacks using CloudGuard WAF. You will perform hands-on exercises to test attack scenarios on vulnerable applications, observe how the WAF detects and prevents threats, and review detailed security logs. Additionally, you will explore API discovery and schema validation features to enhance your understanding of modern web and API protection practices. By the end of this module, you will gain practical experience in identifying vulnerabilities, configuring security policies, and leveraging CloudGuard WAF to secure your web assets.

In this module, you will:

- Task 1: Enable logging for all web requests, including legitimate traffic  
- Task 2: Simulate and analyze attack scenarios on Juice Shop
- Task 3: Simulate and analyze attack scenarios on DVWA application  
- Task 4: Configure API Discovery and review discovered APIs on CloudGuard WAF

**Estimated time**: 25 minutes

Important: This exercise requires the previous modules to be finished first.

### Task 1: Enable logging all events, including legitimate

1. Go to **Policy > Triggers > Log Trigger** and enable "All web requests" to log all web requests.

![alt text](/assets/images/infinity-portal-logtrigger.png)

2. Enforce the policy.

### Task 2: Simulate and analyze attack scenarios on Juice Shop

1. Go to your browser and open the following websites in different tabs:

- http://juiceshop-unsecured.app
- http://juiceshop-secured.app

2. Click on Account and then Login. Enter the following payload in the username and  password field:

```
' OR 1=1; --
```

**Expected outcome:** When the WAF is in detect mode (unsecured), you will see that the login bypasses without valid credentials. The SQL injection payload will allow unauthorized admin access, demonstrating a vulnerability in the unsecured application.

![alt text](/assets/images/juiceshop-attack1unsecured.png)

> **Note:**
>
> This attack is a classic SQL injection targeting the login functionality. The payload ' OR 1=1; -- is designed to manipulate the SQL query that checks user credentials. By injecting this string, the attacker causes the condition in the SQL statement to always evaluate as true (1=1), effectively bypassing authentication and granting access without valid credentials. The -- sequence comments out the rest of the SQL query, preventing any syntax errors.
>
> This demonstrates a critical vulnerability: the application does not properly sanitize or validate user input before including it in a SQL query. As a result, attackers can gain unauthorized access, often with elevated privileges such as admin rights. When the WAF is in detect mode (unsecured), this attack succeeds; when in prevention mode (secured), the WAF blocks the malicious input, protecting the application from unauthorized access.

3. Do the same attack on the secured URL (http://juiceshop-secured.app). The login will fail, and an error page will appear, showing that the WAF is blocking the attack.

![alt text](/assets/images/juiceshop-attack1secured.png)

Now let’s take a look at the logs in Cloudguard WAF Portal.

4. To see the logs, go to Monitor -> Important Events.

5. Review the latest logs related to your last SQL test on Juice Shop.

6. Notice that the same attack in unsecured mode is detected, while in secured mode, it is prevented.

![alt text](/assets/images/infinity-portal-events.png)

7. Click on the last log entry and review the log details.

![alt text](/assets/images/infinity-portal-events-detail.png)

8. Review the logs for the other test in detect mode (unsecured Juice Shop).

![alt text](/assets/images/infinity-portal-events-detail2.png)

Now let’s continue with another attack.

9. Copy the following URLs and open them in different browser tabs:

Without Prevention Mode (unsecured)
```
http://juiceshop-unsecured.app/rest/products/search?q=null%27))%20UNION%20SELECT%20password,1,id,%20email,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20users--
```

With Prevention Mode enabled (secured)
```
http://juiceshop-secured.app/rest/products/search?q=null%27))%20UNION%20SELECT%20password,1,id,%20email,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20users--
```

**Expected outcome:** As you can see, using the unsecured URL exposes sensitive information that should not be accessible. This is a successful attack, as the application is not properly securing sensitive data in unsecured mode.

![alt text](/assets/images/juiceshop-attack2unsecured.png)

> **Note:**
>
> This attack is a SQL injection using a UNION SELECT statement. The attacker manipulates the application's search query parameter to inject SQL code that combines the results of a legitimate query with sensitive data from the users table, such as passwords and emails. By crafting the URL, the attacker bypasses normal application logic and retrieves confidential information directly from the database.
>
> In unsecured mode, the application does not properly validate or sanitize user input, allowing the injected SQL to execute and expose sensitive data. With WAF Prevention Mode enabled, the WAF detects and blocks this malicious query, preventing unauthorized access to user credentials and other private information. This demonstrates how SQL injection can compromise data security and how a properly configured WAF can mitigate such risks.

10. Review the output in the other browser tab where the secured URL is used (with WAF in prevention mode). You should notice that the attack is blocked, and sensitive information is not accessible. 

![alt text](/assets/images/juiceshop-attack2secured.png)

Let´s review the logs.

11. Go to **CloudGuard WAF > Monitor > Important Events**.

12. Review the latest logs related to your attack attempts on the Juice Shop web app. You should see entries indicating whether the attack was detected or prevented, depending on the WAF mode. 

![alt text](/assets/images/infinity-portal-events2.png)

13. Click on the event log and review details of log.

### Task 3: Simulate and analyze attack scenarios on DVWA application  

1. Go to your browser and open the following website:

- http://dvwa-unsecured.app

2. Use the following credentials to access DVWA:
- Username: admin
- Password: password

3. You will be redirected Database Setup page. Click on **Create / Reset Database**. Then login again with above given credentials.

4. Once you login, on the homepage, click on DVWA Security and change the default DVWA security setting to **Low** in DVWA. 

![alt text](/assets/images/dwva-securitylevel.png)

5. Go to the SQL Injection part of DVWA. Enter the following text in the User ID field and click Submit:

```
%' OR '42'='42
```

![alt text](/assets/images/dwva-attack1unsecured.png)

Result: It will show you every user of the application, as the SQL injection bypasses the authentication and retrieves all user data from the database.

Now let’s do the same attacks on dvwa-secured web site.

6. Copy the URL which you test last attack on unsecured web app. 

![alt text](/assets/images/dwva-attack1unsecuredurl.png)

7. In the URL change the “-unsecured” with “secured”. The link should be as below follows:
http://dvwa-secured.app/vulnerabilities/sqli/?id=%25%27+OR+%2742%27%3D%2742&Submit=Submit#

8. Copy the this URL on a new browser tab and try to access to this URL.

![alt text](/assets/images/dwva-attack1securedurl.png)

Result: You will notice that the access will not be successful, as the WAF will block the SQL injection attempt, preventing unauthorized access.

9. Go to **CloudGuard WAF > Monitor > Important Logs** and review the logs for both the secured and unsecured web apps. You should see entries that show the SQL injection attempt being detected in the unsecured app and blocked in the secured app.

![alt text](/assets/images/infinity-portal-events3.png)

### Task 4: Configure API Discovery and review discovered APIs on CloudGuard WAF

In this task, we will configure API Discovery and Schema Validation for the Web API.

1. Go to CloudGuard -> Policy -> Assets.

2. Find the vampi-api-demo API and click on it to configure it.

3. Under API Protection Tab, go to API Discovery Practice and make sure that API Discovery is set to **Active**. If not, change it to **Active**.

![alt text](/assets/images/infinity-portal-apiprotection.png)

4. Go to http://vampi-demo.app/ui/ to access the user interface of the VAmPI API. There, you will see various APIs. We will execute some API requests so they can be discovered by CloudGuard WAF. 

Before you start trying various APIs, you need to create the database.

5. Under db-init, go to GET /create to create and populate the database with dummy data.
Click on **Try it out**, then click on **Execute**

![alt text](/assets/images/vampi-dbinit.png)

![alt text](/assets/images/vampi-dbinit2.png)

Then we can try other available APIs. Let’s start with one of them.

6. Under **Books > GET /books/v1**, click on **Try it out**, then click on **Execute**.

![alt text](/assets/images/vampi-books.png)

7. Repeat the same steps for **Users (GET)** and other APIs as well.

8. Review the output of the APIs in the Swagger User Interface.

![alt text](/assets/images/vampi-booksresponse.png)

9. Go back to CloudGuard WAF and select the **vampi_secured API** asset.

10. Go to the **LEARN** tab, and under **API DISCOVERY**, you will see the automatically discovered APIs.

![alt text](/assets/images/infinity-portal-apidiscovery.png)

> **Note:** It may take some time for the API discovery engine to display the requests. You might want to check the logs first.