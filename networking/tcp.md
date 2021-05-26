# TCP

## 1. TCP 3-way handshake

![](assets/tcp-handshake.png)

Step 1: The client sends a message with the SYN flag = 1 + random sequence number (SEQ = x) to the server.

Step 2: After receiving the client's synchronization request, the server sends a message with the SYN flag = 1, ACK flag = 1, Ack number = Client's sequence number + 1 (x + 1) and server random sequence number (SEQ = y).

Step 3: After receiving the SYN from the server, the client sends a message to server with ACK flag = 1, Ack number = Server sequence number + 1 (y + 1), SEQ = x + 1

- The first two handshakes SYN packets cannot carry data. They are using to establish the sequence number for both client and server.
- The third handshake to make sure that the client already received the SYN-ACK message from the server and it can carry data.
