# AWS CloudFront

AWS CloudFront is a globally-distributed network offered by Amazon Web Services, which securely transfers content such as software, SDKs, videos, etc., to the clients, with high transfer speed.

![](https://user-images.githubusercontent.com/17776979/203211420-b200a374-c293-4749-9a53-89c2988438c7.png)

**Benefits of AWS CloudFront**

- It will cache your content in edge locations and decrease the workload, thus resulting in high availability of applications.
- Improve security with traffic encryption and access controls, and use AWS Shield Standard to defend against DDoS attacks at no additional charge.

Amazon CloudFront Basics:

There are three core concepts that you need to understand to start using CloudFront: distributions, origins, and cache control.

**Distributions**

To use Amazon CloudFront, you start by creating a distribution, which is identified by a DNS domain name. To serve files from Amazon CloudFront, you simply use the distribution domain name in place of your website’s domain name; the rest of the file paths stay unchanged.

**Origins**

When you create a distribution, you must specify the DNS domain name of the origin — the Amazon S3 bucket or HTTP server — from which you want Amazon CloudFront to get the definitive version of your objects (web files).

**Cache-Control**

Once requested and served from an edge location, objects stay in the cache until they expire or are evicted to make room for more frequently requested content.

## Origins

- S3 bucket (can be private, need to setup Origin Access Identity + S3 bucket policy)
- S3 website
- Application Load Balancer (ALB must be public, EC2 can be private)
- EC2 instance (must be public)
- Any HTTP backend

## CloudFront Geo Restriction

You can restrict who can access your distribution:

- Whitelist: Allow your users to access your content only if they's are in one of the countries on a list of approved countries
- Blacklist: Prevent your users to access your content only if they's are in one of the countries on a blacklist of banned countries

## CloudFront vs S3 Region Replication

CloudFront

- Global edge network
- Files are cached for a TTL
- Great for static content that must be available everywhere

S3 Region Replication

- Must be setup each region you want replication to happen
- Files are update in nearly realtime
- Read only
- Great for dynamic content that needs to be available at low-latency in few regions

## CloudFront Caching

Reducing the number of requests to our origin server directly is one of the goals of using CloudFront. Due to CloudFront caching, more objects are served from CloudFront edge locations, which are nearer to users. This reduces latency and minimizes the load on our origin server.

We can cache on multiple things:

- Headers
- Session Cookies
- Query string parameters

![](https://user-images.githubusercontent.com/17776979/203322627-f72c5e9a-5183-469b-a303-96e3c89bc30a.png)

In the diagram, we can see that when a client makes a request to the CloudFront Edge Locations the cache will be checked on the basics of the values of headers and the cookies and the query string parameters. And then the cache has an expiry based on the time to live in the cache, and if the value is not in the cache, then the query, or the entire HTTPS request, is forwarded directly to the origin, and then the cache is filled by query response.

This page shows us how to optimize caching in [CloudFront](https://docs.amazonaws.cn/en_us/AmazonCloudFront/latest/DeveloperGuide/ConfiguringCaching.html)
