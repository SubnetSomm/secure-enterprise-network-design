# Part 2 — Firewall selection and placement

**Architecture proposal · ITN 263 · Summer 2026 · Franco Pisano**

This is a course design scenario. Proposed controls, rollout actions and success measures are not evidence of a deployed production network. Vendor references are preserved as part of the historical submission, not a current procurement recommendation.

Corporation Techs Network Design

Project Part 2: Firewall Selection, DMZ, and Authentication

Franco Pisano | ITN 263 | August 2, 2026

Purpose and Recommended Security Architecture

Corporation Techs should replace its aging border firewall with two FortiGate 90G next-generation firewalls (NGFWs) in active-passive high availability. The pair provides stateful inspection, intrusion prevention, application control, web filtering, malware protection, network address translation, logging, and redundant Internet connectivity. Final sizing must use measured peak traffic with all licensed security services enabled. Fortinet identifies integrated NGFW, secure networking, SD-WAN, and zero-trust access capabilities for this model (Fortinet, 2026).

Both ISP circuits and every protected zone connect to the firewall pair. Separate interfaces serve the DMZ, user, server, management, and future VPN zones. All inter-zone traffic traverses deny-by-default policy instead of bypassing inspection at the core switches. This creates the gateway or chokepoint placement recommended for effective filtering (Stewart & Kinsey, 2021).

Network, Server, and Workstation Firewalls

Layered controls are required because a compromised device may already be inside the perimeter:

Network. The FortiGate pair filters Internet, DMZ, inter-VLAN, and future VPN traffic. Only explicitly approved source, destination, application, user, and service combinations are allowed; all other traffic is denied and logged.

Servers. Windows Defender Firewall remains active on all internal servers. Role-specific Group Policy Objects permit only required services from authorized subnets. The Linux/Apache host firewall permits public HTTPS and restricted management only.

Workstations. Windows Defender Firewall blocks unsolicited inbound connections and restricts administrative services to VLAN 60. Microsoft supports centralized deployment through Group Policy (Microsoft, 2026).

DMZ Design and Security Benefit

The public Linux/Apache server moves to DMZ VLAN 10 on a dedicated firewall interface. Public DNS and static network address translation publish the site without placing the host on the internal network. The DMZ buffers Accounting, Sales, databases, and file servers from a compromised public server; properly implemented DMZs and firewalls shield valuable assets and hinder lateral movement (CISA, 2022).

Rules are narrowly scoped and logged. Management originates only from secured administrator systems in VLAN 60, and no firewall management interface is Internet-facing. Any future DMZ-to-application connection requires a documented, exact destination and service rather than broad internal access.

| Source | Destination | Permitted traffic | Purpose |
| --- | --- | --- | --- |
| Internet | DMZ web server | TCP 443; optional TCP 80 redirect | Publish HTTPS website only |
| Internet | Internal network | None | Prevent direct public access |
| DMZ web server | Internal network | None by default | Contain a compromised web server |
| Management VLAN | DMZ web server | SSH 22 from admin hosts | Restricted maintenance |
| Any unmatched flow | Any zone | Deny and log | Enforce least privilege |

Table 1. Initial DMZ policy; production rules require named objects, approved owners, and documented change control.

Firewall Administration and Monitoring

Administrators use separate privileged accounts, MFA, and encrypted protocols from VLAN 60. Encrypted configuration backups are stored separately. Updates follow tested change control, logs feed centralized monitoring, quarterly reviews remove expired rules, and scans plus failover exercises verify continued protection.

Secure Authentication Plan

Centralized Microsoft Active Directory Domain Services (AD DS) replaces separate username-and-password systems. Two domain controllers provide redundant authentication and DNS. Individual accounts and role-based groups grant Accounting, Sales, support, and administrators only required resources. Shared accounts are prohibited; administrators receive separate daily and privileged identities.

MFA is required for administrators, remote access, sensitive applications, and high-risk actions. Phishing-resistant FIDO2 keys or Windows Hello for Business are preferred, with authenticator-app codes as a controlled fallback. NIST states that passwords are not phishing-resistant (NIST, 2025). Passwords are screened against compromised values, rate-limited, and changed when compromise is suspected.

Network and Resource Authentication

Devices. Domain-joined workstations use Kerberos, Group Policy, and device certificates. 802.1X with redundant RADIUS validates users and devices before VLAN assignment; unknown devices enter a restricted network.

Resources. AD groups provide role-based access, service accounts receive minimum rights, and users reach databases through approved applications unless direct access is specifically authorized.

Administration. Privileged actions require MFA, a management workstation in VLAN 60, encrypted protocols, and audit logging. Emergency accounts remain controlled and monitored.

A joiner-mover-leaver process verifies identity, reviews access, and immediately disables departed users or lost authenticators. Authentication events feed centralized monitoring, while MFA enrollment and recovery require identity verification.

Implementation and Acceptance Plan

Implementation begins with traffic baselining, sizing, rule ownership, and an existing-configuration backup. Build and test the appliances offline, connect both ISPs and core switches, and verify failover before cutover. Next, move the patched web server to the DMZ and pilot host policies with IT before staged deployment. Finally, deploy redundant AD DS and RADIUS, create role groups, enroll MFA, migrate applications, and disable legacy authentication after validation.

| Control | Acceptance evidence |
| --- | --- |
| NGFW replacement | Required applications pass inspection; unapproved traffic is denied and logged. |
| High availability | Loss of either appliance or ISP causes no unacceptable interruption. |
| DMZ isolation | External HTTPS succeeds; scans confirm no direct Internet-to-LAN or DMZ-to-LAN path. |
| Host firewalls | Central reports show policy enabled on all managed servers and workstations. |
| Authentication | MFA and role tests succeed for authorized users; unauthorized and disabled accounts fail. |
| Monitoring | Firewall and authentication events reach centralized monitoring and trigger test alerts. |

Conclusion

The NGFW pair, isolated DMZ, managed host firewalls, and centralized authentication create defense in depth. They protect the public website, limit lateral movement, remove the firewall single point of failure, and establish the identity foundation for Part 3 remote access.

## References

Cybersecurity and Infrastructure Security Agency. (2022). Layering network security through segmentation. https://www.cisa.gov/sites/default/files/publications/layering-network-security-segmentation_infographic_508_0.pdf

Fortinet. (2026). FortiGate 90G series data sheet. https://www.fortinet.com/resources/data-sheets/fortigate-90g-series

Microsoft. (2026). Configure firewall rules with Group Policy. https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/configure

National Institute of Standards and Technology. (2025). Digital identity guidelines: Authentication and authenticator management (NIST Special Publication 800-63B-4). https://doi.org/10.6028/NIST.SP.800-63B-4

Stewart, J. M., & Kinsey, D. (2021). Network security, firewalls, and VPNs (3rd ed.). Jones & Bartlett Learning.
