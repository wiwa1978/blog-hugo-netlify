---
title: Hashicorp Vault - Dynamic Secrets
date: 2020-08-26T16:19:50+01:00
draft: false
categories:
  - Devops
  - All
tags:
  - Vault
  - Hashicorp
  - AWS
---
### What are dynamic secrets

In this post, we will discuss the topic of dynamic secrets. What are dynamic secrets then? A dynamic secret is a secret that is generated on demand as opposed to a static secret which is defined ahead of time. Let's imagine you need a database password. Vault could then interact with the database engine and create a dynamic secret and associate it with a lease. Vault will then also automatically destroy the credentials when the lease expires.

Dynamic secrets can be created for various systems:

- Databases (MySQL, PostgreSQL, ....)
- Cloud providers (AWS, Azure....)
- SSH
- PKI
- ...

In this post we will show the principles of dynamic secrets through AWS. The AWS secrets engine generates AWS access credentials dynamically based on IAM policies.

### Enabling AWS secrets engine

First off, we need to enable the AWS secrets engine. This can be done through the CLI, the API or the UI. For simplicity sake, we will work through the UI here. Go to Secrets > Enable new engine and select the AWS secrets engine.

![UI](/images/2020-08-26-1.png)

We will use the default path of AWS, which means our credentials will be stored under that path

![UI](/images/2020-08-26-2.png)

When the engine is enabled you will get a form to create a role.

![UI](/images/2020-08-26-3.png)

### Define role in Vault and assign AWS policy

In terms of defining a role, we will need to give it a name. You will also notice (in below screenshot) that we need to specify a policy ARN or a policy document. Let's head over to AWS console

![UI](/images/2020-08-26-4.png)

In AWS, go to IAM and then go to Policies:

![UI](/images/2020-08-26-5.png)

Copy the ARN (or the JSON)

![UI](/images/2020-08-26-6.png)

Go back to Vault to complete the configuration of the role:

![UI](/images/2020-08-26-7.png)

In this code we have opted to use the ARN but in case you copied the JSON policy, you should paste it in the `policy document` textbox.

![UI](/images/2020-08-26-8.png)

When finished, you will see we have a role available for use.

![UI](/images/2020-08-26-9.png)

And some more details:

![UI](/images/2020-08-26-10.png)

### Configure Vault with AWS user

Next, we need to add an AWS user to Vault. This user will need to have permissions. In below screenshot, I gave full admin access which is something that should not be done. I just took the easiest approach to safe some time. I'm lazy :-)

![UI](/images/2020-08-26-11.png)

![UI](/images/2020-08-26-12.png)

Take note of the security credentials. You will need them later when configuring Vault.

![UI](/images/2020-08-26-13.png)

![UI](/images/2020-08-26-14.png)

Next, go back to Vault > Secrets. Click on the aws path, then select Configuration and finally click on Configure > Dynamic IAM Root credentials. Add the access key and secret key.

![UI](/images/2020-08-26-14a.png)

If required, you can also adapt the lease here.

![UI](/images/2020-08-26-15.png)

### Generate a dynamic secret (AWS credential)

Next we will continue with generating the dynamic secret. This is fairly straightforward:

![UI](/images/2020-08-26-16.png)

As you can see, Vault will generate these credentials on AWS.

![UI](/images/2020-08-26-17.png)

If we go back to the AWS dashboard, you can also see these credentials there.

![UI](/images/2020-08-26-18.png)


### Renew lease

Vault can also renew the lease of the AWS secret. 

![UI](/images/2020-08-26-19.png)

In below screenshot, we have a remaining TTL of 16. So before it expires, we will renew the lease.


![UI](/images/2020-08-26-21.png)

### Revoke lease

Obviously we can also revoke the lease. The lease revoke command revokes the lease on a secret and will invalidate the underlying secret.

![UI](/images/2020-08-26-23.png)

As you can see Vault will not show the secret any longer and it will also be removed from AWS dynamically.

![UI](/images/2020-08-26-24.png)

Hope this was a bit helpful. See you next time.

