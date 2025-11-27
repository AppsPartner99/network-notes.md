
-----------------------------
## 1. IP Formwarding
* IP forwarding means the Linux kernel is allowed to route packets between two or more network interfaces.
* It operates at Layer 3 (Network layer).

### Enable IP forwarding if you plan to route between two interface, e.g. Wi-Fi and Eth0
  ```bash
  echo 1 > /proc/sys/net/ipv4/ip_forward
  ```
  * Note: Firewall/NAT rules if internet sharing is needed (using iptables or nftables).
  * Add NAT Rules
  ```
  iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
  ```

### More info

<details><summary>(Click to expand) More info on IP forwarding </summary>

* IP forwarding means the Linux kernel is allowed to route packets between two or more network interfaces.
* It operates at Layer 3 (Network layer).
* When enabled (echo 1 > /proc/sys/net/ipv4/ip_forward), the device acts like a router:
  * Each interface keeps its own IP subnet.
  * Packets are forwarded based on routing tables.
* Often combined with NAT (iptables MASQUERADE) if one interface provides internet access.
* **Example use case:**
  * Your i.MX8DXL has Wi-Fi (10.0.0.x) and BroadR-Reach (192.168.100.x). If you enable IP forwarding, devices on BRR can reach the internet via Wi-Fi.


</details>

-----------------------------

## 2. Bridge network
### ✅ **What is a Bridge Network?**

*   A **bridge** in Linux combines two or more network interfaces into a single logical interface.
*   Devices connected to any of those interfaces appear to be on the **same Layer 2 network (same subnet)**.
*   All connected devices share the same subnet and appear as if they are on the same switch.
*   No routing or NAT is needed because it’s like a flat network.
*   The bridge acts like a virtual switch: packets can flow between `wlan0` and `eth0` without routing/NAT.

<details><summary>(Click to expand) More info on Bridge Network </summary>

* **Example use case**:
  * If you want Wi-Fi AP and BroadR-Reach devices to be in the same subnet (e.g., 192.168.10.x), you create a bridge br0 and attach wlan0 + eth0 to it.

### ✅ **How is Bridge Network Visible to the User?**

*   When you create a bridge (e.g., `br0`), the user sees:
    *   A new interface `br0` in `ip addr show`.
    *   `wlan0` and `eth0` become **slaves** to `br0`.
*   IP address and DHCP server are configured on `br0`, **not** on `wlan0` or `eth0`.

### ✅ **Combined Bridge Setup Example**

If you want Wi-Fi AP, Ethernt or BroadR-Reach in the same subnet (e.g., `192.168.10.x`):

#### **1. Create Bridge**

```bash
ip link add name br0 type bridge
ip link set br0 up
ip link set wlan0 master br0
ip link set eth0 master br0
```

#### **2. Assign IP to Bridge**

```bash
ip addr add 192.168.10.1/24 dev br0
```

#### **3. dnsmasq.conf**

```ini
interface=br0
dhcp-range=192.168.10.10,192.168.10.100,24h
dhcp-option=3,192.168.10.1
dhcp-option=6,8.8.8.8
bind-interfaces
```

#### **4. systemd-networkd**

Create `/etc/systemd/network/30-br0.netdev`:

```ini
[NetDev]
Name=br0
Kind=bridge
```

Create `/etc/systemd/network/40-br0.network`:

```ini
[Match]
Name=br0

[Network]
Address=192.168.10.1/24
```

And for `wlan0` and `eth0`:

```ini
[Match]
Name=wlan0

[Network]
Bridge=br0
```

```ini
[Match]
Name=eth0

[Network]
Bridge=br0
```

### ✅ **Pros & Cons**

*   **Pros:** Same subnet, easy communication between Wi-Fi and BRR devices.
*   **Cons:** More complexity, bridging Wi-Fi can be tricky (depends on driver support), and performance may drop.

</details>

-----------------------------

## ✅ **3. Key Differences**

| Feature    | IP Forwarding (Routing)   | Bridge Network (Switching)     |
| ---------- | ------------------------- | ------------------------------ |
| Layer      | L3 (Network)              | L2 (Data Link)                 |
| Subnets    | Different per interface   | Same subnet for all interfaces |
| Use case   | Internet sharing, routing | LAN extension, same network    |
| Complexity | Requires routing + NAT    | Requires bridge setup          |

-----------------------------
