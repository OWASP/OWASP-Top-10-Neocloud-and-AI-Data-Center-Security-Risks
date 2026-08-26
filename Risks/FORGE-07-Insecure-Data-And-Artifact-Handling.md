# FORGE-07: Insecure Data and Artifact Handling

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **High** | High | High | Medium |

## Definition

Insecure Data and Artifact Handling is when weak controls permit attackers, insiders, compromised workloads, or privileged operators to access, copy, modify, or export sensitive AI data and derived artifacts. This encompasses training/fine-tuning datasets, evaluation data, embeddings, annotations, prompts, logs, checkpoints, model weights, adapters, tokenizer files, cached copies, backups, and packaged artifacts used throughout AI development and deployment.

## Description

AI systems rely on high-value data and derived artifacts that are collected, stored, transformed, moved, cached, and reused across the AI lifecycle. These assets may contain sensitive customer information, proprietary business knowledge, regulated data, model behavior, or results of significant engineering and compute investment.

The risk is broader than model theft. AI data exposure can reveal training sources, user activity, sensitive information, proprietary methods, or model behavior. Modified data or artifacts can poison downstream training, fine-tuning, evaluation, or deployment. Though mechanics may resemble ordinary data exfiltration or tampering, the impact is AI-specific because these assets can be reused, reconstructed, or silently influence future systems.

This risk emerges when AI data and artifacts are stored, transferred, backed up, cached, mounted, or loaded into runtime environments without strong access controls, encryption, tenant isolation, integrity validation, lifecycle management, and auditability. In AI infrastructure, this includes datasets, object stores, model registries, experiment systems, local caches, backups, and high-performance or distributed storage systems.

If these paths are weakly isolated, over-permissioned, unaudited, or poorly cleaned between workloads or tenants, a compromised identity, compute node, tenant workload, notebook, data pipeline, storage client, registry, or deployment system can turn an initial foothold into long-term loss of confidentiality, integrity, provenance, and control.

## Impact and Failure Modes

### Unprotected model storage and backups

Model weights, checkpoints, or packaged model files in shared file systems, object stores, snapshots, or backup locations can be copied by anyone with read access. Secondary storage locations frequently receive weaker controls than primary model repositories.

### Overbroad model registry or artifact-store access

Model registries and artifact stores can hold approved production models, deployment metadata, adapters, tokenizer files, evaluation outputs, and packaged artifacts. Weak authorization or broad service-account permissions may allow unauthorized users or workloads to download or modify valuable assets.

### Multi-tenant AI storage exposure

High-performance AI storage platforms often host filesystems for multiple tenants or workloads on the same cluster. If client authentication is disabled, network reachability alone can be enough for a compromised host or malicious tenant to mount filesystems outside its workspace, exposing other tenants' datasets, model weights, checkpoints, and training outputs, while allowing silent modification of training data or checkpoints.

### Exposure to privileged infrastructure operators

When models or datasets run on third-party GPU infrastructure, provider operators or platform administrators may access model artifacts, host memory, node storage, logs, or deployment paths depending on the platform trust boundary.

### Detection blindness to model artifact movement

Traditional DLP and storage monitoring may not classify files such as .pt, .pth, .safetensors, .onnx, or .h5 as sensitive IP by default, allowing large transfers to blend into routine deployment, backup, or storage activity.

### Storage substrates left unencrypted or unaudited

AI storage platforms generally support encryption at rest, per-filesystem audit logging, and managed TLS on the storage management plane, but these are not always enabled or correctly configured. Where they aren't, exfiltration can leave no forensic trail, decommissioned media may remain readable, and the management API can be intercepted or impersonated by an attacker on the storage network.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Enforce authentication and tenant isolation at the storage layer:** Require client authentication for all filesystem mounts and storage protocols on AI storage platforms. Enforce tenant separation at the storage layer itself, not only in the scheduler or model registry. Restrict storage-network reachability only to approved hosts.

2. **Encrypt and audit the storage substrate:** Enable encryption at rest on clusters holding model artifacts and training data. Log high-impact events (mounts, deletes, permission changes, snapshots, administrative actions) to an independent system the storage administrator cannot reach or modify. Reserve detailed access logging for the most sensitive filesystems since full per-read logging is often impractical at scale.

3. **Clean temporary and cached copies between tenants:** Limit retention of model artifacts on training and serving nodes, clear node-local caches between tenants or jobs, and ensure snapshots and backups inherit the same access controls as primary repositories.

4. **Limit privileged infrastructure access to tenant artifacts:** Restrict operator, administrator, and service-account access to model artifacts, host memory, node storage, and deployment paths. Use short-lived credentials, scoped workload identities, and separation of duties for systems that can retrieve or deploy models.

### For customers

1. **Inventory model artifacts as high-value assets:** Track weights, checkpoints, adapters, datasets, tokenizer files, packaged files, snapshots, backups, and cached copies. Know where each exists, who can access it, and which systems can move or deploy it.

2. **Enforce least privilege on your model storage and registries:** Apply granular access controls to repositories, object stores, file systems, backup locations, and registries, with separate permissions for reading, exporting, approving, deleting, and deploying artifacts.

3. **Require approval and audit for high-impact actions:** Downloads, exports, permission changes, model promotions, backup restores, and cross-boundary transfers should generate tamper-resistant records and require approval where they could expose production or frontier models.

4. **Monitor for unusual artifact movement:** Alert on bulk downloads, unexpected copies, transfers to new destinations, and large movements of model file types (.pt, .pth, .safetensors, .onnx, .h5), which traditional DLP may not classify as sensitive by default.

5. **Ask how the provider protects the storage beneath your models:** When models run on third-party infrastructure, the storage layer isn't fully visible from inside your workload. Ask whether the provider requires client authentication, separates tenants at the storage layer, encrypts at rest, and audits storage access. Treat a provider that cannot answer as carrying residual risk.

## Attack Scenarios

### Exfiltration via Unsecured Backup

An attacker discovers model checkpoints in backup, snapshot, or mirror locations that don't enforce the same access controls as the primary model repository. Because many security programs focus on source code and traditional sensitive data, model artifacts in secondary storage may be poorly inventoried or insufficiently monitored. Large outbound transfers of model files can blend into normal backup or replication activity and may not be flagged unless model artifact movement is explicitly monitored.

### Compromised Deployment Service Account

An attacker compromises a service account used by a model deployment pipeline. The account is allowed to pull approved model files from a registry and copy them to serving infrastructure. The attacker uses the same trusted path to download model weights outside the approved environment. Because the activity resembles normal deployment traffic, defenders may not notice the theft unless model artifact movement is explicitly monitored.

### Cross-Tenant Model Theft Through Unauthenticated AI Storage

A customer of a neocloud GPU provider gets access to a compute node connected to a shared high-performance AI storage cluster (such as Weka, VAST Data, DDN, Lustre, BeeGFS, CephFS, or similar). The cluster hosts filesystems for multiple tenants, but filesystem client authentication is disabled or left permissive. From their node, the customer mounts another tenant's filesystem and copies model weights, checkpoints, datasets, and training outputs. Because access logging isn't enabled on that filesystem, the provider has limited record of which files were accessed or copied. If write access is also permissive, the same path can be used to tamper with training data or checkpoints. These platforms support client authentication, storage-layer tenant isolation, and audit logging, so the exposure concentrates in deployments where those controls are off or unconfigured.

### Model Cache Left on a GPU Serving Node

A serving node keeps local model file copies to reduce startup time. After a workload is removed or reassigned, cached files remain on disk or in a location readable by another process. A compromised workload or later tenant accesses the leftover cache and copies model artifacts that were never meant to be available outside the original serving environment.

## References

- **MITRE ATLAS.** MITRE. Adversarial threat landscape for AI systems. https://atlas.mitre.org/
- **NIST AI Risk Management Framework.** NIST. Framework for managing AI-related risks. https://www.nist.gov/itl/ai-risk-management-framework
- **Stealing Machine Learning Models via Prediction APIs.** Tramèr et al., USENIX Security 2016. Demonstrates model extraction through query access to prediction APIs. https://arxiv.org/pdf/1609.02943
- **Extracting Training Data from Large Language Models.** Carlini et al., USENIX Security 2021. Shows that LLMs memorize and can leak training data. https://www.usenix.org/conference/usenixsecurity21/presentation/carlini-extracting
- **Hermes Attack: Steal DNN Models with Lossless Inference Accuracy.** Zhu et al., USENIX Security 2021. GPU side-channel attack enabling model extraction. https://www.usenix.org/conference/usenixsecurity21/presentation/zhu
- **NVIDIA Trusted Computing Solutions.** NVIDIA. Confidential computing for GPU workloads. https://docs.nvidia.com/confidential-computing/index.html
