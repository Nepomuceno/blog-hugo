---
title: Personal email sending Email
date: 2018-07-26 01:01:01
---

This is a continuation for the post [Personal email with Azure DNS and Mailgun]({{< ref "/posts/personal-email-with-azure-dns-and-mailgun.md" >}}) but now helping you to also send email using the same configuration.

```bash
# TD;DR
# If you want one line to configure but your emails will probably end up in the span folder:

curl -s --user 'api:YOUR_API_KEY' \
    https://api.mailgun.net/v3/domains/YOUR_DOMAIN_NAME/credentials \
    -F login='YOUR_USERNAME@YOUR_DOMAIN_NAME' \
    -F password='SUPER_SECRET_PASSWORD'
```

<!--more-->

## This port is part of a series

- [Part 1] - [Personal email with Azure DNS and Mailgun]({{< ref "/posts/personal-email-with-azure-dns-and-mailgun.md" >}})
- [Part 2] - [Personal email sending Email]({{< ref "/posts/personal-email-sending-email.md" >}})

## Pre Requisites

You would need to have configured the email server according to what was instructed in the previous post ([Personal email with Azure DNS and Mailgun]({{< ref "/posts/personal-email-with-azure-dns-and-mailgun.md" >}}))

## Configuring SMTP

We are going to connect with your email client it is to setup smtp server to your email client using smtp.

So in order to do that we need to create some smtp credentials in mailgun and change our dns records.

I will be using the [azure cloud shell](https://shell.azure.com) as a way to configure that just because it is a consistent way to have one available shell independent on which OS you are runing or what you do have installed, but you should be able to follow the instructions from any bash with [azure cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) installed.

### Adding a credential to your domain

Using your mailgun api key and you previously created domain creating an SMTP credential will require just one command.

```bash
curl -s --user 'api:YOUR_API_KEY' \
    https://api.mailgun.net/v3/domains/YOUR_DOMAIN_NAME/credentials \
    -F login='YOUR_USERNAME@YOUR_DOMAIN_NAME' \
    -F password='SUPER_SECRET_PASSWORD'
```

Remember that if you want to change the password at any time all you would need to do it is to call.

```bash
curl -s --user 'api:YOUR_API_KEY' -X PUT \
    https://api.mailgun.net/v3/YOUR_DOMAIN_NAME/credentials/YOUR_USERNAME \
    -F password='EXTRA_DOPE_SUPER_SECRET_PASSWORD'
```

### Configuring your email client

Now that you created your credentials it is time to configure your email client in order to send emails with this address.

The basic configuration will be:

- **Server:** smtp.mailgun.org
- **Port:** 587
- **Username:** YOUR_USERNAME@YOUR_DOMAIN_NAME
- **Password:** EXTRA_DOPE_SUPER_SECRET_PASSWORD

This whould be enough to get you sending emails. but most of them will probably arrive in the SPAN but if you want to avaid that follow the next step.

## Avoiding the spam folder

Now you are sending your email through mailgun so you need to tell the email providers in the world that you actually authorized mailgun to do that. and in order to do that you need to change some of your email settings.

first lets get the `sending_dns_records` information part from your domain. To do that just execute:

```bash
curl -s --user 'api:YOUR_MAILGUN_API_KEY' -G \
    https://api.mailgun.net/v3/domains/YOUR_DOMAIN_NAME
```

From the response extract save the `sending_dns_records` part. It should look something like this:

```json
"sending_dns_records": [
    {
      "record_type": "TXT",
      "valid": "valid",
      "name": "YOUR_DOMAIN_NAME",
      "value": "v=spf1 include:mailgun.org ~all"
    },
    {
      "record_type": "TXT",
      "valid": "valid",
      "name": "YOUR_DOMAIN_NAME",
      "value": "k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUA...."
    },
    {
      "record_type": "CNAME",
      "valid": "valid",
      "name": "email.YOUR_DOMAIN_NAME",
      "value": "mailgun.org"
    }
  ]
```

Now we need to make some changes to our Azure DNS

First lets add the spf, for that use the value of the first record from your sending dns records ( it should have spf in it ):

```bash
az network dns record-set txt add-record -z "mydomainname.com" -g "myresourcegroup" --value "v=spf1 include:mailgun.org ~all" -n "@"
```

Now lets add the domain key the second one in your list it should start with something like `k=rsa`

```bash
az network dns record-set txt add-record -z "mydomainname.com" -g "myresourcegroup" --value "k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUA...." -n "smtp._domainkey"
```

By this time this the email should be working already to add the convinience of being able to configure `email.YOUR_DOMAIN_NAME` as your smtp server we can add the last entry in our azure dns

```bash
az network dns record-set cname set-record -z "mydomainname.com" -g "myresourcegroup" -n email -c "mailgun.org"
```

## Conclusions

Now you are able to send and receive emails from any custom domain.
The last step now would be to automate these steps to create a single script that you could do run all of this without being worried about the configuration.

The only limitation now would be the 10,000 free emails per month as part of the free account of mailgun (this counts sending and receiving) so if you go over that you would pay some extra.
