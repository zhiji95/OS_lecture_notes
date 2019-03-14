# Synchronization

## Critical section

1. Definition: a segment of code that accesses a shared variable (or resource)

   No more than one thread in critical section at a time.

2. Requirement:

   1. Safety (Multual exclusion): no more than 1 thread in critical section at a time.
   2. Liveness (aka progress):
      1. Multi thread simultaneously request: must allow one
      2. should not depend on threads out side of critical section
   3. Bounded waiting (aka starvation-free): must allow waiting thread to proceed
   4. Makes no assumptions about the speed and number of CPU.

3. Properties

   1. Efficient: don't consume too much resources 
   2. Fair: don't make one thread wait longer than others.
   3. Simple

## Locks

1. Disable interrupt:

   1. can cheat uniprocessor
   2. both operations are privileged, can't let user program use
   3. doesn't work on multiprocessors
   4. can't use for long critical sections

2. software locks:

   1. Peterson algorithm: software lock implememtation

   2. Assumptions:

      1. loads and stores are atomic
      2. they execute in order
      3. does not require special hardware instructions

   3. Versions:

      1. 1st attempt

         ```c
         int flag = 0;
         void lock() {
             while (flag == 1)
                 ;
             flag = 1;
         }
         unlock() {
             flag = 0;
         }
         ```

         Both threads can be in critical section (*not safe*)

      2. 2nd attempt: 

         ```c
         int flag[2] = {0, 0};
         void lock() {
             flag[self] = 1 //I need lock
             while (flag == 1)
                 ;
         }
         unlock() {
             flag[self] = 0;
         }
         ```

         can dead lock: another thread lock() between `flag[self] = 1 //I need lock` and `while (flag == 1)`

      3. 3rd attempt

         ```c
         int turn = 0;
         void lock() {
             while (turn == 1 - self)
                 ;
         }
         unlock() {
             turn = 1-self;
         }
         ```

         depends on threads outside critical section

         Can not handle repeated calls to lock by same thread.

      4. Peterson algorithm: 

         ```c
         // whose turn is it?
         int turn = 0;
         // 1: a thread wants to enter critical section, 0: it doesn’t 
         int flag[2] = {0, 0};
         lock()
         {
         flag[self] = 1; // I need lock turn = 1 – self;
         // wait for my turn
         while (flag[1-self] == 1 && turn == 1 – self)
         ; // spin wait while the
         // other thread has intent // AND it is the other
         // thread’s turn
         }
         unlock()
         {
         // not any more
         flag[self] = 0; 
         }
         ```

         

   4. Multiprocessor Challenges:

      1. Modern processors are out-of-order/speculative
         1. Reorder instructions
         2. try very hard to avoid inconsistency
         3. guarantees valod only within single execution stream
      2. Memory access guarantees one x86
      3. Breaks Peterson's algorithm

   5. Memory barriers:

      1. Ensures that all memory operations up to the barrier

      2. X86 provides several memory fence

         1. mfence: all prior memory access completed
         2. lfence: all prior loads completed 
         3. sfence: all prior stores flushed

         ```c
         lock() {
             flag[self] = 1; // I need lock
             turn = 1 – self;
             sfence; // Store barrier
             while (flag[1-self] == 1 && turn == 1 – self); 
         }
         ```

         

   6. Lamport's Bakery Algorithm

      1. support more than 2 processes

         ```c
         bool flag[1..NUM_THREADS] = {0}; // Want to enter int token[1..NUM_THREADS] = {0}; // My token lock(i) { // Lock by thread i
         flag [i] = 1;
         token[i] = 1 + max(token[0..NUM_THREADS-1]); 
         flag[i] = 0;
         for (j = 1; j <= NUM_THREADS; j++) {
             while (flag[j]); // Is j getting token?
             while ((token[j] && ((token[j], j) < (token[i], i))); // j has smaller token? 
         }
                
         unlock(integer i) {
         token[i] = 0; }
         ```

3. Hardware Instruction

   ```c
   // special atomic operation
   int test_and_set (int *lock) {        
       int old = *lock;
       *lock = 1;
       return old;
   }
   // Gcc allows write asmbley inside c code
   long test_and_set(volatile long* lock) // implememtation on x86
   {
       
   int old;
   asm("xchgl %0, %1"
       : "=r"(old), "+m"(*lock) // output
       : "0"(1) // input
       : "memory“ // can clobber anything in memory );
   return old; }
   ```

   Limitation of spin_locks

   1. Safe inside the kernel

   2. available in user: risking preempted 

      `pthread_spin_init`

4. Sleeping lock

   1. Idea: add thread to queue when lock unavailable, and wkae up one thread in the queue

   2. Problem1: Lost wakeup

      **Other thread unlock the lock  and called wake up before you add yourselves to wait queue**

   3. problem2: Wrong thread gets lock

   4. Implementation:

      ```c
      typedef struct __mutex_t {
      int flag; // 0: mutex is available, 1: mutex is not available int guard; // guard lock to avoid losing wakeups
      queue_t *q; // queue of waiting threads
      } mutex_t;
      
      void lock(mutex_t *m) {
          while (test_and_set(m->guard))
          ; //acquire guard lock by spinning
          if (m->flag == 0) {
          m->flag = 1; // acquire mutex 
              m->guard = 0;
          } else {
          enqueue(m->q, self); 
          prepare_to_yield(); // fixing the last race condition
          m->guard = 0; 
          yield();
      }} 
      
      void unlock(mutex_t *m) {
          while (test_and_set(m->guard)) ;
          if (queue_empty(m->q))
          // release mutex; no one wants mutex
          m->flag = 0; 
          else
          // direct transfer mutex to next thread 
              wakeup(dequeue(m->q));
          m->guard = 0;
      }
      ```

      

   

### Reader-writer Lock

High overhead

1. rwlock:

   ```c
   struct rwlock_t {
      int nreader; // init to 0
      lock_t lock; // init to unlocked
   }
   ```

   

2. Write_lock: 

   ```c
   write_lock(rwlock_t *l) 
   {
   lock(&l->lock); 
   }
   write_unlock(rwlock_t *l) 
   {
   unlock(&l->lock); 
   }
   
   ```

   

3. Read_lock:

```c
read_lock(&rwlock_t *l){
    lock(&l->guard);
    ++nreader;
    if (nreader == 1) //first reader
        lock(&l->lock);
    unlock(&lock->guard);
}
read_unlock(rwlock_t *l) {
    lock(&l->guard);
    --nreader;
    if(nreader == 0) // last reader
    unlock(&l->lock);
    unlock(&l->guard);
}
```

Might starve write

Solution:

```c
struct rwlock_t { 
    int nreader; 
    lock_t guard; // protecting the counter
    lock_t lock; 
    lock_t writer;
};
void write_lock(rwlock_t *l)
{
    lock(&l->writer);// This lock prevent subsquent readers comming in if there is a writer 
    lock(&l->lock);
    unlock(&l->writer);
}
void read_lock(rwlock_t *l) {
    lock(&l->writer);
    lock(&l->guard);
    ++ nreader;
    if(nreader == 1) // first reader
    lock(&l->lock); 
    unlock(&l->guard);
    unlock(&l->writer);
}
```

### Semaphore

#### Motivation

1. Problem with lock: ensures mutual exclusion, but no execution order
2. PC problem: need to enforce execution order.
   1. Producer: create resources
   2. Consumer:  use resources
   3. bounded buffer between them
   4. execution: producer waits of buffer full, consumer waits of buffer empty.

#### Definition

1. A synchroniza+on variable that contains an integer value
2. must initialized to some value
3. has two operations

#### Uses mutual exclusion with more than one resources

#### Uses execution order

semaphore has to start from value 0, 

Sem_wait(s) should be blocked in thread 1

![image-20190312121818319](/Users/zhiji/Library/Application Support/typora-user-images/image-20190312121818319.png)

#### PC problem using semaphore

Producer:

```c
sem_t full; // # of filled slots
sem_t empty; // # of empty slots
sem_t mutex; // mutual exclusion (This can be just mutex)

sem_init(&full, 0, 0); 
sem_init(&empty, 0, N); // buffer size is n
sem_init(&mutex, 0, 1);

void producer() {
    sem_wait(empty); 
    sem_wait(&mutex); 
    ... // fill a slot 
    sem_post(&mutex); 
    sem_post(full);
}
void consumer() {
    sem_wait(full); 
    sem_wait(&mutex); 
    ... // empty a slot 
    sem_post(&mutex); 
    sem_post(empty);
}
```

#### Monitors

1. Monitors: monitor producer have only one thread at the same time
2. Condition variable
   1. operations:
      1. wait(): suspend the calling thread and release the lock.
      2. Signal(): resumes one thread in waiting
      3. Broadcast(): resumes all thread waiting
   2. Monitor: 1 mutex + N condition variable 
3. PC with monitors:
   1. nedd 2 condition variables:

```java
class monitor ProducerConsumer { int nfull = 0;
cond has_empty, has_full;
void producer() {
if (nfull == N) // this if has to be a while
	wait (has_empty); ... // fill a slot
++ nfull;
signal (has_full);
}
void consumer() {
if (nfull == 0) // this if has to be a while
	wait (has_full); ... // empty a slot
-- nfull;
signal (has_empty);
} }
```

4. CV semantics:

   1. Hoarse semantics: suspends signal thread and immediately transfers control to the waken thread.

      for this semantic, the if is probably ok

   2. Mesa semantics: moves a signal waiting thread form the blocked state to a runnable state them the signaling thread continues until it exits the nmonitor.

   3. 2 is easy to implement, problem: race

   4. need to replace the if with while

