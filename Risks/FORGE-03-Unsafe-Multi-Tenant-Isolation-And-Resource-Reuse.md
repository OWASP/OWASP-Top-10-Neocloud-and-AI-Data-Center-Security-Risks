# FORGE-03: Unsafe Multi-Tenant Isolation and Resource Reuse

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **Critical** | Low | Severe | Very High |

## Definition

This risk occurs when shared, partitioned, or reassigned AI infrastructure fails to maintain reliable boundaries between tenants, workloads, or execution environments. It can allow one tenant to escape its boundary, observe residual state, discover neighboring workloads, affect the host, or degrade other tenants through shared compute, storage, orchestration, GPU, or interconnect resources.

## Description

AI infrastructure often relies on shared, partitioned, or rapidly reassigned compute resources to maximize scarce accelerator capacity. This creates tenant-isolation problems because workloads may share more than an application API - they may also share physical GPUs, bare-metal hosts, local storage, host kernels, driver stacks, container runtimes, orchestration layers, DNS/service discovery systems, RDMA fabrics, and high-speed interconnects.

This risk focuses on failures in how shared or reassigned capacity is isolated, cleaned, and reused across tenants, workloads, or execution environments. The risk arises when isolation at one layer is mistaken for isolation across the full stack. A workload may be separated by a Kubernetes namespace, container runtime, MIG partition, vGPU profile, or dedicated GPU assignment, while still sharing underlying host, driver, storage, network, or orchestration layers.

MIG, vGPU, sandboxed runtimes, and TEE-based confidential computing can strengthen specific parts of the boundary, but none provides complete tenant isolation on its own.

Isolation can fail actively or passively:
- **Active failures** include container or VM escapes, GPU runtime compromise, weak Kubernetes RBAC, unsafe device access, or abuse of shared RDMA and interconnect resources.
- **Passive failures** are quieter: residual VRAM contents, caches, local disk data, credentials, logs, or orchestration metadata may remain after a resource is reused.

This is not limited to live co-tenancy. A bare-metal GPU server may be leased to one customer, returned to the provider pool, and reassigned to another. If the reset process only reloads the operating system image, it may leave local disks, device state, firmware settings, driver state, cached artifacts, credentials, logs, or temporary workload data untouched. Without deliberate cleanup and verification across the full stack, the next tenant may inherit residual data or state.

## Impact and Failure Modes

### Host or cluster compromise through GPU runtime integration

GPU workloads often require privileged runtime components, device mounts, container hooks, drivers, and access to accelerator interfaces. Weaknesses in this stack can allow a crafted container image or tenant workload to access the host filesystem, runtime sockets, credentials, mounted paths, GPU device interfaces, or neighboring workloads on the same node. Example: CVE-2024-0132, an NVIDIA Container Toolkit container escape.

### GPU virtualization failures can cross the guest-host boundary

In virtualized GPU environments, a tenant VM may affect the host through the GPU virtualization or management layer. Example: CVE-2024-0099, an NVIDIA vGPU issue where a guest OS could trigger a buffer overrun in the host Virtual GPU Manager.

### Weak orchestration and service discovery boundaries

Kubernetes namespaces do not provide strong tenant isolation by themselves. Reused service accounts, weak RBAC, shared node pools, unsafe runtime classes, broad device assignment, or shared DNS and service discovery can allow one tenant to discover or reach resources associated with another tenant.

### Cross-tenant interference through shared RDMA or interconnect resources

A tenant may not need to compromise another tenant directly to affect their workload. In shared AI clusters, abuse or overconsumption of RDMA, NIC, or interconnect resources can degrade neighboring jobs, increase latency, reduce bandwidth, or stall distributed training synchronization.

### GPU state and side channels may leak across contexts

Research such as Whispering Pixels and NVBleed has shown that GPU state, memory behavior, and multi-GPU interconnect activity can leak information across execution contexts under specific conditions. These are research findings rather than routine production attacks, but they remain an open problem where mutually untrusted workloads share GPUs or interconnects.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Harden GPU runtime boundaries:** Keep GPU drivers, NVIDIA Container Toolkit, device plugins, and container runtimes patched. Restrict privileged containers, hostPath mounts, broad device access, unsafe container hooks, and unnecessary access to accelerator interfaces. Treat GPU runtime integration as part of the tenant-isolation boundary, not as a trusted implementation detail.

2. **Treat bare-metal reassignment as a security boundary:** Before a bare-metal GPU server is reassigned to another tenant, reset and revalidate the full stack - not only the operating system. This includes GPU and device state, MIG configuration, local storage, cached artifacts, orchestration metadata, host identifiers, credentials, and tenant-specific logs, and includes confirming firmware and driver state is known-good rather than inherited from the prior tenant.

3. **Separate high-risk workloads by tenancy model:** Do not treat containers, Kubernetes namespaces, vGPU, MIG, dedicated GPUs, dedicated hosts, and dedicated bare-metal clusters as equivalent isolation models. Use stronger boundaries for untrusted tenants, sensitive training workloads, regulated data, or customers requiring no co-tenancy.

4. **Harden orchestration boundaries:** Use per-tenant namespaces, least-privilege service accounts, strong RBAC, admission controls, dedicated runtime classes where appropriate, and separate node pools for higher-trust and lower-trust workloads. Avoid sharing service accounts, runtime permissions, device plugins, or management components across tenants unless the isolation model has been explicitly reviewed.

5. **Protect service discovery and DNS boundaries:** Ensure that tenants cannot enumerate other tenants' services, records, hostnames, internal architecture, or infrastructure metadata through reverse DNS, shared DNS zones, Kubernetes service discovery, or predictable naming schemes. Restrict cross-tenant DNS visibility, avoid tenant-identifying names in shared records, and test service discovery exposure from a normal tenant position.

6. **Isolate shared model-loading paths:** Treat model loaders, inference services, notebook runtimes, and artifact-processing pipelines as tenant-boundary components when they process customer-supplied files. Do not load untrusted models, checkpoints, or serialized artifacts inside shared or highly privileged services unless the loading path is sandboxed and isolated.

7. **Offer stronger isolation and attestation options for high-sensitivity workloads:** Where the platform supports it, provide dedicated hosts, dedicated clusters, or hardware-backed confidential computing / TEE-based isolation with remote attestation for customers that require stronger protection from the host, hypervisor, or provider-managed layer. Clearly document whether capabilities such as NVIDIA GPU Confidential Computing on H100/H200 are available, what they protect, what remains trusted, and what evidence customers can verify before workload execution.

8. **Keep the GPU virtualization stack patched:** Quickly patch host-side GPU management components, drivers, and device plugins in vGPU, MIG, or other GPU-sharing setups (for example CVE-2024-0099).

9. **Reduce cross-context leakage risk:** Clear or reinitialize GPU memory, including VRAM, as well as GPU contexts, runtime state, MIG or vGPU configuration, and other residual accelerator state before a GPU, MIG slice, vGPU, or bare-metal node is reassigned to another tenant. Restrict low-level diagnostic, profiling, and debug interfaces unless they are required and safely scoped to the tenant.

10. **Control shared RDMA and interconnect resource abuse:** Monitor fabric utilization, congestion, error counters, packet drops, abnormal bandwidth patterns, and synchronization delays. Use fabric partitioning, tenant-level limits where available, congestion monitoring, and isolation of high-sensitivity workloads. Treat RDMA, NIC, and interconnect resources as part of the tenant boundary in distributed AI environments.

### For customers evaluating a provider

How much you can check depends on the offering. On a dedicated or bare-metal node you can inspect the state you are handed. On MIG, vGPU, or fractional offerings you see less and rely more on the provider.

1. **Check the GPU is clean when you receive it:** At the start of a lease, confirm that no unexpected GPU processes, compute contexts, or unexplained memory usage are visible, for example with nvidia-smi. Some memory may be reserved by the driver or platform, so this is a sanity check rather than proof of secure scrubbing. On a shared GPU, seeing processes or contexts outside your own work is a warning sign. On a MIG instance, vGPU, or VM, you should see only the resources and boundary assigned to you.

2. **Check local scratch storage and caches for residual data:** Inspect any local NVMe or SSD scratch disks attached to the node, along with common cache locations, for leftover datasets, checkpoints, logs, containers, credentials, or model files from a previous tenant. Finding any such data is direct evidence that tenant reset and cleanup are incomplete.

3. **Confirm which isolation model you actually have:** Verify whether you have a hardware-partitioned MIG instance, a vGPU slice, a dedicated GPU, a dedicated host, or a full bare-metal node. These models provide different visibility, performance, and isolation properties and should not be treated as equivalent.

## Attack Scenarios

### Host Access Through GPU Runtime Escape

A tenant submits a crafted GPU container image to a managed AI environment. The image abuses CVE-2024-0132, a vulnerability in NVIDIA Container Toolkit, or a similar weakness in the GPU runtime integration path. The tenant workload gains access to host files or GPU device interfaces outside the assigned workload boundary. From the host, the attacker may reach credentials, mounted paths, runtime sockets, or neighboring workloads on the same node.

### Cross-Tenant Service Discovery Through Reverse DNS

A tenant receives access to an environment that is intended to be isolated from other tenants. From that tenant position, the attacker performs reverse DNS lookups against shared infrastructure address space and discovers Kubernetes service names or records associated with other tenants. The attacker does not need initial access to those tenants' workloads. The failure is that shared service discovery or DNS metadata exposes cross-tenant information that should not be visible. This can reveal tenant identities, service names, internal architecture, workload purpose, or possible follow-on targets.

### Cross-Tenant Interference Through Shared RDMA Resources

A tenant runs a workload that heavily consumes shared RDMA, NIC, or interconnect resources in a multi-tenant AI cluster. Neighboring distributed training jobs experience bandwidth degradation, latency inflation, or stalled synchronization. The attacker does not need to read another tenant's data. The isolation failure is that one tenant can materially degrade another tenant's execution environment through shared low-level resources.

### Residual State Exposure After Bare-Metal Reassignment

A bare-metal GPU server is leased to one tenant, returned to the provider pool, and then assigned to another tenant. The provider reinstalls the operating system but does not fully reset local storage, GPU state, cached artifacts, host identifiers, runtime configuration, or orchestration metadata. The next tenant discovers traces of the previous environment, exposing information that should have been removed before reassignment.

## References

- **Wiz Research, Critical NVIDIA AI Vulnerability, 2024.** Deep dive on NVIDIA Container Toolkit escape, CVE-2024-0132. https://www.wiz.io/blog/wiz-research-critical-nvidia-ai-vulnerability
- **Wiz Research, NVIDIA AI Vulnerability Deep Dive, CVE-2024-0132, 2024.** https://www.wiz.io/blog/nvidia-ai-vulnerability-deep-dive-cve-2024-0132
- **NVIDIA vGPU software issue, CVE-2024-0099.** Guest OS to host Virtual GPU Manager buffer overrun.
- **Whispering Pixels, Exploiting Uninitialized Register Accesses in Modern GPUs.** Cross context GPU leakage across workloads. https://arxiv.org/abs/2401.08881
- **NVIDIA Confidential Containers Reference Architecture.** NVIDIA. Describes NVIDIA GPU Confidential Computing support for running GPU workloads inside a hardware-enforced Trusted Execution Environment on Kubernetes. https://docs.nvidia.com/datacenter/cloud-native/confidential-containers/latest/overview.html
- **NVBleed, Covert and Side Channel Attacks on NVIDIA Multi GPU Interconnect.** Side channel attacks over NVLink interconnects. https://arxiv.org/abs/2503.17847
- **Kubernetes Multi-tenancy Documentation.** Guidance on namespace isolation, RBAC, and workload separation. https://kubernetes.io/docs/concepts/security/multi-tenancy/
- **ReDMArk, Bypassing RDMA Security Mechanisms.** Rothenberger et al., USENIX Security 2021. Demonstrates RDMA credential weaknesses enabling unauthorized memory access. https://www.usenix.org/system/files/sec21-rothenberger.pdf
- **Confidential Containers (CoCo) Project.** CNCF project for running cloud-native workloads inside hardware-backed Trusted Execution Environments with attestation support. https://confidentialcontainers.org/
