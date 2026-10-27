# Part 4 — Final network design report

**Architecture proposal · ITN 263 · Summer 2026 · Franco Pisano**

This is a course design scenario. Proposed controls, rollout actions and success measures are not evidence of a deployed production network. Vendor references are preserved as part of the historical submission, not a current procurement recommendation.

FINAL NETWORK DESIGN REPORT

Corporation Techs

Secure, Available, and Scalable Network Architecture

Prepared by: Franco PisanoRole: Junior Network ArchitectCourse: ITN 263 - Internet/Intranet Firewalls and E-Commerce SystemsDate: August 2, 2026

## Executive Summary

Corporation Techs provides 24/7 technical support and plans to move most representatives to home-based work while doubling its client-facing staff. Its current network - one public web server, six internal Windows servers, 50 Windows workstations, interconnected switches, and one aging border firewall - has weak internal separation and several single points of failure. The proposed design must protect company and client information, keep the public website and support services available, and scale toward approximately 100 workstations plus remote users.

The recommended architecture uses two Internet providers, two FortiGate 90G next-generation firewalls in active-passive high availability, redundant core switches, and separate security zones. The Linux/Apache website moves into a demilitarized zone (DMZ). VLANs separate Accounting, Sales, general users, internal servers, management systems, and remote VPN users. Default-deny rules permit only documented business traffic between zones, limiting lateral movement if a workstation or public server is compromised. Critical equipment receives redundant power, monitored UPS protection, secure backups, and scheduled failover testing.

Centralized Active Directory Domain Services replaces separate password-only access. Role-based groups, separate privileged accounts, multifactor authentication (MFA), device certificates, and 802.1X/RADIUS establish trust in users and devices. Windows Defender Firewall remains active on Windows systems through Group Policy, and the Linux host firewall permits public HTTPS and restricted management only. Logs from firewalls, authentication, VPNs, servers, and endpoints feed centralized monitoring.

Managed remote laptops will use IKEv2/IPsec through FortiClient and enter VPN VLAN 70. A valid account, device certificate, MFA, current patches, encryption, endpoint protection, and an active host firewall are required. Full tunneling sends connected traffic through company inspection, while role-based rules limit each user to approved services. Narrow RemoteApp or browser-based TLS access may support contractors, and administrators must use the VPN plus a hardened jump host rather than exposing management ports to the Internet.

Corporation Techs should retain private IPv4 during implementation while requiring new equipment to support IPv6 and planning a later dual-stack pilot. The project will proceed through discovery, offline testing, redundant infrastructure deployment, segmentation and DMZ migration, identity deployment, VPN piloting, staged rollout, and validation. Success will be measured through availability reports, failover and restoration exercises, blocked unauthorized paths, working MFA and role restrictions, protected VPN payloads, endpoint-policy compliance, centralized alerts, and load testing. The initial target is at least 99.9% monthly availability for critical services, excluding approved maintenance.

## 1. Requirements and Network Design

The design must preserve public website access, secure internal resources, support a primarily remote workforce within six months, operate continuously, and allow growth without rebuilding the network. The present flat network and single border firewall cannot meet those requirements. NIST recommends grouping resources with similar security needs and controlling communication between segments (Chandramouli, 2022).

Two diverse ISP circuits terminate at the firewall pair. Each firewall connects to both redundant Layer 3 core switches, and the switches use resilient inter-switch links. The web server uses a dedicated DMZ interface. Internal switch trunks carry VLAN 20 for Accounting, VLAN 30 for Sales, VLAN 40 for general users, VLAN 50 for the six internal servers, VLAN 60 for management and monitoring, and VLAN 70 for VPN users. Each VLAN receives a separate subnet and DHCP scope.

Inter-zone policy is deny-by-default. Accounting and Sales may reach only approved applications and shares; neither department may initiate traffic to the other. Workstations cannot connect directly to databases, and administration originates only from VLAN 60. CISA explains that segmentation and DMZs hinder lateral movement and separate public-facing systems from internal assets (CISA, 2021, 2022).

## 2. Availability and IP Strategy

Different providers and physical paths are preferred so one carrier failure or cable cut does not disable both links. Firewalls synchronize configuration and state, redundant switches preserve internal connectivity, and critical devices use redundant power and UPS protection. Monitoring covers Internet links, firewalls, switches, servers, certificates, backups, VPN service, and the website. Alerts have owners and escalation paths. Protected backups, documented recovery steps, vendor support, restoration tests, and scheduled failover exercises complete the 24/7 plan (Swanson et al., 2010).

Private IPv4 should remain during this project because it supports expected growth and avoids adding a major migration while segmentation, authentication, firewalls, and remote access are changing. All new equipment must support IPv6. Administrators will inventory IPv6 capability, block or explicitly manage unintended IPv6 traffic, train staff, and later test dual stack in a low-risk VLAN with equivalent filtering and monitoring. NIST warns that IPv6 migration changes the security posture and requires planning and testing (Frankel et al., 2010).

## 3. Firewall Selection, Placement, and DMZ

Two FortiGate 90G NGFWs should replace the aging border firewall. In active-passive high availability they provide stateful inspection, intrusion prevention, application control, web and malware filtering, secure network address translation, VPN termination, SD-WAN functions, and centralized logging (Fortinet, 2026a). Final procurement depends on measured peak traffic, concurrent VPN use, licenses, support, and throughput with every required security service enabled.

Both ISPs and every protected zone connect to the firewall pair, so Internet, DMZ, user, server, management, and VPN traffic cannot bypass policy at the core. Rules identify the approved source, destination, user or group, application, service, owner, and review date. Unmatched traffic is denied and logged. This gateway placement creates the controlled chokepoint recommended for effective filtering (Stewart & Kinsey, 2021).

The Linux/Apache server moves to DMZ VLAN 10. Public DNS and static network address translation publish HTTPS without exposing internal addressing. TCP 443 is allowed, TCP 80 may redirect to HTTPS, and Internet-to-LAN and DMZ-to-LAN initiation are denied. SSH is allowed only from named administrator hosts in VLAN 60. The server is patched, unnecessary services are removed, certificates and security headers are maintained, and its host firewall and logs remain active. Any future internal dependency must identify one destination and service and receive approval.

Windows Defender Firewall stays enabled on all Windows servers and workstations. Group Policy permits required server roles and restricts administrative traffic to VLAN 60 (Microsoft, 2026). Firewall administrators use separate privileged accounts, MFA, encrypted management protocols, and trusted workstations. Configuration backups, tested updates, quarterly rule review, vulnerability scans, and centralized alerting verify continued effectiveness.

## 4. Secure Authentication

Two Active Directory Domain Services controllers provide redundant authentication, DNS, and policy. Individual accounts and role-based groups grant Accounting, Sales, support, and administrators only required resources. Shared accounts are prohibited; administrators receive separate standard and privileged identities. Service accounts receive minimum rights, and users reach databases through approved applications unless direct access is specifically authorized.

MFA is required for remote access, administrators, sensitive applications, and high-risk actions. Phishing-resistant FIDO2 keys or Windows Hello for Business are preferred, with approved authenticator applications as a controlled fallback. NIST states that passwords are not phishing-resistant (NIST, 2025). Passwords are screened against compromised values, rate-limited, and changed when compromise is suspected.

Domain-joined devices use Kerberos, Group Policy, and device certificates. Redundant RADIUS supports 802.1X user and device authentication before VLAN assignment; unknown devices enter a restricted network. A joiner-mover-leaver process verifies identity, adjusts access after role changes, and promptly disables departed users or lost authenticators. Authentication activity feeds centralized monitoring, while recovery and MFA enrollment require identity verification.

## 5. Remote Access and VPNs

Corporation Techs should deploy IKEv2/IPsec for company-managed Windows laptops. FortiClient creates encrypted tunnels to the FortiGate pair, and connected users receive addresses in VPN VLAN 70. IKEv2 negotiates security associations and fresh keys; IPsec Encapsulating Security Payload in tunnel mode protects confidentiality, integrity, origin authentication, and replay resistance. NAT traversal supports typical home routers (Stewart & Kinsey, 2021).

SSL/TLS is useful for browser-only access, but IPsec better supports technicians who need several approved network services. It also follows the selected platform’s current direction: Fortinet removed SSL-VPN tunnel mode beginning with FortiOS 7.6.3 and directs tunnel deployments toward IPsec, while FortiClient supports IKEv2, certificates, and multiple gateways (Fortinet, 2026b, 2026c).

Every connection requires an authorized AD group, company device certificate, and MFA. Managed laptops must be encrypted, patched, supported, centrally controlled, and protected by endpoint security, a host firewall, and screen lock. Lost, revoked, stale, or noncompliant devices are blocked or quarantined. Support, Accounting, Sales, and administrators map to different policies; direct database access, client-to-client traffic, and VPN-to-DMZ access are denied unless approved.

Full tunneling is the default so corporate and Internet traffic passes through company inspection. Split tunneling requires risk approval because it creates simultaneous trusted and untrusted paths and reduces visibility (Souppaya & Scarfone, 2016). Two public gateway addresses use separate ISP paths, and authentication, posture, connection, and denial events create centralized alerts.

RemoteApp or Remote Desktop Gateway may publish a controlled application through TLS and MFA for contractors or personal devices, keeping processing and data inside Corporation Techs. An agentless portal may expose approved web applications without general network access. Administrators establish the VPN and then use a hardened, monitored jump host in VLAN 60 for SSH, RDP, or PowerShell. Public SSH, RDP, SMB, database, and management ports remain closed. CISA recommends MFA and prompt patching for remote-access infrastructure and endpoints (CISA, 2020, 2024).

## 6. Implementation Plan

Discover and baseline. Inventory addressing, devices, users, applications, dependencies, traffic flows, peak bandwidth, remote concurrency, certificates, rules, and recoverable backups.

Finalize and build. Confirm ISP diversity, sizing, licenses, support, VLANs, rule ownership, rollback criteria, and power. Build the firewalls, switches, logging, and test VLANs offline; validate updates, backup restoration, and high availability.

Deploy redundant infrastructure. Install both ISP paths, the firewall pair, core switches, UPS equipment, and monitoring beside the current environment. Verify routing and failover before migrating production traffic.

Migrate the DMZ and internal VLANs. Patch and back up the web server, publish HTTPS from VLAN 10, and verify internal denial. Move management, servers, Accounting, Sales, and general users in controlled groups with a rollback point for each phase.

Deploy identity and host policy. Implement redundant AD DS and RADIUS, role groups, MFA, certificates, account-lifecycle procedures, and 802.1X. Pilot host-firewall policies with IT before staged deployment.

Pilot and roll out VPN access. Build IKEv2 gateways, address pools, DNS, posture rules, authorization, full tunneling, and logging. Test from home, mobile, and restrictive networks; train users and help-desk staff before deployment waves.

Validate and operate. Run vulnerability scans, rule and role tests, packet captures, load tests, restore tests, and ISP/firewall/switch/power failovers. Correct findings, obtain owner approval, then review access and rules quarterly.

## 7. Success Measures

| Enhancement | Acceptance evidence |
| --- | --- |
| Segmentation and DMZ | Approved flows and external HTTPS work; unauthorized inter-VLAN, Internet-to-LAN, and DMZ-to-LAN paths fail. |
| Availability and capacity | Critical services meet 99.9% monthly availability; failover causes no unacceptable outage; projected peak load meets response targets. |
| Firewalls and monitoring | Unapproved traffic is denied and logged; managed hosts report required policy; test events reach assigned responders. |
| Authentication | Authorized role and MFA tests succeed; disabled accounts, missing factors, and excessive access fail; quarterly reviews remove stale rights. |
| VPN and endpoints | External capture shows no readable payload; compliant devices connect; revoked or noncompliant devices and unauthorized destinations fail. |
| Recovery | Configuration and data backups restore within approved recovery objectives, with findings corrected and retested. |

Control owners retain test results, monitoring reports, approved exceptions, and corrective actions. Every firewall rule, privileged group, remote-access exception, and DMZ dependency requires a business owner, purpose, approval, and review date. Results are reviewed after rollout, quarterly, and after major incidents or architecture changes.

Figure 1. Final Corporation Techs Network Design

![Original project evidence](../assets/image1.png)

Note. All inter-zone traffic is controlled by deny-by-default policy. Representative workstations stand for the full workstation population.

## 8. Conclusion

The final design meets Corporation Techs’ growth, remote-work, security, and availability requirements through defense in depth. Redundant communications reduce single points of failure; segmentation, the DMZ, NGFW inspection, and host firewalls limit exposure and lateral movement; and centralized identity, MFA, certificates, and role-based access strengthen trust. IKEv2/IPsec extends these protections to managed remote laptops without bypassing internal policy.

A phased deployment provides testing and rollback at each major transition. Measurable evidence - blocked unauthorized paths, successful role and MFA testing, protected VPN traffic, endpoint compliance, centralized alerts, backup restoration, capacity results, and proven failover - gives management a clear basis for acceptance. The result is a secure and supportable architecture that can scale to approximately twice the present client-facing workforce.

## References

Chandramouli, R. (2022). Guide to a secure enterprise network landscape (NIST Special Publication 800-215). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-215

Cybersecurity and Infrastructure Security Agency. (2020). Enterprise VPN security. https://www.cisa.gov/news-events/cybersecurity-advisories/aa20-073a

Cybersecurity and Infrastructure Security Agency. (2021). Website security. https://www.cisa.gov/news-events/news/website-security

Cybersecurity and Infrastructure Security Agency. (2022). Layering network security through segmentation. https://www.cisa.gov/sites/default/files/publications/layering-network-security-segmentation_infographic_508_0.pdf

Cybersecurity and Infrastructure Security Agency. (2024). Threat actor leverages compromised account of former employee to access state government organization. https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-046a

Fortinet. (2026a). FortiGate 90G series data sheet. https://www.fortinet.com/resources/data-sheets/fortigate-90g-series

Fortinet. (2026b). SSL VPN tunnel mode no longer supported in FortiOS 7.6.3. https://docs.fortinet.com/document/fortimanager/8.0.0/release-notes/755934/

Fortinet. (2026c). Configuring an IPsec VPN connection. https://docs.fortinet.com/document/forticlient/8.0.0/administration-guide/494391/

Frankel, S., Graveman, R., Pearce, J., & Rooks, M. (2010). Guidelines for the secure deployment of IPv6 (NIST Special Publication 800-119). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-119

Microsoft. (2026). Configure firewall rules with Group Policy. https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/configure

National Institute of Standards and Technology. (2025). Digital identity guidelines: Authentication and authenticator management (NIST Special Publication 800-63B-4). https://doi.org/10.6028/NIST.SP.800-63B-4

Souppaya, M., & Scarfone, K. (2016). Guide to enterprise telework, remote access, and bring your own device security (NIST Special Publication 800-46 Rev. 2). https://doi.org/10.6028/NIST.SP.800-46r2

Stewart, J. M., & Kinsey, D. (2021). Network security, firewalls, and VPNs (3rd ed.). Jones & Bartlett Learning.

Swanson, M., Bowen, P., Phillips, A. W., Gallup, D., & Lynes, D. (2010). Contingency planning guide for federal information systems (NIST Special Publication 800-34 Rev. 1). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-34r1
