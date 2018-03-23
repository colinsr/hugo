---
title: "AWS EC2"
date: 2018-03-22T06:58:44-04:00
tags: ["aws", "amazon web services", "EC2"]
categories: ["cloud"]
---

In this post we're going to walk through setting an AWS EC2 (elastic compute cloud) instance.  During the course of this post we'll talk through some of the different options available and why they're important.

Up until now we've been going over what some may consider boring.  [IAM Permissions](/post/AWS_IAM) and [VPC Networking](/post/AWS_VPC) are not necessarily at the top of everyone's list of things they have queued up to learn.  Well I have good news, we're going to have a pair of web servers running in EC2 by the time you finish reading this post.  And all of that Permission and Networking knowledge is really going to pay off.

A quick recap of what we did in the VPC post is in order.  We created a new VPC, created two public subnets, created/attached an Internet Gateway to our VPC, created a new Route Table and added a route for all non-local traffic to pass through the IGW remembering of course to associate it to our public Subnets, added a new Network Access Control List with the requisite ingress/egress rules - SSH from my IP and HTTP/S traffic over ports 80 & 443 as well as the ephemeral ports 32768-65535 before associating it with our public Subnets,  then we created a Security Group and added the same inbound rules omitting the outbound since SGs are stateful.<br>
We certainly were busy.

Picking up where we left off we're going to be creating 2 Amazon linux EC2 instances to act as web servers.  I'll walk through the process on the web server we deploy to our `public-1` Subnet and then you can rinse and repeat for the web server in the `public-2` subnet.

## What is an EC2 instance??
AWS EC2 is just another one of the services that AWS provides.  They are virtual servers, that can be scaled up (add more power) or scaled out (add more capacity) on demand and are able to run workloads on Linux or Windows Operating Systems.<br>
The starting point for an EC2 instance is going to be the AMI or Amazon Machine Image.  An AMI includes some of the OS level settings, root drives, additional drives attached to an instance, installed software/packages, as well as the Operating System itself.  You can use an AMI provided by AWS, any number of companies who provide their own AMIs in the Community Marketplace, or of course, one of your own.

Next we have the Instance Type.  AWS is a bit Baskin Robbins-ish in that they have so many flavors to choose from. Different Instance Types are suited for different workloads, sometimes you'll need to do a little testing to see what Instance Type will work best for you.<br>
They can be broken down into these 5 broad categories:

* General Purpose
* Compute Optimized
* Memory Optimized
* Accelerated Computing
* Storage Optimized

Some have more CPU power, some more memory, etc etc.  We'll be sticking with the AWS Free Tier in this post - which means we'll be using a `t2.micro` instance.  Each Instance Type can be provisioned with different specs allowing you to keep the same type but scale up to get more power.

Network interface card is another option we have.  While a NIC is required we do have some choices to make here.<br>
Do we want only a private IP?  Is a dynamic public IP sufficient for our needs or do we need a static IP?<br>
If we're going to need a static IP then we're going to need to allocate an Elastic IP.

Storage is up next.  There are two options for storage types on an EC2 instance, Instance Store and Elastic Block Store.<br>
EBS is persistent storage, where the EBS volume can continue to live after the EC2 instance has been terminated.<br>
Instance store is also known as ephemeral store, and with this storage option you will lose any data on your storage once your instance has been stopped or terminated.<br>
Of course in addition to the type you have size options to choose as well.  Not to mention that EBS volumes come in 4 different flavors:

* EBS Provisioned IOPS SSD (io1)	
* EBS General Purpose SSD (gp2)*	
* Throughput Optimized HDD (st1)	
* Cold HDD (sc1)

Since this is not a deep dive on EBS, we'll just go with GP2 since that will give us the best price/performance ratio.

Finally we can talk about the 3 purchasing options we have:

* On Demand - pay flat rate per hour
* Reserved - reserve instance for 1 - 3 years and get a discount on the hourly rate
* Spot - set a bid price and have access to instance while spot < bid price

In order to keep it simple, we're going with On Demand.

## Let's create an EC2 Instance!
Let's jump on the AWS console and select the EC2 service.  From the landing page, select Launch Instance.<br>
Step 1 is to choose an AMI, feel free to browse around and get a feel for what AMIs we have available.  Also take notice of the "Free tier eligible" tag as that will save you some money if you're still in the first 12 months of your AWS account.  On this AMI page we're going to choose Amazon Linux AMI, which should be the first option, since that includes a bunch of commonly used tools in the image.

Next we're presented with the Choose an Instance Type screen.  We're going to follow suit and choose the only Free tier eligible type, `t2.micro`.  The t2 instance family provides burstable CPU credits.  These help when you have short bursts of increased load, not to mention that they're free.

The Configure Instance Details screen is up next and this is where we can select quite a few options.

![EC2 Instance Details](/img/EC2_instance_details.png)

Here we'll make sure to choose 1 instance, target our `Dev-VPC` and `public-1` subnet.  Now this Auto-assign Public IP setting defaults to the Subnet level setting for auto-assigning public IPs, but can be overridden here to be set to enable.

If we were to breeze over this setting without assigning a public IP at launch time, we still have the option of attaching an Elastic IP to the instance after launch.  Make sure you select Enable for this to save yourself a bit of hassle.

If you went through my IAM post, you'll notice the IAM role option here. As a quick refresher, we can assign a role to an AWS resource which will give it permissions defined in the role via IAM policies.  This is a best practice and prevents us from having to use any type of AWS credentials from the instance itself.  Since we (currently) have no need to interact with any other AWS services we'll leave that blank for now.  In the past an IAM role could only be attached to an EC2 instance at launch time, but in late 2017 AWS added the ability to add/change roles attached to an instance at any time.

There are a few other settings here, which we'll accept the defaults for, but one in particular that's worth talking about is the Tenancy option.  This goes back to the Shared/Dedicated option we talked about in the VPC post.  Dedicated means you'll be paying a lot more but you'll sleep better at night knowing that your competitor's application is definitely not running on the same physical server - we'll stick with Shared.

Below the basic settings, we'll see the Network Interfaces section.  Here we can add additional NICs or choose a primary IP for use that falls within the CIDR range for our Subnet.  AWS provides DHCP for us, so unless we have a reason to assign a specific IP we'll let AWS DHCP do it's thing.

And bringing up the end of our instance details page is an Advanced Details setting.  This is where we can inject our own custom code which will be run during the instance provisioning process.

Here we can execute bash/powershell scripts to install packages or bootstrap our server to our little hearts content.  Since we're planning on using this instance as a web server, I'd say now is as good a time as any to learn how this works.
Let's add the following to the User Data setting As text.
``` bash
#!/bin/bash

yum update -y
yum install nginx -y
service nginx start
```

The Amazon Linux AMI uses `yum` as a package manager, and what we're doing here is installing and starting nginx.  These User data scripts can get a bit complicated, and in that scenario we could add an IAM role to our instance which allows downloads from a specific S3 bucket where we've parked our complicated bootstrap script.  Alternatively we could use some configuration management tool like Puppet, Chef, Ansible, or SaltStack to handle the configuration for us.<br>
We don't need to worry too much about that since our bootstrapping will just install and start nginx.

Next we're on the Add Storage screen.  Here we can increase the size of our Root volume, change the volume type or attach additional volumes.  We'll take the defaults here as they should be good enough for our purposes.

Next up is Add Tags, we'll want to add a `Name: web-1` tag to our instance so that it can clearly identify it's purpose.  There are many conventions for tagging resources, usually they revolve around Billing and resource grouping.

Step 6 is Configure Security Groups.  An Instance needs to have at least one SG applied to it at launch.  AWS is kind enough to create a new SG for us called `launch-wizard-#` which has port 22/SSH open to the world, but again if you followed along in the VPC post you should already have created a SG that allows SSH only from your own IP and is opened up on port 80 for our nginx traffic.<br>
Make sure to choose Select an existing security group, select our custom SG and we're ready to move on.

Finally, we're presented with a Review screen to make sure all of our settings are correct before launching the new instance.  Once you've validated all the settings, go ahead and click Launch.<br>

WTH, is this stuff about keypairs.  Well in order to SSH into our new server we'll need to provide a private key.  We'll go ahead and create a new key pair, called dev-keypair, and then we'll download it so that we're able to connect to our instance.

![EC2 Key Pair](/img/EC2_keypair.png)

Then we can *actually* launch our instance.  We're presented with a confirmation screen and we can click on the View Instances link which redirects us to the EC2 Instances view.  Our Instance is currently on it's way up.<br>

Before we go about connecting to our instance, let's see if the web server is actually serving up the default nginx page.  Snag the IPv4 Public IP and paste it into your browser.  If you see the default nginx page then we are good to go.

Voila, we've just stood up our first web server.  

Now you just need to repeat this process for `web-2`.

I know, this is not very useful having two EC2 instance that serve up a default nginx page when your route directly to their public IPs, but you need to walk before you can run so bear with me.

Also, worth noting, up until now we have been creating resources that don't incur any cost.  VPCs, Subnets, SGs, and NACLs are free - but instances can cost money to run.  Best to stop/terminate your instances when you're not actively working with them.

:thumbsup: