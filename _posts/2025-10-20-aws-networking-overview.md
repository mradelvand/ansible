---
title: "AWS Networking Overview"
date: 2025-10-20
categories: [AWS Networking]
tags: [VPC, Subnets]
---


![VPC excalidraw](https://github.com/user-attachments/assets/f1f3db9e-2c3f-42d3-8c67-3012f15b9c1f)

# AWS Networking - Big Picture 

## User interaction with app
User looks for example.com, 
**Route 53** resolves domain name to an IP address of LB located in public subnet
The traffic enters VPC through Internet Gateway.

#### Securing resources - private subnet 
To isolate resources from direct Internet, better to place them in private subnet
since Apps often require to send/receive outbound traffic, this is where NAT gateway
comes in. NAT gateway translates private IPs of EC2 to its own public IP.

## Accessing AWS services

#####  Access via NAT gateway
instances in private subnet access S3 or Dynamo DB over Internet through NAT gateway
which cause data transfer cost and public internet connectivity.

#####  VPC endpoint for private access

#####  endpoint gateway used for S3 and Dynamo DB
directs traffic over AWS private network to
these services. Unlike NAT gateway, having no additional cost for data transfer.

##### Interface gateway used for other AWS services 
it is a network interface with a private IP in your subnet powered by AWS private link.

## Connecting Multiple VPC

#####  VPC peering connection
is direct one to one connection between two VPCs.


##### Transit gateway
simplifies connectivity which routes traffic between VPCs,
On-Prem networks and even other AWS account.

## Hybrid Network

##### VPN or AWS Direct Connect used to link on-prem to VPC

##### Setup IPSec VPN tunnel
traffic goes over Internet though.

## Cloudfront 
Increase of users cause increase of cost and latency in case of using S3, ALB
Solution; ditching ALB and using cloud front; 
S3 can serve directly to users through cloudfront 
