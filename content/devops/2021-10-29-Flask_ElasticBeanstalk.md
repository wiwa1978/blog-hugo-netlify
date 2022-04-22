---
title: Deploy Flask App to EBS using CLI
date: 2021-10-29T10:19:50+01:00
draft: false
categories:
  - Cloud Native
  - DevOps
  - All
tags:
  - AWS
  - Flask
  - Elastic Beanstalk
---

### Introduction

In [this](https://blog.wimwauters.com/devops/2021-02-01-flaskbasic/) post, I created a very basic Flask application. The idea behind this post is to deploy the application to AWS Elastic Beanstalk using the Beanstalk CLI.

### What is AWS Beanstalk

AWS Beanstalk is a PaaS style platform offered by AWS. It allows you to easily deploy and manage applications without having to worry about the underlying infrastructure. The easiest way to get started with AWS Beanstalk is through the CLI.

### Install Beanstalk CLI

Let's go ahead and install the CLI. I'm using MAC, so you can follow along. For other platforms, you can follow the install instructions [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html#eb-cli3-install.scripts).

```bash
~/Flask/Flask-Basic-Beanstalk ❯ brew install awsebcli
```

### Application Code

Next, clone the code (code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Flask/Flask-Basic-Beanstalk)). Then create a Python virtual environment and install the requirements as follows:

```bash
~/Flask/Flask-Basic-Beanstalk ❯ python3 -m venv venv
~/Flask/Flask-Basic-Beanstalk ❯ source venv/bin/activate
~/Flask/Flask-Basic-Beanstalk ❯ pip3 install -r requirements.txt
<TRUNCATED>
```

### Run the application locally

Since we cloned the repository, we can run the application locally first to verify if everything works locally and all dependencies are resolved. To start the application, use the command below. This is nothing different from what we did in the earlier [post](https://blog.wimwauters.com/devops/2021-02-01-flaskbasic/).

```bash
~/Flask/Flask-Basic-Beanstalk ❯ gunicorn --bind 0.0.0.0:5000 wsgi:application -w 1
[2021-10-29 11:35:11 +0200] [96917] [INFO] Starting gunicorn 20.0.4
[2021-10-29 11:35:11 +0200] [96917] [INFO] Listening at: http://0.0.0.0:5000 (96917)
[2021-10-29 11:35:11 +0200] [96917] [INFO] Using worker: sync
[2021-10-29 11:35:11 +0200] [96919] [INFO] Booting worker with pid: 96919
```

![flask-basic-beanstalk](/images/2021-10-29-1.png)

Obviously the app is at this moment still running on our local environment, so note the words `will be deployed`.

### Create Beanstalk environment

Next, let's focus on the AWS Beanstalk environment. To do so, create a user in IAM and attach the `AdministratorAccess-AWSElasticBeanstalk` policy to the user you just created. Next, add your AWS credentials to the `~/.aws/credentials` file:

```bash
[elasticbeanstalk]
   aws_access_key_id = ***
   aws_secret_access_key = ***
```

Run the following command in your project folder to initialize the application and register it with AWS Elastic Beanstalk

```bash
~/Flask/Flask-Basic-Beanstalk ❯ eb init -p python-3.8 Flask_Example_EB --region eu-central-1 --profile elasticbeanstalk
Application Flask_Example_EB has been created.
```

Note: the `--profile elasticbeanstalk` refers to the entry you created in the `~/.aws/credentials` file.

Ensure you go to `Applications` on the UI and you will see that an Elastic Beanstalk environment has been created for us.

![flask-basic-beanstalk](/images/2021-10-29-2.png)

Next we need to create an environment (which takes a while to get created)

```bash
~/Flask/Flask-Basic-Beanstalk ❯ eb create test-environment
Creating application version archive "app-211027_152611".
Uploading Flask_Example_EB/app-211027_152611.zip to S3. This may take a while.
Upload Complete.
Environment details for: test-environment
  Application name: Flask_Example_EB
  Region: eu-central-1
  Deployed Version: app-211027_152611
  Environment ID: e-jm8dtxx7wy
  Platform: arn:aws:elasticbeanstalk:eu-central-1::platform/Python 3.8 running on 64bit Amazon Linux 2/3.3.7
  Tier: WebServer-Standard-1.0
  CNAME: UNKNOWN
  Updated: 2021-10-27 13:26:16.321000+00:00
Printing Status:
2021-10-27 13:26:15    INFO    createEnvironment is starting.
2021-10-27 13:26:16    INFO    Using elasticbeanstalk-eu-central-1-852350637351 as Amazon S3 storage bucket for environment data.
2021-10-27 13:26:41    INFO    Created security group named: sg-045a49bae742c56bc
2021-10-27 13:26:56    INFO    Created load balancer named: awseb-e-j-AWSEBLoa-1YPNL9YRGWT2
2021-10-27 13:26:57    INFO    Created security group named: awseb-e-jm8dtxx7wy-stack-AWSEBSecurityGroup-FWEI4WY5W0BG
2021-10-27 13:26:57    INFO    Created Auto Scaling launch configuration named: awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingLaunchConfiguration-16PNYHNEG416L
2021-10-27 13:28:00    INFO    Created Auto Scaling group named: awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingGroup-1N6JPPTQ4O6I6
2021-10-27 13:28:00    INFO    Waiting for EC2 instances to launch. This may take a few minutes.
2021-10-27 13:28:00    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:eu-central-1:852350637351:scalingPolicy:5528e776-782f-4fbb-b4c1-85b5d84cccc0:autoScalingGroupName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingGroup-1N6JPPTQ4O6I6:policyName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingScaleUpPolicy-15AKQZ5X6L9E3
2021-10-27 13:28:00    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:eu-central-1:852350637351:scalingPolicy:ecf24569-8f80-4551-b540-ed5165af689a:autoScalingGroupName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingGroup-1N6JPPTQ4O6I6:policyName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingScaleDownPolicy-Y665GHYKX01H
2021-10-27 13:28:00    INFO    Created CloudWatch alarm named: awseb-e-jm8dtxx7wy-stack-AWSEBCloudwatchAlarmHigh-9IVPW7RSG1WC
2021-10-27 13:28:00    INFO    Created CloudWatch alarm named: awseb-e-jm8dtxx7wy-stack-AWSEBCloudwatchAlarmLow-PPZYWD1FDE35
2021-10-27 13:28:08    INFO    Instance deployment successfully generated a 'Procfile'.
2021-10-27 13:28:11    INFO    Instance deployment completed successfully.
2021-10-27 13:28:41    INFO    Application available at test-environment.eba-dm2jtsxc.eu-central-1.elasticbeanstalk.com.
2021-10-27 13:28:42    INFO    Successfully launched environment: test-environment
```

In your AWS console, under the `Environments` tab you will find the created environment:

![flask-basic-beanstalk](/images/2021-10-29-3.png)

You'll find it in a healthy state.

![flask-basic-beanstalk](/images/2021-10-29-4.png)

In the output abovem you will notice that AWS has created all the underlying infrastructure for our application. You can see it created a security group, a load balancer, an auto scaling group and more.

The CLI makes it easy to view the deployed application. Just issue the `eb open` command:

```bash
~/Flask/Flask-Basic-Beanstalk ❯ eb open
```

![flask-basic-beanstalk](/images/2021-10-29-6.png)

### Update application

Now that our basic Flask application is running on AWS Beanstalk, let's make a little change to the application. In my case, I have updated the `index.html` file. I added the words `version 2` to the smaller tagline under the title of the webpage. After you make the required changes, use the `eb deploy` command to deploy this version. In the output below, Beanstalk mentions indeed a new application version was installed.

```bash
~/Flask/Flask-Basic-Beanstalk ❯ eb deploy
Creating application version archive "app-211027_153326".
Uploading Flask_Example_EB/app-211027_153326.zip to S3. This may take a while.
Upload Complete.
2021-10-27 13:33:28    INFO    Environment update is starting.
2021-10-27 13:33:32    INFO    Deploying new version to instance(s).
2021-10-27 13:33:36    INFO    Instance deployment successfully generated a 'Procfile'.
2021-10-27 13:33:44    INFO    Instance deployment completed successfully.
2021-10-27 13:33:50    INFO    New application version was deployed to running EC2 instances.
2021-10-27 13:33:50    INFO    Environment update completed successfully.
```

Look at the below screenshot to see our two versions.

![flask-basic-beanstalk](/images/2021-10-29-7.png)

And indeed, refresh the browser and you will see our tagline got updated indeed.

![flask-basic-beanstalk](/images/2021-10-29-8.png)

### Terminate the environment

We've come a far way by now. If you are following along this blog post, ensure you also destroy the Beanstalk environment. Otherwise all these resources will continue to run and you'll receive the bill from AWS soon enough. Luckily, AWS makes it really easy for us. Just issue the `eb terminate` command and it'll take care of the destruction of all resources.

```bash
~/Flask/Flask-Basic-Beanstalk ❯ eb terminate
The environment "test-environment" and all associated instances will be terminated.
To confirm, type the environment name: test-environment
2021-10-27 13:39:47    INFO    terminateEnvironment is starting.
2021-10-27 13:40:05    INFO    Deleted CloudWatch alarm named: awseb-e-jm8dtxx7wy-stack-AWSEBCloudwatchAlarmLow-PPZYWD1FDE35
2021-10-27 13:40:05    INFO    Deleted CloudWatch alarm named: awseb-e-jm8dtxx7wy-stack-AWSEBCloudwatchAlarmHigh-9IVPW7RSG1WC
2021-10-27 13:40:05    INFO    Deleted Auto Scaling group policy named: arn:aws:autoscaling:eu-central-1:852350637351:scalingPolicy:ecf24569-8f80-4551-b540-ed5165af689a:autoScalingGroupName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingGroup-1N6JPPTQ4O6I6:policyName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingScaleDownPolicy-Y665GHYKX01H
2021-10-27 13:40:05    INFO    Deleted Auto Scaling group policy named: arn:aws:autoscaling:eu-central-1:852350637351:scalingPolicy:5528e776-782f-4fbb-b4c1-85b5d84cccc0:autoScalingGroupName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingGroup-1N6JPPTQ4O6I6:policyName/awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingScaleUpPolicy-15AKQZ5X6L9E3
2021-10-27 13:40:05    INFO    Waiting for EC2 instances to terminate. This may take a few minutes.
2021-10-27 13:42:07    INFO    Deleted Auto Scaling group named: awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingGroup-1N6JPPTQ4O6I6
2021-10-27 13:42:07    INFO    Deleted load balancer named: awseb-e-j-AWSEBLoa-1YPNL9YRGWT2
2021-10-27 13:42:07    INFO    Deleted Auto Scaling launch configuration named: awseb-e-jm8dtxx7wy-stack-AWSEBAutoScalingLaunchConfiguration-16PNYHNEG416L
2021-10-27 13:42:07    INFO    Deleted security group named: awseb-e-jm8dtxx7wy-stack-AWSEBSecurityGroup-FWEI4WY5W0BG
2021-10-27 13:42:38    INFO    Deleted security group named: sg-045a49bae742c56bc
2021-10-27 13:42:40    INFO    Deleting SNS topic for environment test-environment.
2021-10-27 13:42:41    INFO    terminateEnvironment completed successfully.
```

You will see that the application and environment is still visible despite the `eb terminate` command. However, looking under the environment the UI says `This environment is terminated and cannot be modified. It will remain visible for about an hour`.

![flask-basic-beanstalk](/images/2021-10-29-9.png)

Code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Flask/Flask-Basic-Beanstalk).
