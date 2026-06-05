# hugify.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/runtime/hugify.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/runtime/hugify.cpp. It also sits in code that belongs to the BOLT post-link optimizer subsystem. / 该文件属于 BOLT 链接后优化器子系统。 源码头部说明其职责是：bolt/runtime/hugify.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/runtime/hugify.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===---------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-18

```cpp
#if (defined(__x86_64__) || defined(__aarch64__) || defined(__arm64__)) &&     \
    !defined(__APPLE__)

#include "common.h"

#pragma GCC visibility push(hidden)

// Enables a very verbose logging to stderr useful when debugging
// #define ENABLE_DEBUG
```

- EN: Pulls in 1 header(s) from system dependencies needed by this range. Uses conditional compilation to choose platform- or feature-specific code paths. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 1 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里使用条件编译，根据平台或特性切换不同代码路径。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-26

```cpp
#ifdef ENABLE_DEBUG
#define DEBUG(X)                                                               \
  { X; }
#else
#define DEBUG(X)                                                               \
  {}
#endif
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `DEBUG` for constants or compile-time switches. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `DEBUG`，用于常量或编译期开关。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 27-35

```cpp
// Function constrains trampoline to _start,
// so we can resume regular execution of the function that we hooked.
extern void __bolt_hugify_start_program();

// The __hot_start and __hot_end symbols set by Bolt. We use them to figure
// out the rage for marking huge pages.
extern uint64_t __hot_start;
extern uint64_t __hot_end;
```

- EN: Declares or implements routines including `__bolt_hugify_start_program`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `__bolt_hugify_start_program`.
- CN: 这里声明或实现函数，例如 `__bolt_hugify_start_program`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `__bolt_hugify_start_program`。

### Lines 36-44

```cpp
static void getKernelVersion(uint32_t *Val) {
  // release should be in the format: %d.%d.%d
  // major, minor, release
  struct UtsNameTy UtsName;
  int Ret = __uname(&UtsName);
  const char *Buf = UtsName.release;
  const char *End = Buf + strLen(Buf);
  const char Delims[2][2] = {".", "."};
```

- EN: Introduces type definitions such as `UtsNameTy`. Declares or implements routines including `getKernelVersion`, `__uname`, `strLen`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `UtsNameTy`, `getKernelVersion`, `__uname`, `strLen`.
- CN: 这里引入类型定义，例如 `UtsNameTy`。这里声明或实现函数，例如 `getKernelVersion`, `__uname`, `strLen`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `UtsNameTy`, `getKernelVersion`, `__uname`, `strLen`。

### Lines 45-61

```cpp
  for (int i = 0; i < 3; ++i) {
    if (!scanUInt32(Buf, End, Val[i])) {
      return;
    }
    if (i < sizeof(Delims) / sizeof(Delims[0])) {
      const char *Ptr = Delims[i];
      while (*Ptr != '\0') {
        if (*Ptr != *Buf) {
          return;
        }
        ++Ptr;
        ++Buf;
      }
    }
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 62-71

```cpp
/// Check whether the kernel supports THP via corresponding sysfs entry.
/// thp works only starting from 5.10
static bool hasPagecacheTHPSupport() {
  char Buf[64];

  int FD = __open("/sys/kernel/mm/transparent_hugepage/enabled",
                  0 /* O_RDONLY */, 0);
  if (FD < 0)
    return false;
```

- EN: Declares or implements routines including `hasPagecacheTHPSupport`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `hasPagecacheTHPSupport`.
- CN: 这里声明或实现函数，例如 `hasPagecacheTHPSupport`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `hasPagecacheTHPSupport`。

### Lines 72-81

```cpp
  memset(Buf, 0, sizeof(Buf));
  const size_t Res = __read(FD, Buf, sizeof(Buf));
  if (Res < 0)
    return false;

  if (!strStr(Buf, "[always]") && !strStr(Buf, "[madvise]")) {
    DEBUG(report("[hugify] THP support is not enabled.\n");)
    return false;
  }
```

- EN: Declares or implements routines including `memset`, `__read`, `DEBUG`. Notable symbols here include `memset`, `__read`, `DEBUG`.
- CN: 这里声明或实现函数，例如 `memset`, `__read`, `DEBUG`。这里较值得关注的符号包括 `memset`, `__read`, `DEBUG`。

### Lines 82-89

```cpp
  struct KernelVersionTy {
    uint32_t major;
    uint32_t minor;
    uint32_t release;
  };

  KernelVersionTy KernelVersion;
```

- EN: Introduces type definitions such as `KernelVersionTy`. Notable symbols here include `KernelVersionTy`.
- CN: 这里引入类型定义，例如 `KernelVersionTy`。这里较值得关注的符号包括 `KernelVersionTy`。

### Lines 90-97

```cpp
  getKernelVersion((uint32_t *)&KernelVersion);
  if (KernelVersion.major >= 6 ||
      (KernelVersion.major == 5 && KernelVersion.minor >= 10))
    return true;

  return false;
}
```

- EN: Declares or implements routines including `getKernelVersion`. Notable symbols here include `getKernelVersion`.
- CN: 这里声明或实现函数，例如 `getKernelVersion`。这里较值得关注的符号包括 `getKernelVersion`。

### Lines 98-109

```cpp
static void hugifyForOldKernel(uint8_t *From, uint8_t *To) {
  const size_t Size = To - From;

  uint8_t *Mem = reinterpret_cast<uint8_t *>(
      __mmap(0, Size, 0x3 /* PROT_READ | PROT_WRITE */,
             0x22 /* MAP_PRIVATE | MAP_ANONYMOUS */, -1, 0));

  if (Mem == ((void *)-1) /* MAP_FAILED */) {
    char Msg[] = "[hugify] could not allocate memory for text move\n";
    reportError(Msg, sizeof(Msg));
  }
```

- EN: Declares or implements routines including `hugifyForOldKernel`, `reportError`. Notable symbols here include `hugifyForOldKernel`, `reportError`.
- CN: 这里声明或实现函数，例如 `hugifyForOldKernel`, `reportError`。这里较值得关注的符号包括 `hugifyForOldKernel`, `reportError`。

### Lines 110-127

```cpp
  DEBUG(reportNumber("[hugify] allocated temporary address: ", (uint64_t)Mem,
                     16);)
  DEBUG(reportNumber("[hugify] allocated size: ", (uint64_t)Size, 16);)

  // Copy the hot code to a temporary location.
  memcpy(Mem, From, Size);

  __prctl(41 /* PR_SET_THP_DISABLE */, 0, 0, 0, 0);
  // Maps out the existing hot code.
  if (__mmap(reinterpret_cast<uint64_t>(From), Size,
             0x3 /* PROT_READ | PROT_WRITE */,
             0x32 /* MAP_FIXED | MAP_ANONYMOUS | MAP_PRIVATE */, -1,
             0) == ((void *)-1) /*MAP_FAILED*/) {
    char Msg[] =
        "[hugify] failed to mmap memory for large page move terminating\n";
    reportError(Msg, sizeof(Msg));
  }
```

- EN: Declares or implements routines including `DEBUG`, `memcpy`, `__prctl`, `reportError`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `DEBUG`, `memcpy`, `__prctl`, `reportError`.
- CN: 这里声明或实现函数，例如 `DEBUG`, `memcpy`, `__prctl`, `reportError`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `DEBUG`, `memcpy`, `__prctl`, `reportError`。

### Lines 128-136

```cpp
  // Mark the hot code page to be huge page.
  if (__madvise(From, Size, 14 /* MADV_HUGEPAGE */) == -1) {
    char Msg[] = "[hugify] setting MADV_HUGEPAGE is failed\n";
    reportError(Msg, sizeof(Msg));
  }

  // Copy the hot code back.
  memcpy(From, Mem, Size);
```

- EN: Declares or implements routines including `reportError`, `memcpy`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `reportError`, `memcpy`.
- CN: 这里声明或实现函数，例如 `reportError`, `memcpy`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `reportError`, `memcpy`。

### Lines 137-151

```cpp
  // Change permission back to read-only, ignore failure
  __mprotect(From, Size, 0x5 /* PROT_READ | PROT_EXEC */);

  __munmap(Mem, Size);
}

extern "C" void __bolt_hugify_self_impl() {
  uint8_t *HotStart = (uint8_t *)&__hot_start;
  uint8_t *HotEnd = (uint8_t *)&__hot_end;
  // Make sure the start and end are aligned with huge page address
  const size_t HugePageBytes = 2L * 1024 * 1024;
  uint8_t *From = HotStart - ((intptr_t)HotStart & (HugePageBytes - 1));
  uint8_t *To = HotEnd + (HugePageBytes - 1);
  To -= (intptr_t)To & (HugePageBytes - 1);
```

- EN: Declares or implements routines including `__mprotect`, `__munmap`, `__bolt_hugify_self_impl`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `__mprotect`, `__munmap`, `__bolt_hugify_self_impl`.
- CN: 这里声明或实现函数，例如 `__mprotect`, `__munmap`, `__bolt_hugify_self_impl`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `__mprotect`, `__munmap`, `__bolt_hugify_self_impl`。

### Lines 152-163

```cpp
  DEBUG(reportNumber("[hugify] hot start: ", (uint64_t)HotStart, 16);)
  DEBUG(reportNumber("[hugify] hot end: ", (uint64_t)HotEnd, 16);)
  DEBUG(reportNumber("[hugify] aligned huge page from: ", (uint64_t)From, 16);)
  DEBUG(reportNumber("[hugify] aligned huge page to: ", (uint64_t)To, 16);)

  if (!hasPagecacheTHPSupport()) {
    DEBUG(report(
              "[hugify] workaround with memory alignment for kernel < 5.10\n");)
    hugifyForOldKernel(From, To);
    return;
  }
```

- EN: Declares or implements routines including `DEBUG`, `hugifyForOldKernel`. Notable symbols here include `DEBUG`, `hugifyForOldKernel`.
- CN: 这里声明或实现函数，例如 `DEBUG`, `hugifyForOldKernel`。这里较值得关注的符号包括 `DEBUG`, `hugifyForOldKernel`。

### Lines 164-181

```cpp
  if (__madvise(From, (To - From), 14 /* MADV_HUGEPAGE */) == -1) {
    char Msg[] = "[hugify] failed to allocate large page\n";
    // TODO: allow user to control the failure behavior.
    reportError(Msg, sizeof(Msg));
  }
}

/// This is hooking ELF's entry, it needs to save all machine state.
extern "C" __attribute((naked)) void __bolt_hugify_self() {
  // clang-format off
#if defined(__x86_64__)
  __asm__ __volatile__(SAVE_ALL "call __bolt_hugify_self_impl\n" RESTORE_ALL
                                "jmp __bolt_hugify_start_program\n"
                                :::);
#elif defined(__aarch64__) || defined(__arm64__)
  __asm__ __volatile__(SAVE_ALL "bl __bolt_hugify_self_impl\n" RESTORE_ALL
                                "adrp x16, __bolt_hugify_start_program\n"
                                "add x16, x16, #:lo12:__bolt_hugify_start_program\n"
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Declares or implements routines including `reportError`, `__attribute`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `reportError`, `__attribute`.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里声明或实现函数，例如 `reportError`, `__attribute`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `reportError`, `__attribute`。

### Lines 182-189

```cpp
                                "br x16\n"
                                :::);
#else
  __exit(1);
#endif
  // clang-format on
}
#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Declares or implements routines including `__exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `__exit`.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里声明或实现函数，例如 `__exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `__exit`。

## Key Concepts / 关键概念

- `UtsNameTy`: class or struct interface / 类或结构体接口
- `KernelVersionTy`: class or struct interface / 类或结构体接口
- `__bolt_hugify_start_program`: function or method entry point / 函数或方法入口
- `getKernelVersion`: function or method entry point / 函数或方法入口
- `__uname`: function or method entry point / 函数或方法入口
- `strLen`: function or method entry point / 函数或方法入口
- `hasPagecacheTHPSupport`: function or method entry point / 函数或方法入口
- `DEBUG`: macro or compile-time switch / 宏或编译期开关

## Dependencies / 依赖关系

- System headers / 系统头文件: `common.h`
- Directory context / 目录上下文: `bolt/runtime` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/runtime` 下的相邻文件通常与本文件协作组成对应子系统
