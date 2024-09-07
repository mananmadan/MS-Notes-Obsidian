### What is a process?
- It's an execution instance of an application
- A application can be static (stored on disk) and active (when loaded to main memory and is started execution)
- An active state of application is called a process.
- Active State of same static application can be multiple. Eg: Two launched notepad
- The state they are in depends on input etc as well, basically ***execution states*** of 2 instance of the same application will be different.
### What does a process look like?
- A process encapsulates the below data for a running application
- Text, Data, Heap, Stack
- ![[Pasted image 20240715220521.png]]
- Text and Data
	- text mainly stores the code of that process
	- data stores the initialized variable before the process enters active state, this include static global variable etc.
- Heap
	- This stores the data that is dynamically allocated at execution, so a new/malloc call will be allocated here
- Stack
	- This store the current ***"state"*** of execution of a process
	- For example if we are at a current state in the program X and we are going to a state Y, then the state of X is stored in a stack data structure in this data, as when we have the result of Y, we can retrieve where we have left from X
	- LIFO Queue

### Concept of address space and page tables 
- ![[Pasted image 20240715221236.png]]
- A process containing the data is mapped on Virtual Address Space, this virtual address space is not actually mapped to physical space
- A page table maps addresses on this Virtual Address Space to Physical Memory 
- And this is different for each process .. ie for a Process P1 .. virtual address A can be mapped to Physical Location B and for a Process P2 .. virtual address A can be mapped to a Physical Location C
- In 32 bit system .. this space V0 (can be of 32 bits) .. that is a process can have max 4GB of such space !!
### Using Disk for memory
- It's important to note in the above example that all the processes can't have space in the main memory, so the operating system move some of that information to disk
- So in that case there is no mapping for that virtual address space on the physical main memory.
	- When the program hits that address, it tries to look up the page table, but no page table entry exists for it / or no page table exists for this
	- So a **page fault!** occurs, when this happens OS searches the disk, loads it in memory and updates page table

### How does the OS know what the process is doing?
- OS Need to track the execution state of a process as it needs to know what the process is doing in case it decides to perform a save/restore option basically
- Program Counter is helpful to know the state of the process at it can point out which instruction the current process is at
- CPU Registers also store the state and the information regarding execution state of the process
- Stack Pointer stores the last state the program was in

**To maintain all this information OS maintains a process control block for each process**
- A process control block stores all the necessary information regarding that process
- It's a data structure essentially that stores all this information
- ![[Pasted image 20240717205018.png]]
- PCB is created when a process starts and the fields keep changing with the process state
### How is a PCB Used?
- Each time a save happens to OS Save the state from the CPU's registers in the process control block and then does a restore based on the info stores in the PCB
- **The job of OS here is to save the state from CPU to PCB**
![[Pasted image 20240717210248.png]]
- ***Side Note: Can view some of this information in /proc/<PID/ file system***

### Context Switch
- When context of CPU is shifted from one process to another such in the above example
- **Expensive Operation**
	- Direct Cost
		- Save and Restore of P1 for eg if context is given to P2
	- Indirect Cost
		- When P1 was running, the caches would have been full with the data of P1
		- Now when P2 will run, almost all of the data will have to loaded again and filled in the cache, so there will be no cache hits ***cold cache***
		- If the cache has a lot of hits, it's call ***hot cache***
### Process States
- ![[Pasted image 20240719220151.png]]
- A process can be in the following states based on which it can be managed
- **New** : newly launched process, OS performs some checks and all then let's it into ready state
- **Ready** : Here the process is not scheduled yet .. once it's scheduled and given CPU time, it enters running state
- **Running** : Process enters here when CPU is executing the process 
	- From Here, if there is an IO or event wait, process goes into waiting state
	- ***SIGSTP or CtrlZ*** put the process into waiting state
- **Waiting** : Here the process is waiting for some event or I/O on completion of which the process will enter the ready state

### How a process is created
- Initially when the kernel boots up, some privileged processes are launched at first, like init() process in linux
- Then all the other process are just created from parent processes 
- For example when a new user logs in a csh process is created
- When that user opens vim or emacs that process is created from csh
- #### 2 ways a child process is created
- ##### FORK
-  In fork a new child process is created and a direct copy of the process control of the parent's process is put in the child process control block
- The instruction pointer is also kept at the fork() statement, so basically since the PCB and the instruction set is the same, it will execute the entire instruction as it is written for the main process 
- **Question** : The values that were declared earlier either through malloc calloc, are they accessible in forked process? .. PCB is same that is fine .. but all this data is placed in the address space of that process .. so the forked process can access that as well .. or the instruction space is also copied.
<details>
    When a process forks, the operating system creates a new process (the child process) that is a duplicate of the parent process. However, certain details about how memory and data are handled can be nuanced
  1. **Memory Allocation (malloc, calloc):**
    
    - When a process uses `malloc` or `calloc` to allocate memory, this memory is allocated in the process's heap. Upon forking, the child process receives a copy of the parent's memory, including the heap, stack, and data segments.
    - Initially, the child process has its own copy of the parent's allocated memory, meaning it can access and modify the data independently of the parent process. This is achieved through a mechanism known as **copy-on-write**.
   2. **Copy-on-Write (COW):**
    
    - Copy-on-write is an optimization used in modern operating systems. Instead of immediately copying all the parent's memory pages to the child process, both processes share the same physical memory pages until one of them writes to a page. When a write occurs, the OS creates a copy of the page, ensuring that modifications in one process do not affect the other. This means both processes can read the same memory until a write occurs, at which point they diverge.
   3. **Instruction Space:**
    
    - The instruction space (code segment) is also copied during a fork. Since this segment is typically read-only, both the parent and child processes can share the same physical memory for their code. This is efficient and avoids unnecessary duplication of the executable code.
   4. **Process Control Block (PCB):**
    
    - The PCB of the child process is distinct from that of the parent process. The PCB contains information about process state, registers, and other control information necessary for the OS to manage the process. While the address space (memory) is initially shared (with copy-on-write), the control information (PCB) is unique for each process.
     In summary, after a fork, the child process can access the memory allocated by the parent process using `malloc` or `calloc`, but they do so independently. The actual memory is copied (or shared with copy-on-write semantics) to ensure that changes in one process do not affect the other. The instruction space is effectively shared since it is typically read-only.
    </details> 

- ##### EXEC
- In exec the behavior of the new child is not the same as the parent process and it will have an entirely separate entity like process block and address space as well.
- So basically all the child processes that are given in example in for linux that started from init, they were launched using exec.
- ***Loads a new program and starts the instruction pointer from the beginning***
### Role of a CPU Scheduler
- There are multiple processes in the ready queue, the role of scheduler is to effectively manage following things
	- **Preempt:** scheduler must be able to stop and retain the process state of a running process (save current context)
	- **Schedule**: it must be able to effectively choose which process next to run
	- **Dispatch**: dispatch and switch into context
- ### How often should a scheduler run?
- ![[Pasted image 20240721184842.png]]
- The more often we run the scheduler, more of the CPU time is just spent doing unnecessary work (of the scheduler) and not of the process.
- Hence this is a design consideration while designing a scheduler
- ### IO Considerations
- ![[Pasted image 20240721201954.png]]
- It's possible that certain processes require I/O operations, for that we make a seperate I/O queue and when the I/O request is complete, but it again in the ready queue again to be scheduled by the scheduler.
### Can processes interact?
- Processes can interact with each other, but in order for smooth runtime of these processes, OS naturally tries to maintain isolation b/w 2 processes making process interaction a complex task
- There can be 2 types in which a process can interact
- #### Message Passing
	- This includes message queue, sockets (both TCP/IP or UDP {linux sockets})
	- OS Provides a channel/ abstraction (eg. in case of socket abstraction of network) on which both the processes can write(send) or read(recv)
- #### Shared Memory Passing
	- This is the fastest way, that is there is a shared memory region
	- One process write to it, other one reads
	- In this case **a direct mapping of processes virtual space** is created to the shared memory space, so both the processes will attach to same physical memory location, leading to fast read and write.
	- There's often a need to manage race conditions in such a case as both the processes can read and write simultaneously.
	 - **This is generally only possible when both the processes are on the same machine**

