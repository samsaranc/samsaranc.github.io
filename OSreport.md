# OS Project: Final Report

## Progress
_final levels reached_ 

- Module 1, FSS level achieved: 4
- Module 2, Shared Memory level achieved: 3
- Module 3, Synchronization level achieved: 4
- Module 4, Scheduling level achieved: 4
- Project level achieved: _Super Saiyan_

## Evaluation Documentation

### FSS (module 1)
 - __files added:__     `fss.c`, `fss_ipc.c`, `fss.h` 
 - __files edited:__    `string.c`, `init.c`(added `procstat` syscall for testing)
 - __contents:__
      1. `fss_ipc.c` has function calls (ex `fss_read`, `fss_open` etc), and `fss_action` which handles the ipc/shared memory/mutex stuff
      2. `fss.c` has the constantly running proc which is forked in init, ipc things, a massive switch statement which decides which function to call to do work, and a massive linked list which contains information about files, populated when a new file is opened. `fwrite` loops through the linked list, to find the filename and creates a new identical file in checkpoint; `funlink` does approximately the same thing. `fmkdir` adds a directory identical but in checkpoint. `frestore` loops through the linked list and deletes checkpoint versions and adds them to the actual directory. `fcheckpoint` is identical to `frestore` except does not add the files to the actual directory.
 - __tests:__           `fss_test.c`, `fss_test2.c`, `fss_test3.c`
      1. `fss_test3.c` checks _lvls 0-4_ by making several layers of directories, `fopen`ing files in several different ones of these directories, and unlinking/writing all of these. This was followed by calling `frestore`. After calling restore the tests reopen files and `funlink`/`fwrite` some more. `fcheckpoint` is then called. This shows an efficient testing of every level.

### Shared Memory (module 2)
 - __files added:__     `shm.h`, `shm.c` 
 - __files edited:__    `proc.c`, `proc.h`, `sysproc.c`, `syscall.c`, `syscall.h`, `usys.S`, `user.h`, `main.c`, `vm.c`
 - __contents:__
      1. `Shm.c` has all the basics for shared memory including `shm_create`, `shm_alloc`, `shm_remove`, `shm_fork`
 - __tests:__           `shm_test.c`, `shm_test2.c`, `shm_test3.c`
      1. `shm_test.c` checks _lvls 0-1_ by creating shared memory between 2 processes: 1 writes and the other one reads. Locks are used for Synchronization purposes. The program tests `shm_create`, which finds a location and puts a physical address at that location, while `shm_alloc` maps the mem into the process that requested that memory. Essentially this was the slit of `shm_get` because of time complexity: O(1) vs O(n).
      2. `shm_test2.c` checks _lvl 2_. `shm_fork` is responsible for remapping the physical memory into the child page table. It remaps using unmap pages. This was tested with `shm_test2.c`, which is very similar to `shm_test.c`, but this time the shared memory is declared and allocated in the main function. Then the 2 children write and read to the shared memory space.

      3. `shm_test3.c` checks _lvl 3_. Level 3 was more tricky given that a test was supposed to `panic` or exit randomly. In order to solve this, exit in `proc.c` unmapped and referenced a process that exit without calling `shm_remove`. Furthermore, in case a function panics, the panic executes code similar to `exit`. `shm_test3.c`, a copy of `shm_test2.c`, tests this by executing the same code but will cause a page fault. 
      
### Synch (module 3)
 - __files added:__     `synchro.c`, `synchro.h`, `futex.c`, `futex.h`,  
 - __files edited:__    `proc.c`, `console.c`, `proc.h`, `sysproc.c`, `syscall.c`, `syscall.h`, `usys.S`, `user.h`
 - __contents:__
      1. `synchro.c` has all normal mutex level functionality
      2. `futex.c` has all futex functionality
 - __tests:__           `synch_test.c`, `synch_test2.c`, `synch_test3.c`, `futex_test.c`, `futex_test2.c`, `futex_test3.c`, `big_test.c`
      1. `synch_test.c` checks _lvl 1_ by creating a mutex and forking, where the child creates another mutex, takes it, accesses the critical section, then releases the lock. The parent takes the lock the child created, accesses the cs, releases it, and deletes the mutex. This test shows that mutual exclusion holds because both processes access the cs one at a time without race conditions.
      2. `synch_test2.c` checks _lvl 2_ by creating a mutex and forking, where the child and parent both try to take the lock---the child takes the lock at first, acesses cs, `cv_signal`s to parent that it is done, and releases the lock. The parent, previously sleeping because it called `cv_wait` after taking the lock, receives the signal, accesses the cs, and then deletes the lock. This test shows that mutual exclusion holds with conditional variables because both processes access the cs one at a time without race conditions.
      3. `synch_test3.c` checks _lvl 3_ works by creating the scenario where one process is conditionally waiting on the other and that process faults and triggers a system `panic`. Before we reach the end of `panic`, the faulting process `cv_signal`s to the other and removes its locks 
      4. `futex_test1.c` checks _lvl 4_ by creating a futex and forking, where the child creates another futex, takes it, accesses the critical section, then releases the lock. The parent takes the lock the child created, accesses the cs, releases it, and deletes the futex. This test shows that mutual exclusion holds with futexes because both processes access the cs one at a time without race conditions.      
      
### Scheduling (module 4)
 - __files added:__     `pqueue_1.h` `ring.h` 
 - __files edited:__    `proc.c`, `proc.h`, `sysproc.c`, `syscall.c`, `syscall.h`, `usys.S`, `user.h`
 - __contents:__
      1. `proc.c` implements the functionality of the scheduler with an array of ring buffers where each ring buffer is a priority from 0 to 20. `set_prio` changes the priority based on the specification, a parent can change a priority of a descendant and it has to be inferior to itself, this also applies to a process trying to change its own priority. Furthermore, this is required to change the scheduling policy and the `RUNNABLE` state. The scheduler dequeues and the runnable state indicates that an enqueue should happen. It will dequeue in increasing order from 0 to 20 and if the state is `RUNNABLE`. 
 - __tests:__           `priority_test.c`
      1. `priority_test.c` checks _lvls 0-4_ because 2 cores are used and 3 processes are set to 3 different priorities: High, Medium and Low. Given our policy the High and Medium will run and the Low will not until one of them exists or yields/sleeps. Additionally, when we try to change a process that we're not an Ancestor of it will return -1;
      
### Overall integration tests
 - __tests:__           `big_test.c`, `fss_test3.c`, `buffer_test.c`
      1. `big_test.c` checks that _all levels_ of _modules 2-4_ work by creating in main two mutexes (m1, m2), a futex, and shared memory for 4 processes (2 primary ones, child1 and child2) to use that they then edit in order using conditional variables and futexes to maintain mutual exclusion. The main forks, where its child, child1, uses the m1 to lock, accesses the critical section (shm), sets it to "SHM_TEST," and releases m1. The parent then forks again, where child2 takes m1, accesses the critical section, then releases the lock.  We do something similar with futexes: child1 changes the value of the shm to "FUTEX_TEST and we use the futex to keep mutual exclusion. Child2 accesses it. In the last step, we use m1, m2 to ensure mutual exclusion while we change and print the priorities in order. This test shows that mutual exclusion holds because all processes access the (cs) shared memory one at a time without race conditions---the cs (shm) has the value "SHM_TEST" for the shared memory accesses with normal mutexes, as expected. Then, child1 uses the futex to lock the shm and changes its value to "FUTEX_TEST". Child2 then accesses it. Then parent sets the priority of process with pid 5 (child1) to 8. We then call `ps` and verify process 5 has the right priority, 8. 
      2. `fss_test3.c` checks that _everything_ works because the file system uses mutexes and shared memory, and then calls every function call, which behaves as expected. 
      3. `buffer_test.c` is designed to test the limits of the buffers for SHM, MUTEXES, and FUTEXES. The `MAX_NUM` of all of them are currently set to 16. Upon passing the 17th/16th for SHM, the various create functions return -1.

## Work Attribution

Which team members were involved in leveling up the project for each of the different aspects?  Put names that did more earlier.  

### Module 1: FSS

Member(s) responsible for this module: Marie

(Filled out with example names)

- Level 0: Sebastian, Marie
- Level 1: Marie
- Level 2: Marie
- Level 3: Marie
- Level 4: Marie

### Module 2: Shared Memory

Member(s) responsible for this module: Sebastian

- Level 0: Sebastian
- Level 1: Sebastian
- Level 2: Sebastian
- Level 3: Sebastian

### Module 3: Synchronization

Member(s) responsible for this module: Samsara 

- Level 0: Samsara
- Level 1: Samsara
- Level 2: Sebastian, Samsara
- Level 3: Samsara
- Level 4: Sebastian

### Module 4: Scheduling

Member(s) responsible for this module: Sebastian

- Level 0: Sebastian
- Level 1: Sebastian
- Level 2: Sebastian
- Level 3: Sebastian
- Level 4: Sebastian

### Project Levels

- Level 0: Sebastian, Samsara
- Level 1: Sebastian, Samsara
- Level 2: Sebastian, Samsara
- Level 3: Sebastian, Samsara, Marie
- Level 4: Sebastian, Samsara, Marie
- Level *Super Saiyan*: Sebastian, Samsara, Marie