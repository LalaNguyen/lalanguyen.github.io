---
layout: post
title:  "How ECREATE works in simulation"
date:   2017-05-10 11:40:55 +0800
categories: research sgx
---
## Objective
The objective is to understand how an sgx enclave is created in theory and in practice (i.e., simulation mode).

## Documentation
1. [Intel SGX Explained - Page 63] [sgx-explain]
2. [Intel SGX SDK Developer Reference for Linux 1.8] [sgx-developer]
3. [Intel SGX SDK Functions for Enclave Creation] [github-link]
4. [Intel Programming Reference][program-reference]
5. [Linux's mmap][mmap]

## Theory

An enclave is born when the system software issues the ECREATE instruction, which turns a free EPC page into the SECS for the new enclave. To understand ECREATE, we need to understand SECS, PAGEINFO, and EPCPAGE structure.

### SECS & PAGEINFO
According to [[4]][program-reference], the SECS is a data structure with 4K-Bytes alignment:
![My helpful screenshot]({{site.url}}/images/SECS1.png)
![My helpful screenshot]({{site.url}}/images/SECS2.png)

The SECS has the following properties:
>There is one SECS for each enclave. The SECS contains meta-data which is used by the hardware to protect the enclave. Included in the SECS is a field which stores the enclave build measurement value. This field, MRENCLAVE, is initialized by the ECREATE instruction and updated by every EADD and EEXTEND. It is locked by EINIT. The SECS cannot be accessed by software.

So SECS cannot be accessed by system softwares (OS/hypervisor). We focus on two important fields:
* `MRENCLAVE` is a unique hash for an enclave. This field is initialized by ECREATE and updated by `EADD` and `EEXTEND`, and
* `MRSIGNER` is a proof of enclave's ownership

`PAGEINFO` is an architectural data structure that is used as a parameter to the EPC-management instructions. It requires 32-Byte alignment.
![My helpful screenshot]({{site.url}}/images/PAGEINFO.png)

Unlike `SECS`, `PAGEINFO` can be accessed by the system software.[[1]][sgx-explain]

### ECREATE
System software specifies `base address` and `size of the enclave` in `unprotected SECS` outside of the EPC area. This address range is known as `ELRANGE`.

Following the [[4]][program-reference], the ECREATE leaf instruction is responsible for:
* Set up the initial environment
* ECREATE copies the `unprotected SECS` into EPC area, under newly created SECS page. This page is not part of the `ELRANGE` and shall not be mapped.
* Validate the information used to initialize protected SECS
* Set up fields in the protected SECS and mark the EPC page as `Valid`
* Set up `INIT` attribute (sub-field of the `ATTRIBUTES` field in the enclave's `SECS`) to `FALSE`. The enclave code cannot excute if this field is set to `FALSE`.

This instruction requires ring-0 privileges and receives address of `PAGEINFO` struct and address of destination SECS page, which is an `EPCPAGE`, as an input.
![My helpful screenshot]({{site.url}}/images/ECREATE.png)

As for `ECREATE` instruction, `PAGEINFO` structure is prepared by the system software [[1]][sgx-explain] with the `SECS` field unused [[4]][program-reference]. The `SRCPGE` of `PAGEINFO` contains the effective address of `the source/outside/unprotected SECS`. The RCX register is the effective address of the destination `SECS`. It is an address of an empty slot in the `EPC`.

## Practice

In this practice, I examine the code in `App.cpp` in `SampleEnclave` came along with sdk distribution. Skipping all the launch check, I observe the enclave is first created by `sgx_create_enclave`

### `sgx_create_enclave`
```cpp
linux-sgx/SampleCode/SampleEnclave/App/App.cpp

...
/* Step 2: call sgx_create_enclave to initialize an enclave instance */
/* Debug Support: set 2nd parameter to 1 */
ret = sgx_create_enclave(ENCLAVE_FILENAME, SGX_DEBUG_FLAG, &token, &updated, &global_eid, NULL);
...

```
This function resides in the urts (unTruted Run-time System) library on `psw` directory. I put `prinf` and recompile the source codes to trace the flow
```cpp
linux-sgx/psw/urts/linux/urts.cpp


extern "C" sgx_status_t sgx_create_enclave(const char *file_name, const int debug, sgx_launch_token_t *launch_token, int *launch_token_updated, sgx_enclave_id_t *enclave_id, sgx_misc_attribute_t *misc_attr){
...
printf("-----sgx_create_enclave in urts.cpp-----\n");
ret = _create_enclave(!!debug, fd, file, NULL, launch_token, launch_token_updated, enclave_id, misc_attr);
...
}
```
The function `_create_enclave` is defined in the header of `urts.cpp`, the `urts_com.h`:
```cpp
linux-sgx/psw/urts/urts_com.h

sgx_status_t _create_enclave(const bool debug, se_file_handle_t pfile, se_file_t& file, le_prd_css_file_t *prd_css_file, sgx_launch_token_t *launch, int *launch_updated, sgx_enclave_id_t *enclave_id, sgx_misc_attribute_t *misc_attr){
  // Some codes with parser, I am not sure about the parser
  // Need to set the whole misc_attr instead of just secs_attr.
  do {
      ret = __create_enclave(parser, mh->base_addr, metadata, file, debug, lc, prd_css_file, enclave_id,
                             misc_attr);
      //SGX_ERROR_ENCLAVE_LOST caused by initializing enclave while power transition occurs
  } while(SGX_ERROR_ENCLAVE_LOST == ret);
  ...
}
```
The `metadata` we pass to __create_enclave is actually not documented, but the code can be founded in `metadata.h`. Similarly, `enclave_css_t` holds information to compute enclave signature (i.e., `MRENCLAVE` and `MRSIGNER`), which can be founded in `arch.h`

```cpp
linux-sgx/common/inc/internal/metadata.h

typedef struct _metadata_t
{
    uint64_t            magic_num;             /* The magic number identifying the file as a signed enclave image */
    uint64_t            version;               /* The metadata version */
    uint32_t            size;                  /* The size of this structure */
    uint32_t            tcs_policy;            /* TCS management policy */
    uint32_t            ssa_frame_size;        /* The size of SSA frame in page */
    uint32_t            max_save_buffer_size;  /* Max buffer size is 2632 */
    uint32_t            desired_misc_select;
    uint32_t            reserved;
    uint64_t            enclave_size;          /* enclave virtual size */
    sgx_attributes_t    attributes;            /* XFeatureMask to be set in SECS. */
    enclave_css_t       enclave_css;           /* The enclave signature */
    data_directory_t    dirs[DIR_NUM];
    uint8_t             data[2208];
}metadata_t;
```
```cpp
/****************************************************************************
* Definitions for enclave signature
****************************************************************************/
#define SE_KEY_SIZE         384         /* in bytes */
#define SE_EXPONENT_SIZE    4           /* RSA public key exponent size in bytes */

typedef struct _css_header_t {        /* 128 bytes */
    uint8_t  header[12];                /* (0) must be (06000000E100000000000100H) */
    uint32_t type;                      /* (12) bit 31: 0 = prod, 1 = debug; Bit 30-0: Must be zero */
    uint32_t module_vendor;             /* (16) Intel=0x8086, ISV=0x0000 */
    uint32_t date;                      /* (20) build date as yyyymmdd */
    uint8_t  header2[16];               /* (24) must be (01010000600000006000000001000000H) */
    uint32_t hw_version;                /* (40) For Launch Enclaves: HWVERSION != 0. Others, HWVERSION = 0 */
    uint8_t  reserved[84];              /* (44) Must be 0 */
} css_header_t;
se_static_assert(sizeof(css_header_t) == 128);

typedef struct _css_key_t {           /* 772 bytes */
    uint8_t modulus[SE_KEY_SIZE];       /* (128) Module Public Key (keylength=3072 bits) */
    uint8_t exponent[SE_EXPONENT_SIZE]; /* (512) RSA Exponent = 3 */
    uint8_t signature[SE_KEY_SIZE];     /* (516) Signature over Header and Body */
} css_key_t;
se_static_assert(sizeof(css_key_t) == 772);

typedef struct _css_body_t {            /* 128 bytes */
    sgx_misc_select_t   misc_select;    /* (900) The MISCSELECT that must be set */
    sgx_misc_select_t   misc_mask;      /* (904) Mask of MISCSELECT to enforce */
    uint8_t             reserved[20];   /* (908) Reserved. Must be 0. */
    sgx_attributes_t    attributes;     /* (928) Enclave Attributes that must be set */
    sgx_attributes_t    attribute_mask; /* (944) Mask of Attributes to Enforce */
    sgx_measurement_t   enclave_hash;   /* (960) MRENCLAVE - (32 bytes) */
    uint8_t             reserved2[32];  /* (992) Must be 0 */
    uint16_t            isv_prod_id;    /* (1024) ISV assigned Product ID */
    uint16_t            isv_svn;        /* (1026) ISV assigned SVN */
} css_body_t;
se_static_assert(sizeof(css_body_t) == 128);

typedef struct _css_buffer_t {         /* 780 bytes */
    uint8_t  reserved[12];              /* (1028) Must be 0 */
    uint8_t  q1[SE_KEY_SIZE];           /* (1040) Q1 value for RSA Signature Verification */
    uint8_t  q2[SE_KEY_SIZE];           /* (1424) Q2 value for RSA Signature Verification */
} css_buffer_t;
se_static_assert(sizeof(css_buffer_t) == 780);

typedef struct _enclave_css_t {        /* 1808 bytes */
    css_header_t    header;             /* (0) */
    css_key_t       key;                /* (128) */
    css_body_t      body;               /* (900) */
    css_buffer_t    buffer;             /* (1028) */
} enclave_css_t;
```

The function `__create_enclave` is defined within the same file header `urts_com.h`:
```cpp
linux-sgx/psw/urts/urts_com.h

static int __create_enclave(BinParser &parser, uint8_t* base_addr, const metadata_t *metadata, se_file_t& file, const bool debug, SGXLaunchToken *lc, le_prd_css_file_t *prd_css_file, sgx_enclave_id_t *enclave_id, sgx_misc_attribute_t *misc_attr)
{
    // The "parser" will be registered into "loader" and "loader" will be registered into "enclave".
    // After enclave is created, "parser" and "loader" are not needed any more.
    printf("-----create_enclave in urts_com.h-----\n");
    ret = loader.load_enclave_ex(lc, debug, metadata, prd_css_file, misc_attr);
    ...
}
```
The function `load_enclave_ex` is defined in `loader.cpp`, which in-turn call the `load_enclave`
```cpp
linux-sgx/psw/urts/loader.cpp

int CLoader::load_enclave_ex(SGXLaunchToken *lc, bool debug, const metadata_t *metadata, le_prd_css_file_t *prd_css_file, sgx_misc_attribute_t *misc_attr)
{
  ...
while (retry)
{
    ret = this->load_enclave(lc, debug, metadata, prd_css_file, misc_attr);
    switch(ret)
    {
        //If CreateEnclave failed due to power transition, we retry it.
    case SGX_ERROR_ENCLAVE_LOST:     //caused by loading enclave while power transition occurs
        break;
    ...
}
```
The function `load_enclave` defined in `loader.cpp` does a couple of interesting things:
```cpp
linux-sgx/psw/urts/loader.cpp

int CLoader::load_enclave(SGXLaunchToken *lc, int debug, const metadata_t *metadata, le_prd_css_file_t *prd_css_file, sgx_misc_attribute_t *misc_attr)
{
  ...
  required_attr = &metadata->attributes;
  enclave_css   = &metadata->enclave_css;
  secs_attr     = &sgx_misc_attr->secs_attr;
  ...
  ret = validate_metadata();
  ...
  ret = get_enclave_creator()->get_misc_attr(&sgx_misc_attr, const_cast<metadata_t *>(m_metadata), lc, debug);
  ...
  ret = build_image(lc, &sgx_misc_attr.secs_attr, prd_css_file, &sgx_misc_attr);
  ...
}
```
First, it validates the metadata struct. Second, it calls `get_enclave_creator`, either response with SIM or HW constructor. In our case, the `get_enclave_creator` returns an instance from `enclave_creator_sim.cpp`. In brief, `get_misc_attr` validates information consistency between (1) `sgx_misc_attr` and `m_metadata`, and (2) `sgx_misc_attr` with launch token `lc`:
* FP/SSE are must-have attributes
* secs attributes.xfrm (secs_attr) does NOT match signature attributes.xfrm (enclave_css)
* secs attributes.flag (secs_attr) does NOT match signature attributes.flag (enclave_css)
* secs attributes does NOT match launch token attributes

Once the metadata is verified against the `secs_attr` in `sgx_misc_attr`, the system software begins the enclave building process with only `sgx_misc_attr`. The `build_image` can be found in `loader.cpp`
```cpp
linux-sgx/psw/urts/loader.cpp

int CLoader::build_image(SGXLaunchToken * const lc, sgx_attributes_t * const secs_attr, le_prd_css_file_t *prd_css_file, sgx_misc_attribute_t * const misc_attr)
{
  if(SGX_SUCCESS != (ret = build_secs(secs_attr, misc_attr)))
  ...
}
```
The build_secs seems what we are interested in. Recall that during the ECREATE, the system software must prepare the unprotected SECS struct, which can be accessed through `PAGEINFO`. The `build_secs` is as follows:
```cpp
linux-sgx/psw/urts/loader.cpp

int CLoader::build_secs(sgx_attributes_t * const secs_attr, sgx_misc_attribute_t * const misc_attr)
{
    memset(&m_secs, 0, sizeof(secs_t)); //should set reversed field of secs as 0.
    //create secs structure.
    m_secs.base = 0;    //base is allocated by driver. set it as 0
    // Fill in size of enclave
    m_secs.size = m_metadata->enclave_size;
    // Fill in MISC_SELECT
    m_secs.misc_select = misc_attr->misc_select;

    memcpy_s(&m_secs.attributes,  sizeof(m_secs.attributes), secs_attr, sizeof(m_secs.attributes));
    m_secs.ssa_frame_size = m_metadata->ssa_frame_size;

    EnclaveCreator *enclave_creator = get_enclave_creator();
    if(NULL == enclave_creator)
        return SGX_ERROR_UNEXPECTED;
    // Start address of the enclave is not determined until the next line is executed
    SE_TRACE(SE_TRACE_NOTICE, "enclave start address = %p, size = %x\n", m_start_addr, m_metadata->enclave_size);
    int ret = enclave_creator->create_enclave(&m_secs, &m_enclave_id, &m_start_addr, is_ae(&m_metadata->enclave_css));
    ...
}
```

I input the line `SE_TRACE(SE_TRACE_NOTICE, "enclave start address = %p, size = %x\n", m_start_addr, m_metadata->enclave_size)` and the output is

```bash
[build_secs /home/lalang/linux-sgx/psw/urts/loader.cpp:362] enclave start address = (nil), size = 800000
```
So the size is computed, but the base address is not available yet. One we execute the next line, we obtain the following output:

```bash
[build_secs /home/lalang/linux-sgx/psw/urts/loader.cpp:366] enclave start address = 0x7fe368971000, size = 800000
```
This give us a hint that ECREATE is actually executed inside `create_enclave` of `loader.cpp`. As an instance of `EnclaveCreatorSim`, I look into `enclave_creator_sim.cpp`:
```cpp
linux-sgx/sdk/simulation/urtssim/enclave_creator_sim.cpp

int EnclaveCreatorSim::create_enclave(secs_t *secs, sgx_enclave_id_t *enclave_id, void **start_addr, bool ae)
{
    UNUSED(ae);
    return ::create_enclave(secs, enclave_id, start_addr);
}
```
Since this is simulation, `ae` is not used, and the control flow is forwarded to `::create_enclave(secs, enclave_id, start_addr)`. In C++, this resolves to top-level `create_enclave` function, located in `driver_api.cpp`

```cpp
linux-sgx/sdk/simulation/driver_api/driver_api.cpp

/* Allocate linear address space. */
int create_enclave(secs_t           *secs,
                   sgx_enclave_id_t *enclave_id,
                   void             **start_addr)
{
    CEnclaveSim   *ce;
    sec_info_t    sinfo;
    page_info_t   pinfo;
    BUG_ON(secs == NULL, SGX_ERROR_UNEXPECTED);
    BUG_ON(enclave_id == NULL, SGX_ERROR_UNEXPECTED);
    BUG_ON(start_addr == NULL, SGX_ERROR_UNEXPECTED);
    // clear SECINFO
    memset(&sinfo, 0, sizeof(sinfo));
    sinfo.flags = SI_FLAGS_SECS; // Assign FLAGS to #define SI_FLAG_SECS (0x00<<0x8) /* SECS */ <--- PT_SECS left shift by 8 bits

    // clear PAGEINFO
    memset(&pinfo, 0, sizeof(pinfo));
    pinfo.src_page = secs; // Assign SRCPGE to unprotected SECS
    pinfo.sec_info = &sinfo;  //Assign SECINFO that describe the state of the enclave page

    // let the destination EPCPage filled by hardware since this is simulation, we can only observe PAGEINFO as an input
    ce = reinterpret_cast<CEnclaveSim*>(DoECREATE_SW(&pinfo));
    if (ce == NULL) {
        SE_TRACE(SE_TRACE_DEBUG, "out of memory.\n");
        return SGX_ERROR_OUT_OF_MEMORY;
    }

    *start_addr = ce->get_secs()->base;
    *enclave_id = ce->get_enclave_id();
    secs->base = *start_addr;

    return SGX_SUCCESS;
}
```
Trace through the header, we see that `DoECREATE_SW` is a macro defined in `sw_emu.h`
```cpp
linux_sgx/sdk/simulation/assembly/linux/sw_emu.h

/* This macro is used to generate simulation functions like:
 * DoECREATE_SW, DoEADD_SW, ...
*/
.macro DoSW inst
DECLARE_LOCAL_FUNC Do\()\inst\()_SW
    SE_PROLOG
    \inst\()_SW
    SE_EPILOG
.endm

```
which means `Do'ECREATE'_SW` is declared as `'ECREATE'_SW`. And `ECREATE_SW` is also declared right above.

```cpp
linux_sgx/sdk/simulation/assembly/linux/sw_emu.h

.macro ECREATE_SW
    SE0_SW SE_ECREATE
.endm

.macro EADD_SW
    SE0_SW SE_EADD
.endm

.macro EINIT_SW
    SE0_SW SE_EINIT
.endm

.macro EREMOVE_SW
    SE0_SW SE_EREMOVE
.endm
```

Hence, `DoECREATE_SW()`` actually calls `SE_ECREATE()`` function, defined in `u_instructions.cpp`.
```cpp
linux_sgx/sdk/simulation/uinst/u_instructions

uintptr_t _SE0(uintptr_t xax, uintptr_t xbx,
               uintptr_t xcx, uintptr_t xdx,
               uintptr_t xsi, uintptr_t xdi)
{
    UNUSED(xsi), UNUSED(xdi);

    switch (xax)
    {
    case SE_ECREATE:
        return _ECREATE(reinterpret_cast<page_info_t*>(xbx));
    ...
}
```

Finally, we arrive at the simulated `ECREATE`:
```cpp
linux_sgx/sdk/simulation/uinst/u_instructions

// Returns the pointer to the Enclave instance on success.
uintptr_t _ECREATE(page_info_t* pi)
{
    // Address of unprotected SECS
    secs_t* secs = reinterpret_cast<secs_t*>(pi->src_page);

    // Enclave size must be at least 2 pages and a power of 2.
    GP_ON(!is_power_of_two((size_t)secs->size));
    GP_ON(secs->size < (SE_PAGE_SIZE << 1));

    // Associate a secs without base address to an enclave
    CEnclaveSim* ce = new CEnclaveSim(secs);
    void*   addr;

    // `ce' is not checked against NULL, since it is not
    // allocated with new(std::no_throw).
    addr = se_virtual_alloc(NULL, (size_t)secs->size, MEM_COMMIT);
    if (addr == NULL) {
        delete ce;
        return 0;
    }

    // Mark all the memory inaccessible.
    se_virtual_protect(addr, (size_t)secs->size, SGX_PROT_NONE);
    // Assign the base address in SECS to the beginning of protected region
    ce->get_secs()->base = addr;

    // Initialize Enclave Manager and add the Simulated Enclave to it
    CEnclaveMngr::get_instance()->add(ce);
    return reinterpret_cast<uintptr_t>(ce);
}
```
It is interesting how the unprotected SECS is converted to protected SECS, and the line `CEnclaveSim* ce = new CEnclaveSim(secs);` gives us some hints. The constructor for this class is defined in `enclave_mngr.cpp`.
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

    m_enclave_id = gen_enclave_id();
}

```
The line `memcpy_s` copies the memory area from the source `secs` to the destination `m_secs` which is a private variable owned by the enclave. The line `se_virtual_alloc` in `_ECREATE` function is also interesting. We found it in `se_memory.c`. It returns a fixed region of memory upon successful:
```cpp
linux-sgx/common/inc/internal/se_memory.c

void* se_virtual_alloc(void* address, size_t size, uint32_t type)
/*
Releases, decommits, or releases and decommits a region of pages within the virtual address space of the calling process.
@address:A pointer to the base address of the region of pages to be freed. If the dwFreeType parameter is MEM_RELEASE,
        this parameter must be the base address returned by the se_virtual_alloc function when the region of pages is reserved.
@size:	The size of the region of memory to be freed, in bytes.
@type:	Only MEM_RELEASE accepted
        MEM_RELEASE - releases the specified region of pages. After this operation, the pages are in the free state.
@return value:If the function succeeds, the return value is nonzero.If the function fails, the return value is zero.
*/
{
    UNUSED(type);
    void* pRet = mmap(address, size, PROT_READ | PROT_WRITE, MAP_PRIVATE |  MAP_ANONYMOUS, -1, 0);
    if(MAP_FAILED == pRet)
        return NULL;
    return pRet;
}
```

Upon succesful, mmap [[5]][mmap] returns a pointer to mapped area. The file descriptor `fd =-1` indicates that we do not map to file, but map directly to system's memory.

In summary, we can observe:
* Enclaves are managed by Enclave Manager
* OS prepares the `SECS` structure, fills in the `PAGEINFO` with SRCPGE to unprotected SECS and set the `SECINFO` to `PT_SECS`
* The `PAGEINFO` is fed to the simulated `ECREATE`
* The `ECREATE` checks for if the Enclave size must be at least 2 pages and a power of 2
* The `ECREATE` copies page from unprotected SECS to protected SECS only accessible by the class `CEnclaveSim`
* The `ECREATE` creates a range of protected regions equals to the size specified in `SECS`
* The `ECREATE` fills in protected SECS's `BASEADDR` to point to the start of the protected region. From theory, we knew that the unprotected SECS's `BASEADDR` is filled by system software. [[4]][program-reference]

So the simulation does not deviate too much from the theory. Here is the summarized control flow we walked through above

![My helpful screenshot]({{site.url}}/images/simulation-ecreate.png)

[sgx-explain]:https://eprint.iacr.org/2016/086.pdf
[sgx-developer]:https://download.01.org/intel-sgx/linux-1.8/docs/Intel_SGX_SDK_Developer_Reference_Linux_1.8_Open_Source.pdf
[github-link]:https://insujang.github.io/2017-04-14/intel-sgx-sdk-functions-for-enclave-creation/
[program-reference]:https://software.intel.com/sites/default/files/managed/48/88/329298-002.pdf
[mmap]:http://man7.org/linux/man-pages/man2/mmap.2.html
