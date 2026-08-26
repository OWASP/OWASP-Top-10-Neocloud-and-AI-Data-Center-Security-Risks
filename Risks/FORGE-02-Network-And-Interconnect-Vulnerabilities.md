# FORGE-02: Network & Interconnect Vulnerabilities

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **Critical** | Medium | Severe | High |

## Definition

Network & Interconnect Vulnerabilities arise when high-performance fabrics used by AI clusters-including InfiniBand, RoCE, and RDMA-based communication paths-lack proper isolation, monitoring, or protection. These fabrics prioritize throughput and low latency, so weak controls can expose backend AI environments to discovery, lateral movement, data interception, denial of service, or workload manipulation.

## Description

Modern AI infrastructure uses high-performance interconnects for moving model weights, gradients, checkpoints, and other data across GPU clusters. These environments commonly use [InfiniBand](https://network.nvidia.com/pdf/whitepapers/IB_Intro_WP_190.pdf) and [RoCE](https://en.wikipedia.org/wiki/RDMA_over_Converged_Ethernet), often through RDMA, providing low-latency communication with reduced dependence on the traditional kernel networking path. This boosts performance but can diminish visibility and control available to conventional network and host security tools.

A recurring weakness involves the gap between the frontend network and the backend training fabric. Frontend networks for APIs, orchestration, monitoring, and administrative access typically receive stronger access control and monitoring. Backend fabrics carrying distributed training and storage traffic may be flatter, less inspected, and weakly segmented. When fabric isolation, management controls, or workload boundaries are weak, even routine diagnostic queries from a tenant node can reveal topology, endpoints, and management details that should not be visible.

The security model varies by transport. InfiniBand uses fabric-specific controls like partitions, management keys, and subnet-management policies. RoCE operates over Ethernet without an InfiniBand Subnet Manager, so it depends more on Ethernet/IP segmentation, access controls, and device isolation. In both cases, weak defaults or poorly protected management interfaces can turn a performance fabric into a path for reconnaissance, lateral movement, or disruption.

## Impact and Failure Modes

### Backend traffic may lack cryptographic protection

Fabric traffic often lacks default encryption. In native InfiniBand, keys like P_Key and M_Key are access-control mechanisms rather than cryptographic protections. In RoCE and other Ethernet-based deployments, model weights, gradients, and training data may traverse the backend network without wire-level encryption unless operators explicitly deploy protections like MACsec or IPsec where supported.

### InfiniBand partitioning may be weak or left at defaults

InfiniBand relies on P_Key for partition-level isolation. Without partition configuration, OpenSM can place end ports in the broad default partition (commonly 0x7fff), increasing lateral reachability and the blast radius of a compromised node. Hardened or vendor-managed deployments may override these defaults, so exposure depends on actual subnet-manager configuration.

### InfiniBand management-plane protection may be weak

InfiniBand uses M_Key to protect management operations on switches and adapters. In OpenSM, documented defaults such as m_key=0 and m_key_protection_level=0 leave management-key protection disabled or at the weakest level unless operators explicitly harden them. Combined with broad fabric reachability, this can expose topology, port state, and management functions that should be restricted.

### RDMA memory-access credentials may be abused

RDMA uses keys like rkeys to authorize access to registered memory regions. Research demonstrates that weaknesses in RDMA security mechanisms can enable unauthorized reads or writes in some environments, with limited visibility to normal host-based security tools.

### Fabric management software can become a control point

Vulnerabilities in fabric management software can expose the management plane to privilege escalation, tampering, denial of service, and broader fabric compromise. For example, [CVE-2024-0130](https://nvd.nist.gov/vuln/detail/CVE-2024-0130) affected NVIDIA UFM and could allow improper authentication via the Ethernet management interface.

### RDMA isolation can be weak in containerized environments

In Kubernetes and similar platforms, shared HCA access or shared RDMA-capable devices can weaken workload isolation if operators rely only on ordinary pod-network controls without separately isolating RDMA devices, namespaces, or virtual functions.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Enforce protocol-specific fabric controls, do not rely on defaults:** For InfiniBand, explicitly configure and review controls like P_Key, M_Key, SM_Key, SA_Key, and allowed_sm_guids. For RoCE (running over Ethernet with no subnet manager), apply explicit Ethernet/IP segmentation such as VLANs and ACLs. Many protections are weak or off in upstream defaults-confirm they are actually set rather than assuming the fabric is isolated.

2. **Harden and isolate the fabric management plane:** Restrict access to subnet and fabric management interfaces, keep management traffic off untrusted networks, limit topology discovery and sensitive management operations from untrusted nodes, and monitor counters and management events for signs of spoofing or misconfiguration.

3. **Provide backend isolation comparable to frontend isolation:** Give the RDMA training fabric documented segmentation, workload boundaries, and tenant isolation rather than operating it as a flat internal network.

4. **Use wire-level encryption where supported and required:** For high-sensitivity workloads, deploy MACsec or IPsec on RoCE and other Ethernet-based fabrics where supported. Native InfiniBand keys like P_Key and M_Key are access controls, not encryption. In-fabric encryption on native InfiniBand is hardware-dependent and not always available, so treat backend traffic as unencrypted unless a specific protection is in place.

5. **Harden orchestration and RDMA device isolation:** Use Kubernetes NetworkPolicies for ordinary pod traffic, assign unique least-privilege service accounts per job, and regularly audit and rotate credentials. Do not assume Kubernetes-layer controls alone secure RDMA-capable traffic, as RDMA may bypass or be enforced outside the ordinary pod-networking path where NetworkPolicies apply. For RDMA-capable workloads, isolate underlying devices and network namespaces separately-for example, by assigning dedicated SR-IOV virtual functions to each tenant or job rather than having multiple tenants share the same HCA interface.

### For customers evaluating a provider

1. **See how much of the fabric you can enumerate:** From your node, run standard diagnostic queries such as ibstat, ibv_devinfo, saquery, sminfo, and (where permitted) ibnetdiscover. You should see enough information to operate your assigned ports but not enough to map unrelated tenants, switch topology, or management-plane details. If you can enumerate the full fabric, discover other tenants' nodes, or access management functions outside your allocation, that suggests weak fabric partitioning or management-plane protection.

2. **Check your partition membership:** Confirm which P_Key partitions your ports belong to (e.g., via the P_Key table under `/sys/class/infiniband/<device>/ports/<port>/pkeys/` or vendor tools). If your port appears in the broad default partition (commonly 0x7fff or full-membership 0xffff) with wide reachability rather than a dedicated tenant/job partition, isolation may have been left at defaults.

3. **For RoCE / Ethernet fabrics, scan your backend subnet:** Since RoCE rides on Ethernet, ordinary IP tools apply: check your ARP table, examine your subnet mask (is the backend a flat /16 with everyone on it?), and determine whether you can reach other hosts' IPs and ports on the storage or training network. Reaching other tenants' addresses indicates missing VLAN/segmentation.

4. **Look at port counters for noisy-neighbor signs:** Your own port error and congestion counters (e.g., via perfquery or `/sys/class/infiniband/<device>/ports/<port>/counters/`) can show link errors, congestion, retransmits, drops, or other symptoms that may indicate shared-fabric contention. These counters are useful evidence but should be interpreted against a baseline and your workload profile rather than treated as proof of co-tenant interference.

## Attack Scenarios

### Fabric Management Abuse

An attacker gains a foothold on a tenant node through a compromised container, workload, or account. From that node, they use fabric diagnostic and management queries to map the backend interconnect, discover topology, identify node addresses, and inspect port or switch state. If the InfiniBand fabric is weakly partitioned (e.g., broad use of the default P_Key 0x7fff) and management protections like M_Key are unset or weakly configured, the attacker may gain visibility or influence over fabric behavior outside their allocation. In a hardened environment, tenant nodes should not be able to enumerate unrelated topology or perform management actions beyond their assigned fabric scope.

### RDMA Memory Exposure

An attacker abuses weaknesses in RDMA memory-access protections to obtain or misuse credentials like rkeys for registered memory regions. They then issue one-sided RDMA reads or writes against memory that should not be accessible to them. Because one-sided RDMA operations can complete without active involvement from the remote CPU, the attacker may extract training data, checkpoints, gradients, or model-related buffers with limited visibility to normal host-based security tools.

### Orchestration Collapse

A breach begins in a low-privilege container used for preprocessing or job setup. Because the environment reuses service account credentials and does not enforce strong workload separation, the attacker pivots into higher-value GPU workloads and supporting services. Once inside the trusted backend environment, weak segmentation between orchestration, storage, and the training fabric allows the compromise to spread beyond the original container, turning a limited application breach into broader access to model storage, job control, and distributed training infrastructure.

### RDMA Noisy Neighbor

In a shared AI environment, a malicious tenant abuses RDMA-capable devices, shared HCAs, or interconnect resources to consume bandwidth, trigger congestion, or exhaust RNIC resources. Neighboring distributed training jobs may experience degraded throughput, higher latency, failed synchronization, or stalled execution. The attacker does not need to read or modify another tenant's data-the failure is that one tenant can materially degrade another tenant's workload through shared low-level fabric resources.

## References

- **NVIDIA InfiniBand Security Overview and Guidelines.** NVIDIA. Vendor guidance on IB fabric security controls including P_Key, M_Key, and subnet management. https://docs.nvidia.com/networking/display/nvidiainfinibandsecurityoverviewandguidelines/security+in+infiniband
- **OpenSM Partition Configuration.** linux-rdma. P_Key partition configuration reference for InfiniBand subnet managers. https://github.com/linux-rdma/opensm/blob/master/doc/partition-config.txt
- **NVIDIA MLNX_OFED, Kubernetes with Shared HCA.** NVIDIA. RDMA device sharing and isolation in Kubernetes environments. https://docs.nvidia.com/networking/display/mlnxofedv23106161lts/Kubernetes-with-Shared-HCA
- **opensm(8) man page, M_Key Configuration.** Debian. OpenSM management key defaults and hardening options. https://manpages.debian.org/testing/opensm/opensm.8.en.html#MKEY_CONFIGURATION
- **OpenSM Source, osm_base.h.** linux-rdma. Default M_Key and protection level constants in source code. https://github.com/linux-rdma/opensm/blob/master/include/opensm/osm_base.h
- **Set up InfiniBand on HPC VMs.** Microsoft Azure. InfiniBand configuration for cloud HPC and AI workloads. https://learn.microsoft.com/en-us/azure/virtual-machines/setup-infiniband
- **CVE-2024-0130.** NVIDIA UFM improper authentication via Ethernet management interface. https://nvd.nist.gov/vuln/detail/CVE-2024-0130
- **CVE-2024-0101.** Mellanox OS ipfilter bypass. https://nvd.nist.gov/vuln/detail/CVE-2024-0101
- **ReDMArk: Bypassing RDMA Security Mechanisms.** Rothenberger et al., USENIX Security 2021. Demonstrates weaknesses in RDMA memory-access credentials enabling unauthorized remote reads and writes. https://www.usenix.org/system/files/sec21-rothenberger.pdf
- **NeVerMore: Exploiting RDMA Mistakes in NVMe-oF Storage Applications.** Taranov et al., arXiv 2022. RDMA vulnerabilities in storage applications. https://arxiv.org/abs/2202.08080
- **Distributed Training Attack Surface.** Red Teams AI. Security analysis of distributed training communication and orchestration. https://redteams.ai/topics/training-pipeline/advanced/distributed-training
- **Noisy Neighbor: Exploiting RDMA for Resource Exhaustion in Containerized Clouds.** SecAssure 2025. RDMA-based denial-of-service in shared container environments. https://arxiv.org/abs/2510.12629
- **RFC 4391, Transmission of IP over InfiniBand.** IETF. Section 13 covers security considerations for IPoIB deployments. https://www.rfc-editor.org/rfc/rfc4391
- **NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations.** https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
