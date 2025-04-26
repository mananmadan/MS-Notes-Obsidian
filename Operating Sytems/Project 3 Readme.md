
## <u>Project Design</u>


![[Pasted image 20250320205540.png]]

### <u>Basic Understanding</u>
- The project for part 1 simply required that Proxy Server form the overall HTTP path by attaching a server path in the filename received from the client
- It also required to get the same page using the easy curl interface and sending back data based on the results to the client
### <u>Design  Considerations</u>
- The main design choice here was whether to store the whole data received from the write call back in curl to memory or as soon a chunk of some specific size completes, sending it to the client 
	- STORING WHOLE DATA IN MEMORY:
		- [PRO:] This leads in a simpler code as didn't really need to handle chunking of data and then sending
		- [CONS:] More memory consumption, as every thread will at least have to maintain whole buffer for the file it queried on.
	- STORING DATA IN CHUNKS:
		- [CONS:] A little bit complex handling of chunking data and calling send in the write call back itself
		- [PRO:] Much more memory efficient, because we only have to store data for a chunk at a time, and can reuse once memory is freed

## <u>Code Implementation</u>
- Implementation is fairly standard on part 1
- Existing code, already implements calling of handle_with_curl function on every call by getfile client
- Inside handle_with_curl function
	- Server Path is passed on to handle_with_curl function through main
	- This path path is added to the already recieved filename from the requested path which is also passed in the handle_with_curl function
	- A write call back is registered with curl, which will essentially be called by the curl function when data is to be written
	- A curl request is performed and it's http code is checked, if it's < 400
		- Then gfs_sendheader is sent, with the buffer size (which got written in write callback)
		- Then data is sent through the gfs_send call
	- If not
		- Then gfs_sendheader with FILE_NOT_FOUND is sent.
## <u>Testing</u>
- I modified the workloads.txt file with both kind of file, file that were present on the github server and the file that were not present as well
- I then ran ./gfclient_download binary with the port number and -r argument, this downloaded the files localy as well.
- I then did a diff on file contents and file sizes which resulted in correct behavior
## <u>Reference</u>
- https://curl.se/libcurl/c/libcurl-easy.html: regarding all the implementation of curl and write call back
- Rest of the code similar to the getfile implementation in the prev projects

## <u>Project Design</u>


![[Pasted image 20250320211734.png]]

### <u>Basic Understanding</u>
- The part 2 extends the first project and introduces a cache server to interact to, proxy needs to establish a communication with the cache process to know whether the cache has loaded that filename or not.
- Then a file transfer needs to happen from cache to the proxy via a shared memory channel
- Proxy needs to further send this information to the client
### <u>Design  Considerations</u>
- There are many possible design decisions to consider here 
- Since the code requirement, clearly states that the **command** channel and the **data** channel should be different.
- So the first decision as to what IPC to use for command channel:
	- Sockets: Not really needed as the cache and proxy are on the same machine, hence ruled this out
	- Named Pipes: This would have been a good choice for a single threaded process, as named pipes don't really establish message boundaries unlike in message queues
	- Message Queues: These take in a long indicating the type of a message, this allows for much more flexible communication in multi threaded scenario, hence decided to go with this option
- Synchronization of shared memory
	- Since shared memory is a shared resource across processes, it needs to be accessed only by one process one thread at a time, also there needs to be a communication b/w proxy and cache, regarding when the data is put on shared memory and when the data is read from shared memory. 
	- Following options were considered and implemented as well:
		- Synch Using Semaphores:
			- This was the preffered option as this comes with sem_post and sem_wait option, which could work directly out of the box, b/w the proxy (reader) and the cache (writer)
			- However, I wasn't able to stabilize this, this was leading to a race condition
		- Synch Using Message Queue:
			- Message queue can be used for synch, as they also have msgsnd and msgrcv option, I ultimately ended up using the same message queue for synchronization over the shared memory
			- The proxy posts a different type of message when data is read, and the cache posts a different type of message indication that the data is written on shared memory
- Handling of N segements
	- This was pretty clear, here my ideal design was to keep a flag in the shared memory indication that the segment is in use or not
	- However, this was easily done with a queue, I initiate N segments in the starting when proxy is launched and insert them in a queue, which threads access and then remove and add segments depending on first come first serve basis
## <u>Code Implementation</u>
- The code implementations were mainly on the side of communications which can be divided into two parts, initial bring up and thread communication
- This flow is also described in the image placed under the project design header image
- INITIAL BRING UP:
	- Proxy:
		- Open statically named message queue 
		- Go into msgrcv blocking call, it waits for a "SERVER UP" message from the cache
		- Send a string "NUMBER OF SEGEMENTS, SEGEMENT SIZE" to cache
		- Initialize segements here as well
	- Cache
		- Open statically named message queue
		- Send "SERVER UP" msg on to the queue
		- Wait for the string from proxy
		- Process this string and initialize segments 
		- Make a queue supposed to be containing the requests from proxy
		- Launches all the threads, with the access to this queue in args
- THREAD COMMUNICATION:
	- Proxy:
		- Proxy receives handle_with_cach file from with a new file request
		- It takes a lock (this mutex is process specific)
		- Sends a message on the message queue with FILENAME
		- Proxy waits for the FILESIZE message from the cache, if the recieved file size is 0, proxy directly sends a header with no file found and simply returns
		- Else the proxy sends GF_OK with the file size recieved
		- After this the communication over shared memory segement starts
		- Proxy waits for the msg from cache regarding and copies data in memory and then sends it to client through gfs_send
	- Client:
		- The client main threads runs a while loop with a usleep call which adds in the requests from proxy
		- This queue is populated as and when the request comes
		- One of the threads, take a lock and pop out this request
		- It tried to get a fd corresponding to the given file, using simplecache_get
		- If we do get a fd, we do a fstat on it and put a msg again on msgqueue with FILENAME, FILESIZE
		- Client reads the file and puts a chunk of buffer size(size of the segement) onto the shared memory segement and posts a msg to proxy to read the data
## <u>Testing</u>
- Tested using multiple way
- Using the gfclient_download binaries, a lot of race conditions were debugged this way
	- Added custom files to workload.txt and locals.txt for the cache
- Also used netcat to do a SIMPLE GETFILE for a big file and wrote a script that repeats it multiple times, to test the output on the same file
- Also used the gfclient_measure executable with launching multiple threads and num of requests to test the performance of the system
## <u>Reference</u>
- https://www.geeksforgeeks.org/ipc-using-message-queues/
- https://www.geeksforgeeks.org/introduction-of-shared-memory-segment/
- https://www.geeksforgeeks.org/ipc-shared-memory/