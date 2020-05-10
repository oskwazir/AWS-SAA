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

### EC2 Instance Launch Types

* On Demand Instances: short workload, predictable pricing
* Reserved (Minimum 1 year)
	* Reserved: When you know your workload time
	* Convertible Reserved Instance: Long workloads with flexible instances. Change instance types.
	* Scheduled Reserved Instances: Every Thursday 3pm - 6pm. Good for batch processing.
* Spot Instances: Short workloads for cheap but you can lose an instance
* Dedicated Instance: No customers share the hardware
* Dedicated Hosts: Book an entire **physical** server, control instance placement

### EC2 On Demand

* Pay for what you use (billing per second after first minute)
* Has highest cost with no upfront payment
* No commitment
* Recommended for short-term and un-interrupted workloads, where you can't predict how the app will behave 

### EC2 Reserved Instances

* Pay upfront for what you use with long term commitment
* Reservation period is 1 or 3 years
* **Reserved Instances**
	* Up to 75% discount compared to On-Demand
	* Reserve a specific instance type
	* Recommended for steady state usage like Databases
* **Convertible Reserved Instance**
	* Can change EC2 instance type
	* 54% discount
* **Scheduled Reserved Instances**
	* Launch window with time you reserve
	* When you need fraction of time for a day/week/month

### EC2 Spot Instances

* Up to 90% discount compared to On-Demand
* You can lose the instance if your max price is less than the current spot price
* Useful for workloads resilient to failure
	* Batch Jobs
	* Data Analysis
	* Image processing
	* Anything that can tolerate failure and resume later
	* Not good for databases or critical jobs
* Great combo: Reserved Instances for baseline & On-Demand & Spot for peak processing

### EC2 Dedicated Hosts
* Physical dedicated EC2 server for you to use
* Full control of EC2 placement
* Affinity between a host and instance
* Visibility into underlying sockets & physical cores of the hardware
* 3 year reservation period
* More expensive
* Useful if you have a Bring your own license model (BYOL)
* If you have strong regulatory & compliance requirements

### EC2 Dedicated Instances
* Instance running on hardware dedicated to you
* May share hardware with other instances in the same account
* No control over instance placement (can move hardware after Stop/Start)
* No Visibility into underlying sockets & physical cores of the hardware
* No affinity between a host and instance