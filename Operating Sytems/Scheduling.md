- A scheduler based on it's ultimate goal can generally have 3 choices on how to distribute work to the toy shop / distribute tasks to the CPU
### <u>Choices:</u>
- **Assign Task Immediately**
	- Goal: Scheduling Time is less and Processing is simple
	- FCFS (First Come First Serve)
- **Assign Shortest/Simplest Task First**
	- Goal: Maximize Throughput (basically this will be the first one to finish first tasks as they are shortest task)
	- SJF (Shortest Job Fist)
- **Assign Complex Tasks First**
	- Goal:  Maximize CPU Utilization, Memory, Devices
### <u>Where does scheduler fit in the system:</u>
- The task of the schedule is mainly to decide which process need to be picked from the ready queue and dispatched to the CPU
   - ![[Pasted image 20240917201141.png]]
- A scheduler is generally run:
	- When CPU is idle for eg in the case of I/0 Request
	- When ready queue has a new process so as in fork scenario
		- This is done mainly to see if the new process is having more priority
	- Time Slice Expires, that is the time a process allocated on a CPU is finished
- **Ready Queue Data Structure highly influence what kind of scheduling algo's can a operating system support**
### <u>Run to Completion Thinking:</u>
- While designing algo, and understanding realistic algoritms, we make following assumptions
- Execution time of a process is known to us
- All task arrive at the same time
- **Simple Scheduling:** All tasks run till scheduling that is no preemption
- **FIFO:**
	- Assuming run queue to be a queue actually, and we just add process one by one and execute them in a fifo order
	- Ex: T1 - 1s T2 - 10s T3- 1s
	- **Metrics**:
		- Throughput : 3/ 12s = 0.25 tasks per sec
		- Avg Waiting Time : 0 + 1 + 11 = 4 sec
		- Avg Completion Time : 1 + 11 + 12 = 8 sec
- **SJF**
	- Here we see that run queue, can't be really a queue because then we would have to iterate each of the process to find out the min time taken in a process
	- **Solution:** Make run queue a priority_queue or Tree/ Heap Basically
	- Ex: T1 - 1s T2 - 10s T3- 1s
	- Order : 1, 1, 10
	-  **Metrics**:
		- Throughput : 3/ 12s = 0.25 tasks per sec
		- Avg Waiting Time : 0 + 1 + 2 = 1 sec
		- Avg Completion Time : 1 + 2 + 12 = 5 sec
- SJF much faster than FIFO
### <u>SJF + Preemption:</u>
- We now assume that the **each time a task arrives, scheduler will run and decide which task to be put on priority (based on which is the shortest one in the run queue)
- Ex: T1 - 1S (Arrive at T:2) T2 - 10s (Arrive at T:0) T3 - 1s(Arrive at T:2)
 - ![[Pasted image 20240917205345.png]]
 - Here T2 got preempted as here T1 arrived at 2 sec and was a shorter task, same with T3
- Here we are assuming time of executon of the task, but that is really not possible to calculate accurately, so we perform a heuristic based time estimation technique, ie based on history of the time taken by when that task is formed
### <u>Priority Based SJF:</u>
- Instead of time we could have a situation where these tasks are scheduled/preempted based on their already defined priority ie basically instead of time, priority is imp
- Example
	- ![[Pasted image 20240917210328.png]]
- **Run Queue Structure**
	- Each priority can has it's own queue
		- P1 Queue ======
		- P2 Queue =====
		- P3 Queue =====
	- Whenever a process comes it's added to corresponding level queue and picked up accordingly from the top most priority queue
	- It could also be a tree structure, like in SJS
- **Starvation:**
	- Potential issue in this approach is that High priority tasks keep on coming/ these tasks could take a longer time to execute, leading to low priority tasks waiting forever
	- **Solution:** Make priority a function of time in any of the queue, so a task that is in a low priority queue for a long period of time, will automatically has it's priority increased and put in a higher priority queue
### <u>Priority Inversion:</u>
- This happens when a lower priority task has a lock on resource for which a higher priority task is also waiting, in that case the higher priority task cannot execute and instead we **will have to boost the priority of the lower priority task**, so that it finishes first and release the lock
 - **<u>Priority inheritance</u>**: The lower-priority task holding the lock temporarily "inherits" the higher priority of the blocked task. This allows it to complete its work and release the resource sooner.
- That's why it's important to have information of which thread is locking in the mutex
- Example
- Here T1 has the most priority, but still T3 which had the least priority but came earlier and acquired a lock finished first
	- ![[Pasted image 20240917212335.png]]
### <u>Round Robin:</u>
- The main difference b/w this and the first come first serve is it getting pre-empted as a task is waiting for I/0, which was not the case in run to completion SJFS algorithm
- **By definition round robin includes time sharing, that is it assigns each task a time slice to complete, if it doesn't complete it in that time, is puts it back in the run queue**
- By Chat GPT:
	- Round Robin (RR) scheduling is a preemptive scheduling algorithm widely used in time-sharing systems. In this algorithm, processes are assigned a fixed time slice or "quantum" in a cyclic order. Each process gets executed for the duration of one time slice, after which it is preempted (if it hasn't finished) and placed back in the queue. The next process is then picked for execution.

### <u>Time sharing and Time Slices:</u>

- A time slice is the **maximum amount of uninterrupted running time** that we give a process
- It is to be noted that it is the maximum amount of time that a task can run, if a higher priority task comes along / if the current tasks waits for an I/O operation to be done, it will be preempted earlier than the time quantum
- This **timesharing** is basically the concept of sharing sharing the time of CPU, basically for processes that are generally CPU Bound, it's the only method that can be used to ensure effective utilization of CPU
- This results in faster throughput and waiting time and completion time 
- ![[Pasted image 20240924210441.png]]
- <u>PROS</u>:
	- Short task finish sooner
	- Response time is increased, for example if 3 task come together, at least each of them will be completed for 1s
	- lengthy I/0 operations finish sooner
- <u>CONS</u>:
	- After each time slice we need to spend a certain time in context switching, deciding which task to pick up next **overhead**
	- So it's possible that if we take a very short time for a slice, there will be so much overhead happening and no task actually getting accomplished 

### <u>How long should a timeslice be:</u>
- The shorter the time slice, the more responsiveness we will get out of the system, but we will also face more overhead as well
- How long a time slice would be different for CPU bound tasks and I/0 bound tasks
- **CPU Bound Tasks:**
	- ![[Pasted image 20240924211718.png]]
	- For CPU bound task, we really aren't concerned about wait time as the user want these tasks to finish faster, as compared to waiting for it's response, so as we can see the longer the time slice, the better the throughput and the better the avg completion time, however wait time is imp for I/0 bound task
- **I/O Bound Tasks:**
	- ![[Pasted image 20240924213205.png]]
	- For I/O bound tasks if we assume that in each 1s an I/O operation is called upon, then even in 5s case there will be no difference if both the operation T1 and T2 are I/O bound processes as these will be interrupted by I/O and not the time slice instead
	- So assuming that T1 is a CPU Bound Task, we can see that a shorter time slice means that the throughput is more as well as the wait time is lesser (which is an important factor in I/O bound processes)
	- **Conclusion:**
		- For I/O bound operations 
			- Shorter time slices are preffered
			- I/O bound processes can issue I/O operations earlier
			- Higher CPU and device utilisaton because of this (basically because when an I/O operation is issued I will just pick up the next process)
			- User perceived responsiveness is better
		- For CPU bound operations
			- Longer time slices are preffered
			- Less time spent on context switching
			- Efficient utilisation of CPU and more throughput

### <u>Runqueue data structure for time sharing:</u>
- We can make a data structure such that there exists multiple queues in the data structure of different time slice and priority and based on the type of task ie I/0 task / CPU bound task, we can put these into seperate queues and basically schedule accordingly to reduce time slicing overheads.
			- ![[Pasted image 20240925211743.png]]
- **Imp Point:** How do we decide which task is an I/O based or which task is CPU bound?
#### <u>Multi Level Feedback queue</u>
- We first put whatever task comes in a 8ms time slice
	- If the task yeilds / requests and I/O operation before that it means we are fine with the current queue
	- However if the task uses all of the 8ms on CPU bound activity ie it does not yield before it, then we put the task at somewhat lower time slice queue that is of 16ms
- We repeat this process of demoting and promoting a thread inside this queue data structure
	- ![[Pasted image 20240925212401.png]]
#### <u>Linux O(1) scheduler</u>
- It is called O(1) because it takes O(1) time to select / add tasks 
- Task's having priority 0 - 99 are real time tasks and are scheduled by a real time scheduler
- User task's are given priority 100 - 139
	- By default a user task is given priority 120
	- **Nice Value** : Default Priority + Nice Value == Ultimate Priority
	- Nice value can be b/w -20 to +19, basically mapping it over the whole range of 100 - 139
- ![[Pasted image 20241013130706.png]]
- **Feedback Mechanism** 
	- Linux  increases/decreases the priority of a task based on it's sleep time
	- The more the sleep / idle time ***(this time is basically time spent when the cpu is empty)**
		- It means that the task is more interactive / I/O bound, so we increase it's priority
	- The less the sleep / idle time 
		- It means that the task is more CPU based, as it's using up the CPU more so we decrease it's priority
- **Time Slice and Priority Relation** (Slight Confusion!, need to be cleared up)
	- For **I/O, interactive tasks**, the priority needs to be **increased**, this means that the number priority (priority number) is **decreased**, so like it will be decreased by 5
		- Priority Number - 5
		- ***CONFUSION!*** the diagram indicates in such a case, it's time slice will increase, but since it's an interactive task, shouldn't it's time slice be decreased??
	- For **CPU Bound Tasks**, the priority needs to be decreased, this means that the priority number is increased by 5
		- Priority Number + 5 
		- Same confusion here, should the time slice be increased in this case?
- ##### Requeue Structure in O(1) scheduler
		- ![[Pasted image 20241013131847.png]]
- Each of the tasks have has an **Active Array** and an **Expired Array**
	- Active Array:
		- A task will remain in the active array as long as it's time slice is active
	- Expired Array
		- A task will enter the expired array when it's time slice expires
- When all the tasks in Active Array Are finished, the expired array will become the new Active Array and there will be a new Expired Array made
- This also helps with the aging aspect, that high priority tasks will ultimately use up their time slice value and then be put in the expired array
	- ![[Pasted image 20241013132700.png]]
- This works in O(1), since it's a linked list per task, so selecting can be performed easily, more-over, the task is selected as the first bit that is set ie that has any tasks, so like there are 140 bits and the bits that will be set are the ones basically that will have any tasks, so we will select one of them and basically get the pointer to the linked list that we want

#### <u>Linux CFS scheduler</u>
- **Issues with the O(1) scheduler**
	- Interactive tasks don't get handled properly
		- If the time slice of an interactive task is expired, then it would have to wait for all the active tasks to be put in the expired list before that task is scheduled again, which leads to delays in interactive tasks
	- There is no guarantee of fairness
		- There is no such claim that each of the tasks will get a time slice proportional to their priorities
- Solution
	- In CFS Schedule all the tasks are measure by **VRUNTIME : virtual runtime**
		- This is the amount of time spent on CPU
	- Basic algo for CFS is pick the task with least vruntime, run it till it's greater than the next smallest task, pick the next smallest task
	- To gaurantee preference to priority and niceness , vruntime per cpu unit of time is basically dependent on those, so for a higher priority task, vruntime will increase slowly compared to a lower priority task
		- ![[Pasted image 20241013145232.png]]
	- The runtime for this algo is O(log(n)), as the vruntime of tasks is managed in a red black tree, where addition takes log(n) time and selection can be done in O(1) time.
	- ![[Pasted image 20241013145147.png]]
#### <u>Scheduling in Multi-CPU Systems</u>
- The type of systems discusses are SMP
	- **SMP (Shared Memory Processor)**
		- ![[Pasted image 20241013184917.png]]
	- These have multiple processors that share the same main memory, but have different cache mechanisms and have basically a cache coherence structure
	- L1, L2 Private Caches
	- LLC Common Cache between the CPU 
	- Main Memory is Common
- When a thread is scheduled on a CPU, a lot of it's pages are loaded from the main memory in cache for faster access, so it's reasonable for efficiency reason that scheduler have **cache affinity**
- We can mantain a Load Balance Runqueue per CPU
	-  ![[Pasted image 20241013185635.png]]
- We can mantain a hierarchical scheduler architechture, where a per-cpu runqueue is mantained and the scheduler can load balance b/w the CPU following factors in runqueue
	- Length of the runqueue 
	- Idle time of the CPU
- ##### NUMA (Non-Uniform Memory Access) Systems
	- ![[Pasted image 20241013190011.png]]
	- It is possible to have systems where the main memory is divided amongst the CPU's, basically for each CPU there could exists certain memory nodes that can have faster accesses
	- **Scheduling**
		- In such a case the scheduling need to happen according to node affinity that the thread should be scheduled to the main memory that is having it's pages.
		- In such a case also a hierarchical scheduling structure helps
		- This is called NUMA Aware Scheduling

 ### <u>HyperThreading/ Simultaneous Multi Threading</u>
 
- ![[Pasted image 20241013211152.png]]
- This is the case where in once cpu, multiple registers can be exposed so that a scheduler can run 2 threads on a single CPU instead of once, the context switching among such threads is very fast
- **Basically multiple H/W supported execution contexts are available**
#### <u>How is it Effective?</u> 
- We know already that the condition for context switching is if
	- (t_idle > 2 * t_cntxt_switching)
	- Now it is seen that the time taken to context switch in case of hyperthreading scenarios is much faster than memory access even
	- So essentially if we pair up a cpu bound task and a memory bound task, we will be able to hide the memory latency, and basically use up that time in CPU bound task
- "Chip Multi Threaded Processes Need a new OS Scheduler " by Fedora Et Al
- Assumptions in the paper
	- ![[Pasted image 20241013211936.png]]
- Now comparing up pairing different tasks CPU bound + CPU bound
	- ![[Pasted image 20241013212026.png]]
	- Performance degraded here, infact the performance degrades 2X here
	- **Thing to notice here, that the memory is entirely empty here**
	- Not that the instruction logic is per CPU, so like even if there is hyper threading enables, there would be contention for the ALU and like only one of the tasks will be read at a time
- Memory Bound + Memory Bound 
	- ![[Pasted image 20241013212301.png]]
	- This is some what a better utilization than just having single Memory Bound task, but **here cpu cycles are getting wasted** and there is idle time
- CPU Bound + Memory Bound
	- ![[Pasted image 20241013212436.png]]
	- This leads to complete utilization of the CPU
#### <u>How To Find if the process is CPU Bound or Memory Bound ?</u> 
- TODO: This should be updated more when paper is read
- Basically this slide explains that standard measures such as sleep time would not work here, because process is not idle while it is accessing memory
- For properties like this H/W has counters to inform the resources that a particular process is using like L1,L2, cache misses 
	- perf and the profiling tools read these kind of H/W only to make their reports
- Fedora paper suggest that some kind of hardware counter to track CPI .. cycles per instruction need to be made

#### <u> Fedora Paper</u> 
- Essentially the fedora paper states that cycles per instruction is a good count to measure whether a task is memory bound of CPU bound 
- So in that paper processes are run with a simulator assuming that there is a hardware counter to track this and the scheduler knows this and schedules things accordingly.
	- ![[Pasted image 20241018203722.png]]
	- She ran this kind of workload on a 4 core cpu, and the results were as follows 
		- ![[Pasted image 20241018203907.png]]
     ##### Observations:
     - Instruction per cycle should be max basically, because it indicates good CPU utilization
     - In case of d and c, there were workloads that were closer to each other, that is similar workloads, that is either all CPU Bound and All memory bound paired together, leading to lower IPC
     - In a and b there were mixed workloads essentially that had both cpu and memory paired together, so those had higher IPC due to lower contention of resources
	     - **NOTE:**
		     - What causes contention in a hyper threading platform is the processor pipeline
		     - Basically the ability to read and like execute an instruction is limited to per physical core only, because only one ALU and etc.
			 - ![[Pasted image 20241018204642.png]]
	
     ##### Conclusion:
     - Despite the results, it is seen that in real workloads, the distinction in CPI is not that much and therefore, it can't be the only viable method to distinguish a process as CPU bound of memory bound  i.e even if we calculate CPI of major real work workloads, there values are not so precise to tell about the memory or cpu activity of a process