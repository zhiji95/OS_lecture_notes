##  a. Basic concepts: ##
			i. Is a sleeping lock
			ii. When unavailable: Places the task onto a wait queue and puts the task to sleep
			iii. When available: one of the task on the wait queue is awakened.
##		b. Conclusions:
			i. Well suited locks that are held for a long time
			ii. Not optimal lock for short periods because the overhead of sleeping, maintaining and waking back up
			iii. Must be obtained only in process context
			iv. Can sleep while holding a semaphore, no deadlock when another process acquires the same semaphore
			v. Can not hold a spin lock while you acquire a semaphore
##		c. Counting and binary:
			i. Allows an arbitrary number of simultaneous lock holders
			ii. Usage value:
			iii. Binary: when count == 1, mutex
			iv. Count semaphore: count > 1
			v. P
			vi. V
## Creating and initializting
  ```c
  /* Static */
  struct semaphore name; 
  sema_init(&name, count); 
  
  static DECLARE_MUTEX(name);
  /*Dynamic*/
  sema_init(sem, count); /* sem: pointer*/
  init_MUTEX(sem);
  ```
  
## Using


  
