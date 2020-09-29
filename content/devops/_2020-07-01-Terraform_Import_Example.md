---
title: Terraform import example
date: 2020-07-01T14:39:50+01:00
draft: True
categories:
  - DevOps
  - All
tags:
  - Terraform
  - AWS
---
### Introduction


### Code

```terraform
provider "aws" {
  access_key = "***"
  secret_key = "***"
  region     = "eu-west-1"
}

resource "aws_instance" "ec2_import_example" {
  ami           = "ami-00035f41c82244dab"
  instance_type = "t2.large"
  vpc_security_group_ids = ["sg-c3b88cbd"]
  subnet_id = "subnet-f9b317a3"
  tags = {
    Name = "Terraform-Demo"
  }
  key_name = "AWS-Cisco"
}
```


```
➜  Import git:(master) ✗ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.ec2_import_example will be created
  + resource "aws_instance" "ec2_import_example" {
      + ami                          = "ami-00035f41c82244dab"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + availability_zone            = (known after apply)
      + cpu_core_count               = (known after apply)
      + cpu_threads_per_core         = (known after apply)
      + get_password_data            = false
      + host_id                      = (known after apply)
      + id                           = (known after apply)
      + instance_state               = (known after apply)
      + instance_type                = "t2.large"
      + ipv6_address_count           = (known after apply)
      + ipv6_addresses               = (known after apply)
      + key_name                     = "AWS-Cisco"
      + network_interface_id         = (known after apply)
      + outpost_arn                  = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      + primary_network_interface_id = (known after apply)
      + private_dns                  = (known after apply)
      + private_ip                   = (known after apply)
      + public_dns                   = (known after apply)
      + public_ip                    = (known after apply)
      + security_groups              = (known after apply)
      + source_dest_check            = true
      + subnet_id                    = "subnet-f9b317a3"
      + tags                         = {
          + "Name" = "Terraform-Demo"
        }
      + tenancy                      = (known after apply)
      + volume_tags                  = (known after apply)
      + vpc_security_group_ids       = [
          + "sg-c3b88cbd",
        ]

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```


```
➜  Import git:(master) ✗ terraform import aws_instance.ec2_import_example i-06b9f8ba213daf094
aws_instance.ec2_import_example: Importing from ID "i-06b9f8ba213daf094"...
aws_instance.ec2_import_example: Import prepared!
  Prepared aws_instance for import
aws_instance.ec2_import_example: Refreshing state... [id=i-06b9f8ba213daf094]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```


```
{
  "version": 4,
  "terraform_version": "0.12.24",
  "serial": 1,
  "lineage": "55995191-c684-4a07-29d5-f3663d4c5346",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "ec2_import_example",
      "provider": "provider.aws",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            "ami": "ami-00035f41c82244dab",
            "arn": "arn:aws:ec2:eu-west-1:852350637351:instance/i-06b9f8ba213daf094",
            "associate_public_ip_address": true,
            "availability_zone": "eu-west-1a",
            "cpu_core_count": 2,
            "cpu_threads_per_core": 1,
            "credit_specification": [
              {
                "cpu_credits": "standard"
              }
            ],
            "disable_api_termination": false,
            "ebs_block_device": [],
            "ebs_optimized": false,
            "ephemeral_block_device": [],
            "get_password_data": false,
            "hibernation": false,
            "host_id": null,
            "iam_instance_profile": "",
            "id": "i-06b9f8ba213daf094",
```


```
➜  Import git:(master) ✗ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_instance.ec2_import_example: Refreshing state... [id=i-06b9f8ba213daf094]

------------------------------------------------------------------------

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```