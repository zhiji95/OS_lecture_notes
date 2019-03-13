# Signals 

## Signals and system calls

The following program, from APUE3 section 1.9, adds signal handling to the simple shell program we studied before:

**Signal Funtion** 

```c
#include <signal.h>
void (*signal(int signo, void (*func)(int)))(int);
	//Returns: previous disposition of signal (see following) if OK,SIG_ERR on error
```



```c
#include "apue.h"
#include <sys/wait.h>

static void	sig_int(int);		/* our signal-catching function */

int
main(void)
{
        char	buf[MAXLINE];	/* from apue.h */
        pid_t	pid;
        int		status;

        if (signal(SIGINT, sig_int) == SIG_ERR)
                err_sys("signal error");

        printf("%% ");	/* print prompt (printf requires %% to print %) */
        while (fgets(buf, MAXLINE, stdin) != NULL) {
                if (buf[strlen(buf) - 1] == '\n')
                        buf[strlen(buf) - 1] = 0; /* replace newline with null */

                if ((pid = fork()) < 0) {
                        err_sys("fork error");
                } else if (pid == 0) {		/* child */
                        execlp(buf, buf, (char *)0);
                        err_ret("couldn't execute: %s", buf);
                        exit(127);
                }

                /* parent */
                if ((pid = waitpid(pid, &status, 0)) < 0)
                        err_sys("waitpid error");
                printf("%% ");
        }
        exit(0);
}

void
sig_int(int signo)
{
        printf("interrupt\n%% ");
}
```

Three problems:

1. “Slow” system calls may get interrupted on signals
   - `errno` set to `EINTR`
2. Signals get lost
   - signal handling resets after each signal
3. Signal handler calls non-reentrant functions
   - `malloc()`, `free()`, standard I/O functions

Solutions:

- Don’t use `signal()`; use `sigaction()` instead
- From a signal handler, call only *async-signal-safe* functions (see `man 7 signal`)
- Async-signal-safe: non blocking

## Process groups, sessions, controlling terminal

After issuing the following commands:

```bash
proc1 | proc2 &
proc3 | proc4 | proc5
```

You have:

![Figure 9.7, APUE](http://www.cs.columbia.edu/~jae/4118/L06/fig9.7.jpg)Figure 9.7, APUE

## Sending signals

```c
#include <signal.h>

int kill(pid_t pid, int signo);

int raise(int signo);// kill(getpid(), signo)

        Both return: 0 if OK, –1 on error
```

- if pid < 0, the signal is sent to the process group with pgid == | pid |
- if pid == -1: The signal is sent to all processes on the system for which the sender has permission to send the signal.
- if pid == 0: the signal is sent to the process group with pgid == this pgid
- if pid > 0 : signal send to pid

## SIGALRM

### alarm() and pause() functions

```c
#include <unistd.h>

unsigned int alarm(unsigned int seconds);
        Returns: 0 or number of seconds until previously set alarm

int pause(void);
        Returns: –1 with errno set to EINTR
```

- `alarm(0)` cancels the previous alarm if one was set
- alarm() & pause() can be used to implement `sleep()`, but there are many subtleties

### Waking up slow system calls using alarm()

```c
#include "apue.h"

static void sig_alrm(int);

int
main(void)
{
    int     n;
    char    line[MAXLINE];

    if (signal(SIGALRM, sig_alrm) == SIG_ERR)
        err_sys("signal(SIGALRM) error");

    alarm(10);
    if ((n = read(STDIN_FILENO, line, MAXLINE)) < 0)
        err_sys("read error");
    alarm(0);

    write(STDOUT_FILENO, line, n);
    exit(0);
}

static void
sig_alrm(int signo)
{
    /* nothing to do, just return to interrupt the read */
}
```

Two problems:

1. A race condition: the alarm can be missed between `alarm(10)` and `read()`
2. This doesn’t work if slow system calls are automatically restarted

### Using setjmp & longjmp to solve the two problems (Optional material)

```
#include "apue.h"
#include <setjmp.h>

static void     sig_alrm(int);
static jmp_buf  env_alrm;

int
main(void)
{
    int     n;
    char    line[MAXLINE];

    if (signal(SIGALRM, sig_alrm) == SIG_ERR)
        err_sys("signal(SIGALRM) error");
    if (setjmp(env_alrm) != 0)
        err_quit("read timeout");

    alarm(10);
    if ((n = read(STDIN_FILENO, line, MAXLINE)) < 0)
        err_sys("read error");
    alarm(0);

    write(STDOUT_FILENO, line, n);
    exit(0);
}

static void
sig_alrm(int signo)
{
    longjmp(env_alrm, 1);
}
```

## Signal sets

```c
#include <signal.h>

int sigemptyset(sigset_t *set);

int sigfillset(sigset_t *set);

int sigaddset(sigset_t *set, int signo);

int sigdelset(sigset_t *set, int signo);

        All four return: 0 if OK, –1 on error

int sigismember(const sigset_t *set, int signo);

        Returns: 1 if true, 0 if false, –1 on error
```

A possible implementation:

```
#define sigemptyset(ptr)  (*(ptr) = 0)
#define sigfillset(ptr)   (*(ptr) = ~(sigset_t)0, 0)

/*
 * <signal.h> usually defines NSIG to include signal number 0.
 */
#define SIGBAD(signo)   ((signo) <= 0 || (signo) >= NSIG)

int sigaddset(sigset_t *set, int signo)
{
    if (SIGBAD(signo)) {
        errno = EINVAL;
        return(-1);
    }
    *set |= 1 << (signo - 1);       /* turn bit on */
    return(0);
}

int sigdelset(sigset_t *set, int signo)
{
    if (SIGBAD(signo)) {
        errno = EINVAL;
        return(-1);
    }
    *set &= ~(1 << (signo - 1));    /* turn bit off */
    return(0);
}

int sigismember(const sigset_t *set, int signo)
{
    if (SIGBAD(signo)) {
        errno = EINVAL;
        return(-1);
    }
    return((*set & (1 << (signo - 1))) != 0);
}
```

## sigprocmask() & sigpending() functions

```c
#include <signal.h>
// Mask of a process is the set of signals currently blocked from delivery to that process.
int sigprocmask(int how, const sigset_t *restrict set,
                sigset_t *restrict oset);

        how: SIG_BLOCK, SIG_UNBLOCK, or SIG_SETMASK

        Returns: 0 if OK, –1 on error
// Returns the set of signals that are blocked from delivery
int sigpending(sigset_t *set);

        Returns: 0 if OK, –1 on error
```

Example:

```c
#include "apue.h"

static void sig_quit(int);

int
main(void)
{
    sigset_t    newmask, oldmask, pendmask;

    if (signal(SIGQUIT, sig_quit) == SIG_ERR)
        err_sys("can′t catch SIGQUIT");

    /*
     * Block SIGQUIT and save current signal mask.
     */
    sigemptyset(&newmask);
    sigaddset(&newmask, SIGQUIT);
    if (sigprocmask(SIG_BLOCK, &newmask, &oldmask) < 0)
        err_sys("SIG_BLOCK error");

    sleep(5);   /* SIGQUIT here will remain pending */

    if (sigpending(&pendmask) < 0)
        err_sys("sigpending error");
    if (sigismember(&pendmask, SIGQUIT))
        printf("\nSIGQUIT pending\n");

    /*
     * Restore signal mask which unblocks SIGQUIT.
     */
    if (sigprocmask(SIG_SETMASK, &oldmask, NULL) < 0)
        err_sys("SIG_SETMASK error");
    printf("SIGQUIT unblocked\n");

    sleep(5);   /* SIGQUIT here will terminate with core file */
    exit(0);
}

static void
sig_quit(int signo)
{
    printf("caught SIGQUIT\n");
    if (signal(SIGQUIT, SIG_DFL) == SIG_ERR)
        err_sys("can′t reset SIGQUIT");
}
```

## sigaction() function

```c
#include <signal.h>

int sigaction(int signo, const struct sigaction *restrict act,
              struct sigaction *restrict oact);

        Returns: 0 if OK, –1 on error

struct sigaction {
  void     (*sa_handler)(int);  /* addr of signal handler, */
                                /* or SIG_IGN, or SIG_DFL */
  sigset_t sa_mask;             /* additional signals to block */
  int      sa_flags;            /* signal options, Figure 10.16 */
  /* alternate handler */
  void     (*sa_sigaction)(int, siginfo_t *, void *);
};
```

An implementation of `signal()` using sigaction:

```c
#include "apue.h"

/* Reliable version of signal(), using POSIX sigaction().  */
Sigfunc *
signal(int signo, Sigfunc *func)
{
    struct sigaction    act, oact;

    act.sa_handler = func;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    if (signo == SIGALRM) {
#ifdef  SA_INTERRUPT
        act.sa_flags |= SA_INTERRUPT; // System calls interrupted by this signal are not automattically restarted
#endif
    } else {
        act.sa_flags |= SA_RESTART; // System calls interrupted by this signal are automattically restarted
    }
    if (sigaction(signo, &act, &oact) < 0)
        return(SIG_ERR);
    return(oact.sa_handler);
}
```

1. Signal handler remains in place
2. Interrupted system calls automatically restart, except for SIGALRM

signal_intr() – an alternate version that does not restart system calls:

```c
#include "apue.h"

Sigfunc *
signal_intr(int signo, Sigfunc *func)
{
    struct sigaction    act, oact;

    act.sa_handler = func;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
#ifdef SA_INTERRUPT
    act.sa_flags |= SA_INTERRUPT;
#endif
    if (sigaction(signo, &act, &oact) < 0)
        return(SIG_ERR);
    return(oact.sa_handler);
}
```

