# FORGE-01: Firmware & Hardware Integrity Compromise

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

> **See also:** [FORGE-04: Insecure Out-of-Band Management Plane](FORGE-04-Insecure-Out-Of-Band-Management-Plane.md) for management plane attacks that can lead to firmware compromise.

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

Hardware-backed attestation depends on platform and component support, such as an external Root of Trust (eRoT) that must be designed into the system. NVIDIA datacenter products have broad eRoT coverage, but support across CPUs, NICs, storage controllers, BMCs, and older platforms may vary significantly. Providers cannot assume every component in a mixed fleet can produce useful attestation evidence.

### Attestation evidence needs reference values to be meaningful

An attestation measurement only proves what value a device reported. To determine whether that value is trusted, the verifier needs vendor-published reference values for the specific device and firmware version. CoRIM (Concise Reference Integrity Manifest) is the emerging IETF standard for publishing this evidence, but adoption remains limited. NVIDIA is currently the main hardware vendor publishing CoRIM data for its devices.

## Prevention and Mitigation Strategies

### For providers and datacenter operators

1. **Establish a hardware root of trust:** Use a hardware or silicon root of trust, together with a TPM or equivalent, to anchor platform identity, record boot measurements, and support attestation. Where supported, use silicon root-of-trust efforts such as Caliptra for chip identity, measured boot, and attestation.

2. **Enforce trusted boot and signed firmware:** Enable Secure Boot and Measured Boot, require signed firmware, and prevent unauthorized firmware or boot components from running.

3. **Verify firmware before production use, and plan for partial coverage:** Where supported, use device attestation protocols such as SPDM to validate firmware measurements against vendor-published reference values. Confirm that attestation features are enabled and that reported status reflects firmware integrity, not only device authenticity. For components without attestation coverage, use compensating controls such as re-flashing to known-good images when systems are received or reassigned.

4. **Monitor firmware state across the fleet:** Maintain a continuously updated inventory of firmware versions for components such as BIOS, BMC, GPU, NIC, and storage devices. Treat unexpected firmware changes, baseline drift, or failed or changed attestation results as security events requiring investigation. Alert when nodes deviate from approved firmware baselines, when attestation results change between boots without a corresponding maintenance event, or when attestation fails entirely. Without fleet-level visibility, a single tampered node can remain in production indefinitely.

5. **Gate sensitive workload placement on attestation:** Integrate remote attestation with schedulers so sensitive workloads run only on verified nodes, and automatically quarantine systems that fail verification.

6. **Secure firmware lifecycle operations:** Restrict access to firmware tooling and management interfaces, require approved update workflows, and re-attest systems during intake, servicing, and tenant transitions.

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

- **DMTF SPDM Specification v1.4.0.** Security Protocol and Data Model standard for firmware attestation and device authentication. https://www.dmtf.org/dsp/DSP0274
- **NIST SP 800-155 (Draft).** BIOS Integrity Measurement Guidelines. https://csrc.nist.gov/files/pubs/sp/800/155/ipd/docs/draft-sp800-155_dec2011.pdf
- **TCG TPM Library Specification.** Trusted Computing Group specifications for hardware root of trust. https://trustedcomputinggroup.org/resource/tpm-library-specification/
- **BlackLotus UEFI Bootkit (CVE-2022-21894).** ESET, 2023. UEFI bootkit that bypassed Secure Boot on affected systems, demonstrating firmware-level persistence below the OS. https://www.welivesecurity.com/2023/03/01/blacklotus-uefi-bootkit-myth-confirmed/
- **CosmicStrand UEFI Firmware Rootkit.** Kaspersky, 2022. Compromised motherboard firmware persisting across OS reinstallation, with concerns about infection through hardware resale channels. https://securelist.com/cosmicstrand-uefi-firmware-rootkit/106973/
- **iLOBleed, BMC Firmware Rootkit.** Amnpardaz, 2021. First BMC rootkit discovered in the wild. https://threats.amnpardaz.com/en/2021/12/28/implant-arm-ilobleed-a/
- **CVE-2019-6260, ASPEED BMC "Pantsdown" (CVSS 9.8).** Critical BMC vulnerability enabling host firmware access through unchecked memory mappings. https://nvd.nist.gov/vuln/detail/CVE-2019-6260
- **NSA ANT Catalog.** Electronic Frontier Foundation, 2013. Public archive of leaked hardware and firmware implant capabilities, illustrating supply-chain and pre-boot compromise concerns. https://www.eff.org/files/2014/01/06/20131230-appelbaum-nsa_ant_catalog.pdf
- **IETF RATS CoRIM (Concise Reference Integrity Manifests).** IETF Remote ATtestation procedureS Working Group. Standard for vendor publication of expected attestation values used to evaluate SPDM and similar evidence. https://datatracker.ietf.org/wg/rats/about/
- **HPE iLO 6 User Guide, Global Component Integrity.** HPE. iLO option that authenticates server components using SPDM. https://support.hpe.com/hpesc/public/docDisplay?docId=sd00002007en_us&page=GUID-EBC183E0-5805-4BEE-AFF8-02585DC32C87.html&docLocale=en_US
- **Open Compute Project Security Project.** OCP resources for platform security, including secure boot, attestation, firmware signing, device identity, and secure update practices. https://www.opencompute.org/wiki/Security
