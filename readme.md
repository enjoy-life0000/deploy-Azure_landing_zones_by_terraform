Deploy Azure landing zones using Terraform

Prerequisites
Before you start deploying Azure landing zones using Terraform, you need to install Azure CLI, Terraform, to set Terraform up and authenticate it.

Install Azure CLI on Windows
To install AzureCLI, running the following command in Powershell. The --exact option is to ensure the official Azure CLI package is installed. This command installs the latest version by default. To specify a version, add a --version <version> with your desired version to the command.
winget install --exact --id Microsoft.AzureCLI
To log in to Azure using the Azure CLI, you can use the az login command, 
az login
* Login Process:
Once you enter the `az login` command, it will provide you with a URL and a code.
Open a web browser and go to the provided URL, typically `https://microsoft.com/devicelogin`.
Enter the code provided by the CLI.
This will prompt you to log in with your Azure account credentials.
* Successful Login:
After successfully logging in through the browser, you can return to the command prompt or terminal. It should display the subscriptions associated with your Azure account.
* Setting a Specific Subscription (Optional):
If you have multiple subscriptions and want to set a specific one as the default:
* Check Current Account:
You can check the details of the currently logged-in account with:
az account show


Install Terraform
To install Terraform, find the appropriate download for your operating system on their download page and extract the executable from the ZIP. From there, open your favorite terminal to the directory where it is downloaded. (https://developer.hashicorp.com/terraform/install)
Update your system's global PATH environment variable to include the directory that contains the executable.
Open a terminal window.
terraform –version
Authenticating Terraform
Create a service principal and use that to authenticate to Azure using PowerShell.
Connect-AzAccount
$sp = New-AzADServicePrincipal -DisplayName tf-demo -Role "Contributor"
Then, export several environment variables in the syntax of your shell for authentication. The following is the syntax for PowerShell.
$env:ARM_CLIENT_ID       = $sp.AppId
$env:ARM_SUBSCRIPTION_ID = (Get-AzContext).Subscription.Id
$env:ARM_TENANT_ID       = (Get-AzContext).Tenant.Id
$env:ARM_CLIENT_SECRET   = $sp.PasswordCredentials.SecretText

With those values stored in environment variables, the AzureRM provider will find and use them to authenticate to Azure.
