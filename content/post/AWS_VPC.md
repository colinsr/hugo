---
title: "AWS VPC"
date: 2018-03-19T06:55:51-04:00
draft: true
tags: ["aws", "amazon web services", "VPC"]
categories: ["cloud"]
---

In this post we're going to disect the make up of an AWS VPC.  
VPC's make up the networking component of Amazon Web Services.
They're composed of several distinct pieces, we we'll go over each one independently and talk about their purpose and some of the ins and outs of them.

* [VPC](#vpc)
* [Subnets](#subnet)
* [Internet Gateway](#igw)
* [Route Tables](#routetable)
* [Network Access Control Lists](#nacl)
* [Security Groups](#sgs)
* [link URL](#7)

## VPC <a id="vpc"></a>
VPCs are the heart of AWS networking.<br>
They enable you to create fully customized fit to order virtual networks in the AWS cloud.<br>
While the power of VPCs in AWS is awesome, there are also a lot of pieces that fit together to form a VPC. We'll go over each component separately in this post and try to make sense of all of this VPC talk.<br>
When the rubber meets the road, there are only a few things that a VPC needs in order to be created.<br>
First, it needs a name. Try to be descriptive here and give your VPC a meaningful, self documenting name.  There's nothing worse than seeing a VPC called `vpc-1695` or some other garbage.<br>
Next up it needs a CIDR block, CIDR stands for Classless Inter Domain Routing. What that means is basically the range of private IP addresses which the VPC has at it's disposal.  Take notice that I said "private" IP range.<br>
Back in the day, the [Internet Engineering Task Force (IETF)](https://en.wikipedia.org/wiki/Internet_Engineering_Task_Force) realized that we were going to run out of public IP addresses, so they pulled several IP ranges and marked them as reserved for private networks.  These private ranges fall into the following:

* 10.0.0.0 - 10.255.255.255
* 172.16.0.0 - 172.31.255.255
* 192.168.0.0 - 192.168.255.255

This means that if are on the open internet and you try to hit one of these private IP addresses you are pretty much S.O.L. as they are non-routable.<br>
There are a few limitations when it comes to the ranges you can provide a VPC in AWS.  The netmask must be between `/16` and `/28`. Pre CIDR notation used a Classful scheme where there were Class A, B, C, D, and E.  I'm not going in to all the details of subnetting, but we'll cover enough to get our AWS VPC up and running.<br>
During setup of a VPC you can also choose whether or not to use IPv6, which is the follow up solution our friends at IETF came up with in regards to running out of IP addresses.  We'll have plenty of addresses to work with in our network, and I'd certainly rather be using `10.0.1.157/32` as an IP than `2001:db8:85a3:8d3:1319:8a2e:370:7348`.<br>
The final input required for a VPC is the tenancy level.  This is just talking about what kind of hardware you'd like your VPC to run your EC2 instances on.  Dedicated Tenancy means running on an isolated server, where you are the only person using that server.  It's pretty expensive to run on Dedicated Tenancy, so we'll stick with Default Tenancy unless we have a good reason not to.  Default Tenancy means that our applications are not the only applications running on a given EC2 instance - there could be any number of other apps running alongside yours.  Fear not, there is a hypervisor running on the host which handles the virtualization/isolation.<br>
This setting will override the EC2 Tenancy setting so proceed with caution if you choose Dedicated.

Let's go through the console, and choose VPC from the services.<br>
Then we'll navigate to `Your VPCs` and choose `Create VPC`.  That will bring up a modal where we can enter some values.

![Create VPC](/img/VPC_create.png)

We'll choose a name, a CIDR block, and a tenancy type and then click create.  The `/16` netmask will give our VPC access to over 65,000 private IP addresses, so I'd say we're all good there.

One small note, AWS reserves a handful of addresses inside your CIDR block.

* `10.0.0.0/32`   => `Network Address`
* `10.0.0.1/32`   => `VPC router`
* `10.0.0.2/32`   => `DNS server`
* `10.0.0.3/32`   => `Reserved for "future use"`
* `10.0.0.255/32` => `Network Broadcast Address`

## Subnets <a id="subnet"></a>
[2] We're at Subnets


## Internet Gateways <a id="igw"></a>
[3] We're at IGWs


## Route Tables <a id="routetable"></a>
[4] We're at Route Tables


## Network Access Control Lists (NACLs) <a id="nacl"></a>
[5] We're at NACLs


## Security Groups (SGs) <a id="sgs"></a>
[6] We're at Security Groups


## LINK 7 <a id="7"></a>
[7] WE're at link 7?

