# Firewall Implementation

After establishing a reliable network connection, I started building a custom firewall using nftables. Rather than relying on an existing firewall management tool, I wanted to understand how Linux processes network packets and how individual rules control incoming traffic.

## 1. Firewall Setup

The firewall was implemented directly on the Raspberry Pi using the native Linux packet filtering framework.

| Component | Specification |
|---|---|
| Hardware | Raspberry Pi 3 Model B |
| Operating System | Raspberry Pi OS Lite |
| Firewall | nftables |
| Version | 1.1.3 |
| Service Management | systemd |
| Protocol Support | IPv4 and IPv6 |

```bash
nft --version
```

![nftables Version](../images/06-nftables-version.png)

[View screenshot](../images/06-nftables-version.png)

## 2. Firewall Architecture

I created a dedicated `inet` table named `piwall`, allowing IPv4 and IPv6 traffic to be managed within a single configuration.

| Chain | Policy | Function |
|---|---|---|
| INPUT | DROP | Filters incoming traffic |
| OUTPUT | ACCEPT | Allows outgoing communication |
| FORWARD | ACCEPT | Reserved for gateway development |

The INPUT chain follows a default-deny approach, allowing essential network services while blocking traffic that does not match the permitted rules.

```bash
sudo nft list table inet piwall
```

![Firewall Rules](../images/07-firewall-rules.png)

[View screenshot](../images/07-firewall-rules.png)

## 3. Stateful Packet Filtering

Connection tracking was introduced to distinguish between existing connections, related traffic and invalid packets.

| Rule | Purpose |
|---|---|
| Loopback | Internal system communication |
| Established / Related | Maintain existing connections |
| Invalid | Discard invalid packets |
| TCP 22 | Allow SSH access |
| ICMP / ICMPv6 | Allow network control and diagnostic traffic |
| UDP 67/68 | Allow DHCPv4 responses |
| UDP 5353 | Allow multicast DNS |

DHCP and mDNS traffic are permitted specifically through the wireless interface.

[View complete firewall configuration](../config/piwall.nft)

## 4. Packet and Byte Counters

To observe firewall activity, I added a counter to the final drop rule.

```bash
sudo nft list chain inet piwall input
```

| Parameter | Recorded Value |
|---|---|
| Dropped Packets | 47 |
| Dropped Bytes | 1,692 |
| Final Action | DROP |

These values represent traffic reaching the final drop rule at the time of observation.

![Packet Counter](../images/08-packet-counter.png)

[View screenshot](../images/08-packet-counter.png)

## 5. Traffic Logging

Packet counters provide an overview of discarded traffic, but logging makes it possible to examine where that traffic originates.

Rate-limited logging was introduced to avoid excessive log entries.

| Parameter | Configuration |
|---|---|
| Average Logging Rate | 5 packets/second |
| Burst Allowance | 10 packets |
| Log Prefix | PiWall DROP: |
| Log Source | Linux kernel journal |

Recent firewall activity can be inspected using:

```bash
sudo journalctl -k -g "PiWall DROP:" --no-pager -n 10
```

The captured logs show recurring IGMP multicast traffic originating from the local router.

| Parameter | Observed Value |
|---|---|
| Incoming Interface | wlan0 |
| Source IP | 192.168.0.1 |
| Destination IP | 224.0.0.1 |
| Protocol | IGMP |
| Packet Size | 36 bytes |

This demonstrates that PiWall can record discarded network traffic. The observed multicast packets are normal network control traffic and do not necessarily indicate a security threat.

![Firewall Logging](../images/09-firewall-logging.png)

[View screenshot](../images/09-firewall-logging.png)

## 6. Firewall Persistence

During development, I encountered an issue where manually configured firewall rules were lost after a reboot.

To resolve this, I saved the configuration in `/etc/nftables.conf` and enabled the nftables service to automatically restore the rules during startup.

```bash
sudo systemctl status nftables --no-pager
```

| Parameter | Status |
|---|---|
| Service | Enabled |
| Runtime Status | Active (exited) |
| Execution Result | SUCCESS |
| Automatic Startup | Enabled |

![nftables Service](../images/10-nftables-service.png)

[View screenshot](../images/10-nftables-service.png)

## 7. Configuration Management

To prevent duplicate rules during configuration reloads, the existing PiWall table is replaced before recreating it.

```bash
destroy table inet piwall
```

The saved configuration can be inspected using:

```bash
sudo cat /etc/nftables.conf
```

![Persistent Configuration](../images/11-persistent-configuration.png)

[View screenshot](../images/11-persistent-configuration.png)

Before applying configuration changes, the syntax is validated:

```bash
sudo nft -c -f /etc/nftables.conf
```

The firewall can then be reloaded without rebooting the system:

```bash
sudo systemctl reload nftables
```

## Current Limitations

- Protection is currently limited to the Raspberry Pi itself.
- Outgoing and forwarded traffic remain unrestricted.
- SSH access is not restricted to specific source addresses.
- Packet counters reset when the firewall table is recreated.
- Packets discarded by earlier rules are not included in the final counter.

## Conclusion

Building PiWall gave me practical experience with Linux packet filtering, connection tracking, traffic monitoring and persistent system configuration. Troubleshooting the firewall also helped me understand the importance of maintaining connectivity while applying security policies.

The current implementation provides a working foundation for developing PiWall into a dedicated network security gateway.

## Related Documentation

- [Network Fundamentals](01-network-fundamentals.md)
- [Testing and Validation](03-testing-and-validation.md)
- [Firewall Configuration](../config/piwall.nft)
