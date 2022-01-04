---
title: AWS-CDK - Create AWS EC2 instance (existing VPC)
date: 2021-11-01T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Infrastructure As Code
  - Public Cloud
  - All
tags:
  - AWS
  - AWS-CDK
---

### Introduction

AWS have always positioned Cloudformation as their Infrastructure as Code tool. However if you ever used Cloudformation, you'll surely have noticed that it could be quite hard at times to use the Cloudformation specification. It's not immediately the easiest framework to learn although it works pretty well. One of the disadvantages though is that you need to learn a specific Cloudformation syntax. Now, AWS have released already quite some time ago the AWS Cloud Development Kit (AWS CDK), specifically developed to allow you to use your favorite programming languag in order to provision your infrastructure.

In this post, we will use AWS CDK to create an EC2 instance. In fact very similar to the work we did in this post because I wanted to get a better feel on how both Pulumi and AWS CDK are different.

### Installing AWS-CDK

Let's begin with the installation of the AWS CDK. The AWS CDK toolkit is installed using Node Package Manager. On my Macbook, I ran the following command (see [here](https://docs.aws.amazon.com/cdk/latest/guide/cli.html)):

```bash
/Webserver/AmazonCDK_existingVPC❯ npm install -g aws-cdk
/Webserver/AmazonCDK_existingVPC❯ cdk --version
1.102.0 (build a75d52f)
```

### Create new project

To start of, we will initialize the app using the `cdk init` command and specifying we want to use Python as our programming language. You'll notice that a Python app has been created in your directory. Also a virtual environment is created under the .venv directory.

```bash
/Webserver/AmazonCDK_existingVPC❯ cdk init --language=python
Applying project template app for python

# Welcome to your CDK Python project!

This is a blank project for Python development with CDK.

The `cdk.json` file tells the CDK Toolkit how to execute your app.

This project is set up like a standard Python project.  The initialization
process also creates a virtualenv within this project, stored under the `.venv`
directory.  To create the virtualenv it assumes that there is a `python3`
(or `python` for Windows) executable in your path with access to the `venv`
package. If for any reason the automatic creation of the virtualenv fails,
you can create the virtualenv manually.

<TRUNCATED>

## Useful commands

 * `cdk ls`          list all stacks in the app
 * `cdk synth`       emits the synthesized CloudFormation template
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk docs`        open CDK documentation

Enjoy!

Please run 'python3 -m venv .venv'!
Executing Creating virtualenv...
✅ All done!
```

As a virtual environment has been created we'll need to activate it.

```bash
/Webserver/AmazonCDK_existingVPC❯ source .venv/bin/activate
```

And next, we'll install all the dependencies in the requirements file as follows.

```bash
/Webserver/AmazonCDK_existingVPC❯ pip3 install -r requirements.txt
```

### Code to create an EC2 instance

Next, we'll make the necessary changes to create an EC2 instance as well as the required security groups. As a first step, we'll configure some environment variables

```bash
export CDK_DEFAULT_ACCOUNT=90***26
export CDK_DEFAULT_REGION=eu-central-1
export AWS_ACCESS_KEY_ID=******
export AWS_SECRET_ACCESS_KEY=******
```

We also need to make sure AWS CDK knows about these CDK specific environment variables. Hence we need to make a small change in the `app.py` file:

```python
from aws_cdk import core
from amazon_cdk_existing_vpc.amazon_cdk_existing_vpc_stack import AmazonCdkExistingVpcStack

app = core.App()
AmazonCdkExistingVpcStack(app, "AmazonCdkExistingVpcStack",
    env=core.Environment(
        account=os.getenv('CDK_DEFAULT_ACCOUNT'),
        region=os.getenv('CDK_DEFAULT_REGION')
    ),
)

app.synth()
```

One additional change is to inform the AWS-CDK to install the `aws-cdk.aws_ec2` library. We need to add the following to the `setup.py` file.

```python
install_requires=[
    "aws-cdk.core==1.109.0",
    "aws-cdk.aws_ec2"
]
```

As our goal is to create an EC2 instance, we need to add the following import statement to the `amazon_cdk_stack.py` file in the `aws_cdk` directory:

```python
from aws_cdk import (
    core,
    aws_ec2 as ec2
)
```

We will also need to do the following:

```bash
/Webserver/AmazonCDK_existingVPC❯ pip3 install aws-cdk.aws_ec2
```

Ths file (`aws_cdk_stack.py`) should also contain the code that defines your stack. We will perform the following actions on order to create our stack:

1. Look up the VPC in which we want to create the EC2 instance: we pass the VPC Id as a variable
2. Create a new security group
3. Add an ingress rule to the newly created security group
4. Create the EC2 instance: we pass the instancename, the ami name and the instanceType as variables

```python
instanceName="aws-cdk-instance"
instanceType="t2.micro"
amiName="amzn2-ami-hvm-2.0.20200520.1-x86_64-gp2"
vpcID = "vpc-3187185b"

script = """#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 8080 &
"""

user_data = ec2.UserData.custom(script)

class AmazonCdkExistingVpcStack(cdk.Stack):

    def __init__(self, scope: cdk.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        vpc = ec2.Vpc.from_lookup(
            self,
            "vpc",
            vpc_id=vpcID
        )

        sec_group = ec2.SecurityGroup(
            self,
            "sec-group-allow-http",
            vpc=vpc,
            allow_all_outbound=True,
        )

        sec_group.add_ingress_rule(
            peer=ec2.Peer.ipv4('0.0.0.0/0'),
            description="Allow HTTP connection",
            connection=ec2.Port.tcp(8080)
        )

        ec2_instance = ec2.Instance(
            self,
            "ec2-instance",
            instance_name=instanceName,
            instance_type=ec2.InstanceType(instanceType),
            machine_image=ec2.MachineImage().lookup(name=amiName),
            vpc=vpc,
            security_group=sec_group,
            user_data=user_data
        )
```

### Some more explanation

The above code probably requires a bit more explanation, so let's break it up in several pieces:

```python
vpc = ec2.Vpc.from_lookup(self, "vpc", vpc_id=vpcID)
```

In the above snippet, we are using the `ec2.Vpc` class. The documentation for this class can be found [here](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_ec2/Vpc.html#vpc). There you can read that `Vpc` can be used to create a VPC. However, we don't want to create a new VPC, we want to use an existing one. Luckily, there is a static method `from_lookup` available that can be used to achieve that. The documentation for this method can be found [here](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_ec2/Vpc.html#aws_cdk.aws_ec2.Vpc.from_lookup). You'll read that this function only needs to be used to use VPCs not defined in your CDK application which is the case.

```python
sec_group = ec2.SecurityGroup(self,"sec-group-allow-http",vpc=vpc,allow_all_outbound=True)
sec_group.add_ingress_rule(peer=ec2.Peer.ipv4('0.0.0.0/0'), description="Allow HTTP connection", connection=ec2.Port.tcp(8080))
```

Next, we create a security group and we add an ingress rule to it. We are using the `ec2.SecurityGroup` class for which the documentation can be found [here](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_ec2/SecurityGroup.html). In order to create a security group, we pass the id (which is essentially a string), the VPC Id and the
allow_all_outbound parameter, which specifies whether outbound traffic is allowed or not. Next, we use the `add_ingress_rule` method to add an ingress rule to the security group, in this case to allow inbound traffic on port 8080. The documentation can be found [here](add_ingress_rule).

```python
script = """#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 8080 &
"""

user_data = ec2.UserData.custom(script)

ec2_instance = ec2.Instance(self, "ec2-instance", instance_name=instanceName, instance_type=ec2.InstanceType(instanceType), machine_image=ec2.MachineImage().lookup(name=amiName),vpc=vpc, security_group=sec_group, user_data=user_data)
```

Lastly, we create the EC2 instance itself. Documentation can be found [here](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_ec2/Instance.html). It's a pretty straightforward class to use. We are just poassing the instance name, the instance type, the AMI image, the VPC Id and the security group. One particular point to mention here is that we don't have the AMI id for the image we want to use. So we need to look it up through the ec2.MachineImage().lookup() method for which the documentation can be found [here](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_ec2/MachineImage.html#aws_cdk.aws_ec2.MachineImage.lookup).

Another item to point out is how we handle the user data. We simply put our script in a variable called `script`. Then there is the UserData class (see [here](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_ec2/UserData.html)) that we can use to add the script to our object (e.g. we are using the custom method [here](https://docs.aws.amazon.com/cdk/api/latest/python/aws_cdk.aws_ec2/UserData.html))

### Using CDK to deploy our infrastructure

```bash
(.venv) /Webserver/AmazonCDK_existingVPC❯ cdk deploy
Searching for AMI in 904151566226:eu-central-1
This deployment will make potentially sensitive changes according to your current security approval level (--require-approval broadening).
Please confirm you intend to make the following modifications:

IAM Statement Changes
┌───┬──────────────────────────────────┬────────┬────────────────┬───────────────────────────┬───────────┐
│   │ Resource                         │ Effect │ Action         │ Principal                 │ Condition │
├───┼──────────────────────────────────┼────────┼────────────────┼───────────────────────────┼───────────┤
│ + │ ${ec2-instance/InstanceRole.Arn} │ Allow  │ sts:AssumeRole │ Service:ec2.amazonaws.com │           │
└───┴──────────────────────────────────┴────────┴────────────────┴───────────────────────────┴───────────┘
Security Group Changes
┌───┬─────────────────────────────────┬─────┬────────────┬─────────────────┐
│   │ Group                           │ Dir │ Protocol   │ Peer            │
├───┼─────────────────────────────────┼─────┼────────────┼─────────────────┤
│ + │ ${sec-group-allow-http.GroupId} │ In  │ TCP 8080   │ Everyone (IPv4) │
│ + │ ${sec-group-allow-http.GroupId} │ Out │ Everything │ Everyone (IPv4) │
└───┴─────────────────────────────────┴─────┴────────────┴─────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Do you wish to deploy these changes (y/n)? y
AmazonCdkStack: deploying...
AmazonCdkStack: creating CloudFormation changeset...

 ✅  AmazonCdkExistingVpcStack

Stack ARN:
arn:aws:cloudformation:eu-central-1:904151566226:stack/AmazonCdkExistingVpcStack/b8b44f70-d349-11eb-bce5-0610691261b8
```

Once everything got deployed, you can see the results in your AWS console:

![awscdk](/images/2021-05-10-1.png)

And here's the webpage served by our EC2 instance:

![awscdk](/images/2021-05-10-2.png)

### Modify an EC2 instance

We will keep things simple and just update the name of our EC2 instance. So the changes are as follows:

```python
# Change the name
instanceName="aws-cdk-instance-updated"
```

```bash
(.venv) /Webserver/AmazonCDK_existingVPC❯ cdk deploy
AmazonCdkExistingVpcStack: deploying...
AmazonCdkExistingVpcStack: creating CloudFormation changeset...

 ✅  AmazonCdkExistingVpcStack

Stack ARN:
arn:aws:cloudformation:eu-central-1:904151566226:stack/AmazonCdkExistingVpcStack/b8b44f70-d349-11eb-bce5-0610691261b8
```

### Destroy an EC2 instance

```
(.venv) /Webserver/AmazonCDK_existingVPC❯ cdk deploy
Are you sure you want to delete: AmazonCdkExistingVpcStack (y/n)? y
AmazonCdkExistingVpcStack: destroying...

 ✅  AmazonCdkExistingVpcStack: destroyed
```

This blog post took quite a while. To be honest, AWS-CDK is not the easiest tool and it takes some time to familiarize yourself with it to be honest. If you want to experiment further, my code can be found [here]().
