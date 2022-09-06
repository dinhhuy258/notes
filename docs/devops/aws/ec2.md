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

## AMI

An Amazon Machine Image (AMI) is a master image for the creation of virtual servers (EC2).

AMI are customization of an EC2 instances

- You can add your own software, configuration, operating system, monitoring...
- Faster boot/ configuration time because all your software is pre-packaged

AMI are built for a specific region (and can be copied across regions)

## EBS

EBS is a network drive you can attach to your instances while they run. It's a network drive (not a physical drive) then it uses the network to communicate the instance, which means there might be a bit of latency.

EBS is locked to an Availability Zone thus EBS volume in `us-east-1a` can not be attached to `us-east-1b`.

### EBS snapshots

EBS snapshots make a backup of your EBS volume at a point in time. We can copy snapshots across AZ or region.

### EBS volume types

- gp2/ gp3 (SSD): General purpose SSD volumes
- io1/ io2 (SSD): Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads
- st1 (HDD): Low cost HDD volume designed for frequency accessed, throughput-intensive workloads
- sc1 (HDD): Lowest cost HDD volume designed for less frequency accessed workloads

**Note:** io1/io2 supports EBS multi-attach, that mean we can attach same EBS volume to multiple EC2 instances in the same AZ.

### Commands

- `lsblk`: lists information about all available or the specified block devices
- `sudo file -s /dev/xvdf`: check if the volume has any data
- `sudo mkfs -t ext4(xfs) /dev/xvdf`: format the volume to the `ext4` or `xfs` filesystem
- `sudo mount /dev/xvdf /newvolume/`: mount the volume to `newvolume` directory
- `umount /dev/xvdf`: unmount the volume,
- `df -h .`: summarize free disk space

## EC2 instance store

EBS volumes are network drives with good but `limited` performance. If you need a high performance hardware disk, use EC2 Instance Store.

- Better I/O performance
- EC2 Instance Store lose their storage if they're stopped (ephemeral)
- Good for buffer/ cache/ scratch data/ temporary content
- Risk of data loss if hardware fails

## EFS - Elastic File System

- Managed NFS (network file system) that can me mounted on many EC2
- EFS works with EC2 instances in multiple AZ

**EFS vs EBS**

EBS:

- Can be attached to only one instance at a time
- Are locked at the AZ
- gp2: IO increases if disk size increases
- io1: Can increase IO independently

EFS:

- Mounting 100s of instances across AZ
- Only for Linux instances
