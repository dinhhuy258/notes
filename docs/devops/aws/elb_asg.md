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

- Its possible to implement stickiness so that the same client is always redirected to the same instance behind a load balancer.
- This works for CLB and ALB
- The cookie used for stickiness has an expiration date you control
- Enabling stickiness may bring imbalance to the load over the backend EC2 instances

Use case:

- Make sure the user does not lose his session data

### Cross zone load balancing

![](https://user-images.githubusercontent.com/17776979/192680716-6a54d379-bc9d-4821-8480-8a52ec532436.png)

Application Load Balancer:

- Always on (can't be disabled)
- No charges for inter AZ data

Network Load Balancer

- Disabled by default
- You pay charges ($) for inter AZ data if enabled

Classic Load Balancer

- Disabled by default
- No charges for inter AZ data

### SSL/ TLS

Classic Load Balancer

- Support only one SSL certificate

Application Load Balancer

- Support multiple listeners with multiple SSL certificate
- Use Server Name Indication (SNI) to make it work

Network Load Balancer

- Support multiple listeners with multiple SSL certificate
- Use Server Name Indication (SNI) to make it work

### Connection draining

It's a time to complete `in-flight` requests while the instance is de-registering or unhealthy.

ELB will stop send new requests to the EC2 instance which is de-registering

## Auto Scaling Groups

The goal of the ASG is to:

- Scale out (add EC2 instances) to match an increased load
- Scale in (remove EC2 instances) to match an decreased load
- Ensure we have a minimum and a maximum number of EC2 instances running
- Automatically register new instances to a load balancer
- Re-create EC2 instance in case a previous one is terminated

### Auto Scaling Groups attributes

- A launch template: contains template for creating a new EC2 instances
- Min Size/ Max Size/ Initial Capacity
- Scaling policy

It's possible to scale an ASG based on CloudWatch alarm.

### Good metrics to scale on

- CPUUtilization: Average CPU utilization across your instances
- RequetsCountPerTarget: to make sure the number of requests per EC2 instance is stable
- Average Network In/ Out: if your application is network bound
- Custom metric

### Scaling cooldown

After a scaling activity happens, you are in the cooldown period (default 300 seconds), during cooldown period, ASG will not launch or terminate additional instances (to allow for metrics to stablize)
