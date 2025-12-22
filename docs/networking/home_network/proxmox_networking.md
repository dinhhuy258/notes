# Proxmox Networking

## Network Interfaces (NIC)

A network interface is a thing on Linux that can send/receive network traffic. It can be:

- Physical: a real NIC/port
- Virtual: created by software (bridge, VLAN interface, VM tap device, container veth, etc.)

> [!NOTE]
> A network interface is like a **mailbox slot**. Each slot has its own label so the post office (Linux) knows which slot to deliver packets to.

Modern Linux uses predictable network interface names (instead of old eth0, eth1, ...). These names are based on hardware location to make them stable across reboots and hardware changes.

Example: `enp1s0`

- en = Ethernet (wired networking)
- p1 = device is on PCI bus path `1` (hardware location)
- s0 = slot/function index at that PCI location

**Common prefixes**

| Prefix | Interface type                          |
| ------ | --------------------------------------- |
| en     | Wired Ethernet                          |
| wl     | WLAN, wireless local area network, WiFi |
| lo     | Loopback adapter.                       |

> [!NOTE]
> Loopback adapter is used for connections that should not be routed outside one computer.
> Every computer calls itself "localhost", which resolves to IP address 127.0.0.1.
> IPv4 local addresses are 127.0.0.1/8, and IPv6 local addresses are `::1/128`.

**Examples of common network interface names**

| Interface       | Explanation                                                                     |
| --------------- | ------------------------------------------------------------------------------- |
| wlp4s0          | WiFi card                                                                       |
| enp1s0          | Wired ethernet card                                                             |
| lo              | Loopback adapter                                                                |
| enx738899738899 | Ethernet interface named by MAC address (very common for USB Ethernet adapters) |

To view your interfaces, use the command `ip link show`. This lists all interfaces, their status (up/down), and MAC addresses.

## Linux Bridges in Proxmox

A Linux bridge (often named vmbr0, vmbr1, etc., in Proxmox) is a software-based Layer 2 (Ethernet) switch that runs inside your Proxmox host. It connects multiple network devices or interfaces together, allowing them to communicate as if they were on the same physical network segment.

- Physical NIC (e.g., enp1s0): Acts as the uplink cable connecting to your real router or switch.
- Bridge (e.g., vmbr0): A virtual switch inside Proxmox that multiple devices can plug into.
- VMs or LXCs (Containers): Virtual devices connected to this switch, like computers plugged into a physical switch.

The bridge enables efficient sharing of physical network resources among your virtual guests (VMs and containers).

**What's Happening Under the Hood**

When you attach a VM to a bridge like `vmbr0`:

- Proxmox creates a virtual "port" called a TAP device (e.g., tap100i0 for VM ID 100).
- For LXCs, it creates a veth pair (e.g., veth100i0 on the host side, connected to the container).
- These virtual ports are added to the bridge, just like plugging Ethernet cables into switch ports.
- The bridge forwards Ethernet frames (Layer 2 packets) based on MAC addresses, learning where devices are located (similar to how a real switch operates with a MAC address table).

You can inspect bridges with `ip link show type bridge`.

**Why Use a Bridge in Proxmox?**

1. Guests on Your Real LAN:

- VMs and containers can get IPs directly from your router's DHCP server, just like physical devices.
- They're reachable from other LAN devices (e.g., your laptop can ping a VM).
- Full internet access without extra configuration.

2. Resource Sharing:

- Multiple guests share one (or more) physical NICs efficiently.
- Without a bridge, assigning a physical NIC to a single guest (passthrough) wastes resources, or you'd need complex routing setups.

3. Flexibility:

- Supports VLANs for network segmentation (e.g., tag traffic with VLAN IDs).
- Can bond multiple physical NICs for redundancy or higher bandwidth (using Linux bonding).

**Typical Proxmox Bridge Setup (in /etc/network/interfaces)**

```bash
auto lo
iface lo inet loopback

iface enp1s0 inet manual  # Physical NIC: Don't assign IP here; let the bridge handle it

auto vmbr0
iface vmbr0 inet static
    address 192.168.88.10/24  # IP for the Proxmox host itself
    gateway 192.168.88.1      # Your router's IP
    bridge-ports enp1s0       # Attach the physical NIC as a port
    bridge-stp off            # Disable Spanning Tree Protocol (usually not needed in simple setups)
    bridge-fd 0               # Forwarding delay: 0 for instant forwarding
```

After editing, apply changes with `ifreload -a` or `systemctl restart networking`.

This setup makes `vmbr0` the main interface for the host and guests. Attach VMs/LXCs to `vmbr0` in their network settings for bridged mode.

## Linux Bond in Proxmox

A bond combines 2+ physical NICs into one logical interface (e.g., bond0). Linux then treats bond0 like a normal interface.

Typical Proxmox pattern:

- VMs connect to `vmbr0` (bridge)
- `vmbr0` uses `bond0` as its uplink to the network
- `bond0` aggregates physical NICs

VM → `vmbr0` (bridge) → `bond0` (`enp3s0` + `enp4s0`) → physical switch

### Bonding modes

| Name                     | Behavior                                             | Switch Config     | Use Case                            |
| ------------------------ | ---------------------------------------------------- | ----------------- | ----------------------------------- |
| active-backup            | Only one link active; automatic failover if it fails | None required     | Reliability with minimal complexity |
| 802.3ad (LACP)           | Both links active and load balanced                  | LACP/LAG required | Performance + redundancy            |
| balance-xor, balance-alb | Various load balancing methods                       | Varies by mode    | Special cases; has more gotchas     |

### Quick decision guide

- **Simple redundancy only** → active-backup (mode 1)
- **Redundancy + performance** → 802.3ad (mode 4) — _requires managed switch with LACP_
- **No switch control** → active-backup (mode 1)

## Linux VLAN in Proxmox

A VLAN (Virtual LAN) splits one physical network into multiple isolated virtual networks using VLAN tags (IEEE 802.1Q). This enables network segmentation without requiring separate physical switches or cables.

### What are VLANs?

VLANs use **802.1Q tagging** to mark Ethernet frames with a VLAN ID (1-4094). Tagged packets can traverse the same physical cable while remaining logically separated.

**Key concepts**:

- **Tagged traffic**: Packets with VLAN ID in the header (for trunks between switches/hosts)
- **Untagged traffic**: Normal packets (for end devices like VMs)
- **Trunk port**: Carries multiple VLANs (tagged)
- **Access port**: Carries one VLAN (untagged)

In Proxmox: The bridge acts as a virtual switch that can handle VLAN-tagged traffic.

### Why use VLANs in Proxmox?

**Benefits**:

- Isolate different types of traffic without separate hardware
- Improve security by segmenting networks
- Reduce broadcast domains

**Common use cases**:

| VLAN            | Purpose              | Typical ID | Why Separate?               |
| --------------- | -------------------- | ---------- | --------------------------- |
| Management      | Proxmox web UI, SSH  | 10         | Secure admin access         |
| VMs/Services    | Production workloads | 20         | Isolate from management     |
| Storage         | Ceph, NFS, iSCSI     | 30         | High bandwidth, low latency |
| Guest/Untrusted | Test labs, guest VMs | 40         | Security isolation          |
| Backup          | Backup traffic       | 50         | Prevent backup floods       |

### VLAN-aware bridge

Proxmox supports two approaches for VLANs:

**Approach 1: VLAN-aware bridge** (recommended)

- Single bridge (e.g., vmbr0) handles multiple VLANs
- VMs specify VLAN tag in network config
- Cleaner, more flexible

**Approach 2: Separate bridge per VLAN**

- Create vmbr10, vmbr20, vmbr30, etc.
- More complex configuration
- Useful for specific isolation needs

**VLAN-aware is preferred** for most Proxmox setups.
