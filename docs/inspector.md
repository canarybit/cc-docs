# CanaryBit Inspector

Verify Trusted Execution Environments (TEE) characteristics.

CanaryBit [Inspector](https://docs.confidentialcloud.io/architecture/#inspector) is a Confidential Computing [Remote Attestation (RA)]() service for Confidential VMs (cVM) and containers deployed in Public or Private cloud infrastructures.

It helps end-users verify the security level of the processing environments and ensure user-defined policies are satisfied before any sensitive data analysis is performed.

## Prerequisites

- A [CanaryBit account](https://demo.confidentialcloud.io/signup).
- If you are using automatic deployments with [CC Tower](tower.md), install [Terraform]() or [OpenTofu]().

## Confidential VMs

There are multiple ways to verify Confidential VMs using CC Inspector:

1. Automatically with [CC Tower](tower.md) at deployment time
2. Manually via the Public Cloud Provider Dashboard/CLI at deployment time
3. On already existing Confidential VMs

### 1. CC Tower - Automatic

A Confidential VM is **deployed & attested** using the Terraform/OpenTofu templates provided by the [CC Tower](./tower.md) service.

### 2. Via the CSP Dashboard/CLI - Manual

A Confidential VM is deployed following the steps defined by the Public Cloud provider (e.g. [Azure](https://learn.microsoft.com/en-us/azure/confidential-computing/quick-create-confidential-vm-portal), [AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sev-snp.html), [GCP](https://cloud.google.com/confidential-computing/confidential-vm/docs/create-a-confidential-vm-instance))

Before launch, the Confidential VM is instructed to perform the following additional steps to enable CC Inspector Remote Attestation:

1. Perform [user authentication]() towards CanaryBit's Identity Service Provider (IdP)

2. Create a configuration file containing execution details such as: 
   1. the target hardware environment to verify (e.g. AMD SNP, Intel TDX, NVIDIA H100)
   2. a reference to the cloud provider or baremetal
   3. the custom policy rules
   4. ... and more!

3. Download the required `cbclient` for basic attestation.

   Additionally: download and run the `cbfinder` for custom-policy checking.
   
4. Launch the `cbclient` in **attestation** mode, providing the following information:
   1. the user's authentication token
   2. the configuration file
   3. Additionally: the Confidential VM related information for custom-policy checking
5. Terminate the Confidential VM if either the attestation process is not successful or the provided custom-policies are not satisfied.

### 3. Already existing Confidential VMs - Custom

Similar to the previous approach, the Confidential VM is instructed to perform the required steps to enable CC Inspector Remote Attestation.

## Confidential Containers 

CC Inspector ensures your container runs always on a Confidential Node. It gives the container the responsibility to perform a remote attestation before it is executed. This is **extremely beneficial** when using a Container as a Service (CaaS) solution where `cluster autoscaling` is typically the recommended option.


