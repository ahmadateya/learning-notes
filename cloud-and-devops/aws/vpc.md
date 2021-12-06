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


## Internet Gateways and NAT Gateways
* you can either have an IGW or NAT gateways
* ### [IGWs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
	* IGWs allows public network access (which makes any subnet using the route table that has an IGW a "Public Subnet").
	* The default VPC has one route table that is used across all subnets created within it. This Route Table has an IGW assigned to call traffic headed to 0.0.0.0/0 (basically if some outbound network traffic in the server goes anywhere but the private network, it will get routed through the IGW, allowing public internet access).
	* 
* ### [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
	* you can use it make your servers/resources talk to the outsite internet without making the internet talk to them
	* #### Network Address Translation (NAT)
		* is a process in which one or more local IP address is translated into one or more Global IP address and vice versa in order to provide Internet access to the local hosts.
		* is a way to map local private addresses to a public one before transferring the information. In other words, outbound traffic from a server in a private subnet is routed through the NAT Gateway and then to the outside internet. This allows the resource (server) to talk to the outside internet without allowing the outside internet to reach the resource (server) directly.
	
	* #### Port Address Translation (NAT)
		* Private IP addresses are translated into the public IP address via Port numbers. PAT also uses IPv4 address but with port number.
	
	* #### in AWS
		* The default VPC has no NAT Gateways created, as the default VPC is optimized towards allowing you to quickly spin up a server and have it available on the public internet.
		* NAT Gateways allows private-network-only servers access to the internet (but they can't be reached from the public internet). 
		* In additional to standard bandwidth charges, there are additional charges for NAT Gateways:
			1. There is an hourly charge for NAT Gateways (for example in us-east-2, an hour charge of $0.45/hr - to calculate the monthly charge: $0.045*730 = $32.85/mo)
			2. There is a charge per gigabyte transferred through a NAT Gateway (for example in us-east-2, a charge of $0.045/GB). This is in addition to regular bandwidth charges!

### Private vs Public Subnets
1. The server in the public subnet gets assigned a public IP address. The public internet can reach it, and it can reach the public internet.
2. The server in the private subnet does not get a public IP and is not accessible from the outside internet (but it can reach the internet through the NAT Gateway).
*  To access the private network
	* we can SSH into the "public" server, which then allows us to SSH into the "private" server - we use the public server as a bastion (jump) host to gain access to the private network.

## Resources:
* [VPC Basics - Small Series](https://cloudcasts.io/course/vpc-basics).
* [AWS VPC: Subnets and Routing](https://cloudacademy.com/course/aws-virtual-private-cloud-subnets-and-routing/introduction-95/).
* [IP Subnetting](https://www.udemy.com/course/ip-subnetting/).
* [Cloud and TCP/IP Pre-Requisite Knowledge](https://www.dolfined.com/courses/cloud-and-tcp-ip-pre-requisite-knowledge)
