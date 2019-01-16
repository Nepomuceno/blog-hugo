---
title: Personal email with Azure DNS and Mailgun
date: 2018-07-18 01:01:01
---

I had struggled for a while to get my custom email address working without the need to actually pay for another email provider, but more than that I would love if I could use email servers in the same way that I can use a load balancer you can change or upgrade whatever is behind it without ever needing to upgrade the interface.

Using [Mailgun](https://mailgun.com) and [Azure DNS](https://azure.microsoft.com/en-gb/services/dns/) you can do it paying less than a dollar monthly.

<!--more-->

## This port is part of a series

- [Part 1] - [Personal email with Azure DNS and Mailgun]({{< ref "/posts/personal-email-with-azure-dns-and-mailgun.md" >}})
- [Part 2] - [Personal email sending Email]({{< ref "/posts/personal-email-sending-email.md" >}})
  
## Pre Requisites

What you will need to have:

- [Mailgun Account](https://www.mailgun.com/)
- [Azure Account](https://azure.microsoft.com/)
- A domain account

You will also need to get the Mailgun API that you can get at [https://app.mailgun.com/app/account/security](https://app.mailgun.com/app/account/security)

I am assuming here that you have one domain registered already and know how to change your domain nameservers. In case you don't just search for `change domain nameservers` and look on the faq of your current domain provider.

## The azure shell

I could do everything that I will be doing today through both portals of Mailgun and Azure but things are a lot easier to explain (and require a lot less print screen) if I explain everything through the command line. Don't worry if you are not familiar with the command line I will do every thing step by step.

On the browser go to [https://shell.azure.com](https://shell.azure.com) select a bash shell and let the storage account be created in case you don't have one.

now let's get the list of our accounts to make sure we are in the correct place.

```bash
az account list -o table
```

You should see a result like:

```bash
Name          CloudName    SubscriptionId                        State    IsDefault
------------  -----------  ------------------------------------  -------  -----------
My Account    AzureCloud   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  Enabled  True
```

## Creating the DNS record

[Azure DNS](https://azure.microsoft.com/en-gb/services/dns/) is part of the Azure network resources.
Everything inside azure lives inside a resource group so let's create our RG.
For this tutorial I will be using `North Europe` but you can choose any [region from Azure](https://azure.microsoft.com/en-gb/global-infrastructure/services/)

```bash
az group create --name MyResourceGroup --location "North Europe"
```

Now let's create the DNS zone

```bash
az network dns zone create -g MyResourceGroup -n mydomainname.com
```

You are going to receive an output this:

```json
{
  "etag": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.Network/dnszones/mydomainname.com",
  "location": "global",
  "maxNumberOfRecordSets": 5000,
  "name": "mydomainname.com",
  "nameServers": [
    "ns1-08.azure-dns.com.",
    "ns2-08.azure-dns.net.",
    "ns3-08.azure-dns.org.",
    "ns4-08.azure-dns.info."
  ],
  "numberOfRecordSets": 2,
  "registrationVirtualNetworks": null,
  "resolutionVirtualNetworks": null,
  "resourceGroup": "myresourcegroup",
  "tags": {},
  "type": "Microsoft.Network/dnszones",
  "zoneType": "Public"
}
```

Now you would need to keep a record of those nameservers **you need to change your domain provider to use this name servers in the end**.

## Creating your Mailgun domain

Now it is time to create your Mailgun domain. Fortunately, you can do this through the same command line, just use your API key and the command:

```bash
curl -s --user 'api:YOUR_API_KEY' \
    -X POST \
    https://api.mailgun.net/v3/domains \
    -F name='mydomainname.com' \
    -F smtp_password='supersecretpassword'
```

```json
{
  "domain": {
    "created_at": "Wed, 18 Jul 2018 05:57:18 GMT",
    "id": "5b4ed6befa60950001074cd7",
    "is_disabled": false,
    "name": "myamazingdomain.com",
    "require_tls": false,
    "skip_verification": false,
    "smtp_login": "postmaster@myamazingdomain.com",
    "type": "custom",
    "web_prefix": "email",
    "web_scheme": "http",
    "wildcard": false
  },
  "message": "Domain has been created",
  "receiving_dns_records": [
    {
      "cached": [],
      "priority": "10",
      "record_type": "MX",
      "valid": "unknown",
      "value": "mxa.mailgun.org"
    },
    {
      "cached": [],
      "priority": "10",
      "record_type": "MX",
      "valid": "unknown",
      "value": "mxb.mailgun.org"
    }
  ],
  "sending_dns_records": [
    ...
  ]
}
```

You need to keep a record of this `receiving_dns_records` we will need to configure that in your DNS provider

## Configuring DNS to use Mailgun

Now you will need to configure the other DNS records that you have in your current domain There is an awesome tutorial [here](https://docs.microsoft.com/en-gb/azure/dns/dns-operations-recordsets-cli) on how to do it.

If you have a large set of records already I suggest using [Import/Export](https://docs.microsoft.com/en-gb/azure/dns/dns-import-export) before configuring the email records them doing:

```bash
az network dns record-set mx delete -z mydomainname.com -g myresourcegroup -n @ -y
```

Now we need to configure Azure DNS to point the `MX` records to our servers

```bash
az network dns record-set mx add-record -z mydomainname.com -g myresourcegroup -n @ -e mxa.mailgun.org -p 10
az network dns record-set mx add-record -z mydomainname.com -g myresourcegroup -n @ -e mxb.mailgun.org -p 10
```

## Configuring your routes in Mailgun

Now that you are sending your emails to Mailgun you need to forwards this emails to your current mailbox.

this can be easily done with [mailgun routes](https://documentation.mailgun.com/en/latest/quickstart-receiving.html#inbound-routes-and-parsing)

Let's create our first route, sending anything that it is sent to this domain to go to my `junk` email

```bash
curl -s --user 'api:YOUR_API_KEY' \
    https://api.mailgun.net/v3/routes \
    -F priority=1000 \
    -F expression='match_recipient(".*@mydomainname.com")' \
    -F action='forward("junk@myoldemail.com")' \
    -F action='stop()'
```

Now let's create a new rule that will send everything that arrives in `gabriel@mydomainname.com` to my old email `gabriel@myoldemail.com` since it has a low priority of than the old route it will be executed before it

```bash
curl -s --user 'api:YOUR_API_KEY' \
    https://api.mailgun.net/v3/routes \
    -F priority=100 \
    -F expression='match_recipient("gabriel@mydomainname.com")' \
    -F action='forward("gabriel@myoldemail.com")' \
    -F action='stop()'
```

You can do way more with routes and make smart email the `match_recepient` accepts regex expressions and the actions you can do things like notifying one URL every time that you receive one email (anyone else seeing a nice functions integrations here ;) ).

## Finishing up

Now, all that you need it is to change your domain provider to point to your new nameservers and you are done :)

Hope you all like and still tuned in at [my twitter](https://twitter.com/gBico) where I will publish the next steps including how to automate this process with a simple script and how to use this to not only receive but send emails too.

## Known Limitations

Of course, there are limitations when using this:

1. <s>You are only able to receive emails using this technique. I could extend it to use Mailgun to send emails to and I might do it in another blog stay tuned</s> You can now see hoe to send emails too at: {% post_link personal-email-sending-email %}

2. This is done with a lot of manual steps (coping commands from this post). This could be automated in a simple script but I needed to stop and publish at some time so I decided that I will also leave that for a future post :)

3. Mailgun it is only free up to 10,000 emails per month. but if you are getting more than 10,000 email per month I would suggest first taking a look on why are you receiving that much email and Mailgun still has a great price even above 10,000 emails.

