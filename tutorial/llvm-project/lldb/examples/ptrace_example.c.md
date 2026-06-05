# ptrace_example.c — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/examples/ptrace_example.c`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `ptrace_example`.
  - **CN**: 实现与 `ptrace_example` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```c
//===-- ptrace_example.c --------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-17
```c

#include <asm/ptrace.h>
#include <linux/elf.h>
#include <stdint.h>
#include <stdio.h>
#include <sys/prctl.h>
#include <sys/ptrace.h>
#include <sys/uio.h>
#include <sys/wait.h>
#include <unistd.h>
```
- **EN**: Pulls in the headers needed by this translation unit, including `asm/ptrace.h`, `linux/elf.h`, `stdint.h`, `stdio.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `asm/ptrace.h`, `linux/elf.h`, `stdint.h`, `stdio.h`。

### Lines 18-27
```c

// The demo program shows how to do basic ptrace operations without lldb
// or lldb-server. For the purposes of experimentation or reporting bugs
// in kernels.
//
// It is AArch64 Linux specific, adapt as needed.
//
// Expected output:
// Before breakpoint
// After breakpoint
```
- **EN**: Documents the surrounding declarations, example workflow, or design constraints for this LLDB component.
- **CN**: 说明周边声明、示例流程，或记录该 LLDB 组件的设计约束。

### Lines 28-34
```c

void inferior() {
  if (ptrace(PTRACE_TRACEME, 0, 0, 0) < 0) {
    perror("ptrace");
    return;
  }

```
- **EN**: Implements logic around `inferior`, `ptrace`, `perror`.
- **CN**: 围绕 `inferior`, `ptrace`, `perror` 实现具体逻辑。

### Lines 35-42
```c
  printf("Before breakpoint\n");

  // Go into debugger. Instruction replaced with nop later.
  // We write 2 instuctions because POKETEXT works with
  // 64 bit values and we don't want to overwrite the
  // call to printf accidentally.
  asm volatile("BRK #0 \n nop");

```
- **EN**: Implements logic around `printf`, `volatile`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `printf`, `volatile` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 43-50
```c
  printf("After breakpoint\n");
}

void debugger(pid_t child) {
  int wait_status;
  // Wait until it hits the breakpoint.
  wait(&wait_status);

```
- **EN**: Implements logic around `printf`, `debugger`, `wait`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `printf`, `debugger`, `wait` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 51-56
```c
  while (WIFSTOPPED(wait_status)) {
    if (WIFEXITED(wait_status)) {
      printf("inferior exited normally\n");
      return;
    }

```
- **EN**: Implements logic around `WIFSTOPPED`, `WIFEXITED`, `printf`.
- **CN**: 围绕 `WIFSTOPPED`, `WIFEXITED`, `printf` 实现具体逻辑。

### Lines 57-66
```c
    // Read general purpose registers to find the PC value.
    struct user_pt_regs regs;
    struct iovec io;
    io.iov_base = &regs;
    io.iov_len = sizeof(regs);
    if (ptrace(PTRACE_GETREGSET, child, NT_PRSTATUS, &io) < 0) {
      printf("getregset failed\n");
      return;
    }

```
- **EN**: Introduces declarations for `user_pt_regs`, `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `user_pt_regs`, `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 67-74
```c
    // Replace brk #0 / nop with nop / nop by writing to memory
    // at the current PC.
    uint64_t replace = 0xd503201fd503201f;
    if (ptrace(PTRACE_POKETEXT, child, regs.pc, replace) < 0) {
      printf("replacing bkpt failed\n");
      return;
    }

```
- **EN**: Implements logic around `ptrace`, `printf`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ptrace`, `printf` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 75-80
```c
    // Single step over where the brk was.
    if (ptrace(PTRACE_SINGLESTEP, child, 0, 0) < 0) {
      perror("ptrace");
      return;
    }

```
- **EN**: Implements logic around `ptrace`, `perror`.
- **CN**: 围绕 `ptrace`, `perror` 实现具体逻辑。

### Lines 81-89
```c
    // Wait for single step to be done.
    wait(&wait_status);

    // Run to completion.
    if (ptrace(PTRACE_CONT, child, 0, 0) < 0) {
      perror("ptrace");
      return;
    }

```
- **EN**: Implements logic around `wait`, `ptrace`, `perror`.
- **CN**: 围绕 `wait`, `ptrace`, `perror` 实现具体逻辑。

### Lines 90-94
```c
    // Wait to see that the inferior exited.
    wait(&wait_status);
  }
}

```
- **EN**: Implements logic around `wait`.
- **CN**: 围绕 `wait` 实现具体逻辑。

### Lines 95-104
```c
int main() {
  pid_t child = fork();

  if (child == 0)
    inferior();
  else if (child > 0)
    debugger(child);
  else
    return -1;

```
- **EN**: Implements logic around `main`, `fork`, `inferior`, `debugger`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `main`, `fork`, `inferior`, `debugger` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 105-106
```c
  return 0;
}
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Stop control / 停机控制**:
  - **EN**: Represents stop reasons, conditions, callbacks, and breakpoint resolution state.
  - **CN**: 表示停机原因、条件、回调以及断点解析状态。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **Example workflows / 示例工作流**:
  - **EN**: Provides runnable examples that teach extension points or debugger usage patterns.
  - **CN**: 提供可运行的示例，用于讲解扩展点或调试器使用模式。

## Dependencies / 依赖关系

- **Standard-library headers / 标准库头文件**: `<asm/ptrace.h>`, `<linux/elf.h>`, `<stdint.h>`, `<stdio.h>`, `<sys/prctl.h>`, `<sys/ptrace.h>`, `<sys/uio.h>`, `<sys/wait.h>` ... (+1 more)
