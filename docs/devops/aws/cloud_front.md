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

Origins serve as the source locations for the content distributed through CloudFront. These can be:

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

## TTL and Invalidations

CloudFront’s architecture is centered around caching content at edge locations to enhance content delivery performance and reduce latency for end users. However, this caching mechanism introduces the risk of serving stale data if updates occur at the origin.

### Time-to-Live (TTL)

Time-to-Live (TTL) refers to the duration for which an object is considered valid and cached at edge locations. When an object’s TTL expires, CloudFront initiates an origin fetch to retrieve the latest version of the object from the origin server. If the object has been updated, CloudFront fetches the updated content from the origin server and refreshes its cache. If the object hasn’t been updated, CloudFront continues serving the cached content. This ensures that users receive up-to-date content, promoting a seamless browsing experience.

CloudFront sets a TTL of 24 hours for cached objects by default. However, users can customize TTL values to align with their specific caching requirements.

In addition to this, CloudFront honors TTL directives specified by Cache-Control and Expires headers, allowing origin servers to exert control over caching behavior.

- **Cache-Control header**: The Cache-Control header includes directives instructing CloudFront on caching and serving content. Directives such as `max-age` specify the maximum amount of time (in seconds) that an object should be considered fresh and cached. CloudFront interprets this directive to set the TTL for cached objects accordingly.
- **Expires header**: The Expires header specifies an exact date and time when an object’s cached version should be considered stale and expire. CloudFront uses this information to determine the TTL for the cached object, ensuring it remains cached until the specified expiration date and time.

## How CloudFront works

Consider a scenario where Bob and Alice, two users in different regions, aim to access a popular video hosted in an Amazon S3 bucket. Bob resides near an edge location in Italy, while Alice is closer to an edge location in Spain, but they have the same regional cache.

![imgur.png](https://i.imgur.com/KxizDYA.png)

**Video available at edge location**: When Bob initiates a request to view the video, CloudFront checks if the content is cached in the edge location nearest to him. If the video is cached locally, Bob experiences a `cache hit`, resulting in immediate access to the content with minimal latency.

![imgur.png](https://i.imgur.com/S58JxWQ.png)

**Video available at regional edge location**: On the other hand, if the video is not cached in Bob’s nearby edge location, CloudFront triggers a `cache miss`. In this case, CloudFront retrieves the video from the regional edge cache, a larger cache shared by multiple edge locations within the same geographic area. If the video is found in the regional edge cache, it is promptly delivered to Bob’s edge location, ensuring efficient content delivery despite the cache miss.

![imgur.png](https://i.imgur.com/kAuaojX.png)

**Video not cached at edge or regional edge location**: However, if the video is not available in the regional edge cache, CloudFront performs an origin fetch, retrieving the video directly from the S3 bucket, the content’s origin. Once fetched, the video is stored in the regional edge cache for future requests, optimizing content delivery for subsequent users in the same geographic region.

![imgur.png](https://i.imgur.com/n0bSVDK.png)

**Future fetch requests**: When Alice initiates a request to fetch the same video, CloudFront can retrieve it from the regional edge cache.

![imgur.png](https://i.imgur.com/VkwTnUJ.png)

## Functions at Edge

Functions at Edge (FaE) are lightweight serverless functions that execute at the edge locations of a content delivery network (CDN) like AWS CloudFront. These functions enable developers to execute code in proximity to end-users, and we can use these functions for several purposes:

- **Implementing advanced HTTP logic**: CloudFront offers native features like redirecting from HTTP to HTTPS and routing to different origins based on request paths. Edge functions extend CloudFront’s capabilities, enabling the implementation of advanced HTTP logic such as cache key normalization, URL rewriting, and HTTP CRUD operations.
- **Reducing application latency**: Offloading certain application logic from the origin to the edge leverages caching (e.g., A/B testing) and executes closer to users (e.g., HTTP redirections, URL shortening, HTML rendering). In microservices or micro-frontend architectures, edge functions centralize common logic (e.g., Authorization & Authentication) at the application entry point, streamlining development and reducing redundancy
- **Protecting the application perimeter**: Edge functions enforce security controls like access control and advanced geoblocking at the edge, minimizing the attack surface of the origin and optimizing scalability.
- **Request routing**: Edge functions enable routing each HTTP request to specific origins based on application logic, facilitating scenarios such as advanced failover, origin load balancing, multi-region architectures, migrations, and application routing.


CloudFront offers two types of edge functions: `CloudFront Functions` and `Lambda@Edge`. `CloudFront Functions` offer sub-millisecond startup times and immediate scalability to handle millions of requests per second, making them ideal for lightweight tasks such as cache normalization, URL rewriting, request manipulation, and authorization.

On the other hand, `Lambda@Edge` extends AWS Lambda functionality, distributing execution across Regional Edge Caches. While `Lambda@Edge` provides more computing power and advanced features like external network calls, it has higher costs and latency overhead.

### Lambda@Edge

Lambda@Edge is a powerful feature of CloudFront that enables the execution of lightweight Lambda functions at CloudFront edge locations.

**Limitations of Lambda@Edge**

- **Supported runtimes**: Lambda@Edge supports only Node.js and Python runtimes, limiting the programming languages available for function development.
- **No VPC access**: Lambda@Edge functions execute within the AWS Public Zone, so they lack access to Virtual Private Cloud (VPC)-based resources, which might restrict certain use cases requiring VPC connectivity.
- **No Lambda layers**: Lambda@Edge does not support Lambda layers, limiting the ability to reuse common code across multiple functions.
- **Execution limits**: Lambda@Edge functions have distinct size and execution time limits compared to standard Lambda functions, imposing constraints on memory allocation and function timeout.

**Use cases for Lambda@Edge**

- **Dynamic content manipulation**: Lambda@Edge allows us to execute serverless functions at AWS edge locations, enabling dynamic content manipulation closer to the end user.
- **Content personalization**: We can use Lambda@Edge to customize content based on user requests or device characteristics, such as geolocation or user-agent.
- **Access control and authentication**: Implement access control policies, authentication mechanisms, or authorization logic at the edge to restrict access to resources or content.
- **A/B testing and experimentation**: Conduct A/B testing or experimentation by routing requests to different origins or serving different versions of content based on predefined criteria.

### CloudFront functions

CloudFront Functions are lightweight serverless functions that run at the edge of the AWS CloudFront global network. They enable developers to customize and manipulate content delivery at the edge locations, allowing for real-time processing of requests as they are received and responses as they are generated. This helps optimize content delivery, implement security measures, and personalize content without provisioning or managing additional infrastructure.

**Limitations of CloudFront Functions**

- **Limited language support**: Currently, CloudFront Functions support only JavaScript (Node.js) as the programming language, limiting language choice for developers.
- **Execution time limit**: Functions have a maximum execution time limit of 5 seconds, which may restrict the complexity of operations that can be performed within a single function.
- **Limited libraries and dependencies**: CloudFront Functions have limited access to external libraries and dependencies compared to traditional serverless platforms like AWS Lambda, which may impact the ability to reuse existing code or libraries.
- **Limited debugging tools**: Debugging CloudFront Functions can be challenging, as limited debugging tools and logging capabilities are available compared to other serverless platforms.
- **Statelessness**: Functions are stateless by design, meaning they do not maintain state between invocations, which may require developers to implement additional mechanisms for state management if needed.

**Use cases for CloudFront functions**

- **Lightweight content transformation**: CloudFront Functions are lightweight JavaScript functions that execute at the edge locations, primarily for simple content transformations or modifications.
- **Header manipulation**: Modify request or response headers, such as adding custom headers, removing sensitive headers, or altering caching directives.
- **URL rewriting and redirection**: Implement URL rewriting rules, URL redirections, or URL-based routing logic to customize the behavior of content delivery.
- **Content security policies**: Enforce security policies, such as cross-origin resource sharing (CORS), content security policies (CSP), or cookie attributes, to enhance security and mitigate common web vulnerabilities.
