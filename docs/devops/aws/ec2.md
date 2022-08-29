# EC2

EC2 is a part of Amazon.com's cloud-computing platform, that allows users to rent virtual computers on which to run their own computer applications.

## EC2 sizing & configuration options

- Operating System: Linux, Window or MAC OS
- How much computer power and cores (CPU)
- How much RAM
- How much storage space
- Network card: Speed of the card, Public IP address
- Firewall rules: security group
- Bootstrap script (configure at first launch): EC2 User Data

### EC2 User data

It's possible to bootstrap our instances using EC2 User Data script. It will run only once at the instance first start.
EC2 User Data is used to automate boot tasks such as:

- Installing updates
- Installing software
- Download common files from the internet
- Anything you can think of

The EC2 User data scripts run with the root user

### EC2 instance types

AWS privides us following instance types:

**General Purpose**

Great for a diversity of workloads such as web servers or code repository. It balance between compute, memory and network.

**Compute Optimized**

Great for compute intensive tasks that require high performance processors:

- Batch processing workloads
- Media transcoding
- High performance web servers
- High performance computing
- Scientific modeling & machine learning
- Dedicated gaming servers

**Memory Optimized**

Fast performance for workloads that process large data sets in memory

- High performance, relational/non-relational databases
- Distributed web scale cache stores
- In-memory databases optimized for BI
- Applications performing real-time processing of big unstructured data

**Storage Optimized**

Great for storage-intensive tasks that require high, sequential read and write access to large data sets on local storage.

- High frequency online transaction processing system
- Relational & NOSQL databases
- Cache for in-memory databases
- Data warehousing applications
- Distributed file systems

## Security Group

- Security Group are the fundamental of network security in AWS
- They control how traffic is allowed into or out of our EC2 instances
- Security groups only contain **allow** rules
- Security groups rules can reference by IP or by security group

## SSH to EC2

```sh
ssh -i /path/key-pair-name.pem instance-user-name@instance-public-dns-name
```
