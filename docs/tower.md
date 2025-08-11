# Orchestrate TEEs with CanaryBit Tower

[CanaryBit Tower](https://docs.confidentialcloud.io/architecture/#tower) is an **orchestration** tool to automate the deployment of Trusted Execution Environments (TEEs).

It strictly follows Infrastructure-as-Code (IaC) and cloud security best practices, putting end users firmly in control of their setups. It offers a set of open-source [Terraform](https://terraform.io/)/[OpenTofu](https://opentofu.io/) configurations to help you manage and provision higly secure runtime environments on a wide set of public cloud providers, on-prem or bare-metal setups.

## On Public Clouds (FREE Licence)

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   :material-microsoft-azure:{ .lg .middle } __Azure__

    ---

    Infrastructure as Code (IaC) configuration for **Azure**
        
    [:octicons-link-external-16:{ .lg .middle }  Open-source ](https://github.com/canarybit/tower/tree/main/azure/)

-   :material-aws:{ .lg .middle } __AWS__

    ---

    Infrastructure as Code (IaC) configuration for **AWS**
        
    [:octicons-link-external-16:{ .lg .middle }  Open-source ](https://github.com/canarybit/tower/tree/main/aws/)

-   :material-aws:{ .lg .middle } __GCP__

    ---

    Infrastructure as Code (IaC) configuration for **GCP**
        
    [:octicons-link-external-16:{ .lg .middle }  Open-source ](https://github.com/canarybit/tower/tree/main/gcp/)

</div>


## On premise (Premium License)

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   :simple-vmware:{ .lg .middle } __VMware__

    ---

    Infrastructure as Code (IaC) configuration for **VMware**

    [:octicons-link-external-16:{ .lg .middle }  Premium License ](https://github.com/canarybit/tower)

-   :simple-proxmox:{ .lg .middle } __Proxmox__

    ---

    Infrastructure as Code (IaC) configuration for **Proxmox** 

    [:octicons-link-external-16:{ .lg .middle }  Premium License ](https://github.com/canarybit/tower)

-   :simple-redhatopenshift:{ .lg .middle } __Openshift__

    ---

    Infrastructure as Code (IaC) configuration for **Openshift**

    [:octicons-link-external-16:{ .lg .middle }  Premium License ](https://github.com/canarybit/tower)

-   :simple-openstack:{ .lg .middle } __Libvirt/KVM__

    ---

    Infrastructure as Code (IaC) configuration for **Baremetal**

    [:octicons-link-external-16:{ .lg .middle }  Premium License ](https://github.com/canarybit/tower)

</div>

## Workflow description

1. Deploy the specified amount of Confidential VMs and required virtual resources such as Networks, Security Groups, and more.

2. Apply security best-practices to ensure the Confidential VM is robust and the defined user DOES NOT run with `root` privileges.

3. Create a Manifest file containing execution details such as: 
   
   i. the hardware environment to attest (e.g.AMD SNP, Intel TDX, NVIDIA H100)
   
   ii. the cloud provider
   
   iii. the custom policy rules

4. Download the required `cbclient` (aka CanaryBit Inspector Agent) and
   
   i. collect hardware attestation reports (CPU, GPU and Network)
   
   ii. collect OS environment configuration

5. Launch the `cbclient` providing the following information:
   
   i. the user's authentication token (required)
   
   ii. a Manifest file (required)
   
   iii. a custom Policy file (optional)
   
6. [CanaryBit Inspector](inspector.md) verifies the environment according to the custom policies.