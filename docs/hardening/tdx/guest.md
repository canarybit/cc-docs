# Intel TDX Guest Hardening

This guide covers the guest-side controls for Intel TDX confidential VMs. It explains the kernel parameters, filter settings, and attestation checks required for a hardened TD.


## 1. Objective

The TD should run with the smallest required device, I/O, and ACPI surface. It must avoid trusting host-controlled time and entropy sources and must be verifiable through attestation.

## 2. Required production kernel parameters

These are guest Linux kernel command-line parameters, not commands to run in a shell. Configure them in the guest image's bootloader (for example, its GRUB configuration) or have the VMM supply the guest command line at launch. The exact mechanism depends on the image and platform; coordinate with the host operator when the VMM controls the command line. After reboot, `/proc/cmdline` shows the parameters active in the running guest.

Use the hardened guest configuration:

```text
mce=off
oops=panic
pci=noearly
pci=nommconf
no-kvmclock
random.trust_cpu=y
```

Apply them to the guest kernel command line and reboot.

```bash
cat /proc/cmdline
```

## 3. Keep the device filter enabled

The TDX guest should use the default allow-list model. The guest kernel applies the filter; the image owner configures guest kernel parameters and the VMM may supply or override the boot command line. Coordinate both sides and verify the effective setting in the running guest.

### 3.1 Recommended practice

Only allow the minimal set of drivers required by the workload, such as:

- virtio_net
- virtio_console
- virtio_blk
- 9pnet_virtio

### 3.2 Do not disable the filter in production

```text
tdx_disable_filter
```

`tdx_disable_filter` is a guest kernel command-line parameter, not a shell command. Do not include it in the guest bootloader configuration or allow the VMM to pass it at launch. Check `/proc/cmdline` in the running guest to confirm it is absent.

### 3.3 Extra device allowances

If needed, use explicit allow-list entries:

```text
authorize_allow_devs=pci:<ven_id:dev_id>
```

Use this sparingly and only for approved devices.

## 4. Restrict Port I/O and PCI access

The guest kernel enforces these restrictions; the host/VMM controls which virtual devices and launch parameters are presented. Keep the two configurations consistent and verify the effective guest policy after boot.

### 4.1 Port I/O

Only a minimal set of I/O ports should be accessible. Do not expose extra ports unless absolutely required.

### 4.2 PCI config space

The PCI restrictions listed in section 2 reduce unsafe probing and limit access to untrusted or unnecessary config space.

## 5. Restrict ACPI tables

Configure any guest kernel command-line options through the guest bootloader or the VMM-provided command line described in section 2.

Use the minimal accepted ACPI set rather than broad expansion.

Do not use broad table exceptions in production.

```text
tdx_allow_acpi=
```

This should be used only for debug or tightly controlled exceptions.

## 6. Use secure randomness and secure time

### 6.1 Secure random source

The `random.trust_cpu=y` parameter from section 2 selects the architectural DRNG and avoids weak entropy fallback.

### 6.2 Time source

The `no-kvmclock` parameter from section 2 prevents reliance on the host-controlled KVM clock and leaves the TD using the platform's supported TSC time source.

## 7. Keep the runtime restricted and consistent

These are guest-runtime requirements. The host/VMM must not configure guest launch or paravirtual interfaces in a way that contradicts the approved guest profile.

### 7.1 Use the default hardened interrupt model

The guest should rely on virtualized x2APIC and the TDX module’s interrupt mediation. Legacy xAPIC MMIO and unsafe host-injected interrupt patterns are not appropriate for production TDs.

### 7.2 Use the minimal hypercall/PV set

Only keep the required KVM or PV features; do not turn on unnecessary host-facing interfaces.

## 8. Validate runtime and attestation state

Check the running cmdline:

```bash
cat /proc/cmdline
```

Compare the active command line with the hardened parameters in section 2. Also validate attestation output for:

- device filter is active
- ACPI list is restricted
- kernel cmdline matches the hardened profile
- no broad or unsafe device exceptions are in effect

Inspector policy should validate only claims that its configured verifier actually exposes. Confirm the normalized claim names and enforcement behavior against the deployment's Inspector configuration; do not assume that every item in this list is checked automatically.


## 9. Mitigate StumbleStepping and other side channels

Updating the TDX Module does not eliminate every fine-grained side channel. Intel treats residual leakage such as StumbleStepping as a software mitigation concern, similar to other microarchitectural side channels.

The constant-time software guidance in this section also applies to SEV-SNP workloads; see the [SNP guest hardening guide](../snp/guest.md).

### 9.1 Keep cryptographic software current

Update the guest OS, cryptographic libraries, and applications using the distribution's supported update mechanism. 

- OpenSSL 3.3.1 or newer
- wolfSSL 5.6.6 or newer

Check installed versions where applicable:

```bash
openssl version
pkg-config --modversion wolfssl 2>/dev/null || true
```


### 9.2 Use constant-time cryptographic implementations

Treat the hypervisor as able to observe fine-grained memory-access patterns, potentially at cache-line granularity. For code that handles secrets:

- avoid secret-dependent branches, memory addresses, and execution time
- use vetted cryptographic libraries and APIs with documented constant-time behavior instead of writing cryptographic primitives
- avoid early-exit comparisons for passwords, MACs, and padding; use constant-time comparison routines supplied by the library
- use constant-time modular exponentiation and secret-independent table selection for RSA and Diffie-Hellman operations
- avoid secret-indexed lookup tables, such as conventional AES S-box tables; use vetted constant-time implementations

Compiler transformations and CPU behavior can invalidate home-grown constant-time techniques. Review the generated implementation and use security-reviewed libraries rather than assuming that a source-level construct such as a conditional move guarantees constant-time execution.

Follow Intel's current [Trust Domain Security Guidance for Developers](https://www.intel.com/content/www/us/en/developer/articles/technical/software-security-guidance/best-practices/trusted-domain-security-guidance-for-developers.html) and the applicable software side-channel guidance.


Note: For now in cbclient we use RSA [rust crypto library](https://github.com/RustCrypto/RSA) which is non-constant crypto algorithm and we should look for future patches that fix this issue.

## 10. Verify Intel TDX TCB recovery status

TDX security depends on a Trusted Computing Base (TCB), including platform firmware, microcode, the TDX Module, and related components. After Intel releases a security fix, the affected component's Security Version Number (SVN) and Intel's attestation collateral may be updated through a TCB Recovery (TCB-R).

Installing an update is necessary but does not by itself prove to a relying party that the platform is at an acceptable TCB level. The quote must be checked against current Intel Provisioning Certification Service (PCS) collateral and an explicit trust policy.

### 10.1 Guest or workload owner actions

Before releasing secrets or trusting a TD:

1. Request a fresh quote from the TD using the deployment's attestation workflow.
2. Verify the quote and its TCB status against current Intel PCS collateral using an approved verifier, such as Inspector, Intel DCAP, Intel Trust Authority, or Confidential Containers Trustee.
3. Enforce a documented policy over the verified result: require the latest TCB, allow a defined grace period, or pin an approved intermediate TCB evaluation data number. Do not infer acceptance merely from successful quote verification.
4. Decide whether a TD created before a platform update may continue running, or must be restarted so it is created under the updated TCB.

Note: in our platform the verification is done by Inspector. 

The guest cannot change platform TCB values such as CPUSVN or TEE_TCB_SVN. A guest can choose to restart itself, but only the platform and TDX Module determine the platform TCB reflected by attestation.

Illustrative rules to add to the existing `Inspector` policy. These assume Inspector exposes the verified TDX status under `input.claims.attestations["tdx"].tcb_status`; adjust the path to the normalized claims in your deployment and configure accepted statuses as reference values.

```rego
is_accepted_tdx_tcb_status(value) if {
    value in ref_values_by_type("TDX_ACCEPTED_TCB_STATUS")
}

errors contains "TDX TCB status is not accepted" if {
    tdx := input.claims.attestations["tdx"]
    not is_accepted_tdx_tcb_status(tdx.tcb_status)
}
```

The accepted status set should reflect the organization's choice between strict latest-TCB enforcement and an explicitly bounded grace period. Inspector's quote signature verification, collateral, and freshness validation must also succeed; this policy check selects which verified TCB statuses the relying party accepts.

### 10.2 PCS collateral selection

Intel PCS supports collateral selection parameters for TCB information. Inspector should retrieve the collateral needed to evaluate both the strict/latest and grace-period views where supported, then expose the verified result for the Rego policy to accept or reject. The exact TDX certification API and FMSPC come from the platform's quote/collateral:

- `update=early`: request the latest available TCB information; use for strict, prompt enforcement of TCB-R updates.
- `update=standard`: request the standard collateral level, which may include a grace period. Do not assume it means the latest TCB.
- `tcbEvaluationDataNumber=<number>`: request a specific evaluation data number when supported by the API. It cannot be combined with `update`.

The following illustrates the Intel PCS `update=early` query form from the SGX certification API; a TDX verifier must use the corresponding TDX certification endpoint:

```text
https://api.trustedservices.intel.com/sgx/certification/v4/tcb?fmspc=<FMSPC>&update=early
```

Do not treat a collateral response or URL alone as quote verification. Use a maintained verifier that validates the quote chain, signatures, TCB status, and current collateral. If Inspector fetches more than one collateral view, the policy must explicitly identify which verified status values it accepts.

Deployment action: configure Inspector to retrieve the collateral views needed for the selected policy (including both `early` and `standard` where supported) and expose their verified TCB results to Rego. Do not silently rely on the PCS default of `standard` when the policy requires strict latest-TCB enforcement.

### 10.3 Platform owner and organization actions

- monitor Intel Platform Updates and TCB Recovery Attestation information for affected processors and required actions
- deploy applicable BIOS, microcode, TDX Module, and other platform updates
- re-register updated platforms with Intel where required so current PCK certificates and collateral can be issued
- support runtime updates where the platform supports them; otherwise plan a warm or cold reset as required
- publish the current TCB level, update cadence, and any grace-period policy to tenants
- configure verification services to use current collateral and enforce the organization's policy
- for migratable TDs, include the Migration TD and destination platform in the trust review and attestation policy

## 11. Final guest hardening checklist

- [ ] Guest kernel includes TDX support and is kept current
- [ ] Production kernel arguments from section 2 are active
- [ ] Device filter is enabled and only required devices are allowed
- [ ] `tdx_disable_filter` is absent; device exceptions are explicit and reviewed
- [ ] Port I/O and PCI access are restricted
- [ ] ACPI table allow-list is minimal; broad `tdx_allow_acpi` exceptions are absent
- [ ] Runtime uses the intended x2APIC and minimal hypercall/PV configuration
- [ ] OpenSSL is at least 3.3.1 or wolfSSL is at least 5.6.6 where used, with current vendor security updates applied
- [ ] Secret-handling code uses vetted constant-time cryptographic operations and avoids secret-dependent branches, addresses, and early-exit comparisons
- [ ] A fresh quote is obtained and verified with current Intel PCS collateral before secrets are released
- [ ] TCB trust policy is explicit (`early`, `standard`, or an approved evaluation data number)
- [ ] Platform updates, required resets, and any TD restart policy are addressed
- [ ] Migration and destination-platform verification are included when TD migration is used