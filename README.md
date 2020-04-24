# AWS Workshop

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
8. Autoscaling
  - works with Cloudwatch metrics, cloudwatch alarm adds a new instance
9. Bastion host
  - an EC2 instance in a public subnet, is used as a _point of entry_ to access EC2 instances in private subnets
10. NAT Gateway
  - provides access to the Internet for EC2 instances in private subnets
  - prevents anybody form outside of the VPC from initiating a connection with EC2 instances that are associated with the NAT Gateway
  - NAT Gateway must be in a **public subnet**
  - when creating a NGW, you have to attach an elastic IP to it
