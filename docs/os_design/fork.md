---
title: 'Fork'
original_url: 'http://eXpOSNitc.github.io/os_design-files/fork.html'
---







Fork


































 



























  
  
  




Fork System Call
----------------


  

  

Arguments: None


Return Values:




|  |  |
| --- | --- |
| PID (Integer)  | Success, the return value to the **parent** is the process descriptor(PID) of the child process. |
| 0 | Success, the return value to the **child**. |
| -1 | Failure, Number of processes has reached the maximum limit. Returns to the parent |


*Description*: Replicates the process invoking the system call. The heap, code and library [regions](../os_spec-files/processmodel.html) of the parent are shared by the child. A new stack is allocated to the child and the parent's stack is copied into the child's stack.


When a process executes the Fork system call, the child process shares with the parent all the file and semaphore descriptors previously acquired by the parent. Semaphore/file descriptors acquired subsequent to the fork operation by either the child or the parent will be exclusive to the respective process and will not be shared.


Data Structures modified are [Process Table](process_table.html), [System Status Table](mem_ds.html#ss_table), [Open File Table](mem_ds.html#file_table), [Semaphore Table](mem_ds.html#sem_table), [Memory Free List](mem_ds.html#mem_free_list), [Disk Free List](disk_ds.html#disk_free_list) (in case of swapped pages), [Resource Table](process_table.html#per_process_table) and the [Disk Map Table](process_table.html#disk_map_table).


The mode flag in the [Process Table](process_table.html) has to be set to Kernel mode when the process enters the system call and reset before exiting from the system call.


  


![](../img/roadmap/fork.png)
  

Control flow diagram for *Fork* system call

  
  

#### Algorithm:



```


Set the MODE\_FLAG in the [process table](process_table.html) entry to 8, 
indicating that the process is in the fork system call.

//Switch to Kernel Stack - See [Kernel Stack Management during System Calls](stack_smcall.html). 
Save the value of SP to the USER SP field in the [Process Table](process_table.html) entry of the process.
Set the value of SP to the beginning of User Area Page.

//Allocate memory and set the Process Table
Find a free Process Table entry by invoking the **get\_pcb\_entry()** function in [Process Manager](../os_modules/Module_1.html) module. 
If no free entry is found, return -1.

/* The parent and the child must share the heap.  Hence, if 
  heap pages were not allocated for the parent so far, allocate it now. */
If heap pages are not allocated for the parent process,
then allocate 2 heap pages for the parent using the **get\_free\_page()** function in Memory Manager module
and set the page table entries for the heap pages of the parent.

Invoke the **get\_free\_page()** function in [Memory Manager](../os_modules/Module_2.html) module to obtain 3 
memory pages: 2 for user stack and 1 for User Area Page of the child process.

Copy the parent's [Process Table Entry](process_table.html) except TICK, PID, PPID, USER AREA PAGE NUMBER, 
KERNEL STACK POINTER, INPUT BUFFER, PTBR and PTLR to the child. 

Set the PPID field of child process to the current PID. Also set User Area Page Number 
to the new UA Page, MODE, TICK and Kernel Stack Pointer to 0. 

/* Kernel Context of the child process is empty */

/* PID, PTBR, PTLR fields of the child's process table is initilized by the get\_pcb\_entry function.*/ 

Copy the [per-process resource table](process_table.html#per_process_table) and [per-process disk map table](process_table.html#disk_map_table).
For every open file of the parent, increment the Open Instance Count in the [Open File Table](mem_ds.html#file_table).
For every semaphore acquired by the parent, increment Process Count in the [Semaphore Table](mem_ds.html#sem_table).
/* The child shares open files and acquired semaphores with the parent */

//Set Page Tables
Copy the page table entries (code, heap and library) from the parent's page 
table to the child  /* Shared between parent and child */
/* Code and Library Pages must be shared read only */
For each page shared, increment its value in the Memory Free List
Set the page table entries for stack to the pages acquired earlier.

Copy word by word the contents of the parent user stack to that of the child.

Store the current BP value to the begining of the kernel stack.
/* According to the ExpL calling convention, the BP register is not saved in the user stack. 
This value is saved here so that the context switch module can restore it when the child is run. */
       
Set the return value to 0 for the child process
The PID of the child process is set as the return value for the parent process

Set state of child process to (CREATED, \_ ).

Set the MODE\_FLAG in the [process table](process_table.html) entry of the parent process to 0.

Restore SP to User SP and return to the parent process.
	
**Note:**  At each point of return from the system call, remember to reset the MODE FLAG and switch back to the user stack.

```










































