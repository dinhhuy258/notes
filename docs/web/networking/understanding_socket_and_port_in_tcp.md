# Understanding socket and port in TCP

## What defines a unique connection ?

There are 5 elements that identify a connection:

1. Protocol (TCP/ UDP...)
2. Source IP address
3. Source port
4. Target IP address
5. Target port

## Does TCP listen on one port and talk on another port ?

No. TCP listens on 1 port and talk on that same port. If clients make multiple TCP connection to server. It’s the client OS that must generate different random source ports, so that server can see them as unique connections

## What is the maximum number of concurrent TCP connections that a server can handle, in theory ?

A single listening port can accept more than one connection simultaneously

If a client has many connections to the same port on the same destination, then four elements are the same (1) (2) (4) (5). Only source port varies to differentiate the different connections.
Port are 16-bit numbers, therefore the maximum number of connections any given client can have to any given host port is 64K.

However, multiple clients can each have up to 64K connections to some server’s port, and if the server has multiple ports or either is multi-homed then you can multiply that further

So the real limit is **file descriptors**. Each individual socket connection is given a file descriptor, so the limit is really the number of file descriptors that the system has been configured to allow and resources to handle. The maximum limit is typically up over 300K, but is configurable e.g. with sysctl.

## Concurrent connection request vs Concurrent open connection

When clients want to make TCP connection with server, this request will be queued in server ‘s backlog queue. This backlog queue size is small (about 5–10), and this size limits the number of concurrent connection requests. However, server quickly pick connection request from that queue and accept it. Connection request which is accepted are called open connection. The number of concurrent open connections is limited by server ‘s resources allocated for file descriptor.

## Why connection is rejected after successful TCP handshake ?

When server receive connection request from client (by receiving SYN), it will then response with SYN, ACK, hence cause successful TCP handshake. But this request are stills in backlog queue.

If the application process exceeds the limit of max file descriptors it can use, then when server calls accept, then it realizes that there are no file descriptors available to be the allocated for the socket and fails the accept call and the TCP connection sending a FIN to other side.

## What are active and passive socket ?

Sockets come in two primary flavors:

- An active socket is connected to a remote active socket via an open data connection…
- A passive socket is not connected, but rather awaits an incoming connection, which will spawn a new active socket once a connection is established

Each port can have a single passive socket binded to it, await­ing in­com­ing con­nec­tions, and mul­ti­ple active sockets, each cor­re­spond­ing to an open con­nec­tion on the port.
