# Notes About AWS VPC

* Each region comes with a default VPC.
* The VPC will have one "public" subnet per availability zone within the region.
* Each subnet has its own routing table
* The public subnets are "public" because internet traffic (that's not going through the private network itself) is routed through an Internet Gateway (IGW). This is a setting (route tables + routes) that is specified in each subnet

* Routing traffic through an IGW means two things:
	1. By default, servers within the public subnet will get assigned a public IP address
	2. By default, servers within the public subnet will be able to reach the outside internet, and the outside internet will be able to reach the servers (via the public IP address).

* ### IPv4 Address Classes
	* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-12-03%2016-53-09.png" width="600" height="250">

* In AWS you can choose from these three: [Docs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-sizing-ipv4).
	1. 10.0.0.0/16 (or smaller)
	2. 172.31.0.0/16 (or smaller)
	3. 192.168.0.0/16 (or smaller)

* ### Reserved IP addresses in AWS subnets 
	* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-12-04%2009-25-27.png" width="600" height="250">



## Subnetting
* ### Number of hosts in different Network/Subnet sizes
	* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-12-04%2009-03-07.png" width="600" height="250">
	* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-12-04%2009-07-45.png" width="600" height="250">
	* usually in AWS they allocate to you /16 network by default

* ### The Ovelapping subnets
	* its the subnets whose range of addresses include some of the same addresses.
	* this can be done for example if you have network in the cloud and another one on prem.
	* you should not use all the IPs available with your subnet, because it will cause a problem if you want this network to communicate with another network because the overlapping

* use this [tool](https://www.davidc.net/sites/default/subnets/subnets.html) to calculate your subnet
* you can use the `terraform console` command and `cidrsubnets` method to create subnets
* or you can pick up from this Quick to use networks & subnets 
	* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-12-04%2009-47-11.png" width="600" height="250">









## Resources:
* [VPC Basics - Small Series](https://cloudcasts.io/course/vpc-basics).
* [AWS VPC: Subnets and Routing](https://cloudacademy.com/course/aws-virtual-private-cloud-subnets-and-routing/introduction-95/).
* [IP Subnetting](https://www.udemy.com/course/ip-subnetting/).
* [Cloud and TCP/IP Pre-Requisite Knowledge](https://www.dolfined.com/courses/cloud-and-tcp-ip-pre-requisite-knowledge)
