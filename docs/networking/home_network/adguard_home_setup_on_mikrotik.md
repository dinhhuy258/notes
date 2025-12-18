# AdGuardHome Setup on MikroTik

## Overview

This guide configures AdGuard Home as a DNS server with DNS-over-HTTPS (DoH) support, integrated with MikroTik router for network-wide ad blocking and DNS filtering.

**Architecture:**

- AdGuard Home runs behind Caddy reverse proxy (`192.168.0.5`)
- MikroTik router forwards DNS queries via DoH
- Network clients use router as DNS server (`192.168.0.1`)

## AdGuard Home Configuration

### 1. Enable DNS over HTTPS

Edit the AdGuard Home configuration file:

```bash
sudo vi /opt/AdGuardHome/AdGuardHome.yaml
```

Find the `dns` section and set `allow_unencrypted_doh` to `true`:

```yaml
dns:
  # Other settings...
  allow_unencrypted_doh: true
```

### 2. Restart AdGuard Home

```bash
sudo systemctl restart AdGuardHome
```

### 3. Verify service status

```bash
sudo systemctl status AdGuardHome
```

## MikroTik Router Configuration

### 4. Configure time synchronization

**Important**: Accurate time is required for HTTPS certificate validation.

- `System` → `Clock`
  - Time Zone Name: `Asia/Ho_Chi_Minh`
- `System` → `NTP Client`
  - Enable: `true`
  - NTP Servers: `time.google.com`, `pool.ntp.org`

```bash
/system clock set time-zone-name=Asia/Ho_Chi_Minh
/system ntp client set enabled=yes servers=time.google.com,pool.ntp.org
```

### 5. Add DNS static entry for AdGuard Home

Since AdGuard Home is behind a reverse proxy, add a static DNS entry:

- `IP` → `DNS` → `Static` → `New`
  - Name: `dns.dinhhuy258.dev`
  - Address: `192.168.0.5`
  - TTL: `1d`

```bash
/ip dns static add name=dns.dinhhuy258.dev address=192.168.0.5 ttl=1d
```

> [!NOTE]
> `192.168.0.5` is the Caddy reverse proxy IP address that forwards to AdGuard Home

### 6. Configure DNS over HTTPS

- `IP` → `DNS`
  - Use DoH Server: `https://dns.dinhhuy258.dev/dns-query`
  - Verify DoH Certificate: `Yes`

```bash
/ip dns set use-doh-server="https://dns.dinhhuy258.dev/dns-query" verify-doh-cert=yes
```

### 7. Update DHCP to use router DNS

Ensure DHCP clients use the router as their DNS server:

- `IP` → `DHCP Server` → `Networks`
  - DNS Servers: `192.168.0.1` (router gateway IP)

```bash
/ip dhcp-server network set 0 dns-server=192.168.0.1
```

## Client Configuration

### 8. Disable Chrome's built-in DoH

Chrome has its own DoH implementation that bypasses local DNS settings.

1. Navigate to `chrome://settings/security`
2. Disable: ✔ **Use secure DNS**
3. Click **Save**

## Verification

### 9. Test DoH functionality

#### 9.1 Test with Cloudflare DoH (temporary)

Temporarily change DNS server to test DoH is working:

```bash
/ip dns set use-doh-server="https://cloudflare-dns.com/dns-query"
```

Visit `https://one.one.one.one/help/` and verify:

- **Using DNS over HTTPS (DoH)**: `Yes`

#### 9.2 Restore AdGuard Home DoH

```bash
/ip dns set use-doh-server="https://dns.dinhhuy258.dev/dns-query"
```

#### 9.3 Check MikroTik DNS cache

- `IP` → `DNS` → `Cache`
  - Verify queries are being cached

```bash
/ip dns cache print
```

#### 9.4 Verify from client PC

From a connected client:

```bash
# Check DNS server
nslookup google.com

# Should show 192.168.0.1 as the server
```

### 10. Monitor AdGuard Home

Access AdGuard Home dashboard at `https://dns.dinhhuy258.dev` to verify:

- DNS queries are being received
- Ad blocking is working
- DoH connections are established
