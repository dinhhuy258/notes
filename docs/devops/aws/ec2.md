# EC2

Amazon Elastic Compute Cloud (EC2) is a resizable cloud computing capacity. It allows users to run virtual servers, known as instances, for various computing tasks. EC2 offers different benefits besides being a flexible computing capacity. It allows us to deploy instances in multiple Availability Zones (AZs) within a region in combination with different services such as elastic load balancer and auto scaling group to offer high availability within a region.

![imgur.png](https://i.imgur.com/TOc56fE.png)

## Instance

An EC2 instance is a virtual server in the cloud. It can run different operating systems, including Linux, Windows, and CentOS. Instances are categories based on their computing power, memory, and networking capabilities. We can select any instance type based on our requirements.

Each instance contains a root volume to boot the instance. After launching, an instance works similarly to a server and keeps running until it is stopped, hibernated, terminated, or failed.

## Amazon Machine Image (AMI)

An Amazon Machine Image (AMI) is a pre-configured virtual machine template that contains software configurations like operating systems and other packages used to launch an instance. AMIs serve as a blueprint for creating EC2 instances, allowing for easy replication and scaling of virtual servers. Multiple instances can be launched from a single AMI. AWS offers different AMIs to cater to user requirements, including the popular Amazon Linux, Ubuntu, and Windows.

## Instance types

Instance type specifies the type of hardware for the virtual server in the cloud. AWS offers different types of instances based on their hardware capabilities:

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

![imgur.png](https://i.imgur.com/MrG2mOf.png)

## EC2 User data

It's possible to bootstrap our instances using EC2 User Data script. It will run **ONLY ONCE** at the instance first start.
EC2 User Data is used to automate boot tasks such as:

- Installing updates
- Installing software
- Download common files from the internet
- Anything you can think of

The EC2 User data scripts run with the root user

## Security Group

Security groups are like firewalls to the associated resources; they control the inbound and outbound traffic for an associated resource.

Security groups are used to secure EC2 instances from unwanted requests. We need to specify a security group to secure our EC2 instance whenever we launch an instance. If no security group is selected, EC2 uses the default security group of the VPC. The default security group allows all outbound traffic and only allows inbound requests from resources within the same security group.

### Inbound rules

- Inbound rules are used to define incoming traffic to the associated resources.
- By default, all inbound traffic is denied.

### Outbound rules

- Outbound rules define the outgoing traffic from the associated resource to the internet.
- All outgoing traffic from the associated resource is allowed by default.

We can also allow inbound traffic to a resource from certain security groups; this helps us secure the resource in a more efficient manner.

## Elastic Network Interface

An Elastic Network Interface (ENI) is a virtual network card that can be attached to the EC2 instances. It functions like a network interface card (NIC) and provides networking capabilities to the EC2 instances, allowing them to communicate with other resources in the same Virtual Private Cloud (VPC) or over the internet.

Every instance launched has a default network interface, known as the primary network interface; by default, it offers a private IP address to the instance. However, it can be configured to offer the public as well as the elastic IP address. A primary network interface can not be detached from an instance. However, we can attach more network interfaces to an instance. The number of network interfaces that can be attached to an instance depends upon the instance type and size. For example, `m1.xlarge` can have up to 4 network interfaces; similarly, `t2.micro` can have 2 network interfaces maximum.

## Storage

AWS offers flexible and easy-to-use data storage options for EC2 instances to meet all the requirements. Each option has its performance perks and cost. Some storage options offer persistent storage, while others provide fast temporary storage for the instance.

![imgur.png](https://i.imgur.com/9KLxfZV.png)

### Elastic Block Store

Elastic Block Store (EBS) is a highly reliable, durable block-level storage volume that can be attached to the EC2 instances. Multiple EBS blocks can be attached to an instance. It's a network drive (not a physical drive) then it uses the network to communicate the instance, which means there might be a bit of latency. EBS is locked to an Availability Zone thus EBS volume in `us-east-1a` can not be attached to `us-east-1b`. EBS offers different types of volumes based on different characteristics, such as gp2, gp3, io2 Block Express3, and io1.

- gp2/ gp3 (SSD): General purpose SSD volumes
- io1/ io2 (SSD): Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads
- st1 (HDD): Low cost HDD volume designed for frequency accessed, throughput-intensive workloads
- sc1 (HDD): Lowest cost HDD volume designed for less frequency accessed workloads

> [!NOTE]
> io1/io2 supports EBS multi-attach, that mean we can attach same EBS volume to multiple EC2 instances in the same AZ.

#### EBS snapshots

EBS snapshots make a backup of your EBS volume at a point in time. We can copy snapshots across AZ or region.

### Instance store

![imgur.png](https://i.imgur.com/zQh6elp.png)

An instance store is a temporary block storage for an instance physically attached to the host. Instance storage is also known as ephemeral storage. It is the fastest storage block available for EC2 since it is physically attached to the host, but not all the EC2 instance families support instance stores; for example C6 , and R6 EC2 families don’t support instance stores, while M5 EC2 instance family supports instance stores.

An instance store can not be attached or detached once the instance is launched and only exists during the lifetime of the instance. It is important to note that no two instances can be attached to a single ephemeral storage. The instance store is ideal for temporary memory and cache, offering high read-and-write IOPS and high-performance hardware.

## Elastic File System

Amazon Elastic File System (Amazon EFS) provides serverless, fully elastic file storage so that you can share file data without provisioning or managing storage capacity and performance. EFS allows simultaneous access by **various EC2** instances across **different AZs** within the **same region**, facilitating a shared data source for applications operating on multiple servers.

![imgur.png](https://i.imgur.com/5LElkiS.png)

### EFS vs EBS

**EBS**

- Can be attached to only one instance at a time
- Are locked at the AZ
- Can not be directly accessed via Internet
- Not a managed service. Resources should be provisioned beforehand by the user
- Data remains in the same AZ and multiple replicas are created within the same AZ

**EFS**

- Mounting 100s of instances across AZ
- Only for Linux instances
- Can be accessed via Internet
- A complete managed service, where the resources are provisioned by AWS
- Data remains in the same region and multiple replicas are created within the same region
- Pay as you go model
