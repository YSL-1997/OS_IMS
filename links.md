## This is a list of useful links supporting my research

[Anatomy of a Linux hypervisor - an introduction to KVM and Lguest](https://developer.ibm.com/tutorials/l-hypervisor/)

[Virtio: An I/O virtualization framework for Linux](https://developer.ibm.com/technologies/linux/articles/l-virtio/)

[Paper: virtio: Towards a De-Facto Standard For Virtual I/O Devices](https://www.ozlabs.org/~rusty/virtio-spec/virtio-paper.pdf)

[Linux virtualization and PCI passthrough](https://developer.ibm.com/tutorials/l-pci-passthrough/)

[Dive into the KVM hypervisor](https://developer.ibm.com/articles/cl-hypervisorcompare-kvm/?mhsrc=ibmsearch_a&mhq=virtio)

[The Video4Linux2 API](https://lwn.net/Articles/203924/)


## Details
////////////////////////////////////////////////////////////////////////////////

// it is this host device that actually uses the queue to be a network card.

// When the user wants to send a packet, it will first call send() syscall,

// the send() syscall traps to the kernel, the kernel calls through the TCP/IP

// stack (multiple layers), and then TCP/IP stack will call into the sn_netdev.c

// to send a packet, and sn_netdev.c will call into sn_host.c to enqueue the packet

// in the ring. Hence, it is the kernel that invokes this code on behalf of the

// user mode syscall.


**Q: What is the relationship between SVA, shared queue, and ENQCMD?**

A: When you enqueue a cmd, typically what you're enqueuing is a descriptor. When you want to send data to the device, you do not send the data directly; instead, you enqueue a descriptor that has a pointer to the data you want to send, as well as the length field indicating how big the data is (and maybe some metadata). So, what you are enqueuing is the descriptor that says "hey, here's the data I want you (the device) to operate on". The challenge is that the descriptor has a pointer in it - if you did not have this enqueue command, but only have normal memory operations (store, move, etc.), then when you enqueue an address, user mode is actually enqueuing a user space virtual address. This leads to the fact that when the device does the dequeue, all it could get is the user space virtual address. Since the device will treat the virtual address as the physical address, it will not work because it is the wrong address.

So SVA in the ENQCMD says "we are going to enqueue a virtual address, and we are also going to enqueue an identifier indicating that which address space this virtual address comes from, and which process it belongs to - this allows the device to lookup in the page table to translate the actual physical address." So the device can use an IOMMU to properly translate that virtual address, get the physical address and retrieve the actual data. 

By using DVM, device dequeuing virtual address can skip the address translation.

**Q: What is the use of ```asm volatile("" ::: "memory")``?**

A: It is the compiler barrier. When you implement a shared data structure, you care a lot about the order in which you updating memory. Suppose you have some data that you want to add to the ring buffer, and the tail pointer of where the data is - you want to make sure that you update the data first, then update the tail pointer. The problem is that, the compiler in C are likely to reorder the operations if they think nobody is looking. Compiler in C knows nothing about multi-threaded code, so it may assume that it is the only thread accessing the data, and for optimizations, it may reorder operations. Hence, what the compiler barrier does is that "after executing compiler barrier, you cannot trust anything in memory, since it may have changed; meaning that if you dependent on anything in memory, you have to reload it afterwards." In other words, it makes sure that anything that happened (actually reaches memory) before the compiler barrier - after the compiler barrier, the compiler barrier makes sure that anything happens (reaches memory) needs refetching from memory since it may have changed. 

For instance, if you cached a variable in the register, you need to refetch it from memory because its value of that variable might have changed.

In short, compiler barrier says "everything might have changed, we cannot assume anything about what is in memory. It is needed to sort of make sure that the order of the operations are ordered properly."

**Q: What is the grammar of ```asm```?**

A: ```asm``` says what comes in the parenthesis is going to be **assembly code**. 

The first set of quotes("") contains the assembly code itself - you give it a sequence of assembly instructions, and C compiler passes them to the assembler. 

If you want to take any C variables and put them into register (e.g. %rax comes from local_var1, %rbx comes from local_var2, etc.), then you can pass them in after the first colon (:); which is to make sure that before executing the assembly code, the variables are in these registers.

After the second colon, you specify where to copy things, i.e. take what is in a register and put it into some variables.

Example: ```asm volatile("rdtsc" : "=a" (c), "=d" (d));```

We are going to execute ```rdtsc``` instruction, and as output, we want to take what is in %eax and copy it to variable c, and copy what is in %edx to variable d.

**Q: Implementation of shared queue?**
