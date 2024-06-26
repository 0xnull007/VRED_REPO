### What is Kernel ?

An operating system kernel is the central part of an operating system that manages the resources of the computer and provides a platform for other software to run. It is the lowest level of the operating system, and it communicates directly with the computer's hardware.

The kernel is responsible for managing the interactions between processes and coordinating their access to resources such as memory, input/output devices, and other hardware. It performs tasks such as scheduling processes, allocating memory, and managing input/output operations.

Consider Pakistan-Centric View, In this context, individual processes can be thought of as provincial governments, while the kernel acts as the federal government, responsible for managing relations between the processes and coordinating their actions. The kernel is a critical component of the operating system, as it is responsible for ensuring that the processes running on the system are able to function correctly and efficiently.

### Kernel Resources

Kernel Manages Resources between processes. The Kernel determines whether code can access these resources based on the privileges granted to the process running the code. The kernel controls access to these resources and grants permission to processes as needed. For example, a process may need to request access to a hardware peripheral through the kernel, which will then determine whether to allow the access based on the privileges of the process and the availability of the resource.

>[!info]
>Granting permission here means kernel perform the task asked by user process. Such as reading the bytes from the file via read syscall

But few Resources are only available to kernel and
Examples of kernel-only resources:
-   The hlt instruction, which shuts off CPU computation.
-   The in and out instructions for interacting with hardware peripherals.
-   Special registers such as cr3 (Control Register 3) and MSR_LSTAR (Model-Specific Register, Long Syscall Target Address Register).

### What makes a kernel special?

In a computer system, the CPU (central processing unit) tracks a privilege level that controls access to resources. This privilege level is generally split into "rings" of access, with higher rings having more privileges and lower rings having fewer privileges.

![[Pasted image 20221228193811.png|240]]

-   Ring 3: This is the lowest privilege level and is often referred to as "userspace." Processes running at this level are typically those that are initiated by user applications and have very restricted access to system resources.
  
-   Rings 2 and 1: These rings are generally unused in modern operating systems.
  
-   Ring 0: This is the highest privilege level and is referred to as "kernel mode" or "supervisor
   mode." It is reserved for the operating system kernel and other system-level processes. Processes running at this level have unrestricted access to all system resources and can perform tasks such as modifying memory and accessing hardware peripherals.

The CPU tracks the current privilege level to ensure that processes only have access to the resources they are authorized to use. When a process running at a lower privilege level (such as Ring 3) needs to access a restricted resource, it must request access through the kernel. The kernel will then determine whether to grant the request based on the privileges of the process and the availability of the resource.

Supervisor mode (also known as kernel mode or Ring 0) can pose a security risk when running multiple operating systems on a single physical machine using virtualization technology.

In the early 2000s, one solution to this issue was to force the kernel of the virtual machine (VM) into Ring 1, which has fewer privileges than Ring 0. This prevented the VM kernel from having unrestricted access to the host's physical hardware, but it also required the use of costly and complex emulation methods to simulate some Ring 0 actions of the guest operating system.

A more modern solution is to use a special privilege level known as hypervisor mode, or Ring -1. This level is used by the hypervisor, which is a software layer that sits between the hardware and the operating systems and manages the virtualization of the hardware. When a VM kernel attempts to perform a sensitive Ring 0 action, the hypervisor is able to intercept the request and handle it in the host operating system, rather than allowing the VM kernel to execute the action directly.

This allows the VM kernel to operate in a more isolated environment, while still providing the necessary resources and support for the guest operating system to function. It also helps to improve security by limiting the access of the VM kernel to sensitive system resources and preventing it from potentially damaging the host operating system or other VMs running on the same physical machine.

>[!Note]
>The Ring system is true for "amd64" architecture and may not be true for others.


### Different OS Models

An operating system kernel can be classified into one of three categories based on its design: monolithic, microkernel, or hybrid.

-   **Monolithic kernel**: In this type of kernel, there is a single, unified kernel binary that handles all OS-level tasks. Drivers, which are responsible for interacting with hardware devices, are implemented as libraries that are loaded into this kernel binary. Examples of operating systems that use a monolithic kernel include Linux and FreeBSD.
  
-   **Microkernel**: In a microkernel-based operating system, there is a small "core" kernel binary that provides basic inter-process communication and interacts with the hardware. Drivers are implemented as normal userspace programs that have slightly special privileges. Examples of operating systems that use a microkernel include Minix and seL4.
  
-   **Hybrid kernel**: A hybrid kernel is a combination of a microkernel and a monolithic component. It combines the benefits of both designs, allowing for a more flexible and modular kernel architecture. Examples of operating systems that use a hybrid kernel include Windows NT and macOS.

### Linux Model

In a Linux operating system, drivers are typically implemented as kernel modules, which are loadable pieces of code that can be added to or removed from the kernel at runtime. Kernel modules provide a way to extend the functionality of the kernel without having to rebuild it each time a new device or feature is added.

When a device is connected to a Linux system, the kernel will scan for a matching driver module that is capable of interacting with the device. If a matching driver is found, it will be loaded into the kernel and used to communicate with the device. If no matching driver is found, the device may not be functional until a suitable driver is installed.

Kernel modules can be loaded or unloaded dynamically as needed, allowing for a more flexible and modular approach to device support. This is one of the key features of the Linux operating system that enables it to support a wide range of hardware devices and configurations.

### Drivers in Linux

In a Linux operating system, drivers are implemented as kernel modules and run in Ring 0, which is the highest privilege level and is reserved for the operating system kernel and other system-level processes. This means that drivers have unrestricted access to all system resources and can perform tasks such as modifying memory and accessing hardware peripherals.

This is necessary because drivers are responsible for interacting with hardware devices, which often require access to low-level system resources. However, it is important to note that not all kernel modules are drivers; some may provide other types of functionality such as filesystem support, networking support, or security features.

It's also worth noting that the concept of privilege levels and rings is not specific to Linux; it is a general feature of modern operating systems that is used to control access to resources and protect the system from unauthorized access or misuse. In other operating systems, the names and specific details of these privilege levels may vary, but the basic concept is similar.

![[Pasted image 20221228194602.png]]

### Switching between rings 

The syscall instruction is a mechanism for allowing processes running in userspace (Ring 3) to request services or resources from the kernel (Ring 0). The process of calling a syscall can be summarized as follows:

1.  At bootup, the kernel sets the MSR_LSTAR (Model-Specific Register, Long Syscall Target Address Register) to point to the address of the syscall handler routine.                                                                           
   
2.  When a userspace process needs to access a kernel resource, it calls the syscall instruction. This causes a privilege level switch from Ring 3 to Ring 0.
   
3.  The control flow jumps to the address stored in MSR_LSTAR, which is the starting address of the syscall handler routine. The return address (the address to which control should be returned after the syscall handler routine completes) is saved to the rcx register.
   
4.  The syscall handler routine performs the requested service or resource access and returns control to the calling process when it is finished.
   
5.  When the kernel is ready to return control to the userspace process, it calls the appropriate return instruction (such as sysret for syscall). This causes a privilege level switch from Ring 0 back to Ring 3.
   
6.  The control flow then jumps to the address stored in rcx, which is the return address saved earlier. This completes the syscall process and returns control to the userspace process.

### Kernel-Userspace Relationship

In a modern operating system, each process has its own virtual memory space, which is a mapping of virtual addresses to physical memory addresses. Virtual memory allows processes to use more memory than is physically available by temporarily storing some of the data on disk and swapping it in and out as needed.

In many operating systems, including Linux, userspace processes (processes running in Ring 3) are typically assigned virtual memory addresses in the lower range of the address space, while the kernel (which runs in Ring 0) is assigned virtual memory addresses in the upper range of the address space. This separation helps to protect the kernel from unauthorized access by user processes and to ensure that kernel memory is only accessible from Ring 0.

When a system call is made, the privilege level is switched from Ring 3 to Ring 0, but the virtual memory mapping is not changed. This means that the process can still access its own virtual memory, but it does not have access to the kernel's virtual memory space. The kernel's memory is only accessible from Ring 0, so the process must make a system call to request access to kernel resources.

![[Pasted image 20221228195958.png|500]]

### Kernel Vulnerabilities

Code in the kernel is just code and is subject to the same types of vulnerabilities as code running in other contexts. It is also important to note that the kernel has access to a wide range of system resources and can perform tasks such as modifying memory and accessing hardware peripherals. This means that vulnerabilities in the kernel can potentially allow an attacker to gain access to sensitive system data or to execute arbitrary code with the privileges of the kernel. As a result, it is important to take steps to secure the kernel and to prevent unauthorized access or misuse.

### Attack life Cycle

kernel exploits can come from a variety of sources and can be used to achieve a range of malicious objectives. Here is a summary of the attack lifecycle for kernel exploits:

1.  Attack sources: Kernel exploits can come from a few different directions, including:
-   From the network: Remotely-triggered exploits, such as packets of death, can be used to attack the kernel over the network. These types of attacks are relatively rare, but can be particularly dangerous if successful.

-   From userspace: Vulnerabilities in syscall and ioctl handlers (functions that allow processes to request services or resources from the kernel) can be exploited from within a userspace process. This can be done through a sandboxed environment, such as a web browser, or through a compromised application.

-   From devices: Kernel exploits can also be launched from attached devices, such as USB hardware. These types of attacks can be difficult to detect and can allow an attacker to gain access to the kernel even if the system is otherwise secure.

2.  Attack objectives: Once an attacker has gained access to the kernel, they can use the exploit to achieve a number of malicious objectives, including:
-   Act on userspace: Kernel exploits can be used to perform privilege escalation, which allows an attacker to gain elevated privileges within the system. They can also be used to install rootkits, which are malicious software programs that allow an attacker to maintain control of the system even after the initial exploit has been discovered and removed.

-   Get more access: Kernel exploits can also be used to gain access to other parts of the system, such as trusted execution environments, which can allow the attacker to further escalate their privileges or to attack other systems on the network.

### Kernel Modules

A kernel module is a piece of code that can be loaded into the kernel of an operating system at runtime. It is similar to a userspace library, in that it is a compiled binary file that contains code that can be accessed and executed by other programs. However, there are some key differences between kernel modules and userspace libraries:

-   File format: Kernel modules are typically compiled as ELF (Executable and Linkable Format) files, just like userspace libraries. However, kernel modules use the ".ko" file extension instead of ".so" (shared object).

-   Address space: When a kernel module is loaded, it is placed into the address space of the kernel, rather than into the address space of a userspace process. This means that the code in the module runs with the same privileges as the kernel, and has unrestricted access to all system resources.

-   Functionality: Kernel modules are used to implement a wide range of functionality in an operating system, including device drivers (which allow the kernel to communicate with hardware devices), filesystems (which provide a way to organize and access files on the system), networking functionality (such as support for network protocols and device drivers), and various other types of functionality.

>[!warning]
>Kernel Modules in Linux run with Ring 0 Privilage level so be careful writing them and exploiting them on real system because they can cause your hardware rather useless

### Interacting with Modules

Historically, it was possible for kernel modules to add system call entries to the kernel's system call table, which is a data structure that contains a list of all the system calls that are supported by the kernel. 

Modifying the system call table allowed kernel modules to add new system calls to the kernel, which could then be used by userspace processes to access new functionality or resources. However, this approach had some significant drawbacks, including:

-   Unsupported: Modifying the system call table is explicitly unsupported in modern kernels, and can cause instability or other problems if done incorrectly.
   
-   Security risks: Adding new system calls to the kernel can potentially introduce security vulnerabilities, as it allows processes running in userspace to access kernel resources that may not have been adequately tested or secured.

-   Compatibility issues: Adding new system calls to the kernel can also cause compatibility issues with other software that expects the kernel to support a certain set of system calls.   

As a result, modern kernels generally do not allow kernel modules to modify the system call table, and other approaches are typically used to add functionality to the kernel. These approaches may include using other types of kernel interfaces, such as ioctl (input/output control) or sysfs (system file system), or by implementing new system calls as part of the kernel itself.

#### Modules and Interupts

Theoretically it is possible for a kernel module to register an interrupt handler by using the LIDT (Load Interrupt Descriptor Table) and LGDT (Load Global Descriptor Table) instructions, and to be triggered by an interrupt instruction, such as int 3 or int 1. 

>[!info]
>Interrupts are signals that are sent to the processor to request immediate attention, and can be used to perform various types of processing or to handle exceptions or other events.

One-byte interrupt instructions, such as int 3 (0xcc) and int 1 (0xf1), can be useful for hooking because they can be easily placed in code to trigger an interrupt without consuming a lot of space. Int 3 is normally used to cause a SIGTRAP signal, which is used for debugging purposes, but it can be hooked by a kernel module to perform other actions. Int 1 is normally used for hardware debugging, but it can also be hooked by a kernel module.

A kernel module can also hook the Invalid Opcode Exception interrupt, which is triggered when the processor encounters an invalid instruction in the instruction stream. This can be used to implement custom instructions in software, or to retrofit security measures onto an existing system. 

It is worth noting that these approaches to hooking interrupts and implementing custom instructions are generally considered to be bespoke interaction methods, meaning that they are specific to a particular system or use case, and may not be widely supported or recommended

#### Modules and Files

The most common way of interacting with kernel modules is via file, using special file interfaces that are provided by the operating system. There are several common locations for these interfaces, including:

-   /dev: This directory contains mostly traditional devices, such as block devices (e.g., hard drives, CDs) and character devices (e.g., serial ports, audio devices). Kernel modules that implement device drivers will often register a file in this directory to allow userspace programs to access the device.
   
-   /proc: This directory started out in System V Unix as a way to provide information about running processes. In Linux, it has been expanded to include a wide range of kernel interfaces, including virtual files that provide information about system statistics, process details, and other types of data.

-   /sys: This directory was introduced as a more organized way to provide non-process information about the kernel and the system. It is intended to be a more structured and consistent interface for interacting with the kernel, and is used to expose a variety of system parameters and statistics.   

A kernel module can register a file in one of these locations to provide an interface for userspace programs to interact with the module. Userspace code can use the standard file API (e.g., open(), read(), write(), close()) to access and manipulate the file, which in turn allows the module to perform the desired actions or provide the requested information.

##### File read() and write()

One interaction mode for kernel modules is to handle the read() and write() system calls for a file that is exposed by the module. This allows userspace programs to access and manipulate the file using the standard file API.

From kernel space, the module can define functions such as "device_read()" and "device_write()" to handle these system calls when they are made on the exposed file. These functions typically have the following signature:
```c
static ssize_t device_read(struct file *filp, char *buffer, size_t length, loff_t *off) 
static ssize_t device_write(struct file *filp, const char *buf, size_t len, loff_t *off)
```


The "filp" parameter is a pointer to a "struct file" object that represents the open file, and the "buffer" and "buf" parameters are pointers to the memory locations where the data being read or written is stored. The "length" and "len" parameters indicate the number of bytes to be read or written, and the "offset" and "off" parameters are pointers to variables that track the current position within the file.

From user space, a program can use the read() and write() system calls to access and manipulate the file exposed by the module. For example:

```c
fd = open("/dev/pufcit", 0) read(fd, buffer, 128);
```

This code would open the file **"/dev/pufcit"** in read-only mode, and then use the read() system call to read up to 128 bytes of data into the "buffer" memory location.

This interaction mode is useful for modules that deal with streams of data, such as audio or video data, as it allows the module to handle the data in a continuous stream rather than in discrete blocks. It is also useful for modules that need to support non-blocking I/O, as the read() and write() functions can return a partial count of the requested data if the operation cannot be completed immediately

#### Ioctl Handlers

Input/Output Control provides a much more flexible interface.

```c
/*From kernel space:*/
static long device_ioctl(struct file *filp, unsigned int ioctl_num, unsigned long ioctl_param)
```

```c
/*From user space:*/

int fd = open("/dev/pufcit", 0);

ioctl(fd, COMMAND_CODE, &custom_data_structure);
```

Useful for setting and querying non-stream data (i.e., webcam  resolution settings as opposed to webcam video stream).

[Follow this link to learn how to compile the kernel modules](https://tldp.org/LDP/lkmpg/2.6/html/index.html)

### Memory Corruption in Kernel

Kernel memory is a special type of memory that is reserved for the kernel's use. It is typically located in a separate address space from user-space memory, which is used by user programs and processes. This separation is important for security and stability, as it prevents user programs from directly accessing or modifying kernel memory, which could potentially lead to corruption or other problems.

The `copy_to_user` and `copy_from_user` functions are used to copy data between kernel-space and user-space memory. These functions are typically used when a kernel module or driver needs to access data that is stored in user-space memory, or when a user program needs to access data that is stored in kernel-space memory.

When using these functions, it is important to be careful and ensure that the data being accessed is valid and correctly formatted. If the data is corrupted or otherwise invalid, it could cause problems such as system crashes, bricked systems, or even privilege escalation attacks. Therefore, it is important to carefully handle all user data and use these functions to ensure the integrity of the kernel memory. Kernel code is just code! Memory corruptions, allocator misuse, etc, all happen in the kernel! What can we do once we have exploited the Vulnerability?

### Kernel Race Conditions

One potential issue with kernel modules is that they can be prone to race conditions, which can occur when multiple threads or processes try to access shared resources simultaneously. This can lead to problems such as resource contention, data corruption, or even system crashes.

For example, if two devices open the file `/dev/pufcit` simultaneously, there is a risk that the resources associated with this file could be swapped or disappear mid-execution, leading to unexpected behavior or errors. Similarly, if the kernel module `make_root.ko` is removed while the file `/proc/pufcit` is open, it could cause problems such as system crashes or data corruption.

To prevent these types of issues, it is important to carefully design and implement kernel modules to ensure that they are thread-safe and do not suffer from race conditions. This may involve using synchronization mechanisms such as mutexes or semaphores to ensure that shared resources are accessed in a controlled and consistent manner.

### Privilege Escalation 

Kernel keeps track of various information about each process, including its privilege level, memory usage, and other metadata.

One way that the kernel stores this information is by using a data structure called `struct task_struct`. This structure is defined in the kernel's source code and is used to represent a process in the kernel. It contains a wide range of fields that store information about the process, such as its process ID, priority level, memory usage, and other metadata.

Another data structure that is often used to store information about process privileges is `struct cred`, which stands for "credentials". This structure is used to store information about the user and group IDs of a process, as well as other security-related information such as the capabilities of the process.

Together, `struct task_struct` and `struct cred` are used by the kernel to track the privileges and other data of every running process. This information is used by the kernel to manage and schedule processes effectively, and to ensure that each process is only able to access the resources and perform the actions that it is allowed to.

>[!Note]
>**struct cred** is member of struct **task_struct** 

```c
struct task_struct {
    struct thread_info      thread_info;
    /* -1 unrunnable, 0 runnable, >0 stopped: */
    volatile long           state;
    void                    *stack;
    atomic_t                usage;
  // ...
    int                     prio;
    int                     static_prio;
    int                     normal_prio;
    unsigned int            rt_priority;
    struct sched_info       sched_info;
    struct list_head        tasks;
    pid_t                   pid;
    pid_t                   tgid;
    /* Process credentials: */
    /* Objective and real subjective task credentials (COW): */
    const struct cred __rcu  *real_cred;
    /* Effective (overridable) subjective task credentials (COW): */
    const struct cred __rcu  *cred;
  // ...

};
```

```c

struct cred {
    atomic_t    usage;
    kuid_t      uid;      /* real UID of the task */
    kgid_t      gid;      /* real GID of the task */
    kuid_t      suid;      /* saved UID of the task */
    kgid_t      sgid;      /* saved GID of the task */
    kuid_t      euid;      /* effective UID of the task */
    kgid_t      egid;      /* effective GID of the task */
    kuid_t      fsuid;      /* UID for VFS ops */
    kgid_t      fsgid;      /* GID for VFS ops */
    unsigned    securebits;    /* SUID-less security management */
    kernel_cap_t    cap_inheritable; /* caps our children can inherit */
    kernel_cap_t    cap_permitted;    /* caps we're permitted */
    kernel_cap_t    cap_effective;    /* caps we can actually use */
    kernel_cap_t    cap_bset;    /* capability bounding set */
    kernel_cap_t    cap_ambient;    /* Ambient capability set */
  // ...
};
```

 The `struct cred` structure is typically considered to be immutable, which means that it should not be modified in place. Instead, when the credentials of a process need to be changed, the kernel provides a pair of functions called `commit_creds` and `prepare_kernel_cred` that can be used to replace the existing credentials with new ones.

The `commit_creds` function is used to commit a new set of credentials to a process. It takes a pointer to a `struct cred` structure as an argument, and updates the credentials of the current process to match the ones specified in the structure.

The `prepare_kernel_cred` function is used to create a new set of credentials that can be used to update the credentials of a process. It takes a pointer to a `struct task_struct` structure as an argument, and returns a pointer to a new `struct cred` structure that contains the new credentials. If the `reference_task_struct` argument is set to `NULL`, the function will create a new `struct cred` structure with root access and full privileges.

These functions are typically used when a kernel module or driver needs to change the privileges of a process, or when a user program needs to escalate its privileges in order to perform certain actions. 
Inshort, when we have control all we have to run is `commit_cred(prepare_kernel_cred(0));

>[!Note]
>We might face few complications How do we know where commit_creds and prepare_kernel_cred are in memory?
>

### Escaping Seccomp

We talked about `struct cred` the `struct task_struct` has one more amazing struct known as `struct thread_info`

```c
struct thread_info {  
    unsigned long flags;  /* low level flags */    u32 status;  /* thread synchronous flags */  
};
```

flags is a bit field that, among many other things, holds a bit named TIF_SECCOMP. Guess what it does? 

TIF_SECCOMP is a flag in the `flags` bit field of the `thread_info` structure in the Linux kernel. It indicates whether a thread (a flow of execution in a program) has enabled the secure computing mode (SECCOMP) filter. if It is set to 1 when SECCOMP is enabled for the thread, and 0 when it is disabled. This allows the kernel to efficiently check whether a thread is using SECCOMP without having to inspect the SECCOMP filter itself. So how does it get's checked ? they are checked on the syscall entry.

To escape seccomp, we just need to do (in KERNEL space): `current_task_struct->thread_info.flags &= ~(1 << TIF_SECCOMP)` How do we get the current_task_struct? We're in luck! The kernel points the segment register gs to the current task struct. In kernel development, there is a shorthand macro for this: current The plan: Access current->thread_info.flags via the gs register. Clear the TIF_SECCOMP flag. Get the flag!!! 
**Caveat: our children will still be seccomped (that's stored elsewhere)**

>[!info]
>Since the `task_struct` is very important data structure and is required most of the time the gs segment register is always pointing towards it.

### Memory Management By Kernel

In a Linux system, each process has a virtual memory space that is dedicated to that process. This virtual memory space is created by the kernel when the process is started, and it is used by the process to store its code, data, and other information.

The virtual memory space of a process contains several different regions, including:

-   The binary: This is the actual program code that is being executed by the process. It is stored in the virtual memory space as a series of instructions that can be executed by the processor.
   
-   The libraries: These are code libraries that are used by the program to perform various tasks. They are stored in the virtual memory space and can be accessed by the program as needed.
   
-   The heap: The heap is a region of memory that is used to store dynamically allocated memory. When a program needs to allocate memory at runtime (for example, using the malloc() function in C), it can request memory from the heap.
   
-   The stack: The stack is a region of memory that is used to store function local variables and other data that is needed during the execution of a function. When a function is called, a new stack frame is created on the stack to hold the local variables for that function.

-   Memory specifically mapped by the program: A program can request that specific regions of memory be mapped into its virtual memory space for various reasons. For example, a program might map in a file from the filesystem, or it might map in a shared memory region that is used for inter-process communication.

-   Helper regions: There are also several other regions of memory that are used for various purposes by the operating system and the program. These can include regions for storing thread-specific data, signal handlers, and other miscellaneous data.

-   Kernel code: In addition to the virtual memory space of a process, there is also an "upper half" of memory (above 0x8000000000000000 on 64-bit architectures) that is reserved for the kernel and is inaccessible to processes. This region of memory contains the code and data that is needed by the operating system to manage the system and provide services to processes. 

We can see the virtual memory space of a process by looking at the /proc/self/maps file. This file contains a list of the different memory regions that are mapped into the virtual memory space of the process, along with information about the permissions, offsets, and other attributes of each region.
For Details Read : [[Process in Linux]]
### Virtual Mapping

Physical memory, also known as RAM (random access memory), is the main type of memory that is used by a computer to store data that is being actively used or processed. When a program is running, it is loaded into physical memory so that it can be accessed quickly by the processor.

One of the challenges of managing physical memory is that there is a limited amount of it available on a typical computer, and multiple programs may need to be loaded into memory at the same time. To address this, most operating systems use a technique called "virtual memory" to allow each program to have its own virtual address space, even if there is not enough physical memory to give each program its own dedicated block of RAM.

In this system, each program is given a virtual address space that is much larger than the amount of physical memory available on the system. The operating system then uses a memory management unit (MMU) to map the virtual addresses used by the program to physical addresses in RAM, as needed. This allows multiple programs to share the available physical memory, and it also allows each program to be isolated from the others, so that they do not interfere with each other's operation.

### How to Map Virtual Memory Over Physical Memory

#### Strawman Solution

In this virtual memory system, each process is typically given a fixed-size virtual address space, which is the range of virtual addresses that the process is allowed to use. This address space is mapped on physical memory which was of  4KB in size. 

One advantage of this system is that it is easy for the operating system to track the mappings between virtual and physical memory, because each page of virtual memory is mapped to a corresponding page of physical memory. This makes it easy for the operating system to manage the memory usage of each process and to ensure that the correct data is being accessed by the processor.

But what if process needs more than 4KB memory ?

![[Pasted image 20230102035952.png]]

In a virtual memory system, the physical position of contiguous virtual pages (i.e., a group of consecutive pages in the virtual address space of a process) can be non-contiguous in physical memory. This means that the physical pages that are used to store the virtual pages may not be located next to each other in RAM.

To manage this, the operating system typically keeps track of the physical base address of each page of virtual memory. This allows it to map the virtual addresses used by the process to the correct physical addresses in RAM.

![[Pasted image 20230102040245.png]]

So, how exactly this physical base address for each page can be tracked ? In order to address this sometime way back in hisotry someone came up with idea of the Page Table.

#### Page Tables

A page table is a data structure that is used to store the mappings between virtual and physical memory. Each page table entry (PTE) in the page table corresponds to a page of virtual memory, and it stores the physical address of the page in RAM.

Traditionally, page tables were organized as an array of entries, with each entry containing the mapping for a single page of virtual memory. The size of the page table and the number of entries it contained were determined by the size of the virtual address space and the size of the pages used by the system. For example, if the virtual address space was 2MB and the page size was 4KB, the page table would contain 512 entries, each mapping a 4KB page of virtual memory to a corresponding page of physical memory.

![[Pasted image 20230103184701.png]]

However, this system has some limitations when it comes to managing non-contiguous virtual memory and very large virtual address spaces. For example, if the virtual address space of a process is larger than 2MB, the page table may not be able to hold enough entries to map all of the virtual pages. Similarly, if the virtual memory is non-contiguous (i.e., if the pages are not located next to each other in the virtual address space), it may be difficult to represent this using a simple array of page table entries. The Solution to these problems were solved by introducing the mullti-level paging system

### Page Directory

In a virtual memory system, a multi-level paging structure is a data structure that is used to store the mappings between virtual and physical memory. It is called a "multi-level" paging structure because it consists of multiple levels of data structures, rather than a single flat array.

The top level of the paging structure is called the page directory, and it contains a set of page directory entries (PDEs). Each PDE in the page directory maps to a page table, which is a data structure that contains a set of page table entries (PTEs). Each PTE in a page table maps to a page of physical memory, which is typically 4KB in size.

Using this multi-level paging structure, it is possible to map a large virtual address space (up to 1GB) to physical memory. For example, if each page table contains 512 entries and each page is 4KB in size, the paging structure can map up to 2MB of virtual memory (512 entries * 4KB per entry = 2MB). If the page directory contains 512 PDEs, and each PDE refers to a page table that maps 2MB of virtual memory, the page directory can map a total of 1GB of virtual memory (512 entries * 2MB per entry = 1GB).

![[Pasted image 20230103185140.png]]

There is some overhead associated with using a multi-level paging structure, as each level of the paging structure consumes some memory to store the data structures and the mappings. For example, the page directory itself consumes 0x1000 bytes of memory, and each page table consumes 0x1000 bytes of memory for each 2MB of virtual memory that it maps.

To reduce this overhead, it is possible to set a special flag in a PDE to indicate that it should refer directly to a 2MB region of physical memory, rather than to a page table. This can reduce the overhead of the paging structure. But what if we need more than 1GB of RAM infact we all do way more than 1GB of RAM. 

### Page Directory Page Table

directory page table (PDPT) to map an even larger virtual address space to physical memory. A PDPT is similar to a page directory, but it contains page directory pointers (PDPs) instead of PDEs. Each PDP in the PDPT points to a page directory, which in turn contains PDEs that map to page tables.

Using this structure, it is possible to map a virtual address space of up to 512GB of memory. For example, if each page directory contains 512 PDEs, and each PDE maps to a page table that maps 2MB of virtual memory, the PDPT can map a total of 512GB of virtual memory (512 PDPs * 512 PDEs * 2MB per PDE = 512GB).

There is some overhead associated with using a PDPT, as it consumes 0x1000 bytes of memory to store the data structure and the pointers to the page directories. In addition, each page directory consumes 0x1000 bytes of memory to store the data structure and the PDEs.

![[Pasted image 20230103185607.png]]

To reduce this overhead, it is possible to set a special flag in a PDP to indicate that it should refer directly to a 1GB region of physical memory, rather than to a page directory. This can reduce the overhead of the paging structure, but it also limits the ability of the operating system to fine-tune the mappings between virtual and physical memory. But what if we need more than 512 GB ?

### PML4

In modern virtual memory systems, it is possible to use four levels of paging to map a very large virtual address space to physical memory. The top level of the paging structure is called the page map level 4 (PML4), and it contains page directory page table pointers (PDPTPs). Each PDPTP in the PML4 points to a page directory page table (PDPT), which in turn contains page directory pointers (PDPs) that point to page directories. Each page directory contains page directory entries (PDEs) that map to page tables, and each page table contains page table entries (PTEs) that map to pages of physical memory.

![[Pasted image 20230103202107.png]]

Using this four-level paging structure, it is possible to map a virtual address space of up to 256TB of memory. For example, if each PDPT contains 512 PDPs, and each PDP maps to a page directory that contains 512 PDEs, which in turn map to page tables that map 2MB of virtual memory each, the PML4 can map a total of 256TB of virtual memory (512 PML4 entries * 512 PDPT entries * 512 PDP entries * 512 PDE entries * 2MB per PDE = 256TB).

This four-level paging structure allows the operating system to map very large virtual address spaces to physical memory, and it provides a high degree of flexibility for managing the memory usage of each process. However, it also requires a significant amount of overhead to manage the data structures and the mappings between virtual and physical memory.

### Virtual to Physical Conversion

The number 0x7fff47d4c123 is a hexadecimal representation of a memory address that is 48 bits long. When written in binary, this address can be split into six fields, each 8 bits long:

0111 1111 1111 1111 0100 0111 1101 0100 1100 0001 0010 0011 

A B C D E F

![[Pasted image 20230103205302.png]]

In a virtual memory system with a four-level paging structure, each of these fields corresponds to a different level of the paging structure. The fields are used to index into the paging data structures to determine the physical memory address that is mapped to the virtual memory address.

For example, the instruction `mov rax, [rbx]` might be translated into the following form:

`rax = *(long *)(PML4[A][B][C][D])[E]`

This instruction accesses the physical memory address that is mapped to the virtual memory address stored in the register RBX. To do this, it uses the values of the fields A, B, C, D, and E to index into the PML4, PDPT, PDP, PD, and PT data structures, respectively. The resulting value is then dereferenced and stored in the register RAX.

In this way, the four-level paging structure is used to translate the virtual memory address stored in RBX into a physical memory address, which is then accessed by the instruction. This allows the operating system to manage the memory usage of each process and to provide isolation and protection between different processes.

### Process Isolation

In a virtual memory system with a four-level paging structure, each process has its own page map level 4 (PML4) data structure to manage its virtual memory. The PML4 is used to store the mappings between the virtual and physical memory addresses used by the process.

To access the PML4 of a particular process, the operating system can use the control register CR3. The CR3 register holds the physical address of the PML4 for the current process. The operating system can use the instruction "mov cr3, 0xaabbccdd" to load the physical address of the PML4 into the CR3 register, which allows it to access the PML4 and manage the virtual memory of the process.

It is important to note that the CR3 register and the other control registers are only accessible in ring 0, which is the most privileged level of the operating system. This means that only the operating system and certain trusted programs are allowed to access these registers and modify their contents.

Other control registers exist that are used to set various options and control the behavior of the processor. These registers can be used to enable or disable certain features, set the operating mode of the processor (e.g., 32-bit or 64-bit mode), and perform other tasks. If you are interested in learning more about the control registers and their functions, you can refer to the link provided in the original text for more information.

### Virtual Machine Isolation

In a virtualization system, multiple virtual machines (VMs) may run concurrently on a single physical host machine. Each VM has its own operating system and applications, which may expect to have full access to the physical memory of the host machine. However, it is important to isolate the VMs from each other and prevent them from accessing each other's memory or interfere with the operation of the other VMs.

One way to achieve this isolation is through the use of an extended page table (EPT). An EPT is a data structure that is similar to a page table, but it is used to translate the "physical" memory addresses used by the VMs into the actual physical memory addresses of the host machine.

The EPT consists of multiple levels of data structures, similar to a four-level paging structure. The top level of the EPT is the EPT PML4, which contains pointers to the EPT PDPT data structures. The EPT PDPT contains pointers to the EPT PD data structures, which in turn contain pointers to the EPT PT data structures. Each EPT PT contains entries that map the "physical" memory addresses used by the VMs to the actual physical memory addresses of the host machine.

In this way, the EPT acts as an additional layer of address translation, allowing the VMs to access "physical" memory as if they had full access to the host machine's physical memory. However, in reality, each "physical" memory access is translated through the EPT, which ensures that the VMs are isolated from each other and that they do not interfere with the operation of the other VMs.

![[Pasted image 20230104202801.png]]

Read : [https://rayanfam.com/topics/hypervisor-from-scratch-part-4](https://rayanfam.com/topics/hypervisor-from-scratch-part-4/)

### Hardware Support for Virtual Memory

In a virtual memory system, it would be very slow for the operating system kernel to perform all of the necessary address translations in software. This is because each memory access would require multiple lookups in the page tables and other data structures, which would take a significant amount of time.

To overcome this problem, modern processors include a hardware component called the memory management unit (MMU). The MMU is responsible for performing the address translations in hardware, rather than in software. This allows the MMU to perform the translations much more quickly than the operating system kernel could.

To further improve the performance of the address translation process, the MMU can use a cache called the translation lookaside buffer (TLB) to store the resolved addresses of recently accessed memory locations. When a memory access is made, the MMU can check the TLB to see if the resolved address is already stored there. If it is, the MMU can use the cached address to access the memory, rather than having to perform the full address translation process again.

In this way, the MMU and the TLB work together to improve the performance of the virtual memory system by reducing the number of times that the operating system kernel needs to perform address translations in software. This allows the system to access memory more quickly and efficiently.

So far, we focused on x86. But other architectures are analogous!
ARM. (Simplified view. Full details: [https://developer.arm.com/documentation/100940/0101](https://developer.arm.com/documentation/100940/0101/))  
CR3 equivalent: TTBR0 (for userspace) and TTBR1 (for kernel addresses).  
Translation tables are referred to as Level 0, Level 1, Level 2, Level 3.
Linux: generic terminology.
PML4 = PGD (Page Global Directory)PDPT = PUD (Page Upper Directory) PD  = PMD (Page Mid-Level Directory) PT  = PT  (Page Table)
Linux requires a hardware MMU (although certain forks do not).

### The Kernel Sees All

Memory used by one process is normally isolated from the memory used by other processes. This is done to protect the processes from each other and to prevent one process from accessing or modifying the memory of another process.

However, the operating system kernel is able to access the memory of all processes and has full access to the physical memory of the system. In the past, some versions of the Linux operating system allowed users with root privileges to access the physical memory of the system through the /dev/mem device file. However, this could be a security risk, as it allowed users to potentially access or modify the memory of other processes or the kernel itself.

To address this issue, modern versions of the Linux kernel use a different approach to access physical memory. Rather than allowing users to access physical memory directly, the kernel maps the physical memory into its own virtual address space. This allows the kernel to access the physical memory in a convenient and efficient manner, without exposing it to other processes.

To facilitate the conversion between physical and virtual addresses, the kernel provides two macros: `phys_to_virt()` and `virt_to_phys()`. The `phys_to_virt()` macro converts a physical address into a virtual address that can be used by the kernel. `The virt_to_phys()` macro performs the reverse conversion, converting a virtual address used by the kernel into a physical address.

These macros are typically implemented as simple address arithmetic operations, such as additions and subtractions. As a result, they are easy to compile and reverse-engineer, but they also provide a convenient and efficient way for the kernel to access physical memory.

### Kernel Mitigations

The Linux Kernel had 149 vulnerabilities (as tracked by CVEs) in 2021 alone so what does it means it means kernel will get hacked no matter what. So, what now ? Obviously we came up mitigation for the kernel as we did in userspace.
Read [[Memory Errors]] for Understanding the Stack Canaries and ASLR.
Read [[Dynamic Allocators]] for Understanding the Heap Vulnerabilities.

The kernel has many mitigations. Some are familiar:

- Stack canaries protect the stack
  
- kASLR bases the kernel at a random location at boot.
  
- Heap/stack regions are not executable by default.

Of course, we can bypass them 

- Stack canaries: leak the canary!

- kASLR: leak the kernel base address

- Heap/stack regions NX: ROP!

### SMEP and SMAP

SMEP (Supervisor Mode Execution Prevention) and SMAP (Supervisor Mode Access Prevention) are security features that are designed to protect the system from exploits that try to execute or access user-space memory from kernel-space (also known as ring 0).

SMEP prevents kernel-space code from executing user-space memory at all. This means that any attempt by the kernel to execute code in user-space memory will be blocked.

SMAP prevents the kernel from even accessing user-space memory unless the AC (Access Control) flag in the RFLAGS register is set. The stac and clac instructions can be used to set and clear this flag, respectively.

There are cases where the kernel needs to access user-space memory, such as when it needs to access the path argument to the open() system call. In these cases, the copy_from_user function can be used to set the AC flag and allow the kernel to access the user-space memory.

Overall, the purpose of SMEP and SMAP is to provide an additional layer of protection against exploits that try to execute or access user-space memory from kernel-space. These features help to prevent the kernel from being compromised by malicious code and ensure the integrity of the system.

### What should we do ?

There are many different techniques that can be used to exploit vulnerabilities in the kernel, both by attackers and by defenders. These techniques are constantly evolving, as new vulnerabilities are discovered and new defensive measures are developed.

One technique that is sometimes used in kernel exploitation is the `run_cmd` function, which allows a command to be executed in user-space as the root user. This function is similar to the system() function, which is a standard C library function that allows a command to be executed by the operating system. However, the run_cmd function appears to be running the command within the kernel, rather than in user-space.

It is generally not a good idea to "yolo-run" commands or code, particularly in the kernel, as this can expose the system to security vulnerabilities and other risks. It is important to thoroughly understand the potential consequences and risks of running any code, especially in sensitive areas such as the kernel.

### Kernel ShellCode

So, if you know how to write shellcode that is splendid but userspace shellcode will not work in kernel space because userpace shellcode is full of series of the syscalls. But why is this problem ?
It's problem because the syscall interface if for the userspace to talk with kernelspace.

When a program makes a syscall, execution of the program is temporarily suspended and control is transferred to the kernel. The kernel then performs the requested function and returns control to the program when the function is complete.

The syscall_entry function is a function in the kernel that is responsible for handling syscalls. When a syscall is made, execution of the program is transferred to the syscall_entry function in the kernel. This function is designed to be called from user-space (the space where programs run), not from kernel-space (the space where the kernel runs).

If the syscall_entry function is called from kernel-space, it may lead to unexpected and potentially harmful behavior, as the function is not designed to be called in this way. This can cause chaos and disrupt the normal operation of the system.

In summary, the syscall_entry function is a function in the kernel that is responsible for handling syscalls made by programs. It is designed to be called from user-space and should not be called from kernel-space, as this can lead to unexpected and potentially harmful behavior.
For Details on Syscall 

## How to write the kernel ShellCode ?

Wakeup kid we are inside the kernelspace so we have kernel API. We have to use this in our shellcode. Keep in mind Kernel APIs are functions. You must call (not syscall) them from within the kernel.

The following actions can be taken within the kernel using kernel APIs and kernel objects to achieve certain goals:

-   Privilege elevation: This can be achieved by calling the commit_creds function with the result of the prepare_kernel_cred function as an argument. For example: commit_creds(prepare_kernel_cred(0)). This action changes the credentials (such as the user and group IDs) of the current process, potentially allowing the process to gain additional privileges.
  For Details about Linux Permission Module Read : [[Process in Linux]]
   
-   Seccomp escape: This can be achieved by modifying the TIF_SECCOMP flag in the current task's thread_info structure. For example: current_task_struct->thread_info.flags &= ~(1 << TIF_SECCOMP). Seccomp is a security feature that allows programs to specify a filter that defines what system calls they are allowed to make. Disabling seccomp allows a program to make any system call, potentially bypassing restrictions that have been put in place.
  For Details on SECCOMP read [[Sandboxing]]

-   Command execution: This can be achieved by calling the run_cmd function and providing it with the path to the command that should be executed. For example: run_cmd("/path/to/my/command").    

To perform these actions, it is necessary to find the current_task_struct structure and the offsets of its methods and members, as well as to call various kernel API functions such as prepare_kernel_cred, commit_creds, and run_cmd.  We can get these addresses from the `/proc/kallsyms` file.

-   To find the locations of specific functions in the kernel when ASLR is disabled, look in the /proc/kallsyms file on an identical system. This file can be accessed with root privileges.
  
-   To find the locations of specific functions in the kernel when ASLR is enabled, you will need to leak a kernel address and calculate the offset.
  
-   ASLR is usually enabled on real-world systems, so you will typically need to leak a kernel address and calculate the offset to find the locations of specific functions in memory.

For Details about userspace ShellCode Visit [[Shellcode Injection]] and [[Assembly]]
### Calling kernel APIs

Once you've found the APIs, you'll need to call them. Again, syscall is not the solution here! call is.
The normal call instruction takes a relative 32-bit offset to shift execution by.
To jump to a specific absolute location, without worrying about where your shellcode is in relation to the rest of the kernel, use the absolute form of call

```asm
mov rax, 0xffff414142424242  
call rax

/*This will jump to 0xffff414142424242!*/
```

### Figuring out the Offsets

The kernel is WAY too complex to figure out offsets manually.

Best option:

- Write a kernel module in C with the actions you want your shellcode to do.

- Build it for the kernel you want to attack.

- Reverse-engineer it to see how these actions work in assembly.

- Re-implement that assembly in your shellcode!

This insane process will preserve your sanity.

### Offsets Resolution

The above mentioned process seems very easy but there is catch when we compile the modules using kbuild compiler adds many relative instruction by that i means they will be resolve at load time with help of module symbol and relocation table let's understand this with help of example.

Consider this module 
```c
#include <linux/module.h>
#include <linux/kernel.h>
#include<linux/sched.h>

MODULE_LICENSE("GPL");

int init_module(void)

{

    printk(KERN_INFO "Hello i'll beat you seccomp\n");

    current->thread_info.flags &= ~ (1 << TIF_SECCOMP);

    return 0;

}

void cleanup_module(void)

{
    printk(KERN_INFO "Escaped lol!");

}
```

This module can be used to escape the seccomp filter when we will compile this module and try to look at assembly we will get something like this.


```
Disassembly of section .text.unlikely:

0000000000000000 <init_module>:
   0:   48 c7 c7 00 00 00 00    mov    rdi,0x0
   7:   e8 00 00 00 00          call   c <init_module+0xc>
   c:   65 48 8b 04 25 00 00    mov    rax,QWORD PTR gs:0x0
  13:   00 00 
  15:   48 81 20 ff fe ff ff    and    QWORD PTR [rax],0xfffffffffffffeff
  1c:   31 c0                   xor    eax,eax
  1e:   c3                      ret    

000000000000001f <cleanup_module>:
  1f:   55                      push   rbp
  20:   48 c7 c7 00 00 00 00    mov    rdi,0x0
  27:   48 bd 70 77 6e 2e 63    movabs rbp,0x6c6c6f632e6e7770
  2e:   6f 6c 6c 
  31:   53                      push   rbx
  32:   bb 00 01 00 00          mov    ebx,0x100
  37:   e8 00 00 00 00          call   3c <cleanup_module+0x1d>
  3c:   48 8b 35 00 00 00 00    mov    rsi,QWORD PTR [rip+0x0]        # 43 <cleanup_module+0x24>
  43:   48 01 de                add    rsi,rbx
  46:   48 39 2e                cmp    QWORD PTR [rsi],rbp
  49:   75 0f                   jne    5a <cleanup_module+0x3b>
  4b:   48 89 f2                mov    rdx,rsi
  4e:   48 c7 c7 00 00 00 00    mov    rdi,0x0
  55:   e8 00 00 00 00          call   5a <cleanup_module+0x3b>
  5a:   48 81 c3 00 10 00 00    add    rbx,0x1000
  61:   48 81 fb 00 a1 3f 7c    cmp    rbx,0x7c3fa100
  68:   75 d2                   jne    3c <cleanup_module+0x1d>
  6a:   5b                      pop    rbx
  6b:   5d                      pop    rbp
  6c:   c3                      ret    
```

Now let's focus on this part only because we want to inject shellcode which can escape the seccomp.

```asm
   c:   65 48 8b 04 25 00 00    mov    rax,QWORD PTR gs:0x0
  13:   00 00 
  15:   48 81 20 ff fe ff ff    and    QWORD PTR [rax],0xfffffffffffffeff
  1c:   31 c0                   xor    eax,eax
  1e:   c3                      ret    
```

When you will try to run this shellcode your process will get SEGFAULT. it's because the instruction
`mov    rax,QWORD PTR gs:0x0` is expected to be resolved at load time

When the kernel loads a module containing the line of code `current->thread_info.flags = ~(1 << TIF_SECCOMP)`, it uses the symbol table and relocation information contained in the module's object file to map the symbolic names and offsets used in the code to their corresponding addresses in the kernel's address space.

For example, the symbol `current` is used to access a global variable in the kernel's address space. The symbol table will contain an entry for `current` that specifies the offset of the variable within the module's address space. When the kernel loads the module, it will use this information to determine the actual address of the `current` variable in the kernel's address space and update the code accordingly.

The `thread_info` and `flags` members of the `current` structure are accessed using offsets, which are also contained in the symbol table. When the kernel loads the module, it will use these offsets to determine the actual addresses of these members within the kernel's address space and update the code accordingly.

Finally, the `TIF_SECCOMP` constant is used as an argument to the `<<` operator, which shifts the value `1` to the left by the number of bits specified by `TIF_SECCOMP`. The result is then negated using the `~` operator, and the resulting value is stored in the `flags` member of the `thread_info` structure.

### Workaround

So, how can i get the offset the idea is simple you have to debug the running module to find the offsets. But wait how can i know which instructions are going to be resolved at load time you can use the -r flag in objdump when you will use the objdump with -r flag on the compiled module given in example your out will get output like this

```asm


Disassembly of section .text.unlikely:

0000000000000000 <init_module>:
   0:   48 c7 c7 00 00 00 00    mov    rdi,0x0
                        3: R_X86_64_32S .rodata.str1.1
   7:   e8 00 00 00 00          call   c <init_module+0xc>
                        8: R_X86_64_PLT32       printk-0x4
   c:   65 48 8b 04 25 00 00    mov    rax,QWORD PTR gs:0x0
  13:   00 00 
                        11: R_X86_64_32S        current_task
  15:   48 81 20 ff fe ff ff    and    QWORD PTR [rax],0xfffffffffffffeff
  1c:   31 c0                   xor    eax,eax
  1e:   c3                      ret    

000000000000001f <cleanup_module>:
  1f:   55                      push   rbp
  20:   48 c7 c7 00 00 00 00    mov    rdi,0x0
                        23: R_X86_64_32S        .rodata.str1.1+0x16
  27:   48 bd 70 77 6e 2e 63    movabs rbp,0x6c6c6f632e6e7770
  2e:   6f 6c 6c 
  31:   53                      push   rbx
  32:   bb 00 01 00 00          mov    ebx,0x100
  37:   e8 00 00 00 00          call   3c <cleanup_module+0x1d>
                        38: R_X86_64_PLT32      printk-0x4
  3c:   48 8b 35 00 00 00 00    mov    rsi,QWORD PTR [rip+0x0]        # 43 <cleanup_module+0x24>
                        3f: R_X86_64_PC32       page_offset_base-0x4
  43:   48 01 de                add    rsi,rbx
  46:   48 39 2e                cmp    QWORD PTR [rsi],rbp
  49:   75 0f                   jne    5a <cleanup_module+0x3b>
  4b:   48 89 f2                mov    rdx,rsi
  4e:   48 c7 c7 00 00 00 00    mov    rdi,0x0
                        51: R_X86_64_32S        .rodata.str1.1+0x2a
  55:   e8 00 00 00 00          call   5a <cleanup_module+0x3b>
                        56: R_X86_64_PLT32      printk-0x4
  5a:   48 81 c3 00 10 00 00    add    rbx,0x1000
  61:   48 81 fb 00 a1 3f 7c    cmp    rbx,0x7c3fa100
  68:   75 d2                   jne    3c <cleanup_module+0x1d>
  6a:   5b                      pop    rbx
  6b:   5d                      pop    rbp
  6c:   c3                      ret    
```

### task_struct

Questions related to task_struct.

**What is difference between the active_mm and mm in task_struct?**
In the context of the Linux kernel, `mm` and `active_mm` are two fields within the `task_struct` data structure that represent a process's memory management information.

`mm` stands for "memory management," and it points to the process's main memory descriptor structure. This data structure holds information about the process's virtual address space, including the locations of code, data, and stack segments, as well as other metadata like page tables, memory protection settings, and memory mappings.

`active_mm`, on the other hand, is a pointer to the memory descriptor structure that represents the process's currently active memory mappings. This is particularly relevant in the case of multi-threaded processes, where each thread has its own `mm` structure, but they all share the same `active_mm`. When a thread switches between different memory mappings (for example, when it context-switches to a different task), the `active_mm` field is updated to reflect the new mapping.

In summary, `mm` is a pointer to the main memory descriptor structure for a process, while `active_mm` is a pointer to the memory descriptor structure that currently represents the process's active memory mappings.