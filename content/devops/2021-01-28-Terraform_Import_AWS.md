---
title: Terraform import AWS resources
date: 2021-01-20T14:39:50+01:00
draft: True
categories:
  - DevOps
  - All
tags:
  - Terraform
  - AWS
---

http://100daysofdevops.com/21-days-of-aws-using-terraform-day-20-importing-existing-aws-resources-to-terraform/


### Introduction

![ec2](/images/2021-01-28-1.png)

### Code

```terraform
provider "aws" {
  access_key = "***"
  secret_key = "***"
  region     = "eu-west-3"
}

resource "aws_instance" "import_ec2" {
  ami           = var.aws_ami
  instance_type = var.instance_type
  key_name      = var.key_name
  tags = {
    name    =   "imported-instance"
  }
}

```
with variables file:

```terraform
variable "aws_ami" {
  default = "ami-00798d7180f25aac2"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "key_name" {
  default = "AWS_Paris"
}
```



```bash
~/Terraform/Import master ?1 ❯ terraform init 
```


```
~/S/Programming/blog-hugo-netlify-code/Terraform/Import master ?1 ❯ terraform import aws_instance.import_ec2 i-0443dcc2f431f80dc 
aws_instance.import_ec2: Importing from ID "i-0443dcc2f431f80dc"...
aws_instance.import_ec2: Import prepared!
  Prepared aws_instance for import
aws_instance.import_ec2: Refreshing state... [id=i-0443dcc2f431f80dc]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

The Terraform state file becomes available
```
{
  "version": 4,
  "terraform_version": "0.13.5",
  "serial": 1,
  "lineage": "9a414fb8-a550-1f81-42f0-6950e6085979",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "import_ec2",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            "ami": "ami-00798d7180f25aac2",
            "arn": "arn:aws:ec2:eu-west-3:852350637351:instance/i-0443dcc2f431f80dc",
            "associate_public_ip_address": true,
            "availability_zone": "eu-west-3a",
            "cpu_core_count": 1,
            "cpu_threads_per_core": 1,
            "credit_specification": [
              {
                "cpu_credits": "standard"
              }
            ],
            "disable_api_termination": false,
            "ebs_block_device": [],
            "ebs_optimized": false,
            "enclave_options": [
              {
                "enabled": false
              }
            ],
            "ephemeral_block_device": [],
            "get_password_data": false,
            "hibernation": false,
            "host_id": null,
            "iam_instance_profile": "",
            "id": "i-0443dcc2f431f80dc",
            "instance_initiated_shutdown_behavior": null,
            "instance_state": "running",
            "instance_type": "t2.micro",
            "ipv6_address_count": 0,
            "ipv6_addresses": [],
            "key_name": "AWS_Paris",
            "metadata_options": [
              {
                "http_endpoint": "enabled",
                "http_put_response_hop_limit": 1,
                "http_tokens": "optional"
              }
            ],
            "monitoring": false,
            "network_interface": [],
            "outpost_arn": "",
            "password_data": "",
            "placement_group": "",
            "primary_network_interface_id": "eni-07309ef758393f81f",
            "private_dns": "ip-172-31-15-30.eu-west-3.compute.internal",
            "private_ip": "172.31.15.30",
            "public_dns": "ec2-15-188-75-79.eu-west-3.compute.amazonaws.com",
            "public_ip": "15.188.75.79",
            "root_block_device": [
              {
                "delete_on_termination": true,
                "device_name": "/dev/xvda",
                "encrypted": false,
                "iops": 100,
                "kms_key_id": "",
                "tags": {},
                "throughput": 0,
                "volume_id": "vol-0ad5824779e48b1b7",
                "volume_size": 8,
                "volume_type": "gp2"
              }
            ],
            "secondary_private_ips": [],
            "security_groups": [
              "launch-wizard-5"
            ],
            "source_dest_check": true,
            "subnet_id": "subnet-85dc9aec",
            "tags": {},
            "tenancy": "default",
            "timeouts": {
              "create": null,
              "delete": null,
              "update": null
            },
            "user_data": null,
            "user_data_base64": null,
            "volume_tags": null,
            "vpc_security_group_ids": [
              "sg-02e4776ae6b8044e5"
            ]
          },
          "private": "eyJlMmJmYjczMC1lY2FhLTExZTYtOGY4OC0zNDM2M2JjN2M0YzAiOnsiY3JlYXRlIjo2MDAwMDAwMDAwMDAsImRlbGV0ZSI6MTIwMDAwMDAwMDAwMCwidXBkYXRlIjo2MDAwMDAwMDAwMDB9LCJzY2hlbWFfdmVyc2lvbiI6IjEifQ=="
        }
      ]
    }
  ]
}
```

```
~/S/Programming/blog-hugo-netlify-code/Terraform/Import master ?1 ❯ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_instance.import_ec2: Refreshing state... [id=i-0443dcc2f431f80dc]

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_instance.import_ec2 will be updated in-place
  ~ resource "aws_instance" "import_ec2" {
        ami                          = "ami-00798d7180f25aac2"
        arn                          = "arn:aws:ec2:eu-west-3:852350637351:instance/i-0443dcc2f431f80dc"
        associate_public_ip_address  = true
        availability_zone            = "eu-west-3a"
        cpu_core_count               = 1
        cpu_threads_per_core         = 1
        disable_api_termination      = false
        ebs_optimized                = false
        get_password_data            = false
        hibernation                  = false
        id                           = "i-0443dcc2f431f80dc"
        instance_state               = "running"
        instance_type                = "t2.micro"
        ipv6_address_count           = 0
        ipv6_addresses               = []
        key_name                     = "AWS_Paris"
        monitoring                   = false
        primary_network_interface_id = "eni-07309ef758393f81f"
        private_dns                  = "ip-172-31-15-30.eu-west-3.compute.internal"
        private_ip                   = "172.31.15.30"
        public_dns                   = "ec2-15-188-75-79.eu-west-3.compute.amazonaws.com"
        public_ip                    = "15.188.75.79"
        secondary_private_ips        = []
        security_groups              = [
            "launch-wizard-5",
        ]
        source_dest_check            = true
        subnet_id                    = "subnet-85dc9aec"
      ~ tags                         = {
          + "name" = "imported-instance"
        }
        tenancy                      = "default"
        vpc_security_group_ids       = [
            "sg-02e4776ae6b8044e5",
        ]

        credit_specification {
            cpu_credits = "standard"
        }

        enclave_options {
            enabled = false
        }

        metadata_options {
            http_endpoint               = "enabled"
            http_put_response_hop_limit = 1
            http_tokens                 = "optional"
        }

        root_block_device {
            delete_on_termination = true
            device_name           = "/dev/xvda"
            encrypted             = false
            iops                  = 100
            tags                  = {}
            throughput            = 0
            volume_id             = "vol-0ad5824779e48b1b7"
            volume_size           = 8
            volume_type           = "gp2"
        }

        timeouts {}
    }

Plan: 0 to add, 1 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```


### Example using Terraforming tool

```bash
~/Terraform/Import master ?1 ❯ sudo gem install terraforming
Password:
Fetching aws-partitions-1.418.0.gem
Fetching aws-sigv4-1.2.2.gem
Fetching aws-sdk-dynamodb-1.58.0.gem
Fetching aws-sdk-autoscaling-1.53.0.gem
...
...
24 gems installed
```

```
~/Terraform/Import master ?1 ❯ export AWS_ACCESS_KEY_ID=A***H 
~/Terraform/Import master ?1 ❯ export AWS_SECRET_ACCESS_KEY=G***n
~/Terraform/Import master ?1 ❯ export AWS_REGION=eu-west-3 
```


```
~/S/Programming/blog-hugo-netlify-code/Terraform/Import master ?1 ❯ terraforming ec2                                                                         14:03:23
resource "aws_instance" "i-0443dcc2f431f80dc" {
    ami                         = "ami-00798d7180f25aac2"
    availability_zone           = "eu-west-3a"
    ebs_optimized               = false
    instance_type               = "t2.micro"
    monitoring                  = false
    key_name                    = "AWS_Paris"
    subnet_id                   = "subnet-85dc9aec"
    vpc_security_group_ids      = ["sg-02e4776ae6b8044e5"]
    associate_public_ip_address = true
    private_ip                  = "172.31.15.30"
    source_dest_check           = true

    root_block_device {
        volume_type           = "gp2"
        volume_size           = 8
        delete_on_termination = true
    }

    tags {
    }
}
```