# Architecture

Confidential Cloud is designed in three main **microservices**: [Studio](#studio), [Tower](#tower) and [Inspector](#inspector).

![Confidential Cloud Architecture](./img/confidential-cloud-platform.png)

## Studio

Studio enables organisations to **share & process** sensitive data.
It adds a simple UX interface atop state-of-the-art Confidential Computing security mechanisms. Studio facilitates multi-party data collaborations, controls the ongoing activities and provides a detailed security report.

## Tower

Tower **orchestrates** the execution environment.
It creates all the required infrastructure resources and a Trusted Execution Environment (TEE, see [Confidential Computing](https://www.canarybit.eu/what-is-confidential-computing-and-why-should-i-care/)). Tower can orchestrate TEEs on the resources of a Cloud Service Provider of your choice, or On-Prem (see [Infrastructure]()).
It reliably destroys all the resources once the execution is completed or compromised. Each confidential environment is single-use and immutable once created.

## Inspector

Inspector **verifies & certifies** the execution environment.
It validates that the hardware setup has Confidential Computing capability enabled and collects information on the hardware, firmware, and software level to attest its trustworthiness.
It monitors the infrastructure and guarantees that untrustworthy environments are destroyed if they fail to meet execution policies.
