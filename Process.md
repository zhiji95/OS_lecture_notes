# Process

### Displaying process hierarchy

```bash
ps axf     # display process tree

ps axfj    # with more info

ps axfjww  # even if lines wrap around terminal
```

### Creating processes

Simple shell program from APUE3, 1.6:

```c
#include "apue.h"
#include <sys/wait.h>

int
main(void)
{
        char	buf[MAXLINE];	/* from apue.h */
        pid_t	pid;
        int		status;

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
```
