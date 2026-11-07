# Intel TDX Host Hardening

This guide covers the host-side environment for Intel TDX deployments. It focuses on the host requirements needed to launch and manage hardened TD guests safely.

## 1. Objective

The host should provide a trusted execution environment for TDX guests and ensure the guest launch configuration does not relax the security model.

Key host-side duties:

- keep platform firmware and TDX module updated
- keep kernel up to date
- verify host and hypervisor support for trusted guest deployment
- ensure guest launches use a hardened configuration
- confirm attestation data reflects the expected runtime policy

## 2. Prerequisites

### 2.1 Confirm platform support

```bash
uname -a
lscpu
grep -Ei 'tdx|vmx' /proc/cpuinfo
```

Check kernel messages and logs:

```bash
dmesg | grep -Ei 'tdx|kvm'
journalctl -k | grep -Ei 'tdx|kvm'
```
Make sure there is no error in logs.

## 3. Keep firmware and host kernel updated

### 3.1 Platform firmware

Update BIOS/UEFI and Intel TDX-related firmware or module versions as follows.

Update Intel TDX Module via BIOS update

Steps:

- Reach out to your OEM/ODM or Independent BIOS Vendor (IBV) to ask for a BIOS containing another version of the Intel TDX Module.
- Once available, retrieve the BIOS update.
- Flash BIOS update according the instructions of the BIOS provider.


Update Intel TDX Module via Binary Deployment

Steps:

- Download the Intel TDX Module binary:

To download a specific version of an Intel TDX Module and a corresponding signature structure, navigate to the releases page of the [Intel TDX Module](https://github.com/intel/confidential-computing.tdx.tdx-module/releases). Download the archive intel_tdx_module.tar.gz from the release you want to use.


- Unpack the downloaded archive:

```
tar -xvzf intel_tdx_module.tar.gz
```

If not done before, create an EFI directory:
```
sudo mkdir -p /boot/efi/EFI/TDX/
```

- Copy the Intel TDX Module binary and the corresponding signature structure to the EFI directly created in step 3.
```
sudo cp TDX-Module/intel_tdx_module.so \
    /boot/efi/EFI/TDX/TDX-SEAM.so
sudo cp TDX-Module/intel_tdx_module.so.sigstruct \
    /boot/efi/EFI/TDX/TDX-SEAM.so.sigstruct
```
- Check that the copied files are present in /boot/efi/EFI/TDX/ with a current date:
```
sudo ls -ls /boot/efi/EFI/TDX/
```
- Reboot the host.

References:

- [Intel TDX Enabling Guide ](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/#update-intel-tdx-module-via-binary-deployment) — Update TDX module.
- [Intel TDX Module releases](https://github.com/intel/confidential-computing.tdx.tdx-module/releases) — release binaries, signature structures, hashes, and release-specific instructions. Verify downloaded artifacts against the published release information and follow the instructions for the target platform.
- For BIOS/UEFI updates, follow the instructions and compatibility guidance published by the system OEM/ODM or IBV.

### 3.2 Host virtualization stack

Update the host kernel and KVM/QEMU stack to the latest supported versions for TDX guest launch and secure runtime behavior.

```bash
sudo apt update && sudo apt upgrade
# or
sudo dnf upgrade -y
# or
sudo zypper update
```

## 4. Make sure the host launches guests with hardened defaults

The host/VMM supplies the guest command line when it owns guest boot configuration. Apply the kernel parameters listed in the [TDX guest guide](guest.md#2-required-production-kernel-parameters) through the platform's supported launch configuration. If the guest image manages its own bootloader, coordinate with its owner and verify that the resulting launch matches the approved profile.

These guest parameters and their in-guest verification are maintained in the guest guide to avoid conflicting copies.

## 5. Keep the device and I/O surface minimal

The host should not launch guests that disable TDX device filtering or admit broad untrusted device access.

### 5.1 Avoid unsafe guest launch patterns

Do not pass this guest kernel command-line parameter in production; it disables the device filter:

```text
tdx_disable_filter
```

Avoid broad ACPI or device exceptions unless justified and fully reviewed.

### 5.2 Recommended controls

- keep the device allow-list minimal
- restrict port I/O access
- limit ACPI table acceptance
- avoid unrestricted device expansion

In summary do not use in production:

```text
tdx_disable_filter
unrestricted tdx_allow_acpi=
unrestricted authorize_allow_devs=
```

These options are measured into the attestation quote (RTMRs / cmdline).

## 6. Ensure only trusted guest configuration is accepted

The host should reject or at least flag launch profiles with:

- broad ACPI permitting
- unrestricted device allow-lists
- disabled filtering
- legacy timer behavior
- weak entropy configuration

## 7. Verify launch configuration and guest attestation

Before production, confirm that the VMM launch configuration supplies the approved guest profile and that attestation verifies the required runtime claims. The guest owner checks its active `/proc/cmdline` as described in the guest guide; the host owner checks the launch configuration and the verified quote/claims.

- filter state
- allowed device set
- ACPI restrictions
- any launch-measured command-line or runtime claims required by policy

## 8. Host production checklist

- [ ] host firmware updated
- [ ] host kernel and KVM/QEMU stack updated
- [ ] guest launch config follows hardened profile
- [ ] device filter remains enabled
- [ ] no broad ACPI or device exceptions enabled
- [ ] `no-kvmclock` enforced
- [ ] `random.trust_cpu=y` enforced
- [ ] guest attestation validates runtime security posture

## 9. Final host guidance

A secure Intel TDX host environment depends on controlled guest launch policy and runtime verification. The host must not assume that a running TD is secure simply because it boots; attestation and runtime policy checks are required.