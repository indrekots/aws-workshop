# AWS Workshop

![AWS workshop cover image](cover.jpg)

1. Create VPC (choose network CIDR)
2. Attach Internet Gateway to VPC to get access to the outside world and outside could access you
  - one IGW can be attached to a VPC
  - IGW cannot be deattached from a VPC when there are active AWS resources in the VPC (e.g. EC2 instances) 
  - default VPC already has an IGW
3. Create a route table
  - you cannot delete a route table if it has dependencies
  - default VPC has a _main_ route table
4. Create subnets
  - subnet resides in an availability zone
  - subnets must be associated with a subnet, if you don't do it explicitly, they're associated implicitly with the _main_ route table
  - **public subnet** is associated with a route table that has a route to an IGW
  - **private subnet** is associated with a route table that hasn't got a route to an IGW
  - private subnets can access the Internet when routing traffic through a NAT Gateway
5. Network Access Control Lists (NACL)
  - firewall on the subnet level
  - are stateless, return traffic must be allowed as well, if you have an inbound rule, you must specify an outbound rule as well
  - rules are evaluated in order, starting with the lowest rule number
6. Security Group
  - firewall on the instance level
  - rules are stateful, return traffic requests are always allowed
  - no deny rules
7. Elastic Load Balancing
  - health checks
  - application LB, HTTP, HTTPS
  - path based routing
8. Autoscaling
  - works with Cloudwatch metrics, cloudwatch alarm adds a new instance
9. Bastion host
  - an EC2 instance in a public subnet, is used as a _point of entry_ to access EC2 instances in private subnets
10. NAT Gateway
  - provides access to the Internet for EC2 instances in private subnets
  - prevents anybody form outside of the VPC from initiating a connection with EC2 instances that are associated with the NAT Gateway
  - NAT Gateway must be in a **public subnet**
  - when creating a NGW, you have to attach an elastic IP to it


## Steps

### 1. VPC

In AWS VPC dashboard, create a new VPC. Name it `<your-name>-vpc` so it's distinguishable from others.
Select IPv4 CIDR block, e.g. `10.10.0.0/16`.

### 2. Internet Gateway

In AWS VPC dashboard select the *Internet Gateway* menu option and create a new Internet Gateway.
Name it `<your-name>-igw`.
Attach it to your recently created VPC.

### 3. Subnets

In AWS VPC dashboard, select subnet tab and create 6 subnets.
Use a naming convention so that subnets are easily distinguishable (e.g. `<your-name>-app-1`).

* created subnets should be in your newly created VPC
* use your own scheme for allocating an IPv4 CIDR block for each subnet, e.g. `10.10.1.0/24` for `<your-name>-dmz-1` and `10.10.2.0/24` for `<your-name>-dmz-2`
* create 2 subnets for DMZ, place them in different availability zones
* create 2 subnets for applications, place them in different availability zones
* create 2 subnets for databases, place them in different availability zones
* a region can have several availability zones, place 3 subnets in `az-1` and the rest in `az-2`

### NAT Gateway

In AWS VPC dashboard, select *NAT Gateways* menu option and create a new NAT Gateway.

* place the NAT gateway into one of your DMZ subnets
* allocate a new elastic IP address for the NAT gateway

### Route tables

In AWS VPC dashboard, select *Route Tables*.
We're going to create two route tables.
The first one has a route to the Internet Gateway, the other one has a route the NAT Gateway.

Create a route table and name it `<your-name>-rt-igw`.
Place it in your VPC.
It has a _local_ route created for you.
For all other traffic (i.e. `0.0.0.0/0`), add a route to the internet gateway.

Then, create another route table and name it `<your-name>-rt-ngw`.
Place it in your VPC.
Edit its routes and add a route to the NAT gateway.

Finally, associate route tables with subnets.
Select `<your-name>-rt-igw` route table and edit its subnet associations.
Associate it with both of your DMZ subnets.
For `<your-name>-rt-ngw`, associate it with all other subnets.

### Network Access Control Lists

In AWS VPC dashboard, select *Network ACLs*.
Create three Network Access Control Lists, one for each layer in the infrastructure architecture.
Name them based on the subnet, e.g. `<your-name>-dmz-nacl`, `<your-name>-app-nacl` etc.
Place them in your VPC.
Associate NACLs with your subnets.
For example, `<your-name>-dmz-nacl` should be associated with `<your-name>-dmz-1` and `<your-name>-dmz-2`.

Inbound/outbound rules for NACLs are stateless.
Meaning that for an inbound rule, a matching outbound rule must be created.
Otherwise, traffic can only enter a subnet but can never exit it.

Edit inbound/outbound rules for the `dmz` NACL so that SSH access could be allowed.
Traffic to port 22 from all sources should be allowed.

// outbound rules for ssh access to app and db subnets

![List of inbound rules for NACL](inbound-nacl.png)

In outbound rules, all TCP traffic to ephemeral ports (1024-65535) should be allowed.

![List of outbound rules for NACL](outbound-nacl.png)

// network diagram needed

### Security Groups

In AWS VPC dashboard, select *Security Groups*.
Create three security groups.
The first one is going to be used for the bastion host.
Second and third groups are going to house app and DB servers respectively.
Name your security groups (e.g. `<your-name>-app-sg`), add a description and place them into your VPC.

Configure the security group that's responsible for the bastion host.
Edit its inbound rules to only allow access to port 22 (SSH) from all sources.
Compared to NACLs, security groups are stateful.
The traffic that's allowed to enter a security group is always allowed to leave it as well.

![List of security group rules for DMZ group](sg-rules.png)

For the app security group, allow SSH (port 22) from bastion host subnet and HTTP (port 80) traffic from all sources.
Additionally, since we wish to install software on application servers, let's allow them to connect to the outside world via HTTP/S so they could receive packages and updates.

// example image

### Bastion host

In AWS EC2 dashboard, launch a new instance for the bastion host.
Pick Amazon Linux AMI 2018.03.0 AMI and `t2.micro` as the instance type (it's free tier eligible).
Then click *Configure Instance Details*.
Place the instance in your VPC and select your `dmz-1` subnet.
Also, enable auto-assignment of public IP.
Leave all other options as is.
Click *next* until it's time to configure a security group.
Pick your `dmz-sc` for the security group.
Finally, review and launch the instance.

AWS will ask you to pick a key pair that's used to access the instance.
Crate a new key pair, add a name and then download it.
A `*.pem` file will be downloaded.
Once that's done, launch the instance.
It will take a bit of time for the instance to be ready.

Once ready, in EC2 dashboard, list your instances and connect to your bastion host by clicking *Connect*.
You'll see instructions on how to SSH into the bastion host using the `pem` file you downloaded previously.
If security groups and network access control lists have been configured correctly, you should be able to successfully establish an SSH session.

```
       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/
```

If the connection hangs, it could be that there's an issue with NACLs or security groups.

### Application servers

In AWS EC2 dashboard, create a autoscaling *launch configuration* that's used by the Auto Scaling Group to create new instnaces.
Use Amazon Linux AMI 2018.03.0 AMI and `t2.micro` as the instance type.
Click next, and configure name and some advanced details.
Set the name to `<your-name>-asg-lc`.
Copy the following to the *User Data* text field.

```
#!/bin/bash
yum update -y
yum install -y httpd
service httpd start
```

This script is executed when the instance starts.
It will install the Apache webserver.
Finally, do not assign a public IP address to any instances.

Click next until you can configure security groups.
Select `<your-name>-app-sg` and finish creating the launch configuration.
Use the previously created keypair when asked to provide one.

Next, let's create an auto scaling group based on the existing launch configuration.
Set the name (e.g. `<your-name>-asg`), pick your VPC, set the initial group size to 2 and then select subnets dedicated to application servers (i.e. `<your-name>-app-1` and `<your-name-app-2>`).
Click next to configure scaling policies.
Keep the group at its initial size for now.
Click *Review* and finish creating the auto scaling group.
It takes a bit of time until the desired instances are created.
Later we'll use load balancers to create access to the web servers.

Once the instances have been created, let's try to SSH into them via the bastion host.
The same keypair is used for all EC2 instances.
In order to access instances in your VPC via the bastion host, you must enable SSH key forwarding.

Start an ssh agent
```
ssh-agent bash
```

Add your key
```
ssh-add /path/to/your/keypair.pem
```

SSH into your bastion host

```
ssh -A ec2-user@<bastion-host-ip>
```

From there, SSH into one of the web servers.
You can find the host name and IP address from EC2 dasboard.
If you have trouble connecting to web servers, have a look at your NACL and security group settings.

Once SSH'd into a web server, let's modify the default page that's served.

// TODO: modify index.html
