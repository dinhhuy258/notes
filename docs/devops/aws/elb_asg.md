# Elastic Load Balancing and Auto Scaling groups

![](https://user-images.githubusercontent.com/17776979/195628626-d57e8b58-f520-41dd-b8b3-1547537d3d61.png)

## Elastic Load Balancing

Load balancer are servers that forwards traffic to multiple servers downstream

**Why use a load balancer**

- Spread load across multiple downstream instances
- Expose a single point of access to your application
- Seamlessly handle failures of downstream instances
- Do regular health-check to your instances
- Provide SSL termination (HTTPS) for your website
- Enforce stickiness with cookies
- High availability across zones
- Separate public traffic from private traffic

### Types of load balancer on AWS

AWS provides 4 kinds of managed load balancer:

- Classic load balancer (v1 - old generation)
  - HTTP, HTTPS, TCP, SSL
- Application load balancer (v2 - new generation) - Layer 7
  - HTTP, HTTPS, gRPC, WebSocket
- Network load balancer (v2 - new generation) - Layer 4
  - TCP (HTTP, HTTPS), TLS, UDP
- Gateway load balancer - Layer 3

**Note**: when using ALB, the application servers don't see the IP of the client directly. The true IP of the client is inserted in the header `X-Forwarded-For`

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
