# FORGE-04: Insecure Out-of-Band Management Plane

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **Critical** | Medium | High | Very High |

## Definition

This risk arises when systems managing physical infrastructure outside the host OS are exposed, weakly authenticated, or poorly monitored. The management plane encompasses BMCs, IPMI and Redfish interfaces, dedicated management networks, jump hosts, fleet-management and firmware-update tools, and management switches.

## Description

In AI data centers, out-of-band (OOB) management enables administrators to control physical infrastructure independently of the host operating system. The BMC is a separate management processor on each server handling remote functions like power control, hardware monitoring, firmware updates, remote console access, and virtual media.

The management plane extends beyond individual BMCs to include:

- **The Dedicated Management Network:** Physical and virtual infrastructure connecting OOB endpoints - management VLANs, jump hosts, VPNs, and management switches.
- **Management Protocols:** Interfaces for accessing these systems - IPMI, Redfish APIs, SSH, web consoles, and vendor-specific management interfaces.
- **Centralized Fleet Management Tools:** Platforms storing credentials, inventory systems, and automation workflows for managing large numbers of BMCs and servers at scale (e.g., Dell OpenManage Enterprise).
- **Power Distribution Units (PDUs):** Network-connected smart PDUs and related systems that can remotely cut, cycle, or sequence power to racks and devices.
- **Firmware Update Channels:** Mechanisms to inventory, distribute, validate, and install firmware for platform components such as BIOS/UEFI, BMCs, NICs, GPUs, and storage controllers - for example through fleet-management tools like Dell OpenManage Enterprise.

Security posture varies significantly by OEM, platform generation, and configuration. Mature server platforms may include hardware roots of trust, signed firmware enforcement, integrity validation, lockdown modes, and recovery mechanisms that reduce low-level tampering risk when enabled and verified.

The residual risk is highest where OOB interfaces are reachable from shared or weakly segmented networks, credentials are weak or overprivileged, firmware update paths are insufficiently protected, or management activity is poorly monitored. In those environments, management-plane compromise can provide below-OS capabilities such as remote power control, virtual media mounting, firmware update abuse, OS reinstallation, or persistent access surviving ordinary host remediation. Compromised centralized fleet-management tooling can extend the blast radius from a single node to large parts of the fleet.

## Impact / Examples

### Default or weak BMC credentials

Many BMC platforms historically shipped with default or weak credentials. Newer platforms increasingly ship with unique per-device factory passwords or forced first-login changes, reducing this exposure. Risk now concentrates on older deployments and devices left at factory defaults. Even per-device sticker passwords may have lower real entropy than expected, leaving them vulnerable to offline cracking unless rotated.

### Legacy or weakly protected IPMI sessions

IPMI 2.0 includes insecure options such as Cipher Suite 0 and other suites with no integrity or confidentiality. When enabled, these weaken or eliminate session protection and can expose management traffic to interception or abuse.

### Credential reuse across large fleets

Reused BMC or fleet-management credentials can turn one compromised account into broad management-plane access. Historical IPMI research showed cases where credential material or password hashes could be exposed, enabling offline cracking or recovery.

### Enabled host-to-BMC interfaces

In-band interfaces such as KCS, USB/RNDIS, pass-through, or Redfish Host Interface can allow host-level software to interact with the management controller. If exposed, vulnerable, or misconfigured, a host compromise may become a path into the management plane.

### Smart PDU compromise enabling physical denial of service

An attacker with access to PDU management interfaces can power-cycle racks or entire rows, halting multi-day training jobs and potentially corrupting in-flight checkpoints.

### Critical Redfish and BMC API vulnerabilities

Vendor Redfish and IPMI implementations have repeatedly exposed remote authentication bypass and code execution flaws. Recent examples include:
- CVE-2023-34329
- CVE-2023-34330
- CVE-2024-54085 in AMI MegaRAC-derived platforms
- CVE-2017-12542 in HPE iLO4

### Poor patching and limited visibility

Slow BMC firmware patch cycles, incomplete inventory, and weak audit collection can leave known vulnerabilities unremediated for long periods and make malicious management-plane activity difficult to detect.

### Direct customer BMC exposure in hosted environments

Management interfaces left reachable from tenant networks or the public internet due to misconfigured routing or weak network segmentation, without isolation or monitoring to detect unauthorized access.

### Adjacent management-network devices can become control points

Smart PDUs, console servers, KVM systems, management switches, and SNMP-managed devices can expose inventory, configuration details, console access, or physical control if they use weak credentials, legacy protocols, or shared SNMP community strings.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Segment and monitor the management network, not just its perimeter:** Place BMCs and other OOB endpoints on a dedicated management network or VLAN with no routing from workload, storage, public-facing, or corporate networks, and never expose them directly to the internet. Treat the network as a monitored trust zone: limit lateral movement between management devices and alert on unexpected traffic, so one compromised endpoint cannot freely reach every other.

2. **Replace factory credentials and harden authentication:** Replace factory-default and per-device sticker passwords before deployment, since sticker passwords are often shorter and drawn from a narrower character set than they appear and can be cracked offline. Store replacements in an approved secrets-management system, enforce strong authentication, prefer modern encrypted interfaces, and disable insecure IPMI options such as Cipher Suite 0 where possible.

3. **Restrict who can reach management interfaces:** Limit Redfish, IPMI, SSH, web UI, and vendor-specific interfaces to authorized jump hosts or management systems, using ACLs, IP filtering, VPN access, and MFA where supported.

4. **Disable or minimize host-to-BMC interfaces:** Disable in-band interfaces such as KCS, USB networking, pass-through, or Redfish Host Interface unless operationally required. Where they must remain enabled, minimize privileges and restrict access paths tightly.

5. **Patch and verify BMC firmware:** Maintain an inventory of firmware versions and apply vendor updates promptly, using vendor-signed images verified before writing to flash, and disable unsafe update methods. Keep recovery procedures for cases that may require physical re-flashing. Where attestation is available, use it to verify BMC state, but note that self-attestation has limited coverage today and fleet inventory often relies on version numbers self-reported by the BMC itself.

6. **Collect independent logging and monitoring:** Log logins, power actions, firmware updates, virtual media mounts, console sessions, and account changes to a centralized system that does not depend on the BMC itself.

7. **Treat fleet and OOB management tools as crown-jewel systems:** Centralized BMC and fleet managers such as Dell OpenManage or Redfish aggregators hold credentials to the entire management plane, so a single compromise can reach every node. Restrict them to dedicated admin workstations or bastions, require MFA and least-privilege roles, log all actions, and use unique credentials per device or trust domain so credential reuse cannot turn one foothold into fleet-wide control.

8. **Harden other management-network devices:** Smart PDUs, serial console servers, and management switches often ship with default or shared credentials and legacy SNMP, and a PDU that can cut power to a rack is an availability risk on its own. Change defaults, replace SNMPv1/v2c community strings with SNMPv3, and apply the same segmentation and access controls used for BMCs.

### For customers evaluating a provider

1. **Test what management systems you can reach:** From your node and its network, confirm you cannot reach your BMC/IPMI interface, the management subnet, or other management devices such as PDUs, console servers, or switch management ports. Reaching any of these, including another tenant's BMC, is a segmentation failure you have found directly.

2. **Check for in-band host-to-BMC paths:** On bare metal, check whether interfaces such as KCS, USB/RNDIS, or the Redfish Host Interface are exposed from your OS, since these are the path used to pivot from a host compromise into the management plane.

## Attack Scenarios

### Cluster-Wide Management Plane Takeover

An attacker compromises an exposed or weakly protected BMC using a vulnerability or default credentials. A single BMC compromise does not automatically expose the fleet, but it can become a wider management-plane compromise when credentials are reused, the management network is flat, or the same vulnerable stack is deployed across many servers. The attacker may then enumerate reachable BMCs, reuse credentials, repeat the same exploit, or pivot toward centralized fleet-management tooling.

### Persistent Firmware Implant via BMC

An attacker who gains BMC access exploits a vulnerable update path or modifies BMC firmware to implant persistent malicious code. Because the implant resides in the BMC's own flash storage, it may survive host OS reinstallation, disk replacement, and ordinary node reprovisioning. Real-world reporting on iLOBleed showed that BMC firmware implants have appeared outside the lab, including implants that could interfere with firmware updates and report an expected firmware version while keeping compromised code in place. This is why operators should not rely only on BMC self-reported version numbers as proof of a clean update, and should verify that signed-update enforcement, firmware integrity checks, and recovery protections are enabled where supported.

### Host-to-BMC Pivot Through In-Band Interfaces

An attacker first gains root access on a host through an application, container, or OS compromise. The environment leaves KCS, USB/RNDIS, pass-through, or Redfish Host Interface enabled between the host and the BMC. The attacker uses that path to interact with the management controller, create new accounts, change settings, or pursue firmware-level persistence. In some architectures, local IPMI access means this pivot may not require separate BMC credentials. The result is that an ordinary host compromise is upgraded into a lower-level management-plane compromise that survives OS rebuilding and sits outside normal host-based security visibility.

## References

- **Breaking BMC: The Forgotten Key to the Kingdom.** DEF CON. BMC attack surface overview and exploitation techniques. https://forum.defcon.org/node/245714
- **Vulnerable Firmware in the Supply Chain of Enterprise Servers.** Eclypsium. Firmware supply-chain risk across server platforms. https://eclypsium.com/wp-content/uploads/Vulnerable-Firmware-in-the-Supply-Chain.pdf
- **Analyzing Baseboard Management Controllers to Secure Data Center Infrastructure.** NVIDIA Technical Blog. BMC security analysis for AI data centers. https://developer.nvidia.com/blog/?p=100850
- **NVIDIA DGX Security Guidance, Securing the BMC Port.** NVIDIA. DGX-specific BMC hardening recommendations. https://docs.nvidia.com/dgx/dgxh100-user-guide/security.html
- **Dell iDRAC Security Best Practices.** Dell. Vendor BMC hardening guidance for iDRAC9. https://www.dell.com/support/manuals/en-us/idrac9-lifecycle-controller-v7.x-series/idrac9_scg_tta/best-practices?guid=guid-a7415277-ace9-40bd-9017-467a8c133165&lang=en-us
- **DMTF Redfish Host Interface Specification (DSP0270).** DMTF. Standard for host-to-BMC in-band communication. https://www.dmtf.org/dsp/DSP0270
- **A Penetration Tester's Guide to IPMI and BMCs.** Rapid7, 2013. IPMI attack techniques, default credentials, and enumeration. https://www.rapid7.com/blog/post/2013/07/02/a-penetration-testers-guide-to-ipmi/
- **IPMI: Freight Train to Hell.** Dan Farmer, 2013. Foundational IPMI security research covering protocol weaknesses and credential exposure. http://fish2.com/ipmi/itrain.pdf
- **OpenBMC Project.** Linux Foundation. Open-source BMC firmware stack. https://openbmc.org/
- **CVE-2023-34329.** AMI MegaRAC Redfish authentication bypass. https://nvd.nist.gov/vuln/detail/CVE-2023-34329
- **CVE-2023-34330.** AMI MegaRAC Redfish code execution via Host Interface. https://nvd.nist.gov/vuln/detail/CVE-2023-34330
- **CVE-2024-54085.** AMI MegaRAC critical BMC vulnerability. https://nvd.nist.gov/vuln/detail/CVE-2024-54085
- **CVE-2017-12542.** HPE iLO4 authentication bypass (CVSS 10.0). https://nvd.nist.gov/vuln/detail/CVE-2017-12542
- **BMC&C: Lights Out Forever.** Eclypsium. BMC-based persistent access and command-and-control techniques. https://eclypsium.com/research/bmcc-lights-out-forever/
- **The iLOBleed Implant.** Eclypsium, 2021. First BMC firmware rootkit discovered in the wild. https://eclypsium.com/blog/the-ilobleed-implant-lights-out-management-like-you-wouldnt-believe/
- **Turning your BMC into a revolving door.** Airbus Security Lab, ZeroNights 2018. iLO exploitation techniques including firmware analysis and persistent access. https://airbus-seclab.github.io/ilo/ZERONIGHTS2018-Slides-EN-Turning_your_BMC_into_a_revolving_door-perigaud-gazet-czarny.pdf
- **BMC Attack Surface via Host-to-BMC Interfaces.** ScienceDirect, 2020. Host-to-BMC DMA and memory access risks. https://www.sciencedirect.com/science/article/pii/S2666281720300147
