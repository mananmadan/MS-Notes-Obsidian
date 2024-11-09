### <u>Similarities b/w toy show and memory managment </u> 
- "Parts" are stored in intelligent sized containers
	- OS: In operating system memory is stored in pages or segments for uniformity / faster access
- Not all memory is needed at once
	- OS: In operating system some of the memory is in cache which is to be accessed as fast as can be, and some of the memory is in the main memory
- Optimized for performance
	- OS: Reducing time is loading pages is significant to the performance of the operating system

### <u>Memory Management Goals </u> 
- ![[Pasted image 20241024205329.png]]
- To make each process access of physical memory independent of the structure of the actual memory, addresses are divided into Virtual Address and Physical Addresses
- So the main goals of memory management is
- #### Allocation:
	- ##### Page Based Sytem : Pages ==> Page Frames (hardware is actually divided in Page Frames)
	 - ##### Segment Based System : Segments (hardware is divided here in segements)
	- Operating system has to keep track of how much memory and where memory is been allocated on hardware
	- It also has to make sure if that if some of the memory is on a secondary storage ie disk, then the OS will have up to load up those memory onto the main physical memory 
- #### Arbitration:
	- ##### Page Based System : Page Tables
	 - ##### Segment Based System : Segment Registers (hardware is divided here in segments)
	- Operating system has a task of translating a processes virtual addresses to physical addresses
	- It also has to perform validation ie checking whether the addresses getting asked by a process are valid for it .. ie permission etc

### <u>Hardware Support for Memory Management</u> 
- ![[Pasted image 20241024211146.png]]
- #### MMU:
	- The MMU uses page tables to translate virtual addresses into physical addresses
	- It also reports faults when there are issues with the access of some of the virtual addresses
		 - Which could involve permission issues 
- #### Registers:
	- Pointers to the page table (ie basically store pointer to page table of the current process)
- #### Cache Translation Lookaside Buffer:
	- This stores up small number of valid virtual address - physical address, essentially a cache for faster access of lookup
- #### Translation
	- The page table do not store actually hardware location, but some information that is essentially passed onto the hardware which actually does the physical address translation

### <u>Page Tables</u> 
- ![[Pasted image 20241025195333.png]]
 - **NOTE:** The size of the page and the number of page frames at the DRAM is almost similar to the virtual memory, page table just essentially maps the correct one at it's place
 - Due to this property, we **do not need to store mapping of each addr, we can store mapping of starting of the page frame and then calculate offset according to that essentially**
 - ![[Pasted image 20241025195820.png]]
 - VIRTUAL ADDRESS == {VPN | OFFSET}
  - We calculate the PFN(Physical Frame Number) from the page table corresponding to the VPN and then use the offset to calculate the actually addr on the physical DRAM
  - #### Example of an array initialization
	  - When an array is created, the code is compiled and the process is launched, only a virtual address corresponding to it is located 
	  - The physical address allocation happens only when first write happens (**Allocation on the first touch**)
		  - An interesting property regarding this is that when a process hasn't used it's pages in a long time the OS reclaims them
	- **BASICALLY PROCESS KEEPS TRACK OF IT'S ACTIVE PAGES**
		- This is done using a valid bit column in the page table
		- ![[Pasted image 20241025210604.png]]
		- The process keeps track of the valid bit which indicates whether that page is currently allocated in a memory or not
		- In the array example when the process is run, their will be entry in the page table but the valid bit will be 0
		- In case the valid bit is 0, a fault is raised to the operating system
	        - The operating system decided whether to deny access here, or allocate the memory corresponding to that page
			- If it's a valid virtual addr, MMU will allocate space on the H/W corresponding to that.
			- BASICALLY! 
				- MMU Read Page Table, sees a 0, passes the control to operating system
				- Operating system decides, raises a fault is it's invalid, else allocates
		- **Page Tables are per process, whenever these is a context switch, the CR3 Register for example on x86 platform, will point to the page table of the current process**
		  - ![[Pasted image 20241029203223.png]]
### <u>Page Table Entry</u> 
- ![[Pasted image 20241029203337.png]]
- There are multiple bits corresponding to each entry which indicate different things
	-  P (present) .. ie whether the page table is allocated in physical memory or not
	- R/W (read/write) 
		- 0 Only read permission
		- 1 read and write permissions
	- D (Dirty) written to
		- if a page is loaded from the disk and this bit is not set after that, we don't need to write on the disk again
		- essentially this bit tells that if the page is modified or not
	- A (accessed)
	- U/S 
		- 0 means user mode
		- 1 mean supervisor/ kernel mode
- The MMU uses these bits to lookup if the page can be accessed/ is valid or not and then raises a fault, which is managed by the page fault handler, this handler determines it's action based on error code and the faulting addr.
- The action can be
	- Bringing page from disk to mem
	- protection error (SIGSEGV)
- ![[Pasted image 20241029204214.png]]
### <u>Page Table Size</u> 
- Page Table sizes can be increased a lot
- We can calculate the page table size by (Total Virtual Addresses / Page Table Size ) * One Page Table Entry
- One Page Table Entry is essentially including PFN + Flags information 
- For 32 bit system, this can be
	- 2^32 / 4KB * 4B == 4MB per process 
- For 64 bit system, this can be
	- 2^64 / 4KB * 8B == 32PB per process
- **This main issue is that in a single level page table, there has to be an entry for the each of the virtual page number that can exist because a program can access any of the virtual address and hence the OS Need to be ready for that** 
### <u>Multilevel Page Table</u> 
- Instead of basically using a single page table pointing to each of the page at H/W, we will essentially use the outer page table to point to the page of page tables
- **So the optimization here, is that since the internal table is smaller, we can allocate it on demand, whenever we reach an outer address and see the internal for that is not allocated we will allocate it**
-  ![[Pasted image 20241030222255.png]]
- #### To access multi level structure
- So basically if the logical address let's say is 32 bits
	- p1 can be 10bits
	- p2 can be 10bits
	- d can be 12 bits
	-  ![[Pasted image 20241030222738.png]]
- So we can access the whole 32bit addr range through that, but we're able to save space in terms of addresses that are not used by processes
### <u>Translation LookAside Buffer</u> 
- Since adding this multi level page table, memory is saved but lookup time increases, hence this cache can be used which basically stored direct mapping for a virtual addr table to it's physical addr
- **Property::** The virtual addresses accessed by a program are temporal ie are close by, hence this cache can be made small and we will still be able to utilize this for our case
- ![[Pasted image 20241030223411.png]]
- MMU first looks upon the TLB, if it's a TLB miss then a page is allocated in the page table corresponding to it
### <u>Inverted Page Table</u> 
- In inverted page tables we maintain a **global IPT** instead of keeping at a process level, this page is maintained at a global level and is mapped exactly at the the mapping of physical frame numbers as on H/W
- ![[Pasted image 20241104215950.png]]
- Since it's a global page table, we have to search for pid and virtual page frame number (first part of the virtual address)
	- NOTE: **Since this ordering is at the H/W Level basically, this won't be sorted and we would have to do a linear search here**
- To speed up the process of the lookup, hashing is used
- TLB is also helpful in this case, because this linear search will take a lot of time
- #### Hashing in Inverted Page Table
	- Hash Function can be computed based on part of the addresses, like some bits and then the rest of the list could be basically a list composed of the bits that have that common starting bit for addresses and the rest of the bits are different, so that way to find a match we will only iterate that table, instead of computing the whole page table 
	- ![[Pasted image 20241104221421.png]]
- 
### <u>Segementation</u> 
- Segements refer to grouping of related data together, so this essentially means that the entire addr space is divided up into smaller portions ie **Data Segment, Stack Segment, Code Segment, Head Segment**
- PROS:
	- Combining both segments and paging gives us following advantage
	- It allows us to group related parts of code together, leading to better logical organization of programs
	- Paging reduces fragmentation
- PROCESS:
	-  The CPU generates a logical address comprising a **segment number** and an **offset**. The segment number is used to look up a segment table to find the segment's base address.
	- The **offset** within the segment is divided into a **page number** and a **page offset**, which are used to access the corresponding page frame in physical memory.
	- ![[Pasted image 20241108211034.png]]
	- ![[Pasted image 20241108211222.png]]
### <u>Page Size</u>
- Normally the page size is 4 KB (offset of 12 Bits)
- ![[Pasted image 20241108212750.png]]
- PROS OF HAVING A LARGE PAGE:
	- Fewer entries in page table, more hits on TLB as the number of entries are less
- CONS OF HAVING A LARGE PAGE;
	- Internal Fragmentation: So like if a page table size is 1GB, we will allocate it fully even if smaller data is to be used, this leads to wasted memory  

### <u>Memory Allocation</u> 

- Memory allocation incorporates mechanisms to decide what are the physical pages that will be allocated to a particular virtual memory region. Memory allocators can exist at the kernel level as well as at the user level.

- **Kernel level allocators** are responsible for allocating pages for the kernel and also for certain static state of processes when they are created - the code, the stack and so forth. In addition, the kernel level allocators are responsible for keeping track of the free memory that is available in the system.

- **User level allocators** are used for dynamic process state - the heap. This is memory this is dynamically allocated during the process's execution. The basic interface for these allocators includes `malloc` and `free`. These calls request some amount of memory from kernel's free pages and then ultimately release it when they are done.

- Once the kernel allocates some memory through a `malloc` call, the kernel is no longer involved in the management of that memory. That memory is now under the purview of the user level allocator.
### <u>Memory Allocation Challenges</u> 

- Consider a page-based memory management system that needs to manage 16 page frames. This system takes requests for 2 or 4 pages frames at a time and is currently facing one request for two page frames, and three requests for four page frames. The frames must be allocated contiguously for a given request.

- ![](https://assets.omscs.io/notes/FF84D314-D11E-4B0D-9883-5A4B95B81696.png)

- The operating system may choose to allocate the pages as follows.

- ![](https://assets.omscs.io/notes/2E0A9A0A-A838-4A7F-B3F7-AB1A90BC8718.png)

- Now suppose that the initial two request frames are freed. The page table may look like this now.

- ![](https://assets.omscs.io/notes/D858CB66-A95B-4EFB-84D1-AD3448FEBEC5.png)

- What do we do when a request for four page frames comes in now? We do have four available page frames, but the allocator cannot satisfy this request because the pages are not contiguous.

- This example illustrates a problem called **external fragmentation**. This occurs when we have noncontiguous holes of free memory, but requests for large contiguous blocks of memory cannot be satisfied.

- Perhaps we can do better, using the following allocation strategy.

- ![](https://assets.omscs.io/notes/9E026C20-E04E-4CE6-B31F-F4EB073272EA.png)

- Now when the free request comes in, the first two frames are again freed. However, because of the way that we have laid out our memory, we now have four _contiguous_ free frames. Now a new request for four pages can granted successfully.

In summary, an allocator must allocate memory in such a way that it can coalesce free page frames when that memory is no longer in use in order to limit external fragmentation.

## [](https://www.omscs-notes.com/operating-systems/memory-management/#linux-kernel-allocators)Linux Kernel Allocators

The linux kernel relies on two main allocators: the buddy allocator, and the slab allocator.

The **buddy allocator** starts with some consecutive memory region that is free that is a power of two. Whenever a request comes in, the allocator subdivides the area into smaller chunks such that every one of them is also a power of two. It will continue subdividing until it finds a small enough chunk that is power of two that can satisfy the request.

Let's look at the following sequence of requests and frees.

![](https://assets.omscs.io/notes/ABC594B2-8DD5-4767-8E68-E91770CC47D5.png)

First, a request for 8 units comes in. The allocator divides the 64 unit chunk into two chunks of 32. One chunk of 32 becomes 2 chunks of 16, and one of those chunks becomes two chunks of 8. We can fill our first request. Suppose a request for 8 more units comes in. We have another free chunk of 8 units from splitting 16, so we can fill our second request. Suppose a request for 4 units comes in. We now have to subdivide our other chunk of 16 units into two chunks of 8, and we subdivide one of the chunks of 8 into two chunks of 4. At this point we can fill our third request.

When we release one chunk of 8 units, we have a little bit of fragmentation, but once we release the other chunk of 8 units, those two chunks are combined to make one free chunk of 16 units.

Fragmentation definitely still exists in the buddy allocator, but on free, one chunk can check with its "buddy" chunk (of the same size) to see if it is also free, at which point the two will aggregate into a larger chunk.

This buddy checking step can continue up the tree, aggregating as much as possible. For example, imagine requesting 1 memory unit above, and then freeing it. We would have to subdivide the chunks all the way down to 1, but then could build them all the way back up to 64 on free.

The reason that the chunks are powers of two is so that the addresses of budding only differs by one bit. For example, if my buddy and I are both 4 units long, and my starting address is 0x000, my buddy's starting address will be 0x100. This helps make computation easier.

Because the buddy allocator has granularity of powers of two, there will be some internal fragmentation using the buddy allocator. This is a problem because there are a lot of data structures in the Linux kernel that are not close to powers of 2 in size. For example, the task struct used to represent processes/threads is 1.7Kb.

Thankfully, we can leverage the **slab allocator**. The slab allocator builds custom object caches on top of slabs. The slabs themselves represent contiguously allocated physical memory. When the kernel starts it will pre-create caches for different objects, like the task struct. Then, when an allocation request occurs, it will go straight to the cache and it will use one of the elements in the cache. If none of the entries is available, the kernel will allocate another slab, and it will pre-allocate another portion of contiguous memory to be managed by the slab allocator.

![](https://assets.omscs.io/notes/4D7BC1F3-90FB-44C7-8D77-D9D128DB7B62.png)

The benefit of the slab allocator is that internal fragmentation is avoided. The entities being allocated in the slab are the exact size of the objects being stored in them. External fragmentation isn't really an issue either. Since each entry can store an object of a given size, and only objects of a given size will be stored, there will never be any un-fillable gaps in the slab.

## [](https://www.omscs-notes.com/operating-systems/memory-management/#demand-paging)Demand Paging

Since the physical memory is much smaller than the addressable virtual memory, allocated pages don't always have to present in physical memory. Instead, the backing physical page frame can be repeatedly saved and stored to and from some secondary storage, like disk.

This process is knowing as **paging** or **demand paging**. In this process, pages are **swapped** from DRAM to a secondary storage device like a disk, where the reside on a special swap partition.

When a page is not present in memory, it has its present bit in the paging table entry set to 0. When there is a reference to that page, then the MMU will raise an exception - a page fault - and that will cause a trap into the kernel.

At that point, the kernel can establish that the page has been swapped out, and can determine the location of the page on the secondary device. It will issue an I/O operation to retrieve this page.

Once the page is brought into memory, the OS will determine a free frame where this page can be placed (this will _not_ be the same frame where it resided before), and it will use the PFN to appropriately update the page table entry that corresponds to the virtual address for that page.

At that point, control is handed back to the process that issued this reference, and the program counter of the process will be restarted with the same instruction, so that this reference will now be made again. This time, the reference will succeed.

![](https://assets.omscs.io/notes/76DE9169-A186-412A-B18C-EBD53058BB3A.png)

We may require a page to be constantly present in memory, or maintain its original physical address throughout its lifetime. We will have to **pin** the page. In order words, we disable swapping. This is useful when the CPU is interacting with devices that support direct memory access, and therefore don't pass through the MMU.

## [](https://www.omscs-notes.com/operating-systems/memory-management/#page-replacement)Page Replacement

###### [](https://www.omscs-notes.com/operating-systems/memory-management/#when-should-pages-be-swapped-out-of-main-memory-and-on-to-disk)When should pages be swapped out of main memory and on to disk?

Periodically, when the amount of occupied memory reaches a particular threshold, the operating system will run some **page out daemon** to look for pages that can be freed.

Pages should be swapped when the memory usage in the system exceeds some threshold and the CPU usage is low enough so that this daemon doesn't cause too much interruption of applications.

###### [](https://www.omscs-notes.com/operating-systems/memory-management/#which-pages-should-be-swapped-out)Which pages should be swapped out?

Obviously, the pages that won't be used in the future! Unfortunately, we don't know this directly. However, we can use historic information to help us make informed predictions.

A common algorithm to determine if a page should be swapped out is too look at how recently a page has been used, and use that to inform a prediction about the page's future use. The intuition here is that a page that has been more recently used is more likely to be needed in the immediate future whereas a page that hasn't been accessed in a long time is less likely to be needed.

This policy is known as the **Least-Recently Used** (LRU) policy. This policy uses the **access bit** that is available on modern hardware to keep track of whether or not the page has been referenced.

Other candidates for pages that can be freed from physical memory are pages that don't need to be written out to disk. Writing pages out to secondary storage takes time, and we would like to avoid this overhead. To assist in making this decision, OS can keep track of the **dirty bit** maintained by the MMU hardware which keeps track of whether or not a given page has been modified.

In addition, there may be certain pages that are non-swappable. Making sure that these pages are not considered by the currently executing swapping algorithm is important.

In Linux, a number of parameters are available to help configure the swapping nature of the system. This includes the threshold page count that determines when pages start getting swapped out.

In addition, we can configure how many pages should be replaced during a given period of time. Linux also categorizes the pages into different types, such as claimable and swappable, which helps inform the swapping algorithm as to which pages can be replaced.

Finally, the default replacement algorithm in Linux is a variation of the LRU policy, which gives a **second chance**. It performs two scans before determining which pages are the ones that should be swapped out.

## [](https://www.omscs-notes.com/operating-systems/memory-management/#copy-on-write)Copy On Write

Operating systems rely on the MMU to perform address translation as well as access tracking and protection enforcement. The same hardware can be used to build other useful services beyond just these.

One such mechanism is called **Copy-on-Write** (COW). When we need to create a new process, we need to re-create the entire parent process by copying over its entire address space. However, many of the pages in the parent address space are static - they won't change - so it's unclear why we have to incur the copying cost.

In order to avoid unnecessary copying, a new process's address space, entirely or in part, will just point to the address space of its parent. The same physical address may be referred to by two completely different virtual addresses belonging to the two processes. We have to make sure to **write protect** the page as a way to track accesses to it.

![](https://assets.omscs.io/notes/1763C799-DD05-4442-95AD-F57F32314875.png)

If the page is only going to be read, we save memory and we also save on the CPU cycles we would waste performing the unnecessary copy.

If a write request is issued for the physical address via either one of the virtual addresses, the MMU will detect that the page is write protected and will issue a page fault.

At this point, the operating system will finally create the copy of the memory page, and will update the page table of the faulting process to point to the newly allocated physical memory. Note that only the pages that need to be updated - only those pages that the process was attempting to write to - will be copied.

We call this mechanism copy-on-write because the copy cost will only be paid when a write request comes in.

## [](https://www.omscs-notes.com/operating-systems/memory-management/#failure-management-checkpointing)Failure Management Checkpointing

Another useful operating system service that can benefit from the hardware support for memory management is **checkpointing**.

Checkpointing is a failure and recovery management technique. The idea behind checkpointing is to periodically save process state. A process failure may be unavoidable, but with checkpointing, we can restart the process from a known, recent state instead of having to reinitialize it.

A simple approach to checkpointing would be to pause the execution of the process and copy its entire state.

A better approach will take advantage of the hardware support for memory management and it will try to optimize the disruption that checkpointing will cause on the execution of the process. We can write-protect the process state and try to copy everything once.

However, since the process continues executing, it will continue dirtying pages. We can track the dirty pages - again using MMU support - and we will copy only the diffs on the pages that have been modified. That will allow us to provide for incremental checkpoints.

This incremental checkpointing will make the recovery of the process a little bit more challenging, as we have to rebuild the process state from multiple diffs.

The basic mechanisms used in checkpointing can also be used in other services. For instance, debugging often relies on a technique called **rewind-replay**. Rewind means that we will restart the execution of a process from some earlier checkpoint. We will then replay the execution from that checkpoint onwards to see if we can reproduce the error. We can gradually go back to older and older checkpoints until we find the error.

Migration is another service that can benefit from the same mechanisms behind checkpointing. With migration, we checkpoint the process to another machine, and then we restart it on that other machine. This is useful during disaster recovery, or during consolidation efforts when we try to condense computing into as few machines as possible.

One way that we can implement migration is by repeating checkpoints in a fast loop until there are so few dirtied pages that a pause-and-copy becomes acceptable - or unavoidable.