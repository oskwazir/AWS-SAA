## EC2

* Renting a virtual machine EC2
* Storing data on virtual drives EBS
* Distributing load across machines ELB
* Scaling services using an auto-scaling group ASG

### SSH
You have to `chmod 400` the `*.pem` file because by default it's not secure enough. So you're limiting access on your machine to the `*.pem` file.

### Security Groups

Control how traffic moves in and out of EC2 machines. Act like a firewall for EC2 instances

They regulate:

* Access to ports
* Authorised IP ranges
* Control of inbound network
* Control of outbound network

* All inbound traffic is blocked by default
* All outbound traffic is authorised by default
* Can be attached to multiple instances
* An instance can have multiple security groups
* Scoped to a region/VPC combination
* External to EC2
* Good to maintain one security group only for SSH access
* If app is not avaiable from EC2 with a time out, most likey an issue with security group
* If you get a connection refuesed error then the connection passed the security group and it's an  EC2 or app error.

You can reference security groups in inbound rules. So if you have Security Group A allowing Security Group B and C for inbound traffic, and EC2 instance that is attached to Security Group B can send traffic to the EC2 instance attached to Security Group A.

```
EC2_B <- SG_B
EC2_C <- SG_C
EC2_D <- SG_D

EC2_A <- SG_A[Inbound:SG_B, SG_C] :: EC2_A <- [EC2_B, EC2_C] :: EC2_A !<- EC2_D

```

### Private & Public IP

Two types of IP, IPv4 & IPv6. Public IP speak over internet. Private over internal network.

By default an EC2 instance will change its public IP if the instance is stopped + started again.
If you need a fixed IP, you will need an Elastic IP. Elastic IP is a public IPv4 address. 

* You can only have 5 Elastic IPs for your account. 
* Try to avoid using Elastic IP.
* Use random public IP and attach a domain to it.
* Or better yet use a Load Balance and don't use a Public IP for your EC2 instances.

** You will be charged for an Elastic IP if it is not associated with an EC2 instance**

### EC2 User Data

* You can bootstrap an instance using EC2 User Data script
* A bootstrap script runs once when the instance first starts
* Script runs as super user

The bootstrap script can be added when Launching an instance under Configure Instance Details -> Advanced Details -> User Data.