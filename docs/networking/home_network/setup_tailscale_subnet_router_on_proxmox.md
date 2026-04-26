# Setup Tailscale Subnet Router on Proxmox

## Overview

This guide configures the Proxmox host as a single Tailscale **subnet router** so every LXC, Docker container, and LAN device can reach Tailscale peers without installing Tailscale individually. One central authentication, one place to manage.

**Architecture:**

- Proxmox host (`192.168.0.2`) runs Tailscale and advertises the LAN subnet `192.168.0.0/24`
- MikroTik router (`192.168.0.1`) routes Tailscale CGNAT traffic (`100.64.0.0/10`) through Proxmox
- LXCs and Docker containers reach Tailscale peers transparently — no per-LXC config

```
LXC / Docker container
    ↓ default gateway
MikroTik (192.168.0.1)  ── static route 100.64.0.0/10 → 192.168.0.2
    ↓
Proxmox (192.168.0.2)   ── tailscale0 + MASQUERADE
    ↓
Tailscale peers (100.x.y.z)
```

> [!NOTE]
> `100.64.0.0/10` is the Tailscale CGNAT range (RFC 6598). Every Tailscale node gets an IP in this block, so one route covers all current and future peers. It is reserved and not used on the public internet — safe to route locally.

## Proxmox Host Configuration

### 1. Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Installs `tailscale` and enables `tailscaled.service`.

### 2. Enable IP forwarding

Persistent via sysctl drop-in:

```bash
echo 'net.ipv4.ip_forward = 1' | tee /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
sysctl -p /etc/sysctl.d/99-tailscale.conf
```

### 3. Bring Tailscale up as subnet router

```bash
tailscale up \
    --advertise-routes=192.168.0.0/24 \
    --accept-routes \
    --hostname=proxmox
```

Open the printed login URL in a browser to authenticate.

| Flag | Purpose |
| ---- | ------- |
| `--advertise-routes=192.168.0.0/24` | Tells the tailnet that Proxmox can forward traffic for the LAN. Other peers can reach LXCs by their LAN IPs. |
| `--accept-routes` | Lets Proxmox use subnets advertised by other Tailscale nodes. |
| `--hostname=proxmox` | Name shown in Tailscale admin console and via MagicDNS. |

### 4. Add MASQUERADE for outbound Tailscale traffic

Without this, packets leave Proxmox with the LXC's LAN source IP (`192.168.0.x`), which Tailscale peers don't know how to reply to. MASQUERADE rewrites the source to Proxmox's Tailscale IP.

```bash
iptables -t nat -A POSTROUTING -o tailscale0 -j MASQUERADE
```

> [!NOTE]
> Tailscale's built-in subnet-router masquerade only handles the **inbound** direction (tailnet → LAN). The **outbound** direction (LAN → tailnet) needs this manual rule.

### 5. Persist iptables rule

```bash
DEBIAN_FRONTEND=noninteractive apt-get install -y iptables-persistent
netfilter-persistent save
```

The MASQUERADE rule is saved to `/etc/iptables/rules.v4` and reloaded on boot.

## Tailscale Admin Console Configuration

### 6. Approve the advertised subnet route

1. Go to https://login.tailscale.com/admin/machines
2. Find the `proxmox` row → ⋯ menu → **Edit route settings**
3. Toggle ON: `192.168.0.0/24` → **Save**

### 7. Disable key expiry on Proxmox

Without this, the node key expires (default 180 days) and the entire subnet router goes offline silently — every LXC loses Tailscale connectivity at once.

1. Same machine row → ⋯ menu → **Disable key expiry**
2. Confirm. The expiry column should change to **Disabled**.

> [!NOTE]
> Disable key expiry on always-on infrastructure (subnet routers, servers). Keep it enabled on personal devices (laptop, phone) where re-auth is natural.

## MikroTik Router Configuration

### 8. Add static route for Tailscale CGNAT

- `IP` → `Routes` → `New`
  - Dst. Address: `100.64.0.0/10`
  - Gateway: `192.168.0.2`
  - Comment: `Tailscale via proxmox`
  - Defaults are fine: `distance=1`, `scope=30`, `target-scope=10`

```bash
/ip route add dst-address=100.64.0.0/10 gateway=192.168.0.2 comment="Tailscale via proxmox"
```

> [!NOTE]
> If MikroTik's WAN IP is in `100.64.0.0/10` (some ISPs assign CGNAT IPs to WAN), narrow the route or your WAN traffic will go to Proxmox. Check with `/ip address print where address~"100\\."`.

## Verification

### 9. Verify from Proxmox host

```bash
tailscale status
ping -c 2 <peer-tailscale-ip>
```

Expect to see all peers and successful ping.

### 10. Verify from an LXC

```bash
pct exec <vmid> -- ip route get <peer-tailscale-ip>
# expected: ... via 192.168.0.1 dev eth0  (MikroTik handles routing)

pct exec <vmid> -- ping -c 3 <peer-tailscale-ip>
```

> [!NOTE]
> First ping may show `Redirect Host (New nexthop: 192.168.0.2)` — this is MikroTik telling the LXC the next-hop is on the same subnet. Linux honors the redirect automatically. Subsequent packets bypass MikroTik (TTL drops by one — direct path through Proxmox only).

### 11. Verify from a Docker container

```bash
pct exec <vmid> -- docker run --rm alpine ping -c 3 <peer-tailscale-ip>
```

TTL=62 confirms the chain: container → Docker bridge → LXC eth0 → Proxmox → tailscale0.

## Adding More LXCs

Nothing extra needed. Any new LXC on `192.168.0.0/24` automatically reaches Tailscale peers via:

1. Default gateway → MikroTik (`192.168.0.1`)
2. MikroTik static route → Proxmox (`192.168.0.2`)
3. Proxmox MASQUERADE → tailscale0 → peer

The same applies to any LAN device (laptop, NAS, etc.).

## Troubleshooting

### Ping to peer fails with 100% loss from LXC, but works from Proxmox

The MASQUERADE rule is missing or not active. Verify:

```bash
iptables -t nat -L POSTROUTING -n -v | grep tailscale0
```

Expect: `MASQUERADE  all  --  *  tailscale0  0.0.0.0/0  0.0.0.0/0`

### tcpdump confirms packets leave but no reply

Run on Proxmox:

```bash
tcpdump -i tailscale0 -n 'icmp'
```

If outbound shows source `192.168.0.x` instead of `100.79.97.44`, MASQUERADE isn't applying. Re-add the rule.

### Subnet route not active in admin console

Even with `--advertise-routes` set, the route is dormant until approved. Check `tailscale status --json` for `PrimaryRoutes` — empty means not approved.

### LXCs lose Tailscale connectivity after ~6 months

Node key on Proxmox expired. SSH into Proxmox and run `tailscale up` to re-authenticate. Prevent by disabling key expiry (Step 7).
