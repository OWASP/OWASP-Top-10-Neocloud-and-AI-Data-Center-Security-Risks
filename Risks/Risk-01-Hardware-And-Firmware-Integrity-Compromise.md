# Risk 01: Firmware & Hardware Integrity Compromise

| Risk Level | Likelihood | Impact | Detection Difficulty |
|---|---|---|---|
| **Critical** | Medium | Severe | High |

## Definition

Firmware & Hardware Integrity Compromise refers to unauthorized or attacker-controlled modification of firmware across system components such as UEFI, BMC, NICs, GPUs, and storage controllers. Because this code runs before or below the OS, compromise can persist across OS reinstalls and disk wipes, evade many OS-level security tools, and survive tenant reassignment in shared bare-metal infrastructure.

## Description

Firmware and hardware integrity matters because these components sit beneath the operating system and outside the visibility of most normal security tools. A compromised firmware component can affect how a system boots, how devices behave, and what the OS itself is allowed to see.

This makes trust difficult to establish after the fact. Providers need a way to prove that a platform booted into an expected, untampered state, and that critical components have not drifted from approved versions. Controls such as Secure Boot, Measured Boot, signed firmware enforcement, and cryptographic attestation help provide that assurance, but only when they are present, enabled, correctly configured, monitored, and independently verified.

The risk is especially important in AI infrastructure because these systems are valuable, shared, and often reassigned between tenants. In bare-metal environments, a malicious tenant with sufficient local access may attempt to abuse firmware update paths or device control interfaces before returning the node. If signed-firmware enforcement, attestation, and reassignment checks are absent, disabled, or unverified, the next tenant may inherit a compromised machine.

The BMC is another important path because it can control power, remote console access, virtual media, and firmware update workflows independently of the host OS. If BMC access is exposed through weak credentials, poor segmentation, or vulnerable management interfaces, an attacker may be able to modify firmware or establish persistence below the operating system.

This threat is not theoretical. [BlackLotus](https://www.welivesecurity.com/2023/03/01/blacklotus-uefi-bootkit-myth-confirmed/) showed that UEFI bootkits can execute before the OS loads, and [CosmicStrand](https://securelist.com/cosmicstrand-uefi-firmware-rootkit/106973/) showed that motherboard firmware implants can persist across OS reinstalls and raise concerns around resale or refurbishment channels.

> **See also:** [Risk 04: Insecure Out-of-Band Management Plane](Risk-04-Insecure-Out-Of-Band-Management-Plane.md) for management plane attacks that can lead to firmware compromise.

## Impact and Failure Modes

### Firmware cannot be verified

Attestation and Measured Boot are the features that let a provider prove a machine's firmware hasn't been modified. Where they aren't enabled, providers have no cryptographic proof that the BIOS, BMC, NIC, GPU, or storage firmware can be trusted. The firmware may well be fine, but there is no way to tell. On hardware that supports these features, the gap is usually operational, the feature is off, unconfigured, or unmonitored, rather than a hardware limitation.

### Firmware update paths are exposed or weakly validated

Firmware is updated through vendor flashing tools, BMC update functions, and maintenance workflows. If these paths are reachable without proper authorization, or do not enforce signature checks, an attacker may install firmware that survives normal rebuilds and is difficult to remove.

### Cross-tenant persistence in bare metal

In bare-metal reuse, a malicious tenant with sufficient local or device-level access may abuse firmware or privileged control interfaces before returning a node. If the provider only wipes the OS and does not reflash, or verify firmware state, the next tenant may inherit a compromised machine.

### Attestation signals can be missing or misleading

BMC platforms frequently ship with integrity features turned off. HPE iLO's Global Component Integrity, for example, is disabled by default and requires a full power cycle to enable. Once on, the green checkmark next to a device only confirms the device is genuine, not that its firmware is untampered. Operators who treat the checkmark as proof of trusted firmware develop a false sense of assurance.

### Attestation coverage is uneven across mixed fleets

Hardware-backed attestation depends on platform and component support, such as an external Root of Trust (eRoT) that must be designed into the system. NVIDIA data center products have broad eRoT coverage, but support across CPUs, NICs, storage controllers, BMCs, and older platforms may vary significantly. Providers cannot assume every component in a mixed fleet can produce useful attestation evidence.

### Attestation evidence needs reference values to be meaningful

An attestation measurement only proves what value a device reported. To determine whether that value is trusted, the verifier needs vendor-published reference values for the specific device and firmware version. CoRIM (Concise Reference Integrity Manifest) is an emerging IETF specification for publishing this evidence, but adoption remains limited. NVIDIA is currently the main hardware vendor publishing CoRIM data for its devices.

## Prevention and Mitigation Strategies

### For neocloud providers and data center operators

1. **Establish a hardware root of trust:** Use a hardware or silicon root of trust, together with a TPM or equivalent, to anchor platform identity, securely record boot measurements, and support attestation. Where supported, use silicon root-of-trust implementations such as Caliptra for chip identity, measured boot, and attestation.

2. **Enforce trusted boot and signed firmware:** Enable Secure Boot and Measured Boot, require signed firmware, and prevent unauthorized firmware or boot components from running.

3. **Verify firmware before production use, and plan for partial coverage:** Where supported, use device attestation protocols such as SPDM to validate firmware measurements against vendor-published reference values. Confirm that attestation features are enabled and that reported status reflects firmware integrity, not only device authenticity. For components without attestation coverage, use compensating controls such as re-flashing to known-good images when systems are received or reassigned.

4. **Monitor firmware state across the fleet:** Maintain a continuously updated inventory of firmware versions for components such as BIOS, BMC, GPU, NIC, and storage devices. Alert on and investigate nodes that deviate from approved firmware baselines, attestation results that change between boots without a corresponding maintenance event, and failed attestation. Without fleet-level visibility, a single tampered node can go undetected.

5. **Gate sensitive workload placement on attestation:** Integrate remote attestation with schedulers so sensitive workloads run only on verified nodes, and automatically quarantine systems that fail verification.

6. **Secure firmware lifecycle operations:** Restrict access to firmware tooling and management interfaces (for example, through an approved gateway or bastion host on a network path separate from user access, as described in NIST SP 800-234), require approved update workflows, and repeat the verification steps and applicable compensating controls described in item 3 during intake, servicing, and tenant transitions.

7. **Maintain secure recovery procedures:** Keep known-good firmware images, documented recovery workflows, and replacement criteria so compromised platforms can be restored or removed from service with trust re-established.

### For customers evaluating a provider

How much you can check yourself depends on the offering. On bare-metal or dedicated nodes you can verify the first two directly.

1. **Check that trusted boot is actually on (bare metal offering):** From the OS, confirm Secure Boot is enabled and a TPM is present and recording boot measurements. This is a basic check that the protections are turned on, not a full attestation of firmware integrity, but it is a useful signal that the foundations are in place.

2. **Record firmware versions and watch for drift (bare metal offering):** Capture component firmware versions at the start of a lease and re-check them. Unexpected changes during a lease are worth investigating.

3. **Ask how firmware is handled between tenants:** Ask whether firmware is re-flashed or re-attested before a node is reassigned. A prior tenant's changes are invisible to you, so on any reused machine this is the control that protects you.

## Attack Scenarios

### Supply Chain Interdiction

An adversary tampers with a GPU server before deployment and implants modified device firmware that provides covert persistence or data exfiltration. On a platform without hardware-verified boot, or where that verification is disabled or unchecked, the server is delivered, provisioned, and put into service without deep attestation, so the compromise survives into production and remains invisible for an extended period. On platforms that enforce a verified boot chain and are actually configured to do so, this implant path is far harder to land and more likely to be detected.

### Remote Firmware Compromise via BMC

A customer renting a bare-metal GPU server is accidentally able to reach the BMC/IPMI interface because of faulty network segmentation. From that privileged position, they abuse firmware update mechanisms or platform controls to modify host or device firmware, creating persistence below the OS that survives reinstallation and evades standard monitoring.

### Insider Firmware Tampering

A technician, contractor, or other trusted operator with maintenance access re-flashes a component, weakens integrity controls, or installs a modified image during servicing. The node returns to operation appearing healthy, but it now carries long-lived attacker-controlled logic below the operating system.

### Cross-Tenant Firmware Persistence

A malicious tenant with root access to a bare-metal GPU node modifies component firmware before their lease ends. The provider wipes disks and reprovisions the server, but does not re-flash or re-attest firmware, so the next tenant may receive a compromised host. Whether the tampering succeeds in the first place depends on the platform: signed-firmware enforcement and a verified boot chain raise the bar considerably, while platforms without those controls, or with them disabled, remain exposed.

## References

- **DMTF SPDM Specification (DSP0274).** Security Protocol and Data Model standard for firmware attestation and device authentication. https://www.dmtf.org/dsp/DSP0274
- **NIST SP 800-193.** Platform Firmware Resiliency Guidelines (2018). Guidance on protecting platform firmware, detecting unauthorized changes, and recovering securely. https://csrc.nist.gov/publications/detail/sp/800-193/final
- **NIST SP 800-234.** High-Performance Computing (HPC) Security Overlay (2026). Tailors SP 800-53 moderate-baseline controls for HPC systems, including SI-7 (software, firmware, and information integrity) and AC-17(4) (approved, separated access paths to the management zone). https://csrc.nist.gov/pubs/sp/800/234/final
- **TCG TPM 2.0 Library Specification.** Trusted Computing Group specifications for hardware root of trust. https://trustedcomputinggroup.org/resources/documents/tpm-library-specification
- **BlackLotus UEFI Bootkit (CVE-2022-21894).** ESET, 2023. UEFI bootkit that bypassed Secure Boot on affected systems, demonstrating firmware-level persistence below the OS. https://www.welivesecurity.com/2023/03/01/blacklotus-uefi-bootkit-myth-confirmed/
- **IETF RATS CoRIM (Internet-Draft).** draft-ietf-rats-corim: format for publishing expected attestation values used to evaluate SPDM and similar evidence. https://datatracker.ietf.org/doc/draft-ietf-rats-corim/
- **RFC 9334, RATS Architecture.** IETF. Architecture for remote attestation. https://datatracker.ietf.org/doc/rfc9334/
- **HPE iLO 6 User Guide, Global Component Integrity.** HPE. iLO option that authenticates server components using SPDM. https://support.hpe.com/hpesc/public/docDisplay?docId=sd00002007en_us&page=GUID-EBC183E0-5805-4BEE-AFF8-02585DC32C87.html&docLocale=en_US
- **Caliptra.** Open-source silicon root of trust block for datacenter SoCs, providing identity, measured boot, and attestation; hosted by CHIPS Alliance. https://github.com/chipsalliance/Caliptra