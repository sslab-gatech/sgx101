---
layout: page
title: Enclave Layout
---

In this section we will look into the details of the memory organization of enclaves.

Physical Memory Organization
====

The enclave’s code and data is stored in Processor Reserved Memory (PRM), which is a subset of DRAM that cannot be directly accessed by other software, including system software and System Management Module code (Ring 2). Direct Memory Access targeting the PRM is also rejected by the CPU in order to protect enclave from other peripherals.

Enclave Page Cache (EPC)
----

The contents of enclaves and the associated data structures are stored in the Enclave Page Cache (EPC), which is a subset of the PRM.

![PRM](/sgx101/assets/pics/enclave1.png)

The SGX design supports having multiple enclaves on a system at the same time, which is a necessity in multi-process environments. This is achieved by having the EPC split into 4 KB pages that can be assigned to different enclaves. The EPC uses the same page size as the architecture’s address translation feature.

The EPC is managed by the same system software that manages the rest of the computer’s physical memory. The system software, which can be a hypervisor or an OS kernel, uses SGX instructions to allocate unused pages to enclaves, and to free previously allocated EPC pages. Non-enclave software cannot directly access the EPC, as it is contained in the PRM.

Enclave Page Cache Map (EPCM)
----

The SGX design expects the system software to allocate the EPC pages to enclaves. However, as the system software is not trusted, SGX processors check the correctness of the system software’s allocation decisions, and refuse to perform any action that would compromise SGX’s security guarantees. For example, if the system software attempts to allocate the same EPC page to two enclaves, the SGX instruction used to perform the allocation will fail.

In order to perform its security checks, SGX records some information about the system software’s allocation decisions for each EPC page in the Enclave Page Cache Map (EPCM). The EPCM is an array with one entry per EPC page, so computing the address of a page’s EPCM entry only requires a bitwise shift operation and an addition. (Like the page table in a regular OS)

![EPCM](/sgx101/assets/pics/enclave2.png)

Above are the fields in an EPCM entry that track the ownership of pages.

The SGX instructions that allocate an EPC page set the VALID bit of the corresponding EPCM entry to 1, and refuse to operate on EPC pages whose VALID bit is already set.

The instruction used to allocate an EPC page also determines the page’s intended usage, which is recorded in the page type (PT) field of the corresponding EPCM entry. The pages that store an enclave’s code and data are considered to have a regular type (PT_REG). The pages dedicated to the storage of SGX’s supporting data structures are tagged with special types. For example, the PT_SECS type identifies pages that hold SGX Enclave Control Structures.

A page’s EPCM entry also identifies the enclave that owns the EPC page. This information is used by the mechanisms that enforce SGX’s isolation guarantees to prevent an enclave from accessing another enclave’s private information. As the EPCM identifies a single owning enclave for each EPC page, it is impossible for enclaves to communicate via shared memory using EPC pages.

SGX Enclave Control Structure (SECS)
----

SGX stores per-enclave metadata in a SGX Enclave Control Structure (SECS) associated with each enclave. Each SECS is stored in a dedicated EPC page with the page type PT_SECS. These pages are not intended to be mapped into any enclave’s address space, and are exclusively used by the CPU’s SGX implementation.

An enclave’s identity is almost synonymous to its SECS. The first step in bringing an enclave to life allocates an EPC page to serve as the enclave’s SECS, and the last step in destroying an enclave deallocates the page holding its SECS. The EPCM entry field identifying the enclave that owns an EPC page points to the enclave’s SECS. The system software uses the virtual address of an enclave’s SECS to identify the enclave when invoking SGX instructions.

All SGX instructions take virtual addresses as their inputs. Given that SGX instructions use SECS addresses to identify enclaves, the system software must create entries in its page tables pointing to the SECS of the enclaves it manages. However, the system software cannot access any SECS page, as these pages are stored in the PRM. SECS pages are not intended to be mapped inside their enclaves’ virtual address spaces, and SGX-enabled processors explicitly prevent enclave code from accessing SECS pages.

Memory Layout of Enclave (Virtual Memory)
====

SGX was designed to minimize the effort required to convert application code to take advantage of enclaves. Therefore, it adopts similar memory address translation mechanism as the system software such as operating system or hypervisor.

Enclave Linear Address Range (ELRANGE)
----

_("Linear" roughly means "virtual" in regular memory layout)_ Each enclave designates an area in its virtual address space, called the enclave linear address range (ELRANGE), which is used to map the code and the sensitive data stored in the enclave’s EPC pages. The virtual address space outside ELRANGE is mapped to access non-EPC memory via the same virtual addresses as the enclave’s host process.

![ELRANGE](/sgx101/assets/pics/enclave3.png)

The SGX design guarantees that the enclave’s memory accesses inside ELRANGE obey the virtual memory abstraction, while memory accesses outside ELRANGE receive no guarantees (because system software cannot be trusted). Therefore, enclaves must store all their code and private data inside ELRANGE, and must consider the memory outside ELRANGE to be an untrusted interface to the outside world.

ELRANGE is specified using a base (the BASEADDR field) and a size (the SIZE) in the enclave’s SECS. ELRANGE must meet the same constraints as a variable memory type range and as the PRM range, namely the size must be a power of 2, and the base must be aligned to the size. These restrictions are in place so that the SGX implementation can inexpensively check whether an address belongs to an enclave’s ELRANGE quickly, in either hardware or software.

SGX Enclave Attributes
----

The execution environment of an enclave is heavily influenced by the value of the ATTRIBUTES field in the enclave’s SECS.

![Enclave Attrbites](/sgx101/assets/pics/enclave4.png)

The most important attribute, from a security perspective, is the DEBUG flag. When this flag is set, it enables the use of SGX’s debugging features for this enclave. These debugging features include the ability to read and modify most of the enclave’s memory. Therefore, DEBUG should only be set in a development environment, as it causes the enclave to lose all the SGX security guarantees.

SGX guarantees that enclave code will always run with the XCR0 register set to the value indicated by extended features request mask (XFRM). Enclave authors are expected to use XFRM to specify the set of architectural extensions enabled by the compiler used to produce the enclave’s code.

The MODE64BIT flag is set to true for enclaves that use the 64-bit Intel architecture.

The INIT flag is always false when the enclave’s SECS is created. The flag is set to true at a certain point in the enclave lifecycle.

Address Translation for SGX Enclaves
----

Under SGX, the operating system and hypervisor are still in full control of the page tables and Extended Page Tables, and each enclave’s code uses the same address translation process and page tables as its host application. This minimizes the amount of changes required to add SGX support to existing system software. At the same time, having the page tables managed by untrusted system software opens SGX up to the address translation attacks.

SGX’s active memory mapping attacks defense mechanisms revolve around ensuring that each EPC page can only be mapped at a specific virtual address. When an EPC page is allocated, its intended virtual address is recorded in the EPCM entry for the page, in the ADDRESS field.

When an address translation result is the physical address of an EPC page, the CPU ensures that the virtual address given to the address translation process matches the expected virtual address recorded in the page’s EPCM entry.

![EPCM Entry](/sgx101/assets/pics/enclave5.png)

SGX also protects against some passive memory mapping attacks and fault injection attacks by ensuring that the access permissions of each EPC page always match the enclave author’s intentions. The access permissions for each EPC page are specified when the page is allocated, and recorded in the readable (R), writable (W), and executable (X) fields in the page’s EPCM entry.

An enclave author must include memory layout information ("page table") along with the enclave, in such a way that the system software loading the enclave will know the expected virtual memory address and access permissions for each enclave page. In return, the SGX design guarantees to the enclave authors that the system software, which manages the page tables and Extended Page Table, will not be able to set up an enclave’s virtual address space in a manner that is inconsistent with the author’s expectations.

Finally, a SGX-enabled CPU will ensure that the virtual memory inside ELRANGE is mapped to EPC pages. This prevents the system software from carrying out an address translation attack where it maps the enclave’s entire virtual address space to DRAM pages outside the PRM, which do not trigger any of the checks above, and can be directly accessed by the system software.

Thread Control Structure (TCS)
----

The SGX design fully supports multi-core processors. It is possible for multiple logical processors to concurrently execute the same enclave’s code at the same time, via different threads.

The SGX implementation uses a Thread Control Structure (TCS) for each logical processor that executes an enclave’s code. 

Each TCS is stored in a dedicated EPC page whose EPCM entry type is PT_TCS. Some of the first few fields are considered to belong to the architectural part of the structure, and therefore are guaranteed to have the same semantics on all the processors that support SGX. The rest of the TCS is not documented.

The contents of an EPC page that holds a TCS cannot be directly accessed, even by the code of the enclave that owns the TCS. This restriction is similar to the restriction on accessing EPC pages holding SECS instances. However, the architectural fields in a TCS can be read by enclave debugging instructions.

The architectural fields in the TCS lay out the context switches performed by a logical processor when it transitions between executing non-enclave and enclave code.

State Save Area (SSA)
----

When the processor encounters a hardware exception, such as an interrupt, while executing the code inside an enclave, it performs a privilege level switch and invokes a hardware exception handler provided by the system software. Before executing the exception handler, however, the processor needs a secure area to store the enclave code execution context, so that the information in the execution context is not revealed to the untrusted system software.

In the SGX design, the area used to store an enclave thread’s execution context while a hardware exception is handled is called a State Save Area (SSA). Each TCS references a contiguous sequence of SSAs. The offset of the SSA array (OSSA) field specifies the location of the first SSA in the enclave’s virtual address space. The number of SSAs (NSSA) field indicates the number of available SSAs.

![SSA](/sgx101/assets/pics/enclave6.png)

Each SSA starts at the beginning of an EPC page, and uses up the number of EPC pages that is specified in the SSAFRAMESIZE field of the enclave’s SECS. These alignment and size restrictions most likely simplify the SGX implementation by reducing the number of special cases that it needs to handle.

An enclave thread’s execution context consists of the general-purpose registers (GPRs) and the result of the XSAVE instruction. Therefore, the size of the execution context depends on the requested-feature bitmap (RFBM) used by to XSAVE. All the code in an enclave uses the same RFBM, which is declared in the XFRM enclave attribute. The number of EPC pages reserved for each SSA, specified in SSAFRAMESIZE, must be large enough to fit the XSAVE output for the feature bitmap specified by XFRM.

SSAs are stored in regular EPC pages, whose EPCM page type is PT_REG. Therefore, the SSA contents is accessible to enclave software. This opens up possibilities for an enclave exception handler that is invoked by the host application after a hardware exception occurs, and acts upon the information in a SSA.

----

References:
----
1. https://eprint.iacr.org/2016/086.pdf
2. https://software.intel.com/en-us/blogs/2016/06/10/overview-of-intel-software-guard-extensions-instructions-and-data-structures