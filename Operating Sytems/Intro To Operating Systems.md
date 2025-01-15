## <u>What is an operating system? </u>
- Let's first look at the hardware of a computing system. This computing system consists of:

	* Central Processing Unit (CPU)
	* Physical Memory
	* Network Interfaces (Ethernet/Wifi Card)
	* GPU
	* Storage Devices (Disk, Flash drives,USB)

- In addition, a computing system can have higher level applications. These are the "programs" that we use every day on our computer:

	* Skype
	* Word
	* Chrome

**The Operating System is the layer of software that sits between the hardware components and the higher level applications.**

## <u>Main Functions of an Operating System</u>

#### Operating systems hide hardware complexity. <u>[Abstraction]</u>
- You don't want to have to worry about the nuts and bolts of interacting with storage devices when you are writing an application. The operating system provides a higher level abstraction, the *file*, with a number of operations - like *read* and *write* - which applications can interact with.

#### Operating systems manage underlying hardware resources.<u>[Arbitration]</u>
- Operating system allocates memory for applications, schedules them for execution on the CPU, controls access to various network devices and so on.a
- Operating system distributing memory b/w two process is basically an arbitration whereas operating system providing a socket for the network card is an abstraction

#### Provides isolation and protection.
- When applications are running concurrently, the operating system has to ensure that they can do what they need to without hurting one another. For example, memory allocated to each application must not be readable/writable from another application.

## <u> Operating System Definition </u>
An operating system is a layer of systems software that

* directly has privileged access to the underlying hardware;
* manages hardware on behalf of one or more applications according to some predefined policies.
* Ensures that applications are isolated and protected from one another

## <u>Operating System Examples</u>
- Certain operating systems may target the desktop environment, while others may target an embedded environment, while still others may target a mainframe or a server environment.

- For our purposes, we will focus mainly on operating systems for desktop environments and embedded environments.

- For desktop operating systems we have:

* Microsoft Windows
* Unix-based systems
	* OS X
	* Linux

- For embedded operating systems:

-  Android
* iOS
* Symbian

## <u>OS Elements</u> 
- To achieve its goals, an operating system provides a number of high level abstractions, as well as a number of mechanisms to operate on these abstractions.

- Examples of abstractions include:
	* process, thread (application abstractions)
	* file, socket, memory page (hardware abstractions)

- Corresponding mechanisms could be:
	* create/schedule
	* open/write/allocate
- Operating systems may also integrate specific policies that determine exactly how the mechanisms will be used to manage the underlying hardware.
- For example, a policy could determine the maximum number of sockets that a process has access to.

## <u>OS Elements Example</u>
- Let's look at an example of *memory management*.
- The main abstraction here is the *memory page*, which corresponds to some addressable region of memory of some fixed size.
- The operating system integrates mechanisms to operate on that page like *allocate*, which actually allocates the memory in DRAM. It can also map that page into the address space of the process, so that the process can interact with the underlying DRAM.
- Over time, the page may be moved to different spaces of memory, or may be moved to disk, but those who use the page abstraction don't have to worry about those details. That's the point of the abstraction.
- How do we determine when to move the page from DRAM to disk? This is an example of a *policy*, and one such implementation of that policy would use the least-recently-used (LRU) algorithm, moving pages that have been accessed longest ago onto disk.

#### Technical Sidenote

Any process has a virtual address range ("address space") that is allocated for it's execution
- Page is a grouped virtual address range for a certain process
	- There is a combination of how these addresses are mapped, given in the figure below
- A page table, maps the page to actually physical addresses
- A physical memory is also a address space, where each address can be 32bit for 32bit systems and 64 bit for 64bit systems
```
+-------------------------+ 0xFFFFFFFF (top of the address space)
| Stack Segment           |
|  (grows downward)       |
|-------------------------|
| (free space)            |
|-------------------------|
| Heap Segment            |
|  (grows upward)         | <--- malloc allocations
|-------------------------|
| BSS Segment             | 
|  (uninitialized data)   |
|-------------------------|
| Data Segment            | 
|  (initialized data)     |
|-------------------------|
| Text Segment            |
|  (executable code)      |
+-------------------------+ 0x00000000 (bottom of the address space)

```

## <u>OS Design Principles</u>

### Separation of mechanism and policy
- We want to incorporate flexible mechanisms that can support a number of policies.
- For the example of memory, we can have many policies: LRU, LFU (least-frequently used), random. It is a good design strategy to create our memory management mechanisms such that they can generalize to these different policies.
- In different settings, different policies make more sense.
### Optimize for the common case
* Where will the OS be used?
* What will the user want to execute on that machine?
* What are the workload requirements?

Understanding the common case - which may change in different contexts - helps the OS implement the correct policy, which of course relies on generalized mechanisms.

## <u>OS Protection Boundary</u>
- Computer systems distinguish between at least two modes of execution:
	* user-level (unprivileged)
	* kernel-level (privileged)
- Because an OS must have direct access to hardware, it must operate in kernel mode.
- Applications generally operate in user-mode.
- Hardware access can only be utilized in the kernel mode from the OS directly.
- Crossing from user-level to kernel-level is supported by most modern operating systems.
- As an example, the operating system can flip a bit in the CPU that allows applications executing instructions to have direct access to hardware resources. When the bit is not flipped, operations are forbidden.
- When privileged instructions are encountered during a non-privileged execution, the application will be **trapped**. This means the application's execution will be interrupted, and control will be handed back to the OS.
- The OS can decide whether to grant the access or potentially terminate the process.
- The OS also exposes an interface of  **system calls**, which the application can invoke, which allows privileged access of hardware resources for the application.

- For example:
	* `open` (file)
	* `send` (socket)
	* `mmap` (memory)
- Operating systems also support **signals**, which is a way for the operating system to send notifications to the application.

## <u>System Call Flow</u>
- Begin within the context of a currently executing process. The process needs access to some hardware, and thus needs to make a system call. The application makes the system call (potentially passing arguments), and control is passed to the operating system, which accesses the hardware. Execution control (as well as any necessary data) is passed back from the operating system to the application process.
- In terms of context switching, the process involves a change from user-mode to kernel-mode to user-mode.
- Not necessarily a cheap operation to make a system call!
- Arguments to system call can either be passed directly from process to operating system, or they can be passed indirectly by specifying their address (pointers?)
- In **synchronous mode**, the process waits until the system call completes. Asynchronous modes exist also.

## <u>Crossing the OS Boundary</u>
- User/Kernel transitions are very common and useful throughout the course of application execution.
- The hardware supports user/kernel transitions.
- ![[Pasted image 20240713194403.png]]
- The concept here is when a process needs to switch context from User Mode to Kernel Mode, it raises and interrupt or a trap, to signal the CPU that this process needs attention from the OS
- After that the CPU would set a special flag called trap mode bit = 0, indicating that it's transitioning from user mode to kernel level, this basically involves storing the programs state -> getting the output of the system call -> returning to the programs state
- For example, the hardware will cause a *trap* on illegal executions that require special privilege.
- Hardware initiates transfer of control from process to operating system when a trap occurs.
- User/Kernel transition requires instructions to execute, which can take ~100ns on a 2Ghz Linux box.

In addition, the OS may bring some data into the hardware cache, which will bounce out some other memory used by another application. Accessing data out of cache, say from  main memory, may have as much as 100x impact on the performance of some subsequent actions.

## <u>OS Services</u>
- An operating system provides applications with access to the underlying hardware.
- It does so by exporting a number of services, which are often directly linked to the components of the hardware:
	* Scheduling component (CPU)
	* Memory manager (physical memory)
	* Block device driver (block device)
- In addition, some services are even higher level abstractions, not having a *direct* hardware component. For example, the filesystem as a service.
- Basic services include:
	* Process management
	* File management
	* Device management
	* Memory management
	* Storage management
	* Security

[Windows v. Unix Services](https://s3.amazonaws.com/content.udacity-data.com/courses/ud923/notes/ud923-p1l2-windows-vs-linux-system-calls.png)

## System Call Quiz
- System call to send a signal to a process 
	- Kill (can send kill -9 (terminate forcefully), kill -15 (graceful termination), interrupt, hang and others)
- System call to change group id of a process 
	- setgid (this can be used to give a process permissions to access some files for example .. just basically giving that process admin permissions)
- Mount a file system
	- mount (when you have a hard-disk/attachable device and you want to access it's contents through the file system .. mount can help you to attach them to a mount point)
	- A mount point is a place in the existing file system where you can attach the hard drive and then basically access it through the file system
- Read/Write system parameters
	- sysctl (this command can be used to change kernel parameters at runtime)

## Types of OS Architecture
- ### Monolithic Architecture
	- All of the abstractions/services that the application layer could require like scheduler, memory management, device drivers are part of and managed by the kernel level
	- This type of OS aims to have all the services already present and kind of aims to be "fixed" in nature
	- Eg: This OS can have multiple file systems at the same time
		- File Sytem for Random I/O
		- File System for Sequential I/0
	- PROS:
		- Everything inside the operating system which makes it easier for compile time optimization such as inlining and all. 
		- Better abstraction as all the H/w is hidden away by the operating system
	- CONS:
		- Huge codebase to provide such an architecture, taking up huge memory resulting in ***less memory for the user applications***
		- Maintainability is hard
- ### Modular OS {Linux}
	- This operating system basically has monolithic architecture, but each of the module that it's comprised of such as schedule, memory management, file service is ***replaceable*** 
	- Some basic services are still a part of it, but it passes a module interface that is required to be run by every service in that operating system, so we can essentially install these modules according to work load etc. 
	- Eg: based on the work load, I can replace the scheduler for example of the OS
	- PROS:
		- less resource intensive and maintainable as only few services are provided by the operating system
	- CONS:
		- modules designed for specific use could be used in some other direction impacting performance
		- modules could be essentially open source leading to a buggy module
	 - ***Linux is this type, and it makes sense because linux basically provides a module interface that the core kernel need to interact with, using this interface you can use any open source module and it should work fine***
- ### Microkernel 
- ![[Pasted image 20240715201438.png]]
- This design essentially provides very basic services as a part of OS like Address Space, Thread context, IPC
- The rest of the services which typically should be at the OS Layer are run at the application layer and basically do a kernel crossing to access those basic services
- PROS
	- Small code in design, takes less memory
	- Services running in user level, makes it easier to debug the service and increases verifiability
 - CONS
	 - Very specialized to H/W, hence not easy to port and increases software complexity
	 - ***Obv!*** Takes time in User/Kernel Crossing as multiple interactions
	
 