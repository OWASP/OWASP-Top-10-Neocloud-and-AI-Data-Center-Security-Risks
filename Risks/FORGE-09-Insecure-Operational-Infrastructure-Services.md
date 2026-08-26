# FORGE-09: Insecure Operational Infrastructure Services

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **High** | High | Medium | Medium |

## Definition

This risk involves compromise of privileged platforms and automation used to operate AI infrastructure, including job schedulers, cluster control planes, server-preparation workflows, configuration automation, and internal administrative interfaces. These services determine what runs, where it runs, and how GPU systems are prepared and maintained. A single compromise can give attackers broad control over training and inference environments.

## Description

AI infrastructure relies on an operational layer that converts GPU servers into usable training and inference capacity. Schedulers and cluster control planes (Slurm, Kubernetes, Ray) determine job placement and resource allocation. Server-preparation workflows handle OS environment installation, GPU drivers, container runtime, monitoring agents, and baseline security settings before workloads run on nodes. Configuration automation (Ansible or internal fleet-management tooling) applies system settings, runtime configuration, and security controls across large fleets. Internal administrative tools expose these actions through APIs, dashboards, and support workflows.

This risk emerges when schedulers, provisioning systems, admin tools, automation pipelines, or cluster-management services are exposed, weakly authenticated, over-permissioned, or insufficiently segmented from tenant and enterprise environments. A compromised administrator account, SaaS integration, automation token, CI/CD system, or service credential can transform trusted operational paths into attacker-controlled paths.

The severity stems from the broad authority these privileged services hold over high-value GPU nodes and workloads. An attacker controlling schedulers or configuration systems can manipulate workloads, push malicious changes, weaken isolation, disable monitoring, alter node state, or compromise rebuilt system integrity across the infrastructure.

## Impact and Failure Modes

### Compromised scheduler or orchestration controller

An attacker controlling Slurm, Kubernetes, Ray, or similar orchestration services can potentially submit jobs, modify workloads, change placement decisions, stop training runs, or access operational metadata across the cluster.

### Exposed or weakly controlled job-submission services

Job-submission APIs such as Ray Jobs can become direct execution paths into the cluster. Without appropriate controls, attackers can run unauthorized workloads on GPU infrastructure.

### Configuration automation blast radius

Configuration automation (Ansible or internal fleet-management tooling) often has broad administrative reach. Compromise of one automation path can push malicious settings, weaken isolation, disable monitoring, alter runtime configuration, or deploy persistence across many GPU nodes.

### Privileged internal admin interface abuse

Internal dashboards and platform APIs wrapping scheduler, server-preparation, or lifecycle functions can become direct control surfaces. Weak authentication, broad operator roles, or insufficient approval for high-impact actions can turn a single admin interface compromise into cluster-wide operational control.

### Node-preparation pipeline compromise

Server images, bootstrap scripts, driver installers, startup templates, and node preparation workflows define the trusted baseline for GPU hosts. If compromised, they can introduce backdoors, insecure configuration, vulnerable drivers, or disabled security controls into every node built or rebuilt from that pipeline.

### Service credential and automation token exposure

Tokens used by CI/CD systems, SaaS integrations, deployment tools, schedulers, or automation services may carry broad infrastructure permissions. If leaked or over-scoped, they can allow attackers to submit jobs, modify configuration, pull artifacts, or access operational systems without compromising an interactive user account.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Block tenant access to operational services:** Tenant workloads, notebooks, containers, and ordinary job networks should not reach schedulers, cluster control-plane APIs, configuration automation, node-preparation services, or privileged admin interfaces.

2. **Do not expose operational services broadly:** Operational services should not be reachable from the public internet or broad corporate networks. Restrict access to dedicated management networks, approved VPNs, bastion hosts, or controlled administrative workstations.

3. **Enforce strong authorization for operational actions:** Apply least privilege, RBAC, MFA where supported, and separation of duties for actions that submit jobs, alter cluster state, promote images, push configuration, or return nodes to service.

4. **Require approval and tamper-resistant audit records for fleet-wide actions:** Configuration pushes, server image changes, scheduler policy changes, and broad administrative actions should require explicit approval and generate audit records suitable for incident investigation.

5. **Protect node build and bootstrap files:** Restrict who can modify OS images, startup scripts, and templates used to prepare GPU nodes, and require review before changes are promoted.

6. **Monitor operational service behavior:** Alert on unusual job submissions, node build changes, configuration pushes, scheduler policy changes, and administrative access.

### For customers

1. **Do not expose your own job-submission and orchestration endpoints:** If running Ray, a Kubernetes dashboard, Jupyter/notebook server, or similar interfaces on rented GPUs, keep them off the public internet and behind authentication. Exposed orchestration endpoints (the ShadowRay pattern) are one of the most actively exploited paths into AI workloads, and on rented infrastructure they are usually yours to secure, not the provider's.

2. **Authenticate and scope your own automation:** Require authentication on any job-submission API you operate, and scope credentials/tokens used by CI/CD, automation, and SaaS integrations so a single leaked token cannot reach your whole environment.

3. **Check that nothing operational is exposed to the internet:** From outside, confirm that rented machines' external IPs do not expose schedulers, dashboards, job-submission APIs, admin interfaces, or metrics endpoints without authentication. The ShadowRay pattern is exactly this kind of exposure, found from the internet.

## Attack Scenarios

### ShadowRay: Ray Cluster Abuse

Ray is an open-source framework for distributed AI and Python workloads; its dashboard and Jobs API manage and submit work to Ray clusters. An attacker discovers an exposed Ray dashboard or job-submission interface used for AI workloads. Because the interface is reachable without appropriate access controls, the attacker submits unauthorized jobs through the same operational path used to run distributed workloads. Those jobs can consume GPU capacity, run attacker-controlled code, access environment variables or operational metadata, and create a foothold for follow-on activity. Oligo documented this pattern in its [ShadowRay](https://www.oligo.security/blog/shadowray-attack-ai-workloads-actively-exploited-in-the-wild) research on exposed Ray environments.

### Configuration Automation Compromise

An attacker compromises Ansible or an internal fleet automation tool managing GPU nodes. They push a change disabling security controls, adding an unauthorized key, or altering container runtime configuration. Because these tools apply changes across many systems, a single compromise can affect many nodes before defenders notice.

### Scheduler and Automation Pivot from a Low-Privilege Workload

An attacker compromises a low-privilege container used for preprocessing or job setup. Because the environment reuses service-account credentials and does not strongly separate workload, scheduler, and automation paths, the attacker pivots into job-control systems or supporting operational services. From there, they influence workload placement, access model storage, alter jobs, or reach administrative paths affecting the broader AI environment. A limited workload compromise becomes a wider operational compromise because trusted infrastructure services are over-permissioned or weakly segmented.

## References

- **NIST SP 800-223.** High-Performance Computing Security: Architecture, Threat Analysis, and Security Posture. NIST. HPC security architecture, threat model, and posture guidance relevant to AI infrastructure operations. https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-223.pdf
- **NIST SP 800-234.** High-Performance Computing (HPC) Security Overlay. NIST. Security control overlay for HPC environments, including systems used for AI and machine learning workloads. https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-234.pdf
- **CIS Kubernetes Benchmark.** Center for Internet Security. Kubernetes hardening benchmark. https://www.cisecurity.org/benchmark/kubernetes
- **Ray Security Documentation.** Ray. Security guidance for Ray clusters and interfaces. https://docs.ray.io/en/latest/ray-security/index.html
- **Ray Jobs API.** Ray. API for submitting and managing jobs on remote Ray clusters. https://docs.ray.io/en/latest/cluster/running-applications/job-submission/api.html
- **ShadowRay: AI Workloads Actively Exploited in the Wild.** Oligo Security, 2024. Reporting on exposed Ray environments abused against AI workloads. https://www.oligo.security/blog/shadowray-attack-ai-workloads-actively-exploited-in-the-wild
- **Slurm Authentication.** SchedMD. Authentication guidance for Slurm workload manager deployments. https://slurm.schedmd.com/authentication.html
