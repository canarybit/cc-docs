# Infrastructure

Confidential Cloud is **cloud-agnostic**. [Tower](architecture.md#tower) and [Inspector](architecture.md#inspector) are responsible for various aspects of the Life-Cycle Management (LCM) of infrastructure resources.

Confidential Cloud builds atop Confidential Computing to offer state-of-the-art encryption mechanisms.
Today, hardware and firmware support for Confidential Computing is widely available in modern lines of hardware platforms offered by forward-looking Cloud Service Providers (CSPs).

## Supported Hardware

Support for Confidential Computing widely available on microprocessor product lines available on the market.
Enterprise vendors offer several Confidential Computing implementations, with varying capabilities and performance limitations, and security trade-offs ([Read more](https://www.canarybit.eu/comparing-confidential-computing-platforms/)).

Confidential Cloud currently supports the following hardware:

### AMD SEV-SNP
- Secure Encrypted Virtualization (SEV) - Secure Nested Paging (SNP): [read more](https://www.amd.com/en/developer/sev.html)

### NVIDIA Confidential Computing
- Support for confidential computing on NVIDIA Hopper and Blackwell GPUs: [read more](https://www.nvidia.com/en-us/data-center/solutions/confidential-computing/) 

<!--
<details closed>
<summary>Coming in 2024</summary>
<br>ARM Confidential Compute Architecture (CCA).
<br>&nbsp;
<br>IBM Protected Execution Facility (PEF).
<br>&nbsp;
<br>Intel® Trust Domain Extensions (TDX).
<br>&nbsp;
</details>
-->

<details closed>
<summary>Preview</summary>
<br>- Intel® Trusted Domain Extensions (TDX) - General Availability pending wider hardware availability
<br><a href="https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html" target="_blank">https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html</a>
<br>&nbsp;
<br>- ARM® Confidential Compute Architecture (CCA) - General Availability pending wider hardware availability
<br><a href="https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture" target="_blank">https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture</a>
<br>
<br>Interested in joining the preview programme? <a href="https://www.canarybit.eu/contact/">Reach out to us</a>!
<br>&nbsp;
</details>

<details closed>
<summary>Decommissioned</summary>
<br>- Intel® Software Guard Extension (SGX)
<br><a href="https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html" target="_blank">https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html</a>
<br>&nbsp;
</details>

## Cloud Service Providers (CSPs)

Confidential Cloud provisions Trusted Execution Environments (TEE) on several hyperscalers and locations.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

  -   :material-aws:{ .lg .middle } __AWS__

    ---

    **Europe** (🇸🇪 Stockholm) : _default_
        

  -   :material-microsoft-azure:{ .lg .middle } __Azure__

    ---

    **Europe** (🇸🇪 Stockholm) : _default_
        
  -   :simple-ovh:{ .lg .middle } __OVH (Bare-Metal)__

    ---

    **Europe** (🇩🇪 Frankfurt) : _default_
    
  -   :simple-openstack:{ .lg .middle } __Openstack__

    ---

    **Europe** (🇸🇪 Stockholm) : _default_

    **Europe** (🇮🇹 Italy)
    
</div>

<br>

<details open>
<summary>Partnership Programme</summary>
<br>Are you a CSP, hardware vendor or software solutions provider and interested to support Confidential Cloud?
<br>Join our <a href="https://www.canarybit.eu/become-a-partner/">Partnership programme</a>!
<br>&nbsp;
</details>

## On-Prem / Air-gapped

Confidential Cloud can be configured to deploy resouces On-Prem. This allows to support use cases that require operations in a closely controlled domain - or even as an **air-gapped** setup.

[Get in touch](https://www.canarybit.eu/contact/) for more information!
