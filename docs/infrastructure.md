# Infrastructure

Confidential Cloud is a **cloud-agnostic** platform. [Tower](architecture.md#tower) and [Inspector](architecture.md#inspector) are responsibile for the full Life-Cycle Management (LCM) of the required infrastructure resources.

Confidential Cloud builds on top of Confidential Computing to offer the most advanced encryption mechanisms. Today, this capability is only available in a small set of physical servers provided buy a limited list of Cloud Service Providers (CSPs).

## Hardware Servers

Currently, Confidential Computing capability is only available on few a microprocessors available in the market.
Also, different vendors have different implementation, with different capabilities and performance limitations ([read more](https://www.canarybit.eu/comparing-confidential-computing-platforms/)).

Confidential Cloud currently supports the following hardware:

### AMD SEV-SNP

- Secure Encrypted Virtualization (SEV) - Secure Nested Paging (SNP)

<details closed>
<summary>Coming soon!</summary>
<br>ARM Confidential Compute Architecture (CCA).
<br>&nbsp;
<br>IBM Protected Execution Facility (PEF).
<br>&nbsp;
<br>Intel® Trust Domain Extensions (TDX).
<br>&nbsp;
</details>

<details closed>
<summary>Decomissioned</summary>
<br>Intel® Software Guard Extension (SGX)
<br>&nbsp;
</details>

## Cloud Service Providers (CSPs)

Currently, new execution environments can be provisioned on the following CSPs:

### Azure

- 🇪🇺 Europe - 🇸🇪 Stockholm _(default)_

### OVH (Bare-metal)

- 🇪🇺 Europe - 🇩🇪 Frankfurt _(default)_

### Openstack

- 🇪🇺 Europe - 🇸🇪 Stockholm _(default)_

### AWS

- 🇪🇺 Europe - 🇸🇪 Stockholm _(default)_

## On-prem / Air-gapped

It is also possible to deploy the required resources On-prem, for a fully **air-gapped solution**.

Get in touch for more information!
