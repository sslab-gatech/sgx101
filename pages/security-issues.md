---
layout: page
title: Security Issues
---

Although Intel SGX technology claims to deliver various security promises, which we have gone through in previous sections, it is still vulnerable to certain types of attacks. Recent acadamic researches have discovered several such vulnerabilities and prove that Intel SGX is not as secure as we thought.

In this section, [Prof. Taesoo Kim](https://taesoo.kim/) explaines known security concerns, including cache/branch side-channel attacks and memory safety issues, and corresponding defenses with various working demos.

----

## SGX Security Issues (Taesoo Kim)
<iframe width="560" height="315" src="https://www.youtube.com/embed/0TXR2JNsFBk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Slides are available [here](/sgx101/files/security-issues.pdf).

----

## Demos

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