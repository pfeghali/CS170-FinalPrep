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

