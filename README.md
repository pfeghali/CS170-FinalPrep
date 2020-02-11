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

We call a binary semaphore a mutex lock.  
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
 - `Maximum soze = #entries*page size`

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
Maximum vitrual space size = #entries in L1\*#L2\*pageSize = 4GB
Maximum physical size = 2^20\*4K = 4GB.  

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
