---
title: 'Synchronization and Access control'
original_url: 'http://eXpOSNitc.github.io/os_spec-files/synchronization.html'
---






Synchronization and Access control































 











Toggle navigation


















  

  

  







Synchronization and Access control
----------------------------------


  









#### 
Process Synchronization





eXpOS provides the **Wait** and **Signal** system calls for process synchronization. When a process executes the wait system call specifying the process id of another process as argument, the OS puts the calling process to **sleep.** This means, the OS will schedule the process out and won't execute it until one of the following events occur:


1. The process specified as argument terminates by the **exit** system call.
2. The process specified as argument executes a **Signal** system call. The specification of Signal and Wait system calls are given [here](systemcallinterface.html#synsystemcalls).








#### 
Access Control





 A second major concurrency related requirement is that when multiple processes access the same data (in memory or files), it is often required to have some kind of locking mechanism to ensure that when one process is accessing the shared data, no other process is allowed to modify the same. This is to ensure data integrity and this issue is called the [critical section problem](http://en.wikipedia.org/wiki/Critical_section). 


eXpOS provides (binary) **semaphores** to allow application programs to handle the critical section problem. A (binary) semaphore is conceptually a binary-valued variable whose value can be set or reset only by the OS through designated system call. (Internally how it is implemented is an OS concern and is oblivious to the application programmer). 


A process can acquire a semaphore using the **Semget** system call, which returns a unique **semid** for the semaphore. The semaphore is initially not set. A process may open several files and semaphores and execute the Fork system call multiple times to create an several process that shares all the opened file handles, the heap memory region and all the semaphores acquired by the parent. 


Any of these processes sharing a semaphore identifier can set the semaphore (called locking) using the **SemLock** system call giving the semid as an argument. **SemUnLock** will reset (called release) the semaphore waking up all other processes which went to sleep trying to lock the semaphore. 


### Semantics of Locking operation for semaphores


1. If the semaphore/file is already locked, the process goes to sleep and wakes up only when the semaphore/file is free.
2. Otherwise, process locks the semaphore/file and continues execution.













[![](../img/spec_icon.jpg)](../os_spec.html)










































