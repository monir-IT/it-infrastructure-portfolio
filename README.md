# Moniruzzaman Himel: IT Infrastructure Portfolio

Network & Security Engineer in Riyadh, Saudi Arabia. I lead IT for a multi-brand restaurant group, responsible for its head office and branch networks, security and systems.

**Cisco Certified Specialist – Security Core** · Fortinet FortiGate 7.6 Operator · Fortinet NSE 3 · Google Cloud Cybersecurity

This repository documents the systems I have designed, built and fixed there: what the problem was, what I built, what went wrong, and how I diagnosed it.

**Currently:** extending my work into Azure: Entra ID identity security now, AZ-104 and SC-500 next, alongside Linux, toward cloud security engineering.

**Start here:** [How I built it: eight routers to one firewall](STORY.md), the story behind these projects and what I'd tell anyone doing this job alone.

## Projects

| # | Project | Highlights |
| --- | --- | --- |
| 01 | [Internal VoIP phone system](01-grandstream-voip/README.md) | Grandstream PBX, Wi-Fi handsets, isolated voice network, IP planning |
| 02 | [Door access control and attendance](02-access-control-door-entry/README.md) | SIP door station, fingerprint and RFID access, attendance, lock timing fault, fail-safe design |
| 03 | [CCTV surveillance](03-cctv-surveillance/README.md) | Two recorders, 40+ cameras, dual-NIC isolation that fixed staff network drops |
| 04 | [Switching, VLANs and wireless](04-switching-and-vlans/README.md) | Eight VLANs, five managed switches from three vendors, SSID-to-VLAN mapping |
| 05 | [OPNsense firewall](05-opnsense-firewall/README.md) | Eight routers replaced by one firewall (internet costs cut from 2,030 to 290 SAR a month), default-deny zones, NAT and CGNAT troubleshooting, Tailscale |
| 06 | [Firewall migration to bare metal](06-firewall-migration-bare-metal/README.md) | VM to appliance, config restore, lockout recovery, rollback planning |
| 07 | [Microsoft 365 and email security](07-microsoft-365-email-security/README.md) | Tenant security audit and hardening, PowerShell mailbox audit, lessons from a compromised tenant |

## How the network grew

| Phase | Before | After |
| --- | --- | --- |
| **One** | No internal calling | Nine Wi-Fi phones on an isolated voice network |
| **Two** | Anyone could walk in; no cameras; one router per service | Door entry with RFID and remote release, CCTV, two cabinets and the first VLANs |
| **Three** | Eight separate consumer routers; 40+ cameras on a staff internet router | One OPNsense firewall, eight VLANs, five managed switches, managed Wi-Fi, and 13 phones in total |

## In progress

I use my own production environment to learn: each item below is implemented at work, then documented here.

- [ ] Per-person Wi-Fi logins with FreeRADIUS and WPA2-Enterprise (testing)
- [x] Microsoft 365: emergency access accounts, admin role separation, app-consent control, mail-forwarding audit
- [ ] Microsoft 365: MFA for every account, SPF/DKIM/DMARC, domain protection
- [ ] Endpoint protection policies beyond the defaults
- [ ] Network monitoring on a Linux server
- [ ] Firewall and device backups to Azure Blob Storage
- [ ] Microsoft Sentinel monitoring for Microsoft 365 sign-ins and alerts
- [ ] The head-office design rebuilt in Azure with Terraform: VLANs as subnets, firewall rules as network security groups

## Skills

- **Networking:** VLANs and 802.1Q trunking, subnetting, DHCP and DNS, NAT, managed Wi-Fi, PoE, structured cabling
- **Security:** Cisco Security Core, firewall policy design, network segmentation, least-privilege access between zones, captive portal, RADIUS, physical access control, CCTV
- **Cloud and identity:** Azure / Entra ID, Microsoft 365 security, DNS and email authentication
- **Systems:** OPNsense, Hyper-V, Windows, PowerShell, Grandstream UCM, Hikvision, Ruijie Cloud, Tailscale
- **Practice:** root-cause analysis, change and rollback planning, technical documentation

## Certifications

| Certification | Issuer | Issued |
| --- | --- | --- |
| Cisco Certified Specialist – Security Core | Cisco | Jan 2026 |
| FortiGate 7.6 Operator | Fortinet | Jan 2026 |
| NSE 3 Certified in Cybersecurity | Fortinet | Jan 2026 |
| Google Cloud Cybersecurity Certificate | Google | Oct 2026 |
| Microsoft IT Support Specialist | Microsoft | Sep 2026 |
| Google IT Support | Google | Oct 2026 |

Credentials are verifiable on my [LinkedIn profile](https://www.linkedin.com/in/monir-himel/).

## Learning

- Microsoft Azure Administrator (AZ-104), then Cloud and AI Security Engineer (SC-500)
- Linux fundamentals (in progress)

## Contact

LinkedIn: [linkedin.com/in/monir-himel](https://www.linkedin.com/in/monir-himel/)

---

*Employer name, IP addresses, device names, credentials and floor plans are deliberately left out of this repository.*
