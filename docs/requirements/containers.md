# Containers

## Platforms
- [Kubernetes](https://kubernetes.io/)
- [Openshift](https://www.redhat.com/en/technologies/cloud-computing/openshift) `v4.20+`

!!! note 

    Confidential VMs do not currenly support nested virtualization.

## Runtime

- [Kata-runtime](https://katacontainers.io/) (recommended for full isolation)
    - `kata-qemu-snp` for AMD SEV-SNP 
    - `kata-qemu-tdx` for Intel TDX
- Any other container runtime