# Notes About AWS VPC

* Each region comes with a default VPC.
* The VPC will have one "public" subnet per availability zone within the region.
* Each subnet has its own routing table
* The public subnets are "public" because internet traffic (that's not going through the private network itself) is routed through an Internet Gateway (IGW). This is a setting (route tables + routes) that is specified in each subnet

* Routing traffic through an IGW means two things:
	1. By default, servers within the public subnet will get assigned a public IP address
	2. By default, servers within the public subnet will be able to reach the outside internet, and the outside internet will be able to reach the servers (via the public IP address).


#### The private IP addresses in general:
1. Class A: 10.0.0.0 — 10.255.255.255
2. Class B: 172.16.0.0 — 172.31.255.255 
3. Class C: 192.168.0.0 — 192.168.255.255


















## Resources:
* [VPC Basics - Small Series](https://cloudcasts.io/course/vpc-basics)
