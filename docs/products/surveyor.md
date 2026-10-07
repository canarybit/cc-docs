# CanaryBit Surveyor

*Safeguard Container execution*

---

CanaryBit Surveyor is a **confidential container launcher** that runs containers and pods only after the underlying infrastructure has been validated. 
It works with Kata Containers or directly on confidential nodes in your Kubernetes cluster running on AMD SEV-SNP and Intel TDX hardware. 
Each workload is remotely attested by [CanaryBit Inspector](./inspector.md) before it starts and then re-verified on a schedule (daily by default), so your data and algorithms stay protected inside a hardware-encrypted execution environment.

Key capabilities:

- **Attestation-gated deployment**: workloads launch only after the infrastructure passes remote attestation.
- **Two deployment modes**: kata (recommended) isolates each pod in its own lightweight VM. node runs pods directly on confidential nodes, which gives security but no isolation between pods.
- **Custom policies**: add your own Rego policies, for example to enforce a kernel version, hypervisor or region, on top of the default verifier policies.
Verification reports: download attestation reports and insights from the Inspector dashboard.

## Requirements

- A CanaryBit [account](https://auth.confidentialcloud.io/signup?client_id=54g4h9tpulnnkmhivgn5nipjki&response_type=code&scope=email+openid+profile&redirect_uri=https%3A%2F%2Fdocs.confidentialcloud.io%2F);
- A CanaryBit [Inspector licence](./inspector.md#licences);
- The CanaryBit [CLI](https://docs.confidentialcloud.io/tools/cli/) (`cb-cli`) installed;
- Access to a Kubernetes cluster (`kubeconfig`) running on a [supported](../requirements.md) hardware platform;
- *Only for `kata` mode:*
    - [Helm](https://helm.sh) installed.

## Source your credentials

Source your CanaryBit credentials (e.g. `cb.rc`):

``` title="cb.rc"
export CB_USERNAME=***
export CB_PASSWORD=***
```

## Download

There are two ways to download CanaryBit Surveyor:

1. Through the CanaryBit [Inspector dashboard](https://dashboard.inspector.confidentialcloud.io)

2. Via the CanaryBit CLI.

    ```commandline
    # View all available versions
    $ cb list surveyor
    
    # Install a specific version and distro
    $ cb download surveyor <VERSION>/<FILENAME>
    ```
       
!!! Example
    ```
    $ cb list surveyor
    0.3.0/surveyor-x86_64-unknown-linux-gnu
    latest

    $ cb download surveyor 0.3.0/surveyor-x86_64-unknown-linux-gnu
    Downloaded 0.3.0/surveyor-x86_64-unknown-linux-gnu to surveyor-x86_64-unknown-linux-gnu
    ```

## Configure

### Install Kata (recommended)

Only for `kata` mode, hardware-specific Kata Containers runtime classes are required before deploying confidential workloads. 

To install the Kata runtime classes, first initialize the configuration with: 

```commandline
$ surveyor kata init
```

!!! info 

      CanaryBit Surveyor uses _node-feature-discovery_ labels to identify the cluster nodes with Confidential Computing capabilities:
      
      ```
      labels = [
        "feature.node.kubernetes.io/cpu-security.sev.snp.enabled",
        "feature.node.kubernetes.io/cpu-security.tdx.enabled",
      ]
      ```

Then, install the hardware-specific kata runtime classes (`kata-qemu-snp` and `kata-qemu-tdx`) with:

```
$ surveyor kata setup
```

### Initialize Surveyor

Configure CanaryBit Surveyor to use the right CanaryBit Inspector and registry endpoint:

```
$ surveyor init
```
 
## Deploy & Attest

### Create new secrets

Surveyor requires two secrets to perform a remote attestation: 

- the `inspector` secret to authenticate to CanaryBit Inspector and refresh the user's access token overtime;
- the `registry` secret to pull the CanaryBit Inspector client (`cb-inspector-client`) container image.

To create new secrets, first login via the CanaryBit CLI and export your authentication token (`CB_TOKENS`):

```
$ export CB_TOKENS=$(cb login)
```

Then, create both `inspector` and `registry` secret on your cluster with :

```
$ surveyor secret create canarybit --namespace default
```

### Add custom policies

It's possible to add a custom policies (e.g. `mypolicy.rego`) at different levels in the technology stack (hardware, hypervisor, OS and more). 
Custom policies will be enforced on top of the verifier defaults policies, and together assess both the security level and correctness of each TEE.

To create a custom policy, simply create a file with a custom Rego policy expression.

!!! Example

    A custom policy to enforce a specific OS kernel version, hypervisor, and region for the deployed TEE.

    ``` title="mypolicy.rego"
    package mypolicy
    
    default allow := false
    
    allow if {
      input.claims.attestations.canarybit.kernel_version == "6.17.0-14-generic"
      input.claims.metadata.hypervisor.cpuid_hypervisor == "HyperV"
      input.claims.metadata.instance.region = "northeurope"
    }
    ```

### Run

Deploy your container (e.g. `mypod.yaml`) via Pod/Deployment manifest:

```
$ surveyor deploy --cc-mode kata --kata-runtime-class kata-qemu-snp --attestation-targets snp --attestation-policy mypolicy.rego mypod.yaml
```

!!! info
    CanaryBit Surveyor will extend the Pod manifest by adding the CanaryBit Inspector client (`cb-inspector-client`) as init container, ensuring the workload is launched only upon a successful attestation and verified by CanaryBit Inspector at a custom schedule (defaults to `daily`)

## Download the reports

The verification reports and additional insights are available for download on the CanaryBit [Inspector dashboard.](https://dashboard.inspector.confidentialcloud.io)

