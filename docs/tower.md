
# CanaryBit Tower

*Orchestrate Trusted Execution Environments (TEE).*

## Requirements

- [Terraform](https://developer.hashicorp.com/terraform) or [OpenTofu](https://opentofu.org/docs/) installed

- Credentials to access your cloud platform (e.g Azure, VMware and others)

## Public Clouds

The below configurations are free to use under the Apache-2.0 licence.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   :material-microsoft-azure:{ .lg .middle } __Azure__

    ---

    Infrastructure as Code (IaC) configuration for Confidential VM deployments in **Azure**
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main/azure/)

-   :material-aws:{ .lg .middle } __AWS__

    ---

    Infrastructure as Code (IaC) configuration for Confidential VM deployments in **AWS**
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main/aws/)

-   :material-aws:{ .lg .middle } __GCP__

    ---

    Infrastructure as Code (IaC) configuration for Confidential VM deployments in **GCP**
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main/gcp/)

</div>


## On-Premise 

A Premium License is required for the following configurations.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   :simple-vmware:{ .lg .middle } __VMware__

    ---

    Infrastructure as Code (IaC) configuration for Confidential VM deployments in **VMware**

    [:material-diamond-stone:{ .lg .middle }  Contact us! ](https://www.canarybit.eu/contact)

-   :simple-proxmox:{ .lg .middle } __Proxmox__

    ---

    Infrastructure as Code (IaC) configuration for Confidential VM deployments in **Proxmox** 

    [:material-diamond-stone:{ .lg .middle }  Contact us! ](https://www.canarybit.eu/contact)

-   :simple-redhatopenshift:{ .lg .middle } __Openshift__

    ---

    Infrastructure as Code (IaC) configuration for Confidential VM deployments in **Openshift**

    [:material-diamond-stone:{ .lg .middle }  Contact us! ](https://www.canarybit.eu/contact)

-   :simple-openstack:{ .lg .middle } __Libvirt/KVM__

    ---

    Infrastructure as Code (IaC) configuration for **Baremetal**

    [:material-diamond-stone:{ .lg .middle }  Contact us! ](https://www.canarybit.eu/contact)

</div>

## Configuration Workflow 

The provided configuration performs the following steps on the selected cloud platform:

1. it deploys the Confidential VMs and the required additional virtual resources (e.g. networks, security-groups, etc...)

2. it applies security best-practices to ensure Confidential VM configuration is robust (e.g. disable password login, perform OS updates, open only the specified ports and more) and prevents users to run with `root` privileges.

### Add-on: Remote Attestation

The configuration can be extended by adding [CanaryBit Inspector](inspector.md) Remote Attestation service to verify the actual deployement of the Confidential VMs on the selected cloud platform, in order to guarantee the highest level of security of your execution environment. Specifically:

1. it creates a Manifest file containing execution details such as: 
   
    - the hardware environment to verify (e.g. AMD SNP, Intel TDX, NVIDIA H100, etc...),
    - the custom policy rules;

2. it downloads and runs the CanaryBit Inspector-*agent* (`cbclient`) which

    - collects hardware attestation reports (CPU, GPU and Network),
    - collects OS environment configuration;
    
    by providing the following information:
    
    - the user's authentication token,
    - the Manifest file,
    - a custom Policy file (optional).
   
3. it shut-downs/crashes the Confidential VM according to selected verification exit policy.