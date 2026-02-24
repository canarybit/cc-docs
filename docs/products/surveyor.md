# CanaryBit Surveyor

*Safeguard Container execution*

---

For contenerised application that analyse sensitive data, it became crucial to verify the confidentiality of the nodes where the containers are running, no matter if they are managed by an orchestration service (e.g. Azure AKS or AWS EKS) or directly on a container platform.

CanaryBit Surveyor ensures the contenerised platform uses isolation mechanisms to protect the privacy of the workloads and dataset, and verifies the cluster characteristics:

* before a container is executed (always);

* at a defined schedule (if enabled);

* when specific system-wide events occur (if enabled).

This is extremely beneficial for dynamic clusters where the size of the Kubernetes cluster (# of Nodes) is adjusted based on the utilization of Pods and Nodes in the cluster. This feature, called autoscaling is typically recommended by hyperscalers CaaS solutions.