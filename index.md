![image](https://user-images.githubusercontent.com/48949955/156836938-97972094-f81c-49f0-aace-8ca819b83ef2.png)

<h1>Stop and Start VM using Azure Function Automation with Azure Serverless Functions and PowerShell</h1>
<h2>TABLE OF CONTENTS<h2/><h3><ul>
<li>Create Azure Function App</li>
<li>Configure the System managed Identity for the Function App</li>
<li>What is PowerShell</li>
<li>Enable Az PowerShell module</li>
<li>Create an Azure Virtual Machine</li>
<li>Create Tags for your virtual machine</li>
<li>Conclusion</li>
</ul></h3>
I recently had the opportunity of helping a client automate "stop" and "start" operations on his Azure Virtual Machine. Seeing how happy he was, I decided to share this knowledge with everyone else.

In this article, I will provide you step by step guide on how to automate a stop and start VM operation using Azure Functions.

Make a cup of coffee, and let us have some fun time.

If you do not have an Azure account, then <a href="https://azure.microsoft.com/en-us/free/">create a free Azure account </a>
  <h3> First we create a powershell function app</h3>
Enter the following parameters. Feel free to choose any name for the resource group and function name fields. However, it is best practice to choose a name that helps in identifying resources.
  
  ![image](https://user-images.githubusercontent.com/48949955/156837432-9d184bb7-8e91-442a-a68e-3f2d92357443.png)
  
Afterwards, click on Next: Hosting
  
  ![image](https://user-images.githubusercontent.com/48949955/156837562-9e175240-9b6d-4c88-9ee9-a2be8ab1fc1a.png)
  
Under the operating systems field, select Windows and choose consumption plan (serverless) as the Hosting plan type
Afterward, proceed to the Review + create a tab by clicking Next
  
  ![image](https://user-images.githubusercontent.com/48949955/156837680-b2a2c422-eab1-427a-9e09-1634f2248fa5.png)

Then click Create
  
  ![image](https://user-images.githubusercontent.com/48949955/156838077-a3db48ce-44e9-44c2-b81a-29b2794a6dbf.png)
  
  
Click on Go to resource
<h3>Configure the System managed Identity for the Function App</h3>
  
On the sidebar menu of the Function App page, under the settings section, click on the identity
  
 ![image](https://user-images.githubusercontent.com/48949955/156838479-1dc63464-1ca4-4b19-b8a4-35e4e43b07f1.png)
  
  Managed identities provide an identity for applications to use when connecting to resources that support Azure Active Directory (Azure AD) authentication.

Managed Identity helps to eliminate the hassle that developers face in the Management of secrets and credentials used to secure communication between different components.

In this case, we want to create a managed identity access for the Function App to allow access to the virtual machine resource. Sounds good? , okay let us proceed.

Under the system assigned tab, toggle the status from OFF to ON, then save.

After you click on save, the following section will become available

  ![image](https://user-images.githubusercontent.com/48949955/156838665-d09dc0ae-5970-4790-94f6-793c352aa395.png)

  
Click Azure role assignment.

  ![image](https://user-images.githubusercontent.com/48949955/156838923-c41c2391-4270-4ece-b316-f8be7ee03322.png)

  Next, select Add role assignment (preview)
  
  ![image](https://user-images.githubusercontent.com/48949955/156839053-f2bdc7ce-b5a2-4a04-84b8-527a08a1119b.png)

  There are three steps to consider here:

1. On the pane, select Subscription as the scope
2. Under the role section, type in Virtual Machine Contributor( following the principle of least privilege).
3. Click Save
  
  ![image](https://user-images.githubusercontent.com/48949955/156839307-bc965713-6516-47d5-8df0-fcb665461452.png)

 Afterward, you should see the Assigned role displayed.
  
  ![image](https://user-images.githubusercontent.com/48949955/156839558-afbeb0c9-d193-4cd0-b806-9c64f565a468.png)

  Navigate back to the Function App page, on the sidebar click Function and + Create a Function.
  
  ![image](https://user-images.githubusercontent.com/48949955/156839647-b6636735-b6ac-4496-8240-4415ee5d9e36.png)
  
  Select Timer Trigger, scroll down and set the desired CRON expression for your timer trigger.
  
  ![image](https://user-images.githubusercontent.com/48949955/156839757-1a59f7d4-4e8f-46c3-9c0a-72899a38e294.png)
  
  Preferably, configure the Function to Trigger every hour.
I choose to use the CRON expression 0 0 * * * 1-5, which means my Function will trigger every hour, Monday to Friday of every week and every month of the year.

Azure uses NCRONTAB expression for Timer Triggers. NCRONTAB uses six inputs instead of five, with the extra input representing seconds. e.g., 0 0 * * * 1-5.

{second} {minute} {hour} {day} {month} {day-of-week}

You can check out Azure documentation on NCRONTAB

Are you worried that you may incur a high bill? I've got good news for you

  <i>Azure Functions consumption plan is billed based on per-second resource consumption and executions. Consumption plan pricing includes a monthly free grant of 1 million requests and 400,000 GB-s of resource consumption per month per subscription in pay-as-you-go pricing across all function apps in that subscription.</i>
  
  
![image](https://user-images.githubusercontent.com/48949955/156840226-5406e3d9-f21d-4b00-a7e7-d5918d9a5380.png)

  After successful creation of the Function, click on Code + Test

Paste the PowerShell code below into the editor, modify the subscription to include your valid subscription, and click save.

![image](https://user-images.githubusercontent.com/48949955/156840401-286b87b9-50f8-4e1f-9b9d-f6dd032519c3.png)
  
  <pre><code>
 # Input bindings are passed in via param block.
param($Timer)

# Get the current universal time in the default string format.

$subscriptionId = "your subscription id"
$tenantId = "your tenant id"
$rsgName = "resource group name"
$vmName = "VM name"

Select-AzSubscription -SubscriptionID  $subscriptionId -TenantID $tenantId
Start-AzureRmVM -ResourceGroupName $rsgName -Name $vmName
</code></pre>
  
  <h3>What is PowerShell?</h3>
PowerShell is a cross-platform task automation solution. It is a scripting language and a configuration management framework that can run on Windows, Linux, and macOS.

As a scripting language, PowerShell is used in automating the management of systems.

On the code above, we have a variable declaration, nested "foreach" statements, and conditional statements.

The Get-AzVM command is a PowerShell command used to retrieve the properties of a Virtual machine.

The if..elseif... condition checks for the PowerState of the VM. If the VM is running and the time is greater than the value set in the Virtual machine Tag, we want to Stop the VM and vice versa.

For a complete reference to the PowerShell tags used, see PowerShell docs

PowerShell gives us the control to system properties and settings. It is a significant language for automation and is one of my favorites.

Before we continue, you should take a sip from your coffee.

Does it still taste good? I hope so. Now, let us continue.

  <h3>Enable Az PowerShell module</h3>
Navigate back to the Function App page, on the sidebar menu, click App Files under the Functions section

Click on the dropdown menu and select requirement.psd1.

  ![image](https://user-images.githubusercontent.com/48949955/156842579-b0bef85c-a2f6-4339-87ac-22c5a0c537a5.png)
  On the requirement.psd1 file, uncomment line 7 by removing the # sign.

This will allow the Function to use the Az module. Click Save

<h3>Create an Azure Virtual Machine</h3>
Now, create your virtual machine by following the easy prompt. here

On successful creation of the machine, navigate to the Virtual machine's overview page, Select Tag(change)

  On the requirement.psd1 file, uncomment line 7 by removing the # sign.

This will allow the Function to use the Az module. Click Save

<h3>Create an Azure Virtual Machine</h3>
  
Now, create your virtual machine by following the easy prompt. here

On successful creation of the machine, navigate to the Virtual machine's Access control (IAM) to give the function app a role to enable it perform start and stop operation on the virtual machine 
  
  ![image](https://user-images.githubusercontent.com/48949955/156842945-86ffae03-d7a6-4aed-bc7c-18212b7ce21f.png)
  
  You can navigate to the Activity logs section of the Virtual machine sidebar menu
  
  
![image](https://user-images.githubusercontent.com/48949955/156843997-45465441-5b6f-4400-a1de-f155aea372cc.png)
  
  You can see the Virtual machine was shut down and started by the Function App.
  
  <h3>Conclusion</h3>
We have successfully created a start-and-stop automation for our Virtual machine. The auto-stop and auto-start options are available on every Virtual machine by default. However, by using the Azure Function App (consumption plan) and timer trigger to perform this operation, it saves you cost. Reference to pricing for consumption plan.

Thanks for taking the time to read and implement with me.


  
 


