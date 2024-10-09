# Setup Wireguard VPN on Mikrotik router

In this document, we will set up a Wireguard VPN on a MikroTik router and configure the tunnel for use with a specific IP only.

Please note: Enabling fasttrack along with Wireguard may cause slow requests to Wireguard. The current solution is to remove fasttrack rule.

## Setup Wireguard VPN

### Prepare Wireguard client file

The WireGuard client configuration file should be formatted as follows:

```
[Interface]
PrivateKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Address = x.x.x.x/24
DNS = 8.8.8.8, 8.8.4.4

[Peer]
PublicKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
PresharedKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 0
Endpoint = x.x.x.x:x
```

- PrivateKey: The private key of the client.
- Address: The IP address (with subnet) assigned to the client.
- DNS: The DNS servers to use.
- PublicKey: The public key of the server.
- PresharedKey: The preshared key for added security.
- AllowedIPs: Defines which IP addresses can be routed through the tunnel.
- Endpoint: The IP and port of the WireGuard server.

### Create WireGuard Interface on MikroTik

#### 1. Create the WireGuard Interface

![imgur.png](https://i.imgur.com/azYejNe.png)

In the `Private Key` field, input the `PrivateKey` from your client configuration file.

#### 2. Add the Peer

Fill in the peer information using the details from the client configuration:

- Public Key: The server's public key.
- Endpoint: The IP address and port of the WireGuard server.
- Allowed IPs: Enter the address ranges allowed to use this tunnel (e.g., 0.0.0.0/0 for all traffic).
- Preshared Key: The preshared key (if used).
- Interface: The WireGuard interface created earlier.

**Note**: Leave the `Private Key` empty

![imgur.png](https://i.imgur.com/yvYuWFU.png)

#### 3. Create Address list

1. Go to IP > Addresses.
2. Create a new address list, the `Address` value can be obtained in the Wireguard client file, `Interface` value is the Wireguard interface.

![imgur.png](https://i.imgur.com/ZpvGGmf.png)

### Routing for Wireguard traffic

#### 1. Create a Routing Table

Go to `Routing` -> `Tables`

![imgur.png](https://i.imgur.com/4QBtYG2.png)

#### 2. Create Route

Go to `IP` -> `Routes`

Add a new route with the following settings:

- Dst Address: `0.0.0.0/0`
- Gateway: The Wireguard interface
- Routing Table: Add a new route with the following settings:

![imgur.png](https://i.imgur.com/eNuJYZJ.png)

#### 3. Create NAT rule

Go to `IP` -> `Firewall` -> select tab `NAT`

Create a new NAT rule with the following settings:

- Chain: srcnat
- Out. Interface: The WireGuard interface
- Action: masquarage

Note: Ensure this NAT rule is set as the second priority, below any existing PPPoE NAT rules.

![imgur.png](https://i.imgur.com/7rl0A6g.png)

![imgur.png](https://i.imgur.com/Xhgs1Eb.png)

#### 4. Create Mangle rule

Create a mangle rule with the following concept: all packets destined for the target address will be routed to the WireGuard client. You can find the target IP address by using `host google.com` or `nslookup google.com`.

![imgur.png](https://i.imgur.com/9W6j7mV.png)
![imgur.png](https://i.imgur.com/z4U2uhQ.png)

## Mikrotik script

```rsc
# Create Wireguard interface
/interface wireguard
add name=my-company-wireguard \
    private-key="wireguard_interface_private_key" \
    listen-port=13231 \ 
    mtu=1420

# Add a peer
/interface wireguard peers
add name=my-company-peer \
    interface=my-company-wireguard \
    public-key="peer_public_key" \
    endpoint-address=peer_endpoint_address \
    endpoint-port=peer_endpoint_port \
    allowed-address=0.0.0.0/0,::/0 \
    preshared-key="peer_preshared_key"

# Create address list
/ip address
add interface=my-company-wireguard address=interface_address network=interface_address_network

# Create routing table
/routing table
add disabled=no fib name=output-my-company-wireguad

# Create route
/ip route
add disabled=no \
    dst-address=0.0.0.0/0 \
    gateway=my-company-wireguard \
    routing-table=output-my-company-wireguad \
    suppress-hw-offload=no

# Create NAT rule
/ip firewall nat
add chain=srcnat \
    out-interface=my-company-wireguard \
    action=masquerade \
    comment="My company's Wireguard"

# Create address list
/ip firewall address-list
add list=my_company_address_list address=x.x.x.x comment="My company page"
add list=my_company_address_list address=y.y.y.y comment="My company page"

# Create mangle firewall
/ip firewall mangle
add action=mark-routing \
    chain=prerouting \
    dst-address-list=my_company_address_list \
    new-routing-mark=output-my-company-wireguad \
    passthrough=no \
    comment="My company Wireguard"
```
