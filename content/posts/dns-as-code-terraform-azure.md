---
title: DNS as code with azure DNS and terraform
date: 2018-02-28 01:01:01
---

If your current request for a subdomain requires you to contact the infra team open a ticket and wait for few days until you can test this new cool website on your custom subdomain this might interest you. Even if you are not currently an Azure customer or not currently using terraform this is very easy to setup feature that will help you streamline your domain management.

The code for this example can be found at: [https://github.com/Nepomuceno/sample-terraform-dns](https://github.com/Nepomuceno/sample-terraform-dns)

<!--more-->

## Setting up terraform

If you have not installed terraform you should first install it. fortunately for you, there is excellent documentation on how to do it on [microsoft docs](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/terraform-install-configure)

but if you don't want to install terraform you can use the cloud shell to play with it. You can go to [shell.azure.com](https://shell.azure.com) or open it from inside the portal. Terraform it is already installed by default for you on the azure could shell.

You can check if terraform it is installed on your system by typing:

```sh
terraform --version
```

You should see an output with the current version of Terraform that you will be using at the time that I am writing this the version of the cloud shell it is `v0.11.3`.

## Defining Azure resources

Now that you have Terraform installed time to create your first `tf` file.

I will be using [VSCode](https://code.visualstudio.com/) to manage my project I recommend you to use it to since it has a good [terraform extension](https://marketplace.visualstudio.com/items?itemName=mauve.terraform)

Open the folder of your infrastructure project in VSCode.
Create a file called `mydomain_com_dns.tf` (everytime that I put here `mydomain.com` you should actually put your domain name), there is no requirement on the name of your tf files but I just think this makes things easier.

The first thing you need to define that your provider will be Azure you can do this by putting on the beginning of the file:

```yaml
provider "azurerm" {
}
```

After that, we are going to create a resource group. if you are not familiar with Azure, especially the new azure resource manager, imagine a resource group as bring a folder where you put your resources on.

```yaml
resource "azurerm_resource_group" "dns_management" {
name     = "dns-managment"
location = "West US"
}
```

Now that you have where to put your resources time to create your DNS zone, this is basically creating a DNS manager for your domain.

```yaml
resource "azurerm_dns_zone" "mydomaincom" {
name                = "mdomain.com"
resource_group_name = "${azurerm_resource_group.dns_management.name}"
}
```

This will create the zone for you. You can see that I used the terraform variable in the resource group name `${azurerm_resource_group.dns_management.name}` this it keeps consistency and it is considered the best practice when using terraform.

Now let's create an A (called here `azurerm_dns_a_recod`) domain record:

```yaml
resource "azurerm_dns_a_record" "projectmydomain" {
name                = "project"
zone_name           = "${azurerm_dns_zone.mydomaincom.name}"
resource_group_name = "${azurerm_resource_group.dns_management.name}"
ttl                 = 300
records             = ["127.0.0.1"]
}
```

You can see again the use of variables and you will see that this is really common on Terraform.

## Appling changes

If it is the first time using Terraform you will need to setup a user for it you can do it easily with [azure cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

```bash
az ad sp create-for-rbac --name terraform-automation --password "YOUR_SECRET_PASSWORD"
```

You will see a response to everything that you will need to configure terraform

```json
{
  "appId": "YOUR_APPLICATION_GUID",
  "displayName": "terraform-automation",
  "name": "http://terraform-automation",
  "password": "YOUR_SECRET_PASSWORD",
  "tenant": "YOUR_TENANT_GUID"
}
```

You should use environment variables in order to modify where you are deploying with Terraform (you can also use this on the provider definition but it is not recommended)

You can do it easily with PowerShell

```powershell
$env:ARM_SUBSCRIPTION_ID = "YOUR_SUBSCRIPTION_GUID"
$env:ARM_CLIENT_ID= "YOUR_APPLICATION_GUID"
$env:ARM_CLIENT_SECRET= "YOUR_SECRET_PASSWORD"
$env:ARM_TENANT_ID= "YOUR_TENANT_GUID"
```

Now that you have the environment created you would initialize terraform to make use you have all the modules needed

```bash
terraform init
```

Now you need to execute the terraform Plan

```bash
terraform plan
```

you will see an output with the documents that will be created

```bash
An execution plan has been generated and is shown below.
Resource actions are indicated by the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_dns_a_record.projectmydomain
      id:                        <computed>
      name:                      "project"
      records.#:                 "1"
      records.3619153832:        "127.0.0.1"
      resource_group_name:       "dns-managment"
      tags.%:                    <computed>
      ttl:                       "300"
      zone_name:                 "mydomain.com"

  + azurerm_dns_zone.mydomaincom
      id:                        <computed>
      max_number_of_record_sets: <computed>
      name:                      "mydomain.com"
      name_servers.#:            <computed>
      number_of_record_sets:     <computed>
      resource_group_name:       "dns-managment"
      tags.%:                    <computed>

  + azurerm_resource_group.dns_management
      id:                        <computed>
      location:                  "westeurope"
      name:                      "dns-managment"
      tags.%:                    <computed>


Plan: 3 to add, 0 to change, 0 to destroy.
```

You can verify if this is what you want to create if this is correct you can now apply the plan to

```bash
terraform apply
```

Terraform will show you the plan and ask for confirmation. All your resources should now be created and you will get a response back for example:

```bash
azurerm_resource_group.dns_management: Creating...
  location: "" => "westeurope"
  name:     "" => "dns-managment"
  tags.%:   "" => "<computed>"
azurerm_resource_group.dns_management: Creation complete after 1s (ID: /subscriptions/YOUR_SUBSCRIPTION/resourceGroups/dns-managment)
azurerm_dns_zone.mydomaincom: Creating...
  max_number_of_record_sets: "" => "<computed>"
  name:                      "" => "mydomain.com"
  name_servers.#:            "" => "<computed>"
  number_of_record_sets:     "" => "<computed>"
  resource_group_name:       "" => "dns-managment"
  tags.%:                    "" => "<computed>"
azurerm_dns_zone.mydomaincom: Creation complete after 2s (ID: /subscriptions/YOUR_SUBSCRIPTION/dnszones/mydomain.com)
azurerm_dns_a_record.projectmydomain: Creating...
  name:                "" => "project"
  records.#:           "" => "1"
  records.3619153832:  "" => "127.0.0.1"
  resource_group_name: "" => "dns-managment"
  tags.%:              "" => "<computed>"
  ttl:                 "" => "300"
  zone_name:           "" => "mydomain.com"
azurerm_dns_a_record.projectmydomain: Creation complete after 1s (ID: /subscriptions/YOUR_SUBSCRIPTION/dnszones/mydomain.com/A/project)
```

as you can see the whole process of actually applying the configuration took less than 5 seconds.

## Adding a new resource

Now imagine that a developer wants that `awesome.mydomain.com` points to `project.mydomain.com`

All he needs to do it is to change the `mydomain_com_dns.tf` file to include this CNAME

```yaml
resource "azurerm_dns_cname_record" "awesomemydomain" {
name                = "awesome"
zone_name           = "${azurerm_dns_zone.mydomaincom.name}"
resource_group_name = "${azurerm_resource_group.dns_management.name}"
ttl                 = 300
record              = "project.mydomain.com"
}
```

Now if you run `terraform apply` again the cname record will be created.

From now on managing your DNS it is just a PR away.
