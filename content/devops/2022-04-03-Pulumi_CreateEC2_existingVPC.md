---
title: Pulumi - Create AWS EC2 instance (existing VPC)
date: 2022-04-03T14:39:50+01:00
draft: False
categories:
  - DevOps
  - Infrastructure As Code
  - Public Cloud
  - All
tags:
  - AWS
  - Pulumi
---

### Introduction

In [this](https://blog.wimwauters.com/devops/2022-03-28-pulumi_createec2_defaultvpc/) post, we used Pulumi to create an EC2 instance in the default VPC. What follows is going to be a small variation on that post since we will create an EC2 instance but we will place it in an existing VPC that was created before already.

### Begin situation

We have created manually already a VPC and some subnets. Apologies guys, I'll carve out some time later to also have this automated. Here we have:

- VPC: pulumi-vpc with CIDR 10.0.0.0/16
- Subnet 1: pulumi-subnet-1 with CIDR 10.0.24.0/20
- Subnet 2: pulumi-subnet-2 with CIDR 10.0.32.0/20

Below is a screenshot of the VPC:

![pulumi](/images/2022-04-03-1.png)

And here is a screenshot of the subnets:

![pulumi](/images/2022-04-03-2.png)

### Pulumi code

In terms of code, it's not so much different from what we did in [this](https://blog.wimwauters.com/devops/2022-03-28-pulumi_createec2_defaultvpc/) post. However, here we are using the `aws.ec2.Vpc.get` function to retrieve the VPC based on its name and id. The subnet we retrieve by using the `aws.ec2.get_subnet` function.

```python
import pulumi
import pulumi_aws as aws

size = 't3.micro'

user_data = """#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 8080 &
"""

ami = aws.ec2.get_ami(most_recent="true",
                  owners=["137112412989"],
                  filters=[{"name":"name","values":["amzn-ami-hvm-*"]}])

my_vpc = aws.ec2.Vpc.get(resource_name="pulumi-vpc", id="vpc-0fe55d86283f0705c")

my_subnet =  aws.ec2.get_subnet(filters=[aws.ec2.GetSubnetFilterArgs(
    name="tag:Name",
    values=["pulumi-subnet-1"],
)])

group = aws.ec2.SecurityGroup('pulumi_allow_8080',
            vpc_id=my_vpc.id,
            description='Enable access to port 8080',
            ingress=[
                { 'protocol': 'tcp', 'from_port': 8080, 'to_port': 8080, 'cidr_blocks': ['0.0.0.0/0'] },
                { 'protocol': 'tcp', 'from_port': 22, 'to_port': 22, 'cidr_blocks': ['0.0.0.0/0'] }
            ])

server = aws.ec2.Instance('webserver',
    instance_type=size,
    vpc_security_group_ids=[group.id],
    ami=ami.id,
    subnet_id=my_subnet.id,
    user_data = user_data,
    tags = { "Name": "Pulumi" },
    )

pulumi.export('publicIp', server.public_ip)
pulumi.export('publicHostName', server.public_dns)
```

In order to run the script, we can simply use the `pulumi up` command:

```bash
/Webserver/Pulumi_existingVPC❯ pulumi up
Please choose a stack, or create a new one: dev
Previewing update (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_wim/dev/previews/72afd6c3-ba80-44dd-bdc0-05d895d45a28

     Type                      Name               Plan
 +   pulumi:pulumi:Stack       aws_ec2_wim-dev    create
 +   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080  create
 +   └─ aws:ec2:Instance       webserver          create

Resources:
    + 3 to create

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_wim/dev/updates/20

     Type                      Name               Status
 +   pulumi:pulumi:Stack       aws_ec2_wim-dev    created
 +   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080  created
 +   └─ aws:ec2:Instance       webserver          created

Outputs:
    publicHostName: "ec2-3-120-158-193.eu-central-1.compute.amazonaws.com"
    publicIp      : "3.120.158.193"

Resources:
    + 3 created

Duration: 23s
```

The final result can be seen below. We have created indeed an EC2 instance but rather than placing this into the default VPC, we have placed it in a VPC which was existing already before.

![pulumi](/images/2022-04-03-3.png)

Code for this small variant can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/InfraAsCode/Webserver/Pulumi_AWS_existingVPC).
