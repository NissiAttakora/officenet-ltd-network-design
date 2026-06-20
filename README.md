# OfficeNet Ltd — Enterprise Network Design

A complete, secure small-business network designed and implemented in Cisco Packet Tracer — built as a hands-on CCNA portfolio project covering hierarchical network design, VLAN segmentation, inter-VLAN routing, wireless deployment, perimeter security, and centralised services.

## Network Architecture

```
Internet
   │
Router (ISR4331) — NAT/PAT, WAN edge
   │
ASA 5506-X Firewall — stateful inspection, security zones
   │
Multilayer Switch (3560-24PS) — Layer 3 core, inter-VLAN routing
   │
Access Switch (2960-24TT) — Layer 2 access layer
   │
End devices, servers, dual access points
```

![Network Topology](diagrams/topology.png)

A three-tier hierarchical design (core, distribution/access, edge) was used throughout, separating routing responsibility (multilayer switch) from end-device connectivity (access switch) — the standard enterprise pattern at small scale.

## IP Addressing and Subnetting

Manually calculated from a `192.168.0.0/16` supernet, plus dedicated point-to-point transit links:

| Segment | Subnet | Purpose |
|---|---|---|
| VLAN 10 — HR | 192.168.10.0/27 | Staff (wired + wireless) |
| VLAN 20 — IT | 192.168.20.0/27 | Staff (wired) |
| VLAN 30 — Servers | 192.168.30.0/28 | DHCP/DNS server |
| VLAN 40 — Guest WiFi | 192.168.40.0/27 | Isolated guest internet access |
| VLAN 99 — Management | 192.168.99.0/28 | Out-of-band device administration |
| Router ↔ Firewall | 10.0.0.0/30 | WAN transit link |
| Firewall ↔ Core Switch | 192.168.98.0/30 | Internal transit link |

## Core Implementation

**Switching & VLANs**
- Five VLANs with 802.1Q trunking between access and core switches
- Native VLAN moved off VLAN 1 to a dedicated management VLAN (99)
- Port security (sticky MAC learning, violation restrict) on all end-device ports
- STP hardening via PortFast + BPDU Guard on access ports

**Routing**
- Inter-VLAN routing via SVIs on the multilayer switch
- DHCP relay (`ip helper-address`) forwarding broadcasts across VLAN boundaries to a centralised server

**Wireless**
- Dual SSID deployment (separate access points) — WPA2-PSK/AES on both
- Staff network bridged to VLAN 10; guest network isolated to VLAN 40

**Services**
- Centralised DHCP (per-VLAN scopes) and DNS (custom internal A records) on a single server

**Security**
- ACL-enforced guest network isolation, verified via live traffic testing and hit-counter analysis
- Stateful firewall with security-zone model (inside/outside, security-level 100/0) and Layer 7 application inspection (DNS message-length limiting, FTP/TFTP connection tracking)
- NAT/PAT for internet-bound traffic
- SSH-only remote management restricted to the management VLAN, encrypted passwords, console hardening, legal banners on every device

## Verification & Testing

- End-to-end ping testing confirming inter-VLAN routing for authorised traffic and successful isolation of guest traffic
- DNS resolution testing (`nslookup`) against internal records
- SSH connectivity testing across the management plane, with systematic hop-by-hop fault isolation when one path failed
- ACL hit-counter validation proving security rules were actively filtering live traffic, not just present in configuration

## Notable Engineering Decisions

A few decisions worth calling out specifically, since they reflect judgment rather than just following a checklist:

- **IP addressing conflict** — identified and corrected an address conflict between two devices sharing a subnet.
- **ACL placement re-evaluated, not just applied** — an ACL configured on the firewall would never have matched traffic given the network's actual routing path. Rather than leaving a rule that looked correct on paper but did nothing in practice, it was removed from the firewall and enforcement moved to the correct point in the topology (the core multilayer switch), where it could actually filter the traffic in question.
- **Hop-by-hop fault isolation** — when remote access to a specific device failed, the fault was diagnosed by testing connectivity at each hop in turn rather than guessing, which distinguished a genuine configuration fault from a simulator-specific limitation.

## Files in This Repository

```
officenet-ltd-network-design/
├── README.md
├── OfficeNet_Ltd.pkt              # Full Packet Tracer project file
├── configs/
│   ├── router-isr4331.txt         # Edge router running-config
│   ├── firewall-asa5506x.txt      # ASA firewall running-config
│   └── switch-access.txt          # Access switch running-config
└── diagrams/
    └── topology.png               # Network topology diagram
```

> **Note on configs:** all passwords, enable secrets, and hashed credentials have been redacted (`<redacted>`) before publishing. The configs are otherwise the real, unmodified device output.

## How to View

1. Download [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a NetAcad account)
2. Open `OfficeNet_Ltd.pkt` to explore the full interactive topology
3. Config files in `/configs` can be reviewed as plain text without Packet Tracer installed

## Skills Demonstrated

TCP/IP & Subnetting · VLANs & Trunking · Inter-VLAN Routing (SVI) · ACLs · DHCP Relay · Port Security · STP Hardening (PortFast/BPDU Guard) · Wireless (WPA2-PSK) · NAT/PAT · Stateful Firewall Configuration (ASA) · SSH Hardening · Network Troubleshooting & Fault Isolation

---

*Built as part of independent CCNA-aligned study, alongside Cisco Networking Academy coursework.*
