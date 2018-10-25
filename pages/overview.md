---
layout: page
title: Overview
---

This section offers an overview of the properties provided by Intel SGX.

Intel SGX (Software Guard Extension) is a new instruction set in Skylake Intel CPUs since autumn 2015. It provides a reverse sandbox that protects enclaves from: 

- OS or hypervisor

- BIOS, firmware, drivers

- System management module (Ring 2)

- Intel management engine (ME)

- Any remote attack

In short, SGX architecture is a hardware-enforced security mechanism that requires **Trusted Computing Base (TCB), Hardware Secrets, Remote Attestation, Sealed Storage and Memory Encryption**.

Here, **TCB** will be the CPU’s package boundary and software components related to SGX.

**Hardware Secrets** will be two 128-bit keys at production: Root Provisioning Key and Root Seal Key. Notice that RPK is known to Intel and RSK is not, therefore most of he derived key are based on RSK. We will discuss this later in the tutorial.

**Remote Attestation** is enforced for the client to prove to the service provider that an enclave is running a given software, inside a given CPU, with a given security level, for a given Individual Software Vender (ISV). This is required before the service provider decides to provide requested secrets.

**Sealed Storage** is required to save secret data to untrusted media. Data and code inside enclaves are not secrets. They are just logics that are required to process the secret and most of them are open sourced or can be reverse engineered. Therefore, secrets are provisioned later by the service provider and should be stored out of the enclave through sealing mechanism when necessary (e.g. for future usage).

In summary, Intel SGX offers the following protections from known hardware and software attacks:
----

- Enclave memory cannot be read or written from outside the enclave regardless of the current privilege level and CPU mode.

- Production enclaves cannot be debugged by software or hardware debuggers. 

- The enclave environment cannot be entered through classic function calls, jumps, register manipulation, or stack manipulation. The only way to call an enclave function is through a new instruction that performs several protection checks.

- Enclave memory is encrypted using industry-standard encryption algorithms with replay protection. Tapping the memory or connecting the DRAM modules to another system will yield only encrypted data.

- The memory encryption key randomly changes every power cycle. The key is stored within the CPU and is not accessible.

- Data isolated within enclaves can only be accessed by code that shares the enclave.

As a result, the attack surface can be largely reduced after applying Intel SGX:

![Attack Surface](/sgx101/assets/pics/overview.png)

However, there are still several security limitation faced by Intel SGX:
----
- Cache timing attacks

- Physical attacks. E.g. laser fault injection attacks

- Microcode malicious patching

- Untrusted I/O

- Human element. E.g. trusted development environment

- CPU bugs and bugs in dependencies.

Note that SGX trusted enclaves and microcode can be patched. However, memory encryption crypto cannot be patched since it is fused to hardware.

Nevertheless, those security concerns are out of the tutorial's scope. Now let's get started with SGX!

----

References:
----
1. https://www.blackhat.com/docs/us-16/materials/us-16-Aumasson-SGX-Secure-Enclaves-In-Practice-Security-And-Crypto-Review.pdf
2. http://www.cs.tau.ac.il/~tromer/istvr1516-files/lecture10-trusted-platform-sgx.pdf
3. https://software.intel.com/en-us/articles/intel-software-guard-extensions-tutorial-part-1-foundation
4. https://software.intel.com/en-us/blogs/2016/06/10/overview-of-intel-software-guard-extensions-instructions-and-data-structures
5. https://download.01.org/intel-sgx/linux-2.2/docs/Intel_SGX_Developer_Guide.pdf