# The Top 10 Data Center and AI Infrastructure Security Risks

![FORGE - The Top 10 Data Center and AI Infrastructure Security Risks. Harden the metal beneath the model.](Images/forge-banner.png)

> **FORGE** - Harden the metal beneath the model.
>
> 🌐 **Website:** [https://forge-framework.io](https://forge-framework.io)

## Executive Summary

**AI datacenters are being built faster than they are being secured.**

The rapid expansion of datacenters, GPU clouds, and specialized compute environments has introduced security risks across hardware, networking, storage, orchestration, identity, management planes, and physical operations. Many of these risks resemble traditional datacenter or cloud security issues, but modern data centers and AI infrastructure changes their severity: systems originally designed for trusted operators are now supporting high-value, multi-tenant workloads from unrelated customers.

The **Top 10 Data Centers & AI Infrastructure Security Risks** provides a practical framework for identifying, prioritizing, and reducing the most important security risks in the infrastructure layer that powers AI. The framework defines the most critical failure modes in AI infrastructure and helps translate them into concrete security requirements.

### What This Framework Covers

This framework focuses on the security of AI infrastructure and the datacenters that house it: the physical hardware, networking fabrics, management planes, orchestration systems, storage systems, and operational environments on which AI workloads run.

It does not focus on AI models themselves or application-layer risks such as prompt injection, insecure agent behavior, model abuse, or model-level evaluation. Those risks are addressed by frameworks focused on other parts of the AI stack, including the [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/), [MITRE ATLAS](https://atlas.mitre.org/), the [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework), and [ISO/IEC 42001](https://www.iso.org/standard/81230.html). Together, these resources give practitioners a more complete picture of AI security, from governance and application-layer risks down to the underlying compute infrastructure.

Some risks in this document also exist in traditional datacenter and cloud environments. They are included here because AI infrastructure makes them materially more severe: shared high-value compute, complex accelerator clusters, dense management layers, and multi-tenant operations can turn ordinary infrastructure weaknesses into more severe security risks because the likelihood and impact of any incident are much higher than in traditional enterprise-level software and service deployments.

### Who This Framework Is For

**Neo-cloud providers** can benefit from the framework as a practical guide to the AI infrastructure threat landscape, attack surface review, environment hardening, security prioritization, and maturity demonstration.

**AI infrastructure customers** can benefit from the framework as a practical guide for procurement, security reviews, contractual requirements, provider comparison, and assessing resilience against realistic tenant-to-infrastructure compromise.

**Hybrid Cloud security teams** can benefit from this framework as a practical guide for configuring, maintaining, and securing their on-premise footprint against modern attacks, which can quickly pivot between environments.

## Acknowledgments

Built to evolve with the field, and kept open so the whole AI and security community can use it, challenge it, and improve it.

We would like to thank and acknowledge all experts which took part in reviewing and validating this document.

Have feedback or want to contribute? [research@lavalabs.io](mailto:research@lavalabs.io)

### Authors

- Michael Katchinskiy (Head of Security Research @ Lava)
- Yakir Kadkoda (CTO @ Lava)

### Reviewers

- Tony Rea (Global AI Infrastructure Lead @ Dell)
- Daniel Iziourov (Director of Platform Security @ Nebius)
- Vjaceslavs Klimovs (Senior Technical Director @ Roblox)
- Assaf Namer (Head of AI Security @ Google)
- Tyson Macaulay (Deputy Director @ NC CIPSER)
- Golan Ben-Oni (CIO/CISO @ IDT)
- Florina Ciorba (Associate Professor, Head of High Performance Computing group @ University of Basel)
- Arthur Reed (Security Engineer @ PNNL)
- Deumens Erik (Director Research Computing @ University of Florida)
- Saad Malik (CTO @ Spectro Cloud)
- Selim Aissi (Former Vice President, Global Information Security @ Visa & Intel)
- Guy Bilitski (Leading AI Operations @ SDS AI)
- Dan Farmer (Security Researcher)
- Michael Bargury (CTO @ Zenity)
- Amir Jerbi (Former CTO @ Aqua Security)
- Bill Stout (Former Technical Director, AI Product Security @ ServiceNow)
- Roey Yaacovi (CTO, DSPM & AI Security @ IBM)
- Guy Shanny (Co-Founder & CEO, Polar Security, acquired by IBM)
- Ziv Karliner (CTO @ Pillar)
- Assaf Morag (Security Researcher)
- James Berthoty (Founder & CEO @ Latio)

## The Five Domains of the FORGE Lens

FORGE domains define the evaluation lens: the infrastructure areas where AI security risk lives. The Risk Matrix below maps individual risks into these domains.

| Domain | Name | Description |
|--------|------|-------------|
| **F** | Fleet integrity | Trust in the hardware, firmware, software artifacts, images, dependencies, and supply paths that make up the AI infrastructure fleet. |
| **O** | Operations & management planes | Privileged systems used to control, automate, and administer AI infrastructure, including BMCs, schedulers, orchestration, automation, and admin tooling. |
| **R** | Resource isolation | The boundaries that separate tenants, workloads, execution environments, and reused infrastructure across shared AI systems. |
| **G** | Grid | The networking fabrics and facility systems that connect AI clusters and keep them powered, cooled, and operational. |
| **E** | Evidence & exposure management | The evidence customers need to understand provider security maturity, architectural scope, exposed services, and patch velocity. |

## The Top 10 Risks

FORGE IDs are ordered by severity, from highest to lowest. The matrix groups each risk by domain and shows its likelihood, impact, and detection difficulty.

| ID | Domain | Risk | Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|---|---|---|
| [FORGE-01](Risks/FORGE-01-Hardware-And-Firmware-Integrity-Compromise.md) | F | Hardware & Firmware Integrity Compromise | Critical | Medium | Severe | High |
| [FORGE-02](Risks/FORGE-02-Network-And-Interconnect-Vulnerabilities.md) | G | Network & Interconnect Vulnerabilities | Critical | Medium | Severe | High |
| [FORGE-03](Risks/FORGE-03-Unsafe-Multi-Tenant-Isolation-And-Resource-Reuse.md) | R | Unsafe Multi-Tenant Isolation and Resource Reuse | Critical | Low | Severe | Very High |
| [FORGE-04](Risks/FORGE-04-Insecure-Out-Of-Band-Management-Plane.md) | O | Insecure Out-of-Band Management Plane | Critical | Medium | High | Very High |
| [FORGE-05](Risks/FORGE-05-AI-Infrastructure-Supply-Chain-Compromise.md) | F | AI Infrastructure Supply Chain Compromise | Critical | High | High | High |
| [FORGE-06](Risks/FORGE-06-Insecure-Facility-And-Datacenter-Management-Systems.md) | G | Insecure Facility & Datacenter Management Systems | High | Low | High | Very High |
| [FORGE-07](Risks/FORGE-07-Insecure-Data-And-Artifact-Handling.md) | R | Insecure Data and Artifact Handling | High | High | High | Medium |
| [FORGE-08](Risks/FORGE-08-Certification-Gaps-And-Provider-Transparency-Failures.md) | E | Certification Gaps & Provider Transparency Failures | High | Medium | High | Medium |
| [FORGE-09](Risks/FORGE-09-Insecure-Operational-Infrastructure-Services.md) | O | Insecure Operational Infrastructure Services | High | High | Medium | Medium |
| [FORGE-10](Risks/FORGE-10-Vendor-Embargo-Gaps-And-Patch-Velocity-Failures.md) | E | Vendor Embargo Gaps & Patch Velocity Failures | Medium | Medium | Medium | Low |

Each risk follows a consistent structure: **Definition**, **Description**, **Impact and Failure Modes**, **Prevention and Mitigation Strategies** (for providers and for customers), **Attack Scenarios**, and **References**.

## Introduction

The rate at which the modern datacenter and AI infrastructure is being built is heavily outpacing the ability to secure it. When this infrastructure, GPU clusters, training pipelines, high-performance networking, and inference endpoints, is compromised, the blast radius is unlike conventional compute. Attackers gain access to proprietary models representing hundreds of millions of dollars, the power to poison foundational training data, and persistent footholds in the most highly privileged environments available.

This dynamic is reinforced by a structural market imbalance: GPU scarcity gives providers outsized leverage. When demand for accelerated compute far outstrips supply, customers often cannot choose their provider based on security posture – they take what is available. Providers face little market pressure to invest in security maturity, and customers accept risk they would not tolerate in conventional cloud. This imbalance is the backdrop against which every risk in this document should be read. This market pressure is becoming more dangerous as AI-enabled security research and exploitation capabilities compress the timeline for defenders. Weaknesses that might once have remained obscure for years may now be discovered, chained, and operationalized much faster.

**At the same time, AI-enabled security research and exploitation capabilities are compressing the timeline for defenders:** weaknesses that might once have remained obscure for years may now be discovered, chained, and operationalized much faster.

### Why AI Infrastructure Security Is Different

AI infrastructure sits at the intersection of cloud computing, high-performance computing, and physical datacenter operations. It uses familiar components such as servers, storage, networks, schedulers, management planes, and identity systems, but combines them in ways that change the security model.

Traditional HPC environments were often designed for trusted users, research communities, or internal operators. Modern AI infrastructure increasingly supports commercial, high-value, multi-tenant workloads from unrelated customers. As a result, assumptions that were acceptable in trusted or single-organization environments can become serious security risks when applied to shared AI infrastructure.

Securing AI infrastructure and data centers will require collaboration across cloud providers, hardware and networking vendors, security companies, system integrators, and customers. This is especially important in areas such as high-performance networking, east-west visibility, segmentation, management-plane protection, and infrastructure security controls.

### Recent Reporting

Attackers have already begun targeting the infrastructure, supply chains, and datacenter ecosystems that support advanced AI and HPC workloads. In some cases, the objective is direct theft of sensitive research, models, or engineering data, in others, it is espionage, prepositioning, or the ability to disrupt strategically important compute environments. Recent reporting illustrates both patterns:

- **Reported attacks on U.S. AI data centers** - In April 2025, [*TIME* reported](https://time.com/7279123/ai-datacenter-superintelligence-china-trump-report/) that researchers speaking with national security officials and datacenter operators learned of one case in which a top U.S. technology company's AI datacenter was attacked and intellectual property was stolen. They also described another case in which a similar facility was targeted through a specific unnamed component, had the attack succeeded, it could have taken the entire datacenter offline for months.
- **Nation-state targeting of AI infrastructure IP** - Federal prosecutions have confirmed that nation-states are directly targeting AI datacenter architecture at the hardware and systems level. [In one case](https://www.justice.gov/opa/pr/former-google-engineer-found-guilty-economic-espionage-and-theft-confidential-ai-technology), a former engineer at a major U.S. technology company was convicted of economic espionage after stealing thousands of pages of AI datacenter designs, including chip architecture, systems integration specifications, and orchestration software for large-scale model training.
- **Reported exfiltration from China's National Supercomputing Center in Tianjin** - In April 2026, [public reporting](https://covertaccessteam.substack.com/p/hacker-claims-10-petabytes-stolen) described claims that more than 10 petabytes of data had been stolen from the Tianjin supercomputing hub, a facility that reportedly supports thousands of research, industrial, and government users. Data claimed to originate from the breach was advertised for sale on Telegram channels. The incident illustrates the concentration risk created when high-value workloads, data, and research outputs are pooled in a single advanced compute environment.

These are early examples. The attack surface is likely to expand as the stack matures, and this document will evolve alongside it.

## Threat Actors

The risk is best understood by looking at potential attackers. The threat model spans far more than the classic "outside attacker," and the controls in this document are calibrated against the full set:

- **External attacker** - An unauthenticated or unauthorized actor with network access.
- **Compromised cloud customer** - A legitimate customer account, tenant, or workload that has been compromised and is operating on the provider's shared infrastructure.
- **Malicious enterprise insider** - An employee or contractor of the AI-consuming organization with legitimate access.
- **Malicious cloud contractor** - A third party with scoped access to the provider environment, such as hardware technicians, system integrators, logistics providers, or break-fix vendors.
- **Malicious cloud employee** - A provider-side employee with privileged access to hypervisors, BMCs, physical hosts, storage systems, or tenant data planes.
- **In-transit adversary** - An actor with access to hardware or components during manufacturing, staging, warehousing, shipping, customs handling, or delivery before they reach the provider's secured datacenter environment.
- **System integrator** - An organization responsible for assembling, configuring, or shipping complete systems from individual components.
- **Supplier** - An upstream hardware, firmware, or driver vendor.
- **AI agents** - Any AI agents acting on behalf of and impersonating any of the above.

### Enterprise Compromise as an Initial Access Vector

The risks in this framework focus on AI infrastructure-specific or AI infrastructure-amplified failure modes. They do not replace the need for strong enterprise security controls. In practice, many attacks against AI infrastructure may begin through familiar enterprise compromise paths, including weak or missing MFA, phishing, credential theft, SaaS compromise, and weak identity controls.

These paths should be treated as cross-cutting initial access vectors. A compromised identity, endpoint, SaaS account, CI/CD system, or administrative credential can provide the foothold needed to reach several of the risks described in this framework. These sections therefore focus on what can happen once an attacker reaches, or can influence, the AI infrastructure environment itself.

## How Risks Were Selected

Candidate risks were drawn from:

- Analysis of security research, incident reports, and published CVEs affecting AI-specific components.
- Review of vendor security documentation from NVIDIA, AMD, and major cloud providers.
- Observed vulnerability patterns across GPU cloud environments and ML orchestration platforms.
- Discussions with security practitioners working in AI infrastructure across multiple verticals.

Each candidate was evaluated against four dimensions, Likelihood, Impact, Exploitability, and Detection Difficulty, with the highest-scoring risks selected for inclusion.

## Versioning and Contributions

This is a living document. AI infrastructure is evolving rapidly, and so are the attacks against it. Future revisions will add new risks, refine existing ones, and incorporate lessons learned from the field. Feedback and proposed additions from practitioners are welcome and will be credited upon publication of the new version.

## License

Content in this repository is licensed under [CC BY-NC-SA 4.0](LICENSE).
