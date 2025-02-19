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

Writing the Terraform Configuration

Use the Azure Landing Zones (ALZ) provider to generate data to allow you to simplify provisioning of your ALZ configuration. Its principal use is to generate data to deploy resources with the AzApi provider.

terraform {
  required_providers {
    azapi = {
      source  = "Azure/azapi"
    }
  }
}

provider "azapi" {
  # More information on the authentication methods supported by
  # the AzApi Provider can be found here:
  # https://registry.terraform.io/providers/Azure/azapi/latest/docs

  # subscription_id = "..."
  # client_id       = "..."
  # client_secret   = "..."
  # tenant_id       = "..."
}

// azurerm provider
provider "azurerm" {
  features {}
}

// create a resource group, it's recommended to use azapi provider with azurerm provider
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "West Europe"
}

// create a automation account
resource "azapi_resource" "automationAccount" {
  type      = "Microsoft.Automation/automationAccounts@2021-06-22"
  name      = "myAccount"
  parent_id = azurerm_resource_group.example.id

  location = azurerm_resource_group.example.location
  body = {
    properties = {
      disableLocalAuth    = true
      publicNetworkAccess = false
      sku = {
        name = "Basic"
      }
    }
  }
}

Deploying the Terraform Configuration
Step 1
Initialize Terraform with terraform init.
You will get a lot of output indicating that Terraform is initializing the environment. It should end with the following.
Step 2
Run terraform validate to do some pre-deployment tests on the configuration. This step is not required, but it will give you confidence before running a plan.
Step 3
Run terraform plan, which will show you what it intends to create. Because this module is so complex, there are going to be a lot of resources created. The following is the last item listed.
Notice that for each resource being created, a plus sign is displayed next to it.
The last line indicates that it will create 233 items, change 0 and destroy 0. If you are running this plan again after deploying it and then editing the config, you will likely see resources being changed.
Step 4
Run terraform apply, and type in "yes" when prompted to confirm the plan to apply this configuration.
Be patient for the deployment -- it will take some time.
Once the deploy has completed, you should see the following message.

Delete the resources
If you are deploying these resources as a test and you are done evaluating the deployment, you can destroy the deployed resources with terraform destroy. This will destroy only the resources you created.





