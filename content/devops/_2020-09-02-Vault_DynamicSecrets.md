---
title: Hashicorp Vault - Dynamic Secrets
date: 2020-09-02T16:19:50+01:00
draft: true
categories:
  - Cloud Native
  - All
tags:
  - Vault
---

### Enabling AWS secrets engine
![UI](/images/2020-09-02-1.png)

![UI](/images/2020-09-02-2.png)

![UI](/images/2020-09-02-3.png)

### Define role in Vault and assign AWS policy

You will see (in below screenshot) we need to specify a policy ARN or a policy document. Let's head over to AWS console

![UI](/images/2020-09-02-4.png)

In AWS, go to IAM and then go to Policies:

![UI](/images/2020-09-02-5.png)

Copy the ARN (or the JSON)

![UI](/images/2020-09-02-6.png)

Go back to Vault to complete the configuration of the role:

![UI](/images/2020-09-02-7.png)


![UI](/images/2020-09-02-8.png)

### Configure Vault with AWS user

![UI](/images/2020-09-02-9.png)

![UI](/images/2020-09-02-10.png)


![UI](/images/2020-09-02-11.png)

![UI](/images/2020-09-02-12.png)

![UI](/images/2020-09-02-13.png)

![UI](/images/2020-09-02-14.png)


![UI](/images/2020-09-02-15.png)
### Generate a dynamic secret (AWS credential)

![UI](/images/2020-09-02-16.png)

![UI](/images/2020-09-02-17.png)

![UI](/images/2020-09-02-18.png)


### Renew lease

![UI](/images/2020-09-02-19.png)

![UI](/images/2020-09-02-20.png)

![UI](/images/2020-09-02-21.png)


➜  blog-hugo-netlify git:(master) ✗ vault lease renew -increment=200 aws/creds/EC2-Role-FullAccess/Ci3RpdwBR7TrP8f979I5GI8c           
WARNING! The following warnings were returned from Vault:

  * TTL of "3m20s" exceeded the effective max_ttl of "1m9s"; TTL value is
  capped accordingly

Key                Value
---                -----
lease_id           aws/creds/EC2-Role-FullAccess/Ci3RpdwBR7TrP8f979I5GI8c
lease_duration     1m9s
lease_renewable    true

### Revoke lease
![UI](/images/2020-09-02-22.png)

![UI](/images/2020-09-02-23.png)

![UI](/images/2020-09-02-24.png)
