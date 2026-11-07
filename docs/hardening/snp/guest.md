# AMD SEV-SNP Guest Hardening

This guide covers the guest-side hardening decisions for AMD SEV-SNP confidential VMs. It focuses on the settings a guest owner must verify and the runtime checks needed to confirm the guest is launched in a hardened policy.

## 1. Objective

The guest should boot with the minimum trusted surface and a policy that blocks unsafe features and host-controlled behaviors.

The key principles are:

- keep the guest policy restrictive
- avoid enabling PMC virtualization
- Keep the guest kernel updated
- ensure the guest is built with SNP-aware firmware and kernel support
- validate the attestation report before the VM is accepted 


## 2. Verify the guest stack supports SNP

### 2.1 Check the guest firmware/kernel environment

The guest should use SNP-aware firmware and an SNP-aware Linux guest kernel. For upstream Linux, 6.11 is a baseline for SEV-SNP guest support; distributions may backport that support to earlier kernels. Treat the vendor-supported kernel release and verified SNP capabilities as authoritative, rather than relying on the version number alone. Optional features such as restricted injection may require additional, version-specific patches.

```bash
uname -a
cat /proc/cmdline
```

`uname -a` reports the running kernel release, build, and system architecture. `/proc/cmdline` reports the boot parameters actually passed to the running guest; it does not show the kernel version or prove that the kernel supports a feature.

If the guest is a cloud VM or custom image, verify the image includes the expected SNP support. Use the cloud or image-management update process to install a vendor-supported kernel, then verify the active kernel and its SNP capabilities.

## 3. Prevent PMC virtualization

PMC means performance monitoring counters. These are specialized hardware registers inside AMD CPUs (core PMCs, L3/uncore PMCs, Data Fabric PMCs, etc.) that count low-level processor events such as:

- Retired instructions
- Retired branches / taken branches
- Cache accesses/misses
- Division operations and cycles
- And hundreds of other micro-architectural events

AMD EPYC generations (notably the 8005 series and beyond) added PMC Virtualization / Guest PMC Event Filtering. This allows:

- Hardware filtering of which events a guest is allowed to program 
- Better isolation so the host cannot freely observe guest performance-counter activity.

In the [AMD SEV-SNP Firmware ABI Specification (Publication 56860)](https://www.amd.com/content/dam/amd/en/documents/developer/56860.pdf), `SEV_FEATURES[11]` is `VIRT_PERFMON` (virtualized performance monitoring). Use the ABI revision applicable to the platform when interpreting this and other feature bits.

### 3.1 Security requirement of PMC virtualization

Do not enable `VIRT_PERFMON` (`SEV_FEATURES[11]`) at launch.

This is the guest-side equivalent of a host policy decision and is essential for limiting performance-counter attacks.

If the guest can see or manipulate real hardware performance counters, a malicious hypervisor may exploit that channel.

### 3.2 Verification

Inspect the launch configuration and the attested `SEV_FEATURES` claim. Confirm that bit 11 (`VIRT_PERFMON`) is not set. This is a launch feature, not the separate `PAGE_SWAP_DISABLE` guest-policy bit described in section 6. A guest-side check alone cannot prove that the launch setting was applied.

## 4. Restricted injection 
### 4.1 Definition

Restricted injection limits the hypervisor to special #HV events instead of arbitrary interrupt injection.

### 4.2 Guest side requirements

For the guest to work safely with this feature:

- guest firmware must support #HV handling
- guest kernel must support #HV doorbell handling
- host must provide the matching support

Do not enable or rely on restricted injection unless you have verified the full stack is ready.

Restricted injection is configured by the launch stack, not enabled by the guest commands in this guide. Follow the applicable [AMD SEV-SNP documentation](https://www.amd.com/en/developer/sev.html), [Linux KVM AMD memory-encryption documentation](https://docs.kernel.org/virt/kvm/x86/amd-memory-encryption.html), and [QEMU AMD SEV documentation](https://www.qemu.org/docs/master/system/i386/amd-memory-encryption.html), along with the guest-firmware release documentation for the exact versions in use. Confirm that the complete host-kernel, QEMU, firmware, and guest-kernel stack supports the ABI-defined feature before enabling it; support may depend on vendor or downstream patches.

## 5. Validate attestation 

### 5.1 Check the report and enforce policy

Use Inspector to validate the report and enforce the required claims in Rego. In particular:

- require `ALIAS_CHECK_COMPLETE` to be true
- require `PAGE_SWAP_DISABLE` in the attested guest policy when required
- compare every required guest-policy field with the approved deployment values (guest-policy correctness)
- require prohibited launch features, such as `VIRT_PERFMON`, to be absent

Example attestation snippet

```text
Platform Info (39):
  SMT Enabled: true
  TSME Enabled: true
  ECC Enabled: true
  RAPL Disabled: false
  Ciphertext Hiding Enabled: false
  Alias Check Complete: true
```

Illustrative rules to add to the existing `Inspector` policy. These assume that `input.claims.attestations["snp"]` contains the corresponding normalized claims; adjust field names to the Inspector claims available in the deployment. They add failures to the shared `errors` set, which the core policy uses to decide `allow`.

```rego
errors contains "SNP alias check is incomplete" if {
    snp := input.claims.attestations["snp"]
    not snp.platform_info.alias_check_complete
}

errors contains "SNP page-swap restriction is not enabled" if {
    snp := input.claims.attestations["snp"]
    not snp.guest_policy.page_swap_disable
}

errors contains "SNP guest policy does not match an approved policy" if {
    snp := input.claims.attestations["snp"]
    not is_approved_snp_guest_policy(snp.guest_policy)
}

is_approved_snp_guest_policy(value) if {
    value in ref_values_by_type("SNP_GUEST_POLICY")
}

errors contains "SNP virtualized performance monitoring is enabled" if {
    snp := input.claims.attestations["snp"]
    snp.sev_features.virt_perfmon
}
```

Configure `SNP_GUEST_POLICY` reference values as the complete approved policy objects, guest policy is described in section 6. Include only the checks required for the deployment.

## 6. Harden SEV-SNP launch identity and policy

The guest owner or launch service for example Tower selects several security-relevant values at launch. Review these values as part of image release and guest provisioning; they cannot generally be changed after launch.

### 6.1 Guest policy

Set the required guest policy through Tower. Select only the policy options required by the workload and platform. Review, where supported:

- whether guest debugging is allowed (disable it for production)
- whether SMT is allowed
- whether a migration agent is allowed
- whether the guest is restricted to a single socket
- the minimum required ABI version
- whether ciphertext hiding is required
- whether RAPL must be disabled
- whether AES-256-XTS is required
- whether CXL devices are allowed

For deployments requiring page-swap restriction:

- set `PAGE_SWAP_DISABLE` in the guest policy

`PAGE_SWAP_DISABLE` is bit 25 in the guest policy. It disables page swapping as defined by the applicable SNP ABI; it is not a general-purpose statement that all page movement is blocked. A verifier checks the attested guest-policy bit against the deployment's required value.

Illustrative rule to add to the existing `Inspector` policy. Map the field name to the normalized claim provided by Inspector:

```rego
errors contains "SNP page-swap restriction is not enabled" if {
    snp := input.claims.attestations["snp"]
    not snp.guest_policy.page_swap_disable
}
```

To calculate the hardened policy value manually:

```bash
policy=$((0x30000 | (1 << 25)))
echo "policy = 0x$(printf '%x' $policy)"
```

Expected result:

```bash
0x20030000
```

In Tower if you select the desired policies, the policy values will be automatically created.


The exact available flags depend on the SEV-SNP ABI, platform generation, and launch tooling. Check the tool's documentation and confirm the resulting policy in the attestation report; do not assume that a requested option was applied.

### 6.2 Guest SVN and image identity

`GUEST_SVN`, `FAMILY_ID`, and `IMAGE_ID` are launch-time values supplied by the guest owner through the launch stack (for example, an ID block passed by the VMM). They are not normally chosen from inside the running guest. Whether Tower or another launcher sets them must be confirmed in that environment. In our current workflow, we do not set these values, so do not assume they identify an approved image until their values are observed in a real attestation report.

- **Guest SVN**: owner-assigned version claim for the guest image. To enforce anti-rollback, the verifier must maintain a minimum acceptable SVN and reject reports below it. The field by itself does not make a verifier reject an older image.
- **Family ID and Image ID**: owner-assigned identity labels that can help a verifier select or identify an image family/release when the launch stack populates and authenticates them.

These are separate report claims from `MEASUREMENT`. The measurement is the cryptographic digest of the guest launch contents; checking an Image ID does not avoid measurement verification and is not a substitute for it. An Image ID is useful as an additional, compact policy claim only when its source and binding are trusted. If these fields are used, record the expected claim values in the verifier policy and check them during attestation.

Illustrative rules to add to the existing `Inspector` policy. Configure the corresponding reference values for the deployment; store the minimum SVN reference value as a number:

```rego
is_approved_guest_svn(value) if {
    some minimum in ref_values_by_type("SNP_MIN_GUEST_SVN")
    value >= minimum
}

is_approved_snp_image_id(value) if {
    value in ref_values_by_type("SNP_IMAGE_ID")
}

errors contains "SNP guest SVN is below the approved minimum" if {
    snp := input.claims.attestations["snp"]
    not is_approved_guest_svn(snp.guest_svn)
}

errors contains "SNP image ID is not approved" if {
    snp := input.claims.attestations["snp"]
    not is_approved_snp_image_id(snp.image_id)
}
```

Apply the `IMAGE_ID` comparison only where the launch stack populates and authenticates that claim.

## 7. Bind runtime attestation to the Inspector challenge

For Inspector requests, Inspector generates a fresh nonce for each request and checks that the returned report's `REPORT_DATA` contains that request's expected nonce. This binds the report to the current challenge and helps prevent replay or substitution; it does not itself establish that the guest policy is secure. Verify the security claims separately by checking the attested guest policy and required report fields against the relying party's approved policy, as in section 5.

The report requester selects the VMPL in the report request. VMPL 0 is the highest privilege level; request the level appropriate to the component making the request, and have the verifier check the reported VMPL matches the intended component and trust policy.

## 8. Validate launch-time and current mitigation vectors

For attestation report versions that provide these fields (including supported report version 5 and later), inspect both vectors and interpret them using the matching AMD ABI/report-version definition:

- **LAUNCH_MIT_VECTOR**: mitigation state recorded for guest launch.
- **CURRENT_MIT_VECTOR**: mitigation state recorded when the attestation report is generated.

The vectors are bitmaps, not universal version numbers. Their numeric values and bit meanings depend on the report version and platform support, so there is no single expected hex value that applies to every SNP environment. For example, if policy requires mitigation `M` at launch and at report time, the policy should require the ABI-defined `M` bit to indicate the required state in both fields. If launch and current state differ, evaluate each field according to its own requirement; do not treat one as a replacement for the other.

### 8.1 Generate verifier policy for the deployment environment

Generate or configure the attestation policy from the actual environment rather than copying one policy across all platforms. Account for:

- processor/platform generation and supported SNP report versions
- firmware/ABI version and which claims the attestation pipeline exposes
- expected Guest Policy, measurement, and any populated `GUEST_SVN`, `FAMILY_ID`, and `IMAGE_ID` claims
- mitigation-vector bit meanings and required launch-time versus current states

The policy should check the claims required for that environment, reject missing claims when they are mandatory, and compare identity/SVN values against the approved image policy. Keep measurement verification enabled; identity labels are additional policy inputs, not a shortcut around it. Validate the policy against real reports from each supported environment before relying on it in production.

Illustrative rules to add to the existing `Inspector` policy. The example assumes Inspector (or an upstream normalization step) has mapped the ABI-defined bits to booleans; use the actual field names exposed by the deployment:

```rego
errors contains "required SNP launch mitigation is not enabled" if {
    snp := input.claims.attestations["snp"]
    not snp.mitigations.launch_required
}

errors contains "required current SNP mitigation is not enabled" if {
    snp := input.claims.attestations["snp"]
    not snp.mitigations.current_required
}

is_approved_snp_measurement(value) if {
    value in ref_values_by_type("SNP_MEASUREMENT")
}

errors contains "SNP measurement is not approved" if {
    snp := input.claims.attestations["snp"]
    not is_approved_snp_measurement(snp.measurement)
}
```

The mitigation fields above are illustrative normalized claims, not universal bit numbers. Obtain the actual vector values from the target launch stack and map them using the matching AMD ABI. Add identity checks only for claims populated and authenticated in the environment.

## 9. Final guest hardening checklist

- [ ] Guest firmware is SNP-aware and the guest kernel is current, with SNP support and #HV handling where required
- [ ] `VIRT_PERFMON` (`SEV_FEATURES[11]`) is not enabled for PMC virtualization
- [ ] PAGE_SWAP_DISABLE is enabled in the launch policy when supported and required
- [ ] Restricted Injection is used only when the host kernel, QEMU, guest firmware, and guest kernel all support it
- [ ] Guest Policy is reviewed and set by the authorized guest owner or Tower
- [ ] Debugging is disabled in production unless explicitly approved
- [ ] Guest SVN meets the anti-rollback policy; Family ID and Image ID match the approved image
- [ ] Verifier policy is generated for the platform/report environment and checks required identity, SVN, policy, and mitigation claims
- [ ] Attestation report confirms the expected guest policy and required claims
- [ ] `ALIAS_CHECK_COMPLETE` is true in the attestation report
- [ ] `REPORT_DATA` is bound to a fresh Inspector nonce and validated by Inspector
- [ ] The report is requested at the intended VMPL
- [ ] LAUNCH_MIT_VECTOR and CURRENT_MIT_VECTOR are interpreted using the matching AMD ABI and checked against environment-specific requirements