# Nmap

## Overview

**Nmap** (Network Mapper) is the tool for network discovery and security auditing. In a penetration test it answers two core questions, in order:

1. **Which hosts are alive?** — host discovery.
2. **What is each host running?** — open ports, services, versions, and OS.

A typical engagement runs in this order (the doc groups NSE and version detection together later, since they overlap):

```
Host discovery  →  Port scan  →  Service/version detection  →  NSE scripts  →  Vuln assessment
   (-sn)            (-sS/-sT/-sU)        (-sV)                   (-sC/--script)     (--script vuln)
```

## Host Discovery

Before scanning ports, Nmap usually checks whether the target is alive. This step is called **host discovery**.

Nmap does not rely on only one technique. It chooses from several probe types depending on:

- whether you run Nmap as root/admin,
- whether the target is on the same local network,
- whether the target is routed through another network or VPN,
- firewall behavior.

### Discovery methods

Each method proves a host is alive a different way. Request a specific one with its flag, or let Nmap choose (see the default-behaviour note below).

**ARP**

| Option | Method   | How it decides "up"                                                                                            |
| ------ | -------- | -------------------------------------------------------------------------------------------------------------- |
| `-PR`  | ARP ping | Sends an ARP request. Any ARP reply means the host is alive. This is the default on the same Ethernet segment. |

**ICMP**

| Option | Method                              | How it decides "up"                                                        |
| ------ | ----------------------------------- | -------------------------------------------------------------------------- |
| `-PE`  | ICMP echo request (type 8)          | The classic ping; an echo reply = up. Frequently blocked by firewalls.     |
| `-PP`  | ICMP timestamp request (type 13)    | Fallback when echo is filtered — some hosts answer timestamp but not echo. |
| `-PM`  | ICMP address-mask request (type 17) | Rarely supported, but occasionally slips past filters that drop echo.      |

**TCP**

| Option       | Method       | How it decides "up"                                                         |
| ------------ | ------------ | --------------------------------------------------------------------------- |
| `-PS<ports>` | TCP SYN ping | A `SYN/ACK` or `RST` response proves the host is alive. Default port: `80`. |
| `-PA<ports>` | TCP ACK ping | A `RST` response proves the host is alive. Default port: `80`.              |

**UDP**

| Option       | Method   | How it decides "up"                                                                                        |
| ------------ | -------- | ---------------------------------------------------------------------------------------------------------- |
| `-PU<ports>` | UDP ping | Sends a UDP datagram; an **ICMP port-unreachable** reply means the port is closed _but the host is alive_. |

**Other protocols**

| Option           | Method           | How it decides "up"                                                                |
| ---------------- | ---------------- | ---------------------------------------------------------------------------------- |
| `-PY<ports>`     | SCTP INIT ping   | Sends an SCTP INIT chunk; INIT-ACK or ABORT = up. For SCTP (telecom) environments. |
| `-PO<protocols>` | IP protocol ping | Sends packets with raw IP protocol numbers; any response = up.                     |

**Control flags** (govern discovery, not probes themselves)

| Option               | Effect                                                                          |
| -------------------- | ------------------------------------------------------------------------------- |
| `-sn`                | Host discovery only — run the probes, skip the port scan.                       |
| `-Pn`                | Skip discovery — assume every host is up and go straight to port scanning.      |
| `-sL`                | List scan — only lists / reverse-DNS resolves the targets, sends **no** probes. |
| `--disable-arp-ping` | Turn off the default ARP ping to force IP-layer probes locally.                 |

> [!NOTE]
> **What does a default `-sn` actually send?** It depends on privilege and where the target is:
>
> | Scenario                        | Probes sent                                                                                    |
> | ------------------------------- | ---------------------------------------------------------------------------------------------- |
> | On-link (same Ethernet segment) | **ARP only** — the ARP exchange is mandatory at L2, so Nmap uses it and skips everything else. |
> | Routed target, as root          | `-PE -PP -PS443 -PA80` — host is up if **any** one responds.                                   |
> | Routed target, unprivileged     | TCP `connect()` to ports 80 and 443 (can't craft raw packets).                                 |
>
> Locally, ARP **short-circuits** ICMP: an ARP reply marks the host up and the ICMP echo never goes out. ARP works only within a single broadcast domain — once the target is behind a router, Nmap falls back to its IP-layer probes.

> [!TIP]
> Full host-discovery strategy reference: <https://nmap.org/book/host-discovery-strategies.html>

### Specifying targets

A host-discovery scan always has the same shape — only the **target specification** changes:

```bash
sudo nmap <targets> -sn -oA <name> | grep for | cut -d" " -f5
```

| Target form  | Example                               | Meaning                                  |
| ------------ | ------------------------------------- | ---------------------------------------- |
| Single IP    | `10.129.2.18`                         | One host.                                |
| Multiple IPs | `10.129.2.18 10.129.2.19 10.129.2.20` | Space-separated list.                    |
| Octet range  | `10.129.2.18-20`                      | Consecutive addresses in the last octet. |
| CIDR range   | `10.129.2.0/24`                       | A whole subnet (256 addresses).          |
| From a file  | `-iL hosts.lst`                       | Read targets from a list, one per line.  |

| Option       | Description                                             |
| ------------ | ------------------------------------------------------- |
| `-sn`        | Disables port scanning (host discovery only).           |
| `-oA <name>` | Stores results in all formats with the given base name. |
| `-iL <file>` | Reads targets from the given file.                      |

**Sweep a subnet** (the most common case):

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

```
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
...
```

The `grep for | cut -d" " -f5` extracts just the live IPs from the `Nmap scan report for <ip>` lines.

**Scan from a host list** — during an internal test you're often handed a file of in-scope hosts. Feed it with `-iL`:

```bash
cat hosts.lst
# 10.129.2.4
# 10.129.2.10
# ...

sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

> [!WARNING]
> This only works if host firewalls permit the probes. If only 3 of 7 hosts respond, the rest may simply be ignoring ICMP echo requests due to firewall rules — Nmap gets no response and marks them inactive. For those, use the techniques covered in [Firewall and IDS/IPS Evasion](#firewall-and-idsips-evasion).

## Port Scanning

After host discovery, enumerate each live host's open ports and the services behind them — this is where you learn what's actually reachable. The information you're after:

- Open ports and their services
- Service versions
- Data the services expose
- Operating system

### Common options

The flags you'll reach for most, grouped by purpose. Mix and match — e.g. `sudo nmap <target> -sS -p- --open -oA scan`.

**Scan type** (what kind of probe)

| Option | Scan                                                       |
| ------ | ---------------------------------------------------------- |
| `-sS`  | TCP SYN / half-open. Default when run as root.             |
| `-sT`  | TCP connect, full handshake. Default unprivileged.         |
| `-sU`  | UDP scan.                                                  |
| `-sV`  | Service/version detection.                                 |
| `-sC`  | Run the default NSE scripts.                               |
| `-O`   | OS detection.                                              |
| `-A`   | Aggressive — `-sV` + `-O` + `--traceroute` + `-sC` in one. |

**Port selection** (which ports — applies to both TCP and UDP)

| Option          | Ports                                           |
| --------------- | ----------------------------------------------- |
| `-p 22,25,80`   | Specific ports (comma-separated).               |
| `-p 1-1000`     | A range.                                        |
| `-p-`           | All 65535 ports.                                |
| `-F`            | Fast — top 100 ports.                           |
| `--top-ports=N` | The N most frequent ports from Nmap's database. |

_With no `-p`/`-F`, Nmap scans the top 1000 ports by default._

**Quieting / debugging** (recur throughout the examples below)

| Option               | Effect                                                                                                                                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-Pn`                | Skip host discovery — scan the host even if it doesn't answer pings. Essential when a host blocks ICMP but still has open ports (the default would mark it "down" and skip the port scan entirely). |
| `-n`                 | Disable DNS resolution.                                                                                                                                                                             |
| `--disable-arp-ping` | Skip the local ARP ping.                                                                                                                                                                            |
| `--reason`           | Show **why** a port is in its state (which packet decided it).                                                                                                                                      |
| `--packet-trace`     | Show every packet sent/received.                                                                                                                                                                    |
| `--open`             | Only display open ports.                                                                                                                                                                            |
| `-oA <name>`         | Save output in all formats.                                                                                                                                                                         |

> [!TIP]
> A clean "show me exactly what's on the wire" combo is `--packet-trace --reason -n -Pn --disable-arp-ping` — it strips away discovery/DNS noise so you see only the port-scan packets.

### Port states

A scanned port can be in one of **six** states:

| State              | Meaning                                                                                                     |
| ------------------ | ----------------------------------------------------------------------------------------------------------- |
| `open`             | A connection was established (TCP connection, UDP datagram, or SCTP association).                           |
| `closed`           | The port replied with a TCP RST flag. Can also confirm a host is alive.                                     |
| `filtered`         | Nmap can't tell if the port is open or closed — no response, or an error code came back (often a firewall). |
| `unfiltered`       | Only seen in a TCP-ACK scan: the port is reachable, but open/closed is undetermined.                        |
| `open\|filtered`   | No response received. A firewall or packet filter may be protecting the port.                               |
| `closed\|filtered` | Only seen in IP ID idle scans: can't tell if closed or firewalled.                                          |

### TCP scanning

By default Nmap scans the **top 1000 TCP ports**. The technique depends on privileges:

- **As root** → SYN scan (`-sS`), aka _half-open_ scan (raw packets need socket privileges).
- **Unprivileged** → TCP connect scan (`-sT`).

> [!NOTE]
> The two differ in how far they take the TCP **three-way handshake**:
>
> - **`-sS` (SYN / half-open)** — sends `SYN`, reads the reply (`SYN-ACK` = open, `RST` = closed), then sends `RST` to **abort before completing the handshake**. The final `ACK` is never sent, so the connection is never fully established — quieter, and needs raw-packet privileges.
> - **`-sT` (Connect)** — completes the **full** `SYN → SYN-ACK → ACK` handshake (via the OS `connect()` syscall), then tears it down. No special privileges, but it fully connects, so it's more likely to be logged.

Scanning the top 10 ports:

```bash
sudo nmap 10.129.2.28 --top-ports=10
```

```
PORT     STATE    SERVICE
21/tcp   closed   ftp
22/tcp   open     ssh
25/tcp   open     smtp
80/tcp   open     http
110/tcp  open     pop3
139/tcp  filtered netbios-ssn
443/tcp  closed   https
445/tcp  filtered microsoft-ds
3389/tcp closed   ms-wbt-server
```

#### Tracing the SYN scan packets

To see a clean SYN scan, disable the noise: ICMP echo (`-Pn`), DNS resolution (`-n`), and ARP ping (`--disable-arp-ping`).

```bash
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping
```

```
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S  ttl=56 ... win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 ... win=0
21/tcp closed ftp
```

We sent a packet with the **SYN** flag (`S`); the target replied with **RST+ACK** (`RA`) — `ACK` acknowledges receipt, `RST` tears down the session → port closed.

**SENT line breakdown:**

| Field                                    | Meaning                 |
| ---------------------------------------- | ----------------------- |
| `SENT (0.0429s)`                         | Nmap sent a packet.     |
| `TCP`                                    | Protocol used.          |
| `10.10.14.2:63090 >`                     | Our source IP and port. |
| `10.129.2.28:21`                         | Target IP and port.     |
| `S`                                      | SYN flag.               |
| `ttl=... id=... seq=... win=... mss=...` | TCP header parameters.  |

**RCVD line breakdown:**

| Field              | Meaning                                 |
| ------------------ | --------------------------------------- |
| `RCVD (0.0573s)`   | A packet came back from the target.     |
| `10.129.2.28:21 >` | Target IP and source port of the reply. |
| `10.10.14.2:63090` | Our IP/port being replied to.           |
| `RA`               | RST + ACK flags.                        |

#### Filtered ports: dropped vs. rejected

A `filtered` result usually means a firewall. The firewall can **drop** or **reject** packets — and they look different on the wire.

**Dropped** — no response. Nmap retries (`--max-retries`, default 10), so the scan takes noticeably longer:

```bash
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn
```

```
SENT (0.0381s) TCP ... > 10.129.2.28:139 S ...
SENT (1.0411s) TCP ... > 10.129.2.28:139 S ...   # retry — note the 1s gap
139/tcp filtered netbios-ssn
# scanned in 2.06 seconds  ← much slower than ~0.05s
```

**Rejected** — the firewall replies with an ICMP unreachable:

```bash
sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn
```

```
SENT (0.0388s) TCP ... > 10.129.2.28:445 S ...
RCVD (0.0487s) ICMP [... Port 445 unreachable (type=3/code=3) ...]
445/tcp filtered microsoft-ds
```

> [!NOTE]
> ICMP **type 3 / code 3** = "port unreachable." If you already know the host is alive, a rejected port strongly implies a firewall rule — flag it for a closer look later.

### UDP scanning

UDP is **stateless** — no three-way handshake, no acknowledgment. Nmap sends empty datagrams and often gets nothing back, so timeouts are long and `-sU` is **much slower** than TCP.

```bash
sudo nmap 10.129.2.28 -F -sU
```

```
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
# scanned in 98.07 seconds  ← note the duration
```

| Option | Description          |
| ------ | -------------------- |
| `-sU`  | Performs a UDP scan. |
| `-F`   | Top 100 ports.       |

**How UDP states are inferred:**

| Observed response                       | State            |
| --------------------------------------- | ---------------- |
| UDP response from the application       | `open`           |
| ICMP port unreachable (type 3 / code 3) | `closed`         |
| Any other ICMP error / no response      | `open\|filtered` |

**Open** (an application that actually replies, e.g. NetBIOS):

```bash
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason
# RCVD (0.0398s) UDP 10.129.2.28:137 > ...
# 137/udp open netbios-ns udp-response ttl 64
```

**Closed** (ICMP port unreachable):

```bash
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason
# RCVD (0.1498s) ICMP [... Port unreachable (type=3/code=3) ...]
# 100/udp closed unknown port-unreach ttl 64
```

**open|filtered** (no response after retries):

```bash
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason
# SENT ... SENT (retry) ... no RCVD
# 138/udp open|filtered netbios-dgm no-response
```

## Nmap Scripting Engine (NSE)

NSE lets you run **Lua scripts** that interact with services for discovery, brute-forcing, vuln detection, and more. Scripts fall into **14 categories**:

| Category    | Description                                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------------------- |
| `auth`      | Determine authentication credentials.                                                                                      |
| `broadcast` | Host discovery via broadcast; discovered hosts can be auto-added to scans.                                                 |
| `brute`     | Brute-force login against services.                                                                                        |
| `default`   | Default set, run with `-sC`.                                                                                               |
| `discovery` | Enumerate accessible services.                                                                                             |
| `dos`       | Test for denial-of-service (can harm services — use sparingly).                                                            |
| `exploit`   | Attempt to exploit known vulnerabilities.                                                                                  |
| `external`  | Use external services for processing.                                                                                      |
| `fuzzer`    | Send malformed fields to find unexpected handling (slow).                                                                  |
| `intrusive` | May negatively affect the target.                                                                                          |
| `malware`   | Check for malware infection.                                                                                               |
| `safe`      | Defensive, non-intrusive scripts.                                                                                          |
| `version`   | Extends service detection — runs as part of a `-sV` scan (see [Service & Version Detection](#service--version-detection)). |
| `vuln`      | Identify specific vulnerabilities.                                                                                         |

### Running scripts

```bash
# Default scripts
sudo nmap <target> -sC

# A whole category
sudo nmap <target> --script <category>

# Specific named scripts (comma-separated)
sudo nmap <target> --script <script-name>,<script-name>,...
```

**Example — SMTP enumeration:**

```bash
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```

```
25/tcp open  smtp
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: inlane, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ...
```

The `banner` script reveals the OS (Ubuntu); `smtp-commands` lists supported commands (`VRFY` can help enumerate users).

### Aggressive scan (`-A`)

`-A` bundles several heavy options at once:

```bash
sudo nmap 10.129.2.28 -p 80 -A
```

```
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-title: blog.inlanefreight.com
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), ...
```

`-A` = `-sV` (service detection) + `-O` (OS detection) + `--traceroute` + `-sC` (default scripts).

### Vulnerability assessment (`--script vuln`)

```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```

```
80/tcp open  http  Apache httpd 2.4.29 ((Ubuntu))
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
| http-wordpress-users:
|   Username found: admin
| vulners:
|   cpe:/a:apache:http_server:2.4.29:
|       CVE-2019-0211   7.2 https://vulners.com/cve/CVE-2019-0211
|       CVE-2018-1312   6.8 https://vulners.com/cve/CVE-2018-1312
```

These scripts query the service and cross-reference vuln databases (e.g. `vulners`) for known CVEs.

> [!TIP]
> Script reference: <https://nmap.org/nsedoc/index.html>

## Service & Version Detection

`-sV` probes open ports to extract service names, versions, and other details.

```bash
sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason -sV
```

```
PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: Ubuntu
```

Under the hood `-sV` connects with `nsock`, sends a series of probes (`NULL`, then `SMBProgNeg`, etc.), and matches responses against its signature database to fingerprint the service.

| Option | Description                |
| ------ | -------------------------- |
| `-sV`  | Service/version detection. |

> [!NOTE]
> `-sV` and NSE overlap: the [`version` category](#nmap-scripting-engine-nse) runs **as part of** a `-sV` scan, and scripts like `banner` grab service banners too.

> [!TIP]
> More port scanning techniques: <https://nmap.org/book/man-port-scanning-techniques.html>

## Firewall and IDS/IPS Evasion

A **firewall** passes, drops, or rejects each packet by rule. An **IDS** passively watches traffic and alerts on attack patterns; an **IPS** acts on them — usually by blacklisting the source IP. Loud scans get you filtered or blocked, so the aim here is to slip probes past the rules and stay quiet.

### Drop vs reject

The distinction behind a `filtered` port (see [Port states](#port-states)):

- **Drop** — the packet is silently discarded. No reply, so Nmap retries and the scan crawls.
- **Reject** — the firewall answers with an explicit denial: a TCP `RST`, or an ICMP unreachable (net / host / port / proto unreachable or prohibited, all **type 3**). A response came back, so Nmap concludes faster.

### Why an ACK scan slips through (`-sA`)

A SYN scan (`-sS`) is exactly what stateful firewalls block — an inbound `SYN` is a new connection attempt. An **ACK scan** sends a bare `ACK`, which looks like it belongs to an _already established_ connection, so the firewall often lets it pass. It can't tell open from closed, but it does reveal **filtered vs not**:

| `-sA` result | Reply                   | Meaning                                         |
| ------------ | ----------------------- | ----------------------------------------------- |
| `unfiltered` | `RST`                   | the ACK reached the host — port is not filtered |
| `filtered`   | none / ICMP unreachable | the firewall is dropping packets here           |

Run `-sS` and `-sA` together: SYN gives open/closed, ACK gives filtered/unfiltered — between them you've mapped the firewall's rules.

### Evasion flags

| Flag                          | Purpose                                                                                                                                                                                              |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-sA`                         | ACK scan — map which ports the firewall filters vs lets through.                                                                                                                                     |
| `-D <ip>,RND:5,ME`            | **Decoys** — spoof extra source IPs so your real one hides in the crowd. `RND:5` = 5 random decoys, `ME` = your slot in the list. Decoys must be alive, or SYN-flood protections may break the scan. |
| `-S <ip>`                     | **Spoof the source IP** — handy when only certain subnets are allowed through. Replies go to the spoofed IP, so pair with `-e`.                                                                      |
| `-e <iface>`                  | Force the sending interface (e.g. `-e tun0`); required alongside `-S`.                                                                                                                               |
| `-g` / `--source-port <port>` | Send from a **trusted source port**. Firewalls often blanket-trust `53` (DNS) or `80`/`443` — `-g 53` can walk a probe straight through.                                                             |
| `--dns-server <ns>`           | Resolve via a chosen DNS server. In a DMZ, the org's own DNS servers are trusted and can reach internal hosts.                                                                                       |

### Examples

**Hide among decoys** — your real IP is mixed with five random ones:

```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -D RND:5
```

**Scan from a trusted source port** — a port that looks `filtered` from a random source…

```bash
sudo nmap 10.129.2.28 -p 50000 -sS -Pn          # 50000/tcp filtered
```

…often opens up when the probe appears to come from DNS:

```bash
sudo nmap 10.129.2.28 -p 50000 -sS -Pn -g 53    # 50000/tcp open
```

Once you know port `53` is trusted, reach the service directly with the same trick:

```bash
ncat -nv --source-port 53 10.129.2.28 50000
```

> [!TIP]
> To detect an **IPS**, scan aggressively from a single VPS. If that host suddenly loses all access to the target, an IPS blacklisted it — switch to another source IP and go quiet (slow timing, decoys, trusted source ports).

## Performance

Performance matters on large networks or low-bandwidth links. The trade-off is constant: **faster scans risk missing hosts and ports.**

| Option                         | Controls                                      |
| ------------------------------ | --------------------------------------------- |
| `-T <0-5>`                     | Overall timing template (aggressiveness).     |
| `--min-parallelism <n>`        | Minimum number of probes in parallel.         |
| `--initial-rtt-timeout <time>` | Starting round-trip timeout.                  |
| `--max-rtt-timeout <time>`     | Maximum round-trip timeout.                   |
| `--min-rate <n>`               | Minimum packets per second to send.           |
| `--max-retries <n>`            | Retransmissions per probed port (default 10). |

### Timeouts (RTT)

Nmap starts with a generous timeout (~100 ms). Tightening it speeds scans but can miss slow hosts:

```bash
# Default
sudo nmap 10.129.2.0/24 -F
# 256 IPs (10 hosts up) scanned in 39.44 seconds

# Optimized RTT
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
# 256 IPs (8 hosts up) scanned in 12.29 seconds  ← 2 fewer hosts, ¼ the time
```

> [!WARNING]
> Setting `--initial-rtt-timeout` too low can cause you to **overlook live hosts**.

### Max retries

```bash
sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l
# 23

sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l
# 21  ← faster, but 2 open ports missed
```

`--max-retries 0` skips a port the moment it doesn't respond.

### Rates

When you know the available bandwidth (e.g. a whitelisted white-box test), pin a send rate:

```bash
# Default
sudo nmap 10.129.2.0/24 -F -oN tnet.default
# scanned in 29.83 seconds

# Min rate 300 packets/sec
sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300
# scanned in 8.67 seconds
```

```bash
cat tnet.default     | grep "/tcp" | wc -l   # 23
cat tnet.minrate300  | grep "/tcp" | wc -l   # 23  ← same coverage, far faster
```

Raising the rate (when bandwidth allows) is one of the safest ways to speed up without losing results.

### Timing templates (`-T`)

For black-box tests where manual tuning isn't possible, use a template. Higher = more aggressive (and noisier — security systems may block you). Default is `-T 3`.

| Template | Name             |
| -------- | ---------------- |
| `-T 0`   | paranoid         |
| `-T 1`   | sneaky           |
| `-T 2`   | polite           |
| `-T 3`   | normal (default) |
| `-T 4`   | aggressive       |
| `-T 5`   | insane           |

```bash
# Default (-T 3)
sudo nmap 10.129.2.0/24 -F -oN tnet.default
# scanned in 32.44 seconds

# Insane (-T 5)
sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5
# scanned in 18.07 seconds
```

```bash
cat tnet.default | grep "/tcp" | wc -l   # 23
cat tnet.T5      | grep "/tcp" | wc -l   # 23
```

> [!TIP]
> Template details: <https://nmap.org/book/performance-timing-templates.html> · General performance: <https://nmap.org/book/man-performance.html>

## Quick Reference

| Goal                    | Command                                             |
| ----------------------- | --------------------------------------------------- |
| Ping sweep a subnet     | `sudo nmap 10.129.2.0/24 -sn -oA tnet`              |
| Scan from a host list   | `sudo nmap -sn -iL hosts.lst -oA tnet`              |
| Top 1000 TCP (SYN)      | `sudo nmap <target> -sS`                            |
| All TCP ports           | `sudo nmap <target> -p-`                            |
| Fast (top 100)          | `sudo nmap <target> -F`                             |
| UDP scan                | `sudo nmap <target> -sU`                            |
| Service/version         | `sudo nmap <target> -sV`                            |
| Default NSE scripts     | `sudo nmap <target> -sC`                            |
| Aggressive (all-in-one) | `sudo nmap <target> -A`                             |
| Vuln scan               | `sudo nmap <target> -p 80 -sV --script vuln`        |
| Save all formats        | `sudo nmap <target> -oA <name>`                     |
| See the packets         | `--packet-trace --reason -n -Pn --disable-arp-ping` |
