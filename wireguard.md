# WireGuard

## Overview

WireGuard creates a private tunnel between the VPS and the local machine. All traffic proxied by VPS nginx travels through this tunnel to reach services on the local machine.

---

## Peers

| Peer | Role |
|------|------|
| VPS | Entry point, receives public traffic, forwards via tunnel |
| Local machine | Runs all services, receives forwarded traffic |
| Piotr's machine | Additional peer, direct access to local network |

---

## Network

| Property | Value |
|----------|-------|
| WireGuard interface | `wg0` |
| Listen port | `51820` |
| Subnet | `10.0.0.0/24` |
| VPS | `10.0.0.1` |
| Local machine | `10.0.0.2` |
| Piotr's machine | `10.0.0.3` |

---

## VPS Config Template

```
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <server_private_key>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Local machine
PublicKey = <local_machine_public_key>
AllowedIPs = 10.0.0.2/32

[Peer]
# Piotr's machine
PublicKey = <piotr_public_key>
AllowedIPs = 10.0.0.3/32
```

## Local Machine Config Template

```ini
[Interface]
PrivateKey = <local_machine_private_key>
Address = 10.0.0.2/24

[Peer]
# VPS
PublicKey = <vps_public_key>
Endpoint = 178.104.50.250:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

`PersistentKeepalive = 25` keeps the tunnel alive through NAT — necessary since the local machine has no public IP.

---

## IP Forwarding & NAT (VPS only)

The VPS config includes `PostUp` and `PostDown` iptables rules that run when the WireGuard interface comes up or goes down:

```
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

**What these do:**

- `FORWARD -i wg0 -j ACCEPT` — allows packets arriving on the WireGuard interface to be forwarded to other interfaces (i.e. out to the internet or back to nginx)
- `POSTROUTING -o eth0 -j MASQUERADE` — rewrites the source IP of outgoing packets on `eth0` to the VPS public IP, so return traffic is routed back correctly

**Why this is needed:**

Without these rules, the Linux kernel drops forwarded packets by default. Traffic coming from the tunnel (10.0.0.x) would never reach nginx or get a response routed back. These rules make the VPS act as a NAT gateway for the WireGuard subnet.

IP forwarding must also be enabled at the kernel level:

```bash
# Check current value (should be 1)
sysctl net.ipv4.ip_forward

# If not set, add to /etc/sysctl.conf:
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Note: IP forwarding must also be enabled at the kernel level (`net.ipv4.ip_forward=1` in `/etc/sysctl.conf`).

---

## Config Location

| Machine | Path |
|---------|------|
| VPS | `/etc/wireguard/wg0.conf` |
| Local machine | `/etc/wireguard/wg0.conf` |

VPS key files:

| File | Purpose |
|------|---------|
| `/etc/wireguard/server_private.key` | VPS private key |
| `/etc/wireguard/server_public.key` | VPS public key (share with peers) |

To regenerate keys and update the config:

```bash
# Generate new keypair
wg genkey | tee /etc/wireguard/server_private.key | wg pubkey > /etc/wireguard/server_public.key
chmod 600 /etc/wireguard/server_private.key

# Print the new private key to paste into wg0.conf
cat /etc/wireguard/server_private.key

# Print the new public key to share with peers
cat /etc/wireguard/server_public.key
```

After regenerating, update `PrivateKey` in `/etc/wireguard/wg0.conf` with the new private key, then update the VPS public key on all peers and restart the tunnel:

```bash
sudo systemctl restart wg-quick@wg0
```

---

## Useful Commands

```bash
# Check tunnel status
sudo wg show

# Bring tunnel up/down
sudo wg-quick up wg0
sudo wg-quick down wg0

# Check if service is enabled on boot
sudo systemctl is-enabled wg-quick@wg0
```
