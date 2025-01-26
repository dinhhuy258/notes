# NACL and Security Groups

Network access control lists (NACLs) and security groups are types of firewalls that control the network traffic.

Security groups are stateful firewalls that analyze everything in the data packets of the incoming traffic and maintain the state. We only need to configure rules for the incoming traffic, and the stateful firewall automatically configures the outgoing rules accordingly.

The NACLs are stateless firewalls that check the source, destination, and other parameters/rules to allow or reject the traffic.

## Security groups

In the AWS environment, a security group is a VPC-based resource that works at the EC2 instance level. It validates the incoming traffic and allows only connection requests passed by the inbound rules. We specify a security group to secure our EC2 instance; if no security group is selected, EC2 uses the default security group of the VPC. The default security group has no inbound rules and allows all outbound traffic.

## NACLs

A network access control list (NACL) is a VPC-based firewall that works on the subnet level and controls the ingress and egress traffic. Because of its stateless nature, we need to take care of the outbound and inbound rules. Every inbound rule must have an outbound rule if we want the traffic to leave our network. In NACLs, each rule is assigned a rule number that is processed in ascending order. This means that only one rule is processed at a time. We don’t get charged for using NACLs.

![imgur.png](https://i.imgur.com/pAo19WR.png)

## NACLs rules

## NACL Inbound Rules

| Rule # | Type        | Protocol | Port Range | Source IP      | Action |
| ------ | ----------- | -------- | ---------- | -------------- | ------ |
| 100    | HTTP        | TCP      | 80         | 0.0.0.0/0      | ALLOW  |
| 110    | HTTPS       | TCP      | 443        | 0.0.0.0/0      | ALLOW  |
| 120    | SSH         | TCP      | 22         | 192.168.0.0/24 | DENY   |
| 130    | ICMP        | ICMP     | N/A        | 0.0.0.0/0      | ALLOW  |
| 150    | All traffic | All      | All        | 0.0.0.0/0      | DENY   |

## NACL Outbound Rules

| Rule # | Type        | Protocol | Port Range | Destination IP | Action |
| ------ | ----------- | -------- | ---------- | -------------- | ------ |
| 100    | HTTP        | TCP      | 80         | 0.0.0.0/0      | ALLOW  |
| 110    | HTTPS       | TCP      | 443        | 0.0.0.0/0      | ALLOW  |
| 120    | SSH         | TCP      | 22         | 192.168.0.0/24 | DENY   |
| 130    | ICMP        | ICMP     | N/A        | 0.0.0.0/0      | ALLOW  |
| 150    | All traffic | All      | All        | 0.0.0.0/0      | DENY   |

## Security group vs NACL

| Security group                                                          | Network ACL                                                                                                                                      |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Operates at the instance level                                          | Operates at the subnet level                                                                                                                     |
| Applies to an instance only if it is associated with the instance       | Applies to all instances deployed in the associated subnet (providing an additional layer of defense if security group rules are too permissive) |
| Supports allow rules only                                               | Supports allow rules and deny rules                                                                                                              |
| Evaluates all rules before deciding whether to allow traffic            | Evaluates rules in order, starting with the lowest numbered rule, when deciding whether to allow traffic                                         |
| Stateful: Return traffic (outbound) is allowed, regardless of the rules | Stateless: Return traffic (outbound) must be explicitly allowed by the rules                                                                     |
