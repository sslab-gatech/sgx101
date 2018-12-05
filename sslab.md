---
layout: page
title: SSLab
sidebar_link: true
sidebar_sort_order: 2
---
SSLab represents the [Systems Software & Security Lab](https://gts3.org/) from 
Georgia Institute of Technology led by [Prof. Taesoo Kim](https://taesoo.kim/).

We have been actively working on SGX related research.
These research projects can be broadly classified into three different
categories: System Design, Defense, and Attack.

----

System Design
----

- OpenSGX:
  An open-source platform for SGX research that
  consists of a QEMU-based emulator and a
  software development kit (SDK)

- S-NFV:
  A protection scheme for network function virtualization (NFV)
  applications that uses SGX to secure the applications' internal states

- AirBox:
  A secure design of edge function platforms using SGX for ensuring
  code integrity and data confidentiality of an edge function

- SGX-Tor:
  A design of Tor that enhances the security and privacy of the protocol by
  utilizing SGX

----

Defense
----

- T-SGX:
  A compiler-level approach that incorporates Intel TSX to prevent SGX
  enclaves from controlled-channel attacks

- SGX-Shield:
  A software-based design of SGX enclaves that enables fine-grained
  address space layout randomization (ASLR)

----

Attack
----

- Branch Shadowing:
  A novel side-channel attack against SGX exploiting
  branch history states preserved across an SGX mode switch and
  last branch record (LBR)

- Dark ROP:
  A novel blind return-oriented programming (ROP) attack against SGX
  exploiting uninitialized registers across an enclave exit

- SGX-Bomb:
  A rowhammer attack against SGX resulting in processor lockdown,
  i.e., a cold reboot is necessary to use the machine again

- SGX-Bleed:
  A vulnerability that can leak uninitialized SGX memory through
  structure padding

----

Publications
----

- Leaking Uninitialized Secure Enclave Memory via Structure Padding (Extended Abstract, arXiv.org) [[pdf]](https://arxiv.org/abs/1710.09061)
- SGX-Bomb: Locking Down the Processor via Rowhammer Attack (SysTEX 2017) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2017/jang:sgx-bomb.pdf)
- Inferring Fine-grained Control Flow Inside SGX Enclaves with Branch Shadowing (Security 2017) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2017/lee:sgx-branch-shadow.pdf)
- Hacking in Darkness: Return-oriented Programming against Secure Enclaves (Security 2017) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2017/lee:darkrop.pdf)
- Enhancing Security and Privacy of Tor's Ecosystem by using Trusted Execution Environments (NSDI 2017) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2017/kim:sgx-tor.pdf)
- SGX-Shield: Enabling Address Space Layout Randomization for SGX Programs (NDSS 2017) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2017/seo:sgx-shield.pdf)
- T-SGX: Eradicating Controlled-Channel Attacks Against Enclave Programs (NDSS 2017) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2017/shih:tsgx.pdf)
- Fast, Scalable and Secure Onloading of Edge Functions using AirBox (SEC 2016) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2016/bhardwaj:airbox.pdf)
- S-NFV: Securing NFV states by using SGX (SDNNFVSEC 2016) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2016/shih:snfv.pdf)
- OpenSGX: An Open Platform for SGX Research (NDSS 2016) [[pdf]](https://sslab.gtisc.gatech.edu/assets/papers/2016/jain:opensgx.pdf)

----

Demos
----

#### 1. Branch Shadowing

This video shows how the branch shadowing attack can extract
RSA private key bits

- Target code: Sliding window exponentiation of mbedTLS
- Attack code: We modified Linux SGX SDK to run our shadow code
- Kernel log: Our attack code prints the output of LBR via dmesg

<iframe width="560" height="315" src="https://www.youtube.com/embed/R_C0rfbg3p8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

#### 2. Dark ROP

This video shows how the Dark ROP attack detects
memcpy() and copy the entire memory contents of
an enclave to the outside.

<iframe width="560" height="315" src="https://www.youtube.com/embed/kixVRC-SHQE" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

#### 3. SGX page-table-based attack

This video presents the page-table-based attack,
which is also known as the controlled-channel attack.
By manipulating the page table and hooking the page
fault handler, the attacker is able to observe
precise page access patterns.

<iframe width="560" height="315" src="https://www.youtube.com/embed/MCSlgEqNhIA" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

#### 4. SGX-Shield

This video demonstrates the effectiveness of fine-grained
ASLR support of SGX-Shield.

<iframe width="560" height="315" src="https://www.youtube.com/embed/0aSilQ5SR58" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

#### 5. T-SGX

This video shows how T-SGX protect an SGX enclave from
page-table-based attacks.

<iframe width="560" height="315" src="https://www.youtube.com/embed/uHt6D-lHl68" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

#### 6. SGX-Bomb

This video shows how the SGX-Bomb attack locks down
a victim machine.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Sutd0uCdY2E" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

#### 7. SGX-Bleed

This video shows how the SGX-Bleed problem leaks
uninitialized SGX memory via structure padding.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Cd-o_wV4iGk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>