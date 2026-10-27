# Part 3 — Remote access and VPNs

**Architecture proposal · ITN 263 · Summer 2026 · Franco Pisano**

This is a course design scenario. Proposed controls, rollout actions and success measures are not evidence of a deployed production network. Vendor references are preserved as part of the historical submission, not a current procurement recommendation.

Corporation Techs Network Design

Project Part 3: Remote Access and VPNs

Franco Pisano | ITN 263 | August 2, 2026

Purpose and Recommendation

Corporation Techs should deploy an Internet Key Exchange version 2 (IKEv2) Internet Protocol Security (IPsec) remote-access VPN for company-managed Windows laptops. FortiClient establishes encrypted tunnels to the redundant FortiGate 90G pair selected in Part 2, and authenticated users enter VPN VLAN 70 rather than joining an internal user VLAN directly. The firewalls then inspect and restrict VPN-to-server traffic by role. This design supports a workforce approaching 100 client-facing employees while protecting information carried across home, hotel, mobile, and other untrusted networks.

A VPN creates a protected connection across the Internet by combining tunneling, encryption, and authentication (Stewart & Kinsey, 2021). IKEv2 negotiates the security association and keys; IPsec Encapsulating Security Payload in tunnel mode encrypts the original packet and provides confidentiality, integrity, origin authentication, and anti-replay protection. Network address translation traversal allows the tunnel to work through typical home routers.

IPsec Compared with SSL/TLS

SSL/TLS VPNs are attractive when users need browser-based access from unmanaged devices because they require little client configuration. IPsec requires managed client software but provides transparent network-layer access to the file, print, application, and support services that technical representatives use. Corporation Techs is already standardizing Windows endpoints through Active Directory and Group Policy, so centrally deployed FortiClient profiles are a manageable tradeoff.

| Criterion | IKEv2/IPsec | SSL/TLS |
| --- | --- | --- |
| Access scope | Broad approved IP applications and services | Best for web portals or selected applications |
| Endpoint | Managed client and certificate | Browser or client; suitable for limited unmanaged access |
| Fit here | Strong fit for managed technicians needing several services | Secondary fit for browser-only access |
| Platform direction | Current FortiGate remote-access tunnel path | Tunnel mode removed from current FortiOS releases |

Table 1. Comparison for Corporation Techs; the decision reflects both business needs and the selected platform.

The product roadmap makes IPsec the more sustainable choice: Fortinet removed SSL-VPN tunnel mode beginning with FortiOS 7.6.3 and directs tunnel deployments toward IPsec (Fortinet, 2026a). FortiClient supports IKEv2, multiple remote gateways, and certificate-based configurations (Fortinet, 2026b). This avoids designing a new service around a retired tunnel feature while retaining agentless TLS access only where a browser portal is sufficient.

Secure Remote-Access Architecture

Authentication. Active Directory authorizes membership in VPN groups. Every connection requires a device certificate plus phishing-resistant MFA, preferably FIDO2 or Windows Hello for Business. Password-only access and shared accounts are prohibited.

Endpoint trust. Only company-managed, encrypted laptops may form a full tunnel. Devices must have supported software, current patches, active endpoint protection, host firewall, screen lock, and centralized management. Noncompliant or lost devices are blocked or quarantined.

Least privilege. VPN users receive addresses in VLAN 70. Support, Accounting, Sales, and administrators map to separate policies; users may reach only assigned applications, shares, and management targets. Direct database access, lateral client-to-client traffic, and VPN-to-DMZ access are denied unless specifically approved.

Traffic protection. Full tunneling is the default so corporate and Internet traffic from the remote laptop passes through company inspection. Split tunneling reduces bandwidth use but creates simultaneous trusted and untrusted paths and limits organizational monitoring (Souppaya & Scarfone, 2016). Exceptions require risk approval.

Operations. The two public VPN addresses use separate ISP paths and redundant firewalls. Connection, authentication, posture, and policy-denial logs feed centralized monitoring. Administrators patch VPN infrastructure promptly, review access quarterly, revoke departed users immediately, and alert on repeated failures or unusual locations.

Other Relevant Forms of Remote Access

Remote Desktop Gateway or RemoteApp can publish a controlled Windows desktop or individual application through TLS after MFA. It is useful for contractors, personal devices, or sensitive tools because application execution and data remain inside Corporation Techs. It must sit behind the firewall or a reverse proxy, expose no direct Remote Desktop Protocol port to the Internet, restrict clipboard and drive redirection, and record administrative activity. This is a limited alternative, not an unrestricted path into the LAN.

Administrators may use Secure Shell for the Linux web server and Remote Desktop or PowerShell remoting for Windows systems only after entering the VPN and then connecting through a hardened jump host in VLAN 60. Public SSH, RDP, database, and file-sharing ports remain closed. A browser-based agentless TLS portal may provide approved web applications to short-term users, but it receives no general network access. Site-to-site IPsec can later connect a permanent branch office; it is not a substitute for identifying individual home users.

Implementation and Testing

First, measure concurrent-user demand, bandwidth, and application latency, then confirm licenses and firewall capacity with inspection enabled. Build redundant IKEv2 gateways, certificates, address pools, DNS, group policies, timeouts, and logging in a test environment. Pilot with IT from home and mobile networks, then document enrollment, recovery, and help-desk procedures before staged deployment. CISA recommends promptly patching VPN infrastructure and endpoints and using MFA for remote access (CISA, 2020, 2024).

| Control | Acceptance evidence |
| --- | --- |
| Encryption | Packet capture shows no readable organizational payload outside the tunnel. |
| Identity | Authorized user, device certificate, and MFA succeed; missing factors and disabled users fail. |
| Authorization | Each role reaches approved services only; lateral movement and direct database access fail. |
| Endpoint posture | Compliant managed devices connect; stale, unmanaged, or revoked devices are blocked. |
| Availability | Sessions reconnect through the alternate firewall or ISP with no unacceptable outage. |
| Capacity and monitoring | Peak-load testing meets targets, and test events create centralized alerts. |

Conclusion

IKEv2/IPsec provides secure, scalable access for managed technical representatives and aligns with the FortiGate platform selected in Part 2. MFA, device certificates, endpoint compliance, full tunneling, segmentation, and least privilege keep the VPN from bypassing internal controls. Controlled RemoteApp, agentless web access, and a jump host cover narrower use cases without directly exposing internal services.

## References

Cybersecurity and Infrastructure Security Agency. (2020). Enterprise VPN security. https://www.cisa.gov/news-events/cybersecurity-advisories/aa20-073a

Cybersecurity and Infrastructure Security Agency. (2024). Threat actor leverages compromised account of former employee to access state government organization. https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-046a

Fortinet. (2026a). SSL VPN tunnel mode no longer supported in FortiOS 7.6.3. https://docs.fortinet.com/document/fortimanager/8.0.0/release-notes/755934/

Fortinet. (2026b). Configuring an IPsec VPN connection. https://docs.fortinet.com/document/forticlient/8.0.0/administration-guide/494391/

Souppaya, M., & Scarfone, K. (2016). Guide to enterprise telework, remote access, and bring your own device security (NIST SP 800-46 Rev. 2). https://doi.org/10.6028/NIST.SP.800-46r2

Stewart, J. M., & Kinsey, D. (2021). Network security, firewalls, and VPNs (3rd ed.). Jones & Bartlett Learning.
