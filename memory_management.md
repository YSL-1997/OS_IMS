## Memory Management in Linux Kernel

[Figure 15-1. (p. 414)](https://static.lwn.net/images/pdf/LDD3/ch15.pdf)

### Address types

+ user virtual addr (regular addresses seen by user-space programs)
+ physical addr (used between processor and system’s memory)
+ bus addr (addresses used between peripheral buses and memory)
+ kernel logical addr (normal address space of the kernel)
  + ```kmalloc()``` 
  + linear, direct physical mapping (logical addr and associated physical addr differ by a constant offset)
  + stored in variables of type ```unsigned long``` or ```void *```
  + all kernel logical addr are kernel virtual addr (KLA < KVA)
+ kernel virtual addr (a mapping from a kernel-space address to a physical address) 
  + ```vmalloc()``` and ```kmap()```
  + stored in pointer variables
  + not necessarily linear

```
#include <asm/page.h>

__pa() // given a logical addr, return its physical addr
__va() // given a physical addr (in low-memory pages), map it back to logical addr

```

### Physical Address & Pages
```
#include <asm/page.h>

PAGE_SIZE // defined in <asm/page.h>, returns page size
PAGE_SHIFT // tells how many bits must be shifted right to get PFN 
```

### High and Low memory

What's the difference between kernel virt addr and kernel logical addr?

- High memory: kernel virtual addr (reserved for user-space process pages)
- Low memory: kernel logical addr (stores kernel data structures)

On 32-bit systems, kernel can address 4 GB of memory. Typically, kernel splits such 4GB of memory into 3 GB (user space) and 1 GB (kernel space). 
However, the biggest consumer of kernel address space is **virtual mappings for physical memory**.
The kernel cannot directly manipulate memory which is not mapped into the kernel's address space; hence, it needs its own **virtual addr for any memory it must touch directly**.
Before accessing a high-memory page, the kernel must set up an explicit virtual mapping to make that page available in the kernel’s address space. 
Thus, many kernel data structures must be placed in low memory; high memory tends to be reserved for user-space process pages.

### Memory Map and struct page

Historically, kernel mapped logical addr to physical addr. However, logical addr is not available for high memory; hence, BIG problem.

Therefore, kernel functions dealing with memory are using pointers to ```struct page (linux/mm.h)``` instead.

What is ```struct page```? This data structure keeps track of almost everything kernel needs to know about physical memory.

```
#include <linux/mm.h>

// some fields in struct page:
struct page {
    atomic_t count; // # of references to this page. 
                    // When the count drops to 0, the page is returned to the free list.

    void *virtual; // kernel virtual address of the page, if it is mapped; NULL, otherwise. 
                   // Lowmemory pages are always mapped; high-memory pages usually are not. 
                   // This field does not appear on all architectures; 
                   // it generally is compiled only where the kernel virtual address of a page 
                   // cannot be easily calculated. If you want to look at this field, the proper 
                   // method is to use the page_address macro.
    
    unsigned long flags; // A set of bit flags describing the status of the page. 
                         // e.g. PG_locked: the page has been locked in memory.
                         // e.g. PG_reserved: prevents the memory management system 
                         //                   from working with the page at all.
```

The kernel maintains one or more arrays of struct page entries that track all of the physical memory on the system. e.g. a single array ```mem_map```.

Some functions and macros for translating between struct page poiters and virtual addresses:
```
// This macro, defined in <asm/page.h>, takes a kernel logical address and returns
// its associated struct page pointer. Since it requires a logical address, it does not
// work with memory from vmalloc or high memory.
struct page *virt_to_page(void *kaddr);

// Returns the struct page pointer for the given page frame number. If necessary,
// it checks a page frame number for validity with pfn_valid before passing it to
// pfn_to_page.
struct page *pfn_to_page(int pfn);

// Returns the kernel virtual address of this page, if such an address exists. For high
// memory, that address exists only if the page has been mapped. This function is
// defined in <linux/mm.h>. In most situations, you want to use a version of kmap
// rather than page_address
void *page_address(struct page *page);


#include <linux/highmem.h>
void *kmap(struct page *page);
void kunmap(struct page *page);
// kmap returns a kernel virtual address for any page in the system. 
// For low-memory pages, it just returns the logical address of the page; 
// for high-memory pages, kmap creates a special mapping in a dedicated part of the kernel address space.
// Mappings created with kmap should always be freed with kunmap; a limited
// number of such mappings is available, so it is better not to hold on to them for
// too long. kmap calls maintain a counter, so if two or more functions both call
// kmap on the same page, the right thing happens. Note also that kmap can sleep
// if no mappings are available.


#include <linux/highmem.h>
#include <asm/kmap_types.h>
void *kmap_atomic(struct page *page, enum km_type type);
void kunmap_atomic(void *addr, enum km_type type);
// kmap_atomic is a high-performance form of kmap. Each architecture maintains a
// small list of slots (dedicated page table entries) for atomic kmaps; a caller of
// kmap_atomic must tell the system which of those slots to use in the type argument. 
// The only slots that make sense for drivers are KM_USER0 and KM_USER1 (for
// code running directly from a call from user space), and KM_IRQ0 and KM_IRQ1 (for
// interrupt handlers). Note that atomic kmaps must be handled atomically; your
// code cannot sleep while holding one. Note also that nothing in the kernel keeps
// two functions from trying to use the same slot and interfering with each other
// (although there is a unique set of slots for each CPU). In practice, contention for
// atomic kmap slots seems to not be a problem.
```

### Page tables
The processor's mechanism for translating virtual addr to physical addr.

### Virtual Memory Areas
+ What is VMA? 
  + The kernel data structure used to manage distinct regions of a process's address space.
+ What VMA represents?
  + A homogeneous region in virtual memory of a process
  + A contiguous range of virtual addresses that have the same permission flags, and are backed up by the same object (file, or swap space)
  + A memory object with its own properties.
+ What does VMA consist of?
  + 1 area for **text** (program's executable code)
  + multiple areas for **data** (initialized data, uninitialized data (BSS - block started by symbol), program stack)
  + 1 area for **each active memory mapping**
+ How to see the memory areas of a process?
  + By looking in /proc/<pid/maps>. /proc/self refers to the current process
  + The fields in each line are: *start-end perm offset major:minor inode image*

The fields on the line above are the fields in ```struct vm_area_struct```.

### mmap
Mapping a device means associating a range of user-space addresses to device memory. Whenever the program reads or writes in the assigned address range, it is actually accessing the device. 
The kernel can manage virtual addresses only at the level of page tables; therefore, the mapped area must be a multiple of ```PAGE_SIZE``` and must live in physical memory starting at an address that is a multiple of PAGE_SIZE. 

To implement mmap, the driver only has to build suitable page tables for the address range and, if necessary, replace ```vma->vm_ops``` with a new set of operations.

The job of building new page tables to map a range of physical addresses is handled by ```remap_pfn_range``` and ```io_remap_page_range```.
- ```remap_pfn_range``` is intended for situations where pfn refers to actual system RAM
- ```io_remap_page_range``` should be used when ```phys_addr``` points to I/O memory.

Prototypes of both functions:
```
/**
 * @param vma: The virtual memory area into which the page range is being mapped.
 * @param virt_addr: The user virtual address where remapping should begin. 
 *                   The function builds page tables for the virtual address range 
 *                   between virt_addr and virt_addr+size.
 * @param pfn: The page frame number corresponding to the physical address to which the virtual address should be mapped. 
 *             PFN is simply the physical address right-shifted by PAGE_SHIFT bits. 
 *             For most uses, the vm_pgoff field of the VMA structure contains exactly the value you need. 
 *             The function affects physical addresses from (pfn<<PAGE_SHIFT) to (pfn<<PAGE_SHIFT)+size.
 * @param size: The dimension, in bytes, of the area being remapped.
 * @param prot: The “protection” requested for the new VMA. The driver can (and should) use
 *              the value found in vma->vm_page_prot.
 *
 * @return: 0 on success, negative error code on error.
 */
int remap_pfn_range(struct vm_area_struct *vma,
                    unsigned long virt_addr, unsigned long pfn,
                    unsigned long size, pgprot_t prot);
int io_remap_page_range(struct vm_area_struct *vma,
                    unsigned long virt_addr, unsigned long phys_addr,
                    unsigned long size, pgprot_t prot);
```
