---
title: Pulumi - Getting Started
date: 2022-01-17T14:39:50+01:00
draft: True
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

### Create new project

```bash
/Webserver/Pulumi❯ pulumi new aws-python --name aws_ec2_instances
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

project description: (A minimal AWS Python Pulumi program) A simple AWS EC2 instance
Created project 'aws_ec2_instances'

Please enter your desired stack name.
To create a stack in an organization, use the format <org-name>/<stack-name> (e.g. `acmecorp/dev`).
stack name: (dev) dev
Created stack 'dev'

aws:region: The AWS region to deploy into: (us-east-1) eu-west-1
Saved config

Creating virtual environment...

<TRUNCATED>

Finished installing dependencies
Your new project is ready to go! ✨

To perform an initial deployment, run 'pulumi up'


```

### Create EC2 instance

Open `__main__.py`

```python
import pulumi
import pulumi_aws as aws

size = 't3.micro'

user_data = """#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 8080 &
"""

#ami = aws.ec2.get_ami(most_recent="true",
#                  owners=["099720109477"],
#                  filters=[{"name":"name","values":["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]}])
ami = aws.ec2.get_ami(most_recent="true",
                  owners=["137112412989"],
                  filters=[{"name":"name","values":["amzn-ami-hvm-*"]}])


group = aws.ec2.SecurityGroup('pulumi_allow_8080',
            description='Enable access to port 8080',
            ingress=[
                { 'protocol': 'tcp', 'from_port': 22, 'to_port': 22, 'cidr_blocks': ['0.0.0.0/0'] },
                { 'protocol': 'tcp', 'from_port': 8080, 'to_port': 8080, 'cidr_blocks': ['0.0.0.0/0'] }
            ])

server = aws.ec2.Instance('webserver',
    instance_type=size,
    vpc_security_group_ids=[group.id],
    ami=ami.id,
    user_data = user_data,
    tags = { "Name": "Testserver-Pulumi" },
    )


pulumi.export('publicIp', server.public_ip)
pulumi.export('publicHostName', server.public_dns)
```

### Deploy resources

```bash
/Webserver/Pulumi❯ pulumi up
Previewing update (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_instances/dev/previews/b28356ca-49ca-4494-9259-1d5941359359

     Type                      Name                   Plan
 +   pulumi:pulumi:Stack       aws_ec2_instances-dev  create
 +   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080      create
 +   └─ aws:ec2:Instance       webserver              create

Resources:
    + 3 to create

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_instances/dev/updates/1

     Type                      Name                   Status
 +   pulumi:pulumi:Stack       aws_ec2_instances-dev  created
 +   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080      created
 +   └─ aws:ec2:Instance       webserver              created

Outputs:
    publicHostName: "ec2-54-77-125-139.eu-west-1.compute.amazonaws.com"
    publicIp      : "54.77.125.139"

Resources:
    + 3 created

Duration: 27s
```

### View resources

```bash
/Webserver/Pulumi❯ pulumi stack
Current stack is dev:
    Owner: wiwa1978
    Last updated: 3 minutes ago (2021-05-05 11:49:30.163976 +0200 CEST)
    Pulumi version: v3.1.0
Current stack resources (5):
    TYPE                                        NAME
    pulumi:pulumi:Stack                         aws_ec2_instances-dev
    ├─ aws:ec2/securityGroup:SecurityGroup      pulumi_allow_8080
    ├─ aws:ec2/instance:Instance                webserver
    ├─ pulumi:providers:aws                     default
    └─ pulumi:providers:aws                     default_4_2_0

Current stack outputs (2):
    OUTPUT          VALUE
    publicHostName  ec2-54-77-125-139.eu-west-1.compute.amazonaws.com
    publicIp        54.77.125.139

More information at: https://app.pulumi.com/wiwa1978/aws_ec2_instances/dev
```

Photos: pulumi-1, 2 en 3

### Update the resources

Updates:

- remove the security group 22
- rename the server

```python
import pulumi
import pulumi_aws as aws

size = 't3.micro'

user_data = """#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 8080 &
"""

#ami = aws.ec2.get_ami(most_recent="true",
#                  owners=["099720109477"],
#                  filters=[{"name":"name","values":["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]}])
ami = aws.ec2.get_ami(most_recent="true",
                  owners=["137112412989"],
                  filters=[{"name":"name","values":["amzn-ami-hvm-*"]}])


group = aws.ec2.SecurityGroup('pulumi_allow_8080',
            description='Enable access to port 8080',
            ingress=[
                { 'protocol': 'tcp', 'from_port': 8080, 'to_port': 8080, 'cidr_blocks': ['0.0.0.0/0'] }
            ])

server = aws.ec2.Instance('webserver',
    instance_type=size,
    vpc_security_group_ids=[group.id],
    ami=ami.id,
    user_data = user_data,
    tags = { "Name": "Testserver-Pulumi-updated" },
    )


pulumi.export('publicIp', server.public_ip)
pulumi.export('publicHostName', server.public_dns)
```

```bash
/Webserver/Pulumi❯ pulumi up
Previewing update (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_instances/dev/previews/32d85d81-c43b-433f-9b3c-034f7f6f5504

     Type                      Name                   Plan       Info
     pulumi:pulumi:Stack       aws_ec2_instances-dev
 ~   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080      update     [diff: ~ingress]
 ~   └─ aws:ec2:Instance       webserver              update     [diff: ~tags]

Resources:
    ~ 2 to update
    1 unchanged

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_instances/dev/updates/2

     Type                      Name                   Status      Info
     pulumi:pulumi:Stack       aws_ec2_instances-dev
 ~   ├─ aws:ec2:SecurityGroup  pulumi_allow_8080      updated     [diff: ~ingress]
 ~   └─ aws:ec2:Instance       webserver              updated     [diff: ~tags]

Outputs:
    publicHostName: "ec2-54-77-125-139.eu-west-1.compute.amazonaws.com"
    publicIp      : "54.77.125.139"

Resources:
    ~ 2 updated
    1 unchanged

Duration: 11s
```

Photos: pulumi-4 en 5

### Destroy the resources

- Run pulumi destroy to tear down all resources. You'll be prompted to make sure you really want to delete these resources. A destroy operation may take some time, since Pulumi waits for the resources to finish shutting down before it considers the destroy operation to be complete.
- To delete the stack itself, run pulumi stack rm. Note that this command deletes all deployment history from the Pulumi Console.

```bash
/Webserver/Pulumi❯  pulumi destroy
Previewing destroy (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_instances/dev/previews/0a4e7b05-4d1b-4b3f-b6a3-f4d214f730c9

     Type                      Name                   Plan
 -   pulumi:pulumi:Stack       aws_ec2_instances-dev  delete
 -   ├─ aws:ec2:Instance       webserver              delete
 -   └─ aws:ec2:SecurityGroup  pulumi_allow_8080      delete

Outputs:
  - publicHostName: "ec2-54-77-125-139.eu-west-1.compute.amazonaws.com"
  - publicIp      : "54.77.125.139"

Resources:
    - 3 to delete

Do you want to perform this destroy? yes
Destroying (dev)

View Live: https://app.pulumi.com/wiwa1978/aws_ec2_instances/dev/updates/3

     Type                      Name                   Status
 -   pulumi:pulumi:Stack       aws_ec2_instances-dev  deleted
 -   ├─ aws:ec2:Instance       webserver              deleted
 -   └─ aws:ec2:SecurityGroup  pulumi_allow_8080      deleted

Outputs:
  - publicHostName: "ec2-54-77-125-139.eu-west-1.compute.amazonaws.com"
  - publicIp      : "54.77.125.139"

Resources:
    - 3 deleted

Duration: 36s

The resources in the stack have been deleted, but the history and configuration associated with the stack are still maintained.
If you want to remove the stack completely, run 'pulumi stack rm dev'.
```

```bash
/Webserver/Pulumi❯ pulumi stack rm dev
This will permanently remove the 'dev' stack!
Please confirm that this is what you'd like to do by typing ("dev"): dev
Stack 'dev' has been removed!
```
