---
layout: post
title: How EADD works in Simulation Mode
date: 2017-05-11 03:40:55 +0000
categories: research sgx

---
## Objective
The objective is to understand how a free EPC page is selected by a system software, and how `EADD` loads code and data into an enclave. Similar to `ECREATE`, we review theory and observe the code in practice (i.e., simulation mode).

## Documentation
1. [Intel SGX Explained - Page 64] [sgx-explain]
2. [Intel SGX SDK Developer Reference for Linux 1.8] [sgx-developer]
3. [Intel SGX SDK Functions for Enclave Creation] [github-link]
4. [Intel Programming Reference][program-reference]
5. [Linux's mmap][mmap]

## Theory

`ECREATE` marks the newly created SECS as uninitialized. While an enclave’s SECS is in this state, the system software can use `EADD` instructions to load the initial code and data into the enclave. `EADD` also receives `PAGEINFO` as an input.

### PAGEINFO & SECINFO
`EADD` receives `PAGEINFO` as an input which specifies the source page in unprotected memory region. The `PAGEINFO` is prepared by system software
![My helpful screenshot]({{site.url}}/images/PAGEINFO.png)
The `SECINFO` further describe the page type and the state of the page
![My helpful screenshot]({{site.url}}/images/SECINFO.png)
![My helpful screenshot]({{site.url}}/images/SECINFOFLAG.png)
![My helpful screenshot]({{site.url}}/images/SECINFOFLAG2.png)
### EADD
`EADD` is used to convert a free EPC page into either TCS pages (i.e., PT_TCS) or regular pages (i.e., PT_REG)[[4]][program-reference]. When `EADD` is invoked, a couple of things happen:
* `EADD` receives `PAGEINFO` as an input which specifies the source page in unprotected memory region. The `PAGEINFO` is prepared by system software
* `EADD` copies 4KB of data from unprotected memory outside of EPC to the allocated EPC selected by system software
* `EADD` updates `Page Table Entry` for the new mapping inside EPC
* The system software (OS/Hypervisor) is responsible for selecting (1)  free EPC page, (2) type of page to be added, (3) attribute of the page, (4) content of the page, and (5) the SECS to which the page is added to. So the `SECS` is determined by the untrusted system software.
* `EADD` asks the processor to associate the page to the `protected SECS` provided as input. This association happens in `EPCM`. During the association, protected `SECS.MRENCLAVE` is updated by `EEXTEND`
* `EADD` records `EPCM` information in a cryptographic log stored in `SECS`.
* `EADD` asks the processor to initialize the EPCM entry to add the type of page (e.g., PT_REG or PT_TCS), the  linear address used by enclave to access the page, and the enclave RWX permission on the page

After a page has been added to an enclave, software can measure a 256 byte region as determined by the developer by invoking `EEXTEND`.

This `EADD` instruction requires ring-0 privileges and receives address of PAGEINFO struct and address of destination EPC page
![My helpful screenshot]({{site.url}}/images/EADD.png)

As for `EADD` instruction, `PAGEINFO` structure is prepared by the system software [[1]][sgx-explain] with the `SECS` of owner enclave [[4]][program-reference]. It is important to note that the `SECS` is determined by the system software. The `SRCPGE` of `PAGEINFO` contains the effective address of the source page. The RCX register is the effective address of the destination EPC. It is an address of an empty slot in the EPC. The figure belows [[1]][sgx-explain] illustrates the role of `PAGEINFO`.

![My helpful screenshot]({{site.url}}/images/eadd-epcm.png)


## Practice

Continue the experiment in `ECREATE`, we begin with piece of codes that begin after the `SECS` Page is allocated.

```cpp
linux-sgx/psw/urts/loader.cpp

int CLoader::build_image(SGXLaunchToken * const lc, sgx_attributes_t * const secs_attr, le_prd_css_file_t *prd_css_file, sgx_misc_attribute_t * const misc_attr)
{
  if(SGX_SUCCESS != (ret = build_secs(secs_attr, misc_attr)))
    {
        SE_TRACE(SE_TRACE_WARNING, "build secs failed\n");
        return ret;
      };
  ...
    // read reloc bitmap before patch the enclave file
    // If load_enclave_ex try to load the enclave for the 2nd time,
    // the enclave image is already patched, and parser cannot read the information.
    // For linux, there's no map conflict. We assume load_enclave_ex will not do the retry.
  ...
  //build sections, copy export function table as well;
  if(SGX_SUCCESS != (ret = build_sections(&bitmap)))
  {
      SE_TRACE(SE_TRACE_WARNING, "build sections failed\n");
      goto fail;
  }
```
I have no clues why we need a bitmap before patching the enclave file, and what the bitmap is. However, the `build_sections` links to the `build_pages` so I think this is where they add non-EPC memory to EPC memory
```cpp
linux-sgx/psw/urts/loader.cpp

int CLoader::build_sections(vector<uint8_t> *bitmap)
{
  ...
  sec_info_t sinfo;
  memset(&sinfo, 0, sizeof(sinfo));
  // OS determines what type of page to be mapped
  sinfo.flags = last_section->get_si_flags();
  uint64_t rva = ROUND_TO_PAGE(last_section->get_rva() + last_section->virtual_size());
  if(SGX_SUCCESS != (ret = build_pages(rva, size, 0, sinfo, ADD_EXTEND_PAGE)))
                return ret;
  ...
}

int CLoader::build_pages(const uint64_t start_rva, const uint64_t size, const void *source, const sec_info_t &sinfo, const uint32_t attr)
{
  int ret = SGX_SUCCESS;
  uint64_t offset = 0;
  uint64_t rva = start_rva;

  while(offset < size)
  {
      //For each page, call driver to add page;
      //GET_PTR returns source at offset 0
      if(SGX_SUCCESS != (ret = get_enclave_creator()->add_enclave_page(ENCLAVE_ID_IOCTL, GET_PTR(void, source, 0), rva, sinfo, attr)))
      {
          //if add page failed , we should remove enclave somewhere;
          return ret;
      }
      offset += SE_PAGE_SIZE;
      rva += SE_PAGE_SIZE;
  }
```

First, we notice that the `ENCLAVE_ID_IOCTL` is passed to `add_enclave_page`. In simulation, it is defined as a macro for `m_enclave_id`:
```cpp
linux-sgx/psw/urts/loader.h

#if defined(SE_SIM)
#define ENCLAVE_ID_IOCTL m_enclave_id
#else

sgx_enclave_id_t    m_enclave_id;
```

Recall that during `ECREATE`,  `m_enclave_id` is set after `ECREATE` copies unprotected SECS to protected SECS. So I guess this to tell the `add_enclave_page` that we use expect the enclave with `m_enclave_id` to add the EPC page.
```cpp
linux-sgx/sdk/simulation/uinst/enclave_mngr.cpp

CEnclaveSim::CEnclaveSim(const secs_t* secs)
{
    m_cpages = static_cast<size_t>(secs->size >> SE_PAGE_SHIFT);
    m_flags = new si_flags_t[m_cpages];

    // pages flags are initialized to -1
    memset(m_flags, 0xff,  m_cpages * sizeof(si_flags_t));

    // ECREATE copies unprotected SECS to protected SECS
    memcpy_s(&m_secs, sizeof(m_secs), secs, sizeof(*secs));

    // Associate a Simulated Enclave with an ID
    m_enclave_id = gen_enclave_id();
}

```

Second, we can see that the `SECINFO` is prepared by system software. So this will call driver to add the page, this reminds me of `driver_api.cpp`. Before that, let's check the enclave simulator

```cpp
linux-sgx/sdk/simulation/urtssim/enclave_creator_sim.cpp

int EnclaveCreatorSim::add_enclave_page(sgx_enclave_id_t enclave_id, void *src, uint64_t offset, const sec_info_t &sinfo, uint32_t attr)
{
    // Make the pointer source point to the same pointee
    void* source = src;
    // Allocate an empty page with SE_PAGE_SIZE = 0x1000 equals to 4KB
    uint8_t color_page[SE_PAGE_SIZE];
    // succeeds if source is null
    if(!source)
    {
        // Clean up the empty page
        memset(color_page, 0, SE_PAGE_SIZE);
        // Make it point to the empty page
        source = reinterpret_cast<void*>(&color_page);
    }
    return ::add_enclave_page(enclave_id, source, (size_t)offset, sinfo, attr);
}
```
If the page is empty, fill it with `0`, and it is passed to `driver_api.cpp`.
```cpp
linux-sgx/sdk/simulation/driver_api/driver_api.cpp

int add_enclave_page(sgx_enclave_id_t enclave_id,
                     void             *source,
                     size_t           offset,
                     const sec_info_t &secinfo,
                     uint32_t         attr)
{
    sec_info_t   sinfo;
    page_info_t  pinfo;
    CEnclaveMngr *mngr;
    CEnclaveSim  *ce;

    UNUSED(attr);
    // Enclaves are managed by Enclave Manager
    mngr = CEnclaveMngr::get_instance();
    // Ask the Enclave Manager to return the enclave we created during ECREATE
    ce = mngr->get_enclave(enclave_id);
    if (ce == NULL) {
        SE_TRACE(SE_TRACE_DEBUG,
                 "enclave (id = %llu) not found.\n",
                 enclave_id);
        return SGX_ERROR_INVALID_ENCLAVE_ID;
    }
    // Clear the SECINFO struct
    memset(&sinfo, 0, sizeof(sec_info_t));

    // Prepare the SECINFO struct
    sinfo.flags = secinfo.flags;
    if(memcmp(&sinfo, &secinfo, sizeof(sec_info_t)))
        return SGX_ERROR_UNEXPECTED;

    memset(&pinfo, 0, sizeof(pinfo));

    // Prepare the PAGEINFO struct
    // Assign the destination SECS to the Simulated Enclave
    pinfo.secs     = ce->get_secs();
    // Assign the linear address of the Simulated Enclave + offset for the next page. Since the first page is allocated to SECS
    pinfo.lin_addr = (char*)ce->get_secs()->base + offset;
    // Assign source page in unprotected memory area
    pinfo.src_page = source;
    // Assign security flags and page type, these fields are set by OS during the section parsing
    pinfo.sec_info = &sinfo;
    /* Passing NULL here when there is no EPC mgmt. */
    return (int)DoEADD_SW(&pinfo, GET_PTR(void, ce->get_secs()->base, offset));
}
```
Similar to `ECREATE`, `EADD` also has a simulated version which can be found under `u_instructions.cpp`. The `EADD` receives `PAGEINFO` and the offset within Simulated Enclave.

```cpp
linux_sgx/sdk/simulation/uinst/u_instructions.cpp

uintptr_t _EADD(page_info_t* pi, void *epc_lin_addr)
{

    void     *src_page = pi->src_page;
    CEnclaveMngr *mngr = CEnclaveMngr::get_instance();
    // EADD does not receive enclave_id, so we need to identify its through lin_addr
    CEnclaveSim    *ce = mngr->get_enclave(pi->lin_addr);
    if (ce == NULL) {
        SE_TRACE(SE_TRACE_DEBUG, "failed to get enclave instance\n");
        return SGX_ERROR_UNEXPECTED;
    }
    // Check for page alignment
    GP_ON(!IS_PAGE_ALIGNED(epc_lin_addr));
    // Check for if the SGX_FLAGS_INITTED is on since it cannot add more page when the attribute is set to initialized
    GP_ON((ce->get_secs()->attributes.flags & SGX_FLAGS_INITTED) != 0);

    // Make the page writable before doing memcpy()
    se_virtual_protect(epc_lin_addr, SE_PAGE_SIZE, SI_FLAGS_RW);

    // EADD copies the page from src_page to epc_lin_addr
    mcp_same_size(epc_lin_addr, src_page, SE_PAGE_SIZE);

    se_virtual_protect(epc_lin_addr, SE_PAGE_SIZE, (uint32_t)pi->sec_info->flags);

    //check to see if the page is successfully added to the enclave
    GP_ON(!ce->add_page(pi->lin_addr, pi->sec_info->flags));
    return SGX_SUCCESS;
}
```

Up to this point, it is clear that OS fills in `PAGEINFO` and `EADD` cannot proceed if `SGX_FLAGS_INITTED` is set. However, it is not clear how OS selects a free EPC page, and a newly created EPC is assigned to SECS page. Since the EPC pages are allocated during `ECREATE`, I assume that they only deal with virtual memory address of EPC. It is expected that:
* The processor updates `EPCM`'s`SECS` with `SECS` from `PAGEINFO`
* The processor updates `EPCM`'s`LINADDR` with `epc_lin_addr` of newly created EPC.

Since `EPCM` is not available in the simulation scheme, I guess this is what they meant by `no hardware protection`. While loading an enclave, the system software will also use the `EEXTEND` instruction, which updates the
enclave’s measurement used in the software attestation process. Nonetheless, there is no simulation code for `EEXTEND`.

[sgx-explain]:https://eprint.iacr.org/2016/086.pdf
[sgx-developer]:https://download.01.org/intel-sgx/linux-1.8/docs/Intel_SGX_SDK_Developer_Reference_Linux_1.8_Open_Source.pdf
[github-link]:https://insujang.github.io/2017-04-14/intel-sgx-sdk-functions-for-enclave-creation/
[program-reference]:https://software.intel.com/sites/default/files/managed/48/88/329298-002.pdf
[mmap]:http://man7.org/linux/man-pages/man2/mmap.2.html