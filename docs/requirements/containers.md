# Containers

## Platforms
- [Kubernetes](https://kubernetes.io/)
- [Openshift](https://www.redhat.com/en/technologies/cloud-computing/openshift) `v4.20+`

**Note.** Confidential VMs do not currenly support nested virtualization.

## Runtime

- [Kata-runtime](https://katacontainers.io/) (Recommended for full isolation)
    - `kata-qemu-snp` for AMD SEV-SNP 
    - `kata-qemu-tdx` for Intel TDX
- Any other container runtime