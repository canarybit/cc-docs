# Deploy a secure infrastructure with Tower

[Tower](https://docs.confidentialcloud.io/architecture/#tower) is CanaryBit's **orchestration** service to automate the deployment of Trusted Execution Environments (TEEs) infrastructure resources.

It strictly follows Infrastructure-as-Code (IaC) and cloud security best practices, putting end users firmly in control of their setups. It offers a set of open-source [Terraform](https://terraform.io/) configurations to help you manage and provision higly secure runtime environments on a wide set of public cloud providers, on-prem or bare-metal setups.

## Download the code 

**Note.** These links lead to the private `cc-tower` repository visible only to Confidential Cloud subscribers.

<div class="grid cards" markdown>
<!-- https://squidfunk.github.io/mkdocs-material/reference/grids/#using-card-grids !-->

-   :material-microsoft-azure:{ .lg .middle } __Azure__

    ---

    Terraform configuration for Trusted Execution Environments (TEEs) deployments in **Azure**
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub](https://github.com/canarybit/cc-tower/tree/main/azure/)

-   :material-aws:{ .lg .middle } __AWS__

    ---

    Terraform configuration for Trusted Execution Environments (TEEs) deployments in **AWS**
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub](https://github.com/canarybit/cc-tower/tree/main/aws/)

-   :material-aws:{ .lg .middle } __GCP__

    ---

    Terraform configuration for Trusted Execution Environments (TEEs) deployments in **GCP**
        
    [:octicons-link-external-16:{ .lg .middle }  GitHub](https://github.com/canarybit/cc-tower/tree/main/gcp/)

-   :simple-openstack:{ .lg .middle } __Openstack__

    ---

    Terraform configuration for Trusted Execution Environments (TEEs) deployments in **Openstack**

    [:octicons-link-external-16:{ .lg .middle }  Available soon!](https://github.com/canarybit/cc-tower)
    
-   :material-server:{ .lg .middle } __Baremetal__

    ---

    Terraform configuration for Trusted Execution Environments (TEEs) deployments for **Baremetal** setups with `libvirt`.

    [:octicons-link-external-16:{ .lg .middle }  Available soon!](https://github.com/canarybit/cc-tower)

-   :simple-redhatopenshift:{ .lg .middle } __Openshift__

    ---

    Terraform configuration for Trusted Execution Environments (TEEs) deployments in **Openshift**

    [:octicons-link-external-16:{ .lg .middle }  Available soon!](https://github.com/canarybit/cc-tower)

</div>

## Resources Deployment and Attestation

**Note:** Before provisioning the resources, [user authentication]() is required towards CanaryBit's Identity Service Provider (IdP).

During the deployment, the Infrastructure as Code configuration will perform the following activities:

1. It deploys the specified amount of Confidential VMs and required virtual resources such as Networks, Security Groups, and more.
2. It applies security best-practices to ensure the Confidential VM is robust and the defined user DOES NOT run with `root` privileges.
3. It creates a configuration file containing execution details such as: 
   1. the hardware environment to attest (e.g.AMD SNP, Intel TDX, NVIDIA H100)
   2. the cloud provider
   3. the custom policy rules
   4. ... and more!
4. It downloads the required `cbclient` and additional binaries, e.g. `cbfinder` used for custom-policy checking
5. It launches the `cbclient` in **attestation** mode, providing the following information:
   1. the user's authentication token
   2. the configuration file
   3. the Confidential VM information for custom-policy checking
6. [CC Inspector]() verifies the environment according to the custom policies.