# CanaryBit Surveyor

*Safeguard Container execution*

---

CanaryBit Surveyor is a **Confidential Container launcher**. It helps end-users to run containar/pods only upon validation of the underlying infrastructure, running under Kata Containers (AMD SEV-SNP, Intel TDX) or on confidential nodes directly.

It guarantees confidentiality and privacy allowing end-users to select between two modes:

   1. `kata` **(recommended)**: each container/pod is hypervisor isolated inside a lightweight VM - known as [Kata Containers](https://katacontainers.io/) - and remotely attested by CanaryBit Inspector. This mode guarantees security and isolation.
   2. `node`: each container/pod runs on confidential nodes directly, and remotely attested by CanaryBit Inspector. This mode guarantees security but no isolation between containers/pods.

!!! Info 
      Kata containers gives stronger isolation but running a container/pod as a kata container means launching a nested VM dedicated to the container/pod. Confidential Computing does not currently support nested virtualization.

## Requirements

- A CanaryBit [account](https://auth.confidentialcloud.io/signup?client_id=54g4h9tpulnnkmhivgn5nipjki&response_type=code&scope=email+openid+profile&redirect_uri=https%3A%2F%2Fdocs.confidentialcloud.io%2F);
- A CanaryBit [Inspector licence](./inspector.md#licences);
- The CanaryBit [CLI](https://docs.confidentialcloud.io/tools/cli/) (`cb-cli`) installed;
- Access to a Kubernetes cluster (`kubeconfig`) running on a [supported](../requirements.md) hardware platform;
- [Helm](https://helm.sh) - the package manager for Kubernetes - installed.

## Source your credentials

Source your CanaryBit credentials (e.g. `cb.rc`):

``` title="cb.rc"
export CB_USERNAME=***
export CB_PASSWORD=***
```

## Download

There are two ways to download CanaryBit Surveyor:

1. On the CanaryBit Inspector [Dashboard](https://dashboard.inspector.confidentialcloud.io)

2. Via the CanaryBit CLI.

    ```commandline
    # Install latest
    $ cb download surveyor latest
    
    # Install a specific version
    $ cb download surveyor 0.2.0
    
    # Install a specific version and distro
    $ cb download surveyor 0.2.0/surveyor-x86_64-unknown-linux-gnu
    ```


## Configure

### Install Kata

In `kata` mode, hardware-specific Kata Containers runtime classes are required before deploying confidential workloads. 

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
- the `registry` secret to pull the CanaryBit Inspector client (`cbclient`) container image.

To create the new secrets, first login via the CanaryBit CLI and export your authentication token (`CB_TOKENS`):

```
$ export CB_TOKENS=$(cb login)
```

Then, create the `inspector` secret in your cluster with :

```
$ surveyor secret create inspector \
    --cb-tokens "$CB_TOKENS" \
    --namespace default \
```

Finally, create the `registry` secret with:

```
$ surveyor secret create registry \
    --password $(cb login registry cbclient | base64 -d | cut -f 2 -d :) \
    --namespace default \ 
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
$ surveyor deploy \
  --cc-mode kata \
  --kata-runtime-class kata-qemu-snp \
  --attestation-targets snp \
  --attestation-policy mypolicy.rego (if specified)
  mypod.yaml
```

!!! info
    CanaryBit Surveyor will extend the Pod manifest by adding the CanaryBit Inspector client (`cbclient`) as init container, ensuring the workload is launched only upon a successful attestation and verified by CanaryBit Inspector at a custom schedule (defaults to `daily`)

## Download the reports

The verification reports and additional insights are available for download on the [CanaryBit Inspector Dashboard.](https://dashboard.inspector.confidentialcloud.io)

