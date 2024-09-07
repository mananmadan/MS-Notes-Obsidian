### <u>Kernel Level vs User Level Threads</u>  
- Basically there are user level threads and kernel level threads
- Operating itself is multi threaded, so it's possible that one or more threads correspond to a single kernel level thread for the operating system
- User Level Threads are managed by User Libraries such as PThreads
       - ![[Pasted image 20240819104835.png]]
### <u>Single CPU Case</u>:
- In case you assume a single CPU case, but multiple threads at the kernel level as well are multiple threads at the user level
- **At User Level:** Threading library that you're using will store the thread id, it's stack etc
- **How to Represent Multiple Kernel Level Threads:**
	- At kernel level one option would be to maintain multiple instance of this Process Control Block (PCB), but here lot of this information is same, as that belongs to the process 
	- So we can divide this information into 2 parts, a Process Control Block storing only the values at the process level, and one **Kernel Level Thread Data Structure** which could store the stack pointer, register ie information specific to each of the thread
	 ![[Pasted image 20240824133235.png]]
### <u>Thread Level Data Structures</u>
- The thinking behind making multiple data structures is for making it easier to retain common information (like mapping and all) and easily replace information that is specific to a thread
- **Information of the PCB is devided into 3 parts according to the lecture**
- Hard Process State: This will contain only the information that is relevant to the process and doesn't change when switching from one thread to another
- Soft Process State & Kernel Level Thread: Soft process state refers to the part of process control block that changes with the context switch of a thread, the kernel thread also contains only the information required to represent the thread at kernel level
- **I think the major difference b/w these is that in Soft Process State, some of the user level thread is stored, where as in Kernel Level Thread data structure only the information about the Kernel Level Threads is stored**
- There would also be a data structure at the cpu level that contains information at the CPU level, such a pointer to threads, which thread is currently running and all
  ![[Pasted image 20240824134629.png]]
- Linux Kernel Structure, wrt to all these structures
```
task_struct
│
├── mm_struct         (Manages memory management for the process)
│
├── signal_struct     (Handles signal-related data)
│
├── files_struct      (Manages open file descriptors)
└── thread_struct     (Stores architecture-specific thread state)
    
thread_info           (Contains low-level thread data, critical for context switching)
│
└── Linked with the kernel stack (not a direct member of thread_struct)
    
kthread_worker
│
└── Uses a kernel thread (represented by a task_struct) to execute work items
  
```
  ## User Level Structures in Solaris 2.0

### <u>SunOS paper</u>

![](https://assets.omscs.io/notes/0C4D3FA7-A3C9-4469-B87B-050484774183.png)

The OS is intended for multiple CPU platforms and the kernel itself is multithreaded. At the user level, the processes can be single or multithreaded, and both many:many and one:one ULT:KLT mappings are supported.

Each kernel level thread that is executing on behalf of a user level thread has a **lightweight process** (LWP) data structure associated with it. From the user level library perspective, these LWPs represent the virtual CPUs onto which the user level threads are scheduled. At the kernel level, there will be a kernel level scheduler responsible for scheduling the kernel level threads onto the CPU.

### Lightweight Threads paper

When a thread is created, the library returns a thread id. This id is not a direct pointer to the thread data structure but is rather an index into an array of thread pointers.

The nice thing about this is that if there is a problem with the thread, the value at the index can change to say -1 instead of the pointer just pointing to some corrupt memory.

The thread data structure contains different fields for:

- execution context
- registers
- signal mask
- priority
- stack pointer
- thread local storage
- stack

The amount of memory needed for a thread data structure is often almost entirely known upfront. This allows for compact representation of threads in memory: basically, one right after the other in a contiguous section of memory.

However, the user library does not control stack growth. With this compact memory representation, there may be an issue if one thread starts to overrun its boundary and overwrite the data for the next thread. If this happens, the problem is that the error won't be detected until the overwritten thread starts to run, even though the cause of the problem is the overwriting thread.

The solution is to separate information about each thread by a **red zone**. The red zone is a portion of the address space that is not allocated. If a thread tries to write to a red zone, the operating system causes a fault. Now it is much easier to reason about what happened as the error is associated with the problematic thread.

![](https://assets.omscs.io/notes/8D7A444B-90D8-451B-9DA4-8D2B8D1D1239.png)

### Kernel Level Structures in Solaris 2.0

For each process, the kernel maintains a bunch of information, such as:

- list of kernel level threads
- virtual address space
- user credentials
- signal handlers

The kernel also maintains a light-weight process (LWP), which contains data that is relevant for some subset of the user threads in a given process. The data contained in an LWP includes:

- user level registers
- system call arguments
- resource usage info
- signal masks

The data contained in the LWP is similar to the data contained in the ULT, but the LWP is visible to the kernel. When the kernel needs to make scheduling decisions, they can look at the LWP to help make decisions.

The kernel level thread contains:

- kernel-level registers
- stack pointer
- scheduling info
- pointers to associated LWPs, and CPU structures

The kernel level thread has information about an execution context that is always needed. There are operating system services (for example, scheduler) that need to access information about a thread even when the thread is not active. As a result, the information in the kernel level thread is **not swappable**. The LWP data does not have to be present when a process is not running, so its data can be swapped out.

The CPU data structure contains:

- current thread
- list of kernel level threads
- dispatching & interrupt handling information

Given a CPU data structure it is easy to traverse and access all the other linked data structures. In SPARC machines (what Solaris runs on), there is a dedicated register that holds the thread that is currently executing. This makes it even easier to identify and understand the current thread.

![](https://assets.omscs.io/notes/D37A8ECF-B01C-4FEC-9B55-8FB7C938B26B.png)

A process data structure has information about the user and points to the virtual address mapping data structure. It also points to a list of kernel level threads. Each kernel level thread structure points to the lightweight process and the stack, which is swappable.

![](https://assets.omscs.io/notes/69BD0A17-1A53-48B8-940A-9B4B96722D25.png)

## Basic Thread Management Interaction

Consider a process with four user threads. However, the process is such that at any given point in time the actual level of concurrency is two. It always happens that two of its threads are blocking on, say, IO and the other two threads are executing.

If the operating system has a limit on the number of kernel threads that it can support, the application might have to request a fixed number of threads to support it. The application might select two kernel level threads, given its concurrency.

When the process starts, maybe the operating system only allocates one kernel level thread to it. The application may specify (through a `set_concurrency` system call) that it would like two threads, and another thread will be allocated.

Consider the scenario where the two user level threads that are scheduled on the kernel level threads happen to be the two that block. The kernel level threads block as well. This means that the whole process is blocked, even though there are user level threads that can make progress. The user threads have no way to know that the kernel threads are about to block, and has no way to decide before this event occurs.

What would be helpful is if the kernel was able to signal to the user level library _before_ blocking, at which point the user level library could potentially request more kernel level threads. The kernel could allocate another thread to the process temporarily to help complete work, and deallocate the thread it becomes idle.

Generally, the problem is that the user level library and the kernel have no insight into one another. To solve this problem, the kernel exposes system calls and special signals to allow the kernel and the ULT library to interact and coordinate.

## Thread Management Visibility and Design

The kernel sees:

- Kernel level threads
- CPUs
- Kernel level scheduler

The user level library sees:

- User level threads
- Available kernel level threads

The user level library can request that one of its threads be bound to a kernel level thread. This means that this user level thread will always execute on top of a specific kernel level thread. This may be useful if in turn the kernel level thread is pinned to a particular CPU.

If a user level thread acquires a lock while running on top of a kernel level thread and that kernel level thread gets preempted, the user level library scheduler will cycle through the remaining user level threads and try to schedule them. If they need the lock, none will be able to execute and time will be wasted until the thread holding the lock is scheduled again.

The user level library will make scheduling changes that the kernel is not aware of which will change the ULT/KLT mapping in the many to many case. Also, the kernel is unaware of the data structures used by the user level, such as mutexes and wait queues.

We should look at 1:1 ULT:KLT models.

The process jumps to the user level library scheduler when:

- ULTs explicitly yield
- Timer set by the by UL library expires
- ULTs call library functions like lock/unlock
- blocked threads become runnable

The library scheduler may also gain execution in response to certain signals from timers and/or the kernel.

## <u>Issue On Multiple CPUs</u>

In a multi CPU system, the kernel level threads that support a process may be running concurrently on multiple CPUs. We may have a situation where the user level library that is operating in the context of one thread on one CPU needs to somehow impact what is running on **another** CPU.

Scenario![](https://assets.omscs.io/notes/8859114F-5FA0-445D-9B5D-9324D73FC541.png)

Currently, T2 is holding the mutex and is executing on one CPU. T3 wants the mutex and is currently blocking. T1 is running on the other CPU.

At some point, T2 releases the mutex, and T3 becomes runnable. T1 needs to be preempted, but we make this realization from the user level thread library as T2 is unlocking the mutex. We need to preempt a thread on a different CPU!

We cannot directly modify the registers of one CPU when executing as another CPU. We need to send a signal from the context of one thread on one CPU to the context of the other thread on the other CPU, to tell the other CPU to execute the library code locally, so that the proper scheduling decisions can be made.

Once the signal occurs, the library code can block T1 and schedule T3, keeping with the thread priorities within the application.

## <u>Synchronization Related Issues</u>

Scenario![](https://assets.omscs.io/notes/EA93A7AA-BDE7-492C-8001-B4DE63CA8BD4.png)

T1 holds the mutex and is executing on one CPU. T2 and T3 are blocked. T4 is executing on another CPU and wishes to lock the mutex.

The normal behavior would be to place T4 on the queue associated with the mutex. However, on a multiprocessor system where things can happen in parallel, it may be the case that by the time T4 is placed on the queue, T1 has released the mutex.

If the critical section is very short, the more efficient case for T4 is not to block, but just to spin (trying to acquire the mutex in a loop). If the critical section is long, it makes more sense to block (that is, be placed on a queue and retrieved at some later point in time). This is because it takes CPU cycles to spin, and we don't want to burn through cycles for a long time.

Mutexes which sometimes block and sometimes spin are called **adaptive mutexes.** These only make sense on multiprocessor systems, since we only want to spin if the owner of the mutex is currently executing in parallel to us.

We need to store some information about the owner of a given mutex at a given time, so we can determine if the owner is currently running on a CPU, which means we should potentially spin. Also, we need to keep some information about the length of the critical section, which will give us further insight into whether we should spin or block.

Once a thread is no longer needed, the memory associated with it should be freed. However, thread creation takes time, so it makes sense to reuse the data structures instead of freeing and creating new ones.

When a thread exits, the data structures are not immediately freed. Instead, the thread is marked as being on **death row**. Periodically, a special **reaper** thread will perform garbage collection on these thread data structures. If a request for a thread comes in before a thread on death row is reaped, the thread structure can be reused, which results in some performance gains

### Interrupts && Signals
- ***Interrupts:***
	- These are generated by external devices, or devices other that CPU like (timers, I/0 devices)
	- These appear asynchronously ie does not necessarily a response to a certain event
	- These are determined on the basis of **physical platform**
- **Signals**:
	- Generated by events triggered by the CPU/ Processes that are running on the cpu
	- These are dependent on the OS that is running
	- Could by asynchronous or synchronous
- **Common Things:**
	--> Both will have a unique ID corresponding to them
	--> Mask can be enabled for both
	- Per CPU mask for a interrupt, if this mask is enabled, the interrupt handler will be enabled by the OS
	- If this mask is set, then per process signal handler can be enabled
### Interrupt Handlers
-  Interrupts are delivered to CPU by directly transmitting over the connected data transmission line PCIe for example
- When suppose a disk is connected and an interrupt is supposed to be raised for a disk, it will send a signal/data in the format of INT#(MSI#)  to the CPU
- The CPU will then lookup the interrupt handler table and point the program counter of the current running thread to the instruction table as mentioned in that interrupt handler table 
 ![[Pasted image 20240825194851.png]]

### Signal Handler
- In case of signals handling is somewhat similar, but the signal is raised by OS, hence the signals are defined by the operating system
- The signal handler table is process specific and the signal raised by the OS will have a set of instructions to be run that are mentioned by the process or are process specific
- When a signal is raised, the process can catch that signal and envoke it's own procedure
- ![[Pasted image 20240825195541.png]]
- There are defaults to in an operating system corresponding to each signal, but generally processes catch these signals and manage their own handling
### Why signals need to be disabled sometimes? 
- It's possible that the current thread that is running acquired a mutex, and your handler required that mutex too.
- In such a case when the process is interrupted in b/w and signals is not disabled, it's program counter will start pointing on the the first instruction of the handler, **which would be a mutex lock**, so in that case a deadlock occurs
- A simple solution of this could be to keep your handler simple so that it does not require a mutex lock to run 
- Another possible solution to would be to prepare the signal mask in such a way that that signal could be delayed
	- In a signal mask, if a bit regarding that signal is disabled, the OS simply ignores that signal, until it is set to true
	- So the thread code could be written like this
```
- set_field_to_zero() //interrupt delayed
- lock_mutex()
- critical_section
- unlock_mutex()
- set_field_to_one() //now the interrupt will be handled
```
- This is corresponding to signal masks (which is dependent on the execution context)
- ***In case of interrupts, the hardware would simply not deliver interrupt to CPU***
#### Types of Signals
- ONE SHOT SIGNALS
	- if there are n pending signals of these kind, only once would the handler be called for these and then rest of times, default action of the OS would be taken on them, ie they aren't added to queue, it's just if they are present or not
- REAL TIME SIGNALS
	- if there are n such pending signals, then signal handler will be called n times for these signals
### <u>Interrupts As Threads</u> 
- So essentially during interrupts or signals, is that the stack pointer of the current process will be pointed to the instruction set that is written for that particular interrupt / signal
- A situation is possible here where the main process has taken up a lock and then before unlocking it's stack pointer is updated, in such a case if the signal handler also requires that lock, there will be a deadlock
	- **Solution:** A possible solution could be deal with interrupts as a thread
- So our signal handler, could be divided in a top half and a bottom half
- **Top {Non Blocking } ::** This will analyze if the signal handler for that particular signal requires a lock, if is does it creates a thread to handle that signal
- **Bottom {Could be Blocking} ::** This will be the way in which we handle this signal in a separate thread, and since it's a separate thread we don't need to worry about lock as that will automatically be handled
![[Pasted image 20240904205837.png]]
- This operation of handling interrupts in a thread is somewhat more costlier than the approach, where we change the interrupt mask again and again 
- There's an overhead of about 40 instructions per interrupt that occurs due to this method, but since we don't have change the signal masks again and again, it saves 12 instructions per mutex.
- But since the number of interrupts is less in the system, than the number of lock/unlock operations overall it's a more efficient approach
	- Here in case of lock/unlock we mean the normal operations as well, because in the case where we will change the kernel masks, we will have to do that for **each lock and unlock** because we don't know during which piece of code a signal could occur.
- There could be cases where signal mask is different at User Level / Kernel Level
	- **Case 1:** Both the kernel and the user level has this bit enabled, so no issues in that case, user process will capture that signal on time
	- **Case 2:** It's possible that the user level bit is zero for one thread, but high for another thread
	- And it's high at kernel level, supposing each of these is handled as a single kernel threads
		![[Pasted image 20240905210125.png]]
	- In such a case, a solution would be to wrap the signal handler within the user level library, that would make it possible, so that when such a signal occurs, library scheduling code is called first and it basically gives the signal to the correct thread.
	- **Case 3:** 
	- Here 2 threads are running, both of them have their own kernel thread, but the signal mask bit situation is as follows.
		- ![[Pasted image 20240905211120.png]]
		- So here the thread lib will direct that signal to another kernel level thread
		- ***This is a possible solution, because signals are generated at a process level***
		- If one thread has a seg fault, then which of the threads is unblocked will handle it
		- So it's imp to know that the signal handling routine of any thread could be called
	- **Case 4:**
		- ![[Pasted image 20240905211816.png]]
		- **Note:** Here kernel masks were set to 1 at first
		- So here kernel mask were one and the user masks are 0, so the efficient approach here is first the signal is generated, the threading library sees that if the current working threads doesn't support that signal, it will make a system call to support it's user thread to be 0 and then direct that signal to other threads, which each do this thing and make the signal mask at the kernel level to 0.
### Task Struct in Linux (PCB of linux)
- Add Notes on this with own research later!