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