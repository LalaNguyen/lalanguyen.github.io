---
layout: post
title:  "Installing Intel SGX v1.8 with Simulation on Ubuntu 16.04 x64 bit"
date:   2017-05-10 11:40:55 +0800
categories: research sgx
---
### Objective
The objective is to successfully install Intel SGX version 1.8 with Simulation library on Ubuntu 16.04 x64 bit without SGX-enabled CPU.

### Documentation
1. [Intel SGX Linux Github Homepage] [linux-sgx]
2. [Intel SGX SDK Installation Guide for Linux 1.8] [sgx-install]


### Few thoughts
The Installation Guide [[2]][sgx-install] provides setup information for Intel SGX, including:
* Installation package for Intel(R) SGX driver (required for hardware protection)
* Installation package for Intel(R) SGX platform software (PSW) (required for hardware protection)
* Installation package for Intel(R) SGX SDK

These packages can be downloaded from `https://download.01.org/intel-sgx/linux-1.8/`. My computer does not have SGX-enabled processor, I only need to install the Intel(R) SGX SDK. It can be difficult to debug the pre-build package. Hence, we will recompile and install from the source files.
### Step-by-step Installation instructions

1. Prepare dependencies documented in [[1]][linux-sgx] on Ubuntu 16.04 to install the required tools to build Intel(R) SGX SDK:
```bash
sudo apt-get install build-essential ocaml automake autoconf libtool wget python
```
2. Clone the source package from home page [[1]][linux-sgx]. I clone this to my home folder at `/home/lalang`:
```bash
git clone https://github.com/01org/linux-sgx.git
```
3. Examine the folder with `ls -l` command:
![My helpful screenshot]({{site.url}}/images/ls.png)
4. Navigate to the new folder `linux-sgx` and execute the following command:
```bash
sudo make sdk_install_pkg USE_OPT_LIBS=0 DEBUG=1
```
A few thing to note about the above command that Intel provides us `precompiled optimized libraries` which can be download through the script `./download_prebuilt.sh`. However, I could not have found any document regarding this optimized libraries. Following the download link, I found out that they are bulk of signed-precompiled `architectural enclaves`. I decide to ignore it by setting `USE_OPT_LIBS=0`. To debug an enclave with gdb, we need to enable `DEBUG=1`.

5. After the `make` command, we have a new installer for Intel SGX SDK:
![My helpful screenshot]({{site.url}}/images/installer.png)

6. From the `linux-sgx` directory, install the Intel SGX SDK:
```bash
sudo ./linux/installer/bin/sgx_linux_x64_sdk_1.8.100.37739.bin
```
7. Upon execution, we will be prompted to set the installation directory as depicted below:
![My helpful screenshot]({{site.url}}/images/complete.png)
It is recommended to answer `no` and provide the destination directory as `/opt/intel` since this matches with the `environment file` supplied by Intel when the installation is complete. Otherwise, we have to change the `environment file`.
8. Execute the command ```source /opt/intel/sgxsdk/environment``` and navigate to `/opt/intel/sgxsdk/SampleCode` to proceed with sample codes.

### Step-by-step Uninstallation instructions
These instructions are helpful as we modify the libraries and want a reinstallation
1. Execute the uninstalled script:
```bash
sudo /opt/intel/sgxsdk/uninstall.sh
```
This will remove everything under `/opt/intel`
2. Navigate back to our cloned `linux-sgx` directory and execute the command:
```bash
sudo make clean
```

### Tips for Debugging
For debugging, I change the library source code and re-install it. Here is the sequences of commands I use everytime I change the SGX library code to trace the sgx program's control flow:
```bash
sudo make clean
sudo /opt/intel/sgxsdk/uninstall.sh
sudo make sdk_install_pkg USE_OPT_LIBS=0 DEBUG=1
sudo ./linux/installer/bin/sgx_linux_x64_sdk_1.8.100.37739.bin
source /opt/intel/sgxsdk/environment
cd /opt/intel/sgxsdk/SampleCode/SampleEnclave/
sudo make SGX_MODE=SIM
```
[linux-sgx]:https://github.com/01org/linux-sgx
[sgx-install]:https://download.01.org/intel-sgx/linux-1.8/docs/Intel_SGX_SDK_Installation_Guide_Linux_1.8_Open_Source.pdf
