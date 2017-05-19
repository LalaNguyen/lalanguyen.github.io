---
layout: post
title:  "How EINIT works in Simulation Mode"
date:   2017-05-19 10:00:00 +0800
categories: research sgx
---
## Objective
The objective is to understand how an enclave is initialized and launched by an architectural enclave - the Launch Enclave (LE). I review this instruction in both theory and practice (i.e., Simulation Mode).

## Documentation
1. [Intel SGX Explained - Page 64] [sgx-explain]
2. [Intel SGX SDK Developer Reference for Linux 1.8] [sgx-developer]
3. [Intel Programming Reference][program-reference]

## Theory

After enclave's code and data are loaded, `EINIT` is invoked to complete the enclave creation process. `EINIT` finalizes the enclave measurement to establish the (1) enclave entity and (2) sealing entity used by `EGETKEY` and `EREPORT` during `software attestation`. Until an `EINIT` is executed, the enclave is not permitted to execute any enclave code(i.e., entering enclave by executing EENTER).

A correct construction results in the cryptographic log `EINITTOKEN` matching the one built by enclave owner in `SIGSTRUCT`.

### SIGSTRUCT & EINITTOKEN

According to


[sgx-explain]:https://eprint.iacr.org/2016/086.pdf
[sgx-developer]:https://download.01.org/intel-sgx/linux-1.8/docs/Intel_SGX_SDK_Installation_Guide_Linux_1.8_Open_Source.pdf
[program-reference]:https://software.intel.com/sites/default/files/managed/48/88/329298-002.pdf
