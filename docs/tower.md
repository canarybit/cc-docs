# CanaryBit Tower

*Orchestrate Trusted Execution Environments (TEE).*

<br>

CanaryBit [Tower](https://docs.confidentialcloud.io/architecture/#inspector) is a Confidential Computing resources orchestration service. It helps end-users to deploy secure processing environments and provides control over configuration drifts.

## Requirements

- A CanaryBit Account, if [Attestation](#confidential-computing-attestation) is required;

- [Terraform](https://developer.hashicorp.com/terraform) or [OpenTofu](https://opentofu.org/docs/) installed;

- CLI access to your cloud platform (e.g [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/what-is-azure-cli?view=azure-cli-latest), [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html), etc...)

## Infrastructure Providers

### Public Clouds

The below configurations are **free to use** under the Apache-2.0 licence.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   :material-microsoft-azure:{ .lg .middle } __Azure__

    ---

    Infrastructure as Code (IaC) configuration for **Azure** Confidential VMs
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main)

-   :material-aws:{ .lg .middle } __AWS__

    ---

    Infrastructure as Code (IaC) configuration for **AWS** Confidential VMs
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main)

-   :material-aws:{ .lg .middle } __GCP__

    ---

    Infrastructure as Code (IaC) configuration for in **GCP** Confidential VMs
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub ](https://github.com/canarybit/tower/tree/main)

</div>


### On-Premise 

A **Premium License** is required for the following configurations.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   :simple-vmware:{ .lg .middle } __VMware__

    ---

    Infrastructure as Code (IaC) configuration for **VMware** Confidential VMs

    [:material-diamond-stone:{ .lg .middle } Buy Premium ](https://www.canarybit.eu/contact)

-   :simple-proxmox:{ .lg .middle } __Proxmox__

    ---

    Infrastructure as Code (IaC) configuration for **Proxmox** Confidential VMs

    [:material-diamond-stone:{ .lg .middle } Buy Premium ](https://www.canarybit.eu/contact)

-   :simple-redhatopenshift:{ .lg .middle } __Openshift__

    ---

    Infrastructure as Code (IaC) configuration for **Openshift** Confidential VMs/Nodes

    [:material-diamond-stone:{ .lg .middle } Buy Premium ](https://www.canarybit.eu/contact)

-   :simple-openstack:{ .lg .middle } __Libvirt/KVM__

    ---

    Infrastructure as Code (IaC) configuration for **Baremetal** Confidential VMs

    [:material-diamond-stone:{ .lg .middle } Buy Premium ](https://www.canarybit.eu/contact)

</div>
   

## Deployment 

CanaryBit Tower can deploy Confidential VMs with and without an additional verification step (recommended).

### Default (unverified)

The provided configuration performs the following steps on the selected infrastructure provider:

1. deploy Confidential VMs and required virtual resources (e.g. networks)
2. apply cloud security best-practices (e.g. password login disabled).
3. inject a `cloud-init` file at booting time ensuring the Confidential VM configuration is robust e.g. a separate user is used (e.g. `cbuser`), OS and packages are updated:

    `cloud-init.yml`:
    
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

*❗ **UNVERIFIED:** The security characteritstic of this environment are not verified! 
**You are still trusting the hypervisor/infrastructure provider.**
For full-privacy and highest security level, verify the environment with CanaryBit Attestation service.*

### Attested (verified)

CanaryBit Attestation service can be easily added as part of the CanaryBit Tower configuration to verify the security characteristics of a Confidential VM, once the resource is correclty deployed on the target infrastructure provider.

In addition to the standard configuration, the `cloud-init` file will:
   
1. Create a `canarybit` user-group and assign it to the user (e.g. `cbuser`), limiting its `root` permissions;
2. Collect information about the current execution environment;
3. Download & run the [CanaryBit Inspector client](inspector.md) (`cbclient`) applying the (optional) Custom Policies; 
4. Return the final **CanaryBit report**. Logs and reports are available for **external logging & auditing** activities.

✅ **VERIFIED**: The security of this **environment has been verified** by CanaryBit Inspector Attestation service and the final report(s) can be collected for auditing activities.