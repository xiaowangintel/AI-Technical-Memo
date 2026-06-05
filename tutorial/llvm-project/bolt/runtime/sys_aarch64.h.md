# sys_aarch64.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/runtime/sys_aarch64.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Save all registers while keeping 16B stack alignment. It also sits in code that belongs to the BOLT post-link optimizer subsystem. / 该文件属于 BOLT 链接后优化器子系统。 源码头部说明其职责是：Save all registers while keeping 16B stack alignment。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
#ifndef LLVM_TOOLS_LLVM_BOLT_SYS_AARCH64
#define LLVM_TOOLS_LLVM_BOLT_SYS_AARCH64

// Save all registers while keeping 16B stack alignment
#define SAVE_ALL                                                               \
  "stp x0, x1, [sp, #-16]!\n"                                                  \
  "stp x2, x3, [sp, #-16]!\n"                                                  \
  "stp x4, x5, [sp, #-16]!\n"                                                  \
  "stp x6, x7, [sp, #-16]!\n"                                                  \
  "stp x8, x9, [sp, #-16]!\n"                                                  \
  "stp x10, x11, [sp, #-16]!\n"                                                \
  "stp x12, x13, [sp, #-16]!\n"                                                \
  "stp x14, x15, [sp, #-16]!\n"                                                \
  "stp x16, x17, [sp, #-16]!\n"                                                \
  "stp x18, x19, [sp, #-16]!\n"                                                \
  "stp x20, x21, [sp, #-16]!\n"                                                \
  "stp x22, x23, [sp, #-16]!\n"                                                \
  "stp x24, x25, [sp, #-16]!\n"                                                \
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `LLVM_TOOLS_LLVM_BOLT_SYS_AARCH64`, `SAVE_ALL` for constants or compile-time switches. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `LLVM_TOOLS_LLVM_BOLT_SYS_AARCH64`, `SAVE_ALL`，用于常量或编译期开关。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-36

```cpp
  "stp x26, x27, [sp, #-16]!\n"                                                \
  "stp x28, x29, [sp, #-16]!\n"                                                \
  "mrs x29, nzcv\n"                                                            \
  "stp x29, x30, [sp, #-16]!\n"
// Mirrors SAVE_ALL
#define RESTORE_ALL                                                            \
  "ldp x29, x30, [sp], #16\n"                                                  \
  "msr nzcv, x29\n"                                                            \
  "ldp x28, x29, [sp], #16\n"                                                  \
  "ldp x26, x27, [sp], #16\n"                                                  \
  "ldp x24, x25, [sp], #16\n"                                                  \
  "ldp x22, x23, [sp], #16\n"                                                  \
  "ldp x20, x21, [sp], #16\n"                                                  \
  "ldp x18, x19, [sp], #16\n"                                                  \
  "ldp x16, x17, [sp], #16\n"                                                  \
  "ldp x14, x15, [sp], #16\n"                                                  \
  "ldp x12, x13, [sp], #16\n"                                                  \
  "ldp x10, x11, [sp], #16\n"                                                  \
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `RESTORE_ALL` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `RESTORE_ALL`.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `RESTORE_ALL`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `RESTORE_ALL`。

### Lines 37-45

```cpp
  "ldp x8, x9, [sp], #16\n"                                                    \
  "ldp x6, x7, [sp], #16\n"                                                    \
  "ldp x4, x5, [sp], #16\n"                                                    \
  "ldp x2, x3, [sp], #16\n"                                                    \
  "ldp x0, x1, [sp], #16\n"

// Anonymous namespace covering everything but our library entry point
namespace {
```

- EN: Works inside namespace scope `covering` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `covering`.
- CN: 这里位于命名空间 `covering` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `covering`。

### Lines 46-63

```cpp
// Get the difference between runtime address of .text section and
// static address in section header table. Can be extracted from arbitrary
// pc value recorded at runtime to get the corresponding static address, which
// in turn can be used to search for indirect call description. Needed because
// indirect call descriptions are read-only non-relocatable data.
uint64_t getTextBaseAddress() {
  uint64_t DynAddr;
  uint64_t StaticAddr;
  __asm__ volatile("b .instr%=\n\t"
                   ".StaticAddr%=:\n\t"
                   ".dword __hot_end\n\t"
                   ".instr%=:\n\t"
                   "ldr %0, .StaticAddr%=\n\t"
                   "adrp %1, __hot_end\n\t"
                   "add %1, %1, :lo12:__hot_end\n\t"
                   : "=r"(StaticAddr), "=r"(DynAddr));
  return DynAddr - StaticAddr;
}
```

- EN: Declares or implements routines including `getTextBaseAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTextBaseAddress`.
- CN: 这里声明或实现函数，例如 `getTextBaseAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTextBaseAddress`。

### Lines 64-78

```cpp

uint64_t __read(uint64_t fd, const void *buf, uint64_t count) {
  uint64_t ret;
  register uint64_t x0 __asm__("x0") = fd;
  register const void *x1 __asm__("x1") = buf;
  register uint64_t x2 __asm__("x2") = count;
  register uint32_t w8 __asm__("w8") = 63;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__read`, `__asm__`. Notable symbols here include `__read`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__read`, `__asm__`。这里较值得关注的符号包括 `__read`, `__asm__`。

### Lines 79-92

```cpp
uint64_t __write(uint64_t fd, const void *buf, uint64_t count) {
  uint64_t ret;
  register uint64_t x0 __asm__("x0") = fd;
  register const void *x1 __asm__("x1") = buf;
  register uint64_t x2 __asm__("x2") = count;
  register uint32_t w8 __asm__("w8") = 64;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__write`, `__asm__`. Notable symbols here include `__write`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__write`, `__asm__`。这里较值得关注的符号包括 `__write`, `__asm__`。

### Lines 93-110

```cpp
void *__mmap(uint64_t addr, uint64_t size, uint64_t prot, uint64_t flags,
             uint64_t fd, uint64_t offset) {
  void *ret;
  register uint64_t x0 __asm__("x0") = addr;
  register uint64_t x1 __asm__("x1") = size;
  register uint64_t x2 __asm__("x2") = prot;
  register uint64_t x3 __asm__("x3") = flags;
  register uint64_t x4 __asm__("x4") = fd;
  register uint64_t x5 __asm__("x5") = offset;
  register uint32_t w8 __asm__("w8") = 222;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(x3), "r"(x4), "r"(x5), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__asm__`. Notable symbols here include `__asm__`.
- CN: 这里声明或实现函数，例如 `__asm__`。这里较值得关注的符号包括 `__asm__`。

### Lines 111-123

```cpp
uint64_t __munmap(void *addr, uint64_t size) {
  uint64_t ret;
  register void *x0 __asm__("x0") = addr;
  register uint64_t x1 __asm__("x1") = size;
  register uint32_t w8 __asm__("w8") = 215;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__munmap`, `__asm__`. Notable symbols here include `__munmap`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__munmap`, `__asm__`。这里较值得关注的符号包括 `__munmap`, `__asm__`。

### Lines 124-135

```cpp
uint64_t __exit(uint64_t code) {
  uint64_t ret;
  register uint64_t x0 __asm__("x0") = code;
  register uint32_t w8 __asm__("w8") = 94;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0)
                       : "r"(w8)
                       : "cc", "memory", "x1");
  return ret;
}
```

- EN: Declares or implements routines including `__exit`, `__asm__`. Notable symbols here include `__exit`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__exit`, `__asm__`。这里较值得关注的符号包括 `__exit`, `__asm__`。

### Lines 136-150

```cpp
uint64_t __open(const char *pathname, uint64_t flags, uint64_t mode) {
  uint64_t ret;
  register int x0 __asm__("x0") = -100;
  register const char *x1 __asm__("x1") = pathname;
  register uint64_t x2 __asm__("x2") = flags;
  register uint64_t x3 __asm__("x3") = mode;
  register uint32_t w8 __asm__("w8") = 56;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(x3), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__open`, `__asm__`. Notable symbols here include `__open`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__open`, `__asm__`。这里较值得关注的符号包括 `__open`, `__asm__`。

### Lines 151-164

```cpp
long __getdents64(unsigned int fd, dirent64 *dirp, size_t count) {
  long ret;
  register unsigned int x0 __asm__("x0") = fd;
  register dirent64 *x1 __asm__("x1") = dirp;
  register size_t x2 __asm__("x2") = count;
  register uint32_t w8 __asm__("w8") = 61;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__getdents64`, `__asm__`. Notable symbols here include `__getdents64`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getdents64`, `__asm__`。这里较值得关注的符号包括 `__getdents64`, `__asm__`。

### Lines 165-179

```cpp
uint64_t __readlink(const char *pathname, char *buf, size_t bufsize) {
  uint64_t ret;
  register int x0 __asm__("x0") = -100;
  register const char *x1 __asm__("x1") = pathname;
  register char *x2 __asm__("x2") = buf;
  register size_t x3 __asm__("x3") = bufsize;
  register uint32_t w8 __asm__("w8") = 78; // readlinkat
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(x3), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__readlink`, `__asm__`. Notable symbols here include `__readlink`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__readlink`, `__asm__`。这里较值得关注的符号包括 `__readlink`, `__asm__`。

### Lines 180-193

```cpp
uint64_t __lseek(uint64_t fd, uint64_t pos, uint64_t whence) {
  uint64_t ret;
  register uint64_t x0 __asm__("x0") = fd;
  register uint64_t x1 __asm__("x1") = pos;
  register uint64_t x2 __asm__("x2") = whence;
  register uint32_t w8 __asm__("w8") = 62;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__lseek`, `__asm__`. Notable symbols here include `__lseek`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__lseek`, `__asm__`。这里较值得关注的符号包括 `__lseek`, `__asm__`。

### Lines 194-206

```cpp
int __ftruncate(uint64_t fd, uint64_t length) {
  int ret;
  register uint64_t x0 __asm__("x0") = fd;
  register uint64_t x1 __asm__("x1") = length;
  register uint32_t w8 __asm__("w8") = 46;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__ftruncate`, `__asm__`. Notable symbols here include `__ftruncate`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__ftruncate`, `__asm__`。这里较值得关注的符号包括 `__ftruncate`, `__asm__`。

### Lines 207-218

```cpp
int __close(uint64_t fd) {
  int ret;
  register uint64_t x0 __asm__("x0") = fd;
  register uint32_t w8 __asm__("w8") = 57;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0)
                       : "r"(w8)
                       : "cc", "memory", "x1");
  return ret;
}
```

- EN: Declares or implements routines including `__close`, `__asm__`. Notable symbols here include `__close`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__close`, `__asm__`。这里较值得关注的符号包括 `__close`, `__asm__`。

### Lines 219-232

```cpp
int __madvise(void *addr, size_t length, int advice) {
  int ret;
  register void *x0 __asm__("x0") = addr;
  register size_t x1 __asm__("x1") = length;
  register int x2 __asm__("x2") = advice;
  register uint32_t w8 __asm__("w8") = 233;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__madvise`, `__asm__`. Notable symbols here include `__madvise`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__madvise`, `__asm__`。这里较值得关注的符号包括 `__madvise`, `__asm__`。

### Lines 233-244

```cpp
int __uname(struct UtsNameTy *buf) {
  int ret;
  register UtsNameTy *x0 __asm__("x0") = buf;
  register uint32_t w8 __asm__("w8") = 160;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0)
                       : "r"(w8)
                       : "cc", "memory", "x1");
  return ret;
}
```

- EN: Introduces type definitions such as `UtsNameTy`. Declares or implements routines including `__uname`, `__asm__`. Notable symbols here include `UtsNameTy`, `__uname`, `__asm__`.
- CN: 这里引入类型定义，例如 `UtsNameTy`。这里声明或实现函数，例如 `__uname`, `__asm__`。这里较值得关注的符号包括 `UtsNameTy`, `__uname`, `__asm__`。

### Lines 245-257

```cpp
uint64_t __nanosleep(const timespec *req, timespec *rem) {
  uint64_t ret;
  register const timespec *x0 __asm__("x0") = req;
  register timespec *x1 __asm__("x1") = rem;
  register uint32_t w8 __asm__("w8") = 101;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__nanosleep`, `__asm__`. Notable symbols here include `__nanosleep`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__nanosleep`, `__asm__`。这里较值得关注的符号包括 `__nanosleep`, `__asm__`。

### Lines 258-275

```cpp
int64_t __fork() {
  uint64_t ret;
  // clone instead of fork with flags
  // "CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD"
  register uint64_t x0 __asm__("x0") = 0x1200011;
  register uint64_t x1 __asm__("x1") = 0;
  register uint64_t x2 __asm__("x2") = 0;
  register uint64_t x3 __asm__("x3") = 0;
  register uint64_t x4 __asm__("x4") = 0;
  register uint32_t w8 __asm__("w8") = 220;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(x3), "r"(x4), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__fork`, `__asm__`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `__fork`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__fork`, `__asm__`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `__fork`, `__asm__`。

### Lines 276-289

```cpp
int __mprotect(void *addr, size_t len, int prot) {
  int ret;
  register void *x0 __asm__("x0") = addr;
  register size_t x1 __asm__("x1") = len;
  register int x2 __asm__("x2") = prot;
  register uint32_t w8 __asm__("w8") = 226;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__mprotect`, `__asm__`. Notable symbols here include `__mprotect`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__mprotect`, `__asm__`。这里较值得关注的符号包括 `__mprotect`, `__asm__`。

### Lines 290-300

```cpp
uint64_t __getpid() {
  uint64_t ret;
  register uint32_t w8 __asm__("w8") = 172;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret)
                       : "r"(w8)
                       : "cc", "memory", "x0", "x1");
  return ret;
}
```

- EN: Declares or implements routines including `__getpid`, `__asm__`. Notable symbols here include `__getpid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getpid`, `__asm__`。这里较值得关注的符号包括 `__getpid`, `__asm__`。

### Lines 301-311

```cpp
uint64_t __getppid() {
  uint64_t ret;
  register uint32_t w8 __asm__("w8") = 173;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret)
                       : "r"(w8)
                       : "cc", "memory", "x0", "x1");
  return ret;
}
```

- EN: Declares or implements routines including `__getppid`, `__asm__`. Notable symbols here include `__getppid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getppid`, `__asm__`。这里较值得关注的符号包括 `__getppid`, `__asm__`。

### Lines 312-324

```cpp
int __setpgid(uint64_t pid, uint64_t pgid) {
  int ret;
  register uint64_t x0 __asm__("x0") = pid;
  register uint64_t x1 __asm__("x1") = pgid;
  register uint32_t w8 __asm__("w8") = 154;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__setpgid`, `__asm__`. Notable symbols here include `__setpgid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__setpgid`, `__asm__`。这里较值得关注的符号包括 `__setpgid`, `__asm__`。

### Lines 325-336

```cpp
uint64_t __getpgid(uint64_t pid) {
  uint64_t ret;
  register uint64_t x0 __asm__("x0") = pid;
  register uint32_t w8 __asm__("w8") = 155;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0)
                       : "r"(w8)
                       : "cc", "memory", "x1");
  return ret;
}
```

- EN: Declares or implements routines including `__getpgid`, `__asm__`. Notable symbols here include `__getpgid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getpgid`, `__asm__`。这里较值得关注的符号包括 `__getpgid`, `__asm__`。

### Lines 337-349

```cpp
int __kill(uint64_t pid, int sig) {
  int ret;
  register uint64_t x0 __asm__("x0") = pid;
  register int x1 __asm__("x1") = sig;
  register uint32_t w8 __asm__("w8") = 129;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__kill`, `__asm__`. Notable symbols here include `__kill`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__kill`, `__asm__`。这里较值得关注的符号包括 `__kill`, `__asm__`。

### Lines 350-361

```cpp
int __fsync(int fd) {
  int ret;
  register int x0 __asm__("x0") = fd;
  register uint32_t w8 __asm__("w8") = 82;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0)
                       : "r"(w8)
                       : "cc", "memory", "x1");
  return ret;
}
```

- EN: Declares or implements routines including `__fsync`, `__asm__`. Notable symbols here include `__fsync`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__fsync`, `__asm__`。这里较值得关注的符号包括 `__fsync`, `__asm__`。

### Lines 362-376

```cpp
uint64_t __sigprocmask(int how, const void *set, void *oldset) {
  uint64_t ret;
  register int x0 __asm__("x0") = how;
  register const void *x1 __asm__("x1") = set;
  register void *x2 __asm__("x2") = oldset;
  register long x3 asm("x3") = 8;
  register uint32_t w8 __asm__("w8") = 135;
  __asm__ __volatile__("svc #0\n"
                       "mov %0, x0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(x3), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__sigprocmask`, `__asm__`, `asm`. Notable symbols here include `__sigprocmask`, `__asm__`, `asm`.
- CN: 这里声明或实现函数，例如 `__sigprocmask`, `__asm__`, `asm`。这里较值得关注的符号包括 `__sigprocmask`, `__asm__`, `asm`。

### Lines 377-393

```cpp
int __prctl(int option, unsigned long arg2, unsigned long arg3,
            unsigned long arg4, unsigned long arg5) {
  int ret;
  register int x0 __asm__("x0") = option;
  register unsigned long x1 __asm__("x1") = arg2;
  register unsigned long x2 __asm__("x2") = arg3;
  register unsigned long x3 __asm__("x3") = arg4;
  register unsigned long x4 __asm__("x4") = arg5;
  register uint32_t w8 __asm__("w8") = 167;
  __asm__ __volatile__("svc #0\n"
                       "mov %w0, w0"
                       : "=r"(ret), "+r"(x0), "+r"(x1)
                       : "r"(x2), "r"(x3), "r"(x4), "r"(w8)
                       : "cc", "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__asm__`. Notable symbols here include `__asm__`.
- CN: 这里声明或实现函数，例如 `__asm__`。这里较值得关注的符号包括 `__asm__`。

### Lines 394-396

```cpp
} // anonymous namespace

#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `UtsNameTy`: class or struct interface / 类或结构体接口
- `getTextBaseAddress`: function or method entry point / 函数或方法入口
- `__read`: function or method entry point / 函数或方法入口
- `__asm__`: function or method entry point / 函数或方法入口
- `__write`: function or method entry point / 函数或方法入口
- `__munmap`: function or method entry point / 函数或方法入口
- `LLVM_TOOLS_LLVM_BOLT_SYS_AARCH64`: macro or compile-time switch / 宏或编译期开关
- `SAVE_ALL`: macro or compile-time switch / 宏或编译期开关

## Dependencies / 依赖关系

- Directory context / 目录上下文: `bolt/runtime` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/runtime` 下的相邻文件通常与本文件协作组成对应子系统
