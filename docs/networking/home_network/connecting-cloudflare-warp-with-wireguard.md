# Connecting to Cloudflare WARP with WireGuard

1. Install wgcf

```sh
brew install wgcf
```

2. Register new account

```sh
wgcf register
```

The new account will be saved under `wgcf-account.toml`

3. Generate WireGuard profile

```sh
wgcf generate
```

The WireGuard profile will be saved under `wgcf-profile.conf`

4. Execute script mikrotik, the script is generated at [https://mikrotik.dinhhuy258.dev/wireguard](https://mikrotik.dinhhuy258.dev/wireguard)

```rcs
# Create Wireguard interface
/interface wireguard
add name=warp-wireguard \
    private-key="private-key" \
    listen-port=13233 \
    mtu=1280

# Add a peer
/interface wireguard peers
add name=warp-peer \
    interface=warp-wireguard \
    public-key="public-key" \
    endpoint-address=engage.cloudflareclient.com \
    endpoint-port=2408 \
    allowed-address=0.0.0.0/0,::/0 \
    preshared-key=""

# Create address
/ip address
add interface=warp-wireguard address=172.16.0.2/32

# Create routing table
/routing table
add disabled=no fib name=warp-wireguard

# Create route
/ip route
add disabled=no \
    dst-address=0.0.0.0/0 \
    gateway=warp-wireguard \
    routing-table=warp-wireguard \
    suppress-hw-offload=no

# Create NAT rule
/ip firewall nat
add chain=srcnat \
    out-interface=warp-wireguard \
    action=masquerade \
    comment="Cloudflare WARP's Wireguard"

# Create routing rule
/routing rule
add action=lookup dst-address=0.0.0.0/0 table=warp-wireguard
```
