# FORGE-06: Insecure Facility & Datacenter Management Systems

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **High** | Low | High | Very High |

## Definition

This category covers weaknesses in the software platforms that run the physical datacenter, including the systems that control cooling, power, and physical access. These include BMS for cooling and environmental controls, EPMS for electrical equipment, and DCIM for rack and asset tracking. Compromise of these systems can disrupt training and inference workloads, corrupt in-flight checkpoints, expose detailed maps of high-value assets, and bypass physical security without directly compromising a workload.

## Description

AI datacenters depend on facility infrastructure that is often invisible to security teams focused on servers, networks, and cloud control planes. BMS platforms control chilled water loops, CRAH and CRAC units, and cooling systems for high-density GPU racks. EPMS platforms monitor and sometimes control switchgear, UPS units, generators, and branch-circuit power distribution. DCIM platforms aggregate telemetry, track rack and asset location, and support capacity planning. Physical security systems govern entry to buildings, data halls, cages, rows, or maintenance areas.

These systems matter more for AI infrastructure than conventional compute because GPU clusters operate closer to thermal and power limits than traditional server fleets. Modern GPU racks can draw 50 to 130 kW or more, well above the 5 to 15 kW common in earlier generations. A facility-side disruption to cooling or power can stop a multi-day training run, corrupt checkpoints across many nodes, or force emergency shutdown of an entire hall.

Facility systems are often less hardened than server-side infrastructure. Legacy OT protocols such as BACnet, Modbus, and SNMPv1 are still common and may lack authentication or encryption by default. Vendor and integrator remote access can be broad or long-lived, and patch cycles are often slower than for IT systems. Exposure is highest in legacy installations, default configurations, abandoned or unpatched products, broad vendor-access setups, and environments where facility and IT security are not jointly owned or verified.

## Impact and Failure Modes

### Legacy building-control protocols with no built-in security

BACnet, Modbus, and SNMPv1 are still widely used to control cooling and power. In their common legacy form they do not authenticate commands or encrypt traffic, so anyone who reaches the network can read sensors, change setpoints, or send shutdown commands. Authenticated variants exist but are not yet the norm in deployed facilities.

### Always-on remote access for outside vendors

Controls integrators and equipment vendors are often given persistent remote access into facility networks, sometimes through shared accounts or broad VPN access. A compromised vendor account, support portal, or remote-access path can become a route into facility controls across one or more sites.

### Internet-exposed management interfaces

Public research has identified tens of thousands of datacenter management interfaces, including DCIM platforms, cooling controllers, UPS units, and rack monitors, reachable directly from the internet. Exposure is especially dangerous when combined with default credentials, weak authentication, or missing VPN and jump-host controls.

### Facility software maintained by small teams or single vendors

BMS, EPMS, DCIM, and related facility platforms may be maintained by small vendor teams, integrators, or open-source communities. When a product is abandoned, customized heavily, or no longer receives security updates, operators may be left without a clear patch path or migration plan. openDCIM, a widely deployed open-source platform, lost its lead maintainer in 2025 with no successor announced.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Treat facility systems as a separate, segmented network:** Place BMS, EPMS, DCIM, and physical security systems on dedicated networks that are not reachable from tenant workloads, corporate IT, or the public internet. Do not assume that being inside the datacenter perimeter is enough.

2. **Govern vendor remote access:** Inventory every remote connection used by integrators, controls vendors, and equipment manufacturers. Replace always-on connections with time-bound, on-demand access through a controlled jump host. Require unique accounts per person, multi-factor authentication, and full session logging. Treat vendor access as a privileged path, not a convenience.

3. **Use authenticated and encrypted protocols where supported:** Where the equipment supports it, prefer modern variants such as BACnet/SC and Modbus Security. Replace SNMPv1 and SNMPv2c with SNMPv3 for monitoring. Where legacy protocols cannot be replaced, isolate them on dedicated network segments and restrict which devices can speak them.

4. **Restrict and monitor DCIM access:** A DCIM platform is a detailed map of high-value assets and should be protected accordingly. Apply least-privilege roles, multi-factor authentication, and audit logging for all access. Limit which users, hosts, and integrations can read or export full inventory data.

5. **Patch facility software and track lifecycle:** Maintain an inventory of BMS, EPMS, DCIM, and physical security software versions, including which products are still actively maintained by their vendors. Apply vendor patches on a defined schedule, and plan ahead for products that are abandoned or near end-of-life.

### For customers evaluating a provider

Facility infrastructure sits almost entirely on the provider side and is rarely visible from inside a rented node, so this is about checking what the provider publishes and asking directly where documentation is silent.

1. **Check the provider's security and trust documentation:** Review trust or security pages, whitepapers, and compliance scope for how facility and physical infrastructure is covered. Facility, power, and environmental controls are often in scope for these reports, so their presence or absence is a useful signal.

2. **Ask directly where the documentation is silent:** Published material rarely covers everything, so ask how facility systems (BMS, EPMS, DCIM) are segmented from tenant and public networks, how integrator and equipment-vendor remote access is governed (time-bound, individually attributed, logged), and whether facility management interfaces are internet-exposed.

## Attack Scenarios

### Vendor Compromise to Cooling Disruption

A controls integrator providing remote support to several datacenter operators is breached through an unpatched public-facing maintenance portal. The integrator maintains always-on VPN connections into each customer's facility network, so once inside the integrator's environment, the attacker can reach the BMS at a GPU site without breaching the operator directly. From the BMS, the attacker lowers cooling capacity by changing chilled-water setpoints and disabling several air handlers. The affected environment becomes unstable, causing workload interruptions and reduced cluster availability.

### Internet-Exposed DCIM as a Map for a Targeted Attack

A DCIM platform is reachable from the public internet because it was made accessible for remote facilities staff and was never placed behind a VPN or jump host. The attacker discovers the instance through internet-wide scanning and logs in using weak or default credentials. The DCIM inventory reveals which racks belong to which tenants, where high-density GPU clusters are located, and which power feeds and cooling zones serve them. The attacker uses this information to plan a targeted disruption or physical-access attempt against a specific high-value tenant.

## References

- **Data Centers Facing The Risk of Cyberattacks.** Cyble Research Labs. Investigation finding more than 20,000 internet-exposed datacenter management interfaces, many secured only by default credentials. https://cyble.com/blog/data-centers-facing-risk-of-cyberattacks/
- **NIST SP 800-82 Rev. 3, Guide to Operational Technology (OT) Security.** NIST. Foundational guidance for securing industrial control and building automation systems, including segmentation, access control, and protocol-specific recommendations. https://csrc.nist.gov/pubs/sp/800/82/r3/final
- **ISA/IEC 62443 Series, Security for Industrial Automation and Control Systems.** ISA/IEC. International standard for OT security covering network segmentation, access control, and lifecycle management of control system components. https://www.isa.org/standards-and-publications/isa-standards/isa-iec-62443-series-of-standards
- **ASHRAE Standard 135, BACnet.** ASHRAE. The base BACnet standard, used by most building management systems for HVAC and environmental control. https://www.ashrae.org/technical-resources/bookstore/standard-135
- **BACnet Secure Connect (BACnet/SC).** ASHRAE. Modern transport addition to BACnet that provides TLS-based authentication and encryption. https://www.ashrae.org/file%20library/technical%20resources/bookstore/bacnet-sc-overview.pdf
- **Modbus Security Specification.** Modbus Organization. Specification adding TLS-based authentication and encryption to the Modbus protocol. https://modbus.org/docs/MB-TCP-Security-v21_2018-07-24.pdf
- **Target 2013 Data Breach Analysis.** Senate Commerce Committee staff report. Documents how an HVAC vendor's compromised remote access was used as the initial access vector into the Target retail network. https://www.commerce.senate.gov/services/files/24d3c229-4f2f-405d-b8db-a3a67f183883
- **ICS-CERT Advisories on Building Management and Power Systems.** CISA. Ongoing vendor advisories covering vulnerabilities in BMS, EPMS, UPS, and DCIM products from manufacturers including Schneider Electric, Siemens, Tridium, and Vertiv. https://www.cisa.gov/news-events/cybersecurity-advisories
- **openDCIM Project.** Open-source DCIM platform widely deployed in academic and commercial datacenters. The project's lead maintainer announced retirement in 2025 with no successor identified at the time of writing. https://github.com/opendcim/openDCIM
