# Route 53

## DNS

Domain Nam System which translates the human friendly hostnames into the machine IP addresses (google.com -> 172.217.18.36)

### DNS Terminologies

- Domain Registrar: Amazon Route 53, GoDaddy...
- DNS Record: A, AAAA, CNAME, NS...
- Zone File: Contains DNS records
- Name Server: resolves DNS queries
- Top Level Domain (TLD): .com, .us, .vn,...
- Second Level Domain (SLD): amazon.com, google.com,...

![](https://user-images.githubusercontent.com/17776979/193863900-b186858d-ed1a-4943-839b-f2d8283d9c40.png)

### How DNS works?

![](https://user-images.githubusercontent.com/17776979/193864202-e903fc72-4bb9-44d7-8f88-88362e16b804.png)

## Amazon Route 53

Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. Route 53 connects user requests to internet applications running on AWS or on-premises.

![](https://user-images.githubusercontent.com/17776979/193864638-68b6c7dc-c6ff-447a-a783-2c26a451941c.png)

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
