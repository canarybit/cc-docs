# CanaryBit Inspector

*Audit Trusted Execution Environments (TEE).*

CanaryBit [Inspector](https://docs.confidentialcloud.io/architecture/#inspector) is a Confidential Computing [Attestation (RA)]() service. It helps end-users verify the security of their processing environments and ensure user-defined policies are met before any application run or data analysis.

## Requirements

- A [CanaryBit Licence](inspector.md#licences)
- A [supported](supported-setups.md) technology stack.
    
## Verification

CanaryBit Inspector verifies Confidential VMs (cVM) and Containers deployed in Public, Private or On-Prem cloud infrastructures.

### cbclient

The `cbclient` is the client implementation for the CanaryBit Inspector Attestation service. Written in [Rust](https://www.rust-lang.org/), it is responsible to collect the attestation data and call the CanaryBit Inspector API to verify the execution environments (Confidential VMs or containers).

### Confidential VMs

CanaryBit Inspector verifies Confidential VMs (cVM) deployed:

  1. **By CanaryBit Tower:**
    
      A Confidential VM is **automatically** deployed and attested by injecting the CanaryBit `cloud-init` file. Examples of different `cloud-init` configurations can be found [here](tower.md).

  2. **By other means:**
   
      The end-user provisions a Confidential VM on the infrastructure provider and injects the CanaryBit `cloud-init` file at creation time.

### Containers

Identically, CanaryBit Inspector can be used to constantly verify the Confidential Nodes where containers are running, no matter if they are managed by an orchestration service (e.g. AKS) or directly on a container platform.

CanaryBit Inspector verifies the Cluster Node characteristics:
- before a container is executed (always);
- at a defined schedule (if enabled);
- when specific system-wide events occur (if enabled).

This is **extremely beneficial** for dynamic clusters with the `autoscaling` feature enabled.

[CanaryBit Surveyor]() provides the configuration to enable Container Attestation for several use-cases.

## Licences

CanaryBit Inspector can be deployed on-prem for internal use or offered as a service.

### Free Trial

Request a 1-month FREE trial via our [contact form](https://www.canarybit.eu/contacts)!

### Enterprise

[Choose the licence](https://www.canarybit.eu) that fits your needs and get CanaryBit Inspector up to speed in minutes. 

### Reseller

If you are interested in offering CanaryBit Inspector as a Service (SaaS), join our [Reseller Programme](https://www.canarybit.eu/contacts)!