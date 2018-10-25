---
layout: page
title: Enclave Lifecycle
---

In this section we will look into the lifecycle of enclaves.

To initialize an enclave, four of the new CPU instructions provided by SGX architecture are used, each will be provided with a wrapper available for developers’ usage (will be explained in sample code):

ECREATE
----

An enclave is born when the system software issues the ECREATE instruction, which turns a free EPC page into the SECS for the new enclave.

ECREATE copies an SECS structure outside the EPC into an SECS page inside the EPC. The internal structure of SECS is not accessible to software. Software sets the following fields in the source structure: SECS:BASEADDR, SECS:SIZE, and ATTRIBUTES.

ECREATE validates the information used to initialize the SECS, and results in a page fault or general protection fault if the information is not valid. ECREATE will also fault if SECS target page is in use; already valid; outside the EPC; addresses are not aligned; unused PAGEINFO fields are not zero.

EADD
----

The system software can use EADD instructions to load the initial code and data into the enclave. EADD is used to create both TCS pages and regular pages. This function copies a source page from non-enclave memory into the EPC, associates the EPC page with an SECS page residing in the EPC, and stores the linear address and security attributes in EPCM. EADD reads its input data from a Page Information (PAGEINFO) structure.

The PAGEINFO structure contains:
----

- The virtual address of the EPC page (LINADDR)

- The virtual address of the non-EPC page whose contents will be copied into the newly allocated EPC page (SRCPGE)

- A virtual address that resolves to the SECS of the enclave that will own the page (SECS).

- The virtual address, pointing to a Security Information (SECINFO) structure, which contains the newly allocated EPC page’s access permissions (R, W, X) and its EPCM page type (RT_REG or PT_TCS).

- EADD validates its inputs, and modifies the newly allocated EPC page and its EPCM entry.

EADD ensures that the EPC page is not allocated to another enclave, the page’s virtual address falls within the enclave’s ELRANGE and all the reserved fields in SECINFO are set to zero.

EEXTEND
----

While loading an enclave, the system software will also use the EEXTEND instruction, which updates the enclave’s measurement used in the software attestation process. It updates the MRENCLAVE measurement register of an SECS with the measurement of an EXTEND string composed of EEXTEND`||`ENCLAVEOFFSET`||`PADDING`||`256 bytes of the enclave page. RCX register contains the effective address of the 256 byte region of an EPC page to be measured.

EINIT
----

This function is the final instruction executed in the enclave build process. After EINIT, the MRENCLAVE measurement is complete, and the enclave is ready to start user code execution using EENTER instruction.

When EINIT completes successfully, it sets the enclave’s INIT attribute to true. This opens the way for ring 3 application software to execute the enclave’s code, using the SGX instructions. On the other hand, once INIT is set to true, EADD cannot be invoked on that enclave anymore, so the system software must load all the pages that make up the enclave’s initial state before executing the EINIT instruction.

END OF ENCLAVE INITIALIZATION

----

Before jumping into design decisions, there are several terms need to be clarified first:
----
* ECall: “Enclave Call”, a call made into an interface function within the enclave

* OCall: “Out Call”, a call made from within the enclave to the outside application

* Trusted: Refers to any code or construct that runs inside an enclave in a “trusted” environment.

* Trusted Thread Context: The context for a thread running inside the enclave. This is composed of:

	- Thread Control Structure (TCS)
	- Thread Data/Thread Local Storage: data within the enclave and specific to the thread
	- State Save Area(SSA): a data buffer which holds register state when an enclave must exit due to an interrupt or exception
	- Stack: a stack located within the enclave

* Untrusted: Refers to any code or construct that runs in the applications “untrusted” environment (outside of the enclave).

This is the general approach we’ll follow for designing an application for Intel SGX:
----

* Identify the application’s secrets.

* Identify the providers and consumers of those secrets.

* Determine the enclave boundary.

* Tailor the application components for the enclave.

The first step in designing enclave is to identify the assets it needs to protect, the data structures where the assets are contained, and the set of code that operates on those data structures and then place them into a separate trusted library. It is a required step because enclave code is loaded in a special way such that once the enclave has been initialized, privileged code and the rest of the untrusted application cannot directly read data that resides in the protected environment, or change the behavior of the code within the enclave without detection.

After defining the trusted (enclave) and untrusted (application) components of an Intel SGX enabled application, we should carefully define the interface between untrusted application and enclave. An enclave must expose an API for untrusted code to call in (ECalls) and express what functions provided by the untrusted code are needed (OCalls). Both ECalls and OCalls are defined by developers using Enclave Definition Language (EDL), and they together consist the enclave boundary.



The Enclave Definition Language
---

Before moving to the actual enclave code sample, we’ll take a few moments to discuss the Enclave Definition Language (EDL) syntax. An enclave’s bridge functions, both its ECALLs and OCALLs, are prototyped in its EDL file and its general structure is as follows:

```
enclave {
	// Include files
	// Import other edl files
	// Data structure declarations to be used as parameters of the function prototypes in edl

	trusted {
		// Include file if any. It will be inserted in the trusted header file (enclave_t.h)
		// Trusted function prototypes (ECALLs)
	};

	untrusted {
		// Include file if any. It will be inserted in the untrusted header file (enclave_u.h)
		// Untrusted function prototypes (OCALLs)
	};
};
```

ECALLs are prototyped in the trusted section, and OCALLs are prototyped in the untrusted section.

When the project is built, the Edger8r tool that is included with the Intel SGX SDK parses the EDL file and generates a series of proxy functions. These proxy functions are essentially wrappers around the real functions that are prototyped in the EDL. Each ECALL and OCALL gets a pair of proxy functions: a trusted half and an untrusted half. The trusted functions go into EnclaveProject_t.h and EnclaveProjct_t.c and are included in the autogenerated Files folder of your enclave project. The untrusted proxies go into EnclaveProject_u.h and EnclaveProject_u.c and are placed in the autogenerated Files folder of the project that will be interfacing with your enclave.

![lifecycle](/sgx101/assets/pics/lifecycle.png)

Your program does not call the ECALL and OCALL functions directly; it calls the proxy functions. When you make an ECALL, you call the untrusted proxy function for the ECALL, which in turn calls the trusted proxy function inside the enclave. That proxy then calls the “real” ECALL and the return value propagates back to the untrusted function. This sequence is shown in the figure above. When you make an OCALL, the sequence is reversed: you call the trusted proxy function for the OCALL, which calls an untrusted proxy function outside the enclave that, in turn, invokes the “real” OCALL.

More details will be explained in the example code.

----

References:
----

1. https://eprint.iacr.org/2016/086.pdf
2. https://download.01.org/intel-sgx/linux-2.2/docs/Intel_SGX_Developer_Reference_Linux_2.2_Open_Source.pdf
3. https://software.intel.com/en-us/blogs/2016/06/10/overview-of-intel-software-guard-extensions-instructions-and-data-structures
4. https://insujang.github.io/2017-04-05/intel-sgx-instructions-in-enclave-initialization/
5. https://software.intel.com/en-us/blogs/2016/06/10/overview-of-intel-software-guard-extensions-instructions-and-data-structures
6. https://software.intel.com/en-us/articles/software-guard-extensions-tutorial-series-part-3
7. https://software.intel.com/en-us/articles/intel-software-guard-extensions-tutorial-part-7-refining-the-enclave