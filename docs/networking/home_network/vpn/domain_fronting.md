# Domain Fronting

Domain fronting is a censorship circumvention and obfuscation technique that hides the true destination of HTTPS traffic by using a trusted domain (e.g., a popular site) in the TLS SNI field, while directing the encrypted request to a hidden, malicious, or blocked site in the HTTP **Host** header. It abuses CDNs, which serve both domains, making traffic appear legitimate to firewalls.

## How CDN Routing Works

CDNs route traffic based on the **Host header**, not the DNS domain used for resolution:

- Multiple domains can resolve to the same CDN IP addresses
- CDN edge servers examine the Host header to determine routing
- The origin server lookup is based entirely on the Host header value

```mermaid
sequenceDiagram
    participant Client
    participant DNS
    participant CDN Edge
    participant Hidden Origin

    Client->>DNS: Query whitelisted.com
    DNS-->>Client: CDN IP (e.g., 13.224.x.x)
    Client->>CDN Edge: HTTP Request<br/>Host: hidden.com
    CDN Edge->>Hidden Origin: Forward request
    Hidden Origin-->>CDN Edge: Response
    CDN Edge-->>Client: Response
```

## CloudFront Alternate Domain Names

CloudFront allows distributions to claim "alternate domain names" (CNAMEs) that determine which distribution handles requests for a given Host header.

### CNAME Uniqueness

- Each alternate domain name can only be associated with **ONE distribution globally**
- First-come-first-served across ALL AWS accounts
- Attempting to claim an already-used domain results in `CNAMEAlreadyExists` error

## Domain Fronting Attack Flow

```mermaid
sequenceDiagram
    participant Client
    participant ISP/Firewall
    participant DNS
    participant CDN Edge
    participant Attacker Origin

    Client->>DNS: 1. Query link.e.tiktok.com
    DNS-->>Client: CloudFront IP

    Client->>ISP/Firewall: 2. TCP to CloudFront IP
    Note over ISP/Firewall: Sees: "Connection to<br/>legitimate CDN" ✓

    Client->>CDN Edge: 3. HTTP Request<br/>Host: attacker-claimed.domain
    Note over CDN Edge: 4. Routes based on<br/>Host header only

    CDN Edge->>Attacker Origin: Forward to attacker's origin
    Attacker Origin-->>CDN Edge: Response
    CDN Edge-->>Client: Response
```
