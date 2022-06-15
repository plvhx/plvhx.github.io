## The real usage of GCC 'regparm' function attribute.

See the code header comments.

```C
/*
 * Compile in 32-bit mode if u'r in 64-bit environment. If you
 * are already in 32-bit environment, leave it as it is.
 *
 * Run inside GDB and inspect eax, edx, and ecx regs.
 * You will be amazed. :)
 */
#include <stdio.h>

#ifndef __regparm
#define __regparm(x) __attribute__((regparm((x))))
#endif

typedef struct __regparm_state {
  unsigned long eax;
  unsigned long edx;
  unsigned long ecx;
} regparm_state_t;

static regparm_state_t rst = {
    .eax = 0,
    .edx = 0,
    .ecx = 0,
};

#define __intercept(state)                                                     \
  do {                                                                         \
    __asm__ __volatile__("nop\n"                                               \
                         "xchgl	%%eax, %%ecx\n"                                \
                         : "=r"(state.eax), "=r"(state.edx), "=r"(state.ecx)); \
    printf("eax: %lu, edx: %lu, ecx: %lu\n", state.eax, state.edx, state.ecx); \
  } while (0)

static void __regparm(3) foobar(int a, int b, int c) {
  __intercept(rst);
  __asm__("int3");
}

int main(void) {
  foobar(7373, 31337, 1440);
  return 0;
}
```