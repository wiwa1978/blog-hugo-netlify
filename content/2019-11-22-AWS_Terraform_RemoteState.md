---
title: Configure AWS instances with Terraform (remote state with S3)
date: 2019-11-22T14:39:50+01:00
draft: false
categories:
   - DevOps
   - Infrastructure As Code
   - Public Cloud
tags:
  - Terraform
  - AWS
---
### Introduction
In an earlier post, we created one EC2 instance. As explained in that post, the Terraform state file was created in your local folder. That’s great if you are the only person in need of the infrastructure you manage. In reality though, teams are sharing and managing the infrastructure. So we need a way to share the state file with other team members, so at all times they have to know the latest state of the infrastructure.

Hence, in this blogpost, we are going to explore a way to share the state file amongst team members. There are multiple ways to achieve this, but I tend to prefer storing the state file in an AWS S3 bucket.

### Configure the remote state location
As mentioned, we will use an S3 bucket to store the state file. To do so, create a bucket on S3 and give it the below permissions (in my case the bucket is called be.wymedia.terraform). Note that we allow Terraform to create a `state` file in the Terraform folder.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::be.wymedia.terraform"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::be.wymedia.terraform/terraform/state"
        }
    ]
}
```
Once this is done on AWS, go back to your Terraform directory and configure the remote state location. I usually call this file `terraform.tf` but it really does not matter how you call it. The content of the file should be:
```terraform
terraform {
  backend "s3" {
    bucket = "be.wymedia.terraform"
    key    = "terraform/state"
    region = "eu-west-1"
  }
}
```
The `variables.tf` file:

```terraform
variable "ec2_region" {
  default = "eu-west-1"
}

variable "ec2_image" {
  default = "ami-00035f41c82244dab"
}

variable "ec2_instance_type" {
  default = "t2.micro"
}

variable "ec2_keypair" {
  default = "AWS-Cisco"
}

variable "ec2_tags" {
  default = "Cisco-Demo-Terraform-1"
}

variable "ec2_count" {
  default = "2"
}
```
The `main.tf` file:
```terraform
resource "aws_instance" "OneServer" {
  ami           = var.ec2_image
  instance_type = var.ec2_instance_type
  key_name      = var.ec2_keypair
  count         = var.ec2_count
  tags = {
    Name = var.ec2_tags
  }
}

output "instance_ip_addr" {
  value       = aws_instance.OneServer.*.private_ip
  description = "The private IP address of the main server instance."
}

output "instance_ips" {
  value = aws_instance.OneServer.*.public_ip
}
```

### Deployment
First off, we start with `terraform init`.
```
WAUTERW-M-65P7:AWS_Terraform_RemoteState wauterw$ terraform init

Initializing the backend...

Error: No valid credential sources found for AWS Provider.
        Please see https://terraform.io/docs/providers/aws/index.html for more information on
        providing credentials for the AWS Provider
```
As you can see this does not work. For the ‘terraform init’ to succeed, it’s important you also set the correct credentials in your ~/.aws/credentials file. If not, it will fail with a ‘No valid credential sources found for AWS Provider’ error. Don’t really understand it as we specified the credentials in the Terraform file but it’s something I came across and wanted to share.

```
[default]
aws_access_key_id = AK***A
aws_secret_access_key = Ld***8j
```
Let’s retry the `terraform init`:

```bash
WAUTERW-M-65P7:AWS_Terraform_RemoteState wauterw$ terraform init

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (hashicorp/aws) 2.54.0...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
Next, issue the `terraform plan` command:

```bash
WAUTERW-M-65P7:AWS_Terraform_RemoteState wauterw$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.OneServer[0] will be created
  + resource "aws_instance" "OneServer" {
      + ami                          = "ami-00035f41c82244dab"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + availability_zone            = (known after apply)
      + cpu_core_count               = (known after apply)

**Truncated***

Plan: 2 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```
And finally, issue the `terraform apply` command:

```bash
WAUTERW-M-65P7:AWS_Terraform_RemoteState wauterw$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.OneServer[0] will be created
  + resource "aws_instance" "OneServer" {
      + ami                          = "ami-00035f41c82244dab"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)

***Truncated***

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.OneServer[1]: Creating...
aws_instance.OneServer[0]: Creating...
aws_instance.OneServer[0]: Still creating... [10s elapsed]
aws_instance.OneServer[1]: Still creating... [10s elapsed]
aws_instance.OneServer[0]: Still creating... [20s elapsed]
aws_instance.OneServer[1]: Still creating... [20s elapsed]
aws_instance.OneServer[1]: Creation complete after 24s [id=i-06ffb9b12c2e16e0b]
aws_instance.OneServer[0]: Still creating... [30s elapsed]
aws_instance.OneServer[0]: Creation complete after 34s [id=i-0ce853038b28166e0]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

instance_ip_addr = [
  "172.31.36.57",
  "172.31.47.209",
]
instance_ips = [
  "34.247.165.168",
  "34.245.202.235",
]
```
And obviously, you get back the IP addresses for the two instances. If all goes well, you will see two instances in AWS.

![AWS](/images/2019-11-22-1.png)

And a more detailed screenshot to verify the IP addresses:

![AWS](/images/2019-11-22-2.png)

If you are following along, you will sure have noticed that this time there is no tfstate file in your local folder. Consult your configured S3 bucket and you will see that it is stored over there (as expected).

![AWS](/images/2019-11-22-3.png)

Below is a screenshot of the statefile that got created by Terraform. Go through it to verify what information is writes to the state file. You will see amongst others the public and private IP addresses mentioned there.

![AWS](/images/2019-11-22-4.png)

The files can be found on [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/AWS_Terraform_RemoteState).
