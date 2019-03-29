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
   2. from running to ready state (interrupt, simple not running now on CPU, The kernel preempted it) [Definitely Preemption]
   3. Switches from wating to ready [Sort of Preemption]
   4. Terminate [nonpreemptive]

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

7. **Multi-level Feedback Queue**

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

      1. Priority inversion: high priority process depends on low one. (Lock)
      2. Priority inheritance: inherit the priority of waiting process
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
      3. Rules: (Fair too simple, a good baseline)
         1. P(A) > P(B): A runs
         2. P(A) == P(B): A, B runs in round robin using time slice of the queue
         3. starts at highest priority queue
         4. once a job used up its time, its priority reduced
         5. After some time period S, move all jobs in the system to the top most queue.

## Multiprocessor scheduling

each core simulates as two CPUs.

1. Symmetric multiprocessing: (Not the only picture for multiple CPUs)

   1. Architecture

      ![image-20190328121939478](/Users/zhiji/Library/Application Support/typora-user-images/image-20190328121939478.png)

   2. Mutiple CPUs, each has a cache and shares shared memory, all CPUs are identical

   3. Same access time to main memory

   4. Private Cache

      1. Cache memory: speed up the memory acces
      2. Cache hit ratio

   5.  NSMP (All CPUs uses piece of memory)

2. Global queue of processes (Linux 2.4)

   1. One queue shared across CPUs
   2. Advantages:
      1. Good CPU utilization
      2. Fair to all Processes
   3. Disadvantages:
      1. Not Scalable
      2. Poor cache locality

3. Per-CPU queue of process 

   1. Static partition of process to CPUs (each CPU have one queue), push to the CPU with least number of processes
   2. Advantages:
      1. Easy to implement
      2. Scalable
      3. Better cache locality
   3. Disadvantages:
      1. Load-imbalance

4. Hybrid approach: (Linux 2.6)

   1. one global queue+per-CPU queue
   2. balance jobs
   3. Processor Affinity

5. Gang scheduling:

   1. cosheduling: run a set of processes simutaneously 
   2. global context switch

6. Real-time scheduling:

   1. Hard Real-time system:  have hard deadline
   2. Soft Real-time computing: critical process receive higher priority. This will not be preempted.
   3. Linux support soft realtime

## Linux Scheduling

1. Overview:

   1. Multilevel Queue scheduler
   2. Two classes of processes

2. Priorities:

   1. Soft real-time scheduling policies

      § either SCHED_FIFO (FCFS) or SCHED_RR (round robin) for each process

      § Priority over normal tasks 

      § 100 static priority levels (1..99) 

   2. Normal scheduling policies

      § SCHED_NORMAL: standard • SCHED_OTHER in POSIX (They are same number)

      § SCHED_BATCH: CPU bound 

      § SCHED_IDLE: lower priority 

      § Static priority is 0 

      1. 40 dynamic priority within 
      2. `nice` values 

   3. `scheduling_setscheduler()` `nice()` `sched(7)` man page for details

3. Implementations 

   1. 2.4: global queue O(n)
   2. 2.5: O(1) scheduler, per-CPU run queue
   3. 2.6: completely fair scheduler (CFS)

4. CFS:

   1. Ideal fair scheduling:
      1. Each runs uniformly 1/n rate.

1. RB tree: sorted by score, each process runs the time it deserves
