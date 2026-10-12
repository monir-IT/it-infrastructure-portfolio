# Moniruzzaman Himel: IT Infrastructure Portfolio

IT Infrastructure & Network Engineer in Riyadh, Saudi Arabia. I am the only IT engineer for a multi-brand restaurant group, responsible for its head office and branch networks.

This repository documents the systems I have designed, built and fixed there: what the problem was, what I built, what went wrong, and how I diagnosed it.

**Currently:** studying for AWS Solutions Architect – Associate and building Linux skills, working toward cloud security engineering.

## Projects

| # | Project | Highlights |
| --- | --- | --- |
| 01 | [Internal VoIP phone system](01-grandstream-voip/README.md) | Grandstream PBX, Wi-Fi handsets, isolated voice network, IP planning |
| 02 | [Door access control and video entry](02-access-control-door-entry/README.md) | SIP door station, RFID, remote release, lock wiring, fail-safe design |
| 03 | [CCTV surveillance](03-cctv-surveillance/README.md) | Two recorders, 40+ cameras, dual-NIC isolation that fixed staff network drops |
| 04 | [Switching, VLANs and wireless](04-switching-and-vlans/README.md) | Eight VLANs, five managed switches from three vendors, SSID-to-VLAN mapping |
| 05 | [OPNsense firewall](05-opnsense-firewall/README.md) | Eight routers replaced by one firewall, default-deny zones, NAT and CGNAT troubleshooting, Tailscale |
| 06 | [Firewall migration to bare metal](06-firewall-migration-bare-metal/README.md) | VM to appliance, config restore, lockout recovery, rollback planning |

## How the network grew

| Phase | Before | After |
| --- | --- | --- |
| **One** | No internal calling | Nine Wi-Fi phones on an isolated voice network |
| **Two** | Anyone could walk in; no cameras; one router per service | Door entry with RFID and remote release, CCTV, two cabinets and the first VLANs |
| **Three** | Eight separate consumer routers; 40+ cameras on a staff internet router | One OPNsense firewall, eight VLANs, five managed switches, managed Wi-Fi and three more phones |

## Skills

- **Networking:** VLANs and 802.1Q trunking, subnetting, DHCP and DNS, NAT, managed Wi-Fi, PoE, structured cabling
- **Security:** firewall policy design, network segmentation, least-privilege access between zones, captive portal, RADIUS, physical access control, CCTV
- **Systems:** OPNsense, Hyper-V, Windows, PowerShell, Grandstream UCM, Hikvision, Ruijie Cloud, Tailscale
- **Practice:** root-cause analysis, change and rollback planning, technical documentation

## Learning

- AWS Solutions Architect – Associate (in progress)
- Linux fundamentals (in progress)

## Contact

LinkedIn: *add your profile link here*

---

*Employer name, IP addresses, device names, credentials and floor plans are deliberately left out of this repository.*
