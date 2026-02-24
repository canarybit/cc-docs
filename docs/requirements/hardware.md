# Hardware

Today, support for Confidential Computing is widely available on microprocessor product lines available on the market.

Enterprise vendors offer several Confidential Computing [implementations](https://www.canarybit.eu/comparing-confidential-computing-platforms/), with varying capabilities and performance limitations, and security trade-offs.

Confidential Cloud currently supports the following hardware: 

## CPU

<div class="grid cards" markdown>
  <!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->
  
  -   :simple-amd:{ .lg .middle } __SEV-SNP__

      ---

      *AMD Secure Encrypted Virtualization (SEV) - Secure Nested Paging (SNP)*

      Confidential Computing capability is available from **3rd Gen AMD `EPYC` Processors** (FamilyName: *Milan*) and newer.

      Read the [official documentation.](https://www.amd.com/en/developer/sev.html)
  
  -   :simple-intel:{ .lg .middle } __TDX__

      ---

      *Intel Trust Domain Extensions (TDX)* <br><br>

      Confidential Computing capability is available from **5th Gen Intel `Xeon` Scalable Processors** (FamilyName: *Emerald Rapids*) and newer.

      Read the [official documentation.](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html)

</div>

## GPU


<div class="grid cards" markdown>
  <!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->
  
  -   :simple-nvidia:{ .lg .middle } __NVIDIA__

      ---

      Confidential Computing capability is available from **H100 `Tensor Core` Processors** (FamilyName: *Hopper*) and newer.

      Read the [official documentation.](https://www.nvidia.com/en-us/data-center/solutions/confidential-computing/)

</div>

## SmartNIC 

:warning: Coming soon:

<div class="grid cards" markdown>
  <!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->
  
  -   :simple-amd:{ .lg .middle } __SEV-TIO__

      ---

      Read the [official documentation.](https://www.amd.com/content/dam/amd/en/documents/developer/sev-tio-whitepaper.pdf)

-   :simple-intel:{ .lg .middle } __TEE-IO__

      ---

      Read the [official documentation.](https://www.intel.com/content/www/us/en/content-details/772642/intel-tdx-connect-tee-io-device-guide.html)
</div>

