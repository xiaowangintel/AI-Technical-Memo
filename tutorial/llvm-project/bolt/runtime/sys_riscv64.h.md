# sys_riscv64.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/runtime/sys_riscv64.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Save all registers while keeping 16B stack alignment. It also sits in code that belongs to the BOLT post-link optimizer subsystem. / 该文件属于 BOLT 链接后优化器子系统。 源码头部说明其职责是：Save all registers while keeping 16B stack alignment。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
#ifndef LLVM_TOOLS_LLVM_BOLT_SYS_RISCV
#define LLVM_TOOLS_LLVM_BOLT_SYS_RISCV

// Save all registers while keeping 16B stack alignment
#define SAVE_ALL                                                               \
  "addi sp, sp, -256\n"                                                        \
  "sd x1, 0(sp)\n"                                                             \
  "sd x2, 8(sp)\n"                                                             \
  "sd x3, 16(sp)\n"                                                            \
  "sd x4, 24(sp)\n"                                                            \
  "sd x5, 32(sp)\n"                                                            \
  "sd x6, 40(sp)\n"                                                            \
  "sd x7, 48(sp)\n"                                                            \
  "sd x8, 56(sp)\n"                                                            \
  "sd x9, 64(sp)\n"                                                            \
  "sd x10, 72(sp)\n"                                                           \
  "sd x11, 80(sp)\n"                                                           \
  "sd x12, 88(sp)\n"                                                           \
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `LLVM_TOOLS_LLVM_BOLT_SYS_RISCV`, `SAVE_ALL` for constants or compile-time switches. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `LLVM_TOOLS_LLVM_BOLT_SYS_RISCV`, `SAVE_ALL`，用于常量或编译期开关。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-36

```cpp
  "sd x13, 96(sp)\n"                                                           \
  "sd x14, 104(sp)\n"                                                          \
  "sd x15, 112(sp)\n"                                                          \
  "sd x16, 120(sp)\n"                                                          \
  "sd x17, 128(sp)\n"                                                          \
  "sd x18, 136(sp)\n"                                                          \
  "sd x19, 144(sp)\n"                                                          \
  "sd x20, 152(sp)\n"                                                          \
  "sd x21, 160(sp)\n"                                                          \
  "sd x22, 168(sp)\n"                                                          \
  "sd x23, 176(sp)\n"                                                          \
  "sd x24, 184(sp)\n"                                                          \
  "sd x25, 192(sp)\n"                                                          \
  "sd x26, 200(sp)\n"                                                          \
  "sd x27, 208(sp)\n"                                                          \
  "sd x28, 216(sp)\n"                                                          \
  "sd x29, 224(sp)\n"                                                          \
  "sd x30, 232(sp)\n"                                                          \
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 37-54

```cpp
  "sd x31, 240(sp)\n"
// Mirrors SAVE_ALL
#define RESTORE_ALL                                                            \
  "ld x1, 0(sp)\n"                                                             \
  "ld x2, 8(sp)\n"                                                             \
  "ld x3, 16(sp)\n"                                                            \
  "ld x4, 24(sp)\n"                                                            \
  "ld x5, 32(sp)\n"                                                            \
  "ld x6, 40(sp)\n"                                                            \
  "ld x7, 48(sp)\n"                                                            \
  "ld x8, 56(sp)\n"                                                            \
  "ld x9, 64(sp)\n"                                                            \
  "ld x10, 72(sp)\n"                                                           \
  "ld x11, 80(sp)\n"                                                           \
  "ld x12, 88(sp)\n"                                                           \
  "ld x13, 96(sp)\n"                                                           \
  "ld x14, 104(sp)\n"                                                          \
  "ld x15, 112(sp)\n"                                                          \
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `RESTORE_ALL` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `RESTORE_ALL`.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `RESTORE_ALL`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `RESTORE_ALL`。

### Lines 55-72

```cpp
  "ld x16, 120(sp)\n"                                                          \
  "ld x17, 128(sp)\n"                                                          \
  "ld x18, 136(sp)\n"                                                          \
  "ld x19, 144(sp)\n"                                                          \
  "ld x20, 152(sp)\n"                                                          \
  "ld x21, 160(sp)\n"                                                          \
  "ld x22, 168(sp)\n"                                                          \
  "ld x23, 176(sp)\n"                                                          \
  "ld x24, 184(sp)\n"                                                          \
  "ld x25, 192(sp)\n"                                                          \
  "ld x26, 200(sp)\n"                                                          \
  "ld x27, 208(sp)\n"                                                          \
  "ld x28, 216(sp)\n"                                                          \
  "ld x29, 224(sp)\n"                                                          \
  "ld x30, 232(sp)\n"                                                          \
  "ld x31, 240(sp)\n"                                                          \
  "addi sp, sp,  256\n"
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 73-90

```cpp
// Anonymous namespace covering everything but our library entry point
namespace {

// Get the difference between runtime address of .text section and
// static address in section header table. Can be extracted from arbitrary
// pc value recorded at runtime to get the corresponding static address, which
// in turn can be used to search for indirect call description. Needed because
// indirect call descriptions are read-only non-relocatable data.
uint64_t getTextBaseAddress() {
  uint64_t DynAddr;
  uint64_t StaticAddr;
  __asm__ volatile("lla %0, __hot_end\n\t"
                   "lui %1, %%hi(__hot_end)\n\t"
                   "addi %1, %1, %%lo(__hot_end)\n\t"
                   : "=r"(DynAddr), "=r"(StaticAddr));
  return DynAddr - StaticAddr;
}
```

- EN: Works inside namespace scope `covering` to organize symbols. Declares or implements routines including `getTextBaseAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTextBaseAddress`, `covering`.
- CN: 这里位于命名空间 `covering` 中，用于组织符号作用域。这里声明或实现函数，例如 `getTextBaseAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTextBaseAddress`, `covering`。

### Lines 91-105

```cpp
uint64_t __read(uint64_t fd, const void *buf, uint64_t count) {
  uint64_t ret;
  register uint64_t a0 __asm__("a0") = fd;
  register const void *a1 __asm__("a1") = buf;
  register uint64_t a2 __asm__("a2") = count;
  register uint64_t a7 __asm__("a7") =
      63; // Assuming 63 is the syscall number for read
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret)
                       : "r"(a0), "r"(a1), "r"(a2), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__read`, `__asm__`. Notable symbols here include `__read`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__read`, `__asm__`。这里较值得关注的符号包括 `__read`, `__asm__`。

### Lines 106-120

```cpp
uint64_t __write(uint64_t fd, const void *buf, uint64_t count) {
  uint64_t ret;
  register uint64_t a0 __asm__("a0") = fd;
  register const void *a1 __asm__("a1") = buf;
  register uint64_t a2 __asm__("a2") = count;
  register uint32_t a7 __asm__("a7") =
      64; // Assuming 64 is the syscall number for write
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret)
                       : "r"(a0), "r"(a1), "r"(a2), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__write`, `__asm__`. Notable symbols here include `__write`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__write`, `__asm__`。这里较值得关注的符号包括 `__write`, `__asm__`。

### Lines 121-138

```cpp
void *__mmap(uint64_t addr, uint64_t size, uint64_t prot, uint64_t flags,
             uint64_t fd, uint64_t offset) {
  void *ret;
  register uint64_t a0 __asm__("a0") = addr;
  register uint64_t a1 __asm__("a1") = size;
  register uint64_t a2 __asm__("a2") = prot;
  register uint64_t a3 __asm__("a3") = flags;
  register uint64_t a4 __asm__("a4") = fd;
  register uint64_t a5 __asm__("a5") = offset;
  register uint32_t a7 __asm__("a7") =
      222; // Assuming 222 is the syscall number for mmap
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret)
                       : "r"(a0), "r"(a1), "r"(a2), "r"(a3), "r"(a4), "r"(a5),
                         "r"(a7)
                       : "memory");
  return ret;
```

- EN: Declares or implements routines including `__asm__`. Notable symbols here include `__asm__`.
- CN: 这里声明或实现函数，例如 `__asm__`。这里较值得关注的符号包括 `__asm__`。

### Lines 139-153

```cpp
}

uint64_t __munmap(void *addr, uint64_t size) {
  uint64_t ret;
  register void *a0 __asm__("a0") = addr;
  register uint64_t a1 __asm__("a1") = size;
  register uint32_t a7 __asm__("a7") = 215;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__munmap`, `__asm__`. Notable symbols here include `__munmap`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__munmap`, `__asm__`。这里较值得关注的符号包括 `__munmap`, `__asm__`。

### Lines 154-165

```cpp
uint64_t __exit(uint64_t code) {
  uint64_t ret;
  register uint64_t a0 __asm__("a0") = code;
  register uint32_t a7 __asm__("a7") = 94;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__exit`, `__asm__`. Notable symbols here include `__exit`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__exit`, `__asm__`。这里较值得关注的符号包括 `__exit`, `__asm__`。

### Lines 166-182

```cpp
uint64_t __open(const char *pathname, uint64_t flags, uint64_t mode) {
  uint64_t ret;
  register int a0 __asm__("a0") =
      -100; // Assuming -100 is an invalid file descriptor
  register const char *a1 __asm__("a1") = pathname;
  register uint64_t a2 __asm__("a2") = flags;
  register uint64_t a3 __asm__("a3") = mode;
  register uint64_t a7 __asm__("a7") =
      56; // Assuming 56 is the syscall number for open
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret)
                       : "r"(a0), "r"(a1), "r"(a2), "r"(a3), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__open`, `__asm__`. Notable symbols here include `__open`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__open`, `__asm__`。这里较值得关注的符号包括 `__open`, `__asm__`。

### Lines 183-196

```cpp
long __getdents64(unsigned int fd, dirent64 *dirp, size_t count) {
  long ret;
  register unsigned int a0 __asm__("a0") = fd;
  register dirent64 *a1 __asm__("a1") = dirp;
  register size_t a2 __asm__("a2") = count;
  register uint32_t a7 __asm__("a7") = 61;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__getdents64`, `__asm__`. Notable symbols here include `__getdents64`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getdents64`, `__asm__`。这里较值得关注的符号包括 `__getdents64`, `__asm__`。

### Lines 197-211

```cpp
uint64_t __readlink(const char *pathname, char *buf, size_t bufsize) {
  uint64_t ret;
  register int a0 __asm__("a0") = -100;
  register const char *a1 __asm__("a1") = pathname;
  register char *a2 __asm__("a2") = buf;
  register size_t a3 __asm__("a3") = bufsize;
  register uint32_t a7 __asm__("a7") = 78; // readlinkat
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a3), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__readlink`, `__asm__`. Notable symbols here include `__readlink`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__readlink`, `__asm__`。这里较值得关注的符号包括 `__readlink`, `__asm__`。

### Lines 212-225

```cpp
uint64_t __lseek(uint64_t fd, uint64_t pos, uint64_t whence) {
  uint64_t ret;
  register uint64_t a0 __asm__("a0") = fd;
  register uint64_t a1 __asm__("a1") = pos;
  register uint64_t a2 __asm__("a2") = whence;
  register uint32_t a7 __asm__("a7") = 62;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__lseek`, `__asm__`. Notable symbols here include `__lseek`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__lseek`, `__asm__`。这里较值得关注的符号包括 `__lseek`, `__asm__`。

### Lines 226-238

```cpp
int __ftruncate(uint64_t fd, uint64_t length) {
  int ret;
  register uint64_t a0 __asm__("a0") = fd;
  register uint64_t a1 __asm__("a1") = length;
  register uint32_t a7 __asm__("a7") = 46;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__ftruncate`, `__asm__`. Notable symbols here include `__ftruncate`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__ftruncate`, `__asm__`。这里较值得关注的符号包括 `__ftruncate`, `__asm__`。

### Lines 239-251

```cpp
int __close(uint64_t fd) {
  int ret;
  register uint64_t a0 __asm__("a0") = fd;
  register uint32_t a7 __asm__("a7") =
      57; // Assuming 57 is the syscall number for close
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret)
                       : "r"(a0), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__close`, `__asm__`. Notable symbols here include `__close`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__close`, `__asm__`。这里较值得关注的符号包括 `__close`, `__asm__`。

### Lines 252-265

```cpp
int __madvise(void *addr, size_t length, int advice) {
  int ret;
  register void *a0 __asm__("a0") = addr;
  register size_t a1 __asm__("a1") = length;
  register int a2 __asm__("a2") = advice;
  register uint32_t a7 __asm__("a7") = 233;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__madvise`, `__asm__`. Notable symbols here include `__madvise`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__madvise`, `__asm__`。这里较值得关注的符号包括 `__madvise`, `__asm__`。

### Lines 266-277

```cpp
int __uname(struct UtsNameTy *buf) {
  int ret;
  register UtsNameTy *a0 __asm__("a0") = buf;
  register uint32_t a7 __asm__("a7") = 160;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Introduces type definitions such as `UtsNameTy`. Declares or implements routines including `__uname`, `__asm__`. Notable symbols here include `UtsNameTy`, `__uname`, `__asm__`.
- CN: 这里引入类型定义，例如 `UtsNameTy`。这里声明或实现函数，例如 `__uname`, `__asm__`。这里较值得关注的符号包括 `UtsNameTy`, `__uname`, `__asm__`。

### Lines 278-290

```cpp
uint64_t __nanosleep(const timespec *req, timespec *rem) {
  uint64_t ret;
  register const timespec *a0 __asm__("a0") = req;
  register timespec *a1 __asm__("a1") = rem;
  register uint32_t a7 __asm__("a7") = 101;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__nanosleep`, `__asm__`. Notable symbols here include `__nanosleep`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__nanosleep`, `__asm__`。这里较值得关注的符号包括 `__nanosleep`, `__asm__`。

### Lines 291-308

```cpp
int64_t __fork() {
  uint64_t ret;
  // clone instead of fork with flags
  // "CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD"
  register uint64_t a0 __asm__("a0") = 0x1200011;
  register uint64_t a1 __asm__("a1") = 0;
  register uint64_t a2 __asm__("a2") = 0;
  register uint64_t a3 __asm__("a3") = 0;
  register uint64_t a4 __asm__("a4") = 0;
  register uint32_t a7 __asm__("a7") = 220;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a3), "r"(a4), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__fork`, `__asm__`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `__fork`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__fork`, `__asm__`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `__fork`, `__asm__`。

### Lines 309-322

```cpp
int __mprotect(void *addr, size_t len, int prot) {
  int ret;
  register void *a0 __asm__("a0") = addr;
  register size_t a1 __asm__("a1") = len;
  register int a2 __asm__("a2") = prot;
  register uint32_t a7 __asm__("a7") = 226;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__mprotect`, `__asm__`. Notable symbols here include `__mprotect`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__mprotect`, `__asm__`。这里较值得关注的符号包括 `__mprotect`, `__asm__`。

### Lines 323-333

```cpp
uint64_t __getpid() {
  uint64_t ret;
  register uint32_t a7 __asm__("a7") = 172;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__getpid`, `__asm__`. Notable symbols here include `__getpid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getpid`, `__asm__`。这里较值得关注的符号包括 `__getpid`, `__asm__`。

### Lines 334-344

```cpp
uint64_t __getppid() {
  uint64_t ret;
  register uint32_t a7 __asm__("a7") = 173;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__getppid`, `__asm__`. Notable symbols here include `__getppid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getppid`, `__asm__`。这里较值得关注的符号包括 `__getppid`, `__asm__`。

### Lines 345-357

```cpp
int __setpgid(uint64_t pid, uint64_t pgid) {
  int ret;
  register uint64_t a0 __asm__("a0") = pid;
  register uint64_t a1 __asm__("a1") = pgid;
  register uint32_t a7 __asm__("a7") = 154;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__setpgid`, `__asm__`. Notable symbols here include `__setpgid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__setpgid`, `__asm__`。这里较值得关注的符号包括 `__setpgid`, `__asm__`。

### Lines 358-369

```cpp
uint64_t __getpgid(uint64_t pid) {
  uint64_t ret;
  register uint64_t a0 __asm__("a0") = pid;
  register uint32_t a7 __asm__("a7") = 155;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__getpgid`, `__asm__`. Notable symbols here include `__getpgid`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__getpgid`, `__asm__`。这里较值得关注的符号包括 `__getpgid`, `__asm__`。

### Lines 370-382

```cpp
int __kill(uint64_t pid, int sig) {
  int ret;
  register uint64_t a0 __asm__("a0") = pid;
  register int a1 __asm__("a1") = sig;
  register uint32_t a7 __asm__("a7") = 129;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__kill`, `__asm__`. Notable symbols here include `__kill`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__kill`, `__asm__`。这里较值得关注的符号包括 `__kill`, `__asm__`。

### Lines 383-394

```cpp
int __fsync(int fd) {
  int ret;
  register int a0 __asm__("a0") = fd;
  register uint32_t a7 __asm__("a7") = 82;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0)
                       : "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__fsync`, `__asm__`. Notable symbols here include `__fsync`, `__asm__`.
- CN: 这里声明或实现函数，例如 `__fsync`, `__asm__`。这里较值得关注的符号包括 `__fsync`, `__asm__`。

### Lines 395-409

```cpp
uint64_t __sigprocmask(int how, const void *set, void *oldset) {
  uint64_t ret;
  register int a0 __asm__("a0") = how;
  register const void *a1 __asm__("a1") = set;
  register void *a2 __asm__("a2") = oldset;
  register long a3 asm("a3") = 8;
  register uint32_t a7 __asm__("a7") = 135;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a3), "r"(a7)
                       : "memory");
  return ret;
}
```

- EN: Declares or implements routines including `__sigprocmask`, `__asm__`, `asm`. Notable symbols here include `__sigprocmask`, `__asm__`, `asm`.
- CN: 这里声明或实现函数，例如 `__sigprocmask`, `__asm__`, `asm`。这里较值得关注的符号包括 `__sigprocmask`, `__asm__`, `asm`。

### Lines 410-427

```cpp
int __prctl(int option, unsigned long arg2, unsigned long arg3,
            unsigned long arg4, unsigned long arg5) {
  int ret;
  register int a0 __asm__("a0") = option;
  register unsigned long a1 __asm__("a1") = arg2;
  register unsigned long a2 __asm__("a2") = arg3;
  register unsigned long a3 __asm__("a3") = arg4;
  register unsigned long a4 __asm__("a4") = arg5;
  register uint32_t a7 __asm__("a7") = 167;
  __asm__ __volatile__("ecall\n\t"
                       "mv %0, a0"
                       : "=r"(ret), "+r"(a0), "+r"(a1)
                       : "r"(a2), "r"(a3), "r"(a4), "r"(a7)
                       : "cc", "memory");
  return ret;
}
} // anonymous namespace
```

- EN: Declares or implements routines including `__asm__`. Notable symbols here include `__asm__`.
- CN: 这里声明或实现函数，例如 `__asm__`。这里较值得关注的符号包括 `__asm__`。

### Lines 428-428

```cpp
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
- `LLVM_TOOLS_LLVM_BOLT_SYS_RISCV`: macro or compile-time switch / 宏或编译期开关
- `SAVE_ALL`: macro or compile-time switch / 宏或编译期开关

## Dependencies / 依赖关系

- Directory context / 目录上下文: `bolt/runtime` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/runtime` 下的相邻文件通常与本文件协作组成对应子系统
