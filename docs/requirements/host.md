# Host

## BIOS

The host BIOS must support the features and be configured correctly before the kernel or hypervisor can use them. Menu names, exact option labels, and even the presence of some settings vary by OEM (Dell, Lenovo, HPE, Supermicro, etc.) and by BIOS version.

!!! warning 
    **Always check with OEM/ODM or independent BIOS vendor** for the precise path and recommended values on the platform. The examples below are common patterns only.

### Intel TDX

#### Settings

```
Socket Configuration → Processor Configuration

    Memory Encryption (TME)                             → Enabled
    Total Memory Encryption (TME) Bypass                → Disabled
    Total Memory Encryption Multi-Tenant (TME-MT)       → Enabled
    Memory Integrity                                    → Enabled 
    Trust Domain Extension (TDX)                        → Enabled
    TDX Secure Arbitration Mode Loader (SEAM Loader)    → Enabled 
    TME-MT / TDX key split                              → 1
    SW Guard Extensions (SGX)                           → Enabled
    SGX PRM / PRMRR size                                → 2 GB (or more)
```

More information on the [Intel official host setup documentation.](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/){:target="_blank"}

#### Verification

Log into the host and run:

```
sudo dmesg | grep -i tdx
```

Expect a line such as `virt/tdx: module initialized`, preceded by `virt/tdx: BIOS enabled: private KeyID range`.


### AMD SEV-SNP

#### Settings

```
Advanced → AMD CBS → CPU Common Options

    SMEE                              → Enabled
    SEV Control                       → Enabled
    SEV-ES ASID Space Limit           → 100
    SNP Memory (RMP Table) Coverage   → Enabled

Advanced → NBIO Common Options → IOMMU/Security

    SEV-SNP Support                   → Enabled
    IOMMU                             → Enabled

```

More information on the [AMD official host setup documentation.](https://github.com/AMDESE/AMDSEV#prepare-host){:target="_blank"}

#### Verification

Log into the host and run:

```
sudo dmesg | grep -i -e rmp -e sev
cat /sys/module/kvm_amd/parameters/sev_snp
```

Expect `SEV-SNP: RMP table physical range ...`, `kvm_amd: SEV-SNP enabled (ASIDs 1 - N)`, and `Y` from the sysfs file.

### NVIDIA GPU

**No extra BIOS settings are required.**

Once the host BIOS is [correctly configured](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware){:target="_blank"} for Intel TDX and AMD SEV-SNP, the GPU Confidential Computing features can be enabled from the host OS or driver side.

## Additional Software

### Intel QGS

For **Intel TDX attestation only**, the [Intel Quote Generation Service (QGS)](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/05/host_os_setup/#install-qgs){:target="_blank"} must be installed on the host. 

### NVIDIA GPU Admin Tools

For **NVIDIA attestation only**, the [NVIDIA GPU admin tools](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/host){:target="_blank"} and required drivers must be configured on the host. 