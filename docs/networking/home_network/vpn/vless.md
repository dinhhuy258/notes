# VLESS

VLESS (short for "VMess Less") is a modern, lightweight transport protocol designed for secure communication between clients and servers.
It is widely used in VPN and proxy solutions, especially within the Xray and V2Ray frameworks, to bypass censorship and enhance privacy

## How VLESS Works

1. Client-Server Model

VLESS operates on a client-server model:

- The client connects to a VLESS server using a unique user ID.
- The server verifies the ID and establishes a session.

2. No Native Encryption

Unlike VMess, VLESS does not encrypt traffic by itself. Instead, it relies on TLS or XTLS for encryption and obfuscation, ensuring data confidentiality and making traffic harder to detect

3. Stateless Operation

VLESS is stateless, meaning the server does not store session data, which improves scalability and reduces resource usage.

4. Simple Configuration

A typical VLESS configuration requires:

- Server address and port
- User ID (UUID or custom string)
- Transport protocol (e.g., TCP, WebSocket)
- TLS/XTLS settings for encryption
