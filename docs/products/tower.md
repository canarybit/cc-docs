# CanaryBit Tower

*Orchestrate Trusted Execution Environments (TEE)*

---

**CanaryBit Tower** is a Confidential Computing resources orchestration service. It helps end-users to deploy secure processing environments and provides control over configuration drifts.

It creates all the required infrastructure resources and one or more Trusted Execution Environments (TEEs, see [Confidential Computing](https://www.canarybit.eu/what-is-confidential-computing-and-why-should-i-care/)). 
Tower orchestrates TEEs either on the resources of a Cloud Service Provider of your choice, or On-Prem.
It destroys all the resources once the execution is completed or compromised. Each trusted execution environment is single-use and immutable once created.

## Requirements

- A CanaryBit Account

- [Terraform](https://developer.hashicorp.com/terraform) or [OpenTofu](https://opentofu.org/docs/) installed;

- CLI access to your cloud platform (e.g [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/what-is-azure-cli?view=azure-cli-latest), [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html), etc...)

## Deployment

CanaryBit Tower supports multiple configurations to deploy Confidential VMs with or without CanaryBit Inspector Attestation service.

!!! info "NEVER TRUST, ALWAYS VERIFY"

    Environment verification with CanaryBit Inspector service is recommended to certify the security capabilities of the execution environments, mitigate risks and ensure privacy**.

### 1. With Attestation (recommended)

In this scenario, the provided configuration performs the following steps on the selected infrastructure provider:

1. Deploy Confidential VMs and required virtual resources (e.g. networks, security groups, etc...)
2. Inject the [`attested.yml`](https://github.com/canarybit/terraform-canarybit-tower/blob/main/cloud-init/attested.yml) *cloud-init* file at booting time ensuring robust Confidential VMs configuration and **security verification** of the deployed setup.
3. Return details about the configured resources;

!!! success 

    The security characteristics of this environment are VERIFIED! </br>
    In this scenario, you are only trusting the chip vendor.
    The full report can be downloaded from the CanaryBit [Inspector Dashbord.](inspector.md)

### 2. Without Attestation

In this scenario, the provided configuration performs the following steps on the selected infrastructure provider:

1. Deploy one or more Confidential VMs and required virtual resources (e.g. networks, security groups, etc...);
2. Apply cloud security best-practices (e.g. disable password login);
3. Inject the [`default.yml`](https://github.com/canarybit/terraform-canarybit-tower/blob/main/cloud-init/default.yml) *cloud-init* file at booting time ensuring robust Confidential VM configuration.
4. Return details about the Confidential VM and other configured resource.

!!! danger 
    
    The security characteristics of this environment are NOT VERIFIED! </br>
    In this scenario, you are still trusting the hypervisor/infrastructure provider.

## Download & Run

### On Public Clouds

The below configurations are **free to use** under the Apache-2.0 licence.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   #### :material-microsoft-azure:{ .lg .middle } __Azure__

    ---

    Infrastructure as Code (IaC) configuration for **Azure** Confidential VMs
        
    [:simple-terraform:{ .lg .middle }  Terraform ](https://registry.terraform.io/modules/canarybit/tower/canarybit/latest/submodules/azure) &nbsp; | &nbsp; 
    [:simple-opentofu:{ .lg .middle }  OpenTofu ](https://search.opentofu.org/module/canarybit/tower/canarybit/latest/submodule/azure)

-   #### :material-aws:{ .lg .middle } __AWS__

    ---

    Infrastructure as Code (IaC) configuration for **AWS** Confidential VMs
        
    [:simple-terraform:{ .lg .middle }  Terraform ](https://registry.terraform.io/modules/canarybit/tower/canarybit/latest/submodules/aws) &nbsp; | &nbsp; 
    [:simple-opentofu:{ .lg .middle }  OpenTofu ](https://search.opentofu.org/module/canarybit/tower/canarybit/latest/submodule/aws)
    
-   #### :material-google-cloud:{ .lg .middle } __GCP__

    ---

    Infrastructure as Code (IaC) configuration for in **GCP** Confidential VMs
        
    [:simple-terraform:{ .lg .middle }  Terraform ](https://registry.terraform.io/modules/canarybit/tower/canarybit/latest/submodules/gcp) &nbsp; | &nbsp; 
    [:simple-opentofu:{ .lg .middle }  OpenTofu ](https://search.opentofu.org/module/canarybit/tower/canarybit/latest/submodule/gcp)

</div>


### Bare-metal / Private setups

A **Premium License** is required for the following configurations: [:material-diamond-stone:{ .lg .middle } Buy Premium](https://www.canarybit.eu/contact)

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   #### :simple-vmware:{ .lg .middle } __VMware__

    ---

    Infrastructure as Code (IaC) configuration for **VMware** Confidential VMs


-   #### :simple-proxmox:{ .lg .middle } __Proxmox__

    ---

    Infrastructure as Code (IaC) configuration for **Proxmox** Confidential VMs


-   #### :simple-redhatopenshift:{ .lg .middle } __Openshift__

    ---

    Infrastructure as Code (IaC) configuration for **Openshift** Confidential VMs/Nodes


-   #### :simple-openstack:{ .lg .middle } __Libvirt/QEMU__

    ---

    Infrastructure as Code (IaC) configuration for **Baremetal** Confidential VMs


</div>