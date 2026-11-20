# BIOS Setup for Confidential Computing: Intel TDX and AMD SEV-SNP

*What to change in firmware once you have the hardware. Compiled from OEM, Intel, AMD and community documentation. Menu names vary by OEM, BIOS generation and firmware version, so treat every path below as a starting point and confirm it on your own machine.*


## 1. Naming cheat sheet

### Intel TDX

Sources: [Intel guide](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/).

| Concept | Intel reference BIOS |
|---|---|
| Base memory encryption | Memory Encryption (TME), Total Memory Encryption Multi-Tenant (TME-MT) |
| TDX switch | Trust Domain Extension (TDX) |
| Module loader | TDX Secure Arbitration Mode Loader (SEAM Loader) |
| Key split | TME-MT/TDX key split |
| SGX (for attestation) | SW Guard Extensions (SGX) |
| Physical address limit | Limit CPU PA to 46 bits |

### AMD SEV-SNP

Sources: [AMDSEV project](https://github.com/AMDESE/AMDSEV).

| Concept | AMD reference (CBS) |
|---|---|
| Memory encryption | SMEE |
| SEV switch | SEV Control |
| ASID split | SEV-ES ASID Space Limit |
| SNP switch | SEV-SNP Support |
| RMP table | SNP Memory (RMP Table) Coverage |
| Virtualization | SVM Mode |
| IOMMU | IOMMU |

## 2. Intel TDX

### 2.1 What the settings do

Setting descriptions follow [Intel's hardware setup page](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/) unless noted.

- **TME** turns on Total Memory Encryption, the prerequisite for everything else.
- **TME-MT / MK-TME** adds multiple keys. TDX uses it for the main memory encryption of Trust Domains.
- **TDX and SEAM Loader.** TDX activates the feature. The SEAM Loader controls where the TDX module is loaded from (BIOS or the EFI system partition).
- **TME-MT/TDX key split** divides the key IDs between plain MK-TME and TDX. It must be non-zero. [Red Hat](https://developers.redhat.com/articles/2025/02/19/how-deploy-confidential-containers-bare-metal) notes that 1 allows up to 64 concurrent TDX VMs.
- **SGX** is used by Intel's TDX attestation setup for quote generation. Enable it when configuring an attestation-enabled TDX deployment.
- **Memory integrity** adds cryptographic integrity using ECC bits. Intel says it requires DDR5 10x4 DIMMs and reduces error-correction margin.
- **TME bypass** lets non-confidential memory skip encryption for performance. It is optional.

### 2.2 Baseline values in Intel TDX

| Setting | Value | Notes |
|---|---|---|
| TME / Memory Encryption | Enabled | Dell: Memory Encryption = Multiple Keys ([Dell KB](https://www.dell.com/support/kbdoc/en-uk/000226452/enableinteltdxondell16g)) |
| TME bypass | Auto | Intel's Quanta example shows Disabled, [Canonical](https://ubuntu.com/server/docs/how-to/virtualisation/intel-tdx) calls Enabled optional, [Dell's walkthrough](https://www.dell.com/en-us/infohub/articles/enabling-intel-r-tdx-on-dell-poweredge) shows Enabled, [Lenovo](https://lenovopress.lenovo.com/lp2383.pdf) says Auto. Not required for TDX. |
| TME-MT / MK-TME | Enabled | [Intel](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/), [Lenovo](https://lenovopress.lenovo.com/lp2383.pdf) |
| Memory integrity | Disabled | Intel allows either. [Dell](https://www.dell.com/support/kbdoc/en-uk/000226452/enableinteltdxondell16g), [NVIDIA](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware) and [Canonical](https://ubuntu.com/server/docs/how-to/virtualisation/intel-tdx) use Disabled. Only enable with DDR5 10x4 DIMMs. |
| TDX | Enabled | All sources |
| SEAM Loader | Enabled | All sources |
| TME-MT/TDX key split | 1 (any non-zero) | [Dell](https://www.dell.com/support/kbdoc/en-uk/000226452/enableinteltdxondell16g), [Lenovo](https://lenovopress.lenovo.com/lp2383.pdf), [Red Hat](https://developers.redhat.com/articles/2025/02/19/how-deploy-confidential-containers-bare-metal) |
| SGX | Enabled (Dell calls it On) | Used for TDX attestation in Intel's setup ([Intel](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/)) |
| PRMRR / SGX PRM size | 2G | Used in [Dell](https://www.dell.com/en-us/infohub/articles/enabling-intel-r-tdx-on-dell-poweredge) and [HPE](https://infoages.tistory.com/2440) walkthroughs. Intel says a minimum is needed to run the quote generation service. Size up only for real SGX enclave workloads: [VMware](https://blogs.vmware.com/cloud-foundation/2026/09/15/confidential-computing-complete-setup-walkthrough-intel-tdx/) suggests 2 to 4 GB for light services, 64 to 128 GB for large confidential databases, plus 10 to 15% headroom. |
| Limit CPU PA to 46 bits | Disabled | [Intel](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/), [Lenovo](https://lenovopress.lenovo.com/lp2383.pdf), [NVIDIA](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware) |
| Disable excluding Mem below 1MB in CMR | Auto | [Lenovo](https://lenovopress.lenovo.com/lp2383.pdf) and [NVIDIA](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware) say Auto. A [Dell walkthrough](https://www.dell.com/en-us/infohub/articles/enabling-intel-r-tdx-on-dell-poweredge) sets Disabled. |
| x2APIC (Dell) | Enabled | [Dell KB](https://www.dell.com/support/kbdoc/en-uk/000226452/enableinteltdxondell16g) |
| Node Interleaving (Dell) | Disabled | [Dell KB](https://www.dell.com/support/kbdoc/en-uk/000226452/enableinteltdxondell16g) |

### 2.3 Where to find them, by OEM

**Intel reference BIOS.** On a Quanta S6Q with 5th Gen Xeon: Socket Configuration > Processor Configuration. On the Beechnut City Xeon 6 reference: Socket Configuration > Security Configuration, with the PA limit under Uncore Configuration > Uncore General Configuration ([Intel](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/)).

**Dell PowerEdge** (F2 at boot, System Setup > System BIOS). Sources: [Dell KB 000226452](https://www.dell.com/support/kbdoc/en-uk/000226452/enableinteltdxondell16g), [Dell InfoHub](https://www.dell.com/en-us/infohub/articles/enabling-intel-r-tdx-on-dell-poweredge), [VMware](https://blogs.vmware.com/cloud-foundation/2026/09/15/confidential-computing-complete-setup-walkthrough-intel-tdx/).

- Memory Settings: Node Interleaving = Disabled
- Processor Settings: x2APIC Mode = Enabled, CPU Physical Address Limit = Disabled
- System Security: Memory Encryption = Multiple Keys, Global Memory Integrity = Disabled, Intel TDX = Enabled, TME-MT/TDX Key Split = 1, SEAM Loader = Enabled, Intel SGX = Enabled, PRMRR Size as above
- Some Dell walkthroughs also set Intel TXT to On ([InfoHub](https://www.dell.com/en-us/infohub/articles/enabling-intel-r-tdx-on-dell-poweredge)). Dell's own 16G KB does not list it as required.
- The 17G R770 with Xeon 6 in VMware's lab uses the same System Security page for TME, MK-TME, TDX, SEAM, SGX and PRMRR.
- Dell's KB covers 16G models with 5th Gen Xeon (R660, R760, R860, R960, XE9680 and others).

**Lenovo ThinkSystem** (F1 at boot, System Settings > Processors; [LP2418](https://lenovopress.lenovo.com/lp2418.pdf) labels the page Processor Details). Source: [Lenovo LP2383](https://lenovopress.lenovo.com/lp2383.pdf).

1. Total Memory Encryption (TME) = Enabled
2. TME Bypass = Auto
3. Multikey TME (MK-TME) = Enabled
4. Trust Domain Extension (TDX) = Enabled
5. TDX Secure Arbitration Mode Loader = Enabled
6. Disable excluding Mem below 1MB in CMR = Auto
7. TME-MT/TDX key split = 1
8. SW Guard Extensions (SGX) = Enabled
9. Limit CPU PA to 46 bits = Disabled

**HPE ProLiant** (System Utilities > System Configuration > BIOS/Platform Configuration (RBSU))

- HPE's [Gen11 System ROM 2.10 release notes](https://downloads.hpe.com/pub/softlib2/software1/pubfw-uefi/p1722410572/v229054/U54_2.10_11_28_2023) confirm RBSU options for TDX were added in that release.
- [community walkthrough](https://infoages.tistory.com/2440) on a DL380a Gen12, not an HPE document: Server Security > Intel Security Options: SGX = Enabled, TDX = Enabled, SEAM Loader = Enabled, TME-MT/TDX key split = 1, PRMRR Size = 2G, and in that walkthrough also Intel TXT Support = Enabled. Processor Options > Processor Physical Addressing = Default. Workload Profile = Virtualization - Max Performance.


**Supermicro**

- Supermicro's [FAQ](https://www.supermicro.com/support/faqs/faq.php?faq=42133) lists three requirements: a TDX-capable BIOS, a TDX-capable CPU, and 8 DIMMs per CPU.
- [NVIDIA's deployment guide](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware) shows an AMI-style layout (CPU Configuration > Processor Configuration > Intel TME, Intel TME-MT, Intel TDX) with the same settings, but does not name the board vendor.

### 2.4 Verify from the host OS

```
sudo dmesg | grep -i tdx
```

Expect a line such as `virt/tdx: module initialized`, preceded by `virt/tdx: BIOS enabled: private KeyID range` ([Lenovo LP2383](https://lenovopress.lenovo.com/lp2383.pdf)). [Canonical](https://ubuntu.com/server/docs/how-to/virtualisation/intel-tdx) also documents two MSR checks (msr-tools): bit 1 of MSR 0x982 is 1 when MK-TME is on, and bit 11 of MSR 0x1401 is 1 when the SEAM range register is enabled.


## 3. AMD SEV-SNP

### 3.1 What the settings do

- **SMEE / SME** enables AMD memory encryption and SEV and SNP depend on it.
- **SEV Control** enables SEV ([Lenovo LP2349](https://lenovopress.lenovo.com/lp2349-configuring-confidential-computing-with-amd-sev-snp-and-vmware-esxi-90)).
- **SEV-ES ASID Space Limit** (Dell: Minimum SEV non-ES ASID) splits the ASID pool. ASIDs below the limit serve SEV-ES and SNP guests. The rest serve plain SEV. Linux shows this as, for example, `SEV-SNP enabled (ASIDs 1 - 99)` for a limit of 100 ([Lenovo LP2383](https://lenovopress.lenovo.com/lp2383.pdf)), so the limit affects how many concurrent SNP guests you can run. Exact allocation behavior and defaults vary by OEM and processor family.
- **SEV-SNP Support** enables the SNP feature set (memory integrity, guest isolation).
- **SNP Memory (RMP Table) Coverage** reserves memory for the Reverse Map Table that tracks page ownership. The RMP must cover all of memory for Linux to enable SNP ([Linux kernel docs](https://www.kernel.org/doc/html/v6.15/arch/x86/amd-memory-encryption.html)).
- **SVM Mode and IOMMU** must be on. SNP uses the IOMMU to stop devices DMAing into guest private memory ([SUSE](https://documentation.suse.com/en-us/sles/15-SP7/html/SLES-all/cha-vm-security.html)).

### 3.2 Baseline values

| Setting | Value | Notes |
|---|---|---|
| SVM Mode / Virtualization Technology | Enabled | [Lenovo LP2349](https://lenovopress.lenovo.com/lp2349-configuring-confidential-computing-with-amd-sev-snp-and-vmware-esxi-90) |
| SMEE / SME | Enabled | [AMDSEV](https://github.com/AMDESE/AMDSEV) |
| SEV Control | Enabled | [Lenovo LP2349](https://lenovopress.lenovo.com/lp2349-configuring-confidential-computing-with-amd-sev-snp-and-vmware-esxi-90) |
| SEV-ES ASID Space Limit | 99 or 100 for older EPYC platforms | [AMDSEV](https://github.com/AMDESE/AMDSEV) and [NVIDIA](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware) use 99/100. Lenovo's [ESXi paper](https://lenovopress.lenovo.com/lp2349-configuring-confidential-computing-with-amd-sev-snp-and-vmware-esxi-90) uses 10. Pick a value supported by the processor and BIOS based on the concurrent SNP guests needed. |
| SEV-SNP Support | Enabled | Lives under NBIO Common Options > IOMMU/Security on AMD reference BIOS ([AMDSEV](https://github.com/AMDESE/AMDSEV)) |
| SNP Memory (RMP Table) Coverage | Enabled | See Hyper-V and EPYC 9004/9005 notes below |
| IOMMU | Enabled | Hypervisor may need non-passthrough mode ([SUSE](https://documentation.suse.com/en-us/sles/15-SP7/html/SLES-all/cha-vm-security.html)) |

### 3.3 Where to find them, by OEM

**AMD reference BIOS** ([AMDSEV project](https://github.com/AMDESE/AMDSEV)): Advanced > AMD CBS > CPU Common Options for SMEE, SEV Control, SEV-ES ASID Space Limit and SNP Memory (RMP Table) Coverage. Then Advanced > NBIO Common Options > IOMMU/Security > SEV-SNP Support. [NVIDIA's guide](https://docs.nvidia.com/confidential-computing/virtual-machines/deployment-guide/hardware) shows an AMI-style variant: Advanced > CPU Configuration (SMEE, SEV ASID Count 509, SEV-ES ASID Space Limit Control = Manual, limit 100, SNP Memory Coverage) and Advanced > NB Configuration (IOMMU, SEV-SNP Support).

**Dell PowerEdge** (System Setup > System BIOS > Processor Settings). Source: [R6525 walkthrough](https://arkivm.github.io/sev/2022/11/25/running-amd-svsm).

- Virtualization Technology, IOMMU Support and Kernel DMA Protection = Enabled
- Secure Memory Encryption (SME) = Enabled
- Minimum SEV non-ES ASID = a value above the default of 1
- Secured Nested Paging (SNP) = Enabled (Dell's manual says Disabled by default)
- SNP Memory Coverage = Enabled
- Boot Mode = UEFI
- This list comes from Dell's R6525 (EPYC 7003) manual and a third-party walkthrough. Names on newer EPYC 9004/9005 PowerEdge models may differ slightly.

**Lenovo ThinkSystem** (F1 at boot). Sources: [LP2349](https://lenovopress.lenovo.com/lp2349-configuring-confidential-computing-with-amd-sev-snp-and-vmware-esxi-90), [LP2383](https://lenovopress.lenovo.com/lp2383.pdf), [LP1893](https://lenovopress.lenovo.com/lp1893).

- System Settings > Memory: SMEE = Enabled, SEV Control = Enabled, SEV-ES ASID Space Limit (10 in the ESXi paper, 100 in the NVIDIA GPU paper)
- System Settings > Processors: SVM Mode = Enabled, SEV-SNP Support = Enabled, SNP Memory (RMP Table) Coverage = Enabled
- IOMMU = Enabled (shown under a Secured-Core menu in the GPU paper)
- For EPYC 9004/9005, Lenovo's SNP paper (LP1893) shows RMP coverage set to Custom with an Amount of Memory to Cover value instead of Enabled. Check which options your BIOS offers.
- **Hyper-V exception:** Lenovo's Hyper-V guidance sets SNP Memory (RMP Table) Coverage to Disabled, because Hyper-V allocates and manages its own RMP rather than using platform firmware.

**HPE ProLiant** (RBSU)

- HPE's UEFI guide documents [AMD Secure Memory Encryption](https://www.hpe.com/psnow/resources/ebooks/a00112581en_us_v4/t_config_AMD_sec_memory_encryption.html) under System Configuration > BIOS/Platform Configuration (RBSU) > Memory Options. The guide's table of contents also lists "Enabling or disabling AMD Secure Nested Paging" in the Memory Encryption Options section ([Gen11 UEFI guide](https://itpfdoc.hitachi.co.jp/manuals/ha8000v/hard/Gen11/UEFI/30-6E440C90-004.pdf)). Verify the exact SNP label on your ROM.
- Virtualization options include AMD Virtualization Technology, AMD DMA Remapping and Minimum SEV ASID (same guide family).
- Check the Workload Profile if options are locked ([HPE RBSU guide](https://www.manualslib.com/manual/3256824/Hpe-Proliant.html?page=34)).

**Supermicro**

- Supermicro lists fixed BIOS versions for the RMP flaw on H12, H13 and H14 boards, for example H13SSL-N 3.7 and H14SSL-N 1.7 ([Supermicro advisory](https://www.supermicro.com/en/support/security_AMD-SB-3020)). Check it for your SKU.

### 3.4 Verify from the host OS

```
sudo dmesg | grep -i -e rmp -e sev
cat /sys/module/kvm_amd/parameters/sev_snp
```

Expect `SEV-SNP: RMP table physical range ...`, `kvm_amd: SEV-SNP enabled (ASIDs 1 - N)`, and `Y` from the sysfs file ([AMDSEV](https://github.com/AMDESE/AMDSEV), [Lenovo LP2383](https://lenovopress.lenovo.com/lp2383.pdf)). The AMDSEV project lists minimums of host kernel 6.11 and QEMU 9.1 for SNP. Inside a guest, `Memory Encryption Features active: AMD SEV SEV-ES SEV-SNP` confirms it ([SUSE](https://documentation.suse.com/en-us/sles/15-SP7/html/SLES-all/cha-vm-security.html)).




