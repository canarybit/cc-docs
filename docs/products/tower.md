# CanaryBit Tower

*Orchestrate Trusted Execution Environments (TEE)*

---

**CanaryBit Tower** is a Confidential Computing resources orchestration service. It helps end-users to deploy secure processing environments and provides control over configuration drifts.

It creates all the required infrastructure resources and one or more Trusted Execution Environments (TEEs, see [Confidential Computing](https://www.canarybit.eu/what-is-confidential-computing-and-why-should-i-care/)). 
Tower orchestrates TEEs either on the resources of a Cloud Service Provider of your choice, or On-Prem.
It destroys all the resources once the execution is completed or compromised. Each trusted execution environment is single-use and immutable once created.

CanaryBit Tower consists of multiple Terraform / OpenTofu **module** configurations. The Tower module is a collection of resources to provision Confidential VMs resources only.

!!! tip "What is a module?" 

    To learn more about the module concepts and how to use it refer to the official [Terraform](https://developer.hashicorp.com/terraform/language/modules) or [OpenTofu](https://opentofu.org/docs/language/modules/) documentation.

## Requirements


- A [CanaryBit Account](https://auth.confidentialcloud.io/signup?client_id=54g4h9tpulnnkmhivgn5nipjki&response_type=code&scope=email+openid+profile&redirect_uri=https%3A%2F%2Fdocs.confidentialcloud.io%2F);

- Access to the target infrastructure provider;

- [Terraform](https://developer.hashicorp.com/terraform) or [OpenTofu](https://opentofu.org/docs/) installed;

- CLI access to your cloud platform (e.g [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/what-is-azure-cli?view=azure-cli-latest), [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html), etc...).

## Download

Below a list of configuration for both **Public Clouds** and **Bare-metal / Private setups** supported by CanaryBit Tower.

### Public Clouds

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

## Deploy & Verify

Automatically deploy Confidential VMs (cVM) applying specific configuration for the selected target infrastructure as follows:

### Credentials 

1. Source your **CanaryBit** credentials:

    ``` title="cb.rc"
    export CB_USERNAME=***
    export CB_PASSWORD=***
    ```

    !!! tip
        
        The Terraform/OpenTofu module expects your CanaryBit username (`cb_username`) and password (`cb_password`) as input. <br>
        We recommend using environment variables as follows:
        
        ```
        export TF_VAR_cb_username=$CB_USERNAME  
        export TF_VAR_cb_password=$CB_PASSWORD
        ```

2. Source your target infrastructure credentials (e.g. AWS)

    ``` title="aws.rc"
    export AWS_ACCESS_KEY_ID=***
    export AWS_SECRET_ACCESS_KEY=***
    export AWS_REGION=***

    ```

### Configure

Edit the CanaryBit Tower module configuration. 

The below example shows an example of **the lines/code-blocks that can be customized** according to the expected target environment. 

!!! note
    Always refer to the provided [example files](https://github.com/canarybit/terraform-canarybit-tower/tree/main/examples) for up-to-date, supported configuration.


``` title="Example - Module configuration for AWS deployment"
... 

// ========================
//  Confidential VM (CVM)
// ========================

module "confidential-vm" {

    //  ************** DO NOT REMOVE THESE LINES ****************** // 

    source = "canarybit/tower/canarybit//modules/aws"       
    cb_username = var.cb_username
    cb_password = var.cb_password

    //  ************** CUSTOM CONFIG BELOW THIS LINE ************** // 

    // Confidential VM
    count = 2
    cvm_name = "my-cvm-${count.index}"
    ...
}

...
```
   
For more information about the expected arguments, please refer to each module **Inputs** tab (e.g. [AWS: Inputs](https://registry.terraform.io/modules/canarybit/tower/canarybit/latest/submodules/aws?tab=inputs))

#### With Attestation (recommended)

To enable Remote Attestation, simply add the `remote_attestation` code-block in the module providing the required variables.

In this scenario, CanaryBit Tower will use a specific *cloud-init* file ([`attested.yml`](https://github.com/canarybit/terraform-canarybit-tower/blob/main/cloud-init/attested.yml))  ensuring the security characteristics of each Confidential VM are verified at booting time or at a custom cadence.

!!! success "Never trust, always verify!"

    Environment verification with CanaryBit Inspector service is recommended to certify the security capabilities of the execution environments, **mitigate risks and ensure privacy**.


#### Without Attestation

Whenever a Confidential VM is created and **not attested**, the end-user is still trusting the hypervisor and infrastructure provider.
In this scenario, the need of Confidential VMs becomes **worthless**.

!!! danger "You are at risk!"
    
    The security characteristics of this environment are NOT VERIFIED! </br>
    In this scenario, you are still trusting the hypervisor/infrastructure provider.


### Apply 

In your terminal, apply the configuration via:

``` title="Terraform"
terraform init
terraform apply
```
or 

``` title="OpenTofu"
tofu init
tofu apply
```

During the deployment CanaryBit Tower:

1. authenticates the user towards CanaryBit;
2. authenticates the user towards your infrastructure provider;
3. creates the Confidential VMs and required virtual resources, e.g. networks, security groups, etc...;
4. injects the `cloud-init`(with or without Remote Attestation enabled) in each deployed Confidential VMs;
5. returns the details of the provisioned resources.

### Collect the Attestation report 

Login to the [Inspector dashboard](https://dashboard.inspector.confidentialcloud.io) with your CanaryBit and download the full verification and audit report.