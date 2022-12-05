# Infrastructure

Confidential Cloud is a **cloud-agnostic** platform. [Tower](architecture.md#tower) and [Inspector](architecture.md#inspector) are responsibile for the full Life-Cycle Management (LCM) of the required infrastructure resources.

Confidential Cloud builds on top of Confidential Computing to offer the most advanced encryption mechanisms. Today, this capability is only available in a small set of physical servers provided buy a limited list of Cloud Service Providers (CSPs).

## Supported Hardware

Currently, Confidential Computing capability is only available on few a microprocessors available in the market.
Also, different vendors have different implementation, with different capabilities and performance limitations ([Read more](https://www.canarybit.eu/comparing-confidential-computing-platforms/)).

Confidential Cloud currently supports the following hardware:

### AMD SEV-SNP

- Secure Encrypted Virtualization (SEV) - Secure Nested Paging (SNP): [https://www.amd.com/en/processors/amd-secure-encrypted-virtualization](https://www.amd.com/en/processors/amd-secure-encrypted-virtualization)

<!--
<details closed>
<summary>Coming in 2023-2024</summary>
<br>ARM Confidential Compute Architecture (CCA).
<br>&nbsp;
<br>IBM Protected Execution Facility (PEF).
<br>&nbsp;
<br>Intel® Trust Domain Extensions (TDX).
<br>&nbsp;
</details>
-->

<details closed>
<summary>Decomissioned</summary>
<br>- Intel® Software Guard Extension (SGX)
<br><a href="https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html" target="_blank">https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html</a>
<br>&nbsp;
</details>

## Cloud Service Providers (CSPs)

Currently, new execution environments can be provisioned on the following CSPs:

### MS Azure

- **Europe** (🇸🇪 Stockholm) : _default_

### OVH (Bare-metal)

- **Europe** (🇩🇪 Frankfurt) : _default_

### Openstack

- **Europe** (🇸🇪 Stockholm) : _default_
- **Europe** (🇮🇹 Italy)

### AWS

- **Europe** (🇸🇪 Stockholm) : _default_

<br>

<details open>
<summary>Partnership Programme</summary>
<br>Are you a CSP and interested to support Confidential Cloud?
<br>Join our <a href="https://www.canarybit.eu/become-a-partner/">Partnership programme</a>!
<br>&nbsp;
</details>

## On-prem / Air-gapped

It is also possible to deploy the required resources On-prem, for a fully **air-gapped** solution.

[Get in touch](https://www.canarybit.eu/contact/) for more information!
