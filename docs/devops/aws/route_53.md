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
| **A records**     | Maps a domain name to an IPv4 address. For example, we might have an A record that maps "example.com" to "192.0.2.1".                                                                                                   |
| **AAAA records**  | Maps a domain name to an IPv6 address. For example, we might have an A record that maps "example.com" to "2001:db8:3333:4444:5555:6666:7777:8888".                                                                      |
| **CNAME records** | Maps one domain name to another. For instance, we might use a CNAME record to point "www.example.com" to "example.com".                                                                                                 |
| **MX records**    | Specifies mail servers responsible for receiving email on behalf of a domain. MX records often point to mail servers like "mail.example.com".                                                                           |
| **TXT records**   | Stores arbitrary text information associated with a domain name. This can be used for various purposes, such as verifying domain ownership or providing SPF (Sender Policy Framework) records for email authentication. |

## Amazon Route 53

![](https://user-images.githubusercontent.com/17776979/193864638-68b6c7dc-c6ff-447a-a783-2c26a451941c.png)

### Domain Registration


### Hosted zone

A hosted zone in Route 53 is like a digital folder where we store all the important details about a website's address, known as DNS records. 


### Route 53 - Records

Each record contains:

- Domain/ subdomain Name - eg: example.com
- Record Type - eg: A, AAAA...
- Value - eg: 12.12.23.34
- Routing policy: how route 53 responds to queries
- TTL: amount of time the record cached at DNS Resolvers

### Route 53 - Record Types

- A: maps a hostname to IPv4
- AAAA: maps a hostname to IPv6
- CNAME: maps a hostname to another hostname

* The target is a domain name which must have an A or AAAA record
* Can't create CNAME record for the top node of a DNS namespace (Zone Apex). Eg: You can not create for `example.com` but you can create for `xxx.example.com`

- NS - Name Servers for the Hosted Zone

* Control how traffic is routed for a domain

### Route 53 - Hosted Zones

A hosted zone is a container for records, and records contain information about how you want to route traffic for a specific domain, such as example.com, and its subdomains (acme.example.com, zenith.example.com). A hosted zone and the corresponding domain have the same name. There are two types of hosted zones:

- Public hosted zones contain records that specify how you want to route traffic on the internet.
- Private hosted zones contain records that specify how you want to route traffic in an Amazon VPC

![](https://user-images.githubusercontent.com/17776979/193867675-1d36f0e8-c999-4462-bea5-e2c539fb889c.png)

### CNAME vs Alias

**CNAME**

- Point a hostname to any other hostname (app.mydomain.com -> blabla.anything.com)
- Only for non root domain

**Alias**

- Point a hostname to an AWS Resource (app.mydomain.com -> blabla.amazonaws.co)
- Works for root domain and non root domain
- Free of charge
- Native health check
- Can not set the TTL
- Can not set an alias record for an EC2 DNS name

### Route 53 - Routing Policies

Routing Policies defines how Route 53 responds to DNS queries. Route 53 provides following policies:

**Simple**

- Typical route traffic to a single resource
- Can specify multiple values in the same record (a random one is chosen by the client)
- Can not be associated with Health Checks

**Weight**

- Control % of the requests that go to each specific resource.
- Can be associated with Health Checks
- Use case: load balancing between regions, testing new application version...

**Latency based**

- Redirect to the resource that has least latency close to us
- Can be associated with Health Checks

![](https://user-images.githubusercontent.com/17776979/193872163-a409d4b0-e0ea-4d3e-8280-ce784b87ff8e.png)

**Failover**

![](https://user-images.githubusercontent.com/17776979/193872361-31614a43-c40d-48c1-8442-d0190904c207.png)

**Geolocation**

- This routing is based on user location
- Specify location by continent, country
- Should create `Default` record (in case there is no match on location)
- Can be associated with Health Checks

**Geoproximity**

- Route traffic to your resources based on the geographic location of users and resources

**Multi-Value Answer**

- Use when routing traffic to multiple resources
- Up to 8 healthy records are returned for each Multi-Value query
- Can be associated with Health Checks (returns only values for healthy resources)

### Route 53 - Health Checks

- HTTP Health Checks are only for public resources
- About 15 global health checkers will check the endpoint health
- Route 53 health checker are outside the VPC -> they can not access private endpoints (You can create CloudWatch Metric and associate a CloudWatch Alarm, then create a Health Check that checks the alarm itself)
