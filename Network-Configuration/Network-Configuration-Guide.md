

# 📘 Network Configuration Guide (Linux Servers)

This document provides professional reference templates for configuring a **static IP address** on different Linux distributions.
It covers Ubuntu (Netplan), Debian (`/etc/network/interfaces`), CentOS (`ifcfg`), and Rocky/AlmaLinux (`nmconnection`).
Each template includes optional advanced settings (commented).

---

## 🔹 Ubuntu 20.04 / 22.04 / 24.04 (Netplan)

**File:** `/etc/netplan/01-netcfg.yaml`

```yaml
network:
  version: 2
  renderer: networkd   # use NetworkManager on desktop

  ethernets:
    ens160:                      # replace with your NIC name
      dhcp4: no
      dhcp6: no
      addresses:
        - 192.168.1.10/24        # static IP
      gateway4: 192.168.1.1      # default gateway
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]

      # --- Optional advanced settings ---
      # routes:
      #   - to: 10.10.0.0/16
      #     via: 192.168.1.254
      #     metric: 100
      # mtu: 1500
      # wakeonlan: true

  # vlans:
  #   vlan10:
  #     id: 10
  #     link: ens160
  #     addresses: [192.168.10.5/24]

  # bonds:
  #   bond0:
  #     interfaces: [ens161, ens162]
  #     parameters:
  #       mode: active-backup
  #       mii-monitor-interval: 100
  #     addresses: [192.168.20.5/24]
  #     gateway4: 192.168.20.1

  # bridges:
  #   br0:
  #     interfaces: [ens163]
  #     addresses: [192.168.30.5/24]
  #     gateway4: 192.168.30.1
```

**Apply changes:**

```bash
sudo netplan apply
# or safer test mode:
sudo netplan try
```

---

## 🔹 Debian 10 / 11 / 12 (interfaces)

**File:** `/etc/network/interfaces`

```bash
# Loopback
auto lo
iface lo inet loopback

# Static configuration
auto ens160
iface ens160 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1

    # --- Optional advanced settings ---
    # mtu 1500
    # up /sbin/ethtool -s $IFACE wol g
    # dns-search example.local

# DHCP example
# auto ens160
# iface ens160 inet dhcp
```

**Apply changes:**

```bash
sudo systemctl restart networking
# or
sudo ifdown ens160 && sudo ifup ens160
```

---

## 🔹 CentOS 7 (ifcfg format)

**File:** `/etc/sysconfig/network-scripts/ifcfg-ens160`

```bash
TYPE=Ethernet
BOOTPROTO=none         # none=static, dhcp=automatic
NAME=ens160
DEVICE=ens160
ONBOOT=yes

# Static configuration
IPADDR=192.168.1.10
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=1.1.1.1

# --- Optional advanced settings ---
# MTU=1500
# ETHTOOL_OPTS="wol g"
# DOMAIN="example.local"
```

**Static routes (if needed):**
Create `/etc/sysconfig/network-scripts/route-ens160`

```bash
10.10.0.0/16 via 192.168.1.254 metric 100
```

**Apply changes:**

```bash
sudo systemctl restart network
```

---

## 🔹 Rocky Linux / AlmaLinux (RHEL 8/9, nmconnection)

**File:** `/etc/NetworkManager/system-connections/ens160.nmconnection`

```ini
[connection]
id=ens160
uuid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   # use `uuidgen`
type=ethernet
interface-name=ens160
autoconnect=true

[ipv4]
method=manual
addresses=192.168.1.10/24
gateway=192.168.1.1
dns=8.8.8.8;1.1.1.1;
ignore-auto-dns=true
# routes=10.10.0.0/16 192.168.1.254 100
# mtu=1500

[ipv6]
method=ignore

[ethernet]
# wake-on-lan=1
# mtu=1500

[proxy]
```

**Apply changes:**

```bash
nmcli connection reload
nmcli connection up ens160
```

---

## 📊 Quick Comparison

| Distro           | Config file path                                            | Tool to apply changes          |
| ---------------- | ----------------------------------------------------------- | ------------------------------ |
| Ubuntu (20+)     | `/etc/netplan/*.yaml`                                       | `netplan apply`                |
| Debian (10–12)   | `/etc/network/interfaces`                                   | `systemctl restart networking` |
| CentOS 7         | `/etc/sysconfig/network-scripts/ifcfg-<nic>`                | `systemctl restart network`    |
| Rocky/Alma (8/9) | `/etc/NetworkManager/system-connections/<nic>.nmconnection` | `nmcli connection reload/up`   |

---

✅ With these templates, you can configure a **professional and extendable network setup** on any major Linux distribution.
Each section comes with active static config and optional advanced settings (commented).

