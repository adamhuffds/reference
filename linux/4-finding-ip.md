# Finding Your Home Server IP Address (Linux/Ubuntu)

A reference for locating your home server's local (private) and public (external) IP addresses on Ubuntu/Linux.

---

## Concepts

| Term | Description |
|------|-------------|
| **Local/Private IP** | The IP assigned to your server within your home network (e.g., `192.168.1.105`). Only reachable from devices on the same network. |
| **Public/External IP** | The IP your ISP assigns to your router. What the outside internet sees. Shared by all devices on your home network. |
| **Loopback (`127.0.0.1`)** | A special self-referencing address — always refers to the local machine itself, never appears on the network. |
| **Subnet (`/24`)** | Defines the range of addresses in your local network. `/24` means 256 possible addresses (e.g., `192.168.1.0` – `192.168.1.255`). |
| **Network Interface** | A physical or virtual network adapter. Common names: `eth0` (wired), `wlan0` (wireless), `enp3s0` (modern naming convention). |

---

## Find Your Local IP

### Quick (all IPs, no detail)

```bash
hostname -I
```

- `hostname` — prints the system's hostname and network info
- `-I` — prints all assigned local IP addresses, space-separated; excludes loopback (`127.0.0.1`)

**Example output:**
```
192.168.x.105 172.17.0.1
```

The first address is typically your primary LAN IP.

---

### Detailed (interface names + subnet info)

```bash
ip a
```

- `ip` — the modern Linux networking tool (replaces deprecated `ifconfig`)
- `a` — short for `address`; shows all network interfaces and their assigned IPs

**Example output (trimmed):**
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.x.105/24 brd 192.168.x.255 scope global eth0
```

Look for the `inet` line under your active interface. The number before the `/` is your IP.

> **Docs:** https://man7.org/linux/man-pages/man8/ip.8.html

---

### Filter to a specific interface

```bash
ip a show eth0
```

- `show eth0` — limits output to only the `eth0` interface (replace with your interface name)

---

## Find Your Public IP

Run this from the server:

```bash
curl ifconfig.me
```

- `curl` — transfers data from a URL via the command line
- `ifconfig.me` — a public web service that returns your public IP as plain text

**Silent version (no progress bar):**

```bash
curl -s ifconfig.me
```

- `-s` — "silent" mode; suppresses progress output so only the IP prints cleanly

**Alternatives if `ifconfig.me` is slow:**

```bash
curl -s icanhazip.com
curl -s api.ipify.org
```

> **Docs:** https://curl.se/docs/manpage.html

---

## Find Both at Once

```bash
echo "Local:  $(hostname -I)" && echo "Public: $(curl -s ifconfig.me)"
```

- `$(...)` — command substitution; runs the inner command and inserts its output inline
- `&&` — runs the second command only if the first succeeds
- `echo` — prints a string to the terminal

**Example output:**
```
Local:  192.168.x.105
Public: x.x.x.x
```

---

## Scan Your Network for the Server (from another machine)

If you're not at the server and need to find its IP from another device on the same network:

```bash
nmap -sn 192.168.x.0/24
```

- `nmap` — network mapper; a powerful network scanning tool
- `-sn` — "ping scan" only; disables port scanning so it just discovers live hosts (fast and low-noise)
- `192.168.x.0/24` — the CIDR range to scan; replace `192.168.x` with your actual subnet if different

**Find your subnet first:**

```bash
ip route | grep default
```

Look for something like `default via 192.168.x.1` — your subnet is `192.168.x.0/24`.

> **Docs:** https://nmap.org/book/man.html

---

## Make the IP Stick (Static IP via Netplan)

By default, your router assigns IPs via DHCP and they can change on reboot. To pin a static local IP on Ubuntu 20.04+:

Edit your Netplan config (usually in `/etc/netplan/`):

```yaml
network:
  version: 2
  ethernets:
    eth0:                          # replace with your interface name
      dhcp4: false
      addresses:
        - 192.168.x.105/24         # your desired static IP
      routes:
        - to: default
          via: 192.168.x.1         # your router's IP (gateway)
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

Apply the change:

```bash
sudo netplan apply
```

> **Docs:** https://netplan.readthedocs.io/en/stable/

---

## Quick Reference Cheatsheet

| Goal | Command |
|------|---------|
| Local IP (quick) | `hostname -I` |
| Local IP (detailed) | `ip a` |
| Public IP | `curl -s ifconfig.me` |
| Both at once | `echo "Local: $(hostname -I)" && echo "Public: $(curl -s ifconfig.me)"` |
| Scan network for server | `nmap -sn 192.168.x.0/24` |
| Find your subnet/gateway | `ip route \| grep default` |