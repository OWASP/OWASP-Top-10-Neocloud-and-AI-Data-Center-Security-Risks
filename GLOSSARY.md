# OWASP FORGE — Security Glossary

> Draft contribution for review. This document is not an official OWASP publication.

This glossary explains specialized terminology used in the OWASP Top 10 Neocloud and AI Data Center Security Risks (FORGE) framework.

Definitions are concise, vendor-neutral, and focused on infrastructure security. Related risks identify where each concept is particularly relevant.

## Attestation

The process of evaluating evidence about a system's identity or security state against trusted reference values or policies. Hardware-backed attestation may use cryptographic measurements to help verify platform integrity, although its assurance is limited to the components and properties measured.

**Related risks:** FORGE-01, FORGE-03, FORGE-08.

## Baseboard Management Controller (BMC)

A dedicated management processor that provides hardware monitoring and remote administration independently of a server's host operating system. BMCs commonly support power control, remote console access, and firmware management, making them highly privileged infrastructure components.

**Related risks:** FORGE-01, FORGE-04.

## Building Management System (BMS)

A system used to monitor and control building infrastructure such as cooling, ventilation, and environmental conditions. In AI data centers, compromised BMS components can disrupt the physical conditions required for reliable computing operations.

**Related risk:** FORGE-06.

## Data Center Infrastructure Management (DCIM)

Software used to monitor and manage data center infrastructure, including equipment inventory, capacity, environmental conditions, and resource utilization. Unauthorized access may expose sensitive infrastructure information or interfere with operational management.

**Related risk:** FORGE-06.

## Embargoed Patch

A security update or vulnerability fix whose details or distribution are restricted until an agreed disclosure or release date. Organizations managing AI infrastructure must coordinate patch availability, vulnerability disclosure, and deployment timelines to reduce exposure during embargo periods.

**Related risk:** FORGE-10.

## Electrical Power Monitoring System (EPMS)

A system used to monitor and sometimes control electrical infrastructure, including power distribution equipment, generators, and uninterruptible power supplies. Compromise can affect the availability and reliability of data center power systems.

**Related risk:** FORGE-06.

## Firmware

Software embedded in hardware components that controls device initialization, configuration, or low-level operations. Unauthorized firmware modifications can undermine system integrity and may enable persistent compromise that conventional operating system security tools cannot easily detect.

**Related risks:** FORGE-01, FORGE-05.

## FORGE Domains

Five categories used by the FORGE framework to organize AI infrastructure security risks: Fleet integrity (F), Operations and management planes (O), Resource isolation (R), Grid (G), and Evidence and exposure management (E). These domains help readers understand which infrastructure areas are affected by each risk.

**Related risks:** FORGE-01 through FORGE-10.

## Hardware Root of Trust

A hardware-based foundation for security operations such as protecting cryptographic keys, verifying firmware, or supporting trusted measurements. Its effectiveness depends on the implementation, protected components, and security policies built upon it.

**Related risks:** FORGE-01, FORGE-05.

## InfiniBand

A high-performance networking architecture commonly used in high-performance computing and AI clusters. It supports low-latency communication and Remote Direct Memory Access, making fabric access controls, configuration, and isolation important security considerations.

**Related risk:** FORGE-02.

## Intelligent Platform Management Interface (IPMI)

A standardized interface for monitoring and managing server hardware, commonly implemented through a Baseboard Management Controller. Because IPMI can expose privileged hardware management functions, implementations require restricted access and appropriate authentication.

**Related risks:** FORGE-01, FORGE-04.

## Measured Boot

A boot process that records cryptographic measurements of selected firmware and software components as they load, often using a Trusted Platform Module. Unlike Secure Boot, measurement alone does not necessarily prevent unauthorized components from executing.

**Related risk:** FORGE-01.

## Neocloud

A cloud infrastructure provider or environment specializing in accelerated computing resources, particularly GPUs for AI and high-performance computing. Neocloud environments commonly rely on specialized networking, large accelerator clusters, and infrastructure that may be shared among multiple customers.

**Related risks:** FORGE-01 through FORGE-10.

## Out-of-Band (OOB) Management

Administration of infrastructure through management interfaces or communication paths that operate independently of the host operating system. These interfaces enable remote recovery and configuration but require strong authentication, network isolation, and monitoring because they provide privileged infrastructure access.

**Related risk:** FORGE-04.

## Redfish

A management standard developed by DMTF that provides RESTful interfaces for administering and monitoring servers and infrastructure components. Secure deployments require authentication, authorization, encrypted communications, and appropriate network access restrictions.

**Related risk:** FORGE-04.

## Remote Direct Memory Access (RDMA)

A technology that enables data transfer between memory regions on different systems with limited involvement from their CPUs. RDMA supports high-performance distributed computing but requires careful memory registration, access controls, and network isolation.

**Related risks:** FORGE-02, FORGE-03.

## RDMA over Converged Ethernet (RoCE)

A networking technology that enables RDMA communication over Ethernet infrastructure. RoCE deployments in AI clusters require appropriate network configuration, access controls, and segmentation to protect shared computing environments.

**Related risk:** FORGE-02.

## Secure Boot

A security mechanism that verifies cryptographic signatures or other trusted authorization information before allowing selected boot components to execute. Its protection depends on trusted keys, correctly configured policies, and the components covered by verification.

**Related risk:** FORGE-01.

## Software Bill of Materials (SBOM)

A structured inventory of software components and dependencies used in a product or system. SBOMs help infrastructure operators identify affected components, evaluate supply chain exposure, and prioritize vulnerability remediation.

**Related risk:** FORGE-05.

## Tenant Isolation

Security boundaries designed to prevent customers or workloads sharing infrastructure from accessing or interfering with one another's data, identities, execution environments, or resources. In AI infrastructure, effective isolation must account for accelerators, drivers, storage, networks, and residual data when resources are reassigned.

**Related risks:** FORGE-03, FORGE-07.

## References

The following standards and technical resources provide additional background for the terminology used in this glossary.

- [NIST SP 800-193 — Platform Firmware Resiliency](https://csrc.nist.gov/pubs/sp/800/193/final)
- [DMTF — Redfish Standards](https://www.dmtf.org/standards/redfish)
- [DMTF — Security Protocol and Data Model (SPDM)](https://www.dmtf.org/standards/spdm)
- [Trusted Computing Group — Specifications](https://trustedcomputinggroup.org/resource/)
- [OpenBMC Project](https://www.openbmc.org/)
- [Kubernetes Documentation — Multi-tenancy](https://kubernetes.io/docs/concepts/security/multi-tenancy/)
- [NIST — Software Supply Chain Security](https://csrc.nist.gov/Projects/ssdf)
- [Confidential Containers Project](https://confidentialcontainers.org/)

## Framework Risk References

- [FORGE-01 — Hardware and Firmware Integrity Compromise](Risks/FORGE-01-Hardware-And-Firmware-Integrity-Compromise.md)
- [FORGE-02 — Network and Interconnect Vulnerabilities](Risks/FORGE-02-Network-And-Interconnect-Vulnerabilities.md)
- [FORGE-03 — Unsafe Multi-Tenant Isolation and Resource Reuse](Risks/FORGE-03-Unsafe-Multi-Tenant-Isolation-And-Resource-Reuse.md)
- [FORGE-04 — Insecure Out-of-Band Management Plane](Risks/FORGE-04-Insecure-Out-Of-Band-Management-Plane.md)
- [FORGE-05 — AI Infrastructure Supply Chain Compromise](Risks/FORGE-05-AI-Infrastructure-Supply-Chain-Compromise.md)
- [FORGE-06 — Insecure Facility and Data Center Management Systems](Risks/FORGE-06-Insecure-Facility-And-Datacenter-Management-Systems.md)
- [FORGE-07 — Insecure Data and Artifact Handling](Risks/FORGE-07-Insecure-Data-And-Artifact-Handling.md)
- [FORGE-08 — Certification Gaps and Provider Transparency Failures](Risks/FORGE-08-Certification-Gaps-And-Provider-Transparency-Failures.md)
- [FORGE-09 — Insecure Operational Infrastructure Services](Risks/FORGE-09-Insecure-Operational-Infrastructure-Services.md)
- [FORGE-10 — Vendor Embargo Gaps and Patch Velocity Failures](Risks/FORGE-10-Vendor-Embargo-Gaps-And-Patch-Velocity-Failures.md)
