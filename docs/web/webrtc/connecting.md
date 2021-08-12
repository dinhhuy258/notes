# Connecting

## 1. Peer to peer connection

WebRTC does’t use a client/server model, it establishes peer-to-peer (P2P) connections. 

![](../../assets/images/webrtc/p2p.png)

## 2. How does it work

### 2.1 Networking real-world constraints

Most of the time the other WebRTC Agent will not even be in the same network. A typical call is usually between two WebRTC Agents in different networks with no direct connectivity.

![](../../assets/images/webrtc/networking_constraints.png)

For hosts in the same network it is very easy to connect. However, a host using `Router B` has no way to directly access anything behind `Router A`. How would you tell the different between `191.168.0.1` behind `Router A` and the same IP behind `Router B`? They are private IPs. A host using `Router B` could send traffic directly to `Router A`, but the request would end there. How does `Router A` know which host it should forward the message to?

### 2.2 NAT Mapping

IP version 4 addresses are only 32 bit(4 byte) long, which provides 4.29 billion(2 to the power of 32 = 4,294,967,296) unique IP addresses. 4.29 billion address space not enough for give public IP address for all available hosts.

To solve this issue NAT devices introduced. These devices would be responsible for maintaining a table mapping of local IP(private IP) and port tuples to one or more globally unique IP(public IP) and port tuples. By using this technology firewalls and routers allow multiple devices on a LAN with **private IP addresses to share a single public IP address**

![](../../assets/images/webrtc/nat_mapping.png)

### 2.3 STUN

Session Traversal Utilities for NAT (STUN) protocol enables a device to discover its public IP address.

STUN relies on a simple observation: when you talk to a server on the internet from a NATed client, the server sees the public `ip:port` that your NAT device created for you, not your LAN `ip:port`. So, the server can tell you what `ip:port` it saw. That way, you know what traffic from your LAN `ip:port` looks like on the internet, you can tell your peers about that mapping, and now they know where to send packets.

![](../../assets/images/webrtc/stun.png)

### 2.4 TURN

TURN (Traversal Using Relays around NAT) is the solution when direct connectivity isn’t possible. It could be because you have two NAT Types that are incompatible, or maybe can’t speak the same protocol or when a symmetric NAT is in use!

WebRTC agent bypass the Symmetric NAT restriction by opening a connection with a TURN server and relaying all information through that server. You would create a connection with a TURN server and tell all peers to send packets to the server which will then be forwarded to you. This obviously comes with some overhead so it is only used if there are no other alternatives.

![](../../assets/images/webrtc/turn.png)

### 2.5 ICE

ICE (Interactive Connectivity Establishment) is how  WebRTC connects two Agents. ICE is a protocol for establishing connectivity. It determines all the possible routes between the two peers and then ensures you stay connected. 

These routes are known as `Candidate Pairs`, which is a pairing of a local and remote transport address. This is where STUN and TURN come into play with ICE. These addresses can be your local IP Address plus a port, `NAT mapping`, or `Relayed Transport Address`. Each side gathers all the addresses they want to use, exchanges them, and then attempts to connect!

An ICE Agent is either Controlling or Controlled. The Controlling Agent is the one that decides the selected Candidate Pair. Usually, the peer sending the offer is the controlling side.

Each side must have a `user fragment` and a `password`. These two values must be exchanged before connectivity checks can even begin. The `user fragment` is sent in plain text and is useful for demuxing multiple ICE Sessions. The `password` is used to generate a MESSAGE-INTEGRITY attribute. At the end of each STUN packet, there is an attribute that is a hash of the entire packet using the `password` as a key. This is used to authenticate the packet and ensure it hasn’t been tampered with.
