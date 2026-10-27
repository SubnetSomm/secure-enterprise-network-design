# Part 1 — Network design

**Architecture proposal · ITN 263 · Summer 2026 · Franco Pisano**

This is a course design scenario. Proposed controls, rollout actions and success measures are not evidence of a deployed production network. Vendor references are preserved as part of the historical submission, not a current procurement recommendation.

Corporation Techs Network Design

Project Part 1: Secure, Available, and Scalable Infrastructure

Franco Pisano | ITN 263 | August 2, 2026

Purpose and Design Recommendation

Corporation Techs operates one public Linux/Apache web server, six internal Windows servers, and 50 Windows workstations behind one border firewall. This flat design creates weak internal separation and single points of failure. Because most technical representatives will soon work from home and client-facing staff will double, the network must protect internal systems, preserve public website access, and scale to approximately 100 workstations.

The recommended design uses a redundant firewall pair, a demilitarized zone (DMZ) for the public server, redundant Layer 3 core switches, and virtual local area networks (VLANs) grouped by function. NIST describes segment-based designs as grouping resources with similar security requirements and controlling communication between those groups (Chandramouli, 2022). New employees can be placed in the appropriate VLAN without changing the security boundaries.

Physical Topology

Two independent Internet service provider (ISP) circuits terminate at high-availability firewalls. Each firewall connects to both core switches, which use redundant inter-switch links. The public server moves to the DMZ; the application, database, and file/print servers remain protected internally. Critical devices use uninterruptible power supplies and secure configuration backups. Redundancy avoids single points of failure by providing multiple paths for mission-critical work (Stewart & Kinsey, 2021).

Logical Topology and Internal Access

The logical design separates systems that share physical switches. VLAN 10 contains the public server in the DMZ; VLANs 20 and 30 contain Accounting and Sales; VLAN 40 contains other users; VLAN 50 contains the six internal servers; and VLAN 60 contains management systems. Each user VLAN receives a separate subnet and DHCP scope. Part 3 will add a VPN-user VLAN.

Deny-by-default rules control inter-VLAN traffic. Accounting and Sales may reach only approved applications and shares, neither department may initiate traffic to the other, users cannot connect directly to databases, and device management is limited to VLAN 60. CISA explains that segmentation hinders lateral movement and that a DMZ separates the internal network from untrusted networks (Cybersecurity and Infrastructure Security Agency [CISA], 2021, 2022).

Maintaining 24/7 Communications

The ISP circuits should use different providers and physical paths when possible. Active/standby firewalls synchronize policies and fail over automatically; redundant core switches prevent one device or link from isolating services. NIST similarly recommends redundant routing devices so one can assume the other’s workload (Swanson et al., 2010).

Continuous monitoring should cover ISP links, firewalls, switches, servers, and the website. Secure backups, documented recovery steps, vendor support, data-restoration tests, and scheduled failover exercises complete the plan. The initial target is at least 99.9% monthly availability for critical services, excluding approved maintenance.

IPv4 and IPv6 Recommendation

Corporation Techs should retain private IPv4 for this project while requiring all new equipment to support IPv6. Private IPv4 provides enough addresses for planned growth, whereas an immediate IPv6-only conversion would add training, configuration, monitoring, and compatibility work without solving the urgent segmentation and availability problems.

The company should inventory IPv6 capability, disable uncontrolled IPv6 traffic, train administrators, and later test dual stack in one VLAN with equivalent security policies. NIST warns that IPv6 migration can alter the security posture and requires planning and testing (Frankel et al., 2010). This staged approach preserves stability without preventing future migration.

High-Level Implementation Plan

First, document current addressing, connections, traffic flows, and service dependencies. Install and test redundant perimeter and core devices alongside the existing network. Next, create the VLANs and migrate management systems, servers, Accounting, Sales, and other users in controlled phases. Move the public server to the DMZ and validate both external HTTPS and blocked DMZ-to-LAN initiation. Finally, test ISP, firewall, switch, power, backup, and restoration procedures before acceptance. Parts 2 and 3 will add specific firewall, authentication, and VPN selections to this foundation.

Figure 1

Proposed Corporation Techs Network Topology

Note. The diagram shows the current servers and workstation population placed within the proposed redundant and segmented design. The public web server is the only server directly reachable from the Internet, and only through approved HTTPS rules.

Validation and Success Measures

| Control | Acceptance evidence |
| --- | --- |
| Segmentation | Approved inter-VLAN paths work and all unapproved paths are blocked. |
| Public website | HTTPS works externally; the DMZ host cannot initiate internal connections. |
| Redundancy | An ISP, firewall, or core-switch failure causes no unacceptable outage. |
| Capacity | Infrastructure supports about 100 workstations plus remote users. |
| Operations | Monitoring, backups, recovery procedures, and failover tests have owners. |

## References

Chandramouli, R. (2022). Guide to a secure enterprise network landscape (NIST Special Publication 800-215). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-215

Cybersecurity and Infrastructure Security Agency. (2021). Website security. https://www.cisa.gov/news-events/news/website-security

Cybersecurity and Infrastructure Security Agency. (2022). Layering network security through segmentation. https://www.cisa.gov/sites/default/files/publications/layering-network-security-segmentation_infographic_508_0.pdf

Frankel, S., Graveman, R., Pearce, J., & Rooks, M. (2010). Guidelines for the secure deployment of IPv6 (NIST Special Publication 800-119). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-119

Stewart, J. M., & Kinsey, D. (2021). Network security, firewalls, and VPNs (3rd ed.). Jones & Bartlett Learning.

Swanson, M., Bowen, P., Phillips, A. W., Gallup, D., & Lynes, D. (2010). Contingency planning guide for federal information systems (NIST Special Publication 800-34 Rev. 1). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-34r1
