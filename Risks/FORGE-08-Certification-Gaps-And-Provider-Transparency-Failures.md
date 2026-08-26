# FORGE-08: Certification Gaps & Provider Transparency Failures

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **High** | Medium | High | Medium |

## Definition

These failures occur when customers depend on provider certifications, attestations, or security claims that don't clearly apply to the AI infrastructure actually running their workloads. The risk grows when providers fail to disclose audited scope, upstream operators, reselling arrangements, subservice organizations, regional differences, or which controls apply to each environment.

## Description

Many GPU cloud providers present security assurance through certifications like SOC 2 or ISO 27001, but those attestations don't automatically prove the full AI infrastructure stack was assessed. Audit scope may cover the provider's customer portal, API, and internal software processes while excluding physical GPU hosts, backend fabric, BMC environment, firmware lifecycle, node reassignment procedures, or upstream infrastructure operators.

This gap is harder to detect when provider transparency is limited. Some providers resell or aggregate capacity from other operators, lease infrastructure from third parties, or rely on subservice organizations not obvious to the customer. Regional differences can matter if controls, operators, or audited environments vary by location. HPC security guidance such as [NIST SP 800-223](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-223.pdf) and [NIST SP 800-234](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-234.pdf) can help customers ask better questions about specialized hardware, high-speed networks, architecture, and security controls. But these are guidance documents, not provider-specific assurance.

They don't prove a given provider implemented the controls or that audited scope covers infrastructure actually running the workload. The result is a false sense of assurance. Customers may believe a certified provider independently validated the security of the full AI environment when the attested scope is partial or unclear. This is especially important in fast-growing GPU cloud environments where security documentation, control standardization, and infrastructure transparency may lag behind deployment.

## Impact and Failure Modes

### Certification scope excludes AI infrastructure

A provider holds SOC 2 or ISO 27001 but the assessed scope doesn't include GPU hosts, network fabric, BMC controls, firmware integrity, or node lifecycle operations.

### No penetration testing of the infrastructure

Some providers, including both emerging neoclouds and established vendors, don't conduct regular penetration testing of their AI infrastructure environments, meaning fundamental security weaknesses may go undiscovered until exploited.

### Undisclosed upstream operators

A customer contracts with one provider but the workload runs on infrastructure operated by another entity whose controls, patching cadence, or incident response processes aren't clearly disclosed.

### Inconsistent assurance across aggregated capacity

A provider combines capacity from multiple upstream operators with different security controls but presents a single trust posture to customers.

### Regional or subservice gaps

Security controls and attestations differ by geography, vendor, or subservice organization, but customers aren't clearly told which environments fall outside the assurance boundary.

### False confidence from narrow attestations

Customers rely on self-attestation, SOC 2 Type I, or marketing claims as though they prove ongoing operational effectiveness across the full AI stack.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Make attestation scope explicit:** State clearly which parts of the stack a certification actually covers so customers can't mistake a portal-and-API audit for full-stack assurance.

2. **Disclose infrastructure sourcing:** Disclose whether capacity is owned, leased, brokered, or resold, and identify the upstream operators or subservice organizations involved, including regional differences in controls.

3. **Test the infrastructure, not just the software:** Conduct regular penetration testing and security review of the GPU infrastructure environment, not only the customer portal and API.

### For customers evaluating a provider

1. **Review attestation scope carefully:** Verify whether certifications and audit reports include GPU infrastructure, backend fabric, management-plane controls, firmware lifecycle, and tenant reassignment procedures-not just the API or software layer.

2. **Require transparency about infrastructure sourcing:** Contracts and security documentation should disclose whether capacity is owned, leased, brokered, or resold, identifying upstream operators or subservice organizations involved.

3. **Prefer stronger independent assurance where needed:** For production or regulated workloads, require meaningful third-party assurance such as SOC 2 Type II or ISO 27001 with operationally relevant scope rather than relying only on self-attestation or point-in-time reviews.

4. **Verify the AI-specific controls, not just the certificate:** Use a targeted questionnaire or control review of the AI infrastructure itself, not just the general certification. The CSA AI-CAIQ is a useful starting point.

5. **Verify the machine is operated by the provider you expect:** Confirm that assigned infrastructure traces back to the provider you contracted with, not an undisclosed upstream operator. Public ownership records for your node's external IP, reverse DNS, and TLS certificates on management or API endpoints can indicate who actually operates it. A mismatch isn't proof of a problem since infrastructure is often registered to a parent or upstream operator, but it's worth reconciling against what the provider disclosed.

## Attack Scenarios

### Certification Scope Mismatch

An enterprise selects a GPU cloud provider based on SOC 2 Type II and marketing claims of secure AI infrastructure. After a security incident, the customer discovers the audited scope covered only the provider's orchestration platform and customer portal. The physical GPU systems, BMC environment, backend fabric, and storage layer were operated by a third party outside the assessed boundary, leaving the customer without evidence that the actual workload environment met expected controls.

### Hidden Upstream Provider

A provider aggregates capacity from multiple upstream operators but doesn't clearly disclose that arrangement. A customer assumes the advertised security posture applies uniformly across all regions and clusters. After an incident, investigation shows the affected workload ran on infrastructure managed by a different operator with weaker lifecycle controls and different patching practices, creating a gap between promised assurance and the actual environment.

## References

- **NIST SP 800-234.** High-Performance Computing (HPC) Security Overlay. https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-234.pdf
- **NIST SP 800-223.** High-Performance Computing Security: Architecture, Threat Analysis, and Security Posture. https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-223.pdf
- **AICPA SOC 2 / Trust Services Criteria.** Audit standard for service organization controls. https://www.aicpa-cima.com/topic/audit-assurance/audit-and-assurance-greater-than-soc-2
- **ISO/IEC 27001.** Information security management system standard. https://www.iso.org/standard/27001
- **Cloud Controls Matrix (CCM).** Cloud Security Alliance. Cloud-specific security control framework. https://cloudsecurityalliance.org/research/cloud-controls-matrix
- **CAIQ / STAR Level 1 Security Questionnaire.** Cloud Security Alliance. Standardized cloud security self-assessment. https://cloudsecurityalliance.org/artifacts/star-level-1-security-questionnaire-caiq-v4-1
- **STAR Registry.** Cloud Security Alliance. Public registry of cloud provider security postures. https://cloudsecurityalliance.org/star/registry
- **NIST SP 800-161 Rev. 1.** Cybersecurity Supply Chain Risk Management. Supply chain risk management practices for systems and organizations. https://csrc.nist.gov/pubs/sp/800/161/r1/upd1/final
- **Shared Assessments, SIG Questionnaire.** Standardized third-party risk assessment. https://sharedassessments.org/sig/
- **AI Consensus Assessments Initiative Questionnaire (AI-CAIQ).** Cloud Security Alliance. AI-specific cloud security assessment. https://cloudsecurityalliance.org/artifacts/ai-consensus-assessments-initiative-questionnaire-ai-caiq
