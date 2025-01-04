# NAT

Network address translation (NAT) is the process of mapping private IP addresses to a single public IP address while information is being transferred via a router or NAT firewall. NAT is used by organizations with multiple devices needing access to the internet via a single public IP address.

![imgur.png](https://i.imgur.com/ZFX370e.png)

## Source NAT (SNAT)

SNAT changes the source IP address (internal users) of outgoing packets, typically used to:

- Share Internet Access: Allows multiple devices in a private network to use a single public IP address.
- Protect Internal IPs: Masks private IP addresses from the external network.
- Enable Routing Compliance: Adjusts source IP to meet ISP or routing requirements.

**How SNAT works:**

- The source IP is replaced with a public IP (e.g., the router's IP).
- A mapping table tracks the original IP/port to correctly route returning traffic.

![imgur.png](https://i.imgur.com/AMSsWNC.png)

## Destination NAT (DNAT)

DNAT is a technique that modifies the destination IP address and/or port of incoming packets. It is commonly used to route traffic from an external source to an internal resource behind a firewall or router.

**Primary Use Cases:**

- Port Forwarding: Redirect incoming traffic to specific devices or services in a private network.
    - Example: Redirect HTTP requests on 203.0.113.1:80 to a web server at 192.168.1.100:80.
- Load Balancing: Distribute traffic across multiple servers in a backend pool.
- Reverse Proxying: Forward requests from public endpoints to internal services.

**How DNAT works:**

1. A packet arrives at the router/firewall with a public destination IP.
2. The router uses DNAT rules to change the destination IP and/or port to an internal IP and port.
3. The packet is forwarded to the new destination.
4. When the internal server responds, the router translates the destination back to the original public IP and port.

![imgur.png](https://i.imgur.com/Pmz9lML.png)

## NAT hairpinning

NAT Hairpinning, also known as NAT Loopback, is a feature in some routers that allows devices within a private network to access a service hosted on the same private network using the public IP address or domain name of the service.

**How it works:**

1. A device inside the network sends a request to the router using the public IP address or domain of the service (e.g., a web server).
2. The router recognizes that the public IP resolves back to a service inside the same private network.
3. The router `hairpins` the traffic, meaning it:
    - Forwards the request back into the private network to the correct internal device (via DNAT).
    - Ensures that return traffic is properly routed back to the requesting device by modifying the packet headers as needed (via SNAT).

![imgur.png](https://i.imgur.com/HdVIfKS.png)
