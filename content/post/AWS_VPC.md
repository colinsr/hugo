---
title: "AWS VPC"
date: 2018-03-19T06:55:51-04:00
tags: ["aws", "amazon web services", "VPC"]
categories: ["cloud"]
---

In this post we're going to dissect the make up of an AWS VPC.  
VPC's make up the networking component of Amazon Web Services.
They're composed of several distinct pieces, we'll go over each one independently and talk about their purpose and some of the ins and outs of them.

* [VPC](#vpc)
* [Subnets](#subnet)
* [Internet Gateway](#igw)
* [Route Tables](#routetable)
* [Network Access Control Lists](#nacl)
* [Security Groups](#sgs)
* [Virtual Private Gatway](#vpg)
* [VPC Endpoints](#endpoints)

## VPC <a id="vpc"></a>
VPCs are the heart of AWS networking.<br>
They enable you to create fully customized, fit to order virtual networks in the AWS cloud.<br>
While the power of VPCs in AWS is awesome, there are also a lot of pieces that fit together to form a VPC. We'll go over each component separately in this post and try to make sense of all of this VPC talk.<br>
Bear in mind that a VPC can only live in one AWS Region, but it can span multiple Availability Zones.<br>
When the rubber meets the road, there are only a few things that a VPC needs in order to be created.<br>
First, it needs a name. Try to be descriptive here and give your VPC a meaningful, self documenting name.  There's nothing worse than seeing a VPC called `vpc-1695` or some other garbage.<br>
Next up it, needs a CIDR block.  CIDR stands for Classless Inter Domain Routing.<br>
What that means in layman's terms is the VPC needs a range of private IP addresses which it has at it's disposal.  Take notice that I said "private" IP range.<br>
Back in the day, the [Internet Engineering Task Force (IETF)](https://en.wikipedia.org/wiki/Internet_Engineering_Task_Force) realized that we were going to run out of public IP addresses, so they pulled several IP ranges and marked them as reserved for private networks.  These private ranges fall into the following:

* 10.0.0.0 - 10.255.255.255
* 172.16.0.0 - 172.31.255.255
* 192.168.0.0 - 192.168.255.255

This means that if you are on the open internet and try to hit one of these private IP addresses you are pretty much S.O.L. as they are non-routable.<br>
There are a few limitations when it comes to the ranges you can provide a VPC in AWS.  The netmask must be between `/16` and `/28`. Pre CIDR notation used a Classful scheme where there were Class A, B, C, D, and E.  I'm not going in to all the details of subnetting here, but we'll cover enough to get our AWS VPC up and running.<br>
During setup of a VPC you can also choose whether or not to use IPv6, which is the follow up solution our friends at IETF came up with in regards to running out of IP addresses.  We'll have plenty of addresses to work with in our network, and I'd certainly rather be using `10.0.1.157/32` as an IP than `2001:db8:85a3:8d3:1319:8a2e:370:7348`.<br>
The final input required for a VPC is the tenancy level.  This is just talking about what kind of hardware you'd like your VPC to run your EC2 instances on.  Dedicated Tenancy means running on an isolated server, where you are the only person using that server.  It's pretty expensive to run on Dedicated Tenancy, so we'll stick with Default Tenancy unless we have a good reason not to.<br>
Default Tenancy means that our applications are not the only applications running on a given EC2 instance - there could be any number of other apps running alongside yours.  Fear not, there is a hypervisor running on the host which handles the virtualization/isolation.<br>
This setting will override the EC2 Tenancy setting so proceed with caution if you choose Dedicated as all instances launched in your VPC will run in this mode.

Let's go through the console, and choose VPC from the services.<br>
Then we'll navigate to `Your VPCs` and choose `Create VPC`.  That will bring up a modal where we can enter some values.

![Create VPC](/img/VPC_create.png)

We'll choose a name `Dev-VPC`, a CIDR block `10.0.0.0/16`, and a tenancy type `Default` and then click create.  The `/16` netmask will give our VPC access to over 65,000 private IP addresses, so I'd say we're all good there.


## Subnets <a id="subnet"></a>
Subnets have a long list of uses.<br>
They are primarily used to break up your VPC's address range into smaller chunks.  Amazon has multiple availability zones within a region and we'd like to distribute our servers across AZ's in order to maximize high availability and fault tolerance.  A subnet cannot span AZ's so we're going to need a few.<br>
In our `10.0.0.0/16` VPC example, we'd want to set up at least 2 subnets, one for each AZ.  The second would get created in a different AZ than the first, let's say `us-east-2a` and `us-east-2b` (yay Ohio data center).

When you're create your first subnet, go into the Subnets tab under the VPC service, and click on Create Subnet.  We'll provide a name `public-1`, the VPC to associate the subnet with `Dev-VPC`, the AZ to place the subnet, and the IPv4 CIDR block for the subnet.  We'll use a `10.0.1.0/24` block which will give us 256 addresses to play with from: `10.0.1.0` up to `10.0.1.255`, minus the handful held back by AWS.

AWS reserves the following addresses in **each** subnet inside a VPC - in our `10.0.1.0/16` VPC those will be:

* `10.0.1.0/32`   => `Network Address`
* `10.0.1.1/32`   => `VPC router`
* `10.0.1.2/32`   => `DNS server`
* `10.0.1.3/32`   => `Reserved for "future use"`
* `10.0.1.255/32` => `Network Broadcast Address`

This is what that looks like.

![Create Subnet](/img/VPC_subnet.png)

Now repeat the same process - except that it's in `us-east-2b`, the IPv4 CIDR block is `10.0.2.0/24` and the name is `public-2`. Nice! 

## Internet Gateways <a id="igw"></a>
We created two "public" subnets, but in reality they are not connected to the internet and are public in name only.<br>
In order to make our subnets actually public we'll need to create an Internet Gateway.  These are used for all traffic to/from your network and the internet.<br>
You can only have one IGW attached to a VPC and we have none so let's get to getting.

Go into the `Internet Gateways` view and select Create Internet Gateway.<br>
Choose a name - `Dev-VPC-IGW` and click Yes, Create.

![Create Internet Gateway](/img/VPC_igw.png)

OK, great we have an Internet Gateway now let's get it connected to our VPC.<br>
Click on the Attach to VPC button and choose our `Dev-VPC`.

## Route Tables <a id="routetable"></a>
Which brings us to Route Tables.  We now have a VPC, a couple subnets, added an Internet Gateway and still we need to give our subnets a route to the internet.<br>
A Route Table in AWS is pretty much the same as a router in an on premises network.  Only 1 Route Table can be attached to a Subnet, but many subnets can be attached to the same route table.<br>
When a VPC is created, AWS creates a default Route Table along with it.  Any new Subnets you create will automatically be attached to the default Route Table.<br>
This default Route Table only has one route defined - our local network (`10.0.0.0/16`).<br>
These Route Tables take 2 inputs, the Destination and the Target.  Looking at our existing Default Route Table, we can see that we have a Destination which is the same as our CIDR block for our VPC and a target of local.  No mention here of our Internet Gateway.<br>
In order to get our Subnets to be actually public, lets get the Internet Gateway attached to a new Route Table.<br>
Going in to our VPC service and Route Tables section we want to click on Create Route Table.  Once the modal pops up we are going to give a clear name describing what this Route Table is for: `Dev-VPC-Public` sounds about right.  Make sure you choose to create this new Route Table in the correct VPC in this step.

![Create Route Table](/img/VPC_route_table.png)

We're already half way home.  Now we just need to get a route added to pass non-local traffic off to the Internet Gateway.<br>
Back in the AWS Console, select your new Route Table, click on the Routes tab in the bottom dock and select Edit and finally Add another Route.<br>
Our next route is going to cover all non-local traffic.  This is defined with `Destination: 0.0.0.0/0`, meaning when a computer on our VPC tries to access some IP address that does not reside in our VPC, our network is in the `10.0.0.0/16` IP address range, the traffic will be routed to the Internet Gateway.<br>
So now we just need to add the `Target: Our IGW` - which easily identifiable because of proper naming.

![Create Public Route](/img/VPC_public_route.png)

Don't forget to click the Save buton, and we can move on.  We finally have proper public subnets.

## Network Access Control Lists (NACLs) <a id="nacl"></a>
Ruh roh, now that our public Subnets are actually connected to the internet we're going to have to start to think about locking them down so that we don't expose ourselves to malicious attacks.<br>
AWS has multiple layers of securing your network/data, the first layer we'll talk about is the Network Access Control List.<br>
NACLs control traffic at the Subnet level.<br>
A Subnet can only have one NACL attached to it at any given time, but once again, you can re-use NACLs on multiple Subnets.<br>
NACLs are stateless, meaning that if you have an explicit Inbound ALLOW rule, **you must** also have a corresponding explicit Outbound ALLOW rule in your configuration.<br>
These Rules are executed in ascending order, lower numbers are evaluated first, and once a Rule is hit no other Rules are evaluated.<br>
As per usual, AWS has been kind enough to create a default NACL for our VPC which has an ALL Traffic, ALL Protocol, ALL Port, allow Inbound and Outbound rule in place for traffic to/from all IPs.<br>
This is good for some folks, but we care about security so we'll be creating a new custom NACL.<br>
Back in the AWS Console, in the VPC section, under the Security section we'll choose Network ACLs.<br>
Once inside, we'll choose Create Network ACL.  Again we want to choose a meaningful name and make sure that we pick the correct VPC.

![Create Network ACL](/img/VPC_NACL.png)

With the easy part out of the way, we can drill in to the Inbound/Outbound rules.  First make sure you've selected our newly created NACL, then select Inbound Rules.<br>
It's important to notice here that we have an explicit DENY ALL rule currently.  We are going to have to specify exactly what traffic we want to allow into our public subnets.<br>
Select Edit, and Add another rule.  Remember I said these rules are executed in ascending order?  We need to give the rule a priority - let's start at 100, Type - SSH (port 22 here), Protocol TCP, and Source - we'll want to use our public IP to allow SSH access to our yet to be created instances.

Protip: if you have curl installed you can use `curl ipinfo.io` to pull your public IP, otherwise [Whats my ip](https://whatsmyip.com/).

Our NACL wants your IP in CIDR notation, meaning that we need to toss a `/32` on the end of our IP address to clarify that we only want to use our specific IP, not a range of IPs.<br>
While we're at it let's add Rule 110 to open up port 80 to the world, rule 120 to open up HTTPS to the world, and 130 to allow ephemeral traffic back in - this is for installing packages from our instances as the traffic doesn't come back in over port 443.

![Network ACL Inbound](/img/VPC_NACL_inbound.png)

Since NACLs are stateless, let's remember to update the outbound rules to allow the outbound traffic.  Also, keep in mind that outbound traffic doesn't always travel over the same port as inbound, it will also travel over ephemeral ports from 1024-65525.

![Network ACL Outbound](/img/VPC_NACL_outbound.png)

Finally, we just need to associate this new NACL with our public Subnets to complete the process.<br>
On the Subnet Associations tab, click edit and select both of our public subnets, then save.

Congratulations, we've just allowed SSH traffic from our own IP and HTTP traffic from anyone.

## Security Groups (SGs) <a id="sgs"></a>
Next up in our layered approach to VPC security are Security Groups.<br>
SGs are applied at the NIC (network interface card) level.<br>
It's important to note that Security Groups are stateful, meaning that if you have add an inbound rule the corresponding outbound traffic will automatically be allowed.<br>
Let's create a SG for the instances we plan to launch in our public Subnets.<br>
Again, from the AWS Console we navigate to the Security section, then Security Groups and click Create Security Group.<br>
Here we're prompted for a name, group name, description and the target VPC.<br>
`Dev-VPC-public-SG` should clearly identify what we're creating, so we'll use that for Name and Group name.<br>
Give a brief description: `Public Security Group`, and choose our Dev-VPC.

![Security Group](/img/VPC_Security_Group.png)

Next we'll add a couple Inbound Rules.

![Security Group Inbound](/img/VPC_SG_Inbound.png)

And we're all set, now we just need to spin up some instances to use this Security Group and we should have traffic flowing just as we'd like.<br>
**Remember that best practice is to only allow traffic that is required!**


## Virtual Private Gateway <a id="vpg"></a>
If you have a local on premises network then you're going to want to get a Site to Site IPsec VPN tunnel set up between your local network and you AWS VPC.<br>
We are not going to configure this here, but you should at least be aware of the moving parts involved.<br>
These parts are:

* Customer Gateway
* VPN Connection
* Virtual Private Gateway

In order to configure a Site to Site VPN connectivity we'll need to add a Customer Gateway which is the configuration of your local on prem network.  This gets configured in your AWS VPC under the VPN Connections section of our VPC service in the AWS Console.<br>
We'll also need to set up a Virtual Private Gateway, which is similar-ish to the Internet Gateway as it acts as the connector on the VPC side of the VPN connection.<br>
This is a bit hard to work through since most of us probably don't have a networking lab setup at home.<br>
Although I'm thinking of doing a follow up post in the future on connecting an Azure VNET to an AWS VPC.

## VPC Endpoints <a id="endpoints"></a>
VPC Endpoints... ? What the heck are these?<br>
Well, many of the services AWS offers are web based, meaning that in order to hit an S3 bucket we would need to make a request over the open internet.<br>
What if we had sensitive data that we'd prefer to keep _inside_ our network, you ask?<br>
This is where VPC Endpoints come into play.  VPC Endpoints enable you to keep your requests to any of AWS' long list Endpoint supported services internal.<br>
Some prime examples would be DynamoDB, S3, KMS, Kinesis, and EC2 SSM.<br>
We'll definitely be going into examples of VPC endpoints in a later post.

## Summary
This is by far not an exhaustive list of all the features and capabilities of the AWS VPC, but this post is turning into a full day experience as is and what we've covered so far should be enough to get a very solid foundation of what VPCs are and what problem they accomplish.  