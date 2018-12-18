---
layout: post
title: On the performance of Intel SGX in Simulation Mode
date: 2017-05-12 11:56:55 +0000
categories: research sgx

---
## Objective

The objective is to benchmark Intel SGX performance in term of memory access. I wrote a small program \[\[3\]\]\[benchmark-code\] to compare between the native environment (without SGX) and SGX-protected environment.

## Systems Info

![My helpful screenshot]({{site.url}}/images/system-info.png)

## Experiment Descriptions

The test program aims to test 4 cases on memory access on both SGX and non-SGX version (i.e., native environment):

* Sequential read
* Sequential write
* Random read
* Random write

There are two experiments: one for SGX and one for non-SGX version. For each experiment, I run test on 4 types of memory access on an array.

* For each type of memory access, I run it `1,000` times on a fixed-size `int array[MAX_SIZE]` whose `MAX_SIZE` is varied from `10` to `1,000,000`. Then I calculate the average time the program executes. Time is measured in microsecond.s

## Implementation Details

The program structure is listed as follows:
![My helpful screenshot]({{site.url}}/images/native-benchmark.png)

where as: 

* `App.h` and `App.cpp` includes our main procedure and clock-timing.
* `benchmark.h` and `benchmark.cpp` includes our codes for testing native environment. Four functions are declared as follows:

```cpp
benchmark.h

#ifndef BENCHMARK_H    // To make sure you don't declare the function more than once by including the header multiple times.
#define BENCHMARK_H

#define MAX_ELEMENT 1000000 // No.Elements in array

void test_sequential_read(void);
void test_sequential_write(void);
void test_random_read(void);
void test_random_write(void);

#endif
```

* `Enclave.cpp`, `Enclave.edl`, and `Enclave.h` contains our codes for testing SGX performance via four ECALL enclave's functions as follows:

```cpp
Enclave.edl
...
trusted {
          public void ecall_test_sequential_write(void);
          public void ecall_test_sequential_read(void);
          public void ecall_test_random_read(void);
          public void ecall_test_random_write(void);
  };
...
```

* `Enclave.lds`, `Enclave.config.xml`, `Enclave_private.pem`, and `Makefile` are configuration files copied from Intel SGX `SampleEnclave` directory. Since some tests requires large array with more than `100,000` elements (i.e., > 400KB), I need to extends the size of stack so I need to modify `Enclave.config.xml`. Also, I add `benchmark.cpp` to the `Makefile`.

Here is the visualization of execution flows of the test program:

![My helpful screenshot]({{site.url}}/images/benchmark-execution-flow.png)

It is important to note that `rand()` is unavailable inside enclave so Intel suggests us to use `sgx_read_rand()` instead to generate a random sequence.

## Discussion

The results reveal that SGX in simulation mode suffers around 7.5% overhead performance in comparison to native run. \[IN\] represents codes executed in enclave and \[OUT\] represents codes executed in native environment. I looked up the Intel forum and found out that this conclusion is similar to that described in [\[1\]](https://software.intel.com/en-us/forums/intel-software-guard-extensions-intel-sgx/topic/721026). Following up the response from Intel, the reason was deemed to be linking on different libraries that hold the random number generator function. It is advised to compare between simulation mode and hardware mode. In my computer, L3 cache can hold up to 6MB, which is far greater than 400 KB. As Intel SGX memory encryption engine supports performance by caching data, the  cache miss cannot be a real problem.

## Reference

1. [SGX random memory access overheads by Muhammad Bilal
   ](https://software.intel.com/en-us/forums/intel-software-guard-extensions-intel-sgx/topic/721026)
2. [sgx_read_rand description](https://software.intel.com/en-us/node/709094)
3. \[This tutorial benchmark code\]\[benchmark-code\]

\[benchmark-code\]:{{ site.url }}/codes/BenchmarkEnclave.tar.gz