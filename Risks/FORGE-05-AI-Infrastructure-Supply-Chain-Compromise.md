# FORGE-05: AI Infrastructure Supply Chain Compromise

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **Critical** | High | High | High |

## Definition

AI Infrastructure Supply Chain Compromise refers to introducing compromised, tampered, or untrusted components into AI environments through upstream supply-chain relationships. This encompasses hardware, firmware, software artifacts, images, and infrastructure templates crossing from vendors, integrators, registries, public sources, or internal mirrors into an operator's trusted environment.

## Description

The AI infrastructure supply chain has two lanes. The physical lane covers servers, accelerators, NICs, storage devices, BMCs, and other components passing through manufacturers, integrators, logistics providers, and refurbishment paths before reaching production. Systems may arrive with firmware, management configuration, or node images prepared by third parties without independent operator verification.

The digital lane covers container images, GPU drivers and runtimes, Python packages, Kubernetes operators, model-serving components, infrastructure-as-code templates, and model artifacts from vendor catalogs, public registries, open-source repositories, or internal mirrors. These may be legitimate, maliciously tampered, outdated, misconfigured, or sourced unexpectedly.

Both lanes share the same core problem: components are often trusted because they arrive through an expected path, not because their integrity and provenance were independently verified. This dependency persists throughout the product lifecycle as operators rely on vendors for cryptographic mechanism updates as standards evolve.

Much of this risk is addressable with widely available controls: signed images/artifacts, build provenance, dependency pinning, SBOMs, registry scanning, controlled internal mirrors, and safer model formats like safetensors. Exposure concentrates where these controls aren't adopted or where artifacts flow directly from public sources into production.

> **See also:** [FORGE-10: Vendor Embargo Gaps & Patch Velocity Failures](FORGE-10-Vendor-Embargo-Gaps-And-Patch-Velocity-Failures.md) for container runtime and driver patching risks.

## Impact and Failure Modes

### Unverified third-party staging or integration

A system integrator, reseller, or logistics partner preloads firmware, management credentials, BIOS settings, or node images before delivery. Accepting systems without independent verification means inheriting whatever configuration, vulnerability, or implant was introduced upstream.

### Unsafe model checkpoint loading

Some model-loading paths, especially pickle-based formats, can execute code during deserialization. A malicious or tampered model artifact turns model loading into code execution on training, evaluation, or inference nodes.

### Poisoned AI infrastructure dependency or container image

A compromised CUDA or NCCL dependency, GPU operator, device plugin, model-serving image, or privileged base image can introduce credential theft, backdoors, or cluster-level access into GPU nodes and Kubernetes environments that automatically pull or update trusted infrastructure components.

### Tampered internal mirror or registry

An attacker compromising an internal package index, container registry, model registry, or artifact repository can replace trusted components with malicious versions. Because production systems often trust internal mirrors more than public sources, the compromise can spread quickly across workloads, nodes, or clusters.

### Compromised node image or provisioning template

A malicious or accidental change to a golden image, bootstrap script, Kubernetes manifest, Terraform module, or build pipeline can propagate to every GPU worker provisioned from that artifact.

### Lack of cryptographic agility in long-lived systems

AI/HPC platforms may remain in service for many years while depending on vendors to update firmware signing, key exchange, digital signatures, and authentication mechanisms. If hardware or firmware cannot support newer algorithms (including post-quantum requirements), operators face using outdated cryptography or replacing the platform.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Verify before trust at every handoff:** Independently verify firmware versions, BIOS/management settings, installed images, and component inventory when receiving systems from integrators, resellers, logistics partners, manufacturers, or refurbishment paths. Re-establish trust after any third-party staging, repair, or return.

2. **Control and restrict artifact sources:** Mirror approved packages, container images, GPU drivers, model artifacts, Helm charts, and IaC templates into controlled internal registries. Don't allow production systems to pull directly from unreviewed public registries. Pin images by immutable digest, lock dependency versions, and require signatures, checksums, or build provenance for artifacts in privileged infrastructure paths.

3. **Treat privileged AI infrastructure components as high-risk:** Apply extra scrutiny to components running with cluster or node privilege (GPU operators, device plugins, CUDA/NCCL dependencies, drivers, privileged base/model-serving images). Compromise in these components reaches GPU nodes or the Kubernetes control plane directly; they should come only from verified sources pinned by digest.

4. **Restrict build and provisioning systems:** Limit cloud credentials, registry write access, Kubernetes privileges, and outbound connectivity for image builders, CI/CD pipelines, and fleet automation. A compromised build step shouldn't exfiltrate secrets, overwrite trusted artifacts, or fetch follow-on payloads.

5. **Maintain inventory and prepare for recall:** Track SBOMs, hardware bills of materials, image digests, firmware versions, dependency versions, and deployment locations so compromised artifacts/components can be identified, blocked, and removed quickly fleet-wide.

### For customers and teams running their own pipelines

A meaningful share of supply-chain risk can be reduced in your own pipeline, while the physical hardware and provider-managed base-image lane requires asking the provider for evidence.

1. **Pin and verify what you pull:** Pin container images by digest, lock dependency versions, and verify signatures so upstream changes cannot silently enter your environment.

2. **Prefer safer model-loading paths:** Load models from safer formats like safetensors rather than pickle-based checkpoints where possible; scan model artifacts before loading into privileged training, evaluation, or inference environments.

3. **Mirror and scan instead of pulling directly from public sources:** Pull packages, images, and models through a controlled internal mirror that scans artifacts before promotion to production.

4. **Limit what build and training jobs can reach:** Scope credentials and outbound network access for CI/CD and training jobs so a compromised dependency cannot exfiltrate secrets or fetch follow-on payloads.

## Attack Scenarios

### Typosquatted ML Package

An attacker publishes a malicious package with a name similar to a popular ML dependency, or compromises a legitimate upstream package. A data scientist installs/updates it during environment setup. The package executes during installation or import, stealing credentials, model artifacts, or training data, or establishing a foothold.

### Malicious Model Artifact in a GPU Pipeline

A team downloads a popular pretrained model from a public hub and loads it directly into a privileged training or inference environment. Incidents involving malicious pickle-based checkpoints on Hugging Face showed that model loading can become code execution when unsafe serialization formats are used. Even non-executable formats may contain poisoned models producing attacker-controlled behavior on selected inputs.

### Compromised System Integrator Baseline

A provider receives GPU servers from a third-party integrator that assembled, racked, and imaged systems before delivery. Servers arrive with a pre-installed OS, firmware updates, and management configuration applied during staging. The provider verifies boot and health checks but doesn't independently verify the pre-installed OS image contents. The image contains a persistent backdoor surviving the standard onboarding process.

### Compromised Firmware Signing Key

An attacker obtains a server vendor's firmware signing key and produces a malicious BMC firmware image passing signature verification. They upload it to the vendor's firmware support portal. The provider downloads and deploys it fleet-wide as routine. Once installed, it provides persistent remote access to the management plane on every node that accepted the update.

## References

- **NIST SP 800-218, Secure Software Development Framework.** NIST. Secure SDLC practices for software producers. https://csrc.nist.gov/pubs/sp/800/218/final
- **OpenSSF Scorecard.** Open Source Security Foundation. Automated security health checks for open-source projects. https://github.com/ossf/scorecard
- **PyTorch torch.load() Documentation.** PyTorch. Documents unsafe deserialization risks in pickle-based model loading. https://docs.pytorch.org/docs/stable/generated/torch.load.html
- **Pickle Scanning and Pickle Security.** Hugging Face. Model artifact malware scanning on the Hub. https://huggingface.co/docs/hub/security-pickle
- **Hugging Face Hub Security.** Hugging Face. Platform-level model and artifact security controls. https://huggingface.co/docs/hub/security
- **Safetensors, Secure Model Weight Format.** PyTorch / Hugging Face. Safe alternative to pickle-based model serialization. https://pytorch.org/projects/safetensors/
- **NVIDIA NGC, Signed Container Images.** NVIDIA. Verified GPU container image provenance. https://docs.nvidia.com/ngc/gpu-cloud/ngc-catalog-user-guide/index.html
- **Chainguard Images Documentation.** Chainguard. Minimal, hardened container base images. https://edu.chainguard.dev/chainguard/chainguard-images/
- **Exporting a Software Bill of Materials.** GitHub. SBOM generation for dependency tracking. https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/exporting-a-software-bill-of-materials-for-your-repository
- **Data Scientists Targeted by Malicious Hugging Face ML Models with Silent Backdoor.** JFrog, 2024. Backdoored pickle checkpoints discovered on Hugging Face Hub. https://jfrog.com/blog/data-scientists-targeted-by-malicious-hugging-face-ml-models-with-silent-backdoor/
