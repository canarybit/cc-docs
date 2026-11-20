# BIOS Setup for Confidential Computing: Intel TDX and AMD SEV-SNP

*The host BIOS must support the features and be configured correctly before the kernel or hypervisor can use them. Menu names, exact option labels, and even the presence of some settings vary by OEM (Dell, Lenovo, HPE, Supermicro, etc.) and by BIOS version. Always check with OEM/ODM or independent BIOS vendor for the precise path and recommended values on the platform. The examples below are common patterns only.*


## 1. Intel TDX

Typical settings (names differ by OEM). 
Source: [Intel's hardware setup page](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/) 

```
Socket Configuration → Processor Configuration
  (or System Security / Processor Details on many OEMs)

  Memory Encryption (TME)                  → Enabled
  Total Memory Encryption Multi-Tenant     → Enabled   (may call MK-TME or Multiple Keys)
  Trust Domain Extension (TDX)             → Enabled
  TDX Secure Arbitration Mode Loader       → Enabled   (SEAM Loader)
  TME-MT / TDX key split                   → non-zero (1 is common)
  SW Guard Extensions (SGX)                → Enabled
  SGX PRM / PRMRR size                     → based on the needs (2 GB is a starting point)

```


### 1.1 Verify from the host OS

```
sudo dmesg | grep -i tdx
```

Expect a line such as `virt/tdx: module initialized`, preceded by `virt/tdx: BIOS enabled: private KeyID range`.


## 2. AMD SEV-SNP

Common settings on [AMD reference](https://github.com/AMDESE/AMDSEV#bios-settings) (names differ by OEM):

```
Advanced → AMD CBS → CPU Common Options
  SMEE                          → Enabled
  SEV Control                   → Enabled
  SEV-ES ASID Space Limit       → 99 or 100 (or whatever concurrent guest count needs)
  SNP Memory (RMP Table) Coverage → Enabled

Advanced → NBIO Common Options → IOMMU/Security
  SEV-SNP Support               → Enabled
  IOMMU                         → Enabled

```

### 2.1 Verify from the host OS

```
sudo dmesg | grep -i -e rmp -e sev
cat /sys/module/kvm_amd/parameters/sev_snp
```

Expect `SEV-SNP: RMP table physical range ...`, `kvm_amd: SEV-SNP enabled (ASIDs 1 - N)`, and `Y` from the sysfs file.


## 3. NVIDIA GPUs

No extra [BIOS settings](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware) are required beyond the CPU TEE (TDX or SEV-SNP) itself. Once the platform is correctly configured for Intel TDX and AMD SEV-SNP, the GPU Confidential Computing features can be enabled from the host OS or driver side.

