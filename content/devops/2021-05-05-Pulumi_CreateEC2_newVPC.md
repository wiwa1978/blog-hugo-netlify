---
title: Pulumi - Create AWS EC2 instance (new VPC)
date: 2021-05-05T14:39:50+01:00
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



### Begin situation


![pulumi](/images/2021-05-05-1.png)
![pulumi](/images/2021-05-05-2.png)


### Pulumi Code

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

my_vpc = aws.ec2.Vpc("myVpc",
    cidr_block="172.16.0.0/16",
    tags={
        "Name": "pulumi-new-vpc",
    })

my_subnet = aws.ec2.Subnet("mySubnet",
    vpc_id=my_vpc.id,
    cidr_block="172.16.10.0/24",
    availability_zone="eu-central-1a",
    map_public_ip_on_launch=True,
    tags={
        "Name": "pulumi-new-subnet",
    })

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

### Deployment

```bash
/Webserver/Pulumi_newVPC❯ pulumi up                                    
Please choose a stack, or create a new one: dev
Previewing update (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_wim/dev/previews/67108f93-f157-48b3-888c-d402fa23c6b3

     Type                      Name               Plan       
 +   pulumi:pulumi:Stack       aws_ec2_wim-dev    create     
 +   ├─ aws:ec2:Vpc            myVpc              create     
 +   ├─ aws:ec2:Subnet         mySubnet           create     
 +   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080  create     
 +   └─ aws:ec2:Instance       webserver          create     
 
Resources:
    + 5 to create

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_wim/dev/updates/1

     Type                      Name               Status      
 +   pulumi:pulumi:Stack       aws_ec2_wim-dev    created     
 +   ├─ aws:ec2:Vpc            myVpc              created     
 +   ├─ aws:ec2:Subnet         mySubnet           created     
 +   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080  created     
 +   └─ aws:ec2:Instance       webserver          created     
 
Outputs:
    publicIp      : "3.126.120.156"

Resources:
    + 5 created

Duration: 34s

```

![pulumi](/images/2021-05-05-3.png)
![pulumi](/images/2021-05-05-4.png)
![pulumi](/images/2021-05-05-5.png)

Code for this small variant can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/InfraAsCode/Webserver/Pulumi_newVPC).