# PiWall

**A Raspberry Pi-based firewall for exploring Linux networking, network security and traffic management.**

## Overview

I started PiWall to better understand how firewalls work beyond simply enabling security features on a router. The idea is to build my own firewall from the ground up, understand how network packets are processed and control how devices communicate.

Using a Raspberry Pi and nftables, I have implemented a stateful firewall with packet filtering, traffic logging and persistent configuration.

## Hardware & Software

| Component | Specification |
|---|---|
| Hardware | Raspberry Pi 3 Model B |
| Operating System | Raspberry Pi OS Lite |
| Firewall | nftables |
| Service Management | systemd |
| Remote Access | SSH |
| Network Interfaces | Wi-Fi / Ethernet |

## Current Features

- IPv4 and IPv6 packet filtering
- Stateful connection tracking
- Default-deny incoming traffic policy
- TCP and UDP traffic filtering
- Packet and byte counters
- Rate-limited firewall logging
- Persistent configuration across reboots
- Atomic configuration reloading

## Firewall Architecture

| Chain | Policy | Purpose |
|---|---|---|
| INPUT | DROP | Controls incoming traffic |
| OUTPUT | ACCEPT | Allows outgoing communication |
| FORWARD | ACCEPT | Reserved for future gateway development |

Incoming traffic is evaluated against predefined rules. Packets reaching the final drop rule are logged within the configured rate limit, counted and discarded.

## Testing & Validation

| Test | Result |
|---|---|
| IPv4 and IPv6 SSH connectivity | Passed |
| Unauthorized TCP port blocking | Passed |
| Internet connectivity | Passed |
| DNS resolution | Passed |
| Packet counter validation | Passed |
| Real-time firewall logging | Passed |
| Configuration reload | Passed |
| Reboot persistence | Passed |

## Project Status

PiWall currently operates as a standalone host-based firewall. The next stage focuses on routing and filtering traffic between separate networks, transforming it into a dedicated network security gateway.
