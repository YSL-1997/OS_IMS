## Questions:
1. What is hypervisor? Virtual machine like VMWare? Why we need that virtual machine? Is that a hardware simulator? Can it simulate a hardware?
2. Do I need to learn PCI?
3. While reading the paper, I noticed tons of problems regarding to Direct I/O, and memory virtualization


## Useful links:
+ [Lecture of Virtual Machines by UCSD](http://www.cs.cmu.edu/~dga/15-440/F10/lectures/vm-ucsd.pdf)
+ [CSE 291 Virtualization @UCSD](https://cseweb.ucsd.edu/~yiying/cse291j-winter20/reading/)
+ [Lecture of Virtualizing Memory](https://cseweb.ucsd.edu/~yiying/cse291j-winter20/reading/Virtualize-Memory.pdf)
+ [Nested page tables](https://www.anandtech.com/show/2480/10)
+ [Virtualization and Memory Management on YouTube](https://www.youtube.com/watch?v=e6Zc4hcJjAY)
+ [I/O virtualization 1](https://www.youtube.com/watch?v=2yhcITe_xa0)
+ [I/O virtualization 2](https://www.youtube.com/watch?v=gwMrdCONERo)

## Memory virtualization
Several terms:

+ Guest OS: on a machine, we could install multiple OSs, which are called guest OSs.
+ Guest virtual addr: the virtual addr used by the guest OS
+ Guest physical addr: the physical addr used by the guest OS (not the actual physical address on Host)
+ Host OS: the OS running on the hardware
+ Host physical addr: the actual physical addr of the memory

#### What on earth is Memory Virtualization?

It is mapping the guest virtual addr to the guest physical addr, and mapping the guest physical addr to host physical addr

#### Two ways of memory virtualization

Two approaches - software approach; hardware approach.

+ Software approach - **hypervisor**
	+ Shadow Paging (two tables)
		+ details: use a shadow page table. First table: (GVA -> GPA), and the shadow of the previous page table - (GPA -> HPA)
		+ Use the table (GVA -> GPA) to prevent guest OS directly access the hardware PT.
+ Hardware approach
	+ Nested Page Table (AMD processors)
	+ Extended Page Table (Intel processors)
	+ details: 
		+ For the guest OS, there's one page table (GVA -> GPA), then that page table is linked to another page table (GPA -> HPA), maintained by the virtual machine.
		+ One more level of virtualization (one more virtual machine on the virtual machine, which could be viewed as the virtual host machine), 

+ What's the difference?
	+ software approach - the page tables are maintained by the software - hypervisor
	+ hardware approach - the page tables are maintained by the virtual machine itself.

## Outline

+ Direct I/O
	+ **Advantage**: the best performance **I/O virtualization** method
	+ **Problem**: requires hypervisor to statically pin the entire [guest memory](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/techpaper/perf-vsphere-memory_management.pdf) (refers to the memory that is visible to the guest OS)
	+ **Effect**: hinders efficiency of memory management
	+ **Solution**: virtual IOMMU (vIOMMU)

+ vIOMMU
	+ **Advantage**: emulation of its DMA remapping capability carries sufficient info about guest DMA buffers, allowing the hypervisor to do fine-grained pinning of guest memory
	+ **Problem**: emulation cost too high
	+ **Effect**: cannot reliably eliminate static pinning in direct I/O
	+ **Solution**: coIOMMU - (vIOMMU + cooperative DMA buffer tracking mechanism)

+ coIOMMU
	+ a new vIOMMU architecture for efficient memory management with a cooperative DMA buffer tracking mechanism. 
	+ **cooperative DMA buffer tracking**:
		+ a dedicated interface for hypervisor and guest to **exchange DMA buffer info** over a **shared DMA tracking table (DTT)**.
			+ orthogonal to the costly DMA remapping interface
		+ smart pinning
		+ lazy pinning
		+ (to minimize the impact on the performance of direct I/O)
	+ **Result**: dramatically improves the efficiency of memory management in wide direct I/O usages with negligible cost. Moreover, the desired semantics of DMA remapping can be sustained when
cooperative tracking is enabled alongside.


## Introduction
#### Direct I/O 

+ the best performance [**I/O virtualization**](https://gcallah.github.io/OperatingSystems/VirtualIO.html) method
+ cornerstone capability in data centers and clouds
+ user could directly interact with I/O devices without the intervention from software intermediary.
+ **Problem**: requires hypervisor to statically pin the entire guest memory
+ **Effect**: hinders efficiency of memory management
+ **Solution**: virtual IOMMU

#### IOMMU

+ prevents DMA attacks in Direct I/O by providing the **DMA remapping**.
+ Each assigned device is associated with an IOMMU page table (IOPT), configured by the hypervisor in a way that **only the memory of the guest that owns the device is mapped**.
+ IOMMU walks the IOPTs to validate and translate DMA requests
	+ achieving inter-guest protection among directly assigned devices.

However, most devices do not tolerate DMA faults - guest buffers must be pinned (kind of like but not exactly the same as mapped) in host memory and mapped in IOMMU page table before we perform DMAs. 

#### Problem with IOMMU

But, the hypervisor does not know which pages are mapped by the guest when it is eliminated from the direct I/O path. Hence, it has to pin the entire guest memory upfront, i.e. a static pinning.
- This heavily hinders the efficiency of memory management and worsens memory utilization, why? 
- Because pinned pages cannot be reclaimed for other purposes.

#### Virtual IOMMU

+ providing vIOMMU to the guest 
	+ primary purpose: help the guest protect itself against buggy drivers
	+ allows fine-grained pinning of guest memory for efficient memory management
+ Remember IOMMU provides **DMA remapping** to prevent DMA attacks?
	+ The hypervisor emulates the DMA remapping interface by:
		+ i) walking the virtual IO page table => identify the affected buffers
		+ ii) pinning and unpinning the buffers in the host memory
		+ iii) mapping and unmapping them in the physical IOMMU to enforce protection as desired by the guest.
	+ Naturally, the emulation leads to a fine-grained pinning scheme if the guest always uses the virtual IOMMU to remap its DMA buffers.

#### Problem with vIOMMU
It is not a reliable solution for fine-grained pinning, only used in limited circumstances.
+ Virtual DMA remapping are disabled by most guests in public cloud.
+ Because significant emulation cost may be incurred due to **frequent mapping operations in the guest**. Such cost could be **alleviated** through side-core emulation, or para-virtualized extension. (Well, they still have some new problems, see below):
	+ Side-core emulation (security issues)
		+ requires an additional CPU core to perform the emulation
		+ can only achieve optimal performance with deferred IOTLB invalidation => compromised security.
	+ Para-virtualized extension (performance issues)
		+ reduces the virtualization overhead with optimized interfaces
		+ but still involves large number of VM-exits at the time of guest DMA mappings/unmappings => limiting the performance.

Therefore, vIOMMU is only used when intra-guest protection is valued over the overhead of DMA remapping. (slow, but secure)

#### What the authors say

Mixing the requirements of protection and pinning, through the same costly DMA remapping interface, is needlessly constraining. 
- Protection is a guest requirement.
- Pinning is for host memory management. 
The two do not always match, thus favoring one may easily break the other. Instead, they aim to provide a reliable solution for fine-grained pinning by decoupling it from protection.

#### Here comes coIOMMU
+ A new vIOMMU architecture
+ Helps hypervisor achieve efficient memory management in direct I/O.
+ A mechanism for cooperative DMA buffer tracking, orthogonal to (don't quite understand) the costly DMA remapping interface
+ Allows the hypervisor and guest to communicate over a DMA tracking table (DTT) located in a shared memory region.
	+ 
