# SSH Tunnel

SSH tunneling is a method of transporting arbitrary networking data over an encrypted SSH connection.

It can be used to:

- Add encryption to legacy application
- VPN

## Local Port Forwarding

When a user needs to access a resource or service located on a remote server but is unable to do so directly because of firewall settings, network configurations or private network limitations, local port forwarding is utilized.

```sh
ssh -L [local_addr:]local_port:remote_addr:remote_port [user@]sshd_addr
```

What it actually means is:

- On your machine, the SSH client will start listening on local_port (likely, on localhost).
- Any traffic to this port will be forwarded to the `remote_private_addr:remote_port` on the machine you SSH-ed to.

![picsur.png](https://img.dinhhuy258.dev/i/7b8151d4-1600-4cb4-940d-025c1a47ba34.jpg)

## Remote Port Forwarding (Reverse Tunneling)

Remote port forwarding, also known as reverse tunneling, is used when a user needs to allow external access to a service or application hosted on their local machine, typically behind a firewall or router, and make it accessible to a service or application running on a remote server.

Remote port forwarding reroutes traffic from a specified port on the remote server to a designated port on the local machine. This is in contrast to local port forwarding, which forwards data from a local machine to a remote server.

```sh
ssh -R [remote_addr:]remote_port:local_addr:local_port [user@]gateway_addr
```

For example, suppose a user wants to allow access to a web server (running on port `8080`) hosted on their local machine (with private IP `192.168.1.10`) from a remote server with public IP `123.45.67.89`. The user can use remote port forwarding to redirect traffic from the remote server's port `80` to their local machine’s port `8080`.

```sh
ssh -R 80:192.168.1.10:8080 user@123.45.67.89
```

![picsur.png](https://img.dinhhuy258.dev/i/97455fd3-45cc-4acc-9c36-efff54bcbec5.jpg)

## Dynamic Port Forwarding

Dynamic port forwarding enables users to create a secure tunnel between their local machine and a remote SSH server, turning the SSH server into a proxy server.

This technique enables users to establish a dynamic SOCKS proxy over an SSH connection. Dynamic port forwarding creates a general-purpose encrypted tunnel that can redirect traffic from many ports and applications over the SSH connection. This is different from local and remote port forwarding, which redirects specific ports to specific destinations.

```sh
ssh -D [local_port] [username]@[ssh_server]
```

After initiating the SSH connection with dynamic port forwarding, a SOCKS proxy is created on the specified local port (e.g., `1080`) on the client machine. Applications or services on the client machine can be configured to use this SOCKS proxy (`localhost:port`) as a gateway to route their network traffic through the SSH tunnel to the remote server.
The SSH server forwards the traffic to the final destination addresses, acting as a mediator for all the connections initiated through the SOCKS proxy.

## Troubleshoot 

If you have problem while connect to ssh tunnel. Please make sure that both configurations `AllowTcpForwarding` and `GatewayPorts` are set to `yes`

## References

- [SSH Tunneling Explained](https://www.youtube.com/watch?v=AtuAdk4MwWw)
- [A Visual Guide to SSH Tunnels: Local and Remote Port Forwarding](https://iximiuz.com/en/posts/ssh-tunnels/)
