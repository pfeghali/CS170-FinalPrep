# CS170-FinalPrep
Please note: this document is *heavily* reliant on class lecture slides. This is intended to be a cleaned version of the slides.
### What we expect an OS to do
OS needs to 
 - Manage processes
 - Memory management
 - Storage management
 - Protection and Security

Needs to have high performancee, reliability, security, and portability
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