
## <u>Project Design</u>


![[Pasted image 20250419201429.png]]

### <u>Basic Understanding</u>
- Part1 required to implement basic grpc utilities.
- Following services were implemented:
	- Store File
	- Get File
	- Delete File
	- List Files
## <u>Code Implementation</u>
- Each of the operations get implemented in the proto file
- <u>Server Side Implementations</u>
- Store File:
	- This is actually broken down into 2 calls from client to server
	- 1st is StoreFileHeader, which sends the information about the file name and the file size, that is to be sent to the server
	- The server stores this information in a member variable
	- 2nd is StoreFileBuffer, which actually sends out the char stream of the concerned file.
	- The StoreFileBuffer, opens an ofstream for the concerned file and reads data untill filesize data is read and writes it on the disk
- Get File:
	- This is implemented by the call FetchFileBuffer, which takes into input a FileHeader message, this message contains information about the file name that is being requested by the client, the server sends a char stream to the client, which then writes it on the disk on which the client is mounted 
- List File:
	- This is implemented by the call ListFiles, since we have to list all the files here, this call takes nothing as an input, and output a map of file names and their modified time.
	- Server essentially iterated on the entire dir on which it is mounted and does and fstat on all the files, puts this information in a map and then sends it to the client
- Delete File:
	- This is implemented by the call DeleteFile which takes in the input FileHeader msg, which is essentially info about the file that needs to be deleted, the server deleted the file using std::remove, and then returns a fileAck on the basis of the status of this command
## <u>Testing</u>
- Tested with custom shell script written which tries out different operation fetch, list and delete files on image files, and then uses diff to see if these are matching or not
## <u>Reference</u>
- https://www.geeksforgeeks.org/basics-file-handling-c/ 
- https://grpc.io/docs/languages/cpp/: for different message types and syntax

## <u>Project Design</u>
![[Pasted image 20250419200850.png]]

### <u>Basic Understanding</u>
- Part2 extends part 1 in such a way that there can be multiple clients
- Client will have 2 threads, an notify threads that detects if there a change in the disk that is mounted on client
- This thread is mainly responsible for maintaining a common state b/w the server and the client 
### <u>Design  Considerations</u>
- <u> Synchronized File Handling</u>
	- Since there could be multiple clients, the server needs to handle locking on each of the file, since all the clients cannot write to the file at the same time, also there has to be some sync in the when can the clients read the file (scenario such as a client is updating the file and the read happens in b/w)
	- MOST OPTIMIZED IMPLEMENTATION:
		- The most optimized handling would be to maintain a how many readers and writers are currently trying to access the file
		- Based on that, if the number of writers !=0, then we would wait for the writers to become zero (ie spin the lock in the meantime), and then issue a lock on the file
		- If the readers != 0, but writers = 0, then we can read the file, without facing any issues
		- Similarly if writers !=0, then we will error out with RESOURCE_EXHAUSTED
## <u>Code Implementation</u>
- Locking Related Functionality
- I mantain a map for storing which client currently has a lock on a particular file
	- getLockForFile (filename, client id)
		- adds client id to the list of the clients already taking lock on this file
	- checkIsMyLock (filename, client id)
		- checks if the client id is part of the list of clients that have lock on this file
	- releaseLockForFile
		- deletes all clients, waiting on this file
- The service corresponding to Locking is  GetLock
	- GetLock
		- checkIsMyLock(filename, client id)
		- If lock does not exists, then take a lock using getLockForFile (filename, clientid)
- Here again following operations needed to be supported	
- Store File :
	- checkIsMyLock for the given client id
	- If lock is not there, exit with RESOURCE_EXHAUSTED
	- else, check if the file with the given filename already exists
	- If it does, match the checksum of the file, to check if it's the same file, if not then overwrite the file
	- If checksum matches return ALREADY_EXISTS
- Fetch File:
	- getLockforTheFile for the given filename and client id
	- check if the file exists, if it does match the checksum received from client id, return ALREADY_EXISTS
	- If it does not, read the file in the server and send it to client in chunks
	- After sending file, releaseLockForTheFile
- List Files:
	- This goes through each of the file on the directory where server is mounted on
	- Get's a lock on those file
	- Does a stat call to get the information about the files
	- Releases Lock on the file
	- Uses that stat information to populate the map, that is sent to the client
	- The map is popula
## <u>Testing</u>
- Tested with custom shell script written which tries out different operation fetch, list and delete files on image files, and then uses diff to see if these are matching or notted with filename and the last modified time
- Also tested by mounting an entire disks of files, to the client and see if the server and client are able to come to an equal state eventually 
- Then I modified/ deleted/ added multiple files on this disk to check if the client and server syncs
## <u>Reference</u>
- https://www.geeksforgeeks.org/basics-file-handling-c/ 
- https://grpc.io/docs/languages/cpp/: for different message types and syntax
- https://en.cppreference.com/w/cpp/thread/mutex