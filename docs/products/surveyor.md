# CanaryBit Surveyor

*Safeguard Container execution*

---

For containerized application that analyze sensitive data, it became crucial to verify the confidentiality of the nodes where the workloads are running, no matter if they are managed by an orchestration service (e.g. Azure AKS or AWS EKS) or on a container platform directly (Vanilla Kubernetes, Redhat Openshift, Suse Harvester, etc...).

CanaryBit Surveyor ensures the containerized applications enforce isolation and zero-trust mechanisms to protect the privacy and confidentiality of both the workload and datasets, ensuring the cluster characteristics are verified:

* before a container is executed (always);

* at a defined schedule (if enabled);

* when specific system-wide events occur (if enabled).

This approach is extremely beneficial for dynamic clusters where the size of the Kubernetes cluster (# of Nodes) is adjusted based on the utilization of Pods and Nodes in the cluster. This feature, called `autoscaling` is typically recommended by hyperscalers CaaS solutions.

## Architecture

CanaryBit Surveyor guarantees confidentiality allowing the end-user to select two modes:
- `node` for Security: the entire Kubernetes node is confidential. This approach is implemented for less stringent requirements or when nested virtualization is unavoidable;  
- `kata` for Security and Isolation **(recommended)**: each workload runs inside a lightweight VM, provided by [Kata Containers](https://katacontainers.io/), giving pod-level isolation. Surveyor configures the right Kata Containers runtime based on the type of the confidential node (AMD SNP or Intel TDX).

**Note:** Kata gives stronger isolation but running a Kata Container means starting a nested VM. Confidential Computing does not currently support nested virtualization.

CanaryBit Surveyor uses the CanaryBit user authentication token (`CB_TOKEN`) to both authenticate the user to the CanaryBit Inspector service and to pull the `inspector-client` container image from the CanaryBit Registry.

## Requirements

- A [CanaryBit Account](https://auth.confidentialcloud.io/signup?client_id=54g4h9tpulnnkmhivgn5nipjki&response_type=code&scope=email+openid+profile&redirect_uri=https%3A%2F%2Fdocs.confidentialcloud.io%2F);
- The [CanaryBit CLI](https://docs.confidentialcloud.io/tools/cli/) `cb-cli` installed
- Access to a Kubernetes cluster (`kubeconfig`) running on a [supported](../requirements.md) hardware platform.
- [Helm](https://helm.sh) - the package manager for Kubernetes - installed.

## Download CanaryBit Surveyor

There two ways to download CanaryBit Surveyor:

1. via the CanaryBit CLI:
    ```commandline
    # Install the latest version with CanaryBit CLI
    $ cb download surveyor "latest"
    
    # Or install specific version
    $ cb download surveyor 0.2.0/surveyor-x86_64-unknown-linux-gnu
    ```

2. Directly on the [CanaryBit Inspector Dashboard.](https://dashboard.inspector.confidentialcloud.io)
 
## Attest a Confidential Container or Pod

### Install the Kata runtime

CanaryBit Surveyor needs the Kata Containers runtime installed before it can deploy confidential workloads. Generate a starting configuration:

```commandline
$ surveyor kata init
```

CanaryBit Surveyor uses node-feature-discovery labels to identify the cluster nodes with Confidential Computing capabilities:

```
labels = [
  "feature.node.kubernetes.io/cpu-security.sev.snp.enabled",
  "feature.node.kubernetes.io/cpu-security.tdx.enabled",
]
```

Finally, install the kata runtime classes (e.g. `kata-qemu-snp` and `kata-qemu-tdx`) in the cluster:

```commandline
$ surveyor kata setup
```

### Initialize the service

Configure CanaryBit Surveyor to use the right CanaryBit Inspector and registry endpoint:

```commandline
$ surveyor init
```

### Create the Secrets

Log in via the CanaryBit CLI and provision the CanaryBit user Access Token (`CB_TOKEN`) as Kubernetes Secret:

```commandline
$ export CB_TOKENS=$(cb login)
$ surveyor secret create inspector \
    --cb-tokens "$CB_TOKENS" \
    --namespace default \
```

Create the registry Secret in order to pull the CanaryBit Inspector client (`cbclient`) container image:

```commandline
$ surveyor secret create registry \
    --password $(cb login registry cbclient | base64 -d | cut -f 2 -d :) \
    --namespace default \ 
```

### Deploy and attest your Pod or Container

Deploy your Pod (e.g. `mypod.yaml`) with CanaryBit Surveyor via Pod or Deployment manifest:

```commandline
$ surveyor deploy \
  --cc-mode kata \
  --kata-runtime-class kata-qemu-snp \
  --attestation-targets snp \
  mypod.yaml
```

CanaryBit Surveyor will extend the Pod manifest by adding the CanaryBit Inspector client (`cbclient`) as init container, ensuring the workload is launched only upon a successful attestation by CanaryBit Inspector.

## Apply custom policies

CanaryBit Surveyor allows end-users to apply custom policies (e.g. `mypolicy.rego`) at different levels in the technology stack. 
These policies will then be enforced on top of CanaryBit Inspector default policies and together verify the correctness and security of each Trusted Execution Environment.

To create a custom policy, simply create a file with your policy content. For example:

``` title="mypolicy.rego"

package mypolicy

default allow := false

allow if {
  input.claims.attestations.canarybit.kernel_version == "6.17.0-14-generic"
}
```

Apply the custom policy as follows:

```commandline
$ surveyor deploy \
  --cc-mode kata \
  --kata-runtime-class kata-qemu-snp \
  --attestation-targets snp \
  --attestation-policy mypolicy.rego
  mypod.yaml
```

## Download the reports

The verification reports and additional insights are available for download on the [CanaryBit Inspector Dashboard.](https://dashboard.inspector.confidentialcloud.io)

