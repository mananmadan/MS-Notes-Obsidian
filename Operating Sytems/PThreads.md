- PThreads -> POSIX Threads, POSIX {Portable Operating System Interface}

### PThreads API:
- ![[Pasted image 20240811200530.png]]
- during pthread_create a start routing and arguments are passed to specify what procedure should a thread execute and with what args
- during pthread_create a pthread_attr_t is passed, which is used to modify the properties of pthread_attr_t
- **pthread_attr_t**: It defines the policies such as stack size, joinable, scheduling policy, inheritence, for the new thread
 - ***By Default this can take NULL Value***
 - Joinable is a property that is different from Birell's paper
 - We can set a thread to detachable state in that way our main thread won't wait for the child threads to be joined and would exit whenever it's work is done

#### Detachable Threads
- **WHY:** The main idea for these kind of threads is for asynchronous logging and stuff, the main program wouldn't wait for these threads to be joined and these would exit whenever their work is done 
- **Detached Threads**: Automatically clean up resources upon termination, run independently, and cannot be joined; they are suited for tasks that don’t need synchronization with the main thread or other threads.
- However, it is to be noted that they share the same properties as joinable threads, of have the same PID as the main task, of having a shared address space as the main tasks as well
- **Note:** is the main thread exits, and a detachable thread is running, the process as a whole is not terminated and it's resources aren't cleaned up. 
	- That's why it's imp for detachable threads to have a clean up mechanism
- ***Example Program to Create Detachable Threads:***
- ![[Pasted image 20240811202208.png]]
- ### PThreads Condition Variables and Mutexes
- ![[Pasted image 20240818112728.png]]
- In Mutexes, birell's mechanism and the pthreads api are pretty similar, both the lock and the unlock statement has to be called on explicitly
- Here we always have to call init on the mutex, using pthread_mutex_init option
	- Here we can specify if the scope of that mutex is in the current process only or if it's across processes
- **TryLock** is also supported in pthread, pthread_mutex_trylock, if tries to lock at a point and returns if we aren't able to lock, doesn't get blocked like the pthread_mutex_lock option
- Basically TryLock is a non-blocking call, and Lock is a blocking call
### Condition Variables
- ![[Pasted image 20240818113541.png]]
- All the mechanism's as suggested in Birell's paper are as such supported in PThreads 
- For init we have to use pthread_cond_init, and for destory we use pthread_cond_destroy
  #### Key Points From Exercises
-  Can initialize condition variables and mutexes in the following way
  ```cpp
  pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
  pthread_cond_t  writePhase = PTHREAD_COND_INITIALIZER;
  pthread_cond_t  readPhase = PTHREAD_COND_INITIALIZER;
  ```
- You don't need a mutex to signal and broadcast, condition variables are thread_safe
- A thread if waiting for signal, will get blocked, so even if the threads that are going to send a signal here are gone, the thread will keep waiting
	- So it's imp that each of the waiting thread do get's a signal
- It's important to **pass Id of each thread while doing a pthread create, as it's very imp for debug purposes, detecting race conditions and deadlocks**