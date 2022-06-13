## 'memset()' implementation like Ulrich Drepper on Steroid :)

See the code and all that fanciness below, and this was so fast running on machine with low spec. Props to Ulrich Drepper for optimizing GCC and those related to that magnificent shit.

```C
/**
 * (c) Paulus Gandung Prakosa <gandung@galactic.demon.co.uk>
 */

#include <stdio.h>

#ifndef MAXLEN
#define MAXLEN 128
#endif

static char foo_bar_baz[MAXLEN + 1];

typedef struct regs {
  unsigned long rax;
  unsigned long rbx;
  unsigned long rcx;
  unsigned long rdx;
  unsigned long rdi;
  unsigned long rsi;
} __regs_state_t;

#define __initiate_regstate(name)                                              \
  __regs_state_t __##name = {                                                  \
      .rax = 0,                                                                \
      .rbx = 0,                                                                \
      .rcx = 0,                                                                \
      .rdx = 0,                                                                \
      .rdi = 0,                                                                \
      .rsi = 0,                                                                \
  };

#define __serialize_regstate(name) (__##name)

#ifndef __hot
#define __hot __attribute__((hot))
#endif

#ifndef __nocfi
#define __nocfi
#endif

#ifndef __sect_fastcall
#define __sect_fastcall __attribute__((section(".sect.fastcall")))
#endif

#ifndef __fastcall
#define __fastcall __sect_fastcall __hot __nocfi
#endif

#ifndef __rax
#define __rax(regs) ((regs)->rax)
#endif

#ifndef __rbx
#define __rbx(regs) ((regs)->rbx)
#endif

#ifndef __rcx
#define __rcx(regs) ((regs)->rcx)
#endif

#ifndef __rdx
#define __rdx(regs) ((regs)->rdx)
#endif

#ifndef __rdi
#define __rdi(regs) ((regs)->rdi)
#endif

#ifndef __rsi
#define __rsi(regs) ((regs)->rsi)
#endif

static inline void __fastcall save_rax(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%%rax, %0\r\n" : "=r"(__rax(regs)));
}

static inline void __fastcall save_rbx(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%%rbx, %0\r\n" : "=r"(__rbx(regs)));
}

static inline void __fastcall save_rcx(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%%rcx, %0\r\n" : "=r"(__rcx(regs)));
}

static inline void __fastcall save_rdx(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%%rdx, %0\r\n" : "=r"(__rdx(regs)));
}

static inline void __fastcall save_rdi(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%%rdi, %0\r\n" : "=r"(__rdi(regs)));
}

static inline void __fastcall save_rsi(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%%rsi, %0\r\n" : "=r"(__rsi(regs)));
}

static inline void __fastcall store_rax(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%0, %%rax\r\n" : : "r"(__rax(regs)));
}

static inline void __fastcall store_rbx(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%0, %%rbx\r\n" : : "r"(__rbx(regs)));
}

static inline void __fastcall store_rcx(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%0, %%rcx\r\n" : : "r"(__rcx(regs)));
}

static inline void __fastcall store_rdx(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%0, %%rdx\r\n" : : "r"(__rdx(regs)));
}

static inline void __fastcall store_rdi(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%0, %%rdi\r\n" : : "r"(__rdi(regs)));
}

static inline void __fastcall store_rsi(__regs_state_t *regs) {
  __asm__ __volatile__("movq	%0, %%rsi\r\n" : : "r"(__rsi(regs)));
}

static inline void __fastcall save_regs(__regs_state_t *regs) {
  save_rax(regs);
  save_rbx(regs);
  save_rcx(regs);
  save_rdx(regs);
  save_rdi(regs);
  save_rsi(regs);
}

static inline void __fastcall store_regs(__regs_state_t *regs) {
  store_rax(regs);
  store_rbx(regs);
  store_rcx(regs);
  store_rdx(regs);
  store_rdi(regs);
  store_rsi(regs);
}

static __initiate_regstate(regstate);

static inline void __fastcall *__fastcall_memset(void *buf, int c, size_t n) {
  int ref;

  save_regs(&__serialize_regstate(regstate));

  __asm__ __volatile__("leaq	(%%rax), %%rdi\n"
                       "xorq	%%rcx, %%rcx\n"
                       ".jump_point:\n"
                       "addq	%%rcx, %%rdi\n"
                       "movb	%%bl, (%%rdi)\n"
                       "subq	%%rcx, %%rdi\n"
                       "incq	%%rcx\n"
                       "cmpq	%%rcx, %%rdx\n"
                       "jne	.jump_point\n"
                       : "=a"(buf)
                       : "a"(buf), "b"((char)c), "d"(n)
                       : "memory");

  store_regs(&__serialize_regstate(regstate));
  return buf;
}

void *qmemset(void *buf, int c, size_t n) {
  return __fastcall_memset(buf, c, n);
}

int main(void) {
  printf("buffer: %p\n", foo_bar_baz);
  qmemset(foo_bar_baz, 'A', MAXLEN);
  printf("buffer: %p <%s>\n", foo_bar_baz, foo_bar_baz);
  return 0;
}
```
