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
  + all kernel logical addr are kernel virtual addr
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
The kernel cannot directly manipulate memory which is not mapped into the kernel's address space; hence, it needs its own virtual addr for any memory it must touch directly.
Before accessing a high-memory page, the kernel must set up an explicit virtual mapping to make that page available in the kernel’s address space. 
Thus, many kernel data structures must be placed in low memory; high memory tends to be reserved for user-space process pages.
