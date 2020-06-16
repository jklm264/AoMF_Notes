# AoMF: Introduction

- *Southbridge* has been replaced with **platform controller hub** for I/O transfers.

  - *Firewire* aka IEEE 1394 interface

- Control Registers

  - **CR0** flag that enables paging, **CR1** is reserved, **CR2** has linear address that caused a page fault, **CR3** has physical address of the initial structure used for address translation, **CR4** is PAE

- Segmentation vs Paging

  - Segmentation uses Global Descriptor Table Register and Local Descriptor Table Register
  - Physical Page Extension (PAE) - allows processor to support physical address spaces up to 64GB
    - Page directory pointer table -> Page directory -> Page table -> Page offset

- Interrupt Descriptor Table (IDT) points to interrupt service routines 

- *Process* is an instance of a *program* executing in memory, or an *executable* that has been run.

- Difference between a *process* and *thread* is a thread uses shared memory and it is easier to do context switching.

- Virtual Memory 

  - Memory manager - " refers to the operating system’s algorithms for managing the allocation, deallocation, and organization of physical memory."
  - MMU (Memory Management Unit) performs virtual to physical address translation
  - Demand Paging determines which regions are resident in main memory and which are moved to a slower secondary storage (usually in file or partition called *page file* or *swap*)
    - Relies on *locality of reference* ([src](https://stackoverflow.com/questions/16289423/temporal-vs-spatial-locality-with-arrays))
      - Spatial Locality - likelihood of referencing memory if a resource near it was just accessed
      - Temporal Locality - likelihood that a single memory location accessed in time will be access again soon

- I/O Subsystems

  - Purpose is to abstract I/O devices. When communicating via drivers, the devices are treated as files.

  1. Device Drivers - Kernel modules that instruct how a device controls and transfers data by extending kernel capabilities to the new device
     - Also used to implement virtual, software-only devices; **these are memory resident!**
  2. I/O Controls (IOCTLs)
     - Another technique to easily facilitate communication between user and kernel mode; especially for user apps that talk with peripheral devices *or other OS components* ([terminal I/O and kernel extensions](https://en.wikipedia.org/wiki/Ioctl#Uses))