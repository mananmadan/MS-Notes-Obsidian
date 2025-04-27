##  Proxy
###  <u>Initial Bring-up</u>
-  Create a mutex to govern internal threads 
- Open shared memory segment
- Look for any msg in the named pipe / named msg queue {any msg here indicates that cache is up}
 - These all are steps before launching threads
 - After threads are launched, they will call handle_with_cache
###  <u>Main Flow</u>
- Each thread will call handle_with_cache function
- We will take a lock a mutex here for the entire transfer process (this is will ensure no other threads reads the content is doesn't have to read)
-  Lock Thread 
- Request FileName from cache on command channel
- Wait for reply on command channel
- Use semaphore for further synchronization b/w proxy and cache
	- Will transfer data in chunks of 1024 bytes, when cache  writes it on shared segment, cache informs proxy, this keeps on going until all data is with proxy	
- Unlock Thread 
## Cache
###  <u>Initial Bring-up</u>
- Create a mutex to govern internal threads here (internal as in only for cache)
- Get pointer to the shared memory region
- Create name pipe / name msg queue and send SERVER UP msg here
- Create a queue here using stqueue.h, this will be used to put tasks that proxy gives us, then tasks will be taken out for each threads and handled accordingly
- Launch all the threads
- After threads are launched, all threads will call perform_data_transfer
###  <u>Main Flow</u>
- Each thread will call, perform_data_transfer
- Lock Thread
- Check size of the queue, if there's an element, take it out
- Check if the file exists with us, if it does, perform a fstat on it, to get filesize
- Send YES {FileName} {FileSize} on the msg queue
- Use semaphore further to do synchronization b/w proxy and cache
- Unlock Thread

## Checkpoints
- [x] Launch webproxy, run gfclient_download, print all the requests will all the data from handle_with_cache
- [x] Create shared segement and named pipe, Launch webproxy, see that it waits for the cache server to be up, Launch cache server, see if it sends message to the command channel and the webproxy proceeds
- [x] Complete communication in main flow of proxy to cache and see that content is getting added in the cache queue
- [x] Complete all steps
- [x] Register Signal Handlers for all of them as we will have to clean up shared memory and the msq queue as well


## TODO:
- [ ] Make structures and API's to support sharing of semaphores and char buffer in struct
- [ ] Open n segements and push them into steque
- [ ] Modify thread code according to that