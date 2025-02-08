# Route 53

Amazon Route 53 is an AWS web service that provides Domain Name System (DNS) functionality. Route 53 offers two main services: domain registration and domain hosting. Route 53 is a globally resilient service as the name servers are distributed globally and have the same datasets across all the servers. So, even if a region is affected by outages, Route 53 will still function.

## Domain Name System (DNS)

Every device connected to the internet has its unique IP address, enabling communication with it. When we browse the internet, we access web applications hosted on servers, each identified by a distinct IP address.

The Domain Name System (DNS) is a hierarchical decentralized naming system that translates human-readable domain names, such as `dinhhuy258.dev`, into IP addresses. The DNS organizes domain names into a hierarchical tree-like structure:

- At the top are root domain servers, the highest authority in DNS.
- Below are top-level domains (TLDs) like `.com` and country-code TLDs. Each TLD is managed by its registry.
- Second-level domains (SLDs) sit beneath TLDs and often represent organizations. Subdomains can extend from SLDs.

![imgur.png](https://i.imgur.com/mQ5N15t.png)

### DNS Zone

A DNS zone is a distinct, logical segment within the domain namespace of the Domain Name System (DNS). It provides administrators, organizations, or other entities with fine-grained control over DNS records and configurations for specific parts of a domain.

For instance, the domain `example.com` can be part of a larger DNS zone. Within this zone, subdomains like `blog.example.com` and `community.example.com` can exist. If `community.example.com` requires more specific or granular management—perhaps due to a large number of devices or high traffic—the administrator could decide to split it into its own DNS zone with a separate authoritative name server, thus giving independent control over its DNS records.

### DNS Records

| **Record Type**   | **Value**                                                                                                                                                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SOA records**   | Provides authoritative information about a DNS zone, including details such as the primary name server for the zone, the email address of the zone administrator, and other zone management parameters.                 |
| **NS records**    | Specifies the authoritative name servers for a domain. These servers are responsible for providing DNS information about the domain.                                                                                    |
| **A records**     | Maps a domain name to an IPv4 address. For example, we might have an A record that maps `example.com` to `192.0.2.1`".                                                                                                  |
| **AAAA records**  | Maps a domain name to an IPv6 address. For example, we might have an A record that maps `example.com` to `2001:db8:3333:4444:5555:6666:7777:8888`.                                                                      |
| **CNAME records** | Maps one domain name to another. For instance, we might use a CNAME record to point `www.example.com` to `example.com`.                                                                                                 |
| **MX records**    | Specifies mail servers responsible for receiving email on behalf of a domain. MX records often point to mail servers like `mail.example.com`.                                                                           |
| **TXT records**   | Stores arbitrary text information associated with a domain name. This can be used for various purposes, such as verifying domain ownership or providing SPF (Sender Policy Framework) records for email authentication. |

## Amazon Route 53

![imgur.png](https://i.imgur.com/L6dUAxH.png)

### Domain Registration

AWS Route 53 enables you to register domain names, search for available domains, and manage DNS records all in one place. After registration, Route 53 integrates directly with its DNS service, simplifying domain management. It also offers optional WHOIS privacy protection, automatic domain renewal, and supports domain transfers from other registrars, making it easier to maintain control over your domains and associated resources.

Additionally, you can transfer domain registration from another registrar to AWS Route 53 for supported top-level domains (TLDs). It’s also possible to transfer domains between AWS accounts, streamlining management across different teams or organizations.

### Hosted Zones

A hosted zone is a container for records, and records contain information about how you want to route traffic for a specific domain, such as `example.com`, and its subdomains (`acme.example.com`, `zenith.example.com`). A hosted zone and the corresponding domain have the same name. There are two types of hosted zones:

- Public hosted zones contain records that specify how you want to route traffic on the internet.
- Private hosted zones contain records that specify how you want to route traffic in an Amazon VPC

![imgur.png](https://i.imgur.com/o2q1Mrt.png)

### Health checks

Route 53 health checks are a function that allow you to monitor the health of selected types of AWS resources or any endpoints that can respond to requests. Route 53 health checks can provide notifications of a change in the state of the health check and can help Route 53 to recognize when a record is pointing to an unhealthy resource, allowing Route 53 to failover to an alternate record.

![imgur.png](https://i.imgur.com/5vcKFmy.png)

Types of Route 53 health checks:

- **Endpoint health checks**: You can configure to monitor an endpoint that you specify by IP address or domain name. Within a fixed time interval that you specify. Route 53 submits automated requests over the Internet to your application, server, or other resources to verify that it is accessible, available, and functioning properly.
- **Health checks that monitor other health checks**: This type of health check monitors other Route 53 health checks. Basically, a `parent` health check will monitor one or more `child` health checks. If the provided number of child health checks report as healthy, then parent health checks will also be healthy. If the number of healthy `child` checks falls below a set threshold, the `parent` check will be unhealthy.
- **Health checks for Amazon CloudWatch Alarms**: You can also perform health checks associated with alarms created in the CloudWatch service. These types of Route 53 health checks monitor CloudWatch data streams sent to previously configured alarms. If the status of the CloudWatch alarm is OK, the health check will report as OK.

### Routing Policies

Routing policies are the rules and algorithms that route traffic to different endpoints like IP addresses, AWS resources, or other domain names based on various criteria. These routing policies provide flexibility and control over how traffic is distributed to different endpoints, allowing users to optimize the performance, availability, and cost-effectiveness of their applications and services hosted on AWS.

#### **Routing strategies**

Route 53 offers several routing policies to allow users to implement sophisticated traffic routing strategies tailored to their specific requirements. In this lesson, we will explore the common routing policies provided by Route 53. Let’s explore the Routing Policies that are provided by Route 53.

**Simple routing policy**

The Simple Routing Policy is used for a single resource that performs a specific function for your domain. For example, you might use it to route traffic to an Amazon EC2 instance serving content for the `example.com` website.

With this policy, you can configure multiple values (such as multiple IP addresses or endpoints). When multiple values are configured, Route 53 will randomly choose one to route traffic to.

Note that the Simple Routing Policy cannot be used in conjunction with Health Checks.

![imgur.png](https://i.imgur.com/TLmWr8w.png)

**Failover routing policy**

The Failover Routing Policy is designed to provide high availability by routing traffic to a backup resource when the primary resource becomes unavailable. It allows users to define a primary resource and a standby (failover) resource, along with health checks to monitor the availability of each resource.

When Route 53 detects that the primary resource is unhealthy, it automatically directs traffic to the failover resource, ensuring seamless failover in case of resource failures or disruptions.

![imgur.png](https://i.imgur.com/UULI0gQ.png)

**Weighted routing policy**

Weighted routing policy enables users to control the distribution of traffic among multiple resources by assigning weights to each resource. This allows you to assign weights to resource record sets. For instance, you can specify `25` for one resource and `75` for another, meaning that `25%` of requests will go to the first resource and `75%` will be routed to the second.

![imgur.png](https://i.imgur.com/zV8EbY1.png)

**Latency routing policy**

Latency Routing Policy routes traffic to the resource with the lowest latency for the user, ensuring optimal performance and minimal response times.

![imgur.png](https://i.imgur.com/CXitl2c.png)

This policy best suits latency-sensitive applications such as online gaming platforms, video streaming services, or real-time communication applications, where minimizing latency is essential for user satisfaction.

**Geolocation routing policy**

Geolocation routing policy directs traffic based on the user’s geographic location, ensuring that users are routed to the nearest available resource or a resource optimized for their region.

![imgur.png](https://i.imgur.com/qsbS69X.png)

This policy best suits content delivery networks (CDNs), regional applications, or services with data sovereignty requirements, where serving content from geographically closer servers is essential.

**Multivalue routing policy**

Multivalue Routing Policy combines elements of simple routing and failover routing to improve availability and fault tolerance. With multivalue routing, users can create multiple records with the same name, each associated with its **health check**. When a user queries a domain, up to eight healthy records are returned, while unhealthy ones are excluded. If more than eight healthy records exist, they are returned randomly.

![imgur.png](https://i.imgur.com/K7IlTaH.png)

**IP-based routing policy**

IP-based routing policy directs traffic based on the IP address of the requesting device or DNS resolver. It allows users to define routing rules based on IP addresses or ranges, ensuring that requests from specific devices or networks are directed to designated resources.

![imgur.png](https://i.imgur.com/Bklk4Tc.png)
