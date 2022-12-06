# Infrastructure

Confidential Cloud is **cloud-agnostic**. [Tower](architecture.md#tower) and [Inspector](architecture.md#inspector) are responsible for various aspects of the Life-Cycle Management (LCM) of infrastructure resources.

Confidential Cloud builds atop Confidential Computing to offer state-of-the-art encryption mechanisms. Today, hardware and firmware support for Confidential Computing is only available in several modern lines of hardware platforms offered by a limited number of forward-looking Cloud Service Providers (CSPs).

## Supported Hardware

Currently, Confidential Computing is only available on a limited set of microprocessor product lines available in the market.
Enterprise vendors offer several Confidential Computing implementations, with varying capabilities and performance limitations, and security trade-offs ([Read more](https://www.canarybit.eu/comparing-confidential-computing-platforms/)).

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
<summary>Upcoming</summary>
<br>- Intel® Trusted Domain Extensions (TDX) - pending hardware availability
<br><a href="https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html" target="_blank">https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html</a>
<br>&nbsp;
<br>- ARM® Confidential Compute Architecture (CCA) - pending hardware availability and firmware support
<br><a href="https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture" target="_blank">https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture</a>
<br>&nbsp;
</details>

<details closed>
<summary>Decomissioned</summary>
<br>- Intel® Software Guard Extension (SGX)
<br><a href="https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html" target="_blank">https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html</a>
<br>&nbsp;
</details>

## Cloud Service Providers (CSPs)

Currently and as a default, Confidential Cloud provisions execution environments on the following CSPs:

### MS Azure

- **Europe** (🇸🇪 Stockholm) : _default_

### OVH (Bare-metal)

- **Europe** (🇩🇪 Frankfurt) : _default_

### OpenStack-based

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

## On-Prem / Air-gapped

Confidential Cloud can be configured to deploy resouces On-Prem. This allows to support use cases that require operations in a closely controlled domain - or even as an **air-gapped** setup.

[Get in touch](https://www.canarybit.eu/contact/) for more information!
