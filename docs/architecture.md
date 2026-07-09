# Lab Architecture

## Network diagram

(Add diagram here - draw.io, Excalidraw, or hand-drawn plus scanned all work fine. Export as PNG or SVG into screenshots/ and embed below.)

```
[ Internet / NAT ]
        |
     [ WAN ]
   +---------+
   | pfSense |
   +---------+
     [ LAN ]  --- host-only 192.168.56.x
        |
   -----------------------------
   |          |                |
[theone]  [graylog01]      [Kali]
LibreNMS   SIEM             attack/test box
```

## Interface plan

| Interface | Type | Purpose |
|---|---|---|
| WAN | NAT | Simulates internet-facing connection |
| LAN | Host-only (192.168.56.x) | Internal lab segment, all existing VMs live here |

## Rule design philosophy

Default-deny inbound. Every allow rule should have a documented reason tied back to a specific service/host, not a blanket allow. Example target state:

- Allow: Kali to theone on SSH (22) only
- Allow: theone to graylog01 on syslog/GELF port
- Deny: everything else inbound to LAN from WAN
- Log: denied connections forwarded to Graylog for visibility
