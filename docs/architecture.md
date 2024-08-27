# Architecture

Confidential Cloud is runs in three main **microservices**: [Studio](#studio), [Tower](#tower) and [Inspector](#inspector).

![Confidential Cloud Architecture](./img/confidential-cloud-platform.png)

## Studio

Studio enables organisations to securely **share & process** data in trusted execution environments.
It adds a simple UX interface atop state-of-the-art digital security mechanisms, including confidential computing. 
Studio enables and facilitates secure multi-party data collaboration, enforces the scope of data processing and provides a detailed security report.

## Tower

Tower **orchestrates** the infrastructure.
It creates all the required infrastructure resources and one or more trusted execution environments (TEEs, see [Confidential Computing](https://www.canarybit.eu/what-is-confidential-computing-and-why-should-i-care/)). 
Tower can orchestrate TEEs either on the resources of a Cloud Service Provider of your choice, or On-Prem (see [Infrastructure]()).
It destroys all the resources once the execution is completed or compromised. Each trusted execution environment is single-use and immutable once created.

## Inspector

Inspector **verifies & certifies** the infrastructure.
It validates that the underlying platform has support for and uses Confidential Computing capabilties enabled by the platform's instruction set architecture and firmware (see [ISA]()).
Inspector performs the validation based on an Attestation Report provided by a software client deployed in the TEE.
The client software collects information on the hardware, firmware, and software level to attest its trustworthiness.
Inspector monitors the infrastructure and enforces customer-defined deployment policies by destroying infrastructure components that fail to meet the deployment policy.