# Elastic Load Balancing and Auto Scaling groups

![](https://user-images.githubusercontent.com/17776979/195628626-d57e8b58-f520-41dd-b8b3-1547537d3d61.png)

## Elastic Load Balancing

ELB acts as a single point of interaction for clients and distributes incoming traffic across multiple targets (e.g., EC2 instances or IP addresses).

**Why use a load balancer**

- **Distribute Traffic**: Spread load evenly across multiple instances.
- **Single Access Point**: Expose a single endpoint for your app.
- **Fault Tolerance**: Automatically handle failures of backend instances.
- **Health Checks**: Monitor the health of instances and route traffic only to healthy targets.
- **SSL Termination**: Manage HTTPS traffic with centralized SSL/TLS certificates.
- **Sticky Sessions**: Keep sessions consistent using cookies.
- **High Availability**: Distribute traffic across multiple availability zones.
- **Public/Private Traffic Separation**: Manage internal and external traffic separately.

**How ELB Work**

1. The client sends a request to the load balancer.
2. The listeners in your load balancer receive requests matching the protocol and port that you configure (e.g., HTTPS on port 443)..
3. The receiving listener evaluates the incoming request against the rules you specify, and if applicable, routes the request to the appropriate target group. You can use an HTTPS listener to offload the work of TLS encryption and decryption to your load balancer.
4. Healthy targets in one or more target groups receive traffic based on the load-balancing algorithm, and the routing rules you specify in the listener.

### Types of load balancer on AWS

AWS provides 4 kinds of managed load balancer:

- Classic load balancer (v1 - old generation)
  - HTTP, HTTPS, TCP, SSL
- Application load balancer (v2 - new generation) - Layer 7
  - HTTP, HTTPS, gRPC, WebSocket
- Network load balancer (v2 - new generation) - Layer 4
  - TCP (HTTP, HTTPS), TLS, UDP
- Gateway load balancer - Layer 3

1. **Application Load Balancers**: Ideal for routing HTTP/HTTPS traffic and performing advanced traffic routing and content-based routing - Layer 7
   - HTTP, HTTPS, gRPC, WebSocket
2. **Network Load Balancers**: Designed for handling TCP/UDP traffic with high performance and low latency - Layer 4
3. **Gateway Load Balancers**: Used for deploying third-party virtual appliances, such as firewalls, intrusion detection systems, and other network appliances - Layer 3
4. **Classic Load Balancers**: An older type of load balancer that is still available for use, primarily for applications not yet migrated to the newer load balancer types.

> [!NOTE]  
> When using ALB, the application servers don't see the IP of the client directly. The true IP of the client is inserted in the header `X-Forwarded-For`

## Application Load Balancer

An Application Load Balancer operates on the application layer (the seventh layer of the OSI model) and handles HTTP/HTTPS/Web Socket requests.

### Target groups in ALB

Target groups are used to define where an ALB routes the incoming requests from a client. While creating a target group, we define the type of targets, the protocols, and the ports. ALBs support EC2 instances, IP addresses, and Lambda functions as its target types, HTTP and HTTPS as its protocols, and ports 1-65535.

![imgur.png](https://i.imgur.com/fj1EGg3.png)

### Routing algorithms in ALB

Application Load Balancers support various routing algorithms to distribute incoming requests across multiple target groups.

- **Round robin**: This is the default algorithm used by ALBs and distributes incoming requests evenly across multiple targets in a sequential order.
- **Least outstanding requests**: Here, incoming requests are sent to the target with the least number of requests whose status is in progress.
- **Weighted random**: In this algorithm, weights are assigned randomly to requests that evenly distribute requests across targets in a random order.

### Listeners in ALB

Application Load Balancers use listeners, that check connection requests received from the client. We define rules in listeners that determine how our load balancer routes the requests it receives. A single listener can have multiple rules and are evaluated according to the priority we set up. Following are the conditions we can define in rules for our listeners:

- **HTTP header conditions**: These rules are created to handle client requests based on their HTTP headers.
- **HTTP request method conditions**: These rules are created to handle client requests based on their HTTP request methods.
- **Host conditions**: Rules can be created to route requests based on the name of the host.
- **Path conditions**: Path conditions in the URL of the request can be used to create rules to route requests to different targets.
- **Query string conditions**: Key/value pairs in the query string of the incoming request can be used to create rules to route requests.
- **Source IP address conditions**: Rules can be created that route incoming requests based on the source IP address. This IP address must be specified in the CIDR format.

![imgur.png](https://i.imgur.com/bf1x54P.png)

## Network Load Balancer

Network Load Balancer (NLB) operates on the transport layer, the fourth layer of the OSI model, and is used to distribute incoming TCP and UDP traffic across multiple targets. NLB uses a single static IP address and is optimized to handle sudden and volatile traffic patterns.

### Target groups in NLB

Network Load Balancers support EC2 instances, private IP addresses, and Application Load Balancers as their targets. We can use on-premises servers as targets if the selected target type is IP by connecting them to the AWS cloud using AWS Direct Connect or AWS Site-to-Site VPN.

Network Load Balancers can forward the requests it receives to an Application Load Balancer. Following are some use cases where ALBs are added as targets for NLBs:

- **Multimedia services**: A combination of NLB and ALB can be used in multimedia services where a single endpoint is required for multiple protocols, such as HTTP for signaling and RTP to stream content.
- **AWS PrivateLink**: An NLB can be used to create a route between clients and an ALB over AWS PrivateLink.

## Gateway Load Balancer

Gateway Load Balancers (GLB) operate on the network layer, the third layer of the OSI model, and allow us to maintain, scale, and deploy third-party virtual appliances, such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems.

![imgur.png](https://i.imgur.com/ma2EsxJ.png)

In the diagram above, traffic coming from the internet uses the internet gateway to get to the GLB endpoint located in the consumer VPC. This endpoint routes this traffic to the Gateway Load Balancer, which distributes this traffic to the virtual appliance (the target of the GLB). Once the virtual appliance analyzes the traffic, it is sent back to the Gateway Load Balancer, which sends it to the application servers located in the consumer VPC.

### Sticky session

Sticky sessions are a way to ensure that incoming requests, which belong to the same client session, land on the same target behind the load balancer. This feature is invaluable for applications that depend on server-side session state persistence.

**Benefits of using sticky sessions**

- **User experience consistency**: This means, across requests, the state of the sessions should be consistent, and therefore, users should always experience consistency in usage. This uniformity is key to building trust and reliability in the application, as it reassures users that their actions and data are persistently recognized across their journey.
- **Improved performance**: This reduces the overhead of re-establishing the session state with each request and improves application performance. By streamlining session management, applications can serve users more swiftly and efficiently, leading to better overall performance and a more satisfying user experience.

**Considerations for using sticky sessions**

While sticky sessions offer significant benefits, they also present challenges that need careful consideration.

- **Drawback**: Sticky sessions can significantly affect the scalability of an application and limit the load balancer’s ability to distribute traffic equally across all targets. This can lead to some servers being overburdened while others remain underutilized, potentially causing uneven resource consumption and performance issues across the infrastructure.
- **Fault tolerance**: In cases where a session is sticky to a target, unless the application is designed to duplicate the session states, this could lead to lost sessions in the event of a server failure. Therefore, implementing mechanisms for session state redundancy is crucial to prevent data loss and ensure the continuity of user sessions.
- **Session data management**: The application must be designed to manage session data effectively, especially in distributed environments where there is a need to replicate session data or centrally store it. This requires a well-thought-out strategy for session data synchronization and storage to facilitate a seamless user experience and robust data management practices.

> [!NOTE]  
> It's important to note that stickiness, or session affinity, is not supported by Network Load Balancer (NLB).

### Cross zone load balancing

In Elastic Load Balancers, incoming traffic is distributed evenly across load balancer nodes in each AZ, which then forwards these requests to the targets registered within that AZ. However, Elastic Load Balancers allow us to enable cross-zone load balancing, where all load balancer nodes can distribute traffic to all available healthy targets, irrespective of their Availability Zone.

![](https://user-images.githubusercontent.com/17776979/192680716-6a54d379-bc9d-4821-8480-8a52ec532436.png)

**Cross-zone balancing in different load balancers**

Application Load Balancer (ALB)

- Always enabled (cannot be disabled).
- No inter-AZ data transfer charges.

Network Load Balancer (NLB)

- Disabled by default but can be enabled.
- Enabling incurs inter-AZ data transfer charges.

Gateway Load Balancer (GLB)

- Optionally enabled to distribute traffic across all AZs.

Classic Load Balancer (CLB)

- Disabled by default but can be enabled.
- No inter-AZ data transfer charges.

## Auto Scaling Groups

Amazon Auto Scaling Groups (ASG) is a service provided by AWS that helps your applications automatically adjust the number of EC2 instances they are using based on the load and demand. This ensures your application has enough capacity to handle traffic while minimizing unnecessary costs.

The goal of the ASG is to:

- **Scale out**: Automatically add more EC2 instances when there is increased load.
- **Scale in**: Automatically remove EC2 instances when there is less load.
- **Minimum and maximum instance limits**: Ensure there are always a minimum number of instances (to maintain uptime) and never more than a specified maximum (to control costs).
- **Automatic load balancing**: Automatically register new instances with a load balancer to distribute traffic across all healthy instances.
- **Instance replacement**: If an instance is terminated (e.g., due to failure or scaling in), ASG can automatically create a new instance to replace it.

**Attributes of an Auto Scaling Group**

1. Launch Template:

   - A blueprint that defines the configuration for new EC2 instances.
   - Includes settings such as the AMI (Amazon Machine Image), instance type, key pair, security groups, and any startup scripts.

2. Minimum Size / Maximum Size / Initial Capacity

   - The minimum number of instances ASG will maintain.
   - The maximum number of instances allowed in the group.
   - The starting number of instances when the ASG is created.

3. Scaling Policies

   - Define how the ASG should respond to changes in load.
   - Can be configured based on metrics like CPU usage, network activity, or custom metrics.

ASG can also scale automatically based on CloudWatch alarms (e.g., if CPU utilization exceeds a threshold, ASG can trigger scaling).

**Recommended Metrics for Scaling**

- CPUUtilization: Monitors the average CPU usage across all instances in the ASG. Scaling out can occur when CPU usage is too high.
- RequetsCountPerTarget: Ensures the number of requests per instance is stable. Useful for web applications or APIs.
- Average Network In/ Out: Useful for applications that are network-intensive.
- Custom metric: You can define custom metrics (e.g., queue length or memory usage) based on the unique needs of your application.

**Scaling Cooldown Period**

- After a scaling activity (either scaling out or scaling in), the ASG enters a cooldown period.
- Default Cooldown: 300 seconds (5 minutes).
- During this period, no further scaling actions will be taken to allow the system to stabilize. This helps prevent over-scaling or unnecessary actions due to temporary spikes or drops in metrics.

**How It All Works Together**

1. If traffic to your application increases, CloudWatch monitors metrics (e.g., high CPU usage or increased requests).
2. ASG triggers a scale-out action and launches new EC2 instances using the launch template.
3. New instances are automatically registered to a load balancer.
4. When traffic decreases, ASG triggers a scale-in action and terminates excess instances.
