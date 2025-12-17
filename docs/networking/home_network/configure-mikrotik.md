# MikroTik Router Configuration

## Initial Setup & Security

### 1. Factory Reset

- `System` → `Reset Configuration`
- **Important**: Check `No Default Configuration` to start clean
- After reset, default credentials are:
  - Username: `admin`
  - Password: (empty)

![picsur.png](https://img.dinhhuy258.dev/i/587dc1f5-14bb-4b83-aa48-4a3278a8927d.jpg)

### 2. Secure Access (Do this before connecting to internet)

#### 2.1 Limit management services

- `IP` → `Services`
- Disable everything you don’t use
- Keep only `WinBox` enabled

![picsur.png](https://img.dinhhuy258.dev/i/eca6aa79-6fa9-4848-8382-db7c0428ac52.jpg)

#### 2.2 Change admin password / user

- `System` → `Users`
- Set a strong password (or create a new admin user and disable admin)

## Internet Connection

### 3. Create PPPoE client

- `PPP` → `New` → `PPPoE Client`
  - Interface: ether1
  - User / Password: from ISP

### 4. NAT (Masquerade)

- `IP` → `Firewall` → `NAT` → `New`
  - Chain: `srcnat`
  - Out. Interface: your PPPoE interface (e.g. `pppoe-out1`)
  - Action: `masquerade`

### 5. Verify Connectivity

Test internet connection in terminal:

```bash
ping 8.8.8.8
```

![picsur.png](https://img.dinhhuy258.dev/i/9e0340bf-2c47-4160-9c9e-0b80bed179b9.jpg)

## LAN Configuration

### 6. Create Bridge

- `Bridge` → `New`
  - Name: `bridge`

![picsur.png](https://img.dinhhuy258.dev/i/392d911f-7dc7-46ff-85b0-58599e7a2d13.jpg)

### 7. Add LAN ports to bridge

- `Bridge` → `Ports` → `New`
  - Add `ether2`, `ether3`, `ether4`, `ether5` to `bridge`

![picsur.png](https://img.dinhhuy258.dev/i/a1046d55-175a-44ef-a3dc-01969fd0d385.jpg)

### 8. Assign gateway IP to bridge

- `IP` → `Addresses` → `New`
  - Address: `192.168.0.1/24`
  - Interface: `bridge`

![picsur.png](https://img.dinhhuy258.dev/i/082ead58-28f6-4059-98b3-6ced736f58a4.jpg)

### 9. Set DNS servers

- `IP` → `DNS`
  - Servers: `1.1.1.1` (or your preferred DNS)

![picsur.png](https://img.dinhhuy258.dev/i/023ce334-5074-4320-9aaa-6c10de4a841d.jpg)

### 10. DHCP Server for LAN clients

- `IP` → `DHCP Server` → `DHCP Setup`
  - **DHCP Server Interface**: `bridge`
  - **DHCP Address Space**: `192.168.0.0/24`
  - **Gateway**: `192.168.0.1`
  - **Address Pool**: `192.168.0.20-192.168.0.254`
  - **DNS Server**: `1.1.1.1`
  - **Lease Time**: `24:00:00` (24 hours for home use)

### 11. Verify from a client PC

- Connect to `ether2` → `ether5`
- Test `ping 8.8.8.8`
