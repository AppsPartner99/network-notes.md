

### ✅ **1. Adding DHCP for BroadR-Reach**

You can extend `dnsmasq.conf` like this:

    interface=wlan0
    dhcp-range=interface:wlan0,10.0.0.2,10.0.0.70,24h

    interface=eth0
    dhcp-range=interface:eth0,192.168.100.10,192.168.100.50,24h

**Key points:**

*   Use a different subnet for `eth0` to avoid conflicts (e.g., `192.168.100.x`).
*   Restart `dnsmasq` after changes:
    ```bash
    systemctl restart dnsmasq
    ```

***

### ✅ **2. Checklist**

*   **Static IP for your AP interface**: Assign a fixed IP to `wlan0` and `eth0` before starting `dnsmasq`. Example:
    ```bash
    ip addr add 10.0.0.1/24 dev wlan0
    ip addr add 192.168.100.1/24 dev eth0
    ```
*   **Enable IP forwarding** if you plan to route between Wi-Fi and BRR:
    ```bash
    echo 1 > /proc/sys/net/ipv4/ip_forward
    ```
*   **Firewall/NAT rules** if internet sharing is needed (using `iptables` or `nftables`).
*   **dnsmasq service enabled** and configured to start on boot.

***

### ✅ **Extra Considerations**

*   If you want **both DHCP servers active simultaneously**, ensure no overlapping subnets.
*   If BRR is connected to another ECU expecting DHCP, confirm timing and lease duration.
*   If you need **bridging** between Wi-Fi and Eth (BRR) (i.e. same subnet), then DHCP should run on the bridge interface instead.

***

👉 Summary

*   **Edit `/etc/dnsmasq.conf` for both interfaces**,
*   **Assign IP to the interface and start the service**,  

