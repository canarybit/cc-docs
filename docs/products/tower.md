# CanaryBit Tower

*Orchestrate Trusted Execution Environments (TEE).*

CanaryBit [Tower](https://docs.confidentialcloud.io/architecture/#inspector) is a Confidential Computing resources orchestration service. It helps end-users to deploy secure processing environments and provides control over configuration drifts.

It creates all the required infrastructure resources and one or more Trusted Execution Environments (TEEs, see [Confidential Computing](https://www.canarybit.eu/what-is-confidential-computing-and-why-should-i-care/)). 
Tower orchestrates TEEs either on the resources of a Cloud Service Provider of your choice, or On-Prem (see [Infrastructure]()).
It destroys all the resources once the execution is completed or compromised. Each trusted execution environment is single-use and immutable once created.

## Requirements

- A CanaryBit Account, if [Attestation](#confidential-computing-attestation) is required;

- [Terraform](https://developer.hashicorp.com/terraform) or [OpenTofu](https://opentofu.org/docs/) installed;

- CLI access to your cloud platform (e.g [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/what-is-azure-cli?view=azure-cli-latest), [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html), etc...)

## Deployment types

CanaryBit Tower configurations are flexible enough to deploy Confidential VMs with or without CanaryBit Inspector Attestation service.

**Environment verification with CanaryBit Inspector service is recommended to certify the security capabilities of the execution environments, mitigate risks and ensure privacy**.

### 1. With Attestation (recommended)

In this scenario, the provided configuration performs the following steps on the selected infrastructure provider:

1. Deploy Confidential VMs and required virtual resources (e.g. networks, security groups, etc...)
2. Inject a `cloud-init` file at booting time ensuring robust Confidential VMs configuration. Specifically: 
      1. Apply cloud security best-practices:
         1. Create a new user
         2. Disable password login
         3. Assign the new user a new group without wide `root` permissions.
      2. Download & run the [CanaryBit Inspector client](inspector.md) (`cbclient`) applying (optional) Custom Policies; 
      3. Return the final **CanaryBit report**. Logs and reports will be available for **external logging & auditing** activities; 
3. Return details about the configured resources;
4. ✳️ **The security of the environment is verified!** The final report(s) can be collected on the CanaryBit Inspector dashboard;

### 2. Without Attestation

In this scenario, the provided configuration performs the following steps on the selected infrastructure provider:

1. deploy Confidential VMs and required virtual resources (e.g. networks);
2. apply cloud security best-practices (e.g. password login disabled);
3. inject a `cloud-init` file at booting time ensuring the Confidential VM configuration is robust e.g. a separate user is used (e.g. `cbuser`), OS and packages are updated:  
        ```
        #cloud-config
        users:
        - default
        - name: cbuser
            sudo: false
            shell: /bin/bash
            ssh_authorized_keys:
            - ssh-rsa <CBUSER_PUB_KEY>

        timezone: UTC
        locale: "en_US.UTF-8"

        package_update: true
        package_upgrade: true
        package_reboot_if_required: true
        ```
4. Return details about the configured resources;
5. ⚠️ **The security characteristics of this environment are not verified!** In this scenario, you are still trusting the hypervisor/infrastructure provider.

## Download & Run

### On Public Clouds

The below configurations are **free to use** under the Apache-2.0 licence.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   #### :material-microsoft-azure:{ .lg .middle } __Azure__

    ---

    Infrastructure as Code (IaC) configuration for **Azure** Confidential VMs
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main)

-   #### :material-aws:{ .lg .middle } __AWS__

    ---

    Infrastructure as Code (IaC) configuration for **AWS** Confidential VMs
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main)

-   #### :material-aws:{ .lg .middle } __GCP__

    ---

    Infrastructure as Code (IaC) configuration for in **GCP** Confidential VMs
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main)

</div>


### On-Premise / Bare-metal

A **Premium License** is required for the following configurations: [:material-diamond-stone:{ .lg .middle } Buy Premium :octicons-link-external-16:{ .lg .middle }](https://www.canarybit.eu/contact)

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