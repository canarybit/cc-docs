# Verify the security of your infrastructure with Inspector

[Inspector](https://docs.confidentialcloud.io/architecture/#inspector) is CanaryBit's **attestation** service for Confidential Computing setups.

## Attestation Overview
Here we briefly explain a high-level overview of how attestation works within Inspector. Inspector acts as a verifier in a remote attestation architecture [RATS architectire](https://datatracker.ietf.org/doc/rfc9334/). The used terminology here is based on RATS architecture. 

Workload attestation is vital for confidential computing and it verifies the integrity and authenticity of workloads running inside a trusted execution environment (TEE). The successful attestation procedure will estalish the trust between a confidential computing workload and relying parties and ensures that the workload has not been tampered with, compromised, or replaced by malicious code.


Inspector attestation service involves the steps below:

* A confidential computing workload attests its identity and integrity by providing TEE measurements along with other validation data (endorsements) included in attestation report.
* Inspector checks the attestation report against reference values and endorsements to determine its validity and whether specific measurements align with stored values and policies.
* A relying party uses the returned information from Inspector to determine whether to trust the attester or not.

### Verifier

The verifier or Inspector validates, and assesses the attestation evidence. It ensures the integrity and authenticity of workloads by comparing the evidence with stored reference values, endorsements, and policies. Based on these checks, Inspector decides whether the workload is trustworthy and should be allowed to process secure data or application.


### Attester

The attester is a confidential computing environment that must prove its identity and demonstrate that it has not been tampered with or compromised.

We provide a client (CBclient) to streamline the process of obtaining an evidence from the attesting workload. The CBclient uses libraries and tools from AMD to handle low-level platform software calls to obtain the evidence.

The attesting workload is responsible for gathering evidence for the attestaion report, either using the AMD tools or another compatible method. The attestaion report is then sent directly to Inspector (passport model in RATS architecture).


### Relying Party

The relying party which can be data or application owner uses the attestation result returned from Inspector to determine whether to trust the attester and transfer its data or application to the confidential workload. 


## Attestation Security

Inspector uses the following security machanisms to ensure confidentiality, integrity, authenticity, and non-repudiation of attestaion procedure. The used security techniques are described in detail below.   

### Secure Communication

During the attestation procedure, the communication between the attester, verifier, and the relying parties must be secured to protect the confidentiality and integrity of the attestation information. Techniques including encryption and secure communication channel such as TLS are used to establish secure communication. These techniques ensure confientiality and integrity of the acommunication. 

### Nonce
A nonce is a random generated string that uniquely identifies each attestation request. Inspector creates and returns a nonce, which attester use to identify an attestation request. Generated nonces and their creation timestamp are stored on Inspector and each nonce can be only used once within the maximum nonce age. This is to avoid replay attacks that could allow an attacker to reuse the past attestation results.

### Authentication

#### Workload Authentication
The verifier must be able to authenticate the identity of the attester, ensuring that the evidence is coming from the correct source (e.g., a trusted platform or device). The attestation evidence is often signed using digital signatures, ensuring that the verifier can authenticate the origin of the evidence and confirm that it has not been tampered with. 

#### Optional Client Authentication
The attestaion request to the Inspector can include a Certificate Signing Request (CSR) that can be used by a Certificate Authority (CA) to issue digital certificates for client authentication. 
Steps Involved in Using a CSR for client authentication are:

* CBclient first generates a key pair: a public key and a private key.
* CBclient creates a CSR that includes, the client’s public key, identifying information about the client and a signature created using the client’s private key.
* CBclient sends the CSR along with the attestation request to the Inspector. 
* Inspector sends the CSR to the specified CA on behalf of the CBclient using Simple Certificate Enrollment Protocol (SCEP), SCEP is a protocol designed to facilitate the secure issuance of certificates. CSR is submitted along with an optional authentication secret (e.g., a pre-shared key or password) to authenticate the request. This is known as the SCEP challenge password.
* The SCEP CA verifies the CSR by checking the validity of the client’s public key and the digital signature created using the private key. It also checks if the CBclient is authorized to request a certificate (e.g. based on the challenge password).
* If the CSR is valid and the CBclient is authorized, the CA issues the CBclient certificate and resturns it to the Inspector.
* Inspector forwards the signed certificate from the SCEP CA server to the CBclient.
* CBclient then can use the singed certificate to request resources. 

## Attestion FLow
Inspector attestion workflow is represted in the figure below which is based on passport model in RATS architecture: 

![Attestation Flow](./img/Attestation-flow.png)

## Attestation Policies
Inspector uses attestation policies to define and perform tests to the evidence or endorsments during attestation verification. Inspector uses OPA which is an open source, general-purpose policy engine that unifies policy enforcement across the stack. OPA provides a high-level declarative language that lets data or application owners to specify policy as code and simple APIs to offload policy decision-making from the software. OPA policies are expressed in a high-level declarative language called Rego. As an example, a policy can compare the kernel or OS version of the workload to determine if they have the defined values and then set the policy result. 


## GPU Attestation
Inspector has support for remote attestaion of NVIDIA H100 GPU TEE. The concept of a GPU in Trusted Execution Environment (TEE) is relatively new and enhances the capabilities of a traditional CPU TEE. There are many reosurce intensive applications including AI and amchine learning that require the performance boost provided by GPU hardware acceleration. Many AI models and parameters usually include sensitive data and a confidential GPU offers a secure environment for these workloads, ensuring protection against unauthorized access or tampering.

The GPU itself does not constitute a full TEE for confidential computing and it depends on a confidential CPU TEE. The CPU TEE provides the necessary measurements and attestations to establish trust in the GPU. The CPU TEE securely transfers information to the GPU via a fully encrypted channel. The GPU and the confidential VM exchange keys to create a secure, encrypted communication channel.

### GPU Attestation Flow


## Supported Trusted Execution Environments
The following TEEs are currently supported.

### AMD SEV-SNP (Secure Encrypted Virtualization - Secure Nested Paging)
AMD SEV-SNP is the latest security feature in AMD's EPYC processors designed to protect virtualized environments, especially in cloud and multi-tenant environments. SEV-SNP builds upon previous AMD security technologies like SEV and SEV-ES to deliver stronger protections against a broader range of attacks.

The AMD SEV-SNP evidence contains the following fields.

- options--SNP validation options, "DEFAULT" or "HYPERV" (Azure/Hyper-V special case).
- evidence--Attestation report binary file, as Base64-encoded string.
- vcek--VCEK (Versioned Chip Endorsement Key) certificate file, as Base64-encoded string.
- ark--ARK (Attestation Root Key) cerfificate file, as Base64-encoded string.
- ask--ASK (Attestation Signing Key) cerfificate file, as Base64-encoded string.

```
{
  "options": "string",
  "evidence": "string",
  "vcek": "string",
  "ark": "string",
  "ask": "string",
}

```
#### Evidence Verification

* Attestation report verification
Inspector verifies that the attestation report is valid and was signed by the submitted VCEK certificate.

* Certificate chain verification
Inspector verifies the submitted certificate chain. The verified certificate chain can have one of the following formats.

- AMD ARK, ASK, and VCEK (Versioned Chip Endorsement Key)

```
  +----------------------+
  | Attestation Root Key  | 
  |       (ARK)           |
  +----------------------+
            |
  +----------------------+
  | Attestation Signing   | <-- Signed by ARK, used by PSP to sign reports
  |       Key (ASK)       |
  +----------------------+
            |
  +----------------------+
  |  Versioned Chip       | <-- Signed by ASK, includes platform and VM state
  | Endorsement Key (VCEK)|
  +----------------------+
            |
  +----------------------+
  |  SEV-SNP Attestation  | <-- Signed by VCEK
  |        Report         |
  +----------------------+
```

- AMD ARK, ASVK, and VLEK (Versioned Loaded Endorsement Key)

```
 +----------------------+
  | Attestation Root Key  | 
  |       (ARK)           |
  +----------------------+
            |
  +----------------------+
  | Attestation Signing   | <-- Signed by ARK, used by PSP to sign reports
  |       Key (ASK)       |
  +----------------------+
            |
  +----------------------+
  |  Versioned Loaded     | <-- Signed by ASK, includes platform and VM state
  | Endorsement Key (VLEK)|
  +----------------------+
            |
  +----------------------+
  |  SEV-SNP Attestation  | <-- Signed by VLEK
  |        Report         |
  +----------------------+
```

* Measurement evidence claim verification 
Inspector verifies if the measurement value contained in the attestation report is equal to the reference value calculated by using the [sev-snp-measure tool](https://github.com/virtee/sev-snp-measure).

- Report data evidence claim verification
 Inspector verifies if the hash of additional report data is included in the attestation report is equal to the hash of the report data computed by Inspector (skipped for Azure/Hyper-V).


### Intel TDX
- To be defined.

### NVIDIA GPU
NVIDIA GPU Confidential Computing is an initiative that enables secure processing of sensitive data and application on GPUs, ensuring that the data or applicaton remains protected during computation.

The verification for the NVIDIA GPU attestation request is performed using the [GPU attestation tool](https://github.com/canarybit/gpu-attestation).

The GPU report contains the following fields.

- nonce--Request nonce, generated by Inspector for each request.
- arch--GPU architecture.
- driver_version--List of driver versions for each attested GPU.
- vbios_version--List of VBIOS versions for each attested GPU.
- evidence--List of evidence gathered for each attested GPU, with the following contents:
    * certChainBase64Encoded--Certificate chain, encoded as Base64 string.
    * attestationReportHexStr--Attestation report, encoded as hex string.
```
{
  "nonce": "string",
  "arch": "string",
  "driver_version": [
    "string"
  ],
  "vbios_version": [
    "string"
  ],
  "evidence": [
    {
      "certChainBase64Encoded": "string",
      "attestationReportHexStr": "string"
    }
  ]
}
```
