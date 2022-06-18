## Gracefully continuing execution after calling 'execve()'

Suppose we've have problem like this:

```C
#include <unistd.h>

int main(void)
{
	char *args = {"/bin/sh", NULL};
	char *envp = {NULL};

	// even bear know that this address is invalid.
	// but, leave it as it is, for now. :v :v
	char *foo = (char *)0x31337;

	// here comes the giant :)
	execve(args[0], args, envp);

	// what we want is to reach here.
	*foo[0] = 'A';

	return 0;
}
```

If we compiled and run this program above, we never get segmentation fault because execve() internally call exit() after running our given command or binary.

We'll know that execve() create a new process image (with a newly initialized stack, heap and (initialized / uninitialized) .bss or data segments), but doesn't mean it's separated by main process image, it's referenced.

In my opinion, strictly speaking, to solve this kind of mess is to wrap execve() inside new thread, and i'll prove it right, trivially using fork() and waitpid().

- if return value of fork() is 0, it means we're in the child, call execve() here.
- if return value of fork() greater than 0, call wait() or waitpid(-1, &ptr, 0). Why ```&ptr```, because second parameter of waitpid() requires a ```int *```.
- profit :)

```C
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(void)
{
	char *args[] = {"/bin/sh", NULL};
	char *envp[] = {NULL};
	char *foo = (char *)0x31337;
	int ret, wstatus;
	pid_t pid;

	pid = fork();

	if (pid < 0) {
		perror("fork");
		ret = pid;
		goto __fallback;
	}

	if (!pid) {
		// we're in the child
		execve(args[0], args, envp);
	} else {
		waitpid(-1, &wstatus, 0);
	}

	// exit /bin/sh, and you'll get the
	// segmentation fault.
	*foo[0] = 'A';

	return 0;

__fallback:
	return ret;
}
```

Compile and run this program, it will popping a shell, then exit a shell, you will get segmentation fault.