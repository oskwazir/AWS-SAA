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

## Deep Dive into EC2 Spot Instance Requests

* Up to 90% discount compared to on-demand
* Spot price varies among AZ in an availability zone
* Define a **max spot price** and get the instance while **current spot price < your max bid**
* Hourly spot prices varies based on offer and capacity
* If **current spot price > max bid** you can choose to stop OR terminate your instance in a 2 minute grace period.

### Strategy: Spot Block

* *block* a spot instance for a specific time frame without interruptions
* In rare instances the instance may be reclaimed.

### Hot to terminate a spot instance

A spot requests contains the following
1. Maximum price,
1. Desired number of instances
1. Launch specification
1. Request type: one-time | persistent
1. Valid from - Valid until

![Spot Instance Creation](../images/spot_lifecycle.png)

You can only cancel Spot Instance requests that are **open**, **active** or **disabled**
![Spot Instance Request States](../images/spot_request_states.png)

**Cancelling a Spot Request does not terminate instances**

You must first cancel the Spot Requests and then terminate the Spot instances

## Spot Fleets

* Spot Fleets = set of Spot Instances + (optional) On-Demand Instances
* The Spot Fleet will try to meet the target capacity with price constraints
	* Define possible launch pools e.g instance type + OS + AZ
	* Can have multiple launch pools for fleet to choose
	* Spot fleet stops launching instances when reaching capacity or max cost
* Strategies to allocate Spot Instances
	* lowest price: from the pool with the lowest price (cost optimization, short workload)
	* diversified: distributed across all pools (great for availability, long workloads)
	** capacity optimized: pool with optimal capacity for number of instances

**Spot Fleet allows us to automatically request Spot Instances with the lowest price**

## EC2 Instance Types Deep Dive

* R: RAM - Memory intense
* C: CPU - Compute
* M: Medium - General/Web App
* I: IO - Instance Storage/Database
* G: GPU - Video Rendering/ML
* T2/T3 Burstable: Burstable (up to a capacity)
* T2/T3 Unlimited: Unlimited Burst

[ec2instances.info](https://www.ec2instances.info)

### Burstable Instance (T2/T3)

* Overall CPU has okay performance
* During a spike in load, you can burst for higher CPU usage
* Bursting consumes burst credits
* If credits are gone, CPU performance drops
* Over time you acrue credits when not bursting
* If you constantly run out of credits, move to another instance type
* You can see credit usage in CloudWatch

[Earning CPU credits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-credits-baseline-concepts.html#earning-CPU-credits)

### Unlimited (T2/T3)

* You can have unlimited burst credit balance
* You pay more if you go over your credit balance but you don't lose performance
* Careful with costs since this is relatively newish (2017)

[T2 Unlimited – Going Beyond the Burst with High Performance](https://aws.amazon.com/blogs/aws/new-t2-unlimited-going-beyond-the-burst-with-high-performance/)

> The hourly T2 instance price covers all interim spikes in usage if the average CPU utilization is lower than the baseline over a 24-hour window. There’s a small hourly charge if the instance runs at higher CPU utilization for a prolonged period of time. For example, if you run a t2.micro instance at an average of 15% utilization (5% above the baseline) for 24 hours you will be charged an additional 6 cents (5 cents per vCPU-hour * 1 vCPU * 5% * 24 hours).

> T2 Unlimited instances have the ability to borrow an entire day’s worth of future credits, allowing them to perform additional bursting.

## EC2 AMIs

Creating a custom AMI allows you to 
* pre-install packages & apps,
* optimize boot time,
* load enterprise software,
* add Active Directory
* use third-party AMIs 

AMIs are region specific

### Public AMIs

* You can rent an AMI by the hour

** Do not use AMIs you do not trust**

### AMI Storage

* AMIs live in S3
* By default AMIs are private, locked for account & region
* You can make AMIs public and share them with AWS accounts or sell them on the marketplace

### AMI Pricing

* Pay for S3 storage
* Cheap to store AMIs

### Copying AMIs

**Important for exam: Read about rules for copying AMIs**
[Copying AMIs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html)

## Placement Groups

When you need control over the EC2 Instance placement strategy

When creating a placement group you specify one of the following

1. Cluster - Cluster instances into low-latency group in a single AZ
1. Spread - Spread instances across underlying hardware (max 7 instances per group per AZ) - critical applications
1. Partition - Spreads instances across many different partitions (which rely on different set of racks) within an AZ. Scales to 100s of EC2 instances per group (Kafka, Cassandra, Hadoop)

### Cluster Placement Groups

Same Rack & AZ. 

> Cluster placement groups are recommended for applications that benefit from low network latency, high network throughput, or both. They are also recommended when the majority of the network traffic is between the instances in the group.

* Pros: Great network (throughput limit of up to 10 Gbps for TCP/IP traffic
* Cons: Rack failure all instances fail together.

### Partition Placement Groups

* UP to 7 partitions per AZ
* Up to 100s of EC2 instances
* Instances in a partition do not share racks with instances in another partitions
* A partition failure affects many EC2 instances but not other partitions
* EC2 instances get access to partitions metadata

> Partition placement groups help reduce the likelihood of correlated hardware failures for your application. When using partition placement groups, Amazon EC2 divides each group into logical segments called partitions. Amazon EC2 ensures that each partition within a placement group has its own set of racks. Each rack has its own network and power source. No two partitions within a placement group share the same racks, allowing you to isolate the impact of hardware failure within your application. 

> Partition placement groups can be used to deploy large distributed and replicated workloads, such as HDFS, HBase, and Cassandra, across distinct racks

>  ...partition placement groups offer visibility into the partitions — you can see which instances are in which partitions. You can share this information with topology-aware applications, such as HDFS, HBase, and Cassandra. These applications use this information to make intelligent data replication decisions for increasing data availability and durability. 

> Partition placement groups help reduce the likelihood of correlated hardware failures for your application. When using partition placement groups, Amazon EC2 divides each group into logical segments called partitions. Amazon EC2 ensures that each partition within a placement group has its own set of racks. Each rack has its own network and power source. No two partitions within a placement group share the same racks, allowing you to isolate the impact of hardware failure within your application. 

### Spread Placement Groups

* Pros: Can span AZ. Reduced risk of failure. EC2 instances on different hardware
* Cons: Limited to 7 instances per AZ per placement group


> A spread placement group is a group of instances that are each placed on distinct racks, with each rack having its own network and power source. 

> Spread placement groups are recommended for applications that have a small number of critical instances that should be kept separate from each other. Launching instances in a spread placement group reduces the risk of simultaneous failures that might occur when instances share the same racks. Spread placement groups provide access to distinct racks, and are therefore suitable for mixing instance types or launching instances over time. 