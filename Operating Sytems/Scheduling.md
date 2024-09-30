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
	- Response time is increased, for example if 3 task come together, atleast each of them will be completed for 1s
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
			- I/O bound processes can issue I/o operations earlier
			- Higher CPU and device utilisaton because of this (basically because when an I/o is issue I will just pick up the next process)
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
- 