# Spin Lock

## Spin Locks

### Code

```c
lock () {
    if (flag == 0) {
        flag = 1
    }
}

unlock() {
    flag = 0;
}
```

Actual Spining lock: (Short i++)

literally doing following

```c
int test_and_set(int *lock) {
    int odd = *lock;
    *lock = 1;
    return odd;
}

lock () {
    while(test_and_test(&flag)) {
        
    }
}
```

### make sense from user

`pthread_spin_lock()` you know you will not be preempted.

### function:

`spin_lock()` `spin_unlock()`

1. Must not loose CPU while holding a spin lock
2. prevents kernel preemption by ++preempt_count
3. must NOT call any function that can potentially sleep
4. hardware interrupt is ok unless the interrupt handler may try to lock this spin lock
5. keep the critical section as small as possible.

```c
spin_lock_irqsave() spin_unlock_irqrestore()
```

1. **disable all interrupts** on local CPU, lock, unlock, restore interrupts to how it was before
2. need to use this version if the lock is something that an interrupt handler may try to acquire
3. no need to worry about interrupts on other CPUs – spin lock will work normally
4. again, no need to spin in uniproc – just ++preempt_count & disable irq

```c 
spin_lock_irq(), spin_unlock_irq()
```

1. disable & enable irq assuming it was disabled to begin with
2. should not be used in most cases

## Interrupt

1. Spin_lock does not prevent interrupt

2. Can not Reentrent. 

3. Process context vs. Interrupt context

   1. Process context: 

      1. normal things caused by processes themselves (call system call or raising exception). 

      2. ##### can sleep.

   2. Interrupt context: 

      1. still have the notion of current process

      2. but process has nothing to do with

      3. Quick

      4. ##### Can not sleep.

4. Interrupt handler

   1. Interrupt in interrupt
      - This interrupt is disabled while serving this interrupt.
      - Livelock: no work being done.
      - For network it is different, disable the interrupt and keeping looking at Network card. NAP
   2. only time-critical stuff in handlers
      1. eg: Network
         1. Network save into a buffer
         2. CPU grab from buffer and return.
   3. All handlers share one interrupt stack per processor (linux specific)

5. Multi-CPU:

   1. There is a line 
   2. PIC: programmable interrupt controller 
   3. Which CPU wakeup: programmable
      1. Simple OS: dedicate single CPU
      2. Complicated: grab one for that purpose

## kernel preemption

1. If `TIF_NEED_RESCHED` is set, preemption occurs by calling `schedule()` in the following cases:

   1. Returning to user space:
      - from a system call
      - from an interrupt handler
   2. Returning to kernel from an interrupt handler, only if `preempt_count` is zero (you are not holding a spin lock)
   3. `preempt_count` just became zero – right after spin_unlock(), for example
   4. Thread running in kernel mode calls schedule() itself – blocking syscall, for example

2. Linux is fully preempted, can be preempted, anytime no matter in user or kernel

3. Not a good idea sometime (slow) : eg. inside spinlock

4. A way to prevent kernel preemption

5. preempt_count`  (global variable in kernel address space)

6. *There are CPU instructions do ++ atomically* 

   
