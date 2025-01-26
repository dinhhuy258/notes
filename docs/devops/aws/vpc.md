# Virtual Private Cloud (VPC)

A VPC is a logically isolated section of a data center where you have complete control over your virtual networking environment, including the selection of IP address ranges, etc.

## Key components of VPC

- **Subnets**: Within a VPC, subnets are like smaller neighborhoods, dividing the overall IP address range into manageable segments. Each subnet is associated with a specific Availability Zone (AZ), which ensures redundancy and fault tolerance. For example, you might have one subnet for web servers and another for databases, each located in different AZs, to mitigate the impact of failures.
- **Route tables**: Think of route tables as traffic directors within the VPC. They determine how network traffic flows between subnets and external networks like the internet. For instance, a route table might specify that traffic destined for the internet should be directed through the internet gateway, while internal traffic stays within the VPC.
- **Internet gateways**: Internet gateways are the connection points between the VPC and the wider internet. They allow instances within the VPC to communicate with resources outside the VPC, such as websites, APIs, and external databases.
- **Elastic IP addresses (EIP)**: EIPs are like reserved parking spots for instances within the VPC. They provide a static IP address that can be associated with an instance, ensuring that the instance maintains the same public-facing IP address even if it’s stopped and restarted.

## Default VPC

By default, when you create an AWS account, AWS creates a default VPC for you in each AWS Region. The default VPC comes pre-configured with a set of default subnets, route tables, internet gateways, and network access control lists (ACLs). This simplifies the process of launching instances and other resources without needing to create a custom VPC. The default VPC comes pre-configured with the following settings:

- **VPC configuration**: A VPC with a size `/16` IPv4 CIDR block (`172.31.0.0/16`), providing up to `65,536` private IPv4 addresses.
- **Default subnets**: A size `/20` default subnet in each Availability Zone, providing up to `4,096` addresses per subnet. Some addresses are reserved for AWS use.
- **Internet gateway**: An internet gateway connected to the default VPC, allowing instances to communicate with the internet.
- **Route configuration**: A route in the main route table that points all traffic (`0.0.0.0/0`) to the internet gateway.
- **Security group and network ACL**: A default security group and network access control list (ACL) associated with the default VPC.
- **DHCP options set**: The default DHCP options set for your AWS account is associated with your default VPC.

## Subnets

Subnets are vital components within VPC, allowing us to deploy our resources carefully and manage the network access effectively.

### Types of Subnets

- **Public subnet**: This subnet is connected to an internet gateway, allowing resources within it to access the public internet.
- **Private subnet**: The private subnet doesn’t have a direct route to the public internet. It requires a NAT device to allow its resources access to the public internet.
- **VPN-only subnet**: This subnet routes to a Site-to-Site VPN using a virtual private gateway and does not have a direct route to the public internet.
- **Isolated subnet**: Subnets of this type have no routes leading to destinations outside their VPC. Resources within an isolated subnet can solely communicate with other resources within the same VPC.

![imgur.png](https://i.imgur.com/j2lDMYY.png)

## Classless Inter-Domain Routing (CIDR)

An IP address has two parts:

- The network address is a series of numerical digits pointing to the network's unique identifier
- The host address is a series of numbers indicating the host or individual device identifier on the network

**Classful addresses**

- Class A: 8 network prefix bits (supported 16,777,214 hosts)
- Class B: 16 network prefix bits (supported 65,534 hosts)
- Class C: 24 network prefix bits (supported 254 hosts)

The classful arrangement was inefficient when allocating IP addresses and led to a waste of IP address spaces. For example, an organization with 300 devices couldn’t have used a Class C IP address, which only permitted 254 devices. So, the organization would’ve been forced to apply for a Class B IP address, which provided 65,534 unique host addresses. However, only 300 devices would’ve been connected, which would’ve left 65,234 unused IP address spaces.

Classless Inter-Domain Routing (CIDR) is an IP address allocation method that improves data routing efficiency on the internet. CIDR addresses use variable length subnet masking (VLSM) to alter the ratio between the network and host address bits in an IP address.

A VLSM sequence allows network administrators to break down an IP address space into subnets of various sizes. Each subnet can have a flexible host count and a limited number of IP addresses. A CIDR IP address appends a suffix value stating the number of network address prefix bits to a normal IP address.

Let’s look at a few examples to further cement our understanding:

- In `x.x.x.x/24`, `/24` represents the number of network bits, which means the given address block contains 2<sup>8</sup> or 256 hosts or IP addresses.
- Similarly, in `x.x.x.x/20`, `/20` represents the number of network bits, which means the given address block contains 2 <sup>12</sup> or 4096 hosts or IP addresses
- In the edge case of `/32` network bits, there would only be `1` host IP address. While `/0` represents all the 2<sup>32</sup> IP addresses in IPv4.

## Route Table

The Route Table, also referred to as the routing table, is responsible for providing routing instructions within a network and is associated with specific subnets. By default, every VPC is created with a main route table, and each subnet in the VPC is automatically associated with this main route table. The main route table cannot be deleted. However, we can modify its routes. We do have the option of creating customized route tables for our subnet.

For instance, in the scenario where a Virtual Private Cloud (VPC) is established with the network layer `10.10.0.0/16`, along with two subnets, `10.10.1.0/24` and `10.10.2.0/24`, each default subnet will be allocated a default route table.

Inside the route table, there will exist a route entry with the following details:

- Destination: 10.10.0.0/16
- Target: local

This particular route entry signifies that resources created within the same VPC can communicate with each other.

![imgur.png](https://i.imgur.com/6xQYqzY.png)

## Internet gateways

An internet gateway is a component that facilitates communication between instances within a VPC and the internet. It essentially acts as a gateway for internet traffic to and from instances in the VPC. When attached to a VPC, the internet gateway enables instances to connect with resources outside the VPC, such as accessing the internet or communicating with other AWS services outside the VPC.

An internet gateway scales horizontally, is highly available, and doesn’t cause bandwidth constraints for our VPC’s traffic. It facilitates outbound and inbound traffic flows for instances within your VPC.

- **Outbound traffic**: The internet gateway enables instances within our VPC to communicate with resources and services outside the VPC boundaries, such as accessing websites, downloading updates, or interacting with external APIs. It serves as the conduit through which traffic from our VPC is routed to the internet, allowing the instances to interact with the global network.
- **Inbound traffic**: With the appropriate security configurations, the internet gateway permits incoming traffic to reach your instances from the internet. This functionality is crucial for hosting services, websites, or applications that need to be publicly accessible. By properly configuring security groups and network access control lists, you can control and manage the types of inbound traffic allowed to reach your instances, enhancing security posture.

> [!NOTE]  
> Internet gateways don’t give access to the internet by themselves. We must create routes in our route table to allow the subnets in our VPCs to communicate with the internet.

## NAT gateways

A private subnet is any subnet that doesn’t have a route to an Internet Gateway (that is, any subnet that is not public). As a result, they have no means to communicate with the Internet. This is where NAT Gateways and Instances come in to play, they allow a private subnet access to the Internet.

A NAT gateway is a Network Address Translation (NAT) service. You can use a NAT gateway so that instances in a private subnet can connect to services outside your VPC but external services cannot initiate a connection with those instances.

To use the NAT Gateway, assign a route in the private subnet, in lieu of a route to an Internet Gateway. Traffic destined for the Internet will flow from the private subnet to the NAT Gateway in the public subnet, and then out to the Internet through the Internet Gateway.

When you create a NAT gateway, you specify one of the following connectivity types:

- **Public connectivity type**: This is the default connectivity type used by NAT gateways, which enables instances in private subnets to access the internet through a public NAT gateway. However, these instances cannot receive unsolicited inbound connections from the internet. An elastic IP address must be associated with the NAT gateway during its creation in the public connectivity type.
- **Private connectivity type**: It allows instances in private subnets to connect to other VPCs or on-premises networks through a private NAT gateway. An elastic IP address cannot be associated with the private NAT gateway.

## Putting it together

For a public subnet to have Internet access, inbound and outbound, an account needs:

- Internet Gateway attached to a VPC
- Route to the Internet Gateway in the attached route table
- Instances have public IP addresses (auto-assigned or attached Elastic IP address)
- Appropriate security group and NACL allowances

For a private subnet to have Internet access, the following will provide outbound Internet access but not inbound:

- Internet Gateway attached to a VPC
- NAT Gateway or Instance in a public subnet in the same VPC
- Route to the NAT Gateway or Instance in the private subnet’s attached route table
- Appropriate security group and NACL allowances

![imgur.png](https://i.imgur.com/ilxpsuE.png)
