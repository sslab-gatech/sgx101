---
layout: page
title: Attestation Primitives
---

Sometimes enclaves need to collaborate with other enclaves on the same platform due to different reasons such as data exchange if the enclave is too small to hold all the information, or communication with Intel reserved enclaves to conduct specific Intel services. 

Therefore, the two exchanging enclaves have to prove to each other that they can be trusted. In other scenarios when an SGX enabled ISV client requests secrets from its ISV client, password management service for example, the client have to prove to the server that the client application is running on a trusted platform that can process the secrets securely. Both of those two conditions require a proof of secured execution environment, and Intel SGX refers to this proving process as attestation.

There are two types of attestation with respect to the two above mentioned scenarios: **Local Attestation** and **Remote Attestation**.

The successful result of local attestation provides an authenticated assertion between two enclaves running on the same platform that they can trust each other and exchange information safely, while remote attestation provides this kind of verification for the ISV client to the server so that ISV server can confidently provides the client with the secrets it requested.

Before diving into the details of attestation, we need to clarify some required instructions and data.

Device Root Keys
====

There are two device root keys that are fused to SGX CPU at production.

Root Provisioning Key (RPK)
----

This key is randomly generated on a dedicated Hardware Security Module (HSM) within a special purpose facility called Intel Key Generation Facility (iKGF) which is guaranteed to be a well-guarded offline production facility. Intel is also responsible for maintaining a database of all keys ever produced by the HSM. RPKs are delivered out to different factory facilities, named by Intel’s formal publications as the “high volume manufacturing system”, to be integrated into processors’ fuses.

Intel stores all RPKs as they are the basis of how SGX processors demonstrate their genuineness through an online provisioning protocol. For this reason, the iKGF also forwards different derivations of each RPK to Intel’s online servers.

Root Sealing Key (RSK)
----

This key is randomly generated automatically inside the CPU during production to be statistically different from part to part. Intel declares that it attempts to erase all production lines residues of this key so that each platform should assume that its RSK value is both unique and known only to itself. Most of the keys provided by enclave’s trusted interface base their derivation on platform’s RSK so that no other parties can known the keys.

![attestation](/sgx101/assets/pics/attestation.png)

Enclave Measurement (aka Software TCB):
====

When an enclave is built and initialized, Intel SGX will generate a cryptographic log of all the build activities, including:
---

* Content: code, data, stack, heap

* Location of each page within the enclave

* Security flags being used

The “Enclave Identity”, which is a 256-bit hash digest of the log, is stored as MRENCLAVE as the enclave’s software TCB. In order to verify the software TCB, one should first securely obtain the enclave’s software TCB, then securely obtain the expected enclave’s software TCB and compare those two values.

Two user instructions:
====

REPORT Contains following data:
----

* Measurement of the code and data in the enclave.

* A hash of the public key in the ISV certificate presented at enclave initialization time.

* User data.

* Other security related state information (not described here).

* A signature block over the above data, which can be verified by the same platform that produced the report.

EREPORT
----

This instruction generates a cryptographic structure, called REPORT, that binds MRENCLAVE to the target enclave’s REPORT KEY.

REPORT KEY
----

Used by EREPORT to sign all reports generated on that specific platform and destined that for that enclave.

EGETKEY
----

Enclaves can use EGETKEY instruction to get derivatives of device keys. EGETKEY produces symmetric keys for different purposes depending on invoking enclave attributes and the requested key type. There are five different key types. Two are sealing and report keys available for all enclave. The rest are limited only to SGX architectural enclaves.

EGETKEY uses Security Version Number (SVN) specified by the requesting enclave to define requested key characteristics. CPU SVN to reflect processor microcode version, or ISV SVN to reflect enclave software version. EGETKEY checks these values against those stored in SIGSTRUCT and only allows to obtain keys with SVN values lower or equal to those of the invoking enclave so that upgraded versions of the same software can retrieve keys created by former versions.

SIGSTRUCT
----

Enclaves’ certificate is called SIGSTRUCT and is a mandatory supplement for launching any enclave. The SIGSTRUCT holds enclave’s MRENCLAVE together with other enclave attributes. SIGSTRUCTs are signed by the ISV with its private key, which was originally signed by an SGX launch authority. Intel is considered the primary enclave launch authority, however other entities can be trusted by the platform owner to authorize launching of enclaves. The respected launch authority is specified by its public key hash signed by Intel and stored on the platform.

We start with clarifying the process of local attestation and then remote attestation.

----

References:
----

1. https://courses.cs.ut.ee/MTAT.07.022/2017_spring/uploads/Main/hiie-report-s16-17.pdf
2. https://www.idc.ac.il/en/schools/cs/research/Documents/jackson-msc-thesis.pdf
3. https://software.intel.com/en-us/articles/innovative-technology-for-cpu-based-attestation-and-sealing
4. http://tce.webee.eedev.technion.ac.il/wp-content/uploads/sites/8/2015/10/SGX-for-Technion-TCE.pdf