## EC2

* Renting a virtual machine EC2
* Storing data on virtual drives EBS
* Distributing load across machines ELB
* Scaling services using an auto-scaling group ASG

### SSH
You have to `chmod 400` the `*.pem` file because by default it's not secure enough. So you're limiting access on your machine to the `*.pem` file.

### Security Groups

Control how traffic moves in and out of EC2 machines.