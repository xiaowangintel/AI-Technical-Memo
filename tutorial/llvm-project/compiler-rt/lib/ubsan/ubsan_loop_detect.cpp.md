# ubsan_loop_detect.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `compiler-rt/lib/ubsan/ubsan_loop_detect.cpp`
- **Repository**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: Runtime support for -fsanitize-trap-loop.
- **目的（中文）**: 该实现文件提供与 `UndefinedBehaviorSanitizer loop detect` 相关的运行时逻辑。

## Line-by-Line Analysis / 逐行分析
### Line 1
````cpp
//===-- ubsan_loop_detect.cpp ---------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 2
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 3
````cpp
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
````
- **EN**: Comment documenting `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
- **CN**: 注释说明了 `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。

### Line 4
````cpp
// See https://llvm.org/LICENSE.txt for license information.
````
- **EN**: Comment documenting `See https://llvm.org/LICENSE.txt for license information.`.
- **CN**: 注释说明了 `See https://llvm.org/LICENSE.txt for license information.`。

### Line 5
````cpp
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
````
- **EN**: Comment documenting `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
- **CN**: 注释说明了 `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。

### Line 6
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 7
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 8
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 9
````cpp
// Runtime support for -fsanitize-trap-loop.
````
- **EN**: Comment documenting `Runtime support for -fsanitize-trap-loop.`.
- **CN**: 注释说明了 `Runtime support for -fsanitize-trap-loop.`。

### Line 10
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 11
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 12
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 13
````cpp
#include <sanitizer/ubsan_interface.h>
````
- **EN**: Includes the system dependency `sanitizer/ubsan_interface.h`.
- **CN**: 引入系统依赖 `sanitizer/ubsan_interface.h`。

### Line 14
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 15
````cpp
#if defined(__linux__) && (defined(__i386__) || defined(__x86_64__))
````
- **EN**: Starts a preprocessor condition: `#if defined(__linux__) && (defined(__i386__) || defined(__x86_64__))`.
- **CN**: 开始一个预处理条件：`#if defined(__linux__) && (defined(__i386__) || defined(__x86_64__))`。

### Line 16
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 17
````cpp
#include <asm/processor-flags.h>
````
- **EN**: Includes the system dependency `asm/processor-flags.h`.
- **CN**: 引入系统依赖 `asm/processor-flags.h`。

### Line 18
````cpp
#include <signal.h>
````
- **EN**: Includes the system dependency `signal.h`.
- **CN**: 引入系统依赖 `signal.h`。

### Line 19
````cpp
#include <stdint.h>
````
- **EN**: Includes the system dependency `stdint.h`.
- **CN**: 引入系统依赖 `stdint.h`。

### Line 20
````cpp
#include <sys/time.h>
````
- **EN**: Includes the system dependency `sys/time.h`.
- **CN**: 引入系统依赖 `sys/time.h`。

### Line 21
````cpp
#include <sys/ucontext.h>
````
- **EN**: Includes the system dependency `sys/ucontext.h`.
- **CN**: 引入系统依赖 `sys/ucontext.h`。

### Line 22
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 23
````cpp
int __ubsan_is_trap_loop(void *c) {
````
- **EN**: Begins a function or method definition: `int __ubsan_is_trap_loop(void *c) {`.
- **CN**: 开始一个函数或方法定义：`int __ubsan_is_trap_loop(void *c) {`。

### Line 24
````cpp
  auto *uc = reinterpret_cast<ucontext_t *>(c);
````
- **EN**: Invokes a function-like statement: `auto *uc = reinterpret_cast<ucontext_t *>(c);`.
- **CN**: 调用一个类似函数的语句：`auto *uc = reinterpret_cast<ucontext_t *>(c);`。

### Line 25
````cpp
#if defined(__x86_64__)
````
- **EN**: Starts a preprocessor condition: `#if defined(__x86_64__)`.
- **CN**: 开始一个预处理条件：`#if defined(__x86_64__)`。

### Line 26
````cpp
  auto *ip = reinterpret_cast<const uint8_t *>(uc->uc_mcontext.gregs[REG_RIP]);
````
- **EN**: Invokes a function-like statement: `auto *ip = reinterpret_cast<const uint8_t *>(uc->uc_mcontext.gregs[REG_RIP]);`.
- **CN**: 调用一个类似函数的语句：`auto *ip = reinterpret_cast<const uint8_t *>(uc->uc_mcontext.gregs[REG_RIP]);`。

### Line 27
````cpp
#else
````
- **EN**: Starts the fallback branch of the current preprocessor condition.
- **CN**: 开始当前预处理条件的后备分支。

### Line 28
````cpp
  auto *ip = reinterpret_cast<const uint8_t *>(uc->uc_mcontext.gregs[REG_EIP]);
````
- **EN**: Invokes a function-like statement: `auto *ip = reinterpret_cast<const uint8_t *>(uc->uc_mcontext.gregs[REG_EIP]);`.
- **CN**: 调用一个类似函数的语句：`auto *ip = reinterpret_cast<const uint8_t *>(uc->uc_mcontext.gregs[REG_EIP]);`。

### Line 29
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 30
````cpp
  // Test whether IP is at a conditional branch to self instruction.
````
- **EN**: Comment documenting `Test whether IP is at a conditional branch to self instruction.`.
- **CN**: 注释说明了 `Test whether IP is at a conditional branch to self instruction.`。

### Line 31
````cpp
  if ((ip[0] & 0xf0) != 0x70 || ip[1] != 0xfe)
````
- **EN**: Evaluates the conditional branch `if ((ip[0] & 0xf0) != 0x70 || ip[1] != 0xfe)`.
- **CN**: 计算条件分支 `if ((ip[0] & 0xf0) != 0x70 || ip[1] != 0xfe)`。

### Line 32
````cpp
    return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 33
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 34
````cpp
  // If so, test whether the condition is satisfied, in case we happened to
````
- **EN**: Comment documenting `If so, test whether the condition is satisfied, in case we happened to`.
- **CN**: 注释说明了 `If so, test whether the condition is satisfied, in case we happened to`。

### Line 35
````cpp
  // receive the signal at a not-taken branch to self.
````
- **EN**: Comment documenting `receive the signal at a not-taken branch to self.`.
- **CN**: 注释说明了 `receive the signal at a not-taken branch to self.`。

### Line 36
````cpp
  uint64_t eflags = uc->uc_mcontext.gregs[REG_EFL];
````
- **EN**: Assigns or initializes state with `uint64_t eflags = uc->uc_mcontext.gregs[REG_EFL];`.
- **CN**: 使用 `uint64_t eflags = uc->uc_mcontext.gregs[REG_EFL];` 进行赋值或初始化。

### Line 37
````cpp
  switch (ip[0]) {
````
- **EN**: Starts a `switch` dispatch: `switch (ip[0]) {`.
- **CN**: 开始一个 `switch` 分派：`switch (ip[0]) {`。

### Line 38
````cpp
  case 0x70: // JO
````
- **EN**: Marks a `switch` branch: `case 0x70: // JO`.
- **CN**: 标记一个 `switch` 分支：`case 0x70: // JO`。

### Line 39
````cpp
    return eflags & X86_EFLAGS_OF;
````
- **EN**: Returns from the current function with `eflags & X86_EFLAGS_OF;`.
- **CN**: 使用 `eflags & X86_EFLAGS_OF;` 从当前函数返回。

### Line 40
````cpp
  case 0x71: // JNO
````
- **EN**: Marks a `switch` branch: `case 0x71: // JNO`.
- **CN**: 标记一个 `switch` 分支：`case 0x71: // JNO`。

### Line 41
````cpp
    return !(eflags & X86_EFLAGS_OF);
````
- **EN**: Returns from the current function with `!(eflags & X86_EFLAGS_OF);`.
- **CN**: 使用 `!(eflags & X86_EFLAGS_OF);` 从当前函数返回。

### Line 42
````cpp
  case 0x72: // JB
````
- **EN**: Marks a `switch` branch: `case 0x72: // JB`.
- **CN**: 标记一个 `switch` 分支：`case 0x72: // JB`。

### Line 43
````cpp
    return eflags & X86_EFLAGS_CF;
````
- **EN**: Returns from the current function with `eflags & X86_EFLAGS_CF;`.
- **CN**: 使用 `eflags & X86_EFLAGS_CF;` 从当前函数返回。

### Line 44
````cpp
  case 0x73: // JAE
````
- **EN**: Marks a `switch` branch: `case 0x73: // JAE`.
- **CN**: 标记一个 `switch` 分支：`case 0x73: // JAE`。

### Line 45
````cpp
    return !(eflags & X86_EFLAGS_CF);
````
- **EN**: Returns from the current function with `!(eflags & X86_EFLAGS_CF);`.
- **CN**: 使用 `!(eflags & X86_EFLAGS_CF);` 从当前函数返回。

### Line 46
````cpp
  case 0x74: // JE
````
- **EN**: Marks a `switch` branch: `case 0x74: // JE`.
- **CN**: 标记一个 `switch` 分支：`case 0x74: // JE`。

### Line 47
````cpp
    return eflags & X86_EFLAGS_ZF;
````
- **EN**: Returns from the current function with `eflags & X86_EFLAGS_ZF;`.
- **CN**: 使用 `eflags & X86_EFLAGS_ZF;` 从当前函数返回。

### Line 48
````cpp
  case 0x75: // JNE
````
- **EN**: Marks a `switch` branch: `case 0x75: // JNE`.
- **CN**: 标记一个 `switch` 分支：`case 0x75: // JNE`。

### Line 49
````cpp
    return !(eflags & X86_EFLAGS_ZF);
````
- **EN**: Returns from the current function with `!(eflags & X86_EFLAGS_ZF);`.
- **CN**: 使用 `!(eflags & X86_EFLAGS_ZF);` 从当前函数返回。

### Line 50
````cpp
  case 0x76: // JBE
````
- **EN**: Marks a `switch` branch: `case 0x76: // JBE`.
- **CN**: 标记一个 `switch` 分支：`case 0x76: // JBE`。

### Line 51
````cpp
    return (eflags & X86_EFLAGS_CF) || (eflags & X86_EFLAGS_ZF);
````
- **EN**: Returns from the current function with `(eflags & X86_EFLAGS_CF) || (eflags & X86_EFLAGS_ZF);`.
- **CN**: 使用 `(eflags & X86_EFLAGS_CF) || (eflags & X86_EFLAGS_ZF);` 从当前函数返回。

### Line 52
````cpp
  case 0x77: // JA
````
- **EN**: Marks a `switch` branch: `case 0x77: // JA`.
- **CN**: 标记一个 `switch` 分支：`case 0x77: // JA`。

### Line 53
````cpp
    return !(eflags & X86_EFLAGS_CF) && !(eflags & X86_EFLAGS_ZF);
````
- **EN**: Returns from the current function with `!(eflags & X86_EFLAGS_CF) && !(eflags & X86_EFLAGS_ZF);`.
- **CN**: 使用 `!(eflags & X86_EFLAGS_CF) && !(eflags & X86_EFLAGS_ZF);` 从当前函数返回。

### Line 54
````cpp
  case 0x78: // JS
````
- **EN**: Marks a `switch` branch: `case 0x78: // JS`.
- **CN**: 标记一个 `switch` 分支：`case 0x78: // JS`。

### Line 55
````cpp
    return eflags & X86_EFLAGS_SF;
````
- **EN**: Returns from the current function with `eflags & X86_EFLAGS_SF;`.
- **CN**: 使用 `eflags & X86_EFLAGS_SF;` 从当前函数返回。

### Line 56
````cpp
  case 0x79: // JNS
````
- **EN**: Marks a `switch` branch: `case 0x79: // JNS`.
- **CN**: 标记一个 `switch` 分支：`case 0x79: // JNS`。

### Line 57
````cpp
    return !(eflags & X86_EFLAGS_SF);
````
- **EN**: Returns from the current function with `!(eflags & X86_EFLAGS_SF);`.
- **CN**: 使用 `!(eflags & X86_EFLAGS_SF);` 从当前函数返回。

### Line 58
````cpp
  case 0x7A: // JP
````
- **EN**: Marks a `switch` branch: `case 0x7A: // JP`.
- **CN**: 标记一个 `switch` 分支：`case 0x7A: // JP`。

### Line 59
````cpp
    return eflags & X86_EFLAGS_PF;
````
- **EN**: Returns from the current function with `eflags & X86_EFLAGS_PF;`.
- **CN**: 使用 `eflags & X86_EFLAGS_PF;` 从当前函数返回。

### Line 60
````cpp
  case 0x7B: // JNP
````
- **EN**: Marks a `switch` branch: `case 0x7B: // JNP`.
- **CN**: 标记一个 `switch` 分支：`case 0x7B: // JNP`。

### Line 61
````cpp
    return !(eflags & X86_EFLAGS_PF);
````
- **EN**: Returns from the current function with `!(eflags & X86_EFLAGS_PF);`.
- **CN**: 使用 `!(eflags & X86_EFLAGS_PF);` 从当前函数返回。

### Line 62
````cpp
  case 0x7C: // JL
````
- **EN**: Marks a `switch` branch: `case 0x7C: // JL`.
- **CN**: 标记一个 `switch` 分支：`case 0x7C: // JL`。

### Line 63
````cpp
    return !!(eflags & X86_EFLAGS_SF) != !!(eflags & X86_EFLAGS_OF);
````
- **EN**: Returns from the current function with `!!(eflags & X86_EFLAGS_SF) != !!(eflags & X86_EFLAGS_OF);`.
- **CN**: 使用 `!!(eflags & X86_EFLAGS_SF) != !!(eflags & X86_EFLAGS_OF);` 从当前函数返回。

### Line 64
````cpp
  case 0x7D: // JGE
````
- **EN**: Marks a `switch` branch: `case 0x7D: // JGE`.
- **CN**: 标记一个 `switch` 分支：`case 0x7D: // JGE`。

### Line 65
````cpp
    return !!(eflags & X86_EFLAGS_SF) == !!(eflags & X86_EFLAGS_OF);
````
- **EN**: Returns from the current function with `!!(eflags & X86_EFLAGS_SF) == !!(eflags & X86_EFLAGS_OF);`.
- **CN**: 使用 `!!(eflags & X86_EFLAGS_SF) == !!(eflags & X86_EFLAGS_OF);` 从当前函数返回。

### Line 66
````cpp
  case 0x7E: // JLE
````
- **EN**: Marks a `switch` branch: `case 0x7E: // JLE`.
- **CN**: 标记一个 `switch` 分支：`case 0x7E: // JLE`。

### Line 67
````cpp
    return (eflags & X86_EFLAGS_ZF) ||
````
- **EN**: Returns from the current function with `(eflags & X86_EFLAGS_ZF) ||`.
- **CN**: 使用 `(eflags & X86_EFLAGS_ZF) ||` 从当前函数返回。

### Line 68
````cpp
           !!(eflags & X86_EFLAGS_SF) != !!(eflags & X86_EFLAGS_OF);
````
- **EN**: Invokes a function-like statement: `!!(eflags & X86_EFLAGS_SF) != !!(eflags & X86_EFLAGS_OF);`.
- **CN**: 调用一个类似函数的语句：`!!(eflags & X86_EFLAGS_SF) != !!(eflags & X86_EFLAGS_OF);`。

### Line 69
````cpp
  case 0x7F: // JG
````
- **EN**: Marks a `switch` branch: `case 0x7F: // JG`.
- **CN**: 标记一个 `switch` 分支：`case 0x7F: // JG`。

### Line 70
````cpp
    return !(eflags & X86_EFLAGS_ZF) &&
````
- **EN**: Returns from the current function with `!(eflags & X86_EFLAGS_ZF) &&`.
- **CN**: 使用 `!(eflags & X86_EFLAGS_ZF) &&` 从当前函数返回。

### Line 71
````cpp
           !!(eflags & X86_EFLAGS_SF) == !!(eflags & X86_EFLAGS_OF);
````
- **EN**: Invokes a function-like statement: `!!(eflags & X86_EFLAGS_SF) == !!(eflags & X86_EFLAGS_OF);`.
- **CN**: 调用一个类似函数的语句：`!!(eflags & X86_EFLAGS_SF) == !!(eflags & X86_EFLAGS_OF);`。

### Line 72
````cpp
  default:
````
- **EN**: Marks a `switch` branch: `default:`.
- **CN**: 标记一个 `switch` 分支：`default:`。

### Line 73
````cpp
    return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 74
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 75
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 76
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 77
````cpp
static void SigprofHandler(int signo, siginfo_t *si, void *c) {
````
- **EN**: Begins a function or method definition: `static void SigprofHandler(int signo, siginfo_t *si, void *c) {`.
- **CN**: 开始一个函数或方法定义：`static void SigprofHandler(int signo, siginfo_t *si, void *c) {`。

### Line 78
````cpp
  if (__ubsan_is_trap_loop(c)) {
````
- **EN**: Evaluates the conditional branch `if (__ubsan_is_trap_loop(c)) {`.
- **CN**: 计算条件分支 `if (__ubsan_is_trap_loop(c)) {`。

### Line 79
````cpp
    __builtin_trap();
````
- **EN**: Invokes a function-like statement: `__builtin_trap();`.
- **CN**: 调用一个类似函数的语句：`__builtin_trap();`。

### Line 80
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 81
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 82
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 83
````cpp
void __ubsan_install_trap_loop_detection(void) {
````
- **EN**: Begins a function or method definition: `void __ubsan_install_trap_loop_detection(void) {`.
- **CN**: 开始一个函数或方法定义：`void __ubsan_install_trap_loop_detection(void) {`。

### Line 84
````cpp
  struct sigaction sa;
````
- **EN**: Declares the struct `sigaction`.
- **CN**: 声明 struct `sigaction`。

### Line 85
````cpp
  sa.sa_sigaction = SigprofHandler;
````
- **EN**: Assigns or initializes state with `sa.sa_sigaction = SigprofHandler;`.
- **CN**: 使用 `sa.sa_sigaction = SigprofHandler;` 进行赋值或初始化。

### Line 86
````cpp
  sigaction(SIGPROF, &sa, nullptr);
````
- **EN**: Declares an interface element or prototype: `sigaction(SIGPROF, &sa, nullptr);`.
- **CN**: 声明一个接口元素或原型：`sigaction(SIGPROF, &sa, nullptr);`。

### Line 87
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 88
````cpp
  struct itimerval timer;
````
- **EN**: Declares the struct `itimerval`.
- **CN**: 声明 struct `itimerval`。

### Line 89
````cpp
  timer.it_value.tv_sec = 0;
````
- **EN**: Assigns or initializes state with `timer.it_value.tv_sec = 0;`.
- **CN**: 使用 `timer.it_value.tv_sec = 0;` 进行赋值或初始化。

### Line 90
````cpp
  timer.it_value.tv_usec = 100000;
````
- **EN**: Assigns or initializes state with `timer.it_value.tv_usec = 100000;`.
- **CN**: 使用 `timer.it_value.tv_usec = 100000;` 进行赋值或初始化。

### Line 91
````cpp
  timer.it_interval = timer.it_value;
````
- **EN**: Assigns or initializes state with `timer.it_interval = timer.it_value;`.
- **CN**: 使用 `timer.it_interval = timer.it_value;` 进行赋值或初始化。

### Line 92
````cpp
  setitimer(ITIMER_PROF, &timer, NULL);
````
- **EN**: Declares an interface element or prototype: `setitimer(ITIMER_PROF, &timer, NULL);`.
- **CN**: 声明一个接口元素或原型：`setitimer(ITIMER_PROF, &timer, NULL);`。

### Line 93
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 94
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 95
````cpp
#else
````
- **EN**: Starts the fallback branch of the current preprocessor condition.
- **CN**: 开始当前预处理条件的后备分支。

### Line 96
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 97
````cpp
int __ubsan_is_trap_loop(void *c) { return false; }
````
- **EN**: Carries part of the local implementation logic: `int __ubsan_is_trap_loop(void *c) { return false; }`.
- **CN**: 承载局部实现逻辑：`int __ubsan_is_trap_loop(void *c) { return false; }`。

### Line 98
````cpp
void __ubsan_install_trap_loop_detection(void) {}
````
- **EN**: Carries part of the local implementation logic: `void __ubsan_install_trap_loop_detection(void) {}`.
- **CN**: 承载局部实现逻辑：`void __ubsan_install_trap_loop_detection(void) {}`。

### Line 99
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 100
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

## Key Concepts / 关键概念
- Preprocessor control / 预处理控制
- Types and interfaces / 类型与接口
- Function logic / 函数逻辑

## Dependencies / 依赖关系
- **System headers / 系统头文件**: `sanitizer/ubsan_interface.h`, `asm/processor-flags.h`, `signal.h`, `stdint.h`, `sys/time.h`, `sys/ucontext.h`
- **Compile-time conditions / 编译期条件**:
  - `#if defined(__linux__) && (defined(__i386__) || defined(__x86_64__))`
  - `#if defined(__x86_64__)`
