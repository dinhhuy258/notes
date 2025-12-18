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

```bash
/ip settings
set ipv4-multipath-hash-policy=l4
/ipv6 settings
set multipath-hash-policy=l4
```

2. Adjust Route Distances in the main Routing Table

Set incremental distances for default routes in the `main` routing table.

![imgur.png](https://i.imgur.com/QQBWz0w.png)

If multiple routes in the `main` table have the same distance, the router will treat them as `ECMP` routes, and the `+` sign will appear, indicating that `ECMP` is active.
By setting incremental distances, only the route with the lowest distance is used, while the others act as **failover routes**.

This configuration is intentional. The `main` routing table is designed for stable, predictable routing, not load balancing.
Certain traffic requires a consistent outbound path and source IP. Allowing `ECMP` in the `main` table could cause frequent path or IP changes, leading to issues with services such as:

- Banking websites
- VPN connections
- IP-sensitive or session-based applications

For this reason, `ECMP` is explicitly disabled in the `main` table and enabled only in a dedicated `ECMP` routing table.

> [!NOTE]
> Traffic that must avoid ECMP is explicitly selected using routing rules and directed to the main table, while traffic that is safe to load-balance is routed to the ECMP table.

3. Create a new routing table for ECMP.

```bash
/routing table
add fib name=ecmp
```

4. Add Routes to the ECMP Routing Table

Add default routes with equal distances to enable ECMP.

```bash
/ip route
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out1 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out2 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out3 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out4 routing-table=ecmp
add dst-address=0.0.0.0/0 gateway=ether1-pppoe-out5 routing-table=ecmp
```

5. Add Routing Rules to Control Traffic Flow

Routing rules determine which traffic uses which routing table.

**Route Traffic That Must Bypass ECMP**

This rule should be placed at the top:

```bash
/routing rule
add action=lookup disabled=no min-prefix=0 table=main
```

Route all traffic that should bypass ECMP to the main table.

```bash
/routing rule
add action=lookup dst-address=0.0.0.0/0 src-address=192.168.0.244/32 table=main comment="Thuy's IP"
```

Traffic that is safe to load-balance can be explicitly routed to the ECMP table:

```bash
/routing rule
add action=lookup dst-address=0.0.0.0/0 src-address=192.168.0.0/24 table=ecmp
```
