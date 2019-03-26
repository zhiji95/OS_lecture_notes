# Scheduling

## Introduction:

1. Dispatcher: 

   1. low level mechanism
   2. responsibility context switch (The function do all mechanical things)

2. Scheduler:

   1. CFS: Complete Fair Scheduler (3.x)
   2. high level mechanism
   3. responsibility: deciding which process to run

3. Allocator:

   1. Parallel 
   2. distributed system

4. when to schedule: (***What triggers the switch***)

   1. from running to waiting state [nonpreemptive]
   2. from running to ready state (interrupt, simple not running now on CPU, The kernel preempted it)  [Definitely Preemption]
   3. Switches from wating to ready [Sort of Preemption]
   4. Terminate  [nonpreemptive]

   

   

## Algorithm

1. Overview: 
   1. workload: what kind work is this process doing
      1. CPU bound processes: running on CPU, computation
      2. I/O bound: most of time waiting or doing I/O (Editor)
   2. Environment:
      1. Batch: scientific simulation. 
      2. Specialized: run in a series of deadlines. (Real time guarantees)
      3. General : not guarantee the max time of program.

2. Performance matrics:

   1. Min waiting time: 
   2. Max CPU utilization: keep CPU busy. (you can not wait for CPU for blocking system call)
   3. Max throughput: complete as many processes as possible per unit time.
   4. Min response time: respond immediately. (switch to reduce utilization but reduce response time)
   5. Fairness: each process or user, same percentage of CPU 

3. FCFS

   1. Advantages: simple+fair
   2. disadvantages: 
      1. waiting time depend on arrival order
      2. Convoy effect: short process stuck waiting for long process
      3. head of the line blocking

4. Shortest Job first

   1. Advantages: minimized averaged waiting time
   2. Disadvantages: 
      1. Not practical
      2. Still Convoy effect
      3. May starve long job

5. Shortest Remaining time First

   1. Average waiting time is provably optimal

6. **Round Robin**

   1. Time Slice: 
   2. Queue 
   3. advantages:
      1. low response time
      2. fair
      3. low average waiting time when job length vary widely(Not strong)
   4. Disadvantages:
      1. poor average time when jobs have similar lengths
      2. Performance depends on length of time slices
         1. High -> degenerate to FCFS
         2. Low -> toot many context switches costly.

7. Multi-level Feedback Queue

   1. Priority:

      1. round robin among processess in same priority.
      2. Can be preemptive or nonpreemptive.
      3. Can be statistically assigned
      4. Can be dynamicallt changed by OS.

      ​	aging: increase the priority of processes waiting in the ready queue for long time

      ```c
      for(pp = proc; pp < proc+NPROC; pp++) { 
          if (pp->prio != MAX)
          	pp->prio++;
          if (pp->prio > curproc->prio)
          	reschedule();
      }
      ```

      5. Priority inversion: high priority process depends on low one. (Lock)
      6. Priority inheritance: inherit the priority of waiting process
         1. Inherient 

   2. MLFQ: (windows)

      1. processes move between queue
         1. Queues have different priority levels
         2. Priority of process changes based on observerd behavior.
      2. Parameters:
         1. Number of queues
         2. scheduling algorithms for each queue
         3. when to upgrade a process
         4. when to demote a process.
         5. which queue a process will start in.

8. CFS:

   1. RB tree: sorted by score, each process runs the time it deserves
