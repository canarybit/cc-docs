# AMD SEV-SNP - Host Hardening 

This guide covers the host-side hardening controls for AMD SEV-SNP environments. It focuses on firmware, kernel, hypervisor, launch-time policy, and attestation checks that a host owner must validate before production deployment.

## 1. Objective

The host must ensure that confidential guests are launched with the correct protections and that the virtualization stack does not enable unsafe features.

Key host-side objectives:

- keep firmware and host software updated
- verify the kernel supports SEV-SNP 
- verify QEMU/KVM supports the required guest launch features
- ensure guest policy disables unsafe features
- validate attestation results 

## 2. Prerequisites

### 2.1 Confirm CPU and platform capabilities

Check platform logs:

```bash
dmesg | grep -Ei 'sev|snp|kvm'
journalctl -k | grep -Ei 'sev|snp|kvm'
```

Check Current SEV-SNP Status:

```bash
# Quick capability check
sudo dmesg | grep -E 'SEV|SNP|RMP'

# Using sevctl / snphost (recommended)
# Install from https://github.com/virtee/snphost
sevctl ok snp
# or
snphost ok
```
Expected output should confirm SEV-SNP and RMP table initialization.

## 3. Update firmware and virtualization stack

### 3.1 Firmware

Update the platform BIOS/UEFI and AMD PSP/SEV-SNP firmware:
Download the latest firmware matching the CPU family from the AMD developer site (https://www.amd.com/en/developer/sev.html).

Use these commands on the host to find CPU family:

```bash
lscpu
cat /proc/cpuinfo
```
More specifically: 

```bash
lscpu | egrep 'Architecture|Vendor ID|Model name|CPU family|Model|Stepping'
```

After finding the CPU family download the new firmware.
Example for EPYC 7003 (Milan / Fam 19h Model 0xh):

```bash
wget https://download.amd.com/developer/eula/sev/amd_sev_fam19h_model0xh_1.58.02.zip
unzip amd_sev_fam19h_model0xh_1.58.02.zip
```

Install fimware:

```bash
sudo mkdir -p /lib/firmware/amd
sudo cp amd_sev_fam19h_model0xh_1.58.02.sbin /lib/firmware/amd/amd_sev_fam19h_model0xh.sbin
```
Reload the CCP/SEV driver (or reboot):

```bash
sudo rmmod kvm_amd
sudo rmmod ccp
sudo modprobe ccp
sudo modprobe kvm_amd
```
Verfy:

```bash
sudo dmesg | grep -E 'SEV-SNP|firmware|PSP'
snphost show   
```

Example output after a successful update:

```bash
[   ] SEV-SNP: RMP table physical range [0x0000000093300000 - 0x00000000a38fffff]
[   ] AMD-Vi: IOMMU SNP support enabled.
[   ] AMD-Vi: Extended features (0x841f77e022094ace, 0x0): PPR X2APIC NX IA GA PC SNP
[   ] ccp 0000:45:00.1: SEV firmware updated from 1.55.31 to 1.58.2
[   ] ccp 0000:45:00.1: SEV-SNP API:1.58 build:2
[   ] ccp 0000:45:00.1: SEV API:1.58 build:2
[   ] kvm_amd: SEV enabled (ASIDs 100 - 509)
[   ] kvm_amd: SEV-ES enabled (ASIDs 1 - 99)
[   ] kvm_amd: SEV-SNP enabled (ASIDs 1 - 99)
```

### 3.2 Host Kernel Upgrade

Use the distro or vendor update flow:

```bash
sudo apt update && sudo apt upgrade
# or
sudo dnf upgrade -y
# or
sudo zypper update
```

Check the active kernel:

```bash
uname -r
```

## 4. Restricted Injection (Ahoi / WeSee / Heckler mitigation)

Restricted injection limits the interrupts a VMM can inject into an SNP guest. It is intended to mitigate malicious-interrupt attacks such as WeSee and Heckler. Support and setup depend on the exact kernel, QEMU, OVMF, and guest-kernel versions; do not infer availability from a version number or from the presence of a source file alone.

The host needs a kernel with the Restricted Injection KVM/SVM support and #HV doorbell handling.

QEMU and SNP-aware OVMF must support the launch configuration for Restricted Injection. In the SNP firmware ABI, the corresponding launch feature is `SEV_FEATURES[3]`.

Do not rely on older statements about whether support is mainline or exposed as a QEMU property; verify the current release notes and configuration interface for each deployed component.

The guest kernel must also support #HV doorbell handling.

Check whether the kernel includes the required support for SEV-SNP and restricted injection.

```bash
grep -RIn "restricted injection\|#HV doorbell\|doorbell page" /lib/modules/$(uname -r)/build/arch/x86/kvm/svm 2>/dev/null
```

If you have a local kernel tree:

```bash
git -C /usr/src/linux log --grep='restricted injection' --grep='#HV'
git -C /usr/src/linux log --grep='doorbell'
```

The host must confirm all layers support restricted injection before enabling it in production.

Required support stack:

- host kernel: yes
- KVM/SVM support: yes
- QEMU: yes
- guest firmware: SNP-aware, supports #HV
- guest kernel: guest-side #HV support, yes

If a layer is missing support, do not assume the guest receives the protection.

References:

- [AMD SEV-SNP Firmware ABI Specification (Publication 56860)](https://www.amd.com/content/dam/amd/en/documents/developer/56860.pdf) — ABI definition of launch feature bits, including `SEV_FEATURES[3]`.
- [Linux KVM AMD memory-encryption documentation](https://docs.kernel.org/virt/kvm/x86/amd-memory-encryption.html) and the [Linux KVM/SVM SEV implementation](https://github.com/torvalds/linux/blob/master/arch/x86/kvm/svm/sev.c) — host-side SEV/KVM context. These do not by themselves establish that a particular kernel build includes Restricted Injection.

## 5. Host production checklist

Before deployment, confirm the following:

- [ ] BIOS/UEFI and AMD PSP/firmware are updated
- [ ] host kernel supports SEV-SNP and required KVM features
- [ ] QEMU/KVM stack supports the guest launch configuration
- [ ] restricted injection is enabled only where all layers support it
- [ ] host launch settings match the approved guest policy and are confirmed by attestation policy

## 6. Final host guidance

A hardened SEV-SNP host is not defined by one setting alone. It is a combination of:

- updated firmware
- modern kernel and hypervisor
- correct launch-time feature selection
- a verified guest policy

Guest-policy requirements, including PMC virtualization, `PAGE_SWAP_DISABLE`, and attestation checks such as `ALIAS_CHECK_COMPLETE`, are covered in the [SNP guest hardening guide](guest.md). The host owner must verify actual platform state rather than assume default values are secure.