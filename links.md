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

A: When you enqueue a cmd, typically what you're enqueuing is a descriptor. When you want to send data to the device, you do not send the data directly; instead, you enqueue a descriptor that has a pointer to the data you want to send, as well as the length field indicating how big the data is (and maybe some metadata). So, what you are enqueuing is the descriptor that says "hey, here's the data I want you (the device) to operate on". The challenge is that the descriptor has a pointer in it - if you did not have this enqueue command, but only have normal memory operations (store, move, etc.), then when you enqueue an address, user mode is actually enqueuing a user space virtual address. This leads to the fact that when the device does the dequeue, 
