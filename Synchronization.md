# Synchronization

## Critical section

1. Definition: a segment of code that accesses a shared variable (or resource)

​		No more than one thread in critical section at a time.

2. Requirement:
   1. Safety (Multual exclusion): no more than 1 thread in critical section at a time.
   2. Liveness (aka progress):
      1. Multi thread simultaneously request: must allow one
      2. should not depend on threads out side of critical section
   3. Bounded waiting (aka starvation-free):
   4. Makes no assumptions about the speed and number of CPU.
   5. Efficient: don't consume too much resources 
   6. Fair: don't make one thread wait longer than others.
   7. Simple

## Locks

1. Disable interrupt:

2. software locks:

   1. Peterson algorithm: 

   2. Assumptions:

      1. loads and stores are atomic
      2. they execute in order

   3. Versions:

      1. 1st attempt
      2. 2nd attempt: 
      3. 3rd attempt
      4. Peterson algorithm: 

   4. Memory barriers:

      1. Ensures that all memory operations up to the barrier

      2. X86 provides several memory fence

         1. mfence
         2. lfence
         3. sfence

         ```c
         lock() {
             flag[self] = 1; // I need lock
             turn = 1 – self;
             sfence; // Store barrier
             while (flag[1-self] == 1 && turn == 1 – self); 
         }
         ```

         

   5. Lamport's Bakery Algorithm

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
   int test_and_set (int *lock) { 
       int old = *lock;
       *lock = 1;
       return old;
   }
   // Gcc allows write asmbley inside c code
   long test_and_set(volatile long* lock)
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
   1. Idea: add thread to queue when lock unavailable
   2. Problem1: Lost wakeup
   3. problem2: Wrong thread gets lock
