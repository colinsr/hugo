---
title: "AWS ELB"
date: 2018-03-23T06:28:24-04:00
draft: true
tags: ["aws", "amazon web services", "ELB"]
categories: ["cloud"]
---

In this post we're going to make our currently implemented web server a bit more fault tolerant and highly available using AWS Elastic Load Balancing.  

Hopefully, you've been following along with my previous posts on [IAM Permissions](/post/AWS_IAM), [AWS VPC](/post/AWS_VPC), and [AWS EC2](/post/AWS_EC2) because we are going to build off of the resources created in those posts.

In the last post we created a couple of EC2 nginx web server instances that were routable via their public IPs.  Our plan for this post will be to put a public facing load balancer in front of those instances to distribute traffic between the two web servers.

The first thing we're going to do is go to the Load Balancing section of our EC2 dashboard and click Create Load Balancer.  We're presented with a few options for the type of load balancer we want to create.<br>
Since we're dealing with HTTP traffic this is an easy decision for us to choose the Application Load Balancer.  

The other options are Network Load Balancer, and Classic Load Balancer.

The Classic Load Balancer has been around for a long time now, and is not recommended for use in a VPC.  Prior to the launch of VPCs was the world of EC2-Classic.  Unless you created your AWS account prior to 2014 this will not be an option for you.  Classic Load Balancers operate at the layer 4 transport layer, using the TCP protocol.  Since we're using HTTP traffic and our account was created after 2014, we can skip right over the Classic Load Balancer.

The Network Load Balancer also operates at layer 4 of the OSI model.  This type of load balancer is used for ultra high performance workloads.  It has support for a static IP address and should be chosen when static IP is one of your requirements.  Where the Classic Load Balancer and the Application Load Balancer both offer SSL termination, the NLB does not.<br>
SSL termination can take the job of of decrypting your encrypted traffic using an X.509 certificate off your EC2 instances, giving you a boost in performance. 

Now that we've settled on the ALB let's get to setting it up.  Select ALB as the type and we're presented with some Basic Configuration options.  Give it a meaningful name, how about `public-web-alb`, make sure to choose internet-facing, and `ipv4` for the address type.<br>
Next we'll set up the listener.  Since we haven't setup HTTPS we'll be good just setting up the listener on `HTTP/80`.<br>
We should select our VPC from the dropdown we're provided, and then select both of the public subnets (`public-1` and `public-2`) we created in the  [VPC post](/post/AWS_VPC).  This is what will give us the fault tolerance and high availability the AWS cloud is able to provide.  Since we setup our subnets in different Availability Zones, if AWS should have big problems in one of the data centers in the `us-east-1` region, we'll have an entire second data center to fall back on.

Click next and we'll get a warning about security settings.  HTTP is definitely not a secure protocol and I would not recommend anyone use HTTP for **any** production workloads - but it will suit our needs.  And with that, we can click next again.

Here we can either choose an existing Security Group or create a new one, let's go ahead and setup a new Security Group called `Dev-VPC-public-LB`, and open up port 80 to the world.

![ELB Security Group](/img/ELB_SG.png)

We have a few additional rules to allow SSH and HTTPS on the SG we created for our EC2 instances to allow for packages to get installed and SSH access from our own public IP to those servers, and we want to follow the principle of least privilege whenever possible.

OK, now we need to configure the routing.  We need to setup a new target group, let's call it `web-targets` and choose HTTP on port 80, with a target type of `instance`.

The health checks here will be pretty straight forward since our default nginx page is served from the root `/`, we will tweak the advanced health check settings a bit here to give us faster feedback on healthy/unhealthy targets.  We'll enter 2 for healthy, unhealthy threshold and timeout and 5 for the interval.  These health checks are what alert the load balancer whether or not to serve traffic up to the target instance.  We're setting the thresholds the way we did so that we can get faster feedback if one of our servers to become unhealthy.

![ELB Routing Config](/img/ELB_routing_config.png)

With that out of the way let's set up the targets for our target group.  This step can be a little bit strange, actually.  In order to add our 2 running instances as targets first we need to select them both, and then click on the Add to registered button.  Once you see them pop up in the registered targets section above you'll be good to continue.

![ELB Targets](/img/ELB_targets.png)

Finally we can review our configuration, and make sure we didn't fat finger any of the settings.  If you set everything up correctly you should be looking at something like this:

![ELB Review](/img/ELB_review.png)

Go ahead, click on create... wait one moment and you should see the state of your ALB go from: `provisioning => active`.  Once the ALB is active you'll notice a DNS name down in the Description tab, this is what we'll be using to talk to our load balanced web servers.<br>
Copy that DNS name, and paste it into your browser and you should be greeted with your default nginx config.

Jump into the Target Groups section within Load Balancing, along the left hand gutter of the EC2 Dashboard, click on the Targets tab in the bottom and you should see 2 happy healthy instances sitting in your target group.

This is a little bit unfulfilling.  How are we supposed to know which web server is answering the response?

Let's SSH into both of our web servers and replace the default page with a custom index.html that identifies which server is responding to the request to the load balancer.  I'll walk you through connecting to the first instance, as well as updating the default index.html and you can rinse and repeat on web server 2.

First, back in the AWS Console on the EC2 dashboard go to instances, and select `web-1`, click on Actions and Connect.  At this point you should see something along the lines of this:

![Connect to EC2](/img/ELB_connect_to_instance.png)

Make sure to navigate to your downloads directory, or wherever you stashed your private key file, before executing the commands to initiate an SSH connection.

The first thing we'll want to do once we log in to our server is elevate our privileges to root by running the command: `sudo su -`.
Then we can run `echo "Hello from web-1" > /usr/share/nginx/html/index.html` which will overwrite the default nginx page with the string "Hello ...".<br>
At this point we can log off web-1 and you can log into web-2 and repeat the procedure to get both of our servers to let us know who they are.

![SSH session](/img/ELB_ssh_terminal.png)

If we start hitting our ALB again a few times we can see that the servers are sharing the load of the web traffic, and our ALB is doing it's job.  Also, we can sleep better at night knowing that if one of our servers goes down, or an entire AWS data center goes down - we still have a server out there handling our requests.

:thumbsup:

<!-- ## YOUTUBE LINK
{{< youtube ao8L-0nSYzg >}} 
-->