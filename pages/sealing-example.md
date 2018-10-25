---
layout: page
title: Sealing Example
---

This sealing example is only for illustration purpose. It generates a random number inside the enclave and calls the sealing api to seal it. Then it unseals the sealed data structure to verify the number. Code is available [here](https://github.com/sangfansh/SGX101_sample_code).

Original unmodified version is available [here](https://github.com/digawp/hello-enclave).

In `App.cpp`:

<script src="https://gist.github.com/sangfansh/9859ce170fce4fa63deba999d6d13326.js"></script>

The application first initializes the enclave in the main function. Then it makes an `ECall` into the enclave to generate a random number (a fake random number just for simplicity).

<script src="https://gist.github.com/sangfansh/a55622c217d4ce1b8d469667e27e615b.js"></script>

In order to seal the number, the application first has to **allocate memory** for sealed data block (line 30 at App.cpp). Then it makes another `ECall` into the enclave to seal the random secret.

The `seal()` function is an `ECall` wrapper function of the trusted SGX sealing api. It passes the required parameters into function `sgx_seal_data()` provided by SGX SDK (`Sealing.cpp`). If this ECall returns successfully, the random secret will be securely sealed in `(sgx_sealed_data_t*)sealed_data`.

<script src="https://gist.github.com/sangfansh/67214567d3edf0223a1fc925e0d0f698.js"></script>

After the random secret is successfully sealed, the application makes another `ECall` `unseal()` to unseal the sealed_data. Function `unseal()` is also a wrapper function of the trusted SGX sealing api `sgx_unseal_data()`. If this `ECall` returns successfully, the unsealed content of `sealed_data` will be stored into `int unsealed`.

Finally the application verifies the result by printing out and comparing the unsealed secret with the original generated random number (line 50 at `App.cpp`).

To compile and run this example, run make inside the example directory and type `./app` to run the application. It should produce some output like this:

![sealing](/sgx101/assets/pics/sealing_example.png)