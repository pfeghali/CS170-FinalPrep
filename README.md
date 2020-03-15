# CS170-FinalPrep
Please note: this document is *heavily* reliant on class lecture slides. This is intended to be a cleaned version of the slides. This has been very edited, and will continue to be edited as we approach the end of the course. Please let me know if there is anything that you'd like changed. Sorry for the typos.
### What we expect an OS to do
OS needs to 
 - Manage processes
 - Memory management
 - Storage management
 - Protection and Security

Needs to have high performance, reliability, security, and portability
### Computer System Operation
First, a bootstrap program is loaded on boot - stored in some sort of ROM generally, also known as firmware.  
I/O and CPU can execute concurrently when power is on.  
To manage these devices and their controllers effeciently the CPU is interrupt driven.  
A *trap* is a software generated interrupt caused by an error or a user request.  

### OS System Structure
*Multiprogramming* organizes jobs so CPU is always working. Job selection is cone via *job scheduling*. When the OS needs to wait for something to complete, it can work on something else.  
*Timesharing* isa logical extension, beng that an OS wants to seem fast. This is when the CPU switches jobs so frwquently that a user can not notice that it is not working on other work concurrently. This is implemented via context switching.  
As long as the response time is within some threshold, this is a non-issue.  
We define programs running in memory as *processes*.  
Processes run in *virtual memory* managed by the operating system, allocated usch that programs run in an isolate enviornment. This done for security and consistency.
### Program execution contexts
Programs can either run in *user mode* - where an OS's user is the one delivering work - or *krenel mode*  - where the OS is doing work.  
When defining a program, there is a mode bit which is provided by hardware, to state which mode a process is in.  
User programs should interact and interface with kernel programs through a consistent and safe set of functions provided by the operating system. this if rosecurity and sefty, such that a malicous program can not mess with the rest of the system.  
### Memory and Storage Management
OS needs to keep track of which processes own what work.  
It also needs to allocate and deallocate memory to different programs as it becomes necessary.  
The operating system provides nuderlying programs with a logical view of how to interact with both storage - via a *file*.  
Files can have varying underlying mediums and also can have various contexts - read, write, append, etc.  
The operating system provides a basic set of primitives for interacting with the file system:
 - Create
 - Read
 - Write
 - Delete

### Protection and Security
*Protection* is any mechanism controlling process access to resources defined by the OS.  
*Security* is the defense of the system against internal and external attacks.  
Permissions are defined on a number of basises:
 - User identities, where UIDs include names, unique ids, and can be used to define control to files and low-level information.
 - Group Identities allow user to join groups. The OS can allow for entire groups to have generic access to files.
 - Privilege Escalation allows a user to gain more privileges in the system.

## Interfacing with the OS
### System utilities
The OS provides utilities for
 - Process status and managenement
 - File and directory manipulation
 - File modification and text processing
 - Programming language support
 - Progam loading and execution
 - Communications
 - Application programs

Users generally interact with this higher level interface. This interface is a set of *system calls*. These APIs provide a portable and simple layer for users to interface with the OS

### File Descriptors
When opening a file, a file descriptor is returned. Integer which points to a metadata structure. Function takes in a path and flags describing read, write, etc.  
When done with the file, `close(fileDescriptor)` closes the file.  
Each process has a file descriptor table which points to a system-wide file table.  
`read(fileD, buffer, count)` Read from some file to a buffer, count number of bytes.  
`write(fileD, buffer, count)` Write from buffer to a file, count number of bytes.  
As you write, their is a seek offset which is contiuously updated as you write more bytes. Ensures that you write to a new place in your file.

## OS Structures
There are generally monolithic and microkernel structures. A monolithic kernel is one in-which the kernel provides most functions, and in doing so can manage all tasks in a high-performance manner.  
Microkernels are lighterweight and implement less features, allowing them to be easily extended. They are generally more secure and more reliable, since there is less that can fail.  
We can also imagine a virtual machine, where a host provides an illusion of an underlying host to some running programs.

## Process
*Process* is a program in execution. In memory this includes a program counter, the stack and heap, and data/instructions.  
Parent processes create children. Processes are identified via *PID*.
There are options in resouorce sharing: parent and children share all resources, some, or none.  
Parent and children execute concurrently, and the parent wasits for the children to terminate. Children duplicate the address space of the parent.  
*`fork`* creates a new process with a duplicated address space  
*`exec`* is used after fork to replace the memory space with a new program.  
Code example from lecture:  
```c
int pid;
if(pid = fork()){
    exec*("program",[argvp, envp]);
    exit(status);
} else {
    pid = wait*(&status);
}
```
Fork returns PID to parent and 0 to the child.  
All open FDs are duplicated in children, and read/write seek offsets are shared.  
## File I/O standards
 - 0 stdin
 - 1 stdout
 - 2 stderr

```c
dup2(int oldFd, int newFd);
```
Creates a copy of the oldFd for newFd so they refer to the same table entry.  
## Process Communication
After a parent executes the last statement, it asks the OS to delete it. Resources are deallocated.
Parents may terminate child processes.  
Processes communicate via IPC - interprocess communication. This can be implemented in a few ways, primarily shared memory and message passing. Processes that know each other directly can send direct messages via IPC commands, otherwise can send/recieve via ports.  
Generally this can be implemented via
 - Unix Pipes
 - Unix Signals
 - Shared memory IPC
 - Sockets
 - Remote Procedure Calls - IPC

File I/O can be used as communication between processes -  essentially view files are communication channels.   
unix pipes create a pipe with two pointes, the first to read, the second to write.  
Unix signals can be used to communicate, similair to hardware interrupts without priority. They can be called in unix via `kill` or `killall`. Singal handlers are defined to handle them.  
Can share memory between processes.

## OS Running Programs
general structure of running a program is as follows: First load instruction and data segments into memory. Then create a stack and heap. Transfer control to the program, and provide services.
### Spaec Usage
Space is split into:
 - Stack - for function call frames
 - Heap - for dynamically allocated space (`malloc`)
 - BSS - for uninitialized global variables and static variables - initially zero
 - Data - initialized global or static variables
 - Text - binary code + constants

### Executable File space usage
 - header
 - Text - program instructions
 - iData - immutable data
 - wData - writable data
 - Symbol table
 - Relocation records

(last two used by linker - can be removed at the end)

## Memory protection
Simple memory protection can be done by leveraing bounded memory addresses. Anything outisde of some scope is invalid essentiallly. This is suboptimal, as it requires for continuous memory addressing.  
Address space translation can help. Translate safe virtual addresses to new physical addresses.  

### Process State
 - New
 - Running
 - Waiting
 - Ready
 - Terminated

### Switching from one process to another
What needs to be saved to restore? Program Counter, Registers. Save data in *Process Control Block*.
## Process Control Block
Sores process state, PC, program number, copy of CPU registers, CPU scheduling information, memory management information, accounting information, I/O status information.  
When switching processes, we call this *context switching*. Context is defined in the PCB.  
## Process Scheduling Queues
 - Job Queue - set of all processes in then system
 - Ready Queue - set of all processes residing in main emory, ready and waiting to execute.
 -  Device queues - set o f processes waiting for an I/O device

There is multi-level scheduling.
 - Long-term scheduling which selects which processes should be brought into the ready queue - invoked on the minute/second level.  
 - Short-term scheduling involves selecting what process should be executed next on the CPU, executed on the millisecond scale.

## Threads
A *thread* is a fnudamental unit of CPU utilization that forms the basis for multithreading  
Implemented with pthreads, java threads, etc - and threads are associated with a process.  
Threads provide shared memory access for code and data, though with seperate control flow, stack and regs.  
When switching threads, need to save PC and registers. They're saved in a *thread control block*.  
Multithreading provides:
 - faster and more responseive programs
 - resource sharing
 - scales to more resources easily

Execution is interleaved between sets of threads, and on multicore systems these threads can execute their work concurrently.  
### Threads vs Processes
Similair as they both have logical control flows, can run concurrently, and each is context switched.  
Different as threads share code and data, and processes typically do not, and threads are cheaper with less overhead.  
### Creating a pthread
```c
pthread_t thread0;
pthread_Create(&thread0, NULL, Printhello, (void*) 0);
```
Must be compiled and linked with the pthread library:
```sh
gcc -o -lpthread
```
Join allows for one pthread to wait for another to finish and then they can recombine.  
`pthread_yield()` allows one thread to yield for another pthread.  
### User threads vs Kernel threads
Quite obviously, and  user threads are not real threads. User threads are user managed, and while they ar technically a 'thread', the user has to manage them and to allow them to run on multiple cores. Since they are nt system recognized, they run serially by default unless the system knows to schedule work seperately on multicore systems.  
Kernel threads are supported by the system and the oS supports swithcing and such.  
When context xwitching there is overhead, and nothing useful done during that time

## Synchronization
Gieven that a pipe is shard between a reader and a writer, how do you ensure that no one os reading while some other thread is writing? We can generalize this even further, given a set of readers and a set of writers, how do we mediate interactions fo these users.  
*Race Conditions* are when threads/processes are reading and writing on shared data and the result depends on who runs first.  
*Critical Sections* are parts of the program where the code needs to be synchronized intelligently.  
Important properties:
 - Mutual Exlusion - only one thread/proc can enter a critical section
 - Progress - If some process needs to enter a ccrit section, then one can enter for a limted amount of time - essentially do not starve other procs.
 - Bounded waiting - If one process starts to wait, there is a limit on the number of other procs hwich can enter first before we do, inotw, do not starve.

## Locks and things
### Locks
Threads wait if there is a lock, enter the critical section after acquiring a lock. One access completes, it releases the lock. Only the thread which locks the lock can unlock it.  
### Semaphores
A semaphore is a lock representative of a non-negative integer. This integer is dynamic and represents the number of threads which can be in a shared section. Access is granted via atomic operations.
 - `P()` = wait - waits until counter is greater than 0
 - `V()` = signal - signals waiting threads that their is space and increments the counter

Note that a binary semaphore can implement a mutex lock.
### Conditional variables
We use conditional variables with locks to specify more general waiting conditions. Threads are blocked and placed in a waiting queue - woken up after receieving a signal.  
Main functions:
 - Wait
 - Signal
 - Broadcast - wakes up all waiting threads on that condition

Conditional variables and locks can implement semaphores. Semaphores can implement conditional variables and locks.

## Use of synchronization primitives
We can control shynchronizatoin at a fine-grain or coarse-grain level. fine-grain synchronization allows for more performance flexibility and scalability, though potentially more design complexity.  
Coarse grain is easier to ensure correctness, but harder to scale.  
*Deadlock* - two or more threads are waiting indefinitely for an event that can be only caused by one of these waiting processes.
*Starvation* - indefinite blocking on some thread.  

### Generic scheduling types
Hoare-style scheduling is where a signaler gives a lock, waiter runs immediately  
Mesa-style scheduling is where the signaler keeps a lock and continues to process, while the waiter is placed on a ready queue. Therefore no need to recheck the condition after a wait. 

Users need atomic primitives.  
We can avoid context switching by disabling interrupts, but only for trusted users. If using this method, it is important to keep the critical section very short.  
Rather than doing this, we usee atomic hardware instructions. These are instructions such as testAndSet, swap, compare and swap, etc.
```cpp
bool TestAndSet(*target){
    rv = *target;
    *targer = true;
    return rv;
}
```
An example with a spin lock:
```cpp
bool lock = 0;
while(TestAndSet(&lock)) ;   // Acquire
                // Critical section;
lock = False;   //Release lock
```
You can do a better job with test&set + sleep. Spin locks work, but they're ineffecient and they waste plenty of CPU cycles. No guarantee on bounded waiting.  
Consider:
```cpp
int guard = false;
int value = FREE;
Acquire() {
    while (test&set(guard));
    if not busy, sleep
    value = BUSY;
    guard = false;
}

Release() {
    while(test&set(guard));
    wake up others
    value = FREE;
    guard = false
}
```
## Main memory and address translation
Memory management needs to keep track of which parts of memory are being used by whom. Memory management must allocate and deallocate space as needed.  
*Address space translation* is the process of moving the address space which the program operates in to the physical address space of the machine.  
The *logical address* is generated by the CPU, and is also erferred to as the *virtual address*. The *physical address* is what is actually seen in memory.  
### When do we translate?
Sometimes, if the location is known beforehand, you can do address translation during compile time to generate absolute code.  
It can also be done during load time, though code must be generated to be relocatable.  
Lastly, it can be bound at execution. Libraries such as DLLs (Dynamic Link Library) are built to support this.
### Fragmentation
 - External: Free gaps between allocated chunks
 - Internal: Don't need all memory within allocated chunks.
### Methods to allocate memory
#### Contiguous Allocation
The OS is held somewhere in low memory. Base register cotains the smallest physical value. Logical addresses need to be less than the limit register.  
Programs have defined base and limit registers. This process leads to *memory fragmentation* since memory does not take up space that can easily be given to another program.
#### Segments
A program is a collection of segments. Each segment is a unit with some semantic view. Each segment is contiguous, and has a base and a limit.  
This method has a high overhead, and hard to find a place for new segments. Leads to more fragmentation.  
#### Paging
Divide memory into fixed-size pages. Divide logical memory into blocks of the same size, called *pages*.  
We use a page table **per process**.  
### Address Translation Scheme
Address generated by the CPU is split into page number: **p** and offset: **d**.  
Given a 10-bit d, we implictly have 1024 byte pages. The virtual page number is the remaining bits.  
```
virtual address = virtual paeg number + page offset;
page table{
    virtual page number->physical page number;
}
physical address = physical page number + page offset;
```
### Memory Protection
Memory protection is implemented by associating permission bits with each page.
Valid / Invalid bit to tell whether the associated page is in the process' logical address space, and is legal.  
Read/Write permissions, etc can also be there.  
Dirty bits as well, where we can know if a page has been written to and not written back yet.  
### TLB
*TLB* is a translation look-aside buffer. They are associative registers, and act as fast hardware caches specifically for paging and such.  
TLBs are generally fairly fast, roughly a clock cycle access time. If you miss the TLB, then need to go find the page table in memory/cache.  
```
Effective Translation cost = Cost(TLB Lookup) + Cost(Full Translation)*TLB miss rate
```
Consider a 16 byte page, which holds 4 ints per page. If we get int 1, for example, due to spatial locality, we also get int 0, 2, and 3, into the TLB for free.  

### Constraints
Each page table needs to fit within a physical memory page. This is since a page table needs consecutive space.
 - `Maximum size = #entries*page size`

Consider a one-level page table:
 - 32 bit address space, 4kb per page. Page table would contain 2^32/2^12=1 million entries. 4 bytes per entry, 1M*4=4MB page table, which is often not available.  
  - #entries = 4kb/4bytes = 1K
    - Max logical space = 1k*4kb=4MB

### Multi-level page tables
Consider an outer page table, which points to another page table, which points to memory. The first page table is mapped to via a portion of the original address, the second page table with another portion of the address, etc.  
The benefit is that each table has a fixed size, fitting easily into one physical frame.  
Properties:
 - #bits for d = log(page_size)
 - #bits for pageTable1 >= log(# entries in L1 table)
 - #bits for pageTable2 = log(# entries in L2 table)
 - #physicalPages = 2^entry bits in L2
 - Physical space size = #physical pages\*page size
 - Virtual space size = # entries in L1 table \* # entries in L2 table \* page size
   - This may be smaller than the max values of p1\*p2\*page size.

#### Example
Memory page with 4KB holds 1KB entries, 4B each. P1 >= log(L1 entries) -> 1K entries = 10 bits. P2 = log(L2 entries) -> 10 bits.
P1 = P2 = 10, d = 12.  
Maximum virtual space size = #entries in L1\*#L2\*pageSize = 4GB
Maximum physical size = 2^20\*4K = 4GB. The 20 is given, as the physical page number is 20 bits. Otherwise, with table entry size being 4B = 32bits, Maximum physical size is 2^32\*4K.  

Memory page with 4KB holds 1KB entries, **2B** each. P1 >= log(L1 entries) -> 1K entries = 10 bits. P2 = log(L2 entries) -> 10 bits.
P1 = 9, P2 = 11, d = 12.  
Maximum vitrual space size = #entries in L1\*#L2\*pageSize   = 4GB
Maximum physical size = 2^16\*4K = 256MB.  

As an extension of this, it is trivival to seee that a 3, or 4, or n level page table could be designed to fit larger needs.  
In this case, P3 = log(entries in L3 table)

### Page Sharing
A copy of shared code (read only) can be shared among processors. Shared code has to be in the same location of the logical address space for all the processes. This is sometimes known as lazy copying. The code and data though, would be independently stored and saved, to ensure there is not any interference.  
One can imagine implementing Fork() with this in mind to optimize performance. Lazy copy initially, and copy on write when needed.  

### Demand Paging
Keep only active pages in memory, and put other onto disk and invalidate their bits.  

### Zero fill
Fill a page with zeros with one bit, make that page seem empty rather than overwriting the whole space with 0s, which takes some time.

## Alternatives
### Hashed Page Tables
Size of page table grows with the amount of cirtual memory allocated. we can use a hash table to limit the cost of a search. One hash table per process.

### Inverted page table
Have a single hash table for all processes, with a single entry for each real page of memory. Each entry has the virtual address of the page, with metadata concerning the page.  
This decreases the amount of memory needed to store each page table, but increased the time needed tosearch the table when a page reference occurs.

## Demand Paging
Modern programs require a lot of memory, but generally don't use all of their memory all of the time. There is generally a 90/10 rule, where programs spend 90% of their time using 10% of their code. Therefore we can leverage main memory as a cache for disk.

### 'Infinite Memory'
Virtual memory allows for much larger numbers of running processes than otherwise, allowing more concurrency. Basically fit the key portions of their code which you need in memory, and the rest on disk, but with VM allow for them to look like they are in memory. You'd like to minimize the number of times which you bring pages into memory.

### Metadata
With each page table entry, keep track of a valid/invalid bit. Initially, set to invalid. When not in memory and grabbed, it is then valid after a page fault.  
If the memory is written to but not written back, it is 'dirty'.

### Handling a page fault
1. Reference the page table and see if the memory location exists
2. If it does not, page fault
3. Locate page
4. Bring in missing page
5. If page table is full and need to replace item, follow policy and write bcal
6. Then restart introduction

Note, we move entire blocks in and out of memory.

### Demand paging performance
We define the effective access time (EAT) (take `p`=page fault rate): `=(1-p)*memory access cost + p*pageFaultServiceCost`. `pageFaultServiceCost=pageFaultOverhead + swapPageOut + swapPageIn + restartOverhead`.  

#### Example
Assume MAT = 200ns, avg pageFaultServiceCost = 8ms. `EAT = (1-p)*200 + p(8ms) = 200 + p*7,999,800.`
If p = .1, `EAT=8.2 microseconds`! 40x slowdown.  
What if we want a 10% max slowdown? `EAT < 200ns*1.1 -> p<2.5*10^-6`

### Why do we miss?
Other than being tired and typing incorrectly, there are 3 types of misses
 - Compulsory misses
   - Pages that have never been in memory before
   - These can be reduced using pre-fetching
 - Capacity misses
   - Not enough memory, need to increase DRAM
   - Can also dynamically allocate memory depending on process space allocation
 - Policy Misses
   - Happens when pages were in memory but kicked out prematurely due to our replacement policy

## Replacement Policy
When full, need to make space for new request.
We want an algo which will result in the minimum number of page faults. The goal is to keep important pages in memory.

### Basics
1. Find location of page on disk
2. Find a free frame.
   1. If there is a free frame, use it
   2. Otherwise, use a page replacement policy to select a victim
3.  Swap out the victim page. If the page is dirty, write it back to disk
4.  Bring in the desired page and update the page and fram tables

### Page replacement policies
 - First in First Out (FIFO)
   - Throw our the oldest frame.
   - This is bad, since it is likely that we will throw out heavily used frames
 - Optimum (OPT)
   - Best case assuming that our programs could predict the future. This, like the Loch Ness monster, probably exists but we just haven't found it yet. Similar yet unrelated to the proof that `P=NP`.
 - Random
   - Pick a random page. Simple hardware but no gauruntees
 - Least Recently Used (LRU)
   - Replace the page that hsa not been used for the longest time

#### Least Recently Used
Replace the pgae that has been used the least recently, not a bad min approximatino usually. Can be implemented with some sort of list. This requires keeping track of locations in list which in practice can be expensive. We can approximate LRUs though!

#### BeLady's Anomaly
We'd like the following property - more memory we have the miss rate will go down. Sounds obvious, but with FIFO for example, this may not hold.

### LRU Approximation
We use a *clock algorithm* to approimate the LRU. We replace an old page, which may not be the oldest. This is implemented in conjunction with a hardware 'use' bit. Each time that memory page is referenced, we set that bit. If the bit is not set, then it probably has not been used in a while.
On each page faut, we advance the clock hand and check the use bit:
 - If the use bit is 1, then clear the bit and leave it alone. This is known as a second chance
 - If 0, then it is a candidate for replacement.
If all the use bits are set, then we loop around.  
The clock hand only moves forwards on a pgae fault. We want to have the 'clock hand' moving slowly.

#### Nth chance replcaement
We can instead of using a single use bit, set some sort of counter to keep track of the number of chances we give any page. We can theoretically get better performance by allowing for replcaement on the nth time that the clock hand cycles through an item.

## Swap-Space
In short: We allow virtual memory to act as an extension of main memory
Spaace space can be carved out of the normal file system or a seperate disk partition. We allocate the swap space when the process starts, and hold the text and data segment. The kernel uses swap maps to track usage.  

### Swap allocation + Replacement
We can either allocate space on innitial process creation or on use. In addition, we can either use some sort of global replacement policy or some sort of local policy for management.

#### Memory Page Allocation
 - Initially we can have 0. Using demand paging, we can get more from disk as needed.
 - Equal allocation: Each process can have an equal number of frames
 - Proportial allocation: allocate proportionally to the size of the process.
 - Allocate based on the process priority.

#### Thrashing
If a process does not have enough pages, then the page fault rate is very high. The OS is forces to spend its time swapping pages in and out.

#### Working set
A working set is the set of memory locations that a process or program has references in the recent past. Represents locality. As a program executes, it transitions through sets of working sets.  
We have low page faults if working-sets of all active processes fit into memory. Demand paging works by migrating processes from one locality to another. Localities may overlap, and we need sufficient memory so that the working sets fit into memory.Thrashing occurrs when the size of all the working sets exceeds main memory size.

## Page-Size and performance
Having a bigger page size increases the TLB hit rate, and can increase internal fragmentation. The total page table size can decrease though.

## Memory mapped files
We can map a disk block to a page in memory. This simplifies file access through direct memory access. This can be managed with demand paging, and several processes may map to the same file as shared data.s

## More caching
 - Temporal locality: keep recently accessed items near the processor
 - Spatial lcoality: Data access is generally contiguous

### ZIPF Distribution
For the kth item, the popularity is `1/k^c`
Generally, we should cache popular items to increase our cache hit ratio.

## File Systems
Lets define a file as a contigous logical address space in a persistent storage. Files can be structured:
 - With no structure perse: Sequence of words or bytes with meaning to the file creator, but may not be universally understood
 - Simple record structure: Lines, fixed length or variable length, maybe consider a csv
 - Complex structure: Formatted JPEG or some sort of more complex working model.

#### File attributes
 - Name: only information kept in human readable form
 - Identifer: OS ID
 - Type
 - Location
 - Size
 - Protection: just `chmod -R 777 .` and don't worry about protections!
 - Time, Data, and User identification

#### File ops
 - Create
 - Open
 - Close
 - Write
 - Read
 - Reposition / Seek
 - Delete
 - Truncate

#### Access Options
 - Sequential: loop through the bytes in a file and read in-order
 - Direct: Seek to desired position and read.

#### File System Abstraction
 - Directory
 - Path: UNique string that identifies that file/dir
 - Links
   - Hard - name to metadata location
   - Soft - name to alternate name
 - Mount: From name in one file system to root of another

### UNIX file system API
 - Create
 - Link
 - Unlink
 - Createdir
 - Rmdir
 - Open
 - Close
 - Read
 - Write
 - Seek
 - fync: force file modifications to disk if they're in memory

### Memory protections 101
 - Can either have permissions to read, write, or/and execute.
 - Can give permissions to a user, group, or all.

CHMOD changed permissions. CHGRP chages the group associated with an item.

## File-System Structure
FSs have a special root-block which contains the root directory. We have per-file control blocks (*FCB*) which contains details about the file. Knows as i-nodes in unix.

#### Layered file systems
Virtual File Systems provide an OO way of implementing file systems. They allow for the same API to be used to interface with different types of systems.

### Directory Implementation
 - Linear list of files
   - Easy to program, but will be slow with a lot of files
 - Hash-tables
   - Implement with a linear list with a hash data strucutre. Decreases search time, but we'll get collissions
   - Search trees

### Unix File system
File-systems have I-Nodes and Data Blocks. The max number of I-Nodes is fixed at file-system creation. The typical number is 1% of total file system size. I-Node #2 is for root, and #1 is for tracking bad blocks.  
B-Trees are used in large directories to keep track of filenames.

#### Example
How many disk accesses to read the first data block of "/home/tom/foo.txt"?
1. Read in file header for root
2. Read in first data block for root
3. Read in file header for home
4. Read in first data block for home, search for tom
5. Read in file header for tom
6. Read in first data for tom, search for foo.txt
7. Read in file header for foo.txt
8. Fetch first data block for foo.txt

#### Open Files
When we open a file, we need to manage a file-pointer, file-open-count, keep the disk location of the file, and permissions.

### Disk Space Allocation
 - Contiguous allocation: Each file ocupies a set of contiguous blocks on disk
   - Simple and allows for fast random access
   - Not easy to grow files and leads to external fragmentation
 - Linked allocation
   - Each file is a linked list of blocks
   - Linked list index structure, simple and easy to implement. File tables become a linear map of all blocks on disk
   - Random access is slow and small file access is slow. Easy to find a free block and append. This also leads to freagmentation as file blocks will likely be scattered.
 - Direct Data Pointers
   - Put all the direct data pointers together in the same index block
   - This supports random access well, and allows better mitigation of external fragmentation
   - There is a space overhead to manage an index table.

### Levelled Indexed Allocation
Depending on how tree structured we allow our index tables to be, we can get drastically different max-file sizes.

#### 1-L
Assume 4kb blocks. Each index hold 1024 entries. `1024 * block-size=4MB`

#### 2-L, single indirection
Assume 4kb blocks. 1k entries * 1k entries * 4kb data = 4GB.

### Hybrid scheme
 - First 10-12 are direct - 40-48KB
 - 1 single indirect - 4MB
 - 1 double indirect - 4GB
 - 1 triple indirect - 4TB

Max size = sum.
This gives some good leeway to small files and supports large files well.

## Job-Scheduling
Job-schdeuling helps maximize CPU utilization obtained with multi-programming.
Select from ready processes and allocate CPU to one of them. Active processes transition from ready to running to other waiting queues. Scheduling is the process of decising who gets access to resources.
We find that CPU bursts dominate usage. We can focus on optimizing burts such that I/O is when we handle complexity.
- preemptive scheduling - change process state dynamically
- non-preemprive - a process runs until it is blocked or terminates.

### Scheduling Algorithm
 - Optimize max CPU utilization
 - Max throughput
 - Minimum turnaround time
 - Minimize waiting time
 - Minimize response time

#### FIFO scheduling
Processes arrive in order and process them as they arrive. Let them finish before proceeding. FIFO performance wholly depends on the order in which things arrive.

#### Shortest Job First
Give preference to short jobs and allow them to run first. This reduces waiting time but starves longer runnig jobs. Hard to implement as well since we do not have size information. Can predict using exponential averaging to predict future time. `\tua_{n+1} = a*t_n+(1-a)*\tua_n`. (*note: I know that LaTeX is not supported in github md, use your imagination :)*)

#### Priority Scheduling
Each process has some priority, and run in that order. Still leads to starvation.

#### Round Robin
Each process gets a *quantam* of CPU time, 10-100 ms. The process is then preempted and added to end of queue. If this quantam is too large, then FIFO, and too small then the context switching overhead is too large. While there is some overhead due to context switching leading to longer turnaround time, the response is significantly better.

#### Context Switching Overhead
Thread switching is only slightly fastr than process ~100ns, swithcing cores is around 2x more expensive. Context switch time depends on the size of the working set, and can cinrease by multiple orders of magnitude depending on the size of the working set.

#### Multi-Level Feedback Queue
Maintain multiple queues. Each gets a percentage of system resources. Processes can move between these queues, such that high priority jobs that take a while to complete will slowly lose their priority. Different quesues can also have different scheduling algos and time slices.

#### Thread Scheduling
If you declare kernel threads, time will be shared with other processes and such, otherwise, will be shared in the same time allocation as your process.

#### Affinity
The allocation of a process to the same core for cache reuse. Very important.

## Mass-Storage Systems
 - Positioning Time: Time to move disk arm to cylinder (seek time) + time for head to rotate (rotational latency)
 - Effective Bandwidth: Average data transfer rate during a transfer
 - Total disk access time = seek + rotation + transfer time
   - Seek = time to move arm over track
   - Rotation time = time to wait for disk to rotate under head - average is cost for half of the track
   - Transer time = on and off of disk

#### Disk Random Read
Really slow. For 1 sector `=#reads*(seek + 30/RPM + sectorSize/transferRate)/1000` ~7.3seconds. 1GB takes 8.37 hrs.

#### Disk Sequential Read
`seek + 30/RPM + #reads*(sectorSize/transferRate)` ~16.7ms. 1GB takes 66.8s. Fine performance.

### Disk scheduling
If we schedule I/O requests we can achieve faster performance

#### First Come First Serve
We end up moving the disk head a lot. This is really bad.

#### Shortest Seek Time First
Select request with shortest seek time. This reposiitons actively which is pretty smart, and given batched requests you can not starve.

#### Elevator Scheduling
Move disk arm in one direction until all requests are satififed, then reverse direction. Quickly can realuze this is silly, as we are on a disk (circle) and not an elevator (line).

#### Circular Scan
Scans in a circle, pretty darn fair.

### SSDs
Generally better in every way. faster, higher throughput, no moving parts. Downside is cost and their life-cycle is generally shorter.  
Writing data is somewhat complex and requires that we only write to empty pages. Easing can take 1.5ms. Controller maintains pool of empty blocks. Overwriting is simply writing to an empty space and recording old data as obselete. From time to time, we garbage collecting obselete data by copying valid to new blocks. This requires higher voltage and in the long-term can damage memory cells.  
`Latency = Queuing Time + Controller Time _ Transfer Time`

### Hybrid Drives
Use a small SSD as a budfferm then flush later. Much less-expensive and provides faster performnce for general use.

## Reliable Storage
Disks fail. *Availability* is the probability that the system can accept and process requests. *Durability* is the ability of a system to recover despite faults. *Reliability* is the ability of a system or component to perform its required functions for a specified period of time.  
 - Mean time before failure
   - MTBF - inverse of annual failure rate
 - Mean time to repair
   - MTTR - measures the time to repair a failed component, hours to days
 - Unavailibility - MTTR/MTBF

### How to increase durability
Disk blocks contain ECC codes to deal with small defects.  
We want to make sure writes survive in short-term, sometimes uses battery-backed RAM. Longr-term, we replicate data and ensure that data is kept independent of failure.

### RAID
Redundant Array of Inexpensive Disks.  
Multiple disks work together. We can improve performance with striping - use a group of disks as a single storage unit. We keep a RAID controller to manage disks.

#### RAID 0
 - Non-redundant disk array
 - Files are striped across disks
 - High read throughput, best write throughput
 - Any disk failure is data loss

#### RAID 1
 - Mirrored Disks, data is written twice.
 - On failure, rebuild. On read, choose sastest.
 - This is expensive and has a high space overhead.

#### RAID 1+0
 - Pair mirrors first
 - Stripe on a set of paired mirrors

#### RAID 2
 - Memory style ECC
 - For 4 data blocks 3 to manage error codes

#### RAID 3
 - Bit-interleaved parity.
 - On the bit level keep track of parity in another disk

#### RAID 4
 - Block-interleaved parity

#### RAID 5
 - Block-interleaved distributed parity
 - To write we need to real old data and parity, then write back to both
 - A single disk can fail

#### RAID 6
 - Uses two parity stripes on each disk
 - Two disks can fail before any data is lost

#### Example
If block 3 is lost, then `block3 = block1 XOR block2 XOR parity`

### Copy On Write
If a file is represented as a tree of blocks, only need to update edges on a write. Minimizes possible corruption.

### Transactional File Systems
Keep a log and applies transactions. Actions are not completed unless they are 'committed'.  
Thus, the file-system may not be updated immediately, with data preserved in the log.
1. Get transaction ID
2. Do sequence, if failure, roll-back
   1. Find free data blocks
   2. Find free inode
   3. Find directory insertion point
   4. Write map
   5. Write inode
   6. Write dir entry
3. Commit

Helps ensure the transition between consistent states

#### ACID
 - Atomic
 - Consistent
 - Isolated
 - Durable

Logs allow for safe modification of disk.

#### Crash During Logging
1. Scan the log
2. If a transactin starts with no commit
3. Discard log entried
4. Disk remains unchanged