# Setup ECMP on Mikrotik router

## ECMP

ECMP (Equal-Cost Multi-Path) is a routing technique used in networking where multiple paths to a destination have the same cost. ECMP allows routers to load-balance traffic across these equal-cost paths, improving bandwidth utilization and redundancy.

**Benefits of ECMP**

- Increased Bandwidth and Throughput: ECMP allows for the aggregation of bandwidth across multiple network links, effectively multiplying the capacity and enabling higher throughput for data traffic.
- Load Balancing: Traffic is distributed across multiple paths, which helps to avoid congestion and optimizes the usage of network resources. This distribution can help preven any single path from becoming a bottleneck.
- Redundancy: By having multiple paths available, ECMP enhances the network’s ability to remain operational even if one or more paths fail.
- Scalability: As network demands grow, ECMP can handle increased traffic by simply adding more paths without needing significant changes to the network’s core architecture.

In a network with multiple paths to a destination, without ECMP, **only one path** is selected, leaving the other paths **unused**. This leads to all traffic being routed through a single path, causing bottlenecks and offering no redundancy if the path fails. After applying ECMP, traffic is evenly distributed across all equal-cost paths, reducing congestion and ensuring that if one path fails, the other paths can still carry the traffic, providing both better performance and fault tolerance.

**How ECMP works**

When traffic arrives at the router, the router applies a hashing algorithm to various fields in the packet (such as source/destination IP, source/destination port, etc.) to determine which of the available equal-cost paths to use. This ensures that packets belonging to the same flow (e.g., a TCP session) always take the same path to avoid out-of-order delivery, while different flows may take different paths. This load-balancing mechanism enhances bandwidth utilization and provides redundancy, ensuring that the network can continue operating efficiently even if one path fails.

ECMP typically uses a hashing algorithm to determine the path for each flow, but other algorithms like round-robin, per-packet load balancing, or weighted round-robin can also be used to distribute traffic across multiple paths, depending on the network configuration. These algorithms offer different methods for balancing the load and managing traffic flows in the network.

## Setup ECMP

1. Update the ip hash policy settings from `l3` (layer-3 hashing of src IP, dst IP) to `l4` (layer-3 hashing of src IP, dst IP). This allows for more efficient load balancing across multiple paths.

```rsc
/ip settings
set ipv4-multipath-hash-policy=l4
/ipv6 settings
set multipath-hash-policy=l4
```

2. Adjust the route distances in the `main` routing table to ensure that the distances are set incrementally.

![imgur.png](https://i.imgur.com/QQBWz0w.png)

If the distances are not incremented, the `main` table will not show a `+` sign, indicating `ECMP` is not active. By default, the router selects the route with the lowest distance, and other paths are only used as fallback routes. We configure the `main` table this way because we do not want to apply `ECMP` in the main table. Some routing policies require specific traffic to avoid `ECMP` to prevent frequent IP changes, which could cause issues when connecting to certain services like banking...

3. Create a new routing table for ECMP.

```rsc
/routing table
add fib name=ecmp
```

4. Add routes to the new ECMP routing table.

```rsc
/ip route
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out1 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out2 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out3 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out4 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out5 routing-table=ecmp
```

Make sure the distance values for all routes in the ECMP table are the same to enable equal-cost multi-path routing.

5. Add routing rules to control traffic behavior.

This rule should be put at the top.

```rsc
/routing rule
add action=lookup disabled=no min-prefix=0 table=main
```

Route all traffic that should bypass ECMP to the main table.

```rsc
/routing rule
add action=lookup dst-address=0.0.0.0/0 src-address=192.168.0.244/32 table=main comment="Thuy's IP"

/routing rule
add action=lookup dst-address=x.x.x.x/32 src-address=192.168.0.0/24 table=wireguard
```

Define rules for routing traffic that should use ECMP.

```rsc
/routing rule
add action=lookup dst-address=0.0.0.0/0 src-address=192.168.0.0/24 table=ecmp
```
