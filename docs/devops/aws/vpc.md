# VPC

![](https://www.uturndata.com/wp-content/uploads/2021/02/full-vpc-traffic.png)

A VPC is a logically isolated section of a data center where you have complete control over your virtual networking environment, including the selection of IP address ranges, etc.

## NAT gateways

A NAT gateway is a Network Address Translation (NAT) service. You can use a NAT gateway so that instances in a private subnet can connect to services outside your VPC but external services cannot initiate a connection with those instances.

When you create a NAT gateway, you specify one of the following connectivity types:

- **Public** – (Default) Instances in private subnets can connect to the internet through a public NAT gateway, but cannot receive unsolicited inbound connections from the internet. You create a public NAT gateway in a public subnet and must associate an elastic IP address with the NAT gateway at creation. You route traffic from the NAT gateway to the internet gateway for the VPC. Alternatively, you can use a public NAT gateway to connect to other VPCs or your on-premises network. In this case, you route traffic from the NAT gateway through a transit gateway or a virtual private gateway.

- **Private** – Instances in private subnets can connect to other VPCs or your on-premises network through a private NAT gateway. You can route traffic from the NAT gateway through a transit gateway or a virtual private gateway. You cannot associate an elastic IP address with a private NAT gateway. You can attach an internet gateway to a VPC with a private NAT gateway, but if you route traffic from the private NAT gateway to the internet gateway, the internet gateway drops the traffic.

## Network Access Control List (NACL’s)

A network access control list (ACL) allows or denies specific inbound or outbound traffic at the subnet level.

| Security group                                                          | Network ACL                                                                                                                                      |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Operates at the instance level                                          | Operates at the subnet level                                                                                                                     |
| Applies to an instance only if it is associated with the instance       | Applies to all instances deployed in the associated subnet (providing an additional layer of defense if security group rules are too permissive) |
| Supports allow rules only                                               | Supports allow rules and deny rules                                                                                                              |
| Evaluates all rules before deciding whether to allow traffic            | Evaluates rules in order, starting with the lowest numbered rule, when deciding whether to allow traffic                                         |
| Stateful: Return traffic (outbound) is allowed, regardless of the rules | Stateless: Return traffic (outbound) must be explicitly allowed by the rules                                                                     |

![](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/security-diagram_updated.png)

## CIDR

Classless Inter-Domain Routing (CIDR) is an IP address allocation method that improves data routing efficiency on the internet.

An IP address has two parts:

- The network address is a series of numerical digits pointing to the network's unique identifier
- The host address is a series of numbers indicating the host or individual device identifier on the network

**Classful addresses**

- Class A: 8 network prefix bits (supported 16,777,214 hosts)
- Class B: 16 network prefix bits (supported 65,534 hosts)
- Class C: 24 network prefix bits (supported 254 hosts)

The classful arrangement was inefficient when allocating IP addresses and led to a waste of IP address spaces. For example, an organization with 300 devices couldn’t have used a Class C IP address, which only permitted 254 devices. So, the organization would’ve been forced to apply for a Class B IP address, which provided 65,534 unique host addresses. However, only 300 devices would’ve been connected, which would’ve left 65,234 unused IP address spaces.

CIDR addresses use variable length subnet masking (VLSM) to alter the ratio between the network and host address bits in an IP address. 

A VLSM sequence allows network administrators to break down an IP address space into subnets of various sizes. Each subnet can have a flexible host count and a limited number of IP addresses. A CIDR IP address appends a suffix value stating the number of network address prefix bits to a normal IP address.

For example, 192.0.2.0/24 is an IPv4 CIDR address where the first 24 bits, or 192.0.2, is the network address.

We can convert CIDR to IPv4 using this [page](https://www.ipaddressguide.com/cidr) 
