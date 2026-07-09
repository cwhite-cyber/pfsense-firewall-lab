# pfSense Firewall Lab

Network segmentation and firewall policy build on top of an existing home lab (VirtualBox, Ubuntu Server, LibreNMS, Kali, Graylog). This project adds a pfSense firewall as the gateway between a simulated "outside" network and an internal lab segment, with default-deny inbound rules and log forwarding to Graylog.

> Status: 🚧 In progress — build log and troubleshooting notes below are updated as I go. Only errors I actually hit during this build are documented; nothing here is copy-pasted from someone else's writeup.

## Why this project

Monitoring (LibreNMS) tells you what's happening on the network. A SIEM (Graylog) tells you what happened. A firewall is the piece that actually *enforces* policy — deny by default, allow only what's needed. This project closes that loop: pfSense segments the lab, restricts what can talk to what, and forwards logs into Graylog so firewall events are visible in the SIEM alongside auth logs and other telemetry.

## Goals

- [ ] Stand up pfSense as a two-NIC gateway (WAN via NAT, LAN via host-only)
- [ ] Segment existing lab VMs behind the LAN interface
- [ ] Implement default-deny inbound rules; allow only explicitly needed services
- [ ] Forward pfSense logs to Graylog (`graylog01`)
- [ ] (Stretch) Add Suricata/Snort for IDS/IPS on the WAN interface
- [ ] Diagram the full lab architecture with pfSense as the segmentation point

## Lab architecture

*(Diagram goes here once the topology is finalized — see `docs/architecture.md`)*

**Planned topology:**

| Component | Role | Network |
|---|---|---|
| pfSense | Firewall / gateway | WAN: NAT · LAN: host-only |
| `theone` (Ubuntu Server, LibreNMS) | Monitored host | LAN (host-only) |
| `graylog01` (Graylog SIEM) | Log aggregation target | LAN (host-only) |
| Kali | Attack/test box | LAN (host-only), used to validate rules |

## Build log

| Date | Phase | Notes |
|---|---|---|
| | VM creation + pfSense install | |
| | Interface config (WAN/LAN) + DHCP on LAN | |
| | Inbound rule policy (default-deny) | |
| | Graylog log forwarding | |
| | IDS/IPS package (stretch) | |

## Troubleshooting

Real errors hit during this build, in the order encountered. See `docs/troubleshooting.md` for full detail (symptom, cause, fix, and how it was verified).

*(Empty for now — populated as issues come up during the actual build.)*

## Security+ concepts demonstrated

- Network segmentation / zoning
- Default-deny (implicit deny) rule design
- NAT vs routed interfaces
- Defense in depth (firewall + SIEM + monitoring working together)
- (Stretch) IDS/IPS signature-based detection

## Related lab projects

- [LibreNMS Network Monitoring Lab](#) — link once published
- [Graylog SIEM Build](#) — link once published

## Disclosure

This lab was built with AI assistance (Claude) for guidance, troubleshooting support, and documentation structure. All commands were run and all errors documented here were personally encountered and reproduced in this environment.
