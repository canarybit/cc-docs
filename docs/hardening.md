# Hardening

This section contains practical hardening guidance for AMD SEV-SNP and Intel TDX Confidential Computing deployments. 
The documents are separated by platform and by host/guest responsibility so operators can find the relevant steps quickly.

!!! info "Recommended order"

    1. Select the platform guide: SEV-SNP or TDX.
    2. Review the host guide with the platform or infrastructure operator.
    3. Review the guest guide with the image and workload owner.
    4. Agree on required configuration and attestation claims before deploying workloads.
    5. Validate the deployed platform and guest state using current vendor documentation and attestation tooling.

!!! warning 

    Some controls are set by the host or at guest launch and cannot be changed from inside a running guest. Coordinate host-side changes with the infrastructure owner.

!!! note 

    Treat attestation as a verification step: define which measurements, policy fields, TCB levels, and freshness requirements the relying party must accept.
    
    **A successful boot alone does not prove that the intended hardening policy is active.**

## AMD SEV-SNP

- [Host hardening](./hardening/snp/host.md): firmware and host software updates, kernel and QEMU support, and restricted-injection host requirements.
- [Guest hardening](./hardening/snp/guest.md): guest policy, PMC and page-movement restrictions, restricted injection requirements, guest identity, and attestation validation.

## Intel TDX

- [Host hardening](./hardening/tdx/host.md): platform firmware and virtualization stack, guest launch controls, and host-side verification responsibilities.
- [Guest hardening](./hardening/tdx/guest.md): kernel parameters, device and ACPI restrictions, side-channel mitigations, and TCB recovery/attestation checks.
