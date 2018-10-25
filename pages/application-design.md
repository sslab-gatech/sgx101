---
layout: page
title: Application Design
---

In order to create an application with SGX enabled, we need to establish enclaves, attestation and sealing.

Enclave:
----

Application design with Intel SGX requires that the application be divided into two components:

- Trusted component: This is the enclave. The code that resides in the trusted code is the code that accesses an application’s secrets. An application can have more than one trusted component/enclave.

- Untrusted component: This is the rest of the application and any of its modules. It is important to note that, from the standpoint of an enclave, the OS and the VMM are considered untrusted components.

The trusted component should be as small as possible to save more protected memory and reduce attack surface. Meanwhile, enclaves should also have minimal trusted-untrusted component interaction.

Attestation:
----

In the Intel SGX architecture, attestation refers to the process of demonstrating that a specific enclave was established on a platform. There are two attestation mechanisms:
Local attestation occurs when two enclaves on the same platform authenticate to each other. Remote attestation occurs when an enclave gains the trust of a remote provider.

Sealing:
----

Sealing is the process of encrypting data so that it can be written to untrusted memory or storage without revealing its contents. The data can be read back in by the enclave at a later date and unsealed (decrypted). The encryption keys are derived internally on demand and are not exposed to the enclave.

There are two methods of sealing data:
----
- Enclave Identity: This method produces a key that is unique to this exact enclave.

- Sealing Identity: This method produces a key that is based on the identity of the enclave’s Sealing Authority. Multiple enclaves from the same signing authority can derive the same key.

Now that we have a better understanding of the main structure of an SGX application, we will proceed our tutorial to provide separate detailed explanations of each of the three main components. Each individual discussion of the component will also be followed by an existing code example to help understand implementation details.

At the end of the tutorial, we will develop an SGX-enabled application (a password manager). And after that, you should be well equipped with all the necessary knowledge to develop your own SGX-enabled applications!

Now let's jump into the heart of SGX, Enclave.