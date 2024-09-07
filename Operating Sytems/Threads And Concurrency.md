### Process vs. Thread
A single-threaded process is represented by two components:
* address space (virtual <-> physical memory mapping)
	* code
	* data
	* heap
* execution context
	* CPU registers
	* stack

All of this information is represented by the operating system in a process control block.

Threads represent multiple independent execution contexts within the **same** address space, which means they share the virtual to physical address mapping, as well as the code/data/files that make up the application.

However, they will potentially be running different instructions and accessing different portions of the address space at any given time. This means that each thread will have to have its own registers and stack.

Each and every thread has its own data structure to represent information specific to its execution context.

A multithreaded process will have a more complex process control block structure, as these thread specific execution contexts need to be incorporated.

![](https://assets.omscs.io/notes/984D3189-6CAC-4EDA-8C68-D0315AD3BED6.png)

### Benefits of Multithreading
At any given time when running a multithreaded process on a multiprocessor machine, there may be multiple threads belonging to that process, each running on a given processor. While each thread is executing the *same code* (in the sense of same source code), each thread may be executing a *different instruction* (in the sense of different line or function) at any given time.

As a result, different threads can work in parallel on different components of the program's workload. For example, each thread may be processing a different component of the program's input. By spreading the work from one thread/one processor to multiple threads that can execute in parallel on multiple processors, we have been able to speed up the program's execution.

Another benefit that we can achieve through multithreading is specialization. If we designate certain threads to accomplish only certain tasks or certain types of tasks, we can take a specialized approach with how we choose to manage those threads. For instance, we can give higher priority to tasks that handle more important tasks or service higher paying customers.

Performance of a thread depends on how much information can be stored in the processor cache (remember - cache lookups are super fast). By having threads that are more specialized - that work on small subtasks within the main application - we can potentially have each thread keep it's entire state within the processor cache (*hot cache*), further enhancing the speed at which the thread continuously performs it task.

#### So why not just write a multiprocess application?
A multiprocess application requires a new address space for each process, while a multithreaded application requires only one address space. Thus, the memory requirements for a multiprocess application are greater than those of a multithreaded application.

As a result, a multithreaded application is more likely to fit in memory, and not require as many swaps from disk, which is another performance improvement.

Also, passing data between processes - inter process communication (IPC) - is more costly than inter thread communication, which consists primarily of reading/writing shared variables.

### Benefits of Multithreading: Single CPU
Generally, are threads useful when the number of threads exceeds the number of CPUs?

Consider the situation where a single thread makes a disk request. The disk needs some amount of time to respond to the request, during which the thread cannot do anything useful. If the time that the thread spends waiting greatly exceeds the time it takes to context switch (twice), then it makes sense to switch over to a new thread.

Now, this is true for both processes and threads. One of the most time consuming parts of context switching for processes is setting up the virtual to physical mappings. Thankfully, when we are context switching with threads, we are using the same mappings because we are within the same process. This brings down the total time to context switch, which brings up the number of opportunities in which switching threads can be useful.

![](https://assets.omscs.io/notes/4B5064D0-7B6A-4EEC-9503-895184EA5707.png)

### Benefits of Multithreading: Apps and OS Code
By multithreading the operating system kernel, we allow the operating system to support multiple execution contexts, which is particularly useful when we do have multiple CPUs, which allows the execution contexts to operate concurrently. The OS threads may run on behalf of different application or OS-level services like daemons and device drivers.

![](https://assets.omscs.io/notes/96B2D50D-5518-486F-AFA3-9FC1CAFB5566.png)

### Thread Mechanisms
- **Problem**: Two processes operate on different physical address space, might have same virtual address, but definitely different physical address space. But threads share the same virtual as well as the physical address space, ***page tables for different threads are the same
- **Solution**: We need concurrency mechanisms to manage the access of common data
	- **Mutex:** We maintain a mutex on resource, this means that some other thread is using it and the next thread will wait for it's turn to use that resource.
	- **Condition Variables:**  Here the thread waits for other threads to complete a certain set of action and then that thread does it's work [not very clear what's this]

### A Logical Model to Thread Creation
- To create a thread we need to have a thread data structure, which will store the thread id, program counter, stack pointer, registers, stack attributes
- To create a new thread from a process , we sort of do a fork call ***not unix fork*** this call takes in a proc, and an arg, proc is procedure that threads need to execute, the stack pointer of the thread will be exactly at the code where fork is called and after that it will execute the proc instruction basically
	- ***Note***: Like when a new thread is created it's program counter will basically point out to the proc sample
- After the thread finished, we call a join (thread id) operation
- This operation, waits for the thread to exit and returns it's result to the parent child
-  A join operation is basically blocking operation and blocks the process until that thread is finished and it returns a result

### Mutex 
- ![[Pasted image 20240729224036.png]]
- This is used to lock a shared resource of code (which is called a critical section) a lock is indicated in such a way that other thread's will be blocked until the current owner of that threads, completes that code and releases this code
### Producer-Consumer And the need for condition variables
- ***Problem Statement:*** Let's say there are many producer's and only one consumer, there is a shared list where the producers will insert elements and the consumer is only supposed to read and empty the list when it get's full
- **Solution With Mutex**: 
 ```
 Producer()
 {
   Lock(m);
    if(sharedList.isNotFull())
      sharedList.insert(element)
   Unlock(m)
 }
 Consumer()
 {
   Lock(m);
   if(sharedList.isFull())
     sharedList.PrintAndRemoveAll();
   Unlock(m);
 }
```
- Here the consumer operation is wasteful as each time it get's a context, it's checking whether the list is full or not and then trying to do some operation, most of the time this will be a wasted operation
- ***Solution with Condition Variables***
 ```
 condition_variable listFull;
 Producer()
 {
   Lock(m)
   if(sharedList.isNotFull())
     shareList.insert(element)
   if(sharedList.isFull())
     signal(listFull);
   Unlock(m)
 }
 Consumer()
 {
    while(!sharedList.isFull())
      wait(m,listFull); //this will automatically aquire lock and will basically awake only when list full message is given
	 sharedList.printAndRemoveElements();
	 Unlock(m);
 }
 ```
### Condition Variables
- API: (General Abstraction as to what a condition variable must implement)
	- Wait(mutex,cond) : mutex is automatically released and reacquired in wait when the cond is met, generally it's used inside of while(!condition) wait(mutex,cond) to avoid spurious wakeups
	-  Signal(cond): this is to signal a signal thread to wake up, when the condition is met
	- BroadCast(cond): this is to signal all the threads to wake up, when the condition is met
- Internal Logic Inside Wait 
	- ![[Pasted image 20240810201833.png]]

### Reader Writer Problem
- ![[Pasted image 20240810204031.png]]
-  In this problem, there are multiple threads of which 1 is writer and rest are readers
- We have to code in such a way that reader's can simultaneously read the resource, but parallel writer's cannot write on the resource.
- The main solution is protecting the whole resource under a mutex, but that blocks the readers also from reading that resource simultaneously.
- Hence to solve this problem, we create a **resource_counter** and use that to manage access on the resource
- Each time a reader get's created, we increment the resource_counter, and each time a reader exits the critical section we decrement the resource_counter.
- If the resource_counter == 0, then we know all the readers are done, we write data and signal to all the readers that write is done
- **Main Point**: The read data is not protected under a mutex, so while reading each thread comes, increases the resource_counter, reads the data, decrements the resource_counter. 
### Common Pitfalls
- **Spurious Wakeups**:
	- This refers to the phenomenon, that the condition variables exhibit.
	- They wake up irrespective of the condition being met, hence a while loop should always be placed before, so that it ensures correctness 
	- ``while(!condition) {condition_variable.wait()}
	- so even if the condition variable get's woken up, it waits for the condition to be satisfied
- **Ineffiecient Signalling**:
	- This can be caused by for eg issuing a broadcast operation before releasing a mutex, although this is functionally correct, but this will make the system slow
	- As many threads will wake up but they won't be able to do anything, unless that mutex is unlocked()
	- This can be avoided by making sure that mutex is unlocked before calling a broadcast or signal operation
- **DeadLocks**:
	- ![[Pasted image 20240810211506.png]]
	- This is caused, when there's circular waiting, like A + B Operation but A is locked by T1 and B is Locked by T2, so both are waiting for each other
	- There is no particular solution for this