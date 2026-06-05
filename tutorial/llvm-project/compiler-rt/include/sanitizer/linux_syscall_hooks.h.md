# linux_syscall_hooks.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/include/sanitizer/linux_syscall_hooks.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This file is a part of public sanitizer interface.
  - **CN**: 声明 sanitizer 公开接口中与 `linux_syscall_hooks` 相关的 API、类型或常量。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行
```cpp
   1 | //===-- linux_syscall_hooks.h ---------------------------------------------===//
   2 | //
   3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4 | // See https://llvm.org/LICENSE.txt for license information.
   5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6 | //
   7 | //===----------------------------------------------------------------------===//
   8 | //
   9 | // This file is a part of public sanitizer interface.
  10 | //
```
- **Line 1 / 第 1 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2 / 第 2 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3 / 第 3 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4 / 第 4 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 5 / 第 5 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 6 / 第 6 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 7 / 第 7 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 8 / 第 8 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 9 / 第 9 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 10 / 第 10 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 11-20 / 第 11-20 行
```cpp
  11 | // System call handlers.
  12 | //
  13 | // Interface methods declared in this header implement pre- and post- syscall
  14 | // actions for the active sanitizer.
  15 | // Usage:
  16 | //   __sanitizer_syscall_pre_getfoo(...args...);
  17 | //   long res = syscall(__NR_getfoo, ...args...);
  18 | //   __sanitizer_syscall_post_getfoo(res, ...args...);
  19 | //===----------------------------------------------------------------------===//
  20 | #ifndef SANITIZER_LINUX_SYSCALL_HOOKS_H
```
- **Line 11 / 第 11 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 12 / 第 12 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 13 / 第 13 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 14 / 第 14 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 15 / 第 15 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 16 / 第 16 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 17 / 第 17 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 18 / 第 18 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 19 / 第 19 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 20 / 第 20 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。

### Lines 21-30 / 第 21-30 行
```cpp
  21 | #define SANITIZER_LINUX_SYSCALL_HOOKS_H
  22 | 
  23 | #define __sanitizer_syscall_pre_time(tloc)                                     \
  24 |   __sanitizer_syscall_pre_impl_time((long)(tloc))
  25 | #define __sanitizer_syscall_post_time(res, tloc)                               \
  26 |   __sanitizer_syscall_post_impl_time(res, (long)(tloc))
  27 | #define __sanitizer_syscall_pre_stime(tptr)                                    \
  28 |   __sanitizer_syscall_pre_impl_stime((long)(tptr))
  29 | #define __sanitizer_syscall_post_stime(res, tptr)                              \
  30 |   __sanitizer_syscall_post_impl_stime(res, (long)(tptr))
```
- **Line 21 / 第 21 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 22 / 第 22 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 23 / 第 23 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 24 / 第 24 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 25 / 第 25 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 26 / 第 26 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 27 / 第 27 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 28 / 第 28 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 29 / 第 29 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 30 / 第 30 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 31-40 / 第 31-40 行
```cpp
  31 | #define __sanitizer_syscall_pre_gettimeofday(tv, tz)                           \
  32 |   __sanitizer_syscall_pre_impl_gettimeofday((long)(tv), (long)(tz))
  33 | #define __sanitizer_syscall_post_gettimeofday(res, tv, tz)                     \
  34 |   __sanitizer_syscall_post_impl_gettimeofday(res, (long)(tv), (long)(tz))
  35 | #define __sanitizer_syscall_pre_settimeofday(tv, tz)                           \
  36 |   __sanitizer_syscall_pre_impl_settimeofday((long)(tv), (long)(tz))
  37 | #define __sanitizer_syscall_post_settimeofday(res, tv, tz)                     \
  38 |   __sanitizer_syscall_post_impl_settimeofday(res, (long)(tv), (long)(tz))
  39 | #define __sanitizer_syscall_pre_adjtimex(txc_p)                                \
  40 |   __sanitizer_syscall_pre_impl_adjtimex((long)(txc_p))
```
- **Line 31 / 第 31 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 32 / 第 32 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 33 / 第 33 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 34 / 第 34 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 35 / 第 35 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 36 / 第 36 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 37 / 第 37 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 38 / 第 38 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 39 / 第 39 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 40 / 第 40 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 41-50 / 第 41-50 行
```cpp
  41 | #define __sanitizer_syscall_post_adjtimex(res, txc_p)                          \
  42 |   __sanitizer_syscall_post_impl_adjtimex(res, (long)(txc_p))
  43 | #define __sanitizer_syscall_pre_times(tbuf)                                    \
  44 |   __sanitizer_syscall_pre_impl_times((long)(tbuf))
  45 | #define __sanitizer_syscall_post_times(res, tbuf)                              \
  46 |   __sanitizer_syscall_post_impl_times(res, (long)(tbuf))
  47 | #define __sanitizer_syscall_pre_gettid() __sanitizer_syscall_pre_impl_gettid()
  48 | #define __sanitizer_syscall_post_gettid(res)                                   \
  49 |   __sanitizer_syscall_post_impl_gettid(res)
  50 | #define __sanitizer_syscall_pre_nanosleep(rqtp, rmtp)                          \
```
- **Line 41 / 第 41 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 42 / 第 42 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 43 / 第 43 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 44 / 第 44 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 45 / 第 45 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 46 / 第 46 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 47 / 第 47 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 48 / 第 48 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 49 / 第 49 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 50 / 第 50 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 51-60 / 第 51-60 行
```cpp
  51 |   __sanitizer_syscall_pre_impl_nanosleep((long)(rqtp), (long)(rmtp))
  52 | #define __sanitizer_syscall_post_nanosleep(res, rqtp, rmtp)                    \
  53 |   __sanitizer_syscall_post_impl_nanosleep(res, (long)(rqtp), (long)(rmtp))
  54 | #define __sanitizer_syscall_pre_alarm(seconds)                                 \
  55 |   __sanitizer_syscall_pre_impl_alarm((long)(seconds))
  56 | #define __sanitizer_syscall_post_alarm(res, seconds)                           \
  57 |   __sanitizer_syscall_post_impl_alarm(res, (long)(seconds))
  58 | #define __sanitizer_syscall_pre_getpid() __sanitizer_syscall_pre_impl_getpid()
  59 | #define __sanitizer_syscall_post_getpid(res)                                   \
  60 |   __sanitizer_syscall_post_impl_getpid(res)
```
- **Line 51 / 第 51 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 52 / 第 52 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 53 / 第 53 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 54 / 第 54 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 55 / 第 55 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 56 / 第 56 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 57 / 第 57 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 58 / 第 58 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 59 / 第 59 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 60 / 第 60 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 61-70 / 第 61-70 行
```cpp
  61 | #define __sanitizer_syscall_pre_getppid() __sanitizer_syscall_pre_impl_getppid()
  62 | #define __sanitizer_syscall_post_getppid(res)                                  \
  63 |   __sanitizer_syscall_post_impl_getppid(res)
  64 | #define __sanitizer_syscall_pre_getuid() __sanitizer_syscall_pre_impl_getuid()
  65 | #define __sanitizer_syscall_post_getuid(res)                                   \
  66 |   __sanitizer_syscall_post_impl_getuid(res)
  67 | #define __sanitizer_syscall_pre_geteuid() __sanitizer_syscall_pre_impl_geteuid()
  68 | #define __sanitizer_syscall_post_geteuid(res)                                  \
  69 |   __sanitizer_syscall_post_impl_geteuid(res)
  70 | #define __sanitizer_syscall_pre_getgid() __sanitizer_syscall_pre_impl_getgid()
```
- **Line 61 / 第 61 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 62 / 第 62 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 63 / 第 63 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 64 / 第 64 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 65 / 第 65 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 66 / 第 66 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 67 / 第 67 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 68 / 第 68 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 69 / 第 69 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 70 / 第 70 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 71-80 / 第 71-80 行
```cpp
  71 | #define __sanitizer_syscall_post_getgid(res)                                   \
  72 |   __sanitizer_syscall_post_impl_getgid(res)
  73 | #define __sanitizer_syscall_pre_getegid() __sanitizer_syscall_pre_impl_getegid()
  74 | #define __sanitizer_syscall_post_getegid(res)                                  \
  75 |   __sanitizer_syscall_post_impl_getegid(res)
  76 | #define __sanitizer_syscall_pre_getresuid(ruid, euid, suid)                    \
  77 |   __sanitizer_syscall_pre_impl_getresuid((long)(ruid), (long)(euid),           \
  78 |                                          (long)(suid))
  79 | #define __sanitizer_syscall_post_getresuid(res, ruid, euid, suid)              \
  80 |   __sanitizer_syscall_post_impl_getresuid(res, (long)(ruid), (long)(euid),     \
```
- **Line 71 / 第 71 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 72 / 第 72 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 73 / 第 73 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 74 / 第 74 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 75 / 第 75 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 76 / 第 76 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 77 / 第 77 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 78 / 第 78 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 79 / 第 79 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 80 / 第 80 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 81-90 / 第 81-90 行
```cpp
  81 |                                           (long)(suid))
  82 | #define __sanitizer_syscall_pre_getresgid(rgid, egid, sgid)                    \
  83 |   __sanitizer_syscall_pre_impl_getresgid((long)(rgid), (long)(egid),           \
  84 |                                          (long)(sgid))
  85 | #define __sanitizer_syscall_post_getresgid(res, rgid, egid, sgid)              \
  86 |   __sanitizer_syscall_post_impl_getresgid(res, (long)(rgid), (long)(egid),     \
  87 |                                           (long)(sgid))
  88 | #define __sanitizer_syscall_pre_getpgid(pid)                                   \
  89 |   __sanitizer_syscall_pre_impl_getpgid((long)(pid))
  90 | #define __sanitizer_syscall_post_getpgid(res, pid)                             \
```
- **Line 81 / 第 81 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 82 / 第 82 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 83 / 第 83 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 84 / 第 84 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 85 / 第 85 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 86 / 第 86 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 87 / 第 87 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 88 / 第 88 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 89 / 第 89 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 90 / 第 90 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 91-100 / 第 91-100 行
```cpp
  91 |   __sanitizer_syscall_post_impl_getpgid(res, (long)(pid))
  92 | #define __sanitizer_syscall_pre_getpgrp() __sanitizer_syscall_pre_impl_getpgrp()
  93 | #define __sanitizer_syscall_post_getpgrp(res)                                  \
  94 |   __sanitizer_syscall_post_impl_getpgrp(res)
  95 | #define __sanitizer_syscall_pre_getsid(pid)                                    \
  96 |   __sanitizer_syscall_pre_impl_getsid((long)(pid))
  97 | #define __sanitizer_syscall_post_getsid(res, pid)                              \
  98 |   __sanitizer_syscall_post_impl_getsid(res, (long)(pid))
  99 | #define __sanitizer_syscall_pre_getgroups(gidsetsize, grouplist)               \
 100 |   __sanitizer_syscall_pre_impl_getgroups((long)(gidsetsize), (long)(grouplist))
```
- **Line 91 / 第 91 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 92 / 第 92 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 93 / 第 93 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 94 / 第 94 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 95 / 第 95 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 96 / 第 96 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 97 / 第 97 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 98 / 第 98 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 99 / 第 99 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 100 / 第 100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 101-110 / 第 101-110 行
```cpp
 101 | #define __sanitizer_syscall_post_getgroups(res, gidsetsize, grouplist)         \
 102 |   __sanitizer_syscall_post_impl_getgroups(res, (long)(gidsetsize),             \
 103 |                                           (long)(grouplist))
 104 | #define __sanitizer_syscall_pre_setregid(rgid, egid)                           \
 105 |   __sanitizer_syscall_pre_impl_setregid((long)(rgid), (long)(egid))
 106 | #define __sanitizer_syscall_post_setregid(res, rgid, egid)                     \
 107 |   __sanitizer_syscall_post_impl_setregid(res, (long)(rgid), (long)(egid))
 108 | #define __sanitizer_syscall_pre_setgid(gid)                                    \
 109 |   __sanitizer_syscall_pre_impl_setgid((long)(gid))
 110 | #define __sanitizer_syscall_post_setgid(res, gid)                              \
```
- **Line 101 / 第 101 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 102 / 第 102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 103 / 第 103 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 104 / 第 104 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 105 / 第 105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 106 / 第 106 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 107 / 第 107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 108 / 第 108 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 109 / 第 109 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 110 / 第 110 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 111-120 / 第 111-120 行
```cpp
 111 |   __sanitizer_syscall_post_impl_setgid(res, (long)(gid))
 112 | #define __sanitizer_syscall_pre_setreuid(ruid, euid)                           \
 113 |   __sanitizer_syscall_pre_impl_setreuid((long)(ruid), (long)(euid))
 114 | #define __sanitizer_syscall_post_setreuid(res, ruid, euid)                     \
 115 |   __sanitizer_syscall_post_impl_setreuid(res, (long)(ruid), (long)(euid))
 116 | #define __sanitizer_syscall_pre_setuid(uid)                                    \
 117 |   __sanitizer_syscall_pre_impl_setuid((long)(uid))
 118 | #define __sanitizer_syscall_post_setuid(res, uid)                              \
 119 |   __sanitizer_syscall_post_impl_setuid(res, (long)(uid))
 120 | #define __sanitizer_syscall_pre_setresuid(ruid, euid, suid)                    \
```
- **Line 111 / 第 111 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 112 / 第 112 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 113 / 第 113 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 114 / 第 114 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 115 / 第 115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 116 / 第 116 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 117 / 第 117 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 118 / 第 118 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 119 / 第 119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 120 / 第 120 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 121-130 / 第 121-130 行
```cpp
 121 |   __sanitizer_syscall_pre_impl_setresuid((long)(ruid), (long)(euid),           \
 122 |                                          (long)(suid))
 123 | #define __sanitizer_syscall_post_setresuid(res, ruid, euid, suid)              \
 124 |   __sanitizer_syscall_post_impl_setresuid(res, (long)(ruid), (long)(euid),     \
 125 |                                           (long)(suid))
 126 | #define __sanitizer_syscall_pre_setresgid(rgid, egid, sgid)                    \
 127 |   __sanitizer_syscall_pre_impl_setresgid((long)(rgid), (long)(egid),           \
 128 |                                          (long)(sgid))
 129 | #define __sanitizer_syscall_post_setresgid(res, rgid, egid, sgid)              \
 130 |   __sanitizer_syscall_post_impl_setresgid(res, (long)(rgid), (long)(egid),     \
```
- **Line 121 / 第 121 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 122 / 第 122 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 123 / 第 123 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 124 / 第 124 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 125 / 第 125 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 126 / 第 126 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 127 / 第 127 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 128 / 第 128 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 129 / 第 129 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 130 / 第 130 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 131-140 / 第 131-140 行
```cpp
 131 |                                           (long)(sgid))
 132 | #define __sanitizer_syscall_pre_setfsuid(uid)                                  \
 133 |   __sanitizer_syscall_pre_impl_setfsuid((long)(uid))
 134 | #define __sanitizer_syscall_post_setfsuid(res, uid)                            \
 135 |   __sanitizer_syscall_post_impl_setfsuid(res, (long)(uid))
 136 | #define __sanitizer_syscall_pre_setfsgid(gid)                                  \
 137 |   __sanitizer_syscall_pre_impl_setfsgid((long)(gid))
 138 | #define __sanitizer_syscall_post_setfsgid(res, gid)                            \
 139 |   __sanitizer_syscall_post_impl_setfsgid(res, (long)(gid))
 140 | #define __sanitizer_syscall_pre_setpgid(pid, pgid)                             \
```
- **Line 131 / 第 131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 132 / 第 132 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 133 / 第 133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 134 / 第 134 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 135 / 第 135 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 136 / 第 136 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 137 / 第 137 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 138 / 第 138 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 139 / 第 139 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 140 / 第 140 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 141-150 / 第 141-150 行
```cpp
 141 |   __sanitizer_syscall_pre_impl_setpgid((long)(pid), (long)(pgid))
 142 | #define __sanitizer_syscall_post_setpgid(res, pid, pgid)                       \
 143 |   __sanitizer_syscall_post_impl_setpgid(res, (long)(pid), (long)(pgid))
 144 | #define __sanitizer_syscall_pre_setsid() __sanitizer_syscall_pre_impl_setsid()
 145 | #define __sanitizer_syscall_post_setsid(res)                                   \
 146 |   __sanitizer_syscall_post_impl_setsid(res)
 147 | #define __sanitizer_syscall_pre_setgroups(gidsetsize, grouplist)               \
 148 |   __sanitizer_syscall_pre_impl_setgroups((long)(gidsetsize), (long)(grouplist))
 149 | #define __sanitizer_syscall_post_setgroups(res, gidsetsize, grouplist)         \
 150 |   __sanitizer_syscall_post_impl_setgroups(res, (long)(gidsetsize),             \
```
- **Line 141 / 第 141 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 142 / 第 142 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 143 / 第 143 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 144 / 第 144 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 145 / 第 145 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 146 / 第 146 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 147 / 第 147 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 148 / 第 148 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 149 / 第 149 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 150 / 第 150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 151-160 / 第 151-160 行
```cpp
 151 |                                           (long)(grouplist))
 152 | #define __sanitizer_syscall_pre_acct(name)                                     \
 153 |   __sanitizer_syscall_pre_impl_acct((long)(name))
 154 | #define __sanitizer_syscall_post_acct(res, name)                               \
 155 |   __sanitizer_syscall_post_impl_acct(res, (long)(name))
 156 | #define __sanitizer_syscall_pre_capget(header, dataptr)                        \
 157 |   __sanitizer_syscall_pre_impl_capget((long)(header), (long)(dataptr))
 158 | #define __sanitizer_syscall_post_capget(res, header, dataptr)                  \
 159 |   __sanitizer_syscall_post_impl_capget(res, (long)(header), (long)(dataptr))
 160 | #define __sanitizer_syscall_pre_capset(header, data)                           \
```
- **Line 151 / 第 151 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 152 / 第 152 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 153 / 第 153 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 154 / 第 154 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 155 / 第 155 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 156 / 第 156 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 157 / 第 157 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 158 / 第 158 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 159 / 第 159 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 160 / 第 160 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 161-170 / 第 161-170 行
```cpp
 161 |   __sanitizer_syscall_pre_impl_capset((long)(header), (long)(data))
 162 | #define __sanitizer_syscall_post_capset(res, header, data)                     \
 163 |   __sanitizer_syscall_post_impl_capset(res, (long)(header), (long)(data))
 164 | #define __sanitizer_syscall_pre_personality(personality)                       \
 165 |   __sanitizer_syscall_pre_impl_personality((long)(personality))
 166 | #define __sanitizer_syscall_post_personality(res, personality)                 \
 167 |   __sanitizer_syscall_post_impl_personality(res, (long)(personality))
 168 | #define __sanitizer_syscall_pre_sigpending(set)                                \
 169 |   __sanitizer_syscall_pre_impl_sigpending((long)(set))
 170 | #define __sanitizer_syscall_post_sigpending(res, set)                          \
```
- **Line 161 / 第 161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 162 / 第 162 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 163 / 第 163 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 164 / 第 164 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 165 / 第 165 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 166 / 第 166 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 167 / 第 167 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 168 / 第 168 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 169 / 第 169 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 170 / 第 170 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 171-180 / 第 171-180 行
```cpp
 171 |   __sanitizer_syscall_post_impl_sigpending(res, (long)(set))
 172 | #define __sanitizer_syscall_pre_sigprocmask(how, set, oset)                    \
 173 |   __sanitizer_syscall_pre_impl_sigprocmask((long)(how), (long)(set),           \
 174 |                                            (long)(oset))
 175 | #define __sanitizer_syscall_post_sigprocmask(res, how, set, oset)              \
 176 |   __sanitizer_syscall_post_impl_sigprocmask(res, (long)(how), (long)(set),     \
 177 |                                             (long)(oset))
 178 | #define __sanitizer_syscall_pre_getitimer(which, value)                        \
 179 |   __sanitizer_syscall_pre_impl_getitimer((long)(which), (long)(value))
 180 | #define __sanitizer_syscall_post_getitimer(res, which, value)                  \
```
- **Line 171 / 第 171 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 172 / 第 172 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 173 / 第 173 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 174 / 第 174 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 175 / 第 175 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 176 / 第 176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 177 / 第 177 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 178 / 第 178 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 179 / 第 179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 180 / 第 180 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 181-190 / 第 181-190 行
```cpp
 181 |   __sanitizer_syscall_post_impl_getitimer(res, (long)(which), (long)(value))
 182 | #define __sanitizer_syscall_pre_setitimer(which, value, ovalue)                \
 183 |   __sanitizer_syscall_pre_impl_setitimer((long)(which), (long)(value),         \
 184 |                                          (long)(ovalue))
 185 | #define __sanitizer_syscall_post_setitimer(res, which, value, ovalue)          \
 186 |   __sanitizer_syscall_post_impl_setitimer(res, (long)(which), (long)(value),   \
 187 |                                           (long)(ovalue))
 188 | #define __sanitizer_syscall_pre_timer_create(which_clock, timer_event_spec,    \
 189 |                                              created_timer_id)                 \
 190 |   __sanitizer_syscall_pre_impl_timer_create(                                   \
```
- **Line 181 / 第 181 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 182 / 第 182 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 183 / 第 183 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 184 / 第 184 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 185 / 第 185 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 186 / 第 186 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 187 / 第 187 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 188 / 第 188 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 189 / 第 189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 190 / 第 190 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 191-200 / 第 191-200 行
```cpp
 191 |       (long)(which_clock), (long)(timer_event_spec), (long)(created_timer_id))
 192 | #define __sanitizer_syscall_post_timer_create(                                 \
 193 |     res, which_clock, timer_event_spec, created_timer_id)                      \
 194 |   __sanitizer_syscall_post_impl_timer_create(res, (long)(which_clock),         \
 195 |                                              (long)(timer_event_spec),         \
 196 |                                              (long)(created_timer_id))
 197 | #define __sanitizer_syscall_pre_timer_gettime(timer_id, setting)               \
 198 |   __sanitizer_syscall_pre_impl_timer_gettime((long)(timer_id), (long)(setting))
 199 | #define __sanitizer_syscall_post_timer_gettime(res, timer_id, setting)         \
 200 |   __sanitizer_syscall_post_impl_timer_gettime(res, (long)(timer_id),           \
```
- **Line 191 / 第 191 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 192 / 第 192 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 193 / 第 193 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 194 / 第 194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 195 / 第 195 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 196 / 第 196 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 197 / 第 197 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 198 / 第 198 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 199 / 第 199 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 200 / 第 200 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 201-210 / 第 201-210 行
```cpp
 201 |                                               (long)(setting))
 202 | #define __sanitizer_syscall_pre_timer_getoverrun(timer_id)                     \
 203 |   __sanitizer_syscall_pre_impl_timer_getoverrun((long)(timer_id))
 204 | #define __sanitizer_syscall_post_timer_getoverrun(res, timer_id)               \
 205 |   __sanitizer_syscall_post_impl_timer_getoverrun(res, (long)(timer_id))
 206 | #define __sanitizer_syscall_pre_timer_settime(timer_id, flags, new_setting,    \
 207 |                                               old_setting)                     \
 208 |   __sanitizer_syscall_pre_impl_timer_settime((long)(timer_id), (long)(flags),  \
 209 |                                              (long)(new_setting),              \
 210 |                                              (long)(old_setting))
```
- **Line 201 / 第 201 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 202 / 第 202 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 203 / 第 203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 204 / 第 204 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 205 / 第 205 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 206 / 第 206 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 207 / 第 207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 208 / 第 208 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 209 / 第 209 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 210 / 第 210 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 211-220 / 第 211-220 行
```cpp
 211 | #define __sanitizer_syscall_post_timer_settime(res, timer_id, flags,           \
 212 |                                                new_setting, old_setting)       \
 213 |   __sanitizer_syscall_post_impl_timer_settime(                                 \
 214 |       res, (long)(timer_id), (long)(flags), (long)(new_setting),               \
 215 |       (long)(old_setting))
 216 | #define __sanitizer_syscall_pre_timer_delete(timer_id)                         \
 217 |   __sanitizer_syscall_pre_impl_timer_delete((long)(timer_id))
 218 | #define __sanitizer_syscall_post_timer_delete(res, timer_id)                   \
 219 |   __sanitizer_syscall_post_impl_timer_delete(res, (long)(timer_id))
 220 | #define __sanitizer_syscall_pre_clock_settime(which_clock, tp)                 \
```
- **Line 211 / 第 211 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 212 / 第 212 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 213 / 第 213 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 214 / 第 214 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 215 / 第 215 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 216 / 第 216 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 217 / 第 217 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 218 / 第 218 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 219 / 第 219 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 220 / 第 220 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 221-230 / 第 221-230 行
```cpp
 221 |   __sanitizer_syscall_pre_impl_clock_settime((long)(which_clock), (long)(tp))
 222 | #define __sanitizer_syscall_post_clock_settime(res, which_clock, tp)           \
 223 |   __sanitizer_syscall_post_impl_clock_settime(res, (long)(which_clock),        \
 224 |                                               (long)(tp))
 225 | #define __sanitizer_syscall_pre_clock_gettime(which_clock, tp)                 \
 226 |   __sanitizer_syscall_pre_impl_clock_gettime((long)(which_clock), (long)(tp))
 227 | #define __sanitizer_syscall_post_clock_gettime(res, which_clock, tp)           \
 228 |   __sanitizer_syscall_post_impl_clock_gettime(res, (long)(which_clock),        \
 229 |                                               (long)(tp))
 230 | #define __sanitizer_syscall_pre_clock_adjtime(which_clock, tx)                 \
```
- **Line 221 / 第 221 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 222 / 第 222 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 223 / 第 223 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 224 / 第 224 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 225 / 第 225 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 226 / 第 226 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 227 / 第 227 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 228 / 第 228 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 229 / 第 229 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 230 / 第 230 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 231-240 / 第 231-240 行
```cpp
 231 |   __sanitizer_syscall_pre_impl_clock_adjtime((long)(which_clock), (long)(tx))
 232 | #define __sanitizer_syscall_post_clock_adjtime(res, which_clock, tx)           \
 233 |   __sanitizer_syscall_post_impl_clock_adjtime(res, (long)(which_clock),        \
 234 |                                               (long)(tx))
 235 | #define __sanitizer_syscall_pre_clock_getres(which_clock, tp)                  \
 236 |   __sanitizer_syscall_pre_impl_clock_getres((long)(which_clock), (long)(tp))
 237 | #define __sanitizer_syscall_post_clock_getres(res, which_clock, tp)            \
 238 |   __sanitizer_syscall_post_impl_clock_getres(res, (long)(which_clock),         \
 239 |                                              (long)(tp))
 240 | #define __sanitizer_syscall_pre_clock_nanosleep(which_clock, flags, rqtp,      \
```
- **Line 231 / 第 231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 232 / 第 232 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 233 / 第 233 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 234 / 第 234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 235 / 第 235 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 236 / 第 236 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 237 / 第 237 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 238 / 第 238 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 239 / 第 239 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 240 / 第 240 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 241-250 / 第 241-250 行
```cpp
 241 |                                                 rmtp)                          \
 242 |   __sanitizer_syscall_pre_impl_clock_nanosleep(                                \
 243 |       (long)(which_clock), (long)(flags), (long)(rqtp), (long)(rmtp))
 244 | #define __sanitizer_syscall_post_clock_nanosleep(res, which_clock, flags,      \
 245 |                                                  rqtp, rmtp)                   \
 246 |   __sanitizer_syscall_post_impl_clock_nanosleep(                               \
 247 |       res, (long)(which_clock), (long)(flags), (long)(rqtp), (long)(rmtp))
 248 | #define __sanitizer_syscall_pre_nice(increment)                                \
 249 |   __sanitizer_syscall_pre_impl_nice((long)(increment))
 250 | #define __sanitizer_syscall_post_nice(res, increment)                          \
```
- **Line 241 / 第 241 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 242 / 第 242 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 243 / 第 243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 244 / 第 244 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 245 / 第 245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 246 / 第 246 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 247 / 第 247 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 248 / 第 248 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 249 / 第 249 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 250 / 第 250 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 251-260 / 第 251-260 行
```cpp
 251 |   __sanitizer_syscall_post_impl_nice(res, (long)(increment))
 252 | #define __sanitizer_syscall_pre_sched_setscheduler(pid, policy, param)         \
 253 |   __sanitizer_syscall_pre_impl_sched_setscheduler((long)(pid), (long)(policy), \
 254 |                                                   (long)(param))
 255 | #define __sanitizer_syscall_post_sched_setscheduler(res, pid, policy, param)   \
 256 |   __sanitizer_syscall_post_impl_sched_setscheduler(                            \
 257 |       res, (long)(pid), (long)(policy), (long)(param))
 258 | #define __sanitizer_syscall_pre_sched_setparam(pid, param)                     \
 259 |   __sanitizer_syscall_pre_impl_sched_setparam((long)(pid), (long)(param))
 260 | #define __sanitizer_syscall_post_sched_setparam(res, pid, param)               \
```
- **Line 251 / 第 251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 252 / 第 252 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 253 / 第 253 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 254 / 第 254 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 255 / 第 255 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 256 / 第 256 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 257 / 第 257 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 258 / 第 258 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 259 / 第 259 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 260 / 第 260 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 261-270 / 第 261-270 行
```cpp
 261 |   __sanitizer_syscall_post_impl_sched_setparam(res, (long)(pid), (long)(param))
 262 | #define __sanitizer_syscall_pre_sched_getscheduler(pid)                        \
 263 |   __sanitizer_syscall_pre_impl_sched_getscheduler((long)(pid))
 264 | #define __sanitizer_syscall_post_sched_getscheduler(res, pid)                  \
 265 |   __sanitizer_syscall_post_impl_sched_getscheduler(res, (long)(pid))
 266 | #define __sanitizer_syscall_pre_sched_getparam(pid, param)                     \
 267 |   __sanitizer_syscall_pre_impl_sched_getparam((long)(pid), (long)(param))
 268 | #define __sanitizer_syscall_post_sched_getparam(res, pid, param)               \
 269 |   __sanitizer_syscall_post_impl_sched_getparam(res, (long)(pid), (long)(param))
 270 | #define __sanitizer_syscall_pre_sched_setaffinity(pid, len, user_mask_ptr)     \
```
- **Line 261 / 第 261 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 262 / 第 262 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 263 / 第 263 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 264 / 第 264 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 265 / 第 265 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 266 / 第 266 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 267 / 第 267 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 268 / 第 268 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 269 / 第 269 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 270 / 第 270 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 271-280 / 第 271-280 行
```cpp
 271 |   __sanitizer_syscall_pre_impl_sched_setaffinity((long)(pid), (long)(len),     \
 272 |                                                  (long)(user_mask_ptr))
 273 | #define __sanitizer_syscall_post_sched_setaffinity(res, pid, len,              \
 274 |                                                    user_mask_ptr)              \
 275 |   __sanitizer_syscall_post_impl_sched_setaffinity(                             \
 276 |       res, (long)(pid), (long)(len), (long)(user_mask_ptr))
 277 | #define __sanitizer_syscall_pre_sched_getaffinity(pid, len, user_mask_ptr)     \
 278 |   __sanitizer_syscall_pre_impl_sched_getaffinity((long)(pid), (long)(len),     \
 279 |                                                  (long)(user_mask_ptr))
 280 | #define __sanitizer_syscall_post_sched_getaffinity(res, pid, len,              \
```
- **Line 271 / 第 271 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 272 / 第 272 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 273 / 第 273 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 274 / 第 274 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 275 / 第 275 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 276 / 第 276 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 277 / 第 277 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 278 / 第 278 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 279 / 第 279 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 280 / 第 280 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 281-290 / 第 281-290 行
```cpp
 281 |                                                    user_mask_ptr)              \
 282 |   __sanitizer_syscall_post_impl_sched_getaffinity(                             \
 283 |       res, (long)(pid), (long)(len), (long)(user_mask_ptr))
 284 | #define __sanitizer_syscall_pre_sched_yield()                                  \
 285 |   __sanitizer_syscall_pre_impl_sched_yield()
 286 | #define __sanitizer_syscall_post_sched_yield(res)                              \
 287 |   __sanitizer_syscall_post_impl_sched_yield(res)
 288 | #define __sanitizer_syscall_pre_sched_get_priority_max(policy)                 \
 289 |   __sanitizer_syscall_pre_impl_sched_get_priority_max((long)(policy))
 290 | #define __sanitizer_syscall_post_sched_get_priority_max(res, policy)           \
```
- **Line 281 / 第 281 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 282 / 第 282 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 283 / 第 283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 284 / 第 284 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 285 / 第 285 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 286 / 第 286 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 287 / 第 287 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 288 / 第 288 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 289 / 第 289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 290 / 第 290 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 291-300 / 第 291-300 行
```cpp
 291 |   __sanitizer_syscall_post_impl_sched_get_priority_max(res, (long)(policy))
 292 | #define __sanitizer_syscall_pre_sched_get_priority_min(policy)                 \
 293 |   __sanitizer_syscall_pre_impl_sched_get_priority_min((long)(policy))
 294 | #define __sanitizer_syscall_post_sched_get_priority_min(res, policy)           \
 295 |   __sanitizer_syscall_post_impl_sched_get_priority_min(res, (long)(policy))
 296 | #define __sanitizer_syscall_pre_sched_rr_get_interval(pid, interval)           \
 297 |   __sanitizer_syscall_pre_impl_sched_rr_get_interval((long)(pid),              \
 298 |                                                      (long)(interval))
 299 | #define __sanitizer_syscall_post_sched_rr_get_interval(res, pid, interval)     \
 300 |   __sanitizer_syscall_post_impl_sched_rr_get_interval(res, (long)(pid),        \
```
- **Line 291 / 第 291 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 292 / 第 292 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 293 / 第 293 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 294 / 第 294 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 295 / 第 295 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 296 / 第 296 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 297 / 第 297 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 298 / 第 298 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 299 / 第 299 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 300 / 第 300 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 301-310 / 第 301-310 行
```cpp
 301 |                                                       (long)(interval))
 302 | #define __sanitizer_syscall_pre_setpriority(which, who, niceval)               \
 303 |   __sanitizer_syscall_pre_impl_setpriority((long)(which), (long)(who),         \
 304 |                                            (long)(niceval))
 305 | #define __sanitizer_syscall_post_setpriority(res, which, who, niceval)         \
 306 |   __sanitizer_syscall_post_impl_setpriority(res, (long)(which), (long)(who),   \
 307 |                                             (long)(niceval))
 308 | #define __sanitizer_syscall_pre_getpriority(which, who)                        \
 309 |   __sanitizer_syscall_pre_impl_getpriority((long)(which), (long)(who))
 310 | #define __sanitizer_syscall_post_getpriority(res, which, who)                  \
```
- **Line 301 / 第 301 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 302 / 第 302 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 303 / 第 303 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 304 / 第 304 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 305 / 第 305 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 306 / 第 306 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 307 / 第 307 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 308 / 第 308 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 309 / 第 309 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 310 / 第 310 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 311-320 / 第 311-320 行
```cpp
 311 |   __sanitizer_syscall_post_impl_getpriority(res, (long)(which), (long)(who))
 312 | #define __sanitizer_syscall_pre_shutdown(arg0, arg1)                           \
 313 |   __sanitizer_syscall_pre_impl_shutdown((long)(arg0), (long)(arg1))
 314 | #define __sanitizer_syscall_post_shutdown(res, arg0, arg1)                     \
 315 |   __sanitizer_syscall_post_impl_shutdown(res, (long)(arg0), (long)(arg1))
 316 | #define __sanitizer_syscall_pre_reboot(magic1, magic2, cmd, arg)               \
 317 |   __sanitizer_syscall_pre_impl_reboot((long)(magic1), (long)(magic2),          \
 318 |                                       (long)(cmd), (long)(arg))
 319 | #define __sanitizer_syscall_post_reboot(res, magic1, magic2, cmd, arg)         \
 320 |   __sanitizer_syscall_post_impl_reboot(res, (long)(magic1), (long)(magic2),    \
```
- **Line 311 / 第 311 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 312 / 第 312 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 313 / 第 313 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 314 / 第 314 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 315 / 第 315 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 316 / 第 316 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 317 / 第 317 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 318 / 第 318 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 319 / 第 319 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 320 / 第 320 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 321-330 / 第 321-330 行
```cpp
 321 |                                        (long)(cmd), (long)(arg))
 322 | #define __sanitizer_syscall_pre_restart_syscall()                              \
 323 |   __sanitizer_syscall_pre_impl_restart_syscall()
 324 | #define __sanitizer_syscall_post_restart_syscall(res)                          \
 325 |   __sanitizer_syscall_post_impl_restart_syscall(res)
 326 | #define __sanitizer_syscall_pre_kexec_load(entry, nr_segments, segments,       \
 327 |                                            flags)                              \
 328 |   __sanitizer_syscall_pre_impl_kexec_load((long)(entry), (long)(nr_segments),  \
 329 |                                           (long)(segments), (long)(flags))
 330 | #define __sanitizer_syscall_post_kexec_load(res, entry, nr_segments, segments, \
```
- **Line 321 / 第 321 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 322 / 第 322 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 323 / 第 323 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 324 / 第 324 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 325 / 第 325 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 326 / 第 326 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 327 / 第 327 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 328 / 第 328 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 329 / 第 329 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 330 / 第 330 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 331-340 / 第 331-340 行
```cpp
 331 |                                             flags)                             \
 332 |   __sanitizer_syscall_post_impl_kexec_load(res, (long)(entry),                 \
 333 |                                            (long)(nr_segments),                \
 334 |                                            (long)(segments), (long)(flags))
 335 | #define __sanitizer_syscall_pre_exit(error_code)                               \
 336 |   __sanitizer_syscall_pre_impl_exit((long)(error_code))
 337 | #define __sanitizer_syscall_post_exit(res, error_code)                         \
 338 |   __sanitizer_syscall_post_impl_exit(res, (long)(error_code))
 339 | #define __sanitizer_syscall_pre_exit_group(error_code)                         \
 340 |   __sanitizer_syscall_pre_impl_exit_group((long)(error_code))
```
- **Line 331 / 第 331 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 332 / 第 332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 333 / 第 333 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 334 / 第 334 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 335 / 第 335 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 336 / 第 336 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 337 / 第 337 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 338 / 第 338 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 339 / 第 339 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 340 / 第 340 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 341-350 / 第 341-350 行
```cpp
 341 | #define __sanitizer_syscall_post_exit_group(res, error_code)                   \
 342 |   __sanitizer_syscall_post_impl_exit_group(res, (long)(error_code))
 343 | #define __sanitizer_syscall_pre_wait4(pid, stat_addr, options, ru)             \
 344 |   __sanitizer_syscall_pre_impl_wait4((long)(pid), (long)(stat_addr),           \
 345 |                                      (long)(options), (long)(ru))
 346 | #define __sanitizer_syscall_post_wait4(res, pid, stat_addr, options, ru)       \
 347 |   __sanitizer_syscall_post_impl_wait4(res, (long)(pid), (long)(stat_addr),     \
 348 |                                       (long)(options), (long)(ru))
 349 | #define __sanitizer_syscall_pre_waitid(which, pid, infop, options, ru)         \
 350 |   __sanitizer_syscall_pre_impl_waitid(                                         \
```
- **Line 341 / 第 341 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 342 / 第 342 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 343 / 第 343 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 344 / 第 344 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 345 / 第 345 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 346 / 第 346 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 347 / 第 347 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 348 / 第 348 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 349 / 第 349 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 350 / 第 350 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 351-360 / 第 351-360 行
```cpp
 351 |       (long)(which), (long)(pid), (long)(infop), (long)(options), (long)(ru))
 352 | #define __sanitizer_syscall_post_waitid(res, which, pid, infop, options, ru)   \
 353 |   __sanitizer_syscall_post_impl_waitid(res, (long)(which), (long)(pid),        \
 354 |                                        (long)(infop), (long)(options),         \
 355 |                                        (long)(ru))
 356 | #define __sanitizer_syscall_pre_waitpid(pid, stat_addr, options)               \
 357 |   __sanitizer_syscall_pre_impl_waitpid((long)(pid), (long)(stat_addr),         \
 358 |                                        (long)(options))
 359 | #define __sanitizer_syscall_post_waitpid(res, pid, stat_addr, options)         \
 360 |   __sanitizer_syscall_post_impl_waitpid(res, (long)(pid), (long)(stat_addr),   \
```
- **Line 351 / 第 351 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 352 / 第 352 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 353 / 第 353 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 354 / 第 354 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 355 / 第 355 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 356 / 第 356 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 357 / 第 357 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 358 / 第 358 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 359 / 第 359 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 360 / 第 360 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 361-370 / 第 361-370 行
```cpp
 361 |                                         (long)(options))
 362 | #define __sanitizer_syscall_pre_set_tid_address(tidptr)                        \
 363 |   __sanitizer_syscall_pre_impl_set_tid_address((long)(tidptr))
 364 | #define __sanitizer_syscall_post_set_tid_address(res, tidptr)                  \
 365 |   __sanitizer_syscall_post_impl_set_tid_address(res, (long)(tidptr))
 366 | #define __sanitizer_syscall_pre_init_module(umod, len, uargs)                  \
 367 |   __sanitizer_syscall_pre_impl_init_module((long)(umod), (long)(len),          \
 368 |                                            (long)(uargs))
 369 | #define __sanitizer_syscall_post_init_module(res, umod, len, uargs)            \
 370 |   __sanitizer_syscall_post_impl_init_module(res, (long)(umod), (long)(len),    \
```
- **Line 361 / 第 361 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 362 / 第 362 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 363 / 第 363 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 364 / 第 364 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 365 / 第 365 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 366 / 第 366 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 367 / 第 367 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 368 / 第 368 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 369 / 第 369 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 370 / 第 370 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 371-380 / 第 371-380 行
```cpp
 371 |                                             (long)(uargs))
 372 | #define __sanitizer_syscall_pre_delete_module(name_user, flags)                \
 373 |   __sanitizer_syscall_pre_impl_delete_module((long)(name_user), (long)(flags))
 374 | #define __sanitizer_syscall_post_delete_module(res, name_user, flags)          \
 375 |   __sanitizer_syscall_post_impl_delete_module(res, (long)(name_user),          \
 376 |                                               (long)(flags))
 377 | #define __sanitizer_syscall_pre_rt_sigprocmask(how, set, oset, sigsetsize)     \
 378 |   __sanitizer_syscall_pre_impl_rt_sigprocmask(                                 \
 379 |       (long)(how), (long)(set), (long)(oset), (long)(sigsetsize))
 380 | #define __sanitizer_syscall_post_rt_sigprocmask(res, how, set, oset,           \
```
- **Line 371 / 第 371 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 372 / 第 372 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 373 / 第 373 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 374 / 第 374 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 375 / 第 375 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 376 / 第 376 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 377 / 第 377 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 378 / 第 378 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 379 / 第 379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 380 / 第 380 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 381-390 / 第 381-390 行
```cpp
 381 |                                                 sigsetsize)                    \
 382 |   __sanitizer_syscall_post_impl_rt_sigprocmask(                                \
 383 |       res, (long)(how), (long)(set), (long)(oset), (long)(sigsetsize))
 384 | #define __sanitizer_syscall_pre_rt_sigpending(set, sigsetsize)                 \
 385 |   __sanitizer_syscall_pre_impl_rt_sigpending((long)(set), (long)(sigsetsize))
 386 | #define __sanitizer_syscall_post_rt_sigpending(res, set, sigsetsize)           \
 387 |   __sanitizer_syscall_post_impl_rt_sigpending(res, (long)(set),                \
 388 |                                               (long)(sigsetsize))
 389 | #define __sanitizer_syscall_pre_rt_sigtimedwait(uthese, uinfo, uts,            \
 390 |                                                 sigsetsize)                    \
```
- **Line 381 / 第 381 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 382 / 第 382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 383 / 第 383 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 384 / 第 384 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 385 / 第 385 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 386 / 第 386 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 387 / 第 387 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 388 / 第 388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 389 / 第 389 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 390 / 第 390 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 391-400 / 第 391-400 行
```cpp
 391 |   __sanitizer_syscall_pre_impl_rt_sigtimedwait(                                \
 392 |       (long)(uthese), (long)(uinfo), (long)(uts), (long)(sigsetsize))
 393 | #define __sanitizer_syscall_post_rt_sigtimedwait(res, uthese, uinfo, uts,      \
 394 |                                                  sigsetsize)                   \
 395 |   __sanitizer_syscall_post_impl_rt_sigtimedwait(                               \
 396 |       res, (long)(uthese), (long)(uinfo), (long)(uts), (long)(sigsetsize))
 397 | #define __sanitizer_syscall_pre_rt_tgsigqueueinfo(tgid, pid, sig, uinfo)       \
 398 |   __sanitizer_syscall_pre_impl_rt_tgsigqueueinfo((long)(tgid), (long)(pid),    \
 399 |                                                  (long)(sig), (long)(uinfo))
 400 | #define __sanitizer_syscall_post_rt_tgsigqueueinfo(res, tgid, pid, sig, uinfo) \
```
- **Line 391 / 第 391 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 392 / 第 392 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 393 / 第 393 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 394 / 第 394 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 395 / 第 395 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 396 / 第 396 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 397 / 第 397 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 398 / 第 398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 399 / 第 399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 400 / 第 400 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 401-410 / 第 401-410 行
```cpp
 401 |   __sanitizer_syscall_post_impl_rt_tgsigqueueinfo(                             \
 402 |       res, (long)(tgid), (long)(pid), (long)(sig), (long)(uinfo))
 403 | #define __sanitizer_syscall_pre_kill(pid, sig)                                 \
 404 |   __sanitizer_syscall_pre_impl_kill((long)(pid), (long)(sig))
 405 | #define __sanitizer_syscall_post_kill(res, pid, sig)                           \
 406 |   __sanitizer_syscall_post_impl_kill(res, (long)(pid), (long)(sig))
 407 | #define __sanitizer_syscall_pre_tgkill(tgid, pid, sig)                         \
 408 |   __sanitizer_syscall_pre_impl_tgkill((long)(tgid), (long)(pid), (long)(sig))
 409 | #define __sanitizer_syscall_post_tgkill(res, tgid, pid, sig)                   \
 410 |   __sanitizer_syscall_post_impl_tgkill(res, (long)(tgid), (long)(pid),         \
```
- **Line 401 / 第 401 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 402 / 第 402 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 403 / 第 403 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 404 / 第 404 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 405 / 第 405 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 406 / 第 406 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 407 / 第 407 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 408 / 第 408 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 409 / 第 409 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 410 / 第 410 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 411-420 / 第 411-420 行
```cpp
 411 |                                        (long)(sig))
 412 | #define __sanitizer_syscall_pre_tkill(pid, sig)                                \
 413 |   __sanitizer_syscall_pre_impl_tkill((long)(pid), (long)(sig))
 414 | #define __sanitizer_syscall_post_tkill(res, pid, sig)                          \
 415 |   __sanitizer_syscall_post_impl_tkill(res, (long)(pid), (long)(sig))
 416 | #define __sanitizer_syscall_pre_rt_sigqueueinfo(pid, sig, uinfo)               \
 417 |   __sanitizer_syscall_pre_impl_rt_sigqueueinfo((long)(pid), (long)(sig),       \
 418 |                                                (long)(uinfo))
 419 | #define __sanitizer_syscall_post_rt_sigqueueinfo(res, pid, sig, uinfo)         \
 420 |   __sanitizer_syscall_post_impl_rt_sigqueueinfo(res, (long)(pid), (long)(sig), \
```
- **Line 411 / 第 411 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 412 / 第 412 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 413 / 第 413 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 414 / 第 414 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 415 / 第 415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 416 / 第 416 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 417 / 第 417 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 418 / 第 418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 419 / 第 419 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 420 / 第 420 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 421-430 / 第 421-430 行
```cpp
 421 |                                                 (long)(uinfo))
 422 | #define __sanitizer_syscall_pre_sgetmask()                                     \
 423 |   __sanitizer_syscall_pre_impl_sgetmask()
 424 | #define __sanitizer_syscall_post_sgetmask(res)                                 \
 425 |   __sanitizer_syscall_post_impl_sgetmask(res)
 426 | #define __sanitizer_syscall_pre_ssetmask(newmask)                              \
 427 |   __sanitizer_syscall_pre_impl_ssetmask((long)(newmask))
 428 | #define __sanitizer_syscall_post_ssetmask(res, newmask)                        \
 429 |   __sanitizer_syscall_post_impl_ssetmask(res, (long)(newmask))
 430 | #define __sanitizer_syscall_pre_signal(sig, handler)                           \
```
- **Line 421 / 第 421 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 422 / 第 422 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 423 / 第 423 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 424 / 第 424 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 425 / 第 425 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 426 / 第 426 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 427 / 第 427 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 428 / 第 428 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 429 / 第 429 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 430 / 第 430 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 431-440 / 第 431-440 行
```cpp
 431 |   __sanitizer_syscall_pre_impl_signal((long)(sig), (long)(handler))
 432 | #define __sanitizer_syscall_post_signal(res, sig, handler)                     \
 433 |   __sanitizer_syscall_post_impl_signal(res, (long)(sig), (long)(handler))
 434 | #define __sanitizer_syscall_pre_pause() __sanitizer_syscall_pre_impl_pause()
 435 | #define __sanitizer_syscall_post_pause(res)                                    \
 436 |   __sanitizer_syscall_post_impl_pause(res)
 437 | #define __sanitizer_syscall_pre_sync() __sanitizer_syscall_pre_impl_sync()
 438 | #define __sanitizer_syscall_post_sync(res)                                     \
 439 |   __sanitizer_syscall_post_impl_sync(res)
 440 | #define __sanitizer_syscall_pre_fsync(fd)                                      \
```
- **Line 431 / 第 431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 432 / 第 432 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 433 / 第 433 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 434 / 第 434 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 435 / 第 435 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 436 / 第 436 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 437 / 第 437 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 438 / 第 438 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 439 / 第 439 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 440 / 第 440 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 441-450 / 第 441-450 行
```cpp
 441 |   __sanitizer_syscall_pre_impl_fsync((long)(fd))
 442 | #define __sanitizer_syscall_post_fsync(res, fd)                                \
 443 |   __sanitizer_syscall_post_impl_fsync(res, (long)(fd))
 444 | #define __sanitizer_syscall_pre_fdatasync(fd)                                  \
 445 |   __sanitizer_syscall_pre_impl_fdatasync((long)(fd))
 446 | #define __sanitizer_syscall_post_fdatasync(res, fd)                            \
 447 |   __sanitizer_syscall_post_impl_fdatasync(res, (long)(fd))
 448 | #define __sanitizer_syscall_pre_bdflush(func, data)                            \
 449 |   __sanitizer_syscall_pre_impl_bdflush((long)(func), (long)(data))
 450 | #define __sanitizer_syscall_post_bdflush(res, func, data)                      \
```
- **Line 441 / 第 441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 442 / 第 442 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 443 / 第 443 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 444 / 第 444 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 445 / 第 445 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 446 / 第 446 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 447 / 第 447 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 448 / 第 448 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 449 / 第 449 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 450 / 第 450 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 451-460 / 第 451-460 行
```cpp
 451 |   __sanitizer_syscall_post_impl_bdflush(res, (long)(func), (long)(data))
 452 | #define __sanitizer_syscall_pre_mount(dev_name, dir_name, type, flags, data)   \
 453 |   __sanitizer_syscall_pre_impl_mount((long)(dev_name), (long)(dir_name),       \
 454 |                                      (long)(type), (long)(flags),              \
 455 |                                      (long)(data))
 456 | #define __sanitizer_syscall_post_mount(res, dev_name, dir_name, type, flags,   \
 457 |                                        data)                                   \
 458 |   __sanitizer_syscall_post_impl_mount(res, (long)(dev_name), (long)(dir_name), \
 459 |                                       (long)(type), (long)(flags),             \
 460 |                                       (long)(data))
```
- **Line 451 / 第 451 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 452 / 第 452 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 453 / 第 453 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 454 / 第 454 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 455 / 第 455 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 456 / 第 456 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 457 / 第 457 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 458 / 第 458 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 459 / 第 459 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 460 / 第 460 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 461-470 / 第 461-470 行
```cpp
 461 | #define __sanitizer_syscall_pre_umount(name, flags)                            \
 462 |   __sanitizer_syscall_pre_impl_umount((long)(name), (long)(flags))
 463 | #define __sanitizer_syscall_post_umount(res, name, flags)                      \
 464 |   __sanitizer_syscall_post_impl_umount(res, (long)(name), (long)(flags))
 465 | #define __sanitizer_syscall_pre_oldumount(name)                                \
 466 |   __sanitizer_syscall_pre_impl_oldumount((long)(name))
 467 | #define __sanitizer_syscall_post_oldumount(res, name)                          \
 468 |   __sanitizer_syscall_post_impl_oldumount(res, (long)(name))
 469 | #define __sanitizer_syscall_pre_truncate(path, length)                         \
 470 |   __sanitizer_syscall_pre_impl_truncate((long)(path), (long)(length))
```
- **Line 461 / 第 461 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 462 / 第 462 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 463 / 第 463 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 464 / 第 464 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 465 / 第 465 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 466 / 第 466 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 467 / 第 467 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 468 / 第 468 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 469 / 第 469 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 470 / 第 470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 471-480 / 第 471-480 行
```cpp
 471 | #define __sanitizer_syscall_post_truncate(res, path, length)                   \
 472 |   __sanitizer_syscall_post_impl_truncate(res, (long)(path), (long)(length))
 473 | #define __sanitizer_syscall_pre_ftruncate(fd, length)                          \
 474 |   __sanitizer_syscall_pre_impl_ftruncate((long)(fd), (long)(length))
 475 | #define __sanitizer_syscall_post_ftruncate(res, fd, length)                    \
 476 |   __sanitizer_syscall_post_impl_ftruncate(res, (long)(fd), (long)(length))
 477 | #define __sanitizer_syscall_pre_stat(filename, statbuf)                        \
 478 |   __sanitizer_syscall_pre_impl_stat((long)(filename), (long)(statbuf))
 479 | #define __sanitizer_syscall_post_stat(res, filename, statbuf)                  \
 480 |   __sanitizer_syscall_post_impl_stat(res, (long)(filename), (long)(statbuf))
```
- **Line 471 / 第 471 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 472 / 第 472 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 473 / 第 473 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 474 / 第 474 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 475 / 第 475 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 476 / 第 476 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 477 / 第 477 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 478 / 第 478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 479 / 第 479 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 480 / 第 480 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 481-490 / 第 481-490 行
```cpp
 481 | #define __sanitizer_syscall_pre_statfs(path, buf)                              \
 482 |   __sanitizer_syscall_pre_impl_statfs((long)(path), (long)(buf))
 483 | #define __sanitizer_syscall_post_statfs(res, path, buf)                        \
 484 |   __sanitizer_syscall_post_impl_statfs(res, (long)(path), (long)(buf))
 485 | #define __sanitizer_syscall_pre_statfs64(path, sz, buf)                        \
 486 |   __sanitizer_syscall_pre_impl_statfs64((long)(path), (long)(sz), (long)(buf))
 487 | #define __sanitizer_syscall_post_statfs64(res, path, sz, buf)                  \
 488 |   __sanitizer_syscall_post_impl_statfs64(res, (long)(path), (long)(sz),        \
 489 |                                          (long)(buf))
 490 | #define __sanitizer_syscall_pre_fstatfs(fd, buf)                               \
```
- **Line 481 / 第 481 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 482 / 第 482 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 483 / 第 483 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 484 / 第 484 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 485 / 第 485 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 486 / 第 486 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 487 / 第 487 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 488 / 第 488 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 489 / 第 489 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 490 / 第 490 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 491-500 / 第 491-500 行
```cpp
 491 |   __sanitizer_syscall_pre_impl_fstatfs((long)(fd), (long)(buf))
 492 | #define __sanitizer_syscall_post_fstatfs(res, fd, buf)                         \
 493 |   __sanitizer_syscall_post_impl_fstatfs(res, (long)(fd), (long)(buf))
 494 | #define __sanitizer_syscall_pre_fstatfs64(fd, sz, buf)                         \
 495 |   __sanitizer_syscall_pre_impl_fstatfs64((long)(fd), (long)(sz), (long)(buf))
 496 | #define __sanitizer_syscall_post_fstatfs64(res, fd, sz, buf)                   \
 497 |   __sanitizer_syscall_post_impl_fstatfs64(res, (long)(fd), (long)(sz),         \
 498 |                                           (long)(buf))
 499 | #define __sanitizer_syscall_pre_lstat(filename, statbuf)                       \
 500 |   __sanitizer_syscall_pre_impl_lstat((long)(filename), (long)(statbuf))
```
- **Line 491 / 第 491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 492 / 第 492 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 493 / 第 493 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 494 / 第 494 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 495 / 第 495 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 496 / 第 496 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 497 / 第 497 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 498 / 第 498 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 499 / 第 499 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 500 / 第 500 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 501-510 / 第 501-510 行
```cpp
 501 | #define __sanitizer_syscall_post_lstat(res, filename, statbuf)                 \
 502 |   __sanitizer_syscall_post_impl_lstat(res, (long)(filename), (long)(statbuf))
 503 | #define __sanitizer_syscall_pre_fstat(fd, statbuf)                             \
 504 |   __sanitizer_syscall_pre_impl_fstat((long)(fd), (long)(statbuf))
 505 | #define __sanitizer_syscall_post_fstat(res, fd, statbuf)                       \
 506 |   __sanitizer_syscall_post_impl_fstat(res, (long)(fd), (long)(statbuf))
 507 | #define __sanitizer_syscall_pre_newstat(filename, statbuf)                     \
 508 |   __sanitizer_syscall_pre_impl_newstat((long)(filename), (long)(statbuf))
 509 | #define __sanitizer_syscall_post_newstat(res, filename, statbuf)               \
 510 |   __sanitizer_syscall_post_impl_newstat(res, (long)(filename), (long)(statbuf))
```
- **Line 501 / 第 501 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 502 / 第 502 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 503 / 第 503 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 504 / 第 504 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 505 / 第 505 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 506 / 第 506 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 507 / 第 507 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 508 / 第 508 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 509 / 第 509 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 510 / 第 510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 511-520 / 第 511-520 行
```cpp
 511 | #define __sanitizer_syscall_pre_newlstat(filename, statbuf)                    \
 512 |   __sanitizer_syscall_pre_impl_newlstat((long)(filename), (long)(statbuf))
 513 | #define __sanitizer_syscall_post_newlstat(res, filename, statbuf)              \
 514 |   __sanitizer_syscall_post_impl_newlstat(res, (long)(filename), (long)(statbuf))
 515 | #define __sanitizer_syscall_pre_newfstat(fd, statbuf)                          \
 516 |   __sanitizer_syscall_pre_impl_newfstat((long)(fd), (long)(statbuf))
 517 | #define __sanitizer_syscall_post_newfstat(res, fd, statbuf)                    \
 518 |   __sanitizer_syscall_post_impl_newfstat(res, (long)(fd), (long)(statbuf))
 519 | #define __sanitizer_syscall_pre_ustat(dev, ubuf)                               \
 520 |   __sanitizer_syscall_pre_impl_ustat((long)(dev), (long)(ubuf))
```
- **Line 511 / 第 511 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 512 / 第 512 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 513 / 第 513 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 514 / 第 514 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 515 / 第 515 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 516 / 第 516 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 517 / 第 517 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 518 / 第 518 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 519 / 第 519 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 520 / 第 520 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 521-530 / 第 521-530 行
```cpp
 521 | #define __sanitizer_syscall_post_ustat(res, dev, ubuf)                         \
 522 |   __sanitizer_syscall_post_impl_ustat(res, (long)(dev), (long)(ubuf))
 523 | #define __sanitizer_syscall_pre_stat64(filename, statbuf)                      \
 524 |   __sanitizer_syscall_pre_impl_stat64((long)(filename), (long)(statbuf))
 525 | #define __sanitizer_syscall_post_stat64(res, filename, statbuf)                \
 526 |   __sanitizer_syscall_post_impl_stat64(res, (long)(filename), (long)(statbuf))
 527 | #define __sanitizer_syscall_pre_fstat64(fd, statbuf)                           \
 528 |   __sanitizer_syscall_pre_impl_fstat64((long)(fd), (long)(statbuf))
 529 | #define __sanitizer_syscall_post_fstat64(res, fd, statbuf)                     \
 530 |   __sanitizer_syscall_post_impl_fstat64(res, (long)(fd), (long)(statbuf))
```
- **Line 521 / 第 521 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 522 / 第 522 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 523 / 第 523 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 524 / 第 524 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 525 / 第 525 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 526 / 第 526 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 527 / 第 527 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 528 / 第 528 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 529 / 第 529 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 530 / 第 530 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 531-540 / 第 531-540 行
```cpp
 531 | #define __sanitizer_syscall_pre_lstat64(filename, statbuf)                     \
 532 |   __sanitizer_syscall_pre_impl_lstat64((long)(filename), (long)(statbuf))
 533 | #define __sanitizer_syscall_post_lstat64(res, filename, statbuf)               \
 534 |   __sanitizer_syscall_post_impl_lstat64(res, (long)(filename), (long)(statbuf))
 535 | #define __sanitizer_syscall_pre_setxattr(path, name, value, size, flags)       \
 536 |   __sanitizer_syscall_pre_impl_setxattr(                                       \
 537 |       (long)(path), (long)(name), (long)(value), (long)(size), (long)(flags))
 538 | #define __sanitizer_syscall_post_setxattr(res, path, name, value, size, flags) \
 539 |   __sanitizer_syscall_post_impl_setxattr(res, (long)(path), (long)(name),      \
 540 |                                          (long)(value), (long)(size),          \
```
- **Line 531 / 第 531 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 532 / 第 532 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 533 / 第 533 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 534 / 第 534 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 535 / 第 535 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 536 / 第 536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 537 / 第 537 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 538 / 第 538 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 539 / 第 539 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 540 / 第 540 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 541-550 / 第 541-550 行
```cpp
 541 |                                          (long)(flags))
 542 | #define __sanitizer_syscall_pre_lsetxattr(path, name, value, size, flags)      \
 543 |   __sanitizer_syscall_pre_impl_lsetxattr(                                      \
 544 |       (long)(path), (long)(name), (long)(value), (long)(size), (long)(flags))
 545 | #define __sanitizer_syscall_post_lsetxattr(res, path, name, value, size,       \
 546 |                                            flags)                              \
 547 |   __sanitizer_syscall_post_impl_lsetxattr(res, (long)(path), (long)(name),     \
 548 |                                           (long)(value), (long)(size),         \
 549 |                                           (long)(flags))
 550 | #define __sanitizer_syscall_pre_fsetxattr(fd, name, value, size, flags)        \
```
- **Line 541 / 第 541 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 542 / 第 542 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 543 / 第 543 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 544 / 第 544 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 545 / 第 545 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 546 / 第 546 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 547 / 第 547 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 548 / 第 548 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 549 / 第 549 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 550 / 第 550 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 551-560 / 第 551-560 行
```cpp
 551 |   __sanitizer_syscall_pre_impl_fsetxattr(                                      \
 552 |       (long)(fd), (long)(name), (long)(value), (long)(size), (long)(flags))
 553 | #define __sanitizer_syscall_post_fsetxattr(res, fd, name, value, size, flags)  \
 554 |   __sanitizer_syscall_post_impl_fsetxattr(res, (long)(fd), (long)(name),       \
 555 |                                           (long)(value), (long)(size),         \
 556 |                                           (long)(flags))
 557 | #define __sanitizer_syscall_pre_getxattr(path, name, value, size)              \
 558 |   __sanitizer_syscall_pre_impl_getxattr((long)(path), (long)(name),            \
 559 |                                         (long)(value), (long)(size))
 560 | #define __sanitizer_syscall_post_getxattr(res, path, name, value, size)        \
```
- **Line 551 / 第 551 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 552 / 第 552 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 553 / 第 553 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 554 / 第 554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 555 / 第 555 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 556 / 第 556 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 557 / 第 557 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 558 / 第 558 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 559 / 第 559 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 560 / 第 560 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 561-570 / 第 561-570 行
```cpp
 561 |   __sanitizer_syscall_post_impl_getxattr(res, (long)(path), (long)(name),      \
 562 |                                          (long)(value), (long)(size))
 563 | #define __sanitizer_syscall_pre_lgetxattr(path, name, value, size)             \
 564 |   __sanitizer_syscall_pre_impl_lgetxattr((long)(path), (long)(name),           \
 565 |                                          (long)(value), (long)(size))
 566 | #define __sanitizer_syscall_post_lgetxattr(res, path, name, value, size)       \
 567 |   __sanitizer_syscall_post_impl_lgetxattr(res, (long)(path), (long)(name),     \
 568 |                                           (long)(value), (long)(size))
 569 | #define __sanitizer_syscall_pre_fgetxattr(fd, name, value, size)               \
 570 |   __sanitizer_syscall_pre_impl_fgetxattr((long)(fd), (long)(name),             \
```
- **Line 561 / 第 561 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 562 / 第 562 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 563 / 第 563 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 564 / 第 564 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 565 / 第 565 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 566 / 第 566 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 567 / 第 567 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 568 / 第 568 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 569 / 第 569 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 570 / 第 570 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 571-580 / 第 571-580 行
```cpp
 571 |                                          (long)(value), (long)(size))
 572 | #define __sanitizer_syscall_post_fgetxattr(res, fd, name, value, size)         \
 573 |   __sanitizer_syscall_post_impl_fgetxattr(res, (long)(fd), (long)(name),       \
 574 |                                           (long)(value), (long)(size))
 575 | #define __sanitizer_syscall_pre_listxattr(path, list, size)                    \
 576 |   __sanitizer_syscall_pre_impl_listxattr((long)(path), (long)(list),           \
 577 |                                          (long)(size))
 578 | #define __sanitizer_syscall_post_listxattr(res, path, list, size)              \
 579 |   __sanitizer_syscall_post_impl_listxattr(res, (long)(path), (long)(list),     \
 580 |                                           (long)(size))
```
- **Line 571 / 第 571 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 572 / 第 572 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 573 / 第 573 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 574 / 第 574 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 575 / 第 575 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 576 / 第 576 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 577 / 第 577 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 578 / 第 578 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 579 / 第 579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 580 / 第 580 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 581-590 / 第 581-590 行
```cpp
 581 | #define __sanitizer_syscall_pre_llistxattr(path, list, size)                   \
 582 |   __sanitizer_syscall_pre_impl_llistxattr((long)(path), (long)(list),          \
 583 |                                           (long)(size))
 584 | #define __sanitizer_syscall_post_llistxattr(res, path, list, size)             \
 585 |   __sanitizer_syscall_post_impl_llistxattr(res, (long)(path), (long)(list),    \
 586 |                                            (long)(size))
 587 | #define __sanitizer_syscall_pre_flistxattr(fd, list, size)                     \
 588 |   __sanitizer_syscall_pre_impl_flistxattr((long)(fd), (long)(list),            \
 589 |                                           (long)(size))
 590 | #define __sanitizer_syscall_post_flistxattr(res, fd, list, size)               \
```
- **Line 581 / 第 581 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 582 / 第 582 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 583 / 第 583 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 584 / 第 584 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 585 / 第 585 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 586 / 第 586 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 587 / 第 587 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 588 / 第 588 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 589 / 第 589 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 590 / 第 590 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 591-600 / 第 591-600 行
```cpp
 591 |   __sanitizer_syscall_post_impl_flistxattr(res, (long)(fd), (long)(list),      \
 592 |                                            (long)(size))
 593 | #define __sanitizer_syscall_pre_removexattr(path, name)                        \
 594 |   __sanitizer_syscall_pre_impl_removexattr((long)(path), (long)(name))
 595 | #define __sanitizer_syscall_post_removexattr(res, path, name)                  \
 596 |   __sanitizer_syscall_post_impl_removexattr(res, (long)(path), (long)(name))
 597 | #define __sanitizer_syscall_pre_lremovexattr(path, name)                       \
 598 |   __sanitizer_syscall_pre_impl_lremovexattr((long)(path), (long)(name))
 599 | #define __sanitizer_syscall_post_lremovexattr(res, path, name)                 \
 600 |   __sanitizer_syscall_post_impl_lremovexattr(res, (long)(path), (long)(name))
```
- **Line 591 / 第 591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 592 / 第 592 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 593 / 第 593 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 594 / 第 594 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 595 / 第 595 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 596 / 第 596 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 597 / 第 597 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 598 / 第 598 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 599 / 第 599 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 600 / 第 600 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 601-610 / 第 601-610 行
```cpp
 601 | #define __sanitizer_syscall_pre_fremovexattr(fd, name)                         \
 602 |   __sanitizer_syscall_pre_impl_fremovexattr((long)(fd), (long)(name))
 603 | #define __sanitizer_syscall_post_fremovexattr(res, fd, name)                   \
 604 |   __sanitizer_syscall_post_impl_fremovexattr(res, (long)(fd), (long)(name))
 605 | #define __sanitizer_syscall_pre_brk(brk)                                       \
 606 |   __sanitizer_syscall_pre_impl_brk((long)(brk))
 607 | #define __sanitizer_syscall_post_brk(res, brk)                                 \
 608 |   __sanitizer_syscall_post_impl_brk(res, (long)(brk))
 609 | #define __sanitizer_syscall_pre_mprotect(start, len, prot)                     \
 610 |   __sanitizer_syscall_pre_impl_mprotect((long)(start), (long)(len),            \
```
- **Line 601 / 第 601 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 602 / 第 602 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 603 / 第 603 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 604 / 第 604 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 605 / 第 605 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 606 / 第 606 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 607 / 第 607 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 608 / 第 608 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 609 / 第 609 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 610 / 第 610 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 611-620 / 第 611-620 行
```cpp
 611 |                                         (long)(prot))
 612 | #define __sanitizer_syscall_post_mprotect(res, start, len, prot)               \
 613 |   __sanitizer_syscall_post_impl_mprotect(res, (long)(start), (long)(len),      \
 614 |                                          (long)(prot))
 615 | #define __sanitizer_syscall_pre_mremap(addr, old_len, new_len, flags,          \
 616 |                                        new_addr)                               \
 617 |   __sanitizer_syscall_pre_impl_mremap((long)(addr), (long)(old_len),           \
 618 |                                       (long)(new_len), (long)(flags),          \
 619 |                                       (long)(new_addr))
 620 | #define __sanitizer_syscall_post_mremap(res, addr, old_len, new_len, flags,    \
```
- **Line 611 / 第 611 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 612 / 第 612 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 613 / 第 613 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 614 / 第 614 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 615 / 第 615 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 616 / 第 616 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 617 / 第 617 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 618 / 第 618 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 619 / 第 619 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 620 / 第 620 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 621-630 / 第 621-630 行
```cpp
 621 |                                         new_addr)                              \
 622 |   __sanitizer_syscall_post_impl_mremap(res, (long)(addr), (long)(old_len),     \
 623 |                                        (long)(new_len), (long)(flags),         \
 624 |                                        (long)(new_addr))
 625 | #define __sanitizer_syscall_pre_remap_file_pages(start, size, prot, pgoff,     \
 626 |                                                  flags)                        \
 627 |   __sanitizer_syscall_pre_impl_remap_file_pages(                               \
 628 |       (long)(start), (long)(size), (long)(prot), (long)(pgoff), (long)(flags))
 629 | #define __sanitizer_syscall_post_remap_file_pages(res, start, size, prot,      \
 630 |                                                   pgoff, flags)                \
```
- **Line 621 / 第 621 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 622 / 第 622 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 623 / 第 623 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 624 / 第 624 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 625 / 第 625 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 626 / 第 626 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 627 / 第 627 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 628 / 第 628 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 629 / 第 629 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 630 / 第 630 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 631-640 / 第 631-640 行
```cpp
 631 |   __sanitizer_syscall_post_impl_remap_file_pages(res, (long)(start),           \
 632 |                                                  (long)(size), (long)(prot),   \
 633 |                                                  (long)(pgoff), (long)(flags))
 634 | #define __sanitizer_syscall_pre_msync(start, len, flags)                       \
 635 |   __sanitizer_syscall_pre_impl_msync((long)(start), (long)(len), (long)(flags))
 636 | #define __sanitizer_syscall_post_msync(res, start, len, flags)                 \
 637 |   __sanitizer_syscall_post_impl_msync(res, (long)(start), (long)(len),         \
 638 |                                       (long)(flags))
 639 | #define __sanitizer_syscall_pre_munmap(addr, len)                              \
 640 |   __sanitizer_syscall_pre_impl_munmap((long)(addr), (long)(len))
```
- **Line 631 / 第 631 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 632 / 第 632 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 633 / 第 633 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 634 / 第 634 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 635 / 第 635 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 636 / 第 636 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 637 / 第 637 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 638 / 第 638 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 639 / 第 639 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 640 / 第 640 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 641-650 / 第 641-650 行
```cpp
 641 | #define __sanitizer_syscall_post_munmap(res, addr, len)                        \
 642 |   __sanitizer_syscall_post_impl_munmap(res, (long)(addr), (long)(len))
 643 | #define __sanitizer_syscall_pre_mlock(start, len)                              \
 644 |   __sanitizer_syscall_pre_impl_mlock((long)(start), (long)(len))
 645 | #define __sanitizer_syscall_post_mlock(res, start, len)                        \
 646 |   __sanitizer_syscall_post_impl_mlock(res, (long)(start), (long)(len))
 647 | #define __sanitizer_syscall_pre_munlock(start, len)                            \
 648 |   __sanitizer_syscall_pre_impl_munlock((long)(start), (long)(len))
 649 | #define __sanitizer_syscall_post_munlock(res, start, len)                      \
 650 |   __sanitizer_syscall_post_impl_munlock(res, (long)(start), (long)(len))
```
- **Line 641 / 第 641 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 642 / 第 642 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 643 / 第 643 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 644 / 第 644 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 645 / 第 645 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 646 / 第 646 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 647 / 第 647 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 648 / 第 648 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 649 / 第 649 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 650 / 第 650 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 651-660 / 第 651-660 行
```cpp
 651 | #define __sanitizer_syscall_pre_mlockall(flags)                                \
 652 |   __sanitizer_syscall_pre_impl_mlockall((long)(flags))
 653 | #define __sanitizer_syscall_post_mlockall(res, flags)                          \
 654 |   __sanitizer_syscall_post_impl_mlockall(res, (long)(flags))
 655 | #define __sanitizer_syscall_pre_munlockall()                                   \
 656 |   __sanitizer_syscall_pre_impl_munlockall()
 657 | #define __sanitizer_syscall_post_munlockall(res)                               \
 658 |   __sanitizer_syscall_post_impl_munlockall(res)
 659 | #define __sanitizer_syscall_pre_madvise(start, len, behavior)                  \
 660 |   __sanitizer_syscall_pre_impl_madvise((long)(start), (long)(len),             \
```
- **Line 651 / 第 651 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 652 / 第 652 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 653 / 第 653 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 654 / 第 654 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 655 / 第 655 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 656 / 第 656 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 657 / 第 657 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 658 / 第 658 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 659 / 第 659 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 660 / 第 660 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 661-670 / 第 661-670 行
```cpp
 661 |                                        (long)(behavior))
 662 | #define __sanitizer_syscall_post_madvise(res, start, len, behavior)            \
 663 |   __sanitizer_syscall_post_impl_madvise(res, (long)(start), (long)(len),       \
 664 |                                         (long)(behavior))
 665 | #define __sanitizer_syscall_pre_mincore(start, len, vec)                       \
 666 |   __sanitizer_syscall_pre_impl_mincore((long)(start), (long)(len), (long)(vec))
 667 | #define __sanitizer_syscall_post_mincore(res, start, len, vec)                 \
 668 |   __sanitizer_syscall_post_impl_mincore(res, (long)(start), (long)(len),       \
 669 |                                         (long)(vec))
 670 | #define __sanitizer_syscall_pre_pivot_root(new_root, put_old)                  \
```
- **Line 661 / 第 661 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 662 / 第 662 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 663 / 第 663 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 664 / 第 664 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 665 / 第 665 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 666 / 第 666 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 667 / 第 667 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 668 / 第 668 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 669 / 第 669 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 670 / 第 670 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 671-680 / 第 671-680 行
```cpp
 671 |   __sanitizer_syscall_pre_impl_pivot_root((long)(new_root), (long)(put_old))
 672 | #define __sanitizer_syscall_post_pivot_root(res, new_root, put_old)            \
 673 |   __sanitizer_syscall_post_impl_pivot_root(res, (long)(new_root),              \
 674 |                                            (long)(put_old))
 675 | #define __sanitizer_syscall_pre_chroot(filename)                               \
 676 |   __sanitizer_syscall_pre_impl_chroot((long)(filename))
 677 | #define __sanitizer_syscall_post_chroot(res, filename)                         \
 678 |   __sanitizer_syscall_post_impl_chroot(res, (long)(filename))
 679 | #define __sanitizer_syscall_pre_mknod(filename, mode, dev)                     \
 680 |   __sanitizer_syscall_pre_impl_mknod((long)(filename), (long)(mode),           \
```
- **Line 671 / 第 671 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 672 / 第 672 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 673 / 第 673 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 674 / 第 674 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 675 / 第 675 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 676 / 第 676 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 677 / 第 677 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 678 / 第 678 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 679 / 第 679 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 680 / 第 680 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 681-690 / 第 681-690 行
```cpp
 681 |                                      (long)(dev))
 682 | #define __sanitizer_syscall_post_mknod(res, filename, mode, dev)               \
 683 |   __sanitizer_syscall_post_impl_mknod(res, (long)(filename), (long)(mode),     \
 684 |                                       (long)(dev))
 685 | #define __sanitizer_syscall_pre_link(oldname, newname)                         \
 686 |   __sanitizer_syscall_pre_impl_link((long)(oldname), (long)(newname))
 687 | #define __sanitizer_syscall_post_link(res, oldname, newname)                   \
 688 |   __sanitizer_syscall_post_impl_link(res, (long)(oldname), (long)(newname))
 689 | #define __sanitizer_syscall_pre_symlink(old, new_)                             \
 690 |   __sanitizer_syscall_pre_impl_symlink((long)(old), (long)(new_))
```
- **Line 681 / 第 681 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 682 / 第 682 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 683 / 第 683 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 684 / 第 684 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 685 / 第 685 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 686 / 第 686 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 687 / 第 687 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 688 / 第 688 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 689 / 第 689 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 690 / 第 690 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 691-700 / 第 691-700 行
```cpp
 691 | #define __sanitizer_syscall_post_symlink(res, old, new_)                       \
 692 |   __sanitizer_syscall_post_impl_symlink(res, (long)(old), (long)(new_))
 693 | #define __sanitizer_syscall_pre_unlink(pathname)                               \
 694 |   __sanitizer_syscall_pre_impl_unlink((long)(pathname))
 695 | #define __sanitizer_syscall_post_unlink(res, pathname)                         \
 696 |   __sanitizer_syscall_post_impl_unlink(res, (long)(pathname))
 697 | #define __sanitizer_syscall_pre_rename(oldname, newname)                       \
 698 |   __sanitizer_syscall_pre_impl_rename((long)(oldname), (long)(newname))
 699 | #define __sanitizer_syscall_post_rename(res, oldname, newname)                 \
 700 |   __sanitizer_syscall_post_impl_rename(res, (long)(oldname), (long)(newname))
```
- **Line 691 / 第 691 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 692 / 第 692 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 693 / 第 693 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 694 / 第 694 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 695 / 第 695 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 696 / 第 696 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 697 / 第 697 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 698 / 第 698 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 699 / 第 699 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 700 / 第 700 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 701-710 / 第 701-710 行
```cpp
 701 | #define __sanitizer_syscall_pre_chmod(filename, mode)                          \
 702 |   __sanitizer_syscall_pre_impl_chmod((long)(filename), (long)(mode))
 703 | #define __sanitizer_syscall_post_chmod(res, filename, mode)                    \
 704 |   __sanitizer_syscall_post_impl_chmod(res, (long)(filename), (long)(mode))
 705 | #define __sanitizer_syscall_pre_fchmod(fd, mode)                               \
 706 |   __sanitizer_syscall_pre_impl_fchmod((long)(fd), (long)(mode))
 707 | #define __sanitizer_syscall_post_fchmod(res, fd, mode)                         \
 708 |   __sanitizer_syscall_post_impl_fchmod(res, (long)(fd), (long)(mode))
 709 | #define __sanitizer_syscall_pre_fcntl(fd, cmd, arg)                            \
 710 |   __sanitizer_syscall_pre_impl_fcntl((long)(fd), (long)(cmd), (long)(arg))
```
- **Line 701 / 第 701 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 702 / 第 702 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 703 / 第 703 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 704 / 第 704 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 705 / 第 705 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 706 / 第 706 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 707 / 第 707 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 708 / 第 708 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 709 / 第 709 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 710 / 第 710 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 711-720 / 第 711-720 行
```cpp
 711 | #define __sanitizer_syscall_post_fcntl(res, fd, cmd, arg)                      \
 712 |   __sanitizer_syscall_post_impl_fcntl(res, (long)(fd), (long)(cmd), (long)(arg))
 713 | #define __sanitizer_syscall_pre_fcntl64(fd, cmd, arg)                          \
 714 |   __sanitizer_syscall_pre_impl_fcntl64((long)(fd), (long)(cmd), (long)(arg))
 715 | #define __sanitizer_syscall_post_fcntl64(res, fd, cmd, arg)                    \
 716 |   __sanitizer_syscall_post_impl_fcntl64(res, (long)(fd), (long)(cmd),          \
 717 |                                         (long)(arg))
 718 | #define __sanitizer_syscall_pre_pipe(fildes)                                   \
 719 |   __sanitizer_syscall_pre_impl_pipe((long)(fildes))
 720 | #define __sanitizer_syscall_post_pipe(res, fildes)                             \
```
- **Line 711 / 第 711 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 712 / 第 712 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 713 / 第 713 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 714 / 第 714 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 715 / 第 715 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 716 / 第 716 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 717 / 第 717 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 718 / 第 718 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 719 / 第 719 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 720 / 第 720 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 721-730 / 第 721-730 行
```cpp
 721 |   __sanitizer_syscall_post_impl_pipe(res, (long)(fildes))
 722 | #define __sanitizer_syscall_pre_pipe2(fildes, flags)                           \
 723 |   __sanitizer_syscall_pre_impl_pipe2((long)(fildes), (long)(flags))
 724 | #define __sanitizer_syscall_post_pipe2(res, fildes, flags)                     \
 725 |   __sanitizer_syscall_post_impl_pipe2(res, (long)(fildes), (long)(flags))
 726 | #define __sanitizer_syscall_pre_dup(fildes)                                    \
 727 |   __sanitizer_syscall_pre_impl_dup((long)(fildes))
 728 | #define __sanitizer_syscall_post_dup(res, fildes)                              \
 729 |   __sanitizer_syscall_post_impl_dup(res, (long)(fildes))
 730 | #define __sanitizer_syscall_pre_dup2(oldfd, newfd)                             \
```
- **Line 721 / 第 721 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 722 / 第 722 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 723 / 第 723 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 724 / 第 724 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 725 / 第 725 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 726 / 第 726 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 727 / 第 727 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 728 / 第 728 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 729 / 第 729 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 730 / 第 730 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 731-740 / 第 731-740 行
```cpp
 731 |   __sanitizer_syscall_pre_impl_dup2((long)(oldfd), (long)(newfd))
 732 | #define __sanitizer_syscall_post_dup2(res, oldfd, newfd)                       \
 733 |   __sanitizer_syscall_post_impl_dup2(res, (long)(oldfd), (long)(newfd))
 734 | #define __sanitizer_syscall_pre_dup3(oldfd, newfd, flags)                      \
 735 |   __sanitizer_syscall_pre_impl_dup3((long)(oldfd), (long)(newfd), (long)(flags))
 736 | #define __sanitizer_syscall_post_dup3(res, oldfd, newfd, flags)                \
 737 |   __sanitizer_syscall_post_impl_dup3(res, (long)(oldfd), (long)(newfd),        \
 738 |                                      (long)(flags))
 739 | #define __sanitizer_syscall_pre_ioperm(from, num, on)                          \
 740 |   __sanitizer_syscall_pre_impl_ioperm((long)(from), (long)(num), (long)(on))
```
- **Line 731 / 第 731 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 732 / 第 732 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 733 / 第 733 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 734 / 第 734 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 735 / 第 735 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 736 / 第 736 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 737 / 第 737 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 738 / 第 738 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 739 / 第 739 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 740 / 第 740 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 741-750 / 第 741-750 行
```cpp
 741 | #define __sanitizer_syscall_post_ioperm(res, from, num, on)                    \
 742 |   __sanitizer_syscall_post_impl_ioperm(res, (long)(from), (long)(num),         \
 743 |                                        (long)(on))
 744 | #define __sanitizer_syscall_pre_ioctl(fd, cmd, arg)                            \
 745 |   __sanitizer_syscall_pre_impl_ioctl((long)(fd), (long)(cmd), (long)(arg))
 746 | #define __sanitizer_syscall_post_ioctl(res, fd, cmd, arg)                      \
 747 |   __sanitizer_syscall_post_impl_ioctl(res, (long)(fd), (long)(cmd), (long)(arg))
 748 | #define __sanitizer_syscall_pre_flock(fd, cmd)                                 \
 749 |   __sanitizer_syscall_pre_impl_flock((long)(fd), (long)(cmd))
 750 | #define __sanitizer_syscall_post_flock(res, fd, cmd)                           \
```
- **Line 741 / 第 741 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 742 / 第 742 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 743 / 第 743 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 744 / 第 744 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 745 / 第 745 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 746 / 第 746 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 747 / 第 747 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 748 / 第 748 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 749 / 第 749 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 750 / 第 750 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 751-760 / 第 751-760 行
```cpp
 751 |   __sanitizer_syscall_post_impl_flock(res, (long)(fd), (long)(cmd))
 752 | #define __sanitizer_syscall_pre_io_setup(nr_reqs, ctx)                         \
 753 |   __sanitizer_syscall_pre_impl_io_setup((long)(nr_reqs), (long)(ctx))
 754 | #define __sanitizer_syscall_post_io_setup(res, nr_reqs, ctx)                   \
 755 |   __sanitizer_syscall_post_impl_io_setup(res, (long)(nr_reqs), (long)(ctx))
 756 | #define __sanitizer_syscall_pre_io_destroy(ctx)                                \
 757 |   __sanitizer_syscall_pre_impl_io_destroy((long)(ctx))
 758 | #define __sanitizer_syscall_post_io_destroy(res, ctx)                          \
 759 |   __sanitizer_syscall_post_impl_io_destroy(res, (long)(ctx))
 760 | #define __sanitizer_syscall_pre_io_getevents(ctx_id, min_nr, nr, events,       \
```
- **Line 751 / 第 751 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 752 / 第 752 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 753 / 第 753 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 754 / 第 754 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 755 / 第 755 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 756 / 第 756 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 757 / 第 757 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 758 / 第 758 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 759 / 第 759 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 760 / 第 760 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 761-770 / 第 761-770 行
```cpp
 761 |                                              timeout)                          \
 762 |   __sanitizer_syscall_pre_impl_io_getevents((long)(ctx_id), (long)(min_nr),    \
 763 |                                             (long)(nr), (long)(events),        \
 764 |                                             (long)(timeout))
 765 | #define __sanitizer_syscall_post_io_getevents(res, ctx_id, min_nr, nr, events, \
 766 |                                               timeout)                         \
 767 |   __sanitizer_syscall_post_impl_io_getevents(res, (long)(ctx_id),              \
 768 |                                              (long)(min_nr), (long)(nr),       \
 769 |                                              (long)(events), (long)(timeout))
 770 | #define __sanitizer_syscall_pre_io_submit(ctx_id, arg1, arg2)                  \
```
- **Line 761 / 第 761 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 762 / 第 762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 763 / 第 763 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 764 / 第 764 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 765 / 第 765 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 766 / 第 766 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 767 / 第 767 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 768 / 第 768 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 769 / 第 769 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 770 / 第 770 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 771-780 / 第 771-780 行
```cpp
 771 |   __sanitizer_syscall_pre_impl_io_submit((long)(ctx_id), (long)(arg1),         \
 772 |                                          (long)(arg2))
 773 | #define __sanitizer_syscall_post_io_submit(res, ctx_id, arg1, arg2)            \
 774 |   __sanitizer_syscall_post_impl_io_submit(res, (long)(ctx_id), (long)(arg1),   \
 775 |                                           (long)(arg2))
 776 | #define __sanitizer_syscall_pre_io_cancel(ctx_id, iocb, result)                \
 777 |   __sanitizer_syscall_pre_impl_io_cancel((long)(ctx_id), (long)(iocb),         \
 778 |                                          (long)(result))
 779 | #define __sanitizer_syscall_post_io_cancel(res, ctx_id, iocb, result)          \
 780 |   __sanitizer_syscall_post_impl_io_cancel(res, (long)(ctx_id), (long)(iocb),   \
```
- **Line 771 / 第 771 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 772 / 第 772 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 773 / 第 773 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 774 / 第 774 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 775 / 第 775 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 776 / 第 776 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 777 / 第 777 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 778 / 第 778 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 779 / 第 779 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 780 / 第 780 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 781-790 / 第 781-790 行
```cpp
 781 |                                           (long)(result))
 782 | #define __sanitizer_syscall_pre_sendfile(out_fd, in_fd, offset, count)         \
 783 |   __sanitizer_syscall_pre_impl_sendfile((long)(out_fd), (long)(in_fd),         \
 784 |                                         (long)(offset), (long)(count))
 785 | #define __sanitizer_syscall_post_sendfile(res, out_fd, in_fd, offset, count)   \
 786 |   __sanitizer_syscall_post_impl_sendfile(res, (long)(out_fd), (long)(in_fd),   \
 787 |                                          (long)(offset), (long)(count))
 788 | #define __sanitizer_syscall_pre_sendfile64(out_fd, in_fd, offset, count)       \
 789 |   __sanitizer_syscall_pre_impl_sendfile64((long)(out_fd), (long)(in_fd),       \
 790 |                                           (long)(offset), (long)(count))
```
- **Line 781 / 第 781 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 782 / 第 782 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 783 / 第 783 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 784 / 第 784 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 785 / 第 785 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 786 / 第 786 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 787 / 第 787 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 788 / 第 788 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 789 / 第 789 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 790 / 第 790 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 791-800 / 第 791-800 行
```cpp
 791 | #define __sanitizer_syscall_post_sendfile64(res, out_fd, in_fd, offset, count) \
 792 |   __sanitizer_syscall_post_impl_sendfile64(res, (long)(out_fd), (long)(in_fd), \
 793 |                                            (long)(offset), (long)(count))
 794 | #define __sanitizer_syscall_pre_readlink(path, buf, bufsiz)                    \
 795 |   __sanitizer_syscall_pre_impl_readlink((long)(path), (long)(buf),             \
 796 |                                         (long)(bufsiz))
 797 | #define __sanitizer_syscall_post_readlink(res, path, buf, bufsiz)              \
 798 |   __sanitizer_syscall_post_impl_readlink(res, (long)(path), (long)(buf),       \
 799 |                                          (long)(bufsiz))
 800 | #define __sanitizer_syscall_pre_creat(pathname, mode)                          \
```
- **Line 791 / 第 791 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 792 / 第 792 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 793 / 第 793 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 794 / 第 794 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 795 / 第 795 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 796 / 第 796 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 797 / 第 797 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 798 / 第 798 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 799 / 第 799 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 800 / 第 800 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 801-810 / 第 801-810 行
```cpp
 801 |   __sanitizer_syscall_pre_impl_creat((long)(pathname), (long)(mode))
 802 | #define __sanitizer_syscall_post_creat(res, pathname, mode)                    \
 803 |   __sanitizer_syscall_post_impl_creat(res, (long)(pathname), (long)(mode))
 804 | #define __sanitizer_syscall_pre_open(filename, flags, mode)                    \
 805 |   __sanitizer_syscall_pre_impl_open((long)(filename), (long)(flags),           \
 806 |                                     (long)(mode))
 807 | #define __sanitizer_syscall_post_open(res, filename, flags, mode)              \
 808 |   __sanitizer_syscall_post_impl_open(res, (long)(filename), (long)(flags),     \
 809 |                                      (long)(mode))
 810 | #define __sanitizer_syscall_pre_close(fd)                                      \
```
- **Line 801 / 第 801 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 802 / 第 802 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 803 / 第 803 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 804 / 第 804 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 805 / 第 805 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 806 / 第 806 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 807 / 第 807 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 808 / 第 808 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 809 / 第 809 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 810 / 第 810 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 811-820 / 第 811-820 行
```cpp
 811 |   __sanitizer_syscall_pre_impl_close((long)(fd))
 812 | #define __sanitizer_syscall_post_close(res, fd)                                \
 813 |   __sanitizer_syscall_post_impl_close(res, (long)(fd))
 814 | #define __sanitizer_syscall_pre_access(filename, mode)                         \
 815 |   __sanitizer_syscall_pre_impl_access((long)(filename), (long)(mode))
 816 | #define __sanitizer_syscall_post_access(res, filename, mode)                   \
 817 |   __sanitizer_syscall_post_impl_access(res, (long)(filename), (long)(mode))
 818 | #define __sanitizer_syscall_pre_vhangup() __sanitizer_syscall_pre_impl_vhangup()
 819 | #define __sanitizer_syscall_post_vhangup(res)                                  \
 820 |   __sanitizer_syscall_post_impl_vhangup(res)
```
- **Line 811 / 第 811 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 812 / 第 812 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 813 / 第 813 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 814 / 第 814 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 815 / 第 815 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 816 / 第 816 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 817 / 第 817 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 818 / 第 818 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 819 / 第 819 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 820 / 第 820 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 821-830 / 第 821-830 行
```cpp
 821 | #define __sanitizer_syscall_pre_chown(filename, user, group)                   \
 822 |   __sanitizer_syscall_pre_impl_chown((long)(filename), (long)(user),           \
 823 |                                      (long)(group))
 824 | #define __sanitizer_syscall_post_chown(res, filename, user, group)             \
 825 |   __sanitizer_syscall_post_impl_chown(res, (long)(filename), (long)(user),     \
 826 |                                       (long)(group))
 827 | #define __sanitizer_syscall_pre_lchown(filename, user, group)                  \
 828 |   __sanitizer_syscall_pre_impl_lchown((long)(filename), (long)(user),          \
 829 |                                       (long)(group))
 830 | #define __sanitizer_syscall_post_lchown(res, filename, user, group)            \
```
- **Line 821 / 第 821 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 822 / 第 822 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 823 / 第 823 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 824 / 第 824 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 825 / 第 825 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 826 / 第 826 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 827 / 第 827 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 828 / 第 828 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 829 / 第 829 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 830 / 第 830 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 831-840 / 第 831-840 行
```cpp
 831 |   __sanitizer_syscall_post_impl_lchown(res, (long)(filename), (long)(user),    \
 832 |                                        (long)(group))
 833 | #define __sanitizer_syscall_pre_fchown(fd, user, group)                        \
 834 |   __sanitizer_syscall_pre_impl_fchown((long)(fd), (long)(user), (long)(group))
 835 | #define __sanitizer_syscall_post_fchown(res, fd, user, group)                  \
 836 |   __sanitizer_syscall_post_impl_fchown(res, (long)(fd), (long)(user),          \
 837 |                                        (long)(group))
 838 | #define __sanitizer_syscall_pre_chown16(filename, user, group)                 \
 839 |   __sanitizer_syscall_pre_impl_chown16((long)(filename), (long)user,           \
 840 |                                        (long)group)
```
- **Line 831 / 第 831 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 832 / 第 832 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 833 / 第 833 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 834 / 第 834 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 835 / 第 835 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 836 / 第 836 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 837 / 第 837 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 838 / 第 838 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 839 / 第 839 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 840 / 第 840 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 841-850 / 第 841-850 行
```cpp
 841 | #define __sanitizer_syscall_post_chown16(res, filename, user, group)           \
 842 |   __sanitizer_syscall_post_impl_chown16(res, (long)(filename), (long)user,     \
 843 |                                         (long)group)
 844 | #define __sanitizer_syscall_pre_lchown16(filename, user, group)                \
 845 |   __sanitizer_syscall_pre_impl_lchown16((long)(filename), (long)user,          \
 846 |                                         (long)group)
 847 | #define __sanitizer_syscall_post_lchown16(res, filename, user, group)          \
 848 |   __sanitizer_syscall_post_impl_lchown16(res, (long)(filename), (long)user,    \
 849 |                                          (long)group)
 850 | #define __sanitizer_syscall_pre_fchown16(fd, user, group)                      \
```
- **Line 841 / 第 841 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 842 / 第 842 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 843 / 第 843 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 844 / 第 844 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 845 / 第 845 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 846 / 第 846 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 847 / 第 847 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 848 / 第 848 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 849 / 第 849 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 850 / 第 850 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 851-860 / 第 851-860 行
```cpp
 851 |   __sanitizer_syscall_pre_impl_fchown16((long)(fd), (long)user, (long)group)
 852 | #define __sanitizer_syscall_post_fchown16(res, fd, user, group)                \
 853 |   __sanitizer_syscall_post_impl_fchown16(res, (long)(fd), (long)user,          \
 854 |                                          (long)group)
 855 | #define __sanitizer_syscall_pre_setregid16(rgid, egid)                         \
 856 |   __sanitizer_syscall_pre_impl_setregid16((long)rgid, (long)egid)
 857 | #define __sanitizer_syscall_post_setregid16(res, rgid, egid)                   \
 858 |   __sanitizer_syscall_post_impl_setregid16(res, (long)rgid, (long)egid)
 859 | #define __sanitizer_syscall_pre_setgid16(gid)                                  \
 860 |   __sanitizer_syscall_pre_impl_setgid16((long)gid)
```
- **Line 851 / 第 851 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 852 / 第 852 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 853 / 第 853 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 854 / 第 854 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 855 / 第 855 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 856 / 第 856 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 857 / 第 857 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 858 / 第 858 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 859 / 第 859 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 860 / 第 860 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 861-870 / 第 861-870 行
```cpp
 861 | #define __sanitizer_syscall_post_setgid16(res, gid)                            \
 862 |   __sanitizer_syscall_post_impl_setgid16(res, (long)gid)
 863 | #define __sanitizer_syscall_pre_setreuid16(ruid, euid)                         \
 864 |   __sanitizer_syscall_pre_impl_setreuid16((long)ruid, (long)euid)
 865 | #define __sanitizer_syscall_post_setreuid16(res, ruid, euid)                   \
 866 |   __sanitizer_syscall_post_impl_setreuid16(res, (long)ruid, (long)euid)
 867 | #define __sanitizer_syscall_pre_setuid16(uid)                                  \
 868 |   __sanitizer_syscall_pre_impl_setuid16((long)uid)
 869 | #define __sanitizer_syscall_post_setuid16(res, uid)                            \
 870 |   __sanitizer_syscall_post_impl_setuid16(res, (long)uid)
```
- **Line 861 / 第 861 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 862 / 第 862 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 863 / 第 863 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 864 / 第 864 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 865 / 第 865 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 866 / 第 866 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 867 / 第 867 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 868 / 第 868 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 869 / 第 869 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 870 / 第 870 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 871-880 / 第 871-880 行
```cpp
 871 | #define __sanitizer_syscall_pre_setresuid16(ruid, euid, suid)                  \
 872 |   __sanitizer_syscall_pre_impl_setresuid16((long)ruid, (long)euid, (long)suid)
 873 | #define __sanitizer_syscall_post_setresuid16(res, ruid, euid, suid)            \
 874 |   __sanitizer_syscall_post_impl_setresuid16(res, (long)ruid, (long)euid,       \
 875 |                                             (long)suid)
 876 | #define __sanitizer_syscall_pre_getresuid16(ruid, euid, suid)                  \
 877 |   __sanitizer_syscall_pre_impl_getresuid16((long)(ruid), (long)(euid),         \
 878 |                                            (long)(suid))
 879 | #define __sanitizer_syscall_post_getresuid16(res, ruid, euid, suid)            \
 880 |   __sanitizer_syscall_post_impl_getresuid16(res, (long)(ruid), (long)(euid),   \
```
- **Line 871 / 第 871 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 872 / 第 872 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 873 / 第 873 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 874 / 第 874 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 875 / 第 875 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 876 / 第 876 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 877 / 第 877 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 878 / 第 878 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 879 / 第 879 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 880 / 第 880 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 881-890 / 第 881-890 行
```cpp
 881 |                                             (long)(suid))
 882 | #define __sanitizer_syscall_pre_setresgid16(rgid, egid, sgid)                  \
 883 |   __sanitizer_syscall_pre_impl_setresgid16((long)rgid, (long)egid, (long)sgid)
 884 | #define __sanitizer_syscall_post_setresgid16(res, rgid, egid, sgid)            \
 885 |   __sanitizer_syscall_post_impl_setresgid16(res, (long)rgid, (long)egid,       \
 886 |                                             (long)sgid)
 887 | #define __sanitizer_syscall_pre_getresgid16(rgid, egid, sgid)                  \
 888 |   __sanitizer_syscall_pre_impl_getresgid16((long)(rgid), (long)(egid),         \
 889 |                                            (long)(sgid))
 890 | #define __sanitizer_syscall_post_getresgid16(res, rgid, egid, sgid)            \
```
- **Line 881 / 第 881 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 882 / 第 882 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 883 / 第 883 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 884 / 第 884 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 885 / 第 885 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 886 / 第 886 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 887 / 第 887 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 888 / 第 888 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 889 / 第 889 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 890 / 第 890 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 891-900 / 第 891-900 行
```cpp
 891 |   __sanitizer_syscall_post_impl_getresgid16(res, (long)(rgid), (long)(egid),   \
 892 |                                             (long)(sgid))
 893 | #define __sanitizer_syscall_pre_setfsuid16(uid)                                \
 894 |   __sanitizer_syscall_pre_impl_setfsuid16((long)uid)
 895 | #define __sanitizer_syscall_post_setfsuid16(res, uid)                          \
 896 |   __sanitizer_syscall_post_impl_setfsuid16(res, (long)uid)
 897 | #define __sanitizer_syscall_pre_setfsgid16(gid)                                \
 898 |   __sanitizer_syscall_pre_impl_setfsgid16((long)gid)
 899 | #define __sanitizer_syscall_post_setfsgid16(res, gid)                          \
 900 |   __sanitizer_syscall_post_impl_setfsgid16(res, (long)gid)
```
- **Line 891 / 第 891 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 892 / 第 892 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 893 / 第 893 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 894 / 第 894 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 895 / 第 895 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 896 / 第 896 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 897 / 第 897 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 898 / 第 898 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 899 / 第 899 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 900 / 第 900 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 901-910 / 第 901-910 行
```cpp
 901 | #define __sanitizer_syscall_pre_getgroups16(gidsetsize, grouplist)             \
 902 |   __sanitizer_syscall_pre_impl_getgroups16((long)(gidsetsize),                 \
 903 |                                            (long)(grouplist))
 904 | #define __sanitizer_syscall_post_getgroups16(res, gidsetsize, grouplist)       \
 905 |   __sanitizer_syscall_post_impl_getgroups16(res, (long)(gidsetsize),           \
 906 |                                             (long)(grouplist))
 907 | #define __sanitizer_syscall_pre_setgroups16(gidsetsize, grouplist)             \
 908 |   __sanitizer_syscall_pre_impl_setgroups16((long)(gidsetsize),                 \
 909 |                                            (long)(grouplist))
 910 | #define __sanitizer_syscall_post_setgroups16(res, gidsetsize, grouplist)       \
```
- **Line 901 / 第 901 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 902 / 第 902 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 903 / 第 903 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 904 / 第 904 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 905 / 第 905 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 906 / 第 906 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 907 / 第 907 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 908 / 第 908 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 909 / 第 909 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 910 / 第 910 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 911-920 / 第 911-920 行
```cpp
 911 |   __sanitizer_syscall_post_impl_setgroups16(res, (long)(gidsetsize),           \
 912 |                                             (long)(grouplist))
 913 | #define __sanitizer_syscall_pre_getuid16()                                     \
 914 |   __sanitizer_syscall_pre_impl_getuid16()
 915 | #define __sanitizer_syscall_post_getuid16(res)                                 \
 916 |   __sanitizer_syscall_post_impl_getuid16(res)
 917 | #define __sanitizer_syscall_pre_geteuid16()                                    \
 918 |   __sanitizer_syscall_pre_impl_geteuid16()
 919 | #define __sanitizer_syscall_post_geteuid16(res)                                \
 920 |   __sanitizer_syscall_post_impl_geteuid16(res)
```
- **Line 911 / 第 911 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 912 / 第 912 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 913 / 第 913 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 914 / 第 914 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 915 / 第 915 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 916 / 第 916 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 917 / 第 917 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 918 / 第 918 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 919 / 第 919 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 920 / 第 920 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 921-930 / 第 921-930 行
```cpp
 921 | #define __sanitizer_syscall_pre_getgid16()                                     \
 922 |   __sanitizer_syscall_pre_impl_getgid16()
 923 | #define __sanitizer_syscall_post_getgid16(res)                                 \
 924 |   __sanitizer_syscall_post_impl_getgid16(res)
 925 | #define __sanitizer_syscall_pre_getegid16()                                    \
 926 |   __sanitizer_syscall_pre_impl_getegid16()
 927 | #define __sanitizer_syscall_post_getegid16(res)                                \
 928 |   __sanitizer_syscall_post_impl_getegid16(res)
 929 | #define __sanitizer_syscall_pre_utime(filename, times)                         \
 930 |   __sanitizer_syscall_pre_impl_utime((long)(filename), (long)(times))
```
- **Line 921 / 第 921 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 922 / 第 922 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 923 / 第 923 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 924 / 第 924 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 925 / 第 925 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 926 / 第 926 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 927 / 第 927 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 928 / 第 928 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 929 / 第 929 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 930 / 第 930 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 931-940 / 第 931-940 行
```cpp
 931 | #define __sanitizer_syscall_post_utime(res, filename, times)                   \
 932 |   __sanitizer_syscall_post_impl_utime(res, (long)(filename), (long)(times))
 933 | #define __sanitizer_syscall_pre_utimes(filename, utimes)                       \
 934 |   __sanitizer_syscall_pre_impl_utimes((long)(filename), (long)(utimes))
 935 | #define __sanitizer_syscall_post_utimes(res, filename, utimes)                 \
 936 |   __sanitizer_syscall_post_impl_utimes(res, (long)(filename), (long)(utimes))
 937 | #define __sanitizer_syscall_pre_lseek(fd, offset, origin)                      \
 938 |   __sanitizer_syscall_pre_impl_lseek((long)(fd), (long)(offset), (long)(origin))
 939 | #define __sanitizer_syscall_post_lseek(res, fd, offset, origin)                \
 940 |   __sanitizer_syscall_post_impl_lseek(res, (long)(fd), (long)(offset),         \
```
- **Line 931 / 第 931 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 932 / 第 932 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 933 / 第 933 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 934 / 第 934 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 935 / 第 935 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 936 / 第 936 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 937 / 第 937 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 938 / 第 938 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 939 / 第 939 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 940 / 第 940 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 941-950 / 第 941-950 行
```cpp
 941 |                                       (long)(origin))
 942 | #define __sanitizer_syscall_pre_llseek(fd, offset_high, offset_low, result,    \
 943 |                                        origin)                                 \
 944 |   __sanitizer_syscall_pre_impl_llseek((long)(fd), (long)(offset_high),         \
 945 |                                       (long)(offset_low), (long)(result),      \
 946 |                                       (long)(origin))
 947 | #define __sanitizer_syscall_post_llseek(res, fd, offset_high, offset_low,      \
 948 |                                         result, origin)                        \
 949 |   __sanitizer_syscall_post_impl_llseek(res, (long)(fd), (long)(offset_high),   \
 950 |                                        (long)(offset_low), (long)(result),     \
```
- **Line 941 / 第 941 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 942 / 第 942 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 943 / 第 943 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 944 / 第 944 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 945 / 第 945 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 946 / 第 946 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 947 / 第 947 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 948 / 第 948 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 949 / 第 949 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 950 / 第 950 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 951-960 / 第 951-960 行
```cpp
 951 |                                        (long)(origin))
 952 | #define __sanitizer_syscall_pre_read(fd, buf, count)                           \
 953 |   __sanitizer_syscall_pre_impl_read((long)(fd), (long)(buf), (long)(count))
 954 | #define __sanitizer_syscall_post_read(res, fd, buf, count)                     \
 955 |   __sanitizer_syscall_post_impl_read(res, (long)(fd), (long)(buf),             \
 956 |                                      (long)(count))
 957 | #define __sanitizer_syscall_pre_readv(fd, vec, vlen)                           \
 958 |   __sanitizer_syscall_pre_impl_readv((long)(fd), (long)(vec), (long)(vlen))
 959 | #define __sanitizer_syscall_post_readv(res, fd, vec, vlen)                     \
 960 |   __sanitizer_syscall_post_impl_readv(res, (long)(fd), (long)(vec),            \
```
- **Line 951 / 第 951 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 952 / 第 952 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 953 / 第 953 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 954 / 第 954 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 955 / 第 955 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 956 / 第 956 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 957 / 第 957 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 958 / 第 958 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 959 / 第 959 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 960 / 第 960 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 961-970 / 第 961-970 行
```cpp
 961 |                                       (long)(vlen))
 962 | #define __sanitizer_syscall_pre_write(fd, buf, count)                          \
 963 |   __sanitizer_syscall_pre_impl_write((long)(fd), (long)(buf), (long)(count))
 964 | #define __sanitizer_syscall_post_write(res, fd, buf, count)                    \
 965 |   __sanitizer_syscall_post_impl_write(res, (long)(fd), (long)(buf),            \
 966 |                                       (long)(count))
 967 | #define __sanitizer_syscall_pre_writev(fd, vec, vlen)                          \
 968 |   __sanitizer_syscall_pre_impl_writev((long)(fd), (long)(vec), (long)(vlen))
 969 | #define __sanitizer_syscall_post_writev(res, fd, vec, vlen)                    \
 970 |   __sanitizer_syscall_post_impl_writev(res, (long)(fd), (long)(vec),           \
```
- **Line 961 / 第 961 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 962 / 第 962 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 963 / 第 963 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 964 / 第 964 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 965 / 第 965 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 966 / 第 966 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 967 / 第 967 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 968 / 第 968 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 969 / 第 969 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 970 / 第 970 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 971-980 / 第 971-980 行
```cpp
 971 |                                        (long)(vlen))
 972 | 
 973 | #ifdef _LP64
 974 | #define __sanitizer_syscall_pre_pread64(fd, buf, count, pos)                   \
 975 |   __sanitizer_syscall_pre_impl_pread64((long)(fd), (long)(buf), (long)(count), \
 976 |                                        (long)(pos))
 977 | #define __sanitizer_syscall_post_pread64(res, fd, buf, count, pos)             \
 978 |   __sanitizer_syscall_post_impl_pread64(res, (long)(fd), (long)(buf),          \
 979 |                                         (long)(count), (long)(pos))
 980 | #define __sanitizer_syscall_pre_pwrite64(fd, buf, count, pos)                  \
```
- **Line 971 / 第 971 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 972 / 第 972 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 973 / 第 973 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 974 / 第 974 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 975 / 第 975 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 976 / 第 976 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 977 / 第 977 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 978 / 第 978 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 979 / 第 979 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 980 / 第 980 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 981-990 / 第 981-990 行
```cpp
 981 |   __sanitizer_syscall_pre_impl_pwrite64((long)(fd), (long)(buf),               \
 982 |                                         (long)(count), (long)(pos))
 983 | #define __sanitizer_syscall_post_pwrite64(res, fd, buf, count, pos)            \
 984 |   __sanitizer_syscall_post_impl_pwrite64(res, (long)(fd), (long)(buf),         \
 985 |                                          (long)(count), (long)(pos))
 986 | #else
 987 | #define __sanitizer_syscall_pre_pread64(fd, buf, count, pos0, pos1)            \
 988 |   __sanitizer_syscall_pre_impl_pread64((long)(fd), (long)(buf), (long)(count), \
 989 |                                        (long)(pos0), (long)(pos1))
 990 | #define __sanitizer_syscall_post_pread64(res, fd, buf, count, pos0, pos1)      \
```
- **Line 981 / 第 981 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 982 / 第 982 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 983 / 第 983 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 984 / 第 984 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 985 / 第 985 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 986 / 第 986 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 987 / 第 987 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 988 / 第 988 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 989 / 第 989 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 990 / 第 990 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 991-1000 / 第 991-1000 行
```cpp
 991 |   __sanitizer_syscall_post_impl_pread64(                                       \
 992 |       res, (long)(fd), (long)(buf), (long)(count), (long)(pos0), (long)(pos1))
 993 | #define __sanitizer_syscall_pre_pwrite64(fd, buf, count, pos0, pos1)           \
 994 |   __sanitizer_syscall_pre_impl_pwrite64(                                       \
 995 |       (long)(fd), (long)(buf), (long)(count), (long)(pos0), (long)(pos1))
 996 | #define __sanitizer_syscall_post_pwrite64(res, fd, buf, count, pos0, pos1)     \
 997 |   __sanitizer_syscall_post_impl_pwrite64(                                      \
 998 |       res, (long)(fd), (long)(buf), (long)(count), (long)(pos0), (long)(pos1))
 999 | #endif
1000 | 
```
- **Line 991 / 第 991 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 992 / 第 992 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 993 / 第 993 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 994 / 第 994 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 995 / 第 995 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 996 / 第 996 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 997 / 第 997 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 998 / 第 998 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 999 / 第 999 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 1000 / 第 1000 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 1001-1010 / 第 1001-1010 行
```cpp
1001 | #define __sanitizer_syscall_pre_preadv(fd, vec, vlen, pos_l, pos_h)            \
1002 |   __sanitizer_syscall_pre_impl_preadv((long)(fd), (long)(vec), (long)(vlen),   \
1003 |                                       (long)(pos_l), (long)(pos_h))
1004 | #define __sanitizer_syscall_post_preadv(res, fd, vec, vlen, pos_l, pos_h)      \
1005 |   __sanitizer_syscall_post_impl_preadv(res, (long)(fd), (long)(vec),           \
1006 |                                        (long)(vlen), (long)(pos_l),            \
1007 |                                        (long)(pos_h))
1008 | #define __sanitizer_syscall_pre_pwritev(fd, vec, vlen, pos_l, pos_h)           \
1009 |   __sanitizer_syscall_pre_impl_pwritev((long)(fd), (long)(vec), (long)(vlen),  \
1010 |                                        (long)(pos_l), (long)(pos_h))
```
- **Line 1001 / 第 1001 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1002 / 第 1002 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1003 / 第 1003 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1004 / 第 1004 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1005 / 第 1005 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1006 / 第 1006 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1007 / 第 1007 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1008 / 第 1008 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1009 / 第 1009 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1010 / 第 1010 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1011-1020 / 第 1011-1020 行
```cpp
1011 | #define __sanitizer_syscall_post_pwritev(res, fd, vec, vlen, pos_l, pos_h)     \
1012 |   __sanitizer_syscall_post_impl_pwritev(res, (long)(fd), (long)(vec),          \
1013 |                                         (long)(vlen), (long)(pos_l),           \
1014 |                                         (long)(pos_h))
1015 | #define __sanitizer_syscall_pre_getcwd(buf, size)                              \
1016 |   __sanitizer_syscall_pre_impl_getcwd((long)(buf), (long)(size))
1017 | #define __sanitizer_syscall_post_getcwd(res, buf, size)                        \
1018 |   __sanitizer_syscall_post_impl_getcwd(res, (long)(buf), (long)(size))
1019 | #define __sanitizer_syscall_pre_mkdir(pathname, mode)                          \
1020 |   __sanitizer_syscall_pre_impl_mkdir((long)(pathname), (long)(mode))
```
- **Line 1011 / 第 1011 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1012 / 第 1012 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1013 / 第 1013 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1014 / 第 1014 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1015 / 第 1015 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1016 / 第 1016 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1017 / 第 1017 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1018 / 第 1018 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1019 / 第 1019 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1020 / 第 1020 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1021-1030 / 第 1021-1030 行
```cpp
1021 | #define __sanitizer_syscall_post_mkdir(res, pathname, mode)                    \
1022 |   __sanitizer_syscall_post_impl_mkdir(res, (long)(pathname), (long)(mode))
1023 | #define __sanitizer_syscall_pre_chdir(filename)                                \
1024 |   __sanitizer_syscall_pre_impl_chdir((long)(filename))
1025 | #define __sanitizer_syscall_post_chdir(res, filename)                          \
1026 |   __sanitizer_syscall_post_impl_chdir(res, (long)(filename))
1027 | #define __sanitizer_syscall_pre_fchdir(fd)                                     \
1028 |   __sanitizer_syscall_pre_impl_fchdir((long)(fd))
1029 | #define __sanitizer_syscall_post_fchdir(res, fd)                               \
1030 |   __sanitizer_syscall_post_impl_fchdir(res, (long)(fd))
```
- **Line 1021 / 第 1021 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1022 / 第 1022 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1023 / 第 1023 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1024 / 第 1024 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1025 / 第 1025 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1026 / 第 1026 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1027 / 第 1027 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1028 / 第 1028 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1029 / 第 1029 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1030 / 第 1030 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1031-1040 / 第 1031-1040 行
```cpp
1031 | #define __sanitizer_syscall_pre_rmdir(pathname)                                \
1032 |   __sanitizer_syscall_pre_impl_rmdir((long)(pathname))
1033 | #define __sanitizer_syscall_post_rmdir(res, pathname)                          \
1034 |   __sanitizer_syscall_post_impl_rmdir(res, (long)(pathname))
1035 | #define __sanitizer_syscall_pre_lookup_dcookie(cookie64, buf, len)             \
1036 |   __sanitizer_syscall_pre_impl_lookup_dcookie((long)(cookie64), (long)(buf),   \
1037 |                                               (long)(len))
1038 | #define __sanitizer_syscall_post_lookup_dcookie(res, cookie64, buf, len)       \
1039 |   __sanitizer_syscall_post_impl_lookup_dcookie(res, (long)(cookie64),          \
1040 |                                                (long)(buf), (long)(len))
```
- **Line 1031 / 第 1031 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1032 / 第 1032 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1033 / 第 1033 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1034 / 第 1034 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1035 / 第 1035 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1036 / 第 1036 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1037 / 第 1037 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1038 / 第 1038 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1039 / 第 1039 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1040 / 第 1040 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1041-1050 / 第 1041-1050 行
```cpp
1041 | #define __sanitizer_syscall_pre_quotactl(cmd, special, id, addr)               \
1042 |   __sanitizer_syscall_pre_impl_quotactl((long)(cmd), (long)(special),          \
1043 |                                         (long)(id), (long)(addr))
1044 | #define __sanitizer_syscall_post_quotactl(res, cmd, special, id, addr)         \
1045 |   __sanitizer_syscall_post_impl_quotactl(res, (long)(cmd), (long)(special),    \
1046 |                                          (long)(id), (long)(addr))
1047 | #define __sanitizer_syscall_pre_getdents(fd, dirent, count)                    \
1048 |   __sanitizer_syscall_pre_impl_getdents((long)(fd), (long)(dirent),            \
1049 |                                         (long)(count))
1050 | #define __sanitizer_syscall_post_getdents(res, fd, dirent, count)              \
```
- **Line 1041 / 第 1041 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1042 / 第 1042 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1043 / 第 1043 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1044 / 第 1044 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1045 / 第 1045 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1046 / 第 1046 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1047 / 第 1047 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1048 / 第 1048 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1049 / 第 1049 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1050 / 第 1050 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1051-1060 / 第 1051-1060 行
```cpp
1051 |   __sanitizer_syscall_post_impl_getdents(res, (long)(fd), (long)(dirent),      \
1052 |                                          (long)(count))
1053 | #define __sanitizer_syscall_pre_getdents64(fd, dirent, count)                  \
1054 |   __sanitizer_syscall_pre_impl_getdents64((long)(fd), (long)(dirent),          \
1055 |                                           (long)(count))
1056 | #define __sanitizer_syscall_post_getdents64(res, fd, dirent, count)            \
1057 |   __sanitizer_syscall_post_impl_getdents64(res, (long)(fd), (long)(dirent),    \
1058 |                                            (long)(count))
1059 | #define __sanitizer_syscall_pre_setsockopt(fd, level, optname, optval, optlen) \
1060 |   __sanitizer_syscall_pre_impl_setsockopt((long)(fd), (long)(level),           \
```
- **Line 1051 / 第 1051 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1052 / 第 1052 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1053 / 第 1053 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1054 / 第 1054 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1055 / 第 1055 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1056 / 第 1056 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1057 / 第 1057 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1058 / 第 1058 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1059 / 第 1059 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1060 / 第 1060 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1061-1070 / 第 1061-1070 行
```cpp
1061 |                                           (long)(optname), (long)(optval),     \
1062 |                                           (long)(optlen))
1063 | #define __sanitizer_syscall_post_setsockopt(res, fd, level, optname, optval,   \
1064 |                                             optlen)                            \
1065 |   __sanitizer_syscall_post_impl_setsockopt(res, (long)(fd), (long)(level),     \
1066 |                                            (long)(optname), (long)(optval),    \
1067 |                                            (long)(optlen))
1068 | #define __sanitizer_syscall_pre_getsockopt(fd, level, optname, optval, optlen) \
1069 |   __sanitizer_syscall_pre_impl_getsockopt((long)(fd), (long)(level),           \
1070 |                                           (long)(optname), (long)(optval),     \
```
- **Line 1061 / 第 1061 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1062 / 第 1062 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1063 / 第 1063 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1064 / 第 1064 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1065 / 第 1065 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1066 / 第 1066 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1067 / 第 1067 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1068 / 第 1068 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1069 / 第 1069 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1070 / 第 1070 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1071-1080 / 第 1071-1080 行
```cpp
1071 |                                           (long)(optlen))
1072 | #define __sanitizer_syscall_post_getsockopt(res, fd, level, optname, optval,   \
1073 |                                             optlen)                            \
1074 |   __sanitizer_syscall_post_impl_getsockopt(res, (long)(fd), (long)(level),     \
1075 |                                            (long)(optname), (long)(optval),    \
1076 |                                            (long)(optlen))
1077 | #define __sanitizer_syscall_pre_bind(arg0, arg1, arg2)                         \
1078 |   __sanitizer_syscall_pre_impl_bind((long)(arg0), (long)(arg1), (long)(arg2))
1079 | #define __sanitizer_syscall_post_bind(res, arg0, arg1, arg2)                   \
1080 |   __sanitizer_syscall_post_impl_bind(res, (long)(arg0), (long)(arg1),          \
```
- **Line 1071 / 第 1071 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1072 / 第 1072 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1073 / 第 1073 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1074 / 第 1074 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1075 / 第 1075 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1076 / 第 1076 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1077 / 第 1077 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1078 / 第 1078 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1079 / 第 1079 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1080 / 第 1080 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1081-1090 / 第 1081-1090 行
```cpp
1081 |                                      (long)(arg2))
1082 | #define __sanitizer_syscall_pre_connect(arg0, arg1, arg2)                      \
1083 |   __sanitizer_syscall_pre_impl_connect((long)(arg0), (long)(arg1), (long)(arg2))
1084 | #define __sanitizer_syscall_post_connect(res, arg0, arg1, arg2)                \
1085 |   __sanitizer_syscall_post_impl_connect(res, (long)(arg0), (long)(arg1),       \
1086 |                                         (long)(arg2))
1087 | #define __sanitizer_syscall_pre_accept(arg0, arg1, arg2)                       \
1088 |   __sanitizer_syscall_pre_impl_accept((long)(arg0), (long)(arg1), (long)(arg2))
1089 | #define __sanitizer_syscall_post_accept(res, arg0, arg1, arg2)                 \
1090 |   __sanitizer_syscall_post_impl_accept(res, (long)(arg0), (long)(arg1),        \
```
- **Line 1081 / 第 1081 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1082 / 第 1082 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1083 / 第 1083 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1084 / 第 1084 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1085 / 第 1085 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1086 / 第 1086 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1087 / 第 1087 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1088 / 第 1088 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1089 / 第 1089 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1090 / 第 1090 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1091-1100 / 第 1091-1100 行
```cpp
1091 |                                        (long)(arg2))
1092 | #define __sanitizer_syscall_pre_accept4(arg0, arg1, arg2, arg3)                \
1093 |   __sanitizer_syscall_pre_impl_accept4((long)(arg0), (long)(arg1),             \
1094 |                                        (long)(arg2), (long)(arg3))
1095 | #define __sanitizer_syscall_post_accept4(res, arg0, arg1, arg2, arg3)          \
1096 |   __sanitizer_syscall_post_impl_accept4(res, (long)(arg0), (long)(arg1),       \
1097 |                                         (long)(arg2), (long)(arg3))
1098 | #define __sanitizer_syscall_pre_getsockname(arg0, arg1, arg2)                  \
1099 |   __sanitizer_syscall_pre_impl_getsockname((long)(arg0), (long)(arg1),         \
1100 |                                            (long)(arg2))
```
- **Line 1091 / 第 1091 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1092 / 第 1092 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1093 / 第 1093 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1094 / 第 1094 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1095 / 第 1095 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1096 / 第 1096 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1097 / 第 1097 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1098 / 第 1098 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1099 / 第 1099 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1100 / 第 1100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1101-1110 / 第 1101-1110 行
```cpp
1101 | #define __sanitizer_syscall_post_getsockname(res, arg0, arg1, arg2)            \
1102 |   __sanitizer_syscall_post_impl_getsockname(res, (long)(arg0), (long)(arg1),   \
1103 |                                             (long)(arg2))
1104 | #define __sanitizer_syscall_pre_getpeername(arg0, arg1, arg2)                  \
1105 |   __sanitizer_syscall_pre_impl_getpeername((long)(arg0), (long)(arg1),         \
1106 |                                            (long)(arg2))
1107 | #define __sanitizer_syscall_post_getpeername(res, arg0, arg1, arg2)            \
1108 |   __sanitizer_syscall_post_impl_getpeername(res, (long)(arg0), (long)(arg1),   \
1109 |                                             (long)(arg2))
1110 | #define __sanitizer_syscall_pre_send(arg0, arg1, arg2, arg3)                   \
```
- **Line 1101 / 第 1101 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1102 / 第 1102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1103 / 第 1103 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1104 / 第 1104 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1105 / 第 1105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1106 / 第 1106 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1107 / 第 1107 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1108 / 第 1108 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1109 / 第 1109 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1110 / 第 1110 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1111-1120 / 第 1111-1120 行
```cpp
1111 |   __sanitizer_syscall_pre_impl_send((long)(arg0), (long)(arg1), (long)(arg2),  \
1112 |                                     (long)(arg3))
1113 | #define __sanitizer_syscall_post_send(res, arg0, arg1, arg2, arg3)             \
1114 |   __sanitizer_syscall_post_impl_send(res, (long)(arg0), (long)(arg1),          \
1115 |                                      (long)(arg2), (long)(arg3))
1116 | #define __sanitizer_syscall_pre_sendto(arg0, arg1, arg2, arg3, arg4, arg5)     \
1117 |   __sanitizer_syscall_pre_impl_sendto((long)(arg0), (long)(arg1),              \
1118 |                                       (long)(arg2), (long)(arg3),              \
1119 |                                       (long)(arg4), (long)(arg5))
1120 | #define __sanitizer_syscall_post_sendto(res, arg0, arg1, arg2, arg3, arg4,     \
```
- **Line 1111 / 第 1111 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1112 / 第 1112 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1113 / 第 1113 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1114 / 第 1114 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1115 / 第 1115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1116 / 第 1116 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1117 / 第 1117 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1118 / 第 1118 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1119 / 第 1119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1120 / 第 1120 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1121-1130 / 第 1121-1130 行
```cpp
1121 |                                         arg5)                                  \
1122 |   __sanitizer_syscall_post_impl_sendto(res, (long)(arg0), (long)(arg1),        \
1123 |                                        (long)(arg2), (long)(arg3),             \
1124 |                                        (long)(arg4), (long)(arg5))
1125 | #define __sanitizer_syscall_pre_sendmsg(fd, msg, flags)                        \
1126 |   __sanitizer_syscall_pre_impl_sendmsg((long)(fd), (long)(msg), (long)(flags))
1127 | #define __sanitizer_syscall_post_sendmsg(res, fd, msg, flags)                  \
1128 |   __sanitizer_syscall_post_impl_sendmsg(res, (long)(fd), (long)(msg),          \
1129 |                                         (long)(flags))
1130 | #define __sanitizer_syscall_pre_sendmmsg(fd, msg, vlen, flags)                 \
```
- **Line 1121 / 第 1121 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1122 / 第 1122 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1123 / 第 1123 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1124 / 第 1124 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1125 / 第 1125 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1126 / 第 1126 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1127 / 第 1127 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1128 / 第 1128 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1129 / 第 1129 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1130 / 第 1130 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1131-1140 / 第 1131-1140 行
```cpp
1131 |   __sanitizer_syscall_pre_impl_sendmmsg((long)(fd), (long)(msg), (long)(vlen), \
1132 |                                         (long)(flags))
1133 | #define __sanitizer_syscall_post_sendmmsg(res, fd, msg, vlen, flags)           \
1134 |   __sanitizer_syscall_post_impl_sendmmsg(res, (long)(fd), (long)(msg),         \
1135 |                                          (long)(vlen), (long)(flags))
1136 | #define __sanitizer_syscall_pre_recv(arg0, arg1, arg2, arg3)                   \
1137 |   __sanitizer_syscall_pre_impl_recv((long)(arg0), (long)(arg1), (long)(arg2),  \
1138 |                                     (long)(arg3))
1139 | #define __sanitizer_syscall_post_recv(res, arg0, arg1, arg2, arg3)             \
1140 |   __sanitizer_syscall_post_impl_recv(res, (long)(arg0), (long)(arg1),          \
```
- **Line 1131 / 第 1131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1132 / 第 1132 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1133 / 第 1133 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1134 / 第 1134 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1135 / 第 1135 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1136 / 第 1136 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1137 / 第 1137 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1138 / 第 1138 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1139 / 第 1139 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1140 / 第 1140 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1141-1150 / 第 1141-1150 行
```cpp
1141 |                                      (long)(arg2), (long)(arg3))
1142 | #define __sanitizer_syscall_pre_recvfrom(arg0, arg1, arg2, arg3, arg4, arg5)   \
1143 |   __sanitizer_syscall_pre_impl_recvfrom((long)(arg0), (long)(arg1),            \
1144 |                                         (long)(arg2), (long)(arg3),            \
1145 |                                         (long)(arg4), (long)(arg5))
1146 | #define __sanitizer_syscall_post_recvfrom(res, arg0, arg1, arg2, arg3, arg4,   \
1147 |                                           arg5)                                \
1148 |   __sanitizer_syscall_post_impl_recvfrom(res, (long)(arg0), (long)(arg1),      \
1149 |                                          (long)(arg2), (long)(arg3),           \
1150 |                                          (long)(arg4), (long)(arg5))
```
- **Line 1141 / 第 1141 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1142 / 第 1142 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1143 / 第 1143 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1144 / 第 1144 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1145 / 第 1145 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1146 / 第 1146 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1147 / 第 1147 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1148 / 第 1148 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1149 / 第 1149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1150 / 第 1150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1151-1160 / 第 1151-1160 行
```cpp
1151 | #define __sanitizer_syscall_pre_recvmsg(fd, msg, flags)                        \
1152 |   __sanitizer_syscall_pre_impl_recvmsg((long)(fd), (long)(msg), (long)(flags))
1153 | #define __sanitizer_syscall_post_recvmsg(res, fd, msg, flags)                  \
1154 |   __sanitizer_syscall_post_impl_recvmsg(res, (long)(fd), (long)(msg),          \
1155 |                                         (long)(flags))
1156 | #define __sanitizer_syscall_pre_recvmmsg(fd, msg, vlen, flags, timeout)        \
1157 |   __sanitizer_syscall_pre_impl_recvmmsg((long)(fd), (long)(msg), (long)(vlen), \
1158 |                                         (long)(flags), (long)(timeout))
1159 | #define __sanitizer_syscall_post_recvmmsg(res, fd, msg, vlen, flags, timeout)  \
1160 |   __sanitizer_syscall_post_impl_recvmmsg(res, (long)(fd), (long)(msg),         \
```
- **Line 1151 / 第 1151 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1152 / 第 1152 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1153 / 第 1153 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1154 / 第 1154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1155 / 第 1155 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1156 / 第 1156 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1157 / 第 1157 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1158 / 第 1158 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1159 / 第 1159 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1160 / 第 1160 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1161-1170 / 第 1161-1170 行
```cpp
1161 |                                          (long)(vlen), (long)(flags),          \
1162 |                                          (long)(timeout))
1163 | #define __sanitizer_syscall_pre_socket(arg0, arg1, arg2)                       \
1164 |   __sanitizer_syscall_pre_impl_socket((long)(arg0), (long)(arg1), (long)(arg2))
1165 | #define __sanitizer_syscall_post_socket(res, arg0, arg1, arg2)                 \
1166 |   __sanitizer_syscall_post_impl_socket(res, (long)(arg0), (long)(arg1),        \
1167 |                                        (long)(arg2))
1168 | #define __sanitizer_syscall_pre_socketpair(arg0, arg1, arg2, arg3)             \
1169 |   __sanitizer_syscall_pre_impl_socketpair((long)(arg0), (long)(arg1),          \
1170 |                                           (long)(arg2), (long)(arg3))
```
- **Line 1161 / 第 1161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1162 / 第 1162 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1163 / 第 1163 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1164 / 第 1164 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1165 / 第 1165 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1166 / 第 1166 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1167 / 第 1167 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1168 / 第 1168 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1169 / 第 1169 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1170 / 第 1170 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1171-1180 / 第 1171-1180 行
```cpp
1171 | #define __sanitizer_syscall_post_socketpair(res, arg0, arg1, arg2, arg3)       \
1172 |   __sanitizer_syscall_post_impl_socketpair(res, (long)(arg0), (long)(arg1),    \
1173 |                                            (long)(arg2), (long)(arg3))
1174 | #define __sanitizer_syscall_pre_socketcall(call, args)                         \
1175 |   __sanitizer_syscall_pre_impl_socketcall((long)(call), (long)(args))
1176 | #define __sanitizer_syscall_post_socketcall(res, call, args)                   \
1177 |   __sanitizer_syscall_post_impl_socketcall(res, (long)(call), (long)(args))
1178 | #define __sanitizer_syscall_pre_listen(arg0, arg1)                             \
1179 |   __sanitizer_syscall_pre_impl_listen((long)(arg0), (long)(arg1))
1180 | #define __sanitizer_syscall_post_listen(res, arg0, arg1)                       \
```
- **Line 1171 / 第 1171 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1172 / 第 1172 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1173 / 第 1173 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1174 / 第 1174 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1175 / 第 1175 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1176 / 第 1176 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1177 / 第 1177 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1178 / 第 1178 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1179 / 第 1179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1180 / 第 1180 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1181-1190 / 第 1181-1190 行
```cpp
1181 |   __sanitizer_syscall_post_impl_listen(res, (long)(arg0), (long)(arg1))
1182 | #define __sanitizer_syscall_pre_poll(ufds, nfds, timeout)                      \
1183 |   __sanitizer_syscall_pre_impl_poll((long)(ufds), (long)(nfds), (long)(timeout))
1184 | #define __sanitizer_syscall_post_poll(res, ufds, nfds, timeout)                \
1185 |   __sanitizer_syscall_post_impl_poll(res, (long)(ufds), (long)(nfds),          \
1186 |                                      (long)(timeout))
1187 | #define __sanitizer_syscall_pre_select(n, inp, outp, exp, tvp)                 \
1188 |   __sanitizer_syscall_pre_impl_select((long)(n), (long)(inp), (long)(outp),    \
1189 |                                       (long)(exp), (long)(tvp))
1190 | #define __sanitizer_syscall_post_select(res, n, inp, outp, exp, tvp)           \
```
- **Line 1181 / 第 1181 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1182 / 第 1182 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1183 / 第 1183 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1184 / 第 1184 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1185 / 第 1185 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1186 / 第 1186 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1187 / 第 1187 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1188 / 第 1188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1189 / 第 1189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1190 / 第 1190 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1191-1200 / 第 1191-1200 行
```cpp
1191 |   __sanitizer_syscall_post_impl_select(res, (long)(n), (long)(inp),            \
1192 |                                        (long)(outp), (long)(exp), (long)(tvp))
1193 | #define __sanitizer_syscall_pre_old_select(arg)                                \
1194 |   __sanitizer_syscall_pre_impl_old_select((long)(arg))
1195 | #define __sanitizer_syscall_post_old_select(res, arg)                          \
1196 |   __sanitizer_syscall_post_impl_old_select(res, (long)(arg))
1197 | #define __sanitizer_syscall_pre_epoll_create(size)                             \
1198 |   __sanitizer_syscall_pre_impl_epoll_create((long)(size))
1199 | #define __sanitizer_syscall_post_epoll_create(res, size)                       \
1200 |   __sanitizer_syscall_post_impl_epoll_create(res, (long)(size))
```
- **Line 1191 / 第 1191 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1192 / 第 1192 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1193 / 第 1193 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1194 / 第 1194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1195 / 第 1195 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1196 / 第 1196 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1197 / 第 1197 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1198 / 第 1198 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1199 / 第 1199 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1200 / 第 1200 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1201-1210 / 第 1201-1210 行
```cpp
1201 | #define __sanitizer_syscall_pre_epoll_create1(flags)                           \
1202 |   __sanitizer_syscall_pre_impl_epoll_create1((long)(flags))
1203 | #define __sanitizer_syscall_post_epoll_create1(res, flags)                     \
1204 |   __sanitizer_syscall_post_impl_epoll_create1(res, (long)(flags))
1205 | #define __sanitizer_syscall_pre_epoll_ctl(epfd, op, fd, event)                 \
1206 |   __sanitizer_syscall_pre_impl_epoll_ctl((long)(epfd), (long)(op), (long)(fd), \
1207 |                                          (long)(event))
1208 | #define __sanitizer_syscall_post_epoll_ctl(res, epfd, op, fd, event)           \
1209 |   __sanitizer_syscall_post_impl_epoll_ctl(res, (long)(epfd), (long)(op),       \
1210 |                                           (long)(fd), (long)(event))
```
- **Line 1201 / 第 1201 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1202 / 第 1202 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1203 / 第 1203 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1204 / 第 1204 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1205 / 第 1205 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1206 / 第 1206 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1207 / 第 1207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1208 / 第 1208 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1209 / 第 1209 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1210 / 第 1210 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1211-1220 / 第 1211-1220 行
```cpp
1211 | #define __sanitizer_syscall_pre_epoll_wait(epfd, events, maxevents, timeout)   \
1212 |   __sanitizer_syscall_pre_impl_epoll_wait((long)(epfd), (long)(events),        \
1213 |                                           (long)(maxevents), (long)(timeout))
1214 | #define __sanitizer_syscall_post_epoll_wait(res, epfd, events, maxevents,      \
1215 |                                             timeout)                           \
1216 |   __sanitizer_syscall_post_impl_epoll_wait(res, (long)(epfd), (long)(events),  \
1217 |                                            (long)(maxevents), (long)(timeout))
1218 | #define __sanitizer_syscall_pre_epoll_pwait(epfd, events, maxevents, timeout,  \
1219 |                                             sigmask, sigsetsize)               \
1220 |   __sanitizer_syscall_pre_impl_epoll_pwait(                                    \
```
- **Line 1211 / 第 1211 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1212 / 第 1212 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1213 / 第 1213 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1214 / 第 1214 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1215 / 第 1215 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1216 / 第 1216 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1217 / 第 1217 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1218 / 第 1218 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1219 / 第 1219 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1220 / 第 1220 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1221-1230 / 第 1221-1230 行
```cpp
1221 |       (long)(epfd), (long)(events), (long)(maxevents), (long)(timeout),        \
1222 |       (long)(sigmask), (long)(sigsetsize))
1223 | #define __sanitizer_syscall_post_epoll_pwait(res, epfd, events, maxevents,     \
1224 |                                              timeout, sigmask, sigsetsize)     \
1225 |   __sanitizer_syscall_post_impl_epoll_pwait(                                   \
1226 |       res, (long)(epfd), (long)(events), (long)(maxevents), (long)(timeout),   \
1227 |       (long)(sigmask), (long)(sigsetsize))
1228 | #define __sanitizer_syscall_pre_epoll_pwait2(epfd, events, maxevents, timeout, \
1229 |                                              sigmask, sigsetsize)              \
1230 |   __sanitizer_syscall_pre_impl_epoll_pwait2(                                   \
```
- **Line 1221 / 第 1221 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1222 / 第 1222 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1223 / 第 1223 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1224 / 第 1224 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1225 / 第 1225 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1226 / 第 1226 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1227 / 第 1227 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1228 / 第 1228 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1229 / 第 1229 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1230 / 第 1230 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1231-1240 / 第 1231-1240 行
```cpp
1231 |       (long)(epfd), (long)(events), (long)(maxevents), (long)(timeout),        \
1232 |       (long)(sigmask), (long)(sigsetsize))
1233 | #define __sanitizer_syscall_post_epoll_pwait2(res, epfd, events, maxevents,    \
1234 |                                               timeout, sigmask, sigsetsize)    \
1235 |   __sanitizer_syscall_post_impl_epoll_pwait2(                                  \
1236 |       res, (long)(epfd), (long)(events), (long)(maxevents), (long)(timeout),   \
1237 |       (long)(sigmask), (long)(sigsetsize))
1238 | #define __sanitizer_syscall_pre_gethostname(name, len)                         \
1239 |   __sanitizer_syscall_pre_impl_gethostname((long)(name), (long)(len))
1240 | #define __sanitizer_syscall_post_gethostname(res, name, len)                   \
```
- **Line 1231 / 第 1231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1232 / 第 1232 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1233 / 第 1233 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1234 / 第 1234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1235 / 第 1235 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1236 / 第 1236 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1237 / 第 1237 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1238 / 第 1238 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1239 / 第 1239 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1240 / 第 1240 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1241-1250 / 第 1241-1250 行
```cpp
1241 |   __sanitizer_syscall_post_impl_gethostname(res, (long)(name), (long)(len))
1242 | #define __sanitizer_syscall_pre_sethostname(name, len)                         \
1243 |   __sanitizer_syscall_pre_impl_sethostname((long)(name), (long)(len))
1244 | #define __sanitizer_syscall_post_sethostname(res, name, len)                   \
1245 |   __sanitizer_syscall_post_impl_sethostname(res, (long)(name), (long)(len))
1246 | #define __sanitizer_syscall_pre_setdomainname(name, len)                       \
1247 |   __sanitizer_syscall_pre_impl_setdomainname((long)(name), (long)(len))
1248 | #define __sanitizer_syscall_post_setdomainname(res, name, len)                 \
1249 |   __sanitizer_syscall_post_impl_setdomainname(res, (long)(name), (long)(len))
1250 | #define __sanitizer_syscall_pre_newuname(name)                                 \
```
- **Line 1241 / 第 1241 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1242 / 第 1242 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1243 / 第 1243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1244 / 第 1244 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1245 / 第 1245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1246 / 第 1246 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1247 / 第 1247 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1248 / 第 1248 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1249 / 第 1249 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1250 / 第 1250 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1251-1260 / 第 1251-1260 行
```cpp
1251 |   __sanitizer_syscall_pre_impl_newuname((long)(name))
1252 | #define __sanitizer_syscall_post_newuname(res, name)                           \
1253 |   __sanitizer_syscall_post_impl_newuname(res, (long)(name))
1254 | #define __sanitizer_syscall_pre_uname(arg0)                                    \
1255 |   __sanitizer_syscall_pre_impl_uname((long)(arg0))
1256 | #define __sanitizer_syscall_post_uname(res, arg0)                              \
1257 |   __sanitizer_syscall_post_impl_uname(res, (long)(arg0))
1258 | #define __sanitizer_syscall_pre_olduname(arg0)                                 \
1259 |   __sanitizer_syscall_pre_impl_olduname((long)(arg0))
1260 | #define __sanitizer_syscall_post_olduname(res, arg0)                           \
```
- **Line 1251 / 第 1251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1252 / 第 1252 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1253 / 第 1253 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1254 / 第 1254 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1255 / 第 1255 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1256 / 第 1256 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1257 / 第 1257 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1258 / 第 1258 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1259 / 第 1259 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1260 / 第 1260 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1261-1270 / 第 1261-1270 行
```cpp
1261 |   __sanitizer_syscall_post_impl_olduname(res, (long)(arg0))
1262 | #define __sanitizer_syscall_pre_getrlimit(resource, rlim)                      \
1263 |   __sanitizer_syscall_pre_impl_getrlimit((long)(resource), (long)(rlim))
1264 | #define __sanitizer_syscall_post_getrlimit(res, resource, rlim)                \
1265 |   __sanitizer_syscall_post_impl_getrlimit(res, (long)(resource), (long)(rlim))
1266 | #define __sanitizer_syscall_pre_old_getrlimit(resource, rlim)                  \
1267 |   __sanitizer_syscall_pre_impl_old_getrlimit((long)(resource), (long)(rlim))
1268 | #define __sanitizer_syscall_post_old_getrlimit(res, resource, rlim)            \
1269 |   __sanitizer_syscall_post_impl_old_getrlimit(res, (long)(resource),           \
1270 |                                               (long)(rlim))
```
- **Line 1261 / 第 1261 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1262 / 第 1262 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1263 / 第 1263 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1264 / 第 1264 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1265 / 第 1265 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1266 / 第 1266 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1267 / 第 1267 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1268 / 第 1268 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1269 / 第 1269 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1270 / 第 1270 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1271-1280 / 第 1271-1280 行
```cpp
1271 | #define __sanitizer_syscall_pre_setrlimit(resource, rlim)                      \
1272 |   __sanitizer_syscall_pre_impl_setrlimit((long)(resource), (long)(rlim))
1273 | #define __sanitizer_syscall_post_setrlimit(res, resource, rlim)                \
1274 |   __sanitizer_syscall_post_impl_setrlimit(res, (long)(resource), (long)(rlim))
1275 | #define __sanitizer_syscall_pre_prlimit64(pid, resource, new_rlim, old_rlim)   \
1276 |   __sanitizer_syscall_pre_impl_prlimit64((long)(pid), (long)(resource),        \
1277 |                                          (long)(new_rlim), (long)(old_rlim))
1278 | #define __sanitizer_syscall_post_prlimit64(res, pid, resource, new_rlim,       \
1279 |                                            old_rlim)                           \
1280 |   __sanitizer_syscall_post_impl_prlimit64(res, (long)(pid), (long)(resource),  \
```
- **Line 1271 / 第 1271 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1272 / 第 1272 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1273 / 第 1273 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1274 / 第 1274 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1275 / 第 1275 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1276 / 第 1276 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1277 / 第 1277 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1278 / 第 1278 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1279 / 第 1279 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1280 / 第 1280 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1281-1290 / 第 1281-1290 行
```cpp
1281 |                                           (long)(new_rlim), (long)(old_rlim))
1282 | #define __sanitizer_syscall_pre_getrusage(who, ru)                             \
1283 |   __sanitizer_syscall_pre_impl_getrusage((long)(who), (long)(ru))
1284 | #define __sanitizer_syscall_post_getrusage(res, who, ru)                       \
1285 |   __sanitizer_syscall_post_impl_getrusage(res, (long)(who), (long)(ru))
1286 | #define __sanitizer_syscall_pre_umask(mask)                                    \
1287 |   __sanitizer_syscall_pre_impl_umask((long)(mask))
1288 | #define __sanitizer_syscall_post_umask(res, mask)                              \
1289 |   __sanitizer_syscall_post_impl_umask(res, (long)(mask))
1290 | #define __sanitizer_syscall_pre_msgget(key, msgflg)                            \
```
- **Line 1281 / 第 1281 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1282 / 第 1282 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1283 / 第 1283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1284 / 第 1284 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1285 / 第 1285 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1286 / 第 1286 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1287 / 第 1287 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1288 / 第 1288 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1289 / 第 1289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1290 / 第 1290 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1291-1300 / 第 1291-1300 行
```cpp
1291 |   __sanitizer_syscall_pre_impl_msgget((long)(key), (long)(msgflg))
1292 | #define __sanitizer_syscall_post_msgget(res, key, msgflg)                      \
1293 |   __sanitizer_syscall_post_impl_msgget(res, (long)(key), (long)(msgflg))
1294 | #define __sanitizer_syscall_pre_msgsnd(msqid, msgp, msgsz, msgflg)             \
1295 |   __sanitizer_syscall_pre_impl_msgsnd((long)(msqid), (long)(msgp),             \
1296 |                                       (long)(msgsz), (long)(msgflg))
1297 | #define __sanitizer_syscall_post_msgsnd(res, msqid, msgp, msgsz, msgflg)       \
1298 |   __sanitizer_syscall_post_impl_msgsnd(res, (long)(msqid), (long)(msgp),       \
1299 |                                        (long)(msgsz), (long)(msgflg))
1300 | #define __sanitizer_syscall_pre_msgrcv(msqid, msgp, msgsz, msgtyp, msgflg)     \
```
- **Line 1291 / 第 1291 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1292 / 第 1292 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1293 / 第 1293 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1294 / 第 1294 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1295 / 第 1295 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1296 / 第 1296 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1297 / 第 1297 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1298 / 第 1298 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1299 / 第 1299 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1300 / 第 1300 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1301-1310 / 第 1301-1310 行
```cpp
1301 |   __sanitizer_syscall_pre_impl_msgrcv((long)(msqid), (long)(msgp),             \
1302 |                                       (long)(msgsz), (long)(msgtyp),           \
1303 |                                       (long)(msgflg))
1304 | #define __sanitizer_syscall_post_msgrcv(res, msqid, msgp, msgsz, msgtyp,       \
1305 |                                         msgflg)                                \
1306 |   __sanitizer_syscall_post_impl_msgrcv(res, (long)(msqid), (long)(msgp),       \
1307 |                                        (long)(msgsz), (long)(msgtyp),          \
1308 |                                        (long)(msgflg))
1309 | #define __sanitizer_syscall_pre_msgctl(msqid, cmd, buf)                        \
1310 |   __sanitizer_syscall_pre_impl_msgctl((long)(msqid), (long)(cmd), (long)(buf))
```
- **Line 1301 / 第 1301 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1302 / 第 1302 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1303 / 第 1303 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1304 / 第 1304 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1305 / 第 1305 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1306 / 第 1306 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1307 / 第 1307 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1308 / 第 1308 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1309 / 第 1309 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1310 / 第 1310 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1311-1320 / 第 1311-1320 行
```cpp
1311 | #define __sanitizer_syscall_post_msgctl(res, msqid, cmd, buf)                  \
1312 |   __sanitizer_syscall_post_impl_msgctl(res, (long)(msqid), (long)(cmd),        \
1313 |                                        (long)(buf))
1314 | #define __sanitizer_syscall_pre_semget(key, nsems, semflg)                     \
1315 |   __sanitizer_syscall_pre_impl_semget((long)(key), (long)(nsems),              \
1316 |                                       (long)(semflg))
1317 | #define __sanitizer_syscall_post_semget(res, key, nsems, semflg)               \
1318 |   __sanitizer_syscall_post_impl_semget(res, (long)(key), (long)(nsems),        \
1319 |                                        (long)(semflg))
1320 | #define __sanitizer_syscall_pre_semop(semid, sops, nsops)                      \
```
- **Line 1311 / 第 1311 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1312 / 第 1312 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1313 / 第 1313 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1314 / 第 1314 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1315 / 第 1315 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1316 / 第 1316 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1317 / 第 1317 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1318 / 第 1318 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1319 / 第 1319 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1320 / 第 1320 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1321-1330 / 第 1321-1330 行
```cpp
1321 |   __sanitizer_syscall_pre_impl_semop((long)(semid), (long)(sops), (long)(nsops))
1322 | #define __sanitizer_syscall_post_semop(res, semid, sops, nsops)                \
1323 |   __sanitizer_syscall_post_impl_semop(res, (long)(semid), (long)(sops),        \
1324 |                                       (long)(nsops))
1325 | #define __sanitizer_syscall_pre_semctl(semid, semnum, cmd, arg)                \
1326 |   __sanitizer_syscall_pre_impl_semctl((long)(semid), (long)(semnum),           \
1327 |                                       (long)(cmd), (long)(arg))
1328 | #define __sanitizer_syscall_post_semctl(res, semid, semnum, cmd, arg)          \
1329 |   __sanitizer_syscall_post_impl_semctl(res, (long)(semid), (long)(semnum),     \
1330 |                                        (long)(cmd), (long)(arg))
```
- **Line 1321 / 第 1321 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1322 / 第 1322 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1323 / 第 1323 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1324 / 第 1324 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1325 / 第 1325 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1326 / 第 1326 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1327 / 第 1327 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1328 / 第 1328 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1329 / 第 1329 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1330 / 第 1330 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1331-1340 / 第 1331-1340 行
```cpp
1331 | #define __sanitizer_syscall_pre_semtimedop(semid, sops, nsops, timeout)        \
1332 |   __sanitizer_syscall_pre_impl_semtimedop((long)(semid), (long)(sops),         \
1333 |                                           (long)(nsops), (long)(timeout))
1334 | #define __sanitizer_syscall_post_semtimedop(res, semid, sops, nsops, timeout)  \
1335 |   __sanitizer_syscall_post_impl_semtimedop(res, (long)(semid), (long)(sops),   \
1336 |                                            (long)(nsops), (long)(timeout))
1337 | #define __sanitizer_syscall_pre_shmat(shmid, shmaddr, shmflg)                  \
1338 |   __sanitizer_syscall_pre_impl_shmat((long)(shmid), (long)(shmaddr),           \
1339 |                                      (long)(shmflg))
1340 | #define __sanitizer_syscall_post_shmat(res, shmid, shmaddr, shmflg)            \
```
- **Line 1331 / 第 1331 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1332 / 第 1332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1333 / 第 1333 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1334 / 第 1334 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1335 / 第 1335 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1336 / 第 1336 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1337 / 第 1337 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1338 / 第 1338 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1339 / 第 1339 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1340 / 第 1340 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1341-1350 / 第 1341-1350 行
```cpp
1341 |   __sanitizer_syscall_post_impl_shmat(res, (long)(shmid), (long)(shmaddr),     \
1342 |                                       (long)(shmflg))
1343 | #define __sanitizer_syscall_pre_shmget(key, size, flag)                        \
1344 |   __sanitizer_syscall_pre_impl_shmget((long)(key), (long)(size), (long)(flag))
1345 | #define __sanitizer_syscall_post_shmget(res, key, size, flag)                  \
1346 |   __sanitizer_syscall_post_impl_shmget(res, (long)(key), (long)(size),         \
1347 |                                        (long)(flag))
1348 | #define __sanitizer_syscall_pre_shmdt(shmaddr)                                 \
1349 |   __sanitizer_syscall_pre_impl_shmdt((long)(shmaddr))
1350 | #define __sanitizer_syscall_post_shmdt(res, shmaddr)                           \
```
- **Line 1341 / 第 1341 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1342 / 第 1342 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1343 / 第 1343 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1344 / 第 1344 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1345 / 第 1345 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1346 / 第 1346 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1347 / 第 1347 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1348 / 第 1348 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1349 / 第 1349 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1350 / 第 1350 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1351-1360 / 第 1351-1360 行
```cpp
1351 |   __sanitizer_syscall_post_impl_shmdt(res, (long)(shmaddr))
1352 | #define __sanitizer_syscall_pre_shmctl(shmid, cmd, buf)                        \
1353 |   __sanitizer_syscall_pre_impl_shmctl((long)(shmid), (long)(cmd), (long)(buf))
1354 | #define __sanitizer_syscall_post_shmctl(res, shmid, cmd, buf)                  \
1355 |   __sanitizer_syscall_post_impl_shmctl(res, (long)(shmid), (long)(cmd),        \
1356 |                                        (long)(buf))
1357 | #define __sanitizer_syscall_pre_ipc(call, first, second, third, ptr, fifth)    \
1358 |   __sanitizer_syscall_pre_impl_ipc((long)(call), (long)(first),                \
1359 |                                    (long)(second), (long)(third), (long)(ptr), \
1360 |                                    (long)(fifth))
```
- **Line 1351 / 第 1351 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1352 / 第 1352 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1353 / 第 1353 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1354 / 第 1354 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1355 / 第 1355 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1356 / 第 1356 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1357 / 第 1357 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1358 / 第 1358 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1359 / 第 1359 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1360 / 第 1360 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1361-1370 / 第 1361-1370 行
```cpp
1361 | #define __sanitizer_syscall_post_ipc(res, call, first, second, third, ptr,     \
1362 |                                      fifth)                                    \
1363 |   __sanitizer_syscall_post_impl_ipc(res, (long)(call), (long)(first),          \
1364 |                                     (long)(second), (long)(third),             \
1365 |                                     (long)(ptr), (long)(fifth))
1366 | #define __sanitizer_syscall_pre_mq_open(name, oflag, mode, attr)               \
1367 |   __sanitizer_syscall_pre_impl_mq_open((long)(name), (long)(oflag),            \
1368 |                                        (long)(mode), (long)(attr))
1369 | #define __sanitizer_syscall_post_mq_open(res, name, oflag, mode, attr)         \
1370 |   __sanitizer_syscall_post_impl_mq_open(res, (long)(name), (long)(oflag),      \
```
- **Line 1361 / 第 1361 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1362 / 第 1362 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1363 / 第 1363 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1364 / 第 1364 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1365 / 第 1365 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1366 / 第 1366 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1367 / 第 1367 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1368 / 第 1368 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1369 / 第 1369 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1370 / 第 1370 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1371-1380 / 第 1371-1380 行
```cpp
1371 |                                         (long)(mode), (long)(attr))
1372 | #define __sanitizer_syscall_pre_mq_unlink(name)                                \
1373 |   __sanitizer_syscall_pre_impl_mq_unlink((long)(name))
1374 | #define __sanitizer_syscall_post_mq_unlink(res, name)                          \
1375 |   __sanitizer_syscall_post_impl_mq_unlink(res, (long)(name))
1376 | #define __sanitizer_syscall_pre_mq_timedsend(mqdes, msg_ptr, msg_len,          \
1377 |                                              msg_prio, abs_timeout)            \
1378 |   __sanitizer_syscall_pre_impl_mq_timedsend((long)(mqdes), (long)(msg_ptr),    \
1379 |                                             (long)(msg_len), (long)(msg_prio), \
1380 |                                             (long)(abs_timeout))
```
- **Line 1371 / 第 1371 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1372 / 第 1372 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1373 / 第 1373 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1374 / 第 1374 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1375 / 第 1375 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1376 / 第 1376 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1377 / 第 1377 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1378 / 第 1378 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1379 / 第 1379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1380 / 第 1380 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1381-1390 / 第 1381-1390 行
```cpp
1381 | #define __sanitizer_syscall_post_mq_timedsend(res, mqdes, msg_ptr, msg_len,    \
1382 |                                               msg_prio, abs_timeout)           \
1383 |   __sanitizer_syscall_post_impl_mq_timedsend(                                  \
1384 |       res, (long)(mqdes), (long)(msg_ptr), (long)(msg_len), (long)(msg_prio),  \
1385 |       (long)(abs_timeout))
1386 | #define __sanitizer_syscall_pre_mq_timedreceive(mqdes, msg_ptr, msg_len,       \
1387 |                                                 msg_prio, abs_timeout)         \
1388 |   __sanitizer_syscall_pre_impl_mq_timedreceive(                                \
1389 |       (long)(mqdes), (long)(msg_ptr), (long)(msg_len), (long)(msg_prio),       \
1390 |       (long)(abs_timeout))
```
- **Line 1381 / 第 1381 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1382 / 第 1382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1383 / 第 1383 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1384 / 第 1384 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1385 / 第 1385 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1386 / 第 1386 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1387 / 第 1387 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1388 / 第 1388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1389 / 第 1389 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1390 / 第 1390 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1391-1400 / 第 1391-1400 行
```cpp
1391 | #define __sanitizer_syscall_post_mq_timedreceive(res, mqdes, msg_ptr, msg_len, \
1392 |                                                  msg_prio, abs_timeout)        \
1393 |   __sanitizer_syscall_post_impl_mq_timedreceive(                               \
1394 |       res, (long)(mqdes), (long)(msg_ptr), (long)(msg_len), (long)(msg_prio),  \
1395 |       (long)(abs_timeout))
1396 | #define __sanitizer_syscall_pre_mq_notify(mqdes, notification)                 \
1397 |   __sanitizer_syscall_pre_impl_mq_notify((long)(mqdes), (long)(notification))
1398 | #define __sanitizer_syscall_post_mq_notify(res, mqdes, notification)           \
1399 |   __sanitizer_syscall_post_impl_mq_notify(res, (long)(mqdes),                  \
1400 |                                           (long)(notification))
```
- **Line 1391 / 第 1391 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1392 / 第 1392 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1393 / 第 1393 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1394 / 第 1394 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1395 / 第 1395 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1396 / 第 1396 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1397 / 第 1397 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1398 / 第 1398 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1399 / 第 1399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1400 / 第 1400 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1401-1410 / 第 1401-1410 行
```cpp
1401 | #define __sanitizer_syscall_pre_mq_getsetattr(mqdes, mqstat, omqstat)          \
1402 |   __sanitizer_syscall_pre_impl_mq_getsetattr((long)(mqdes), (long)(mqstat),    \
1403 |                                              (long)(omqstat))
1404 | #define __sanitizer_syscall_post_mq_getsetattr(res, mqdes, mqstat, omqstat)    \
1405 |   __sanitizer_syscall_post_impl_mq_getsetattr(res, (long)(mqdes),              \
1406 |                                               (long)(mqstat), (long)(omqstat))
1407 | #define __sanitizer_syscall_pre_pciconfig_iobase(which, bus, devfn)            \
1408 |   __sanitizer_syscall_pre_impl_pciconfig_iobase((long)(which), (long)(bus),    \
1409 |                                                 (long)(devfn))
1410 | #define __sanitizer_syscall_post_pciconfig_iobase(res, which, bus, devfn)      \
```
- **Line 1401 / 第 1401 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1402 / 第 1402 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1403 / 第 1403 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1404 / 第 1404 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1405 / 第 1405 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1406 / 第 1406 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1407 / 第 1407 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1408 / 第 1408 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1409 / 第 1409 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1410 / 第 1410 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1411-1420 / 第 1411-1420 行
```cpp
1411 |   __sanitizer_syscall_post_impl_pciconfig_iobase(res, (long)(which),           \
1412 |                                                  (long)(bus), (long)(devfn))
1413 | #define __sanitizer_syscall_pre_pciconfig_read(bus, dfn, off, len, buf)        \
1414 |   __sanitizer_syscall_pre_impl_pciconfig_read(                                 \
1415 |       (long)(bus), (long)(dfn), (long)(off), (long)(len), (long)(buf))
1416 | #define __sanitizer_syscall_post_pciconfig_read(res, bus, dfn, off, len, buf)  \
1417 |   __sanitizer_syscall_post_impl_pciconfig_read(                                \
1418 |       res, (long)(bus), (long)(dfn), (long)(off), (long)(len), (long)(buf))
1419 | #define __sanitizer_syscall_pre_pciconfig_write(bus, dfn, off, len, buf)       \
1420 |   __sanitizer_syscall_pre_impl_pciconfig_write(                                \
```
- **Line 1411 / 第 1411 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1412 / 第 1412 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1413 / 第 1413 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1414 / 第 1414 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1415 / 第 1415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1416 / 第 1416 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1417 / 第 1417 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1418 / 第 1418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1419 / 第 1419 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1420 / 第 1420 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1421-1430 / 第 1421-1430 行
```cpp
1421 |       (long)(bus), (long)(dfn), (long)(off), (long)(len), (long)(buf))
1422 | #define __sanitizer_syscall_post_pciconfig_write(res, bus, dfn, off, len, buf) \
1423 |   __sanitizer_syscall_post_impl_pciconfig_write(                               \
1424 |       res, (long)(bus), (long)(dfn), (long)(off), (long)(len), (long)(buf))
1425 | #define __sanitizer_syscall_pre_swapon(specialfile, swap_flags)                \
1426 |   __sanitizer_syscall_pre_impl_swapon((long)(specialfile), (long)(swap_flags))
1427 | #define __sanitizer_syscall_post_swapon(res, specialfile, swap_flags)          \
1428 |   __sanitizer_syscall_post_impl_swapon(res, (long)(specialfile),               \
1429 |                                        (long)(swap_flags))
1430 | #define __sanitizer_syscall_pre_swapoff(specialfile)                           \
```
- **Line 1421 / 第 1421 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1422 / 第 1422 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1423 / 第 1423 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1424 / 第 1424 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1425 / 第 1425 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1426 / 第 1426 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1427 / 第 1427 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1428 / 第 1428 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1429 / 第 1429 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1430 / 第 1430 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1431-1440 / 第 1431-1440 行
```cpp
1431 |   __sanitizer_syscall_pre_impl_swapoff((long)(specialfile))
1432 | #define __sanitizer_syscall_post_swapoff(res, specialfile)                     \
1433 |   __sanitizer_syscall_post_impl_swapoff(res, (long)(specialfile))
1434 | #define __sanitizer_syscall_pre_sysctl(args)                                   \
1435 |   __sanitizer_syscall_pre_impl_sysctl((long)(args))
1436 | #define __sanitizer_syscall_post_sysctl(res, args)                             \
1437 |   __sanitizer_syscall_post_impl_sysctl(res, (long)(args))
1438 | #define __sanitizer_syscall_pre_sysinfo(info)                                  \
1439 |   __sanitizer_syscall_pre_impl_sysinfo((long)(info))
1440 | #define __sanitizer_syscall_post_sysinfo(res, info)                            \
```
- **Line 1431 / 第 1431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1432 / 第 1432 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1433 / 第 1433 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1434 / 第 1434 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1435 / 第 1435 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1436 / 第 1436 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1437 / 第 1437 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1438 / 第 1438 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1439 / 第 1439 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1440 / 第 1440 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1441-1450 / 第 1441-1450 行
```cpp
1441 |   __sanitizer_syscall_post_impl_sysinfo(res, (long)(info))
1442 | #define __sanitizer_syscall_pre_sysfs(option, arg1, arg2)                      \
1443 |   __sanitizer_syscall_pre_impl_sysfs((long)(option), (long)(arg1), (long)(arg2))
1444 | #define __sanitizer_syscall_post_sysfs(res, option, arg1, arg2)                \
1445 |   __sanitizer_syscall_post_impl_sysfs(res, (long)(option), (long)(arg1),       \
1446 |                                       (long)(arg2))
1447 | #define __sanitizer_syscall_pre_syslog(type, buf, len)                         \
1448 |   __sanitizer_syscall_pre_impl_syslog((long)(type), (long)(buf), (long)(len))
1449 | #define __sanitizer_syscall_post_syslog(res, type, buf, len)                   \
1450 |   __sanitizer_syscall_post_impl_syslog(res, (long)(type), (long)(buf),         \
```
- **Line 1441 / 第 1441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1442 / 第 1442 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1443 / 第 1443 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1444 / 第 1444 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1445 / 第 1445 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1446 / 第 1446 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1447 / 第 1447 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1448 / 第 1448 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1449 / 第 1449 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1450 / 第 1450 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1451-1460 / 第 1451-1460 行
```cpp
1451 |                                        (long)(len))
1452 | #define __sanitizer_syscall_pre_uselib(library)                                \
1453 |   __sanitizer_syscall_pre_impl_uselib((long)(library))
1454 | #define __sanitizer_syscall_post_uselib(res, library)                          \
1455 |   __sanitizer_syscall_post_impl_uselib(res, (long)(library))
1456 | #define __sanitizer_syscall_pre_ni_syscall()                                   \
1457 |   __sanitizer_syscall_pre_impl_ni_syscall()
1458 | #define __sanitizer_syscall_post_ni_syscall(res)                               \
1459 |   __sanitizer_syscall_post_impl_ni_syscall(res)
1460 | #define __sanitizer_syscall_pre_ptrace(request, pid, addr, data)               \
```
- **Line 1451 / 第 1451 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1452 / 第 1452 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1453 / 第 1453 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1454 / 第 1454 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1455 / 第 1455 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1456 / 第 1456 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1457 / 第 1457 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1458 / 第 1458 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1459 / 第 1459 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1460 / 第 1460 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1461-1470 / 第 1461-1470 行
```cpp
1461 |   __sanitizer_syscall_pre_impl_ptrace((long)(request), (long)(pid),            \
1462 |                                       (long)(addr), (long)(data))
1463 | #define __sanitizer_syscall_post_ptrace(res, request, pid, addr, data)         \
1464 |   __sanitizer_syscall_post_impl_ptrace(res, (long)(request), (long)(pid),      \
1465 |                                        (long)(addr), (long)(data))
1466 | #define __sanitizer_syscall_pre_add_key(_type, _description, _payload, plen,   \
1467 |                                         destringid)                            \
1468 |   __sanitizer_syscall_pre_impl_add_key((long)(_type), (long)(_description),    \
1469 |                                        (long)(_payload), (long)(plen),         \
1470 |                                        (long)(destringid))
```
- **Line 1461 / 第 1461 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1462 / 第 1462 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1463 / 第 1463 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1464 / 第 1464 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1465 / 第 1465 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1466 / 第 1466 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1467 / 第 1467 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1468 / 第 1468 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1469 / 第 1469 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1470 / 第 1470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1471-1480 / 第 1471-1480 行
```cpp
1471 | #define __sanitizer_syscall_post_add_key(res, _type, _description, _payload,   \
1472 |                                          plen, destringid)                     \
1473 |   __sanitizer_syscall_post_impl_add_key(                                       \
1474 |       res, (long)(_type), (long)(_description), (long)(_payload),              \
1475 |       (long)(plen), (long)(destringid))
1476 | #define __sanitizer_syscall_pre_request_key(_type, _description,               \
1477 |                                             _callout_info, destringid)         \
1478 |   __sanitizer_syscall_pre_impl_request_key(                                    \
1479 |       (long)(_type), (long)(_description), (long)(_callout_info),              \
1480 |       (long)(destringid))
```
- **Line 1471 / 第 1471 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1472 / 第 1472 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1473 / 第 1473 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1474 / 第 1474 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1475 / 第 1475 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1476 / 第 1476 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1477 / 第 1477 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1478 / 第 1478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1479 / 第 1479 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1480 / 第 1480 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1481-1490 / 第 1481-1490 行
```cpp
1481 | #define __sanitizer_syscall_post_request_key(res, _type, _description,         \
1482 |                                              _callout_info, destringid)        \
1483 |   __sanitizer_syscall_post_impl_request_key(                                   \
1484 |       res, (long)(_type), (long)(_description), (long)(_callout_info),         \
1485 |       (long)(destringid))
1486 | #define __sanitizer_syscall_pre_keyctl(cmd, arg2, arg3, arg4, arg5)            \
1487 |   __sanitizer_syscall_pre_impl_keyctl((long)(cmd), (long)(arg2), (long)(arg3), \
1488 |                                       (long)(arg4), (long)(arg5))
1489 | #define __sanitizer_syscall_post_keyctl(res, cmd, arg2, arg3, arg4, arg5)      \
1490 |   __sanitizer_syscall_post_impl_keyctl(res, (long)(cmd), (long)(arg2),         \
```
- **Line 1481 / 第 1481 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1482 / 第 1482 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1483 / 第 1483 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1484 / 第 1484 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1485 / 第 1485 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1486 / 第 1486 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1487 / 第 1487 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1488 / 第 1488 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1489 / 第 1489 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1490 / 第 1490 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1491-1500 / 第 1491-1500 行
```cpp
1491 |                                        (long)(arg3), (long)(arg4),             \
1492 |                                        (long)(arg5))
1493 | #define __sanitizer_syscall_pre_ioprio_set(which, who, ioprio)                 \
1494 |   __sanitizer_syscall_pre_impl_ioprio_set((long)(which), (long)(who),          \
1495 |                                           (long)(ioprio))
1496 | #define __sanitizer_syscall_post_ioprio_set(res, which, who, ioprio)           \
1497 |   __sanitizer_syscall_post_impl_ioprio_set(res, (long)(which), (long)(who),    \
1498 |                                            (long)(ioprio))
1499 | #define __sanitizer_syscall_pre_ioprio_get(which, who)                         \
1500 |   __sanitizer_syscall_pre_impl_ioprio_get((long)(which), (long)(who))
```
- **Line 1491 / 第 1491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1492 / 第 1492 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1493 / 第 1493 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1494 / 第 1494 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1495 / 第 1495 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1496 / 第 1496 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1497 / 第 1497 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1498 / 第 1498 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1499 / 第 1499 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1500 / 第 1500 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1501-1510 / 第 1501-1510 行
```cpp
1501 | #define __sanitizer_syscall_post_ioprio_get(res, which, who)                   \
1502 |   __sanitizer_syscall_post_impl_ioprio_get(res, (long)(which), (long)(who))
1503 | #define __sanitizer_syscall_pre_set_mempolicy(mode, nmask, maxnode)            \
1504 |   __sanitizer_syscall_pre_impl_set_mempolicy((long)(mode), (long)(nmask),      \
1505 |                                              (long)(maxnode))
1506 | #define __sanitizer_syscall_post_set_mempolicy(res, mode, nmask, maxnode)      \
1507 |   __sanitizer_syscall_post_impl_set_mempolicy(res, (long)(mode),               \
1508 |                                               (long)(nmask), (long)(maxnode))
1509 | #define __sanitizer_syscall_pre_migrate_pages(pid, maxnode, from, to)          \
1510 |   __sanitizer_syscall_pre_impl_migrate_pages((long)(pid), (long)(maxnode),     \
```
- **Line 1501 / 第 1501 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1502 / 第 1502 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1503 / 第 1503 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1504 / 第 1504 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1505 / 第 1505 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1506 / 第 1506 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1507 / 第 1507 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1508 / 第 1508 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1509 / 第 1509 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1510 / 第 1510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1511-1520 / 第 1511-1520 行
```cpp
1511 |                                              (long)(from), (long)(to))
1512 | #define __sanitizer_syscall_post_migrate_pages(res, pid, maxnode, from, to)    \
1513 |   __sanitizer_syscall_post_impl_migrate_pages(                                 \
1514 |       res, (long)(pid), (long)(maxnode), (long)(from), (long)(to))
1515 | #define __sanitizer_syscall_pre_move_pages(pid, nr_pages, pages, nodes,        \
1516 |                                            status, flags)                      \
1517 |   __sanitizer_syscall_pre_impl_move_pages((long)(pid), (long)(nr_pages),       \
1518 |                                           (long)(pages), (long)(nodes),        \
1519 |                                           (long)(status), (long)(flags))
1520 | #define __sanitizer_syscall_post_move_pages(res, pid, nr_pages, pages, nodes,  \
```
- **Line 1511 / 第 1511 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1512 / 第 1512 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1513 / 第 1513 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1514 / 第 1514 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1515 / 第 1515 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1516 / 第 1516 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1517 / 第 1517 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1518 / 第 1518 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1519 / 第 1519 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1520 / 第 1520 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1521-1530 / 第 1521-1530 行
```cpp
1521 |                                             status, flags)                     \
1522 |   __sanitizer_syscall_post_impl_move_pages(res, (long)(pid), (long)(nr_pages), \
1523 |                                            (long)(pages), (long)(nodes),       \
1524 |                                            (long)(status), (long)(flags))
1525 | #define __sanitizer_syscall_pre_mbind(start, len, mode, nmask, maxnode, flags) \
1526 |   __sanitizer_syscall_pre_impl_mbind((long)(start), (long)(len), (long)(mode), \
1527 |                                      (long)(nmask), (long)(maxnode),           \
1528 |                                      (long)(flags))
1529 | #define __sanitizer_syscall_post_mbind(res, start, len, mode, nmask, maxnode,  \
1530 |                                        flags)                                  \
```
- **Line 1521 / 第 1521 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1522 / 第 1522 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1523 / 第 1523 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1524 / 第 1524 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1525 / 第 1525 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1526 / 第 1526 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1527 / 第 1527 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1528 / 第 1528 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1529 / 第 1529 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1530 / 第 1530 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1531-1540 / 第 1531-1540 行
```cpp
1531 |   __sanitizer_syscall_post_impl_mbind(res, (long)(start), (long)(len),         \
1532 |                                       (long)(mode), (long)(nmask),             \
1533 |                                       (long)(maxnode), (long)(flags))
1534 | #define __sanitizer_syscall_pre_get_mempolicy(policy, nmask, maxnode, addr,    \
1535 |                                               flags)                           \
1536 |   __sanitizer_syscall_pre_impl_get_mempolicy((long)(policy), (long)(nmask),    \
1537 |                                              (long)(maxnode), (long)(addr),    \
1538 |                                              (long)(flags))
1539 | #define __sanitizer_syscall_post_get_mempolicy(res, policy, nmask, maxnode,    \
1540 |                                                addr, flags)                    \
```
- **Line 1531 / 第 1531 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1532 / 第 1532 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1533 / 第 1533 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1534 / 第 1534 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1535 / 第 1535 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1536 / 第 1536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1537 / 第 1537 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1538 / 第 1538 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1539 / 第 1539 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1540 / 第 1540 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1541-1550 / 第 1541-1550 行
```cpp
1541 |   __sanitizer_syscall_post_impl_get_mempolicy(res, (long)(policy),             \
1542 |                                               (long)(nmask), (long)(maxnode),  \
1543 |                                               (long)(addr), (long)(flags))
1544 | #define __sanitizer_syscall_pre_inotify_init()                                 \
1545 |   __sanitizer_syscall_pre_impl_inotify_init()
1546 | #define __sanitizer_syscall_post_inotify_init(res)                             \
1547 |   __sanitizer_syscall_post_impl_inotify_init(res)
1548 | #define __sanitizer_syscall_pre_inotify_init1(flags)                           \
1549 |   __sanitizer_syscall_pre_impl_inotify_init1((long)(flags))
1550 | #define __sanitizer_syscall_post_inotify_init1(res, flags)                     \
```
- **Line 1541 / 第 1541 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1542 / 第 1542 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1543 / 第 1543 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1544 / 第 1544 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1545 / 第 1545 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1546 / 第 1546 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1547 / 第 1547 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1548 / 第 1548 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1549 / 第 1549 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1550 / 第 1550 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1551-1560 / 第 1551-1560 行
```cpp
1551 |   __sanitizer_syscall_post_impl_inotify_init1(res, (long)(flags))
1552 | #define __sanitizer_syscall_pre_inotify_add_watch(fd, path, mask)              \
1553 |   __sanitizer_syscall_pre_impl_inotify_add_watch((long)(fd), (long)(path),     \
1554 |                                                  (long)(mask))
1555 | #define __sanitizer_syscall_post_inotify_add_watch(res, fd, path, mask)        \
1556 |   __sanitizer_syscall_post_impl_inotify_add_watch(res, (long)(fd),             \
1557 |                                                   (long)(path), (long)(mask))
1558 | #define __sanitizer_syscall_pre_inotify_rm_watch(fd, wd)                       \
1559 |   __sanitizer_syscall_pre_impl_inotify_rm_watch((long)(fd), (long)(wd))
1560 | #define __sanitizer_syscall_post_inotify_rm_watch(res, fd, wd)                 \
```
- **Line 1551 / 第 1551 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1552 / 第 1552 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1553 / 第 1553 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1554 / 第 1554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1555 / 第 1555 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1556 / 第 1556 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1557 / 第 1557 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1558 / 第 1558 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1559 / 第 1559 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1560 / 第 1560 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1561-1570 / 第 1561-1570 行
```cpp
1561 |   __sanitizer_syscall_post_impl_inotify_rm_watch(res, (long)(fd), (long)(wd))
1562 | #define __sanitizer_syscall_pre_spu_run(fd, unpc, ustatus)                     \
1563 |   __sanitizer_syscall_pre_impl_spu_run((long)(fd), (long)(unpc),               \
1564 |                                        (long)(ustatus))
1565 | #define __sanitizer_syscall_post_spu_run(res, fd, unpc, ustatus)               \
1566 |   __sanitizer_syscall_post_impl_spu_run(res, (long)(fd), (long)(unpc),         \
1567 |                                         (long)(ustatus))
1568 | #define __sanitizer_syscall_pre_spu_create(name, flags, mode, fd)              \
1569 |   __sanitizer_syscall_pre_impl_spu_create((long)(name), (long)(flags),         \
1570 |                                           (long)(mode), (long)(fd))
```
- **Line 1561 / 第 1561 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1562 / 第 1562 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1563 / 第 1563 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1564 / 第 1564 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1565 / 第 1565 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1566 / 第 1566 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1567 / 第 1567 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1568 / 第 1568 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1569 / 第 1569 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1570 / 第 1570 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1571-1580 / 第 1571-1580 行
```cpp
1571 | #define __sanitizer_syscall_post_spu_create(res, name, flags, mode, fd)        \
1572 |   __sanitizer_syscall_post_impl_spu_create(res, (long)(name), (long)(flags),   \
1573 |                                            (long)(mode), (long)(fd))
1574 | #define __sanitizer_syscall_pre_mknodat(dfd, filename, mode, dev)              \
1575 |   __sanitizer_syscall_pre_impl_mknodat((long)(dfd), (long)(filename),          \
1576 |                                        (long)(mode), (long)(dev))
1577 | #define __sanitizer_syscall_post_mknodat(res, dfd, filename, mode, dev)        \
1578 |   __sanitizer_syscall_post_impl_mknodat(res, (long)(dfd), (long)(filename),    \
1579 |                                         (long)(mode), (long)(dev))
1580 | #define __sanitizer_syscall_pre_mkdirat(dfd, pathname, mode)                   \
```
- **Line 1571 / 第 1571 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1572 / 第 1572 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1573 / 第 1573 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1574 / 第 1574 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1575 / 第 1575 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1576 / 第 1576 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1577 / 第 1577 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1578 / 第 1578 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1579 / 第 1579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1580 / 第 1580 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1581-1590 / 第 1581-1590 行
```cpp
1581 |   __sanitizer_syscall_pre_impl_mkdirat((long)(dfd), (long)(pathname),          \
1582 |                                        (long)(mode))
1583 | #define __sanitizer_syscall_post_mkdirat(res, dfd, pathname, mode)             \
1584 |   __sanitizer_syscall_post_impl_mkdirat(res, (long)(dfd), (long)(pathname),    \
1585 |                                         (long)(mode))
1586 | #define __sanitizer_syscall_pre_unlinkat(dfd, pathname, flag)                  \
1587 |   __sanitizer_syscall_pre_impl_unlinkat((long)(dfd), (long)(pathname),         \
1588 |                                         (long)(flag))
1589 | #define __sanitizer_syscall_post_unlinkat(res, dfd, pathname, flag)            \
1590 |   __sanitizer_syscall_post_impl_unlinkat(res, (long)(dfd), (long)(pathname),   \
```
- **Line 1581 / 第 1581 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1582 / 第 1582 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1583 / 第 1583 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1584 / 第 1584 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1585 / 第 1585 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1586 / 第 1586 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1587 / 第 1587 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1588 / 第 1588 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1589 / 第 1589 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1590 / 第 1590 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1591-1600 / 第 1591-1600 行
```cpp
1591 |                                          (long)(flag))
1592 | #define __sanitizer_syscall_pre_symlinkat(oldname, newdfd, newname)            \
1593 |   __sanitizer_syscall_pre_impl_symlinkat((long)(oldname), (long)(newdfd),      \
1594 |                                          (long)(newname))
1595 | #define __sanitizer_syscall_post_symlinkat(res, oldname, newdfd, newname)      \
1596 |   __sanitizer_syscall_post_impl_symlinkat(res, (long)(oldname),                \
1597 |                                           (long)(newdfd), (long)(newname))
1598 | #define __sanitizer_syscall_pre_linkat(olddfd, oldname, newdfd, newname,       \
1599 |                                        flags)                                  \
1600 |   __sanitizer_syscall_pre_impl_linkat((long)(olddfd), (long)(oldname),         \
```
- **Line 1591 / 第 1591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1592 / 第 1592 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1593 / 第 1593 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1594 / 第 1594 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1595 / 第 1595 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1596 / 第 1596 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1597 / 第 1597 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1598 / 第 1598 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1599 / 第 1599 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1600 / 第 1600 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1601-1610 / 第 1601-1610 行
```cpp
1601 |                                       (long)(newdfd), (long)(newname),         \
1602 |                                       (long)(flags))
1603 | #define __sanitizer_syscall_post_linkat(res, olddfd, oldname, newdfd, newname, \
1604 |                                         flags)                                 \
1605 |   __sanitizer_syscall_post_impl_linkat(res, (long)(olddfd), (long)(oldname),   \
1606 |                                        (long)(newdfd), (long)(newname),        \
1607 |                                        (long)(flags))
1608 | #define __sanitizer_syscall_pre_renameat(olddfd, oldname, newdfd, newname)     \
1609 |   __sanitizer_syscall_pre_impl_renameat((long)(olddfd), (long)(oldname),       \
1610 |                                         (long)(newdfd), (long)(newname))
```
- **Line 1601 / 第 1601 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1602 / 第 1602 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1603 / 第 1603 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1604 / 第 1604 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1605 / 第 1605 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1606 / 第 1606 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1607 / 第 1607 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1608 / 第 1608 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1609 / 第 1609 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1610 / 第 1610 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1611-1620 / 第 1611-1620 行
```cpp
1611 | #define __sanitizer_syscall_post_renameat(res, olddfd, oldname, newdfd,        \
1612 |                                           newname)                             \
1613 |   __sanitizer_syscall_post_impl_renameat(res, (long)(olddfd), (long)(oldname), \
1614 |                                          (long)(newdfd), (long)(newname))
1615 | #define __sanitizer_syscall_pre_futimesat(dfd, filename, utimes)               \
1616 |   __sanitizer_syscall_pre_impl_futimesat((long)(dfd), (long)(filename),        \
1617 |                                          (long)(utimes))
1618 | #define __sanitizer_syscall_post_futimesat(res, dfd, filename, utimes)         \
1619 |   __sanitizer_syscall_post_impl_futimesat(res, (long)(dfd), (long)(filename),  \
1620 |                                           (long)(utimes))
```
- **Line 1611 / 第 1611 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1612 / 第 1612 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1613 / 第 1613 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1614 / 第 1614 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1615 / 第 1615 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1616 / 第 1616 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1617 / 第 1617 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1618 / 第 1618 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1619 / 第 1619 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1620 / 第 1620 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1621-1630 / 第 1621-1630 行
```cpp
1621 | #define __sanitizer_syscall_pre_faccessat(dfd, filename, mode)                 \
1622 |   __sanitizer_syscall_pre_impl_faccessat((long)(dfd), (long)(filename),        \
1623 |                                          (long)(mode))
1624 | #define __sanitizer_syscall_post_faccessat(res, dfd, filename, mode)           \
1625 |   __sanitizer_syscall_post_impl_faccessat(res, (long)(dfd), (long)(filename),  \
1626 |                                           (long)(mode))
1627 | #define __sanitizer_syscall_pre_fchmodat(dfd, filename, mode)                  \
1628 |   __sanitizer_syscall_pre_impl_fchmodat((long)(dfd), (long)(filename),         \
1629 |                                         (long)(mode))
1630 | #define __sanitizer_syscall_post_fchmodat(res, dfd, filename, mode)            \
```
- **Line 1621 / 第 1621 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1622 / 第 1622 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1623 / 第 1623 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1624 / 第 1624 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1625 / 第 1625 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1626 / 第 1626 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1627 / 第 1627 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1628 / 第 1628 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1629 / 第 1629 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1630 / 第 1630 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1631-1640 / 第 1631-1640 行
```cpp
1631 |   __sanitizer_syscall_post_impl_fchmodat(res, (long)(dfd), (long)(filename),   \
1632 |                                          (long)(mode))
1633 | #define __sanitizer_syscall_pre_fchownat(dfd, filename, user, group, flag)     \
1634 |   __sanitizer_syscall_pre_impl_fchownat((long)(dfd), (long)(filename),         \
1635 |                                         (long)(user), (long)(group),           \
1636 |                                         (long)(flag))
1637 | #define __sanitizer_syscall_post_fchownat(res, dfd, filename, user, group,     \
1638 |                                           flag)                                \
1639 |   __sanitizer_syscall_post_impl_fchownat(res, (long)(dfd), (long)(filename),   \
1640 |                                          (long)(user), (long)(group),          \
```
- **Line 1631 / 第 1631 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1632 / 第 1632 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1633 / 第 1633 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1634 / 第 1634 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1635 / 第 1635 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1636 / 第 1636 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1637 / 第 1637 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1638 / 第 1638 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1639 / 第 1639 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1640 / 第 1640 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1641-1650 / 第 1641-1650 行
```cpp
1641 |                                          (long)(flag))
1642 | #define __sanitizer_syscall_pre_openat(dfd, filename, flags, mode)             \
1643 |   __sanitizer_syscall_pre_impl_openat((long)(dfd), (long)(filename),           \
1644 |                                       (long)(flags), (long)(mode))
1645 | #define __sanitizer_syscall_post_openat(res, dfd, filename, flags, mode)       \
1646 |   __sanitizer_syscall_post_impl_openat(res, (long)(dfd), (long)(filename),     \
1647 |                                        (long)(flags), (long)(mode))
1648 | #define __sanitizer_syscall_pre_newfstatat(dfd, filename, statbuf, flag)       \
1649 |   __sanitizer_syscall_pre_impl_newfstatat((long)(dfd), (long)(filename),       \
1650 |                                           (long)(statbuf), (long)(flag))
```
- **Line 1641 / 第 1641 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1642 / 第 1642 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1643 / 第 1643 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1644 / 第 1644 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1645 / 第 1645 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1646 / 第 1646 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1647 / 第 1647 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1648 / 第 1648 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1649 / 第 1649 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1650 / 第 1650 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1651-1660 / 第 1651-1660 行
```cpp
1651 | #define __sanitizer_syscall_post_newfstatat(res, dfd, filename, statbuf, flag) \
1652 |   __sanitizer_syscall_post_impl_newfstatat(res, (long)(dfd), (long)(filename), \
1653 |                                            (long)(statbuf), (long)(flag))
1654 | #define __sanitizer_syscall_pre_fstatat64(dfd, filename, statbuf, flag)        \
1655 |   __sanitizer_syscall_pre_impl_fstatat64((long)(dfd), (long)(filename),        \
1656 |                                          (long)(statbuf), (long)(flag))
1657 | #define __sanitizer_syscall_post_fstatat64(res, dfd, filename, statbuf, flag)  \
1658 |   __sanitizer_syscall_post_impl_fstatat64(res, (long)(dfd), (long)(filename),  \
1659 |                                           (long)(statbuf), (long)(flag))
1660 | #define __sanitizer_syscall_pre_readlinkat(dfd, path, buf, bufsiz)             \
```
- **Line 1651 / 第 1651 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1652 / 第 1652 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1653 / 第 1653 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1654 / 第 1654 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1655 / 第 1655 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1656 / 第 1656 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1657 / 第 1657 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1658 / 第 1658 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1659 / 第 1659 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1660 / 第 1660 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1661-1670 / 第 1661-1670 行
```cpp
1661 |   __sanitizer_syscall_pre_impl_readlinkat((long)(dfd), (long)(path),           \
1662 |                                           (long)(buf), (long)(bufsiz))
1663 | #define __sanitizer_syscall_post_readlinkat(res, dfd, path, buf, bufsiz)       \
1664 |   __sanitizer_syscall_post_impl_readlinkat(res, (long)(dfd), (long)(path),     \
1665 |                                            (long)(buf), (long)(bufsiz))
1666 | #define __sanitizer_syscall_pre_utimensat(dfd, filename, utimes, flags)        \
1667 |   __sanitizer_syscall_pre_impl_utimensat((long)(dfd), (long)(filename),        \
1668 |                                          (long)(utimes), (long)(flags))
1669 | #define __sanitizer_syscall_post_utimensat(res, dfd, filename, utimes, flags)  \
1670 |   __sanitizer_syscall_post_impl_utimensat(res, (long)(dfd), (long)(filename),  \
```
- **Line 1661 / 第 1661 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1662 / 第 1662 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1663 / 第 1663 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1664 / 第 1664 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1665 / 第 1665 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1666 / 第 1666 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1667 / 第 1667 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1668 / 第 1668 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1669 / 第 1669 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1670 / 第 1670 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1671-1680 / 第 1671-1680 行
```cpp
1671 |                                           (long)(utimes), (long)(flags))
1672 | #define __sanitizer_syscall_pre_unshare(unshare_flags)                         \
1673 |   __sanitizer_syscall_pre_impl_unshare((long)(unshare_flags))
1674 | #define __sanitizer_syscall_post_unshare(res, unshare_flags)                   \
1675 |   __sanitizer_syscall_post_impl_unshare(res, (long)(unshare_flags))
1676 | #define __sanitizer_syscall_pre_splice(fd_in, off_in, fd_out, off_out, len,    \
1677 |                                        flags)                                  \
1678 |   __sanitizer_syscall_pre_impl_splice((long)(fd_in), (long)(off_in),           \
1679 |                                       (long)(fd_out), (long)(off_out),         \
1680 |                                       (long)(len), (long)(flags))
```
- **Line 1671 / 第 1671 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1672 / 第 1672 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1673 / 第 1673 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1674 / 第 1674 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1675 / 第 1675 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1676 / 第 1676 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1677 / 第 1677 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1678 / 第 1678 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1679 / 第 1679 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1680 / 第 1680 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1681-1690 / 第 1681-1690 行
```cpp
1681 | #define __sanitizer_syscall_post_splice(res, fd_in, off_in, fd_out, off_out,   \
1682 |                                         len, flags)                            \
1683 |   __sanitizer_syscall_post_impl_splice(res, (long)(fd_in), (long)(off_in),     \
1684 |                                        (long)(fd_out), (long)(off_out),        \
1685 |                                        (long)(len), (long)(flags))
1686 | #define __sanitizer_syscall_pre_vmsplice(fd, iov, nr_segs, flags)              \
1687 |   __sanitizer_syscall_pre_impl_vmsplice((long)(fd), (long)(iov),               \
1688 |                                         (long)(nr_segs), (long)(flags))
1689 | #define __sanitizer_syscall_post_vmsplice(res, fd, iov, nr_segs, flags)        \
1690 |   __sanitizer_syscall_post_impl_vmsplice(res, (long)(fd), (long)(iov),         \
```
- **Line 1681 / 第 1681 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1682 / 第 1682 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1683 / 第 1683 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1684 / 第 1684 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1685 / 第 1685 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1686 / 第 1686 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1687 / 第 1687 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1688 / 第 1688 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1689 / 第 1689 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1690 / 第 1690 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1691-1700 / 第 1691-1700 行
```cpp
1691 |                                          (long)(nr_segs), (long)(flags))
1692 | #define __sanitizer_syscall_pre_tee(fdin, fdout, len, flags)                   \
1693 |   __sanitizer_syscall_pre_impl_tee((long)(fdin), (long)(fdout), (long)(len),   \
1694 |                                    (long)(flags))
1695 | #define __sanitizer_syscall_post_tee(res, fdin, fdout, len, flags)             \
1696 |   __sanitizer_syscall_post_impl_tee(res, (long)(fdin), (long)(fdout),          \
1697 |                                     (long)(len), (long)(flags))
1698 | #define __sanitizer_syscall_pre_get_robust_list(pid, head_ptr, len_ptr)        \
1699 |   __sanitizer_syscall_pre_impl_get_robust_list((long)(pid), (long)(head_ptr),  \
1700 |                                                (long)(len_ptr))
```
- **Line 1691 / 第 1691 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1692 / 第 1692 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1693 / 第 1693 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1694 / 第 1694 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1695 / 第 1695 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1696 / 第 1696 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1697 / 第 1697 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1698 / 第 1698 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1699 / 第 1699 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1700 / 第 1700 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1701-1710 / 第 1701-1710 行
```cpp
1701 | #define __sanitizer_syscall_post_get_robust_list(res, pid, head_ptr, len_ptr)  \
1702 |   __sanitizer_syscall_post_impl_get_robust_list(                               \
1703 |       res, (long)(pid), (long)(head_ptr), (long)(len_ptr))
1704 | #define __sanitizer_syscall_pre_set_robust_list(head, len)                     \
1705 |   __sanitizer_syscall_pre_impl_set_robust_list((long)(head), (long)(len))
1706 | #define __sanitizer_syscall_post_set_robust_list(res, head, len)               \
1707 |   __sanitizer_syscall_post_impl_set_robust_list(res, (long)(head), (long)(len))
1708 | #define __sanitizer_syscall_pre_getcpu(cpu, node, cache)                       \
1709 |   __sanitizer_syscall_pre_impl_getcpu((long)(cpu), (long)(node), (long)(cache))
1710 | #define __sanitizer_syscall_post_getcpu(res, cpu, node, cache)                 \
```
- **Line 1701 / 第 1701 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1702 / 第 1702 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1703 / 第 1703 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1704 / 第 1704 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1705 / 第 1705 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1706 / 第 1706 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1707 / 第 1707 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1708 / 第 1708 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1709 / 第 1709 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1710 / 第 1710 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1711-1720 / 第 1711-1720 行
```cpp
1711 |   __sanitizer_syscall_post_impl_getcpu(res, (long)(cpu), (long)(node),         \
1712 |                                        (long)(cache))
1713 | #define __sanitizer_syscall_pre_signalfd(ufd, user_mask, sizemask)             \
1714 |   __sanitizer_syscall_pre_impl_signalfd((long)(ufd), (long)(user_mask),        \
1715 |                                         (long)(sizemask))
1716 | #define __sanitizer_syscall_post_signalfd(res, ufd, user_mask, sizemask)       \
1717 |   __sanitizer_syscall_post_impl_signalfd(res, (long)(ufd), (long)(user_mask),  \
1718 |                                          (long)(sizemask))
1719 | #define __sanitizer_syscall_pre_signalfd4(ufd, user_mask, sizemask, flags)     \
1720 |   __sanitizer_syscall_pre_impl_signalfd4((long)(ufd), (long)(user_mask),       \
```
- **Line 1711 / 第 1711 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1712 / 第 1712 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1713 / 第 1713 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1714 / 第 1714 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1715 / 第 1715 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1716 / 第 1716 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1717 / 第 1717 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1718 / 第 1718 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1719 / 第 1719 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1720 / 第 1720 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1721-1730 / 第 1721-1730 行
```cpp
1721 |                                          (long)(sizemask), (long)(flags))
1722 | #define __sanitizer_syscall_post_signalfd4(res, ufd, user_mask, sizemask,      \
1723 |                                            flags)                              \
1724 |   __sanitizer_syscall_post_impl_signalfd4(res, (long)(ufd), (long)(user_mask), \
1725 |                                           (long)(sizemask), (long)(flags))
1726 | #define __sanitizer_syscall_pre_timerfd_create(clockid, flags)                 \
1727 |   __sanitizer_syscall_pre_impl_timerfd_create((long)(clockid), (long)(flags))
1728 | #define __sanitizer_syscall_post_timerfd_create(res, clockid, flags)           \
1729 |   __sanitizer_syscall_post_impl_timerfd_create(res, (long)(clockid),           \
1730 |                                                (long)(flags))
```
- **Line 1721 / 第 1721 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1722 / 第 1722 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1723 / 第 1723 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1724 / 第 1724 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1725 / 第 1725 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1726 / 第 1726 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1727 / 第 1727 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1728 / 第 1728 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1729 / 第 1729 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1730 / 第 1730 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1731-1740 / 第 1731-1740 行
```cpp
1731 | #define __sanitizer_syscall_pre_timerfd_settime(ufd, flags, utmr, otmr)        \
1732 |   __sanitizer_syscall_pre_impl_timerfd_settime((long)(ufd), (long)(flags),     \
1733 |                                                (long)(utmr), (long)(otmr))
1734 | #define __sanitizer_syscall_post_timerfd_settime(res, ufd, flags, utmr, otmr)  \
1735 |   __sanitizer_syscall_post_impl_timerfd_settime(                               \
1736 |       res, (long)(ufd), (long)(flags), (long)(utmr), (long)(otmr))
1737 | #define __sanitizer_syscall_pre_timerfd_gettime(ufd, otmr)                     \
1738 |   __sanitizer_syscall_pre_impl_timerfd_gettime((long)(ufd), (long)(otmr))
1739 | #define __sanitizer_syscall_post_timerfd_gettime(res, ufd, otmr)               \
1740 |   __sanitizer_syscall_post_impl_timerfd_gettime(res, (long)(ufd), (long)(otmr))
```
- **Line 1731 / 第 1731 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1732 / 第 1732 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1733 / 第 1733 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1734 / 第 1734 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1735 / 第 1735 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1736 / 第 1736 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1737 / 第 1737 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1738 / 第 1738 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1739 / 第 1739 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1740 / 第 1740 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1741-1750 / 第 1741-1750 行
```cpp
1741 | #define __sanitizer_syscall_pre_eventfd(count)                                 \
1742 |   __sanitizer_syscall_pre_impl_eventfd((long)(count))
1743 | #define __sanitizer_syscall_post_eventfd(res, count)                           \
1744 |   __sanitizer_syscall_post_impl_eventfd(res, (long)(count))
1745 | #define __sanitizer_syscall_pre_eventfd2(count, flags)                         \
1746 |   __sanitizer_syscall_pre_impl_eventfd2((long)(count), (long)(flags))
1747 | #define __sanitizer_syscall_post_eventfd2(res, count, flags)                   \
1748 |   __sanitizer_syscall_post_impl_eventfd2(res, (long)(count), (long)(flags))
1749 | #define __sanitizer_syscall_pre_old_readdir(arg0, arg1, arg2)                  \
1750 |   __sanitizer_syscall_pre_impl_old_readdir((long)(arg0), (long)(arg1),         \
```
- **Line 1741 / 第 1741 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1742 / 第 1742 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1743 / 第 1743 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1744 / 第 1744 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1745 / 第 1745 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1746 / 第 1746 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1747 / 第 1747 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1748 / 第 1748 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1749 / 第 1749 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1750 / 第 1750 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1751-1760 / 第 1751-1760 行
```cpp
1751 |                                            (long)(arg2))
1752 | #define __sanitizer_syscall_post_old_readdir(res, arg0, arg1, arg2)            \
1753 |   __sanitizer_syscall_post_impl_old_readdir(res, (long)(arg0), (long)(arg1),   \
1754 |                                             (long)(arg2))
1755 | #define __sanitizer_syscall_pre_pselect6(arg0, arg1, arg2, arg3, arg4, arg5)   \
1756 |   __sanitizer_syscall_pre_impl_pselect6((long)(arg0), (long)(arg1),            \
1757 |                                         (long)(arg2), (long)(arg3),            \
1758 |                                         (long)(arg4), (long)(arg5))
1759 | #define __sanitizer_syscall_post_pselect6(res, arg0, arg1, arg2, arg3, arg4,   \
1760 |                                           arg5)                                \
```
- **Line 1751 / 第 1751 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1752 / 第 1752 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1753 / 第 1753 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1754 / 第 1754 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1755 / 第 1755 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1756 / 第 1756 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1757 / 第 1757 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1758 / 第 1758 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1759 / 第 1759 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1760 / 第 1760 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1761-1770 / 第 1761-1770 行
```cpp
1761 |   __sanitizer_syscall_post_impl_pselect6(res, (long)(arg0), (long)(arg1),      \
1762 |                                          (long)(arg2), (long)(arg3),           \
1763 |                                          (long)(arg4), (long)(arg5))
1764 | #define __sanitizer_syscall_pre_ppoll(arg0, arg1, arg2, arg3, arg4)            \
1765 |   __sanitizer_syscall_pre_impl_ppoll((long)(arg0), (long)(arg1), (long)(arg2), \
1766 |                                      (long)(arg3), (long)(arg4))
1767 | #define __sanitizer_syscall_post_ppoll(res, arg0, arg1, arg2, arg3, arg4)      \
1768 |   __sanitizer_syscall_post_impl_ppoll(res, (long)(arg0), (long)(arg1),         \
1769 |                                       (long)(arg2), (long)(arg3),              \
1770 |                                       (long)(arg4))
```
- **Line 1761 / 第 1761 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1762 / 第 1762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1763 / 第 1763 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1764 / 第 1764 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1765 / 第 1765 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1766 / 第 1766 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1767 / 第 1767 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1768 / 第 1768 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1769 / 第 1769 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1770 / 第 1770 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1771-1780 / 第 1771-1780 行
```cpp
1771 | #define __sanitizer_syscall_pre_syncfs(fd)                                     \
1772 |   __sanitizer_syscall_pre_impl_syncfs((long)(fd))
1773 | #define __sanitizer_syscall_post_syncfs(res, fd)                               \
1774 |   __sanitizer_syscall_post_impl_syncfs(res, (long)(fd))
1775 | #define __sanitizer_syscall_pre_perf_event_open(attr_uptr, pid, cpu, group_fd, \
1776 |                                                 flags)                         \
1777 |   __sanitizer_syscall_pre_impl_perf_event_open((long)(attr_uptr), (long)(pid), \
1778 |                                                (long)(cpu), (long)(group_fd),  \
1779 |                                                (long)(flags))
1780 | #define __sanitizer_syscall_post_perf_event_open(res, attr_uptr, pid, cpu,     \
```
- **Line 1771 / 第 1771 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1772 / 第 1772 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1773 / 第 1773 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1774 / 第 1774 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1775 / 第 1775 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1776 / 第 1776 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1777 / 第 1777 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1778 / 第 1778 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1779 / 第 1779 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1780 / 第 1780 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1781-1790 / 第 1781-1790 行
```cpp
1781 |                                                  group_fd, flags)              \
1782 |   __sanitizer_syscall_post_impl_perf_event_open(                               \
1783 |       res, (long)(attr_uptr), (long)(pid), (long)(cpu), (long)(group_fd),      \
1784 |       (long)(flags))
1785 | #define __sanitizer_syscall_pre_mmap_pgoff(addr, len, prot, flags, fd, pgoff)  \
1786 |   __sanitizer_syscall_pre_impl_mmap_pgoff((long)(addr), (long)(len),           \
1787 |                                           (long)(prot), (long)(flags),         \
1788 |                                           (long)(fd), (long)(pgoff))
1789 | #define __sanitizer_syscall_post_mmap_pgoff(res, addr, len, prot, flags, fd,   \
1790 |                                             pgoff)                             \
```
- **Line 1781 / 第 1781 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1782 / 第 1782 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1783 / 第 1783 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1784 / 第 1784 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1785 / 第 1785 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1786 / 第 1786 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1787 / 第 1787 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1788 / 第 1788 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1789 / 第 1789 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1790 / 第 1790 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1791-1800 / 第 1791-1800 行
```cpp
1791 |   __sanitizer_syscall_post_impl_mmap_pgoff(res, (long)(addr), (long)(len),     \
1792 |                                            (long)(prot), (long)(flags),        \
1793 |                                            (long)(fd), (long)(pgoff))
1794 | #define __sanitizer_syscall_pre_old_mmap(arg)                                  \
1795 |   __sanitizer_syscall_pre_impl_old_mmap((long)(arg))
1796 | #define __sanitizer_syscall_post_old_mmap(res, arg)                            \
1797 |   __sanitizer_syscall_post_impl_old_mmap(res, (long)(arg))
1798 | #define __sanitizer_syscall_pre_name_to_handle_at(dfd, name, handle, mnt_id,   \
1799 |                                                   flag)                        \
1800 |   __sanitizer_syscall_pre_impl_name_to_handle_at(                              \
```
- **Line 1791 / 第 1791 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1792 / 第 1792 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1793 / 第 1793 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1794 / 第 1794 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1795 / 第 1795 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1796 / 第 1796 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1797 / 第 1797 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1798 / 第 1798 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1799 / 第 1799 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1800 / 第 1800 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1801-1810 / 第 1801-1810 行
```cpp
1801 |       (long)(dfd), (long)(name), (long)(handle), (long)(mnt_id), (long)(flag))
1802 | #define __sanitizer_syscall_post_name_to_handle_at(res, dfd, name, handle,     \
1803 |                                                    mnt_id, flag)               \
1804 |   __sanitizer_syscall_post_impl_name_to_handle_at(                             \
1805 |       res, (long)(dfd), (long)(name), (long)(handle), (long)(mnt_id),          \
1806 |       (long)(flag))
1807 | #define __sanitizer_syscall_pre_open_by_handle_at(mountdirfd, handle, flags)   \
1808 |   __sanitizer_syscall_pre_impl_open_by_handle_at(                              \
1809 |       (long)(mountdirfd), (long)(handle), (long)(flags))
1810 | #define __sanitizer_syscall_post_open_by_handle_at(res, mountdirfd, handle,    \
```
- **Line 1801 / 第 1801 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1802 / 第 1802 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1803 / 第 1803 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1804 / 第 1804 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1805 / 第 1805 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1806 / 第 1806 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1807 / 第 1807 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1808 / 第 1808 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1809 / 第 1809 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1810 / 第 1810 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1811-1820 / 第 1811-1820 行
```cpp
1811 |                                                    flags)                      \
1812 |   __sanitizer_syscall_post_impl_open_by_handle_at(                             \
1813 |       res, (long)(mountdirfd), (long)(handle), (long)(flags))
1814 | #define __sanitizer_syscall_pre_setns(fd, nstype)                              \
1815 |   __sanitizer_syscall_pre_impl_setns((long)(fd), (long)(nstype))
1816 | #define __sanitizer_syscall_post_setns(res, fd, nstype)                        \
1817 |   __sanitizer_syscall_post_impl_setns(res, (long)(fd), (long)(nstype))
1818 | #define __sanitizer_syscall_pre_process_vm_readv(pid, lvec, liovcnt, rvec,     \
1819 |                                                  riovcnt, flags)               \
1820 |   __sanitizer_syscall_pre_impl_process_vm_readv(                               \
```
- **Line 1811 / 第 1811 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1812 / 第 1812 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1813 / 第 1813 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1814 / 第 1814 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1815 / 第 1815 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1816 / 第 1816 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1817 / 第 1817 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1818 / 第 1818 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1819 / 第 1819 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1820 / 第 1820 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1821-1830 / 第 1821-1830 行
```cpp
1821 |       (long)(pid), (long)(lvec), (long)(liovcnt), (long)(rvec),                \
1822 |       (long)(riovcnt), (long)(flags))
1823 | #define __sanitizer_syscall_post_process_vm_readv(res, pid, lvec, liovcnt,     \
1824 |                                                   rvec, riovcnt, flags)        \
1825 |   __sanitizer_syscall_post_impl_process_vm_readv(                              \
1826 |       res, (long)(pid), (long)(lvec), (long)(liovcnt), (long)(rvec),           \
1827 |       (long)(riovcnt), (long)(flags))
1828 | #define __sanitizer_syscall_pre_process_vm_writev(pid, lvec, liovcnt, rvec,    \
1829 |                                                   riovcnt, flags)              \
1830 |   __sanitizer_syscall_pre_impl_process_vm_writev(                              \
```
- **Line 1821 / 第 1821 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1822 / 第 1822 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1823 / 第 1823 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1824 / 第 1824 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1825 / 第 1825 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1826 / 第 1826 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1827 / 第 1827 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1828 / 第 1828 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1829 / 第 1829 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1830 / 第 1830 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1831-1840 / 第 1831-1840 行
```cpp
1831 |       (long)(pid), (long)(lvec), (long)(liovcnt), (long)(rvec),                \
1832 |       (long)(riovcnt), (long)(flags))
1833 | #define __sanitizer_syscall_post_process_vm_writev(res, pid, lvec, liovcnt,    \
1834 |                                                    rvec, riovcnt, flags)       \
1835 |   __sanitizer_syscall_post_impl_process_vm_writev(                             \
1836 |       res, (long)(pid), (long)(lvec), (long)(liovcnt), (long)(rvec),           \
1837 |       (long)(riovcnt), (long)(flags))
1838 | #define __sanitizer_syscall_pre_fork() __sanitizer_syscall_pre_impl_fork()
1839 | #define __sanitizer_syscall_post_fork(res)                                     \
1840 |   __sanitizer_syscall_post_impl_fork(res)
```
- **Line 1831 / 第 1831 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1832 / 第 1832 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1833 / 第 1833 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1834 / 第 1834 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1835 / 第 1835 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1836 / 第 1836 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1837 / 第 1837 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1838 / 第 1838 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1839 / 第 1839 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1840 / 第 1840 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1841-1850 / 第 1841-1850 行
```cpp
1841 | #define __sanitizer_syscall_pre_vfork() __sanitizer_syscall_pre_impl_vfork()
1842 | #define __sanitizer_syscall_post_vfork(res)                                    \
1843 |   __sanitizer_syscall_post_impl_vfork(res)
1844 | #define __sanitizer_syscall_pre_sigaction(signum, act, oldact)                 \
1845 |   __sanitizer_syscall_pre_impl_sigaction((long)signum, (long)act, (long)oldact)
1846 | #define __sanitizer_syscall_post_sigaction(res, signum, act, oldact)           \
1847 |   __sanitizer_syscall_post_impl_sigaction(res, (long)signum, (long)act,        \
1848 |                                           (long)oldact)
1849 | #define __sanitizer_syscall_pre_rt_sigaction(signum, act, oldact, sz)          \
1850 |   __sanitizer_syscall_pre_impl_rt_sigaction((long)signum, (long)act,           \
```
- **Line 1841 / 第 1841 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1842 / 第 1842 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1843 / 第 1843 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1844 / 第 1844 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1845 / 第 1845 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1846 / 第 1846 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1847 / 第 1847 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1848 / 第 1848 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1849 / 第 1849 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1850 / 第 1850 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1851-1860 / 第 1851-1860 行
```cpp
1851 |                                             (long)oldact, (long)sz)
1852 | #define __sanitizer_syscall_post_rt_sigaction(res, signum, act, oldact, sz)    \
1853 |   __sanitizer_syscall_post_impl_rt_sigaction(res, (long)signum, (long)act,     \
1854 |                                              (long)oldact, (long)sz)
1855 | #define __sanitizer_syscall_pre_sigaltstack(ss, oss)                           \
1856 |   __sanitizer_syscall_pre_impl_sigaltstack((long)ss, (long)oss)
1857 | #define __sanitizer_syscall_post_sigaltstack(res, ss, oss)                     \
1858 |   __sanitizer_syscall_post_impl_sigaltstack(res, (long)ss, (long)oss)
1859 | #define __sanitizer_syscall_pre_futex(uaddr, futex_op, val, timeout, uaddr2,   \
1860 |                                       val3)                                    \
```
- **Line 1851 / 第 1851 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1852 / 第 1852 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1853 / 第 1853 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1854 / 第 1854 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1855 / 第 1855 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1856 / 第 1856 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1857 / 第 1857 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1858 / 第 1858 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1859 / 第 1859 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1860 / 第 1860 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1861-1870 / 第 1861-1870 行
```cpp
1861 |   __sanitizer_syscall_pre_impl_futex((long)uaddr, (long)futex_op, (long)val,   \
1862 |                                      (long)timeout, (long)uaddr2, (long)val3)
1863 | #define __sanitizer_syscall_post_futex(res, uaddr, futex_op, val, timeout,     \
1864 |                                        uaddr2, val3)                           \
1865 |   __sanitizer_syscall_post_impl_futex(res, (long)uaddr, (long)futex_op,        \
1866 |                                       (long)val, (long)timeout, (long)uaddr2,  \
1867 |                                       (long)val3)
1868 | 
1869 | // And now a few syscalls we don't handle yet.
1870 | #define __sanitizer_syscall_pre_afs_syscall(...)
```
- **Line 1861 / 第 1861 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1862 / 第 1862 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1863 / 第 1863 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1864 / 第 1864 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1865 / 第 1865 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1866 / 第 1866 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1867 / 第 1867 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1868 / 第 1868 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 1869 / 第 1869 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1870 / 第 1870 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1871-1880 / 第 1871-1880 行
```cpp
1871 | #define __sanitizer_syscall_pre_arch_prctl(...)
1872 | #define __sanitizer_syscall_pre_break(...)
1873 | #define __sanitizer_syscall_pre_chown32(...)
1874 | #define __sanitizer_syscall_pre_clone(...)
1875 | #define __sanitizer_syscall_pre_create_module(...)
1876 | #define __sanitizer_syscall_pre_epoll_ctl_old(...)
1877 | #define __sanitizer_syscall_pre_epoll_wait_old(...)
1878 | #define __sanitizer_syscall_pre_execve(...)
1879 | #define __sanitizer_syscall_pre_fadvise64(...)
1880 | #define __sanitizer_syscall_pre_fadvise64_64(...)
```
- **Line 1871 / 第 1871 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1872 / 第 1872 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1873 / 第 1873 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1874 / 第 1874 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1875 / 第 1875 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1876 / 第 1876 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1877 / 第 1877 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1878 / 第 1878 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1879 / 第 1879 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1880 / 第 1880 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1881-1890 / 第 1881-1890 行
```cpp
1881 | #define __sanitizer_syscall_pre_fallocate(...)
1882 | #define __sanitizer_syscall_pre_fanotify_init(...)
1883 | #define __sanitizer_syscall_pre_fanotify_mark(...)
1884 | #define __sanitizer_syscall_pre_fchown32(...)
1885 | #define __sanitizer_syscall_pre_ftime(...)
1886 | #define __sanitizer_syscall_pre_ftruncate64(...)
1887 | #define __sanitizer_syscall_pre_getegid32(...)
1888 | #define __sanitizer_syscall_pre_geteuid32(...)
1889 | #define __sanitizer_syscall_pre_getgid32(...)
1890 | #define __sanitizer_syscall_pre_getgroups32(...)
```
- **Line 1881 / 第 1881 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1882 / 第 1882 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1883 / 第 1883 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1884 / 第 1884 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1885 / 第 1885 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1886 / 第 1886 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1887 / 第 1887 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1888 / 第 1888 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1889 / 第 1889 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1890 / 第 1890 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1891-1900 / 第 1891-1900 行
```cpp
1891 | #define __sanitizer_syscall_pre_get_kernel_syms(...)
1892 | #define __sanitizer_syscall_pre_getpmsg(...)
1893 | #define __sanitizer_syscall_pre_getresgid32(...)
1894 | #define __sanitizer_syscall_pre_getresuid32(...)
1895 | #define __sanitizer_syscall_pre_get_thread_area(...)
1896 | #define __sanitizer_syscall_pre_getuid32(...)
1897 | #define __sanitizer_syscall_pre_gtty(...)
1898 | #define __sanitizer_syscall_pre_idle(...)
1899 | #define __sanitizer_syscall_pre_iopl(...)
1900 | #define __sanitizer_syscall_pre_lchown32(...)
```
- **Line 1891 / 第 1891 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1892 / 第 1892 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1893 / 第 1893 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1894 / 第 1894 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1895 / 第 1895 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1896 / 第 1896 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1897 / 第 1897 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1898 / 第 1898 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1899 / 第 1899 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1900 / 第 1900 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1901-1910 / 第 1901-1910 行
```cpp
1901 | #define __sanitizer_syscall_pre__llseek(...)
1902 | #define __sanitizer_syscall_pre_lock(...)
1903 | #define __sanitizer_syscall_pre_madvise1(...)
1904 | #define __sanitizer_syscall_pre_mmap(...)
1905 | #define __sanitizer_syscall_pre_mmap2(...)
1906 | #define __sanitizer_syscall_pre_modify_ldt(...)
1907 | #define __sanitizer_syscall_pre_mpx(...)
1908 | #define __sanitizer_syscall_pre__newselect(...)
1909 | #define __sanitizer_syscall_pre_nfsservctl(...)
1910 | #define __sanitizer_syscall_pre_oldfstat(...)
```
- **Line 1901 / 第 1901 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1902 / 第 1902 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1903 / 第 1903 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1904 / 第 1904 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1905 / 第 1905 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1906 / 第 1906 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1907 / 第 1907 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1908 / 第 1908 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1909 / 第 1909 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1910 / 第 1910 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1911-1920 / 第 1911-1920 行
```cpp
1911 | #define __sanitizer_syscall_pre_oldlstat(...)
1912 | #define __sanitizer_syscall_pre_oldolduname(...)
1913 | #define __sanitizer_syscall_pre_oldstat(...)
1914 | #define __sanitizer_syscall_pre_prctl(...)
1915 | #define __sanitizer_syscall_pre_prof(...)
1916 | #define __sanitizer_syscall_pre_profil(...)
1917 | #define __sanitizer_syscall_pre_putpmsg(...)
1918 | #define __sanitizer_syscall_pre_query_module(...)
1919 | #define __sanitizer_syscall_pre_readahead(...)
1920 | #define __sanitizer_syscall_pre_readdir(...)
```
- **Line 1911 / 第 1911 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1912 / 第 1912 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1913 / 第 1913 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1914 / 第 1914 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1915 / 第 1915 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1916 / 第 1916 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1917 / 第 1917 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1918 / 第 1918 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1919 / 第 1919 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1920 / 第 1920 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1921-1930 / 第 1921-1930 行
```cpp
1921 | #define __sanitizer_syscall_pre_rt_sigreturn(...)
1922 | #define __sanitizer_syscall_pre_rt_sigsuspend(...)
1923 | #define __sanitizer_syscall_pre_security(...)
1924 | #define __sanitizer_syscall_pre_setfsgid32(...)
1925 | #define __sanitizer_syscall_pre_setfsuid32(...)
1926 | #define __sanitizer_syscall_pre_setgid32(...)
1927 | #define __sanitizer_syscall_pre_setgroups32(...)
1928 | #define __sanitizer_syscall_pre_setregid32(...)
1929 | #define __sanitizer_syscall_pre_setresgid32(...)
1930 | #define __sanitizer_syscall_pre_setresuid32(...)
```
- **Line 1921 / 第 1921 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1922 / 第 1922 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1923 / 第 1923 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1924 / 第 1924 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1925 / 第 1925 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1926 / 第 1926 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1927 / 第 1927 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1928 / 第 1928 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1929 / 第 1929 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1930 / 第 1930 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1931-1940 / 第 1931-1940 行
```cpp
1931 | #define __sanitizer_syscall_pre_setreuid32(...)
1932 | #define __sanitizer_syscall_pre_set_thread_area(...)
1933 | #define __sanitizer_syscall_pre_setuid32(...)
1934 | #define __sanitizer_syscall_pre_sigreturn(...)
1935 | #define __sanitizer_syscall_pre_sigsuspend(...)
1936 | #define __sanitizer_syscall_pre_stty(...)
1937 | #define __sanitizer_syscall_pre_sync_file_range(...)
1938 | #define __sanitizer_syscall_pre__sysctl(...)
1939 | #define __sanitizer_syscall_pre_truncate64(...)
1940 | #define __sanitizer_syscall_pre_tuxcall(...)
```
- **Line 1931 / 第 1931 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1932 / 第 1932 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1933 / 第 1933 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1934 / 第 1934 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1935 / 第 1935 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1936 / 第 1936 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1937 / 第 1937 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1938 / 第 1938 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1939 / 第 1939 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1940 / 第 1940 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1941-1950 / 第 1941-1950 行
```cpp
1941 | #define __sanitizer_syscall_pre_ugetrlimit(...)
1942 | #define __sanitizer_syscall_pre_ulimit(...)
1943 | #define __sanitizer_syscall_pre_umount2(...)
1944 | #define __sanitizer_syscall_pre_vm86(...)
1945 | #define __sanitizer_syscall_pre_vm86old(...)
1946 | #define __sanitizer_syscall_pre_vserver(...)
1947 | 
1948 | #define __sanitizer_syscall_post_afs_syscall(res, ...)
1949 | #define __sanitizer_syscall_post_arch_prctl(res, ...)
1950 | #define __sanitizer_syscall_post_break(res, ...)
```
- **Line 1941 / 第 1941 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1942 / 第 1942 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1943 / 第 1943 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1944 / 第 1944 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1945 / 第 1945 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1946 / 第 1946 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1947 / 第 1947 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 1948 / 第 1948 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1949 / 第 1949 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1950 / 第 1950 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1951-1960 / 第 1951-1960 行
```cpp
1951 | #define __sanitizer_syscall_post_chown32(res, ...)
1952 | #define __sanitizer_syscall_post_clone(res, ...)
1953 | #define __sanitizer_syscall_post_create_module(res, ...)
1954 | #define __sanitizer_syscall_post_epoll_ctl_old(res, ...)
1955 | #define __sanitizer_syscall_post_epoll_wait_old(res, ...)
1956 | #define __sanitizer_syscall_post_execve(res, ...)
1957 | #define __sanitizer_syscall_post_fadvise64(res, ...)
1958 | #define __sanitizer_syscall_post_fadvise64_64(res, ...)
1959 | #define __sanitizer_syscall_post_fallocate(res, ...)
1960 | #define __sanitizer_syscall_post_fanotify_init(res, ...)
```
- **Line 1951 / 第 1951 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1952 / 第 1952 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1953 / 第 1953 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1954 / 第 1954 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1955 / 第 1955 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1956 / 第 1956 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1957 / 第 1957 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1958 / 第 1958 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1959 / 第 1959 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1960 / 第 1960 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1961-1970 / 第 1961-1970 行
```cpp
1961 | #define __sanitizer_syscall_post_fanotify_mark(res, ...)
1962 | #define __sanitizer_syscall_post_fchown32(res, ...)
1963 | #define __sanitizer_syscall_post_ftime(res, ...)
1964 | #define __sanitizer_syscall_post_ftruncate64(res, ...)
1965 | #define __sanitizer_syscall_post_getegid32(res, ...)
1966 | #define __sanitizer_syscall_post_geteuid32(res, ...)
1967 | #define __sanitizer_syscall_post_getgid32(res, ...)
1968 | #define __sanitizer_syscall_post_getgroups32(res, ...)
1969 | #define __sanitizer_syscall_post_get_kernel_syms(res, ...)
1970 | #define __sanitizer_syscall_post_getpmsg(res, ...)
```
- **Line 1961 / 第 1961 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1962 / 第 1962 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1963 / 第 1963 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1964 / 第 1964 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1965 / 第 1965 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1966 / 第 1966 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1967 / 第 1967 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1968 / 第 1968 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1969 / 第 1969 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1970 / 第 1970 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1971-1980 / 第 1971-1980 行
```cpp
1971 | #define __sanitizer_syscall_post_getresgid32(res, ...)
1972 | #define __sanitizer_syscall_post_getresuid32(res, ...)
1973 | #define __sanitizer_syscall_post_get_thread_area(res, ...)
1974 | #define __sanitizer_syscall_post_getuid32(res, ...)
1975 | #define __sanitizer_syscall_post_gtty(res, ...)
1976 | #define __sanitizer_syscall_post_idle(res, ...)
1977 | #define __sanitizer_syscall_post_iopl(res, ...)
1978 | #define __sanitizer_syscall_post_lchown32(res, ...)
1979 | #define __sanitizer_syscall_post__llseek(res, ...)
1980 | #define __sanitizer_syscall_post_lock(res, ...)
```
- **Line 1971 / 第 1971 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1972 / 第 1972 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1973 / 第 1973 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1974 / 第 1974 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1975 / 第 1975 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1976 / 第 1976 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1977 / 第 1977 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1978 / 第 1978 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1979 / 第 1979 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1980 / 第 1980 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1981-1990 / 第 1981-1990 行
```cpp
1981 | #define __sanitizer_syscall_post_madvise1(res, ...)
1982 | #define __sanitizer_syscall_post_mmap2(res, ...)
1983 | #define __sanitizer_syscall_post_mmap(res, ...)
1984 | #define __sanitizer_syscall_post_modify_ldt(res, ...)
1985 | #define __sanitizer_syscall_post_mpx(res, ...)
1986 | #define __sanitizer_syscall_post__newselect(res, ...)
1987 | #define __sanitizer_syscall_post_nfsservctl(res, ...)
1988 | #define __sanitizer_syscall_post_oldfstat(res, ...)
1989 | #define __sanitizer_syscall_post_oldlstat(res, ...)
1990 | #define __sanitizer_syscall_post_oldolduname(res, ...)
```
- **Line 1981 / 第 1981 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1982 / 第 1982 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1983 / 第 1983 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1984 / 第 1984 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1985 / 第 1985 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1986 / 第 1986 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1987 / 第 1987 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1988 / 第 1988 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1989 / 第 1989 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1990 / 第 1990 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1991-2000 / 第 1991-2000 行
```cpp
1991 | #define __sanitizer_syscall_post_oldstat(res, ...)
1992 | #define __sanitizer_syscall_post_prctl(res, ...)
1993 | #define __sanitizer_syscall_post_profil(res, ...)
1994 | #define __sanitizer_syscall_post_prof(res, ...)
1995 | #define __sanitizer_syscall_post_putpmsg(res, ...)
1996 | #define __sanitizer_syscall_post_query_module(res, ...)
1997 | #define __sanitizer_syscall_post_readahead(res, ...)
1998 | #define __sanitizer_syscall_post_readdir(res, ...)
1999 | #define __sanitizer_syscall_post_rt_sigreturn(res, ...)
2000 | #define __sanitizer_syscall_post_rt_sigsuspend(res, ...)
```
- **Line 1991 / 第 1991 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1992 / 第 1992 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1993 / 第 1993 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1994 / 第 1994 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1995 / 第 1995 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1996 / 第 1996 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1997 / 第 1997 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1998 / 第 1998 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1999 / 第 1999 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2000 / 第 2000 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2001-2010 / 第 2001-2010 行
```cpp
2001 | #define __sanitizer_syscall_post_security(res, ...)
2002 | #define __sanitizer_syscall_post_setfsgid32(res, ...)
2003 | #define __sanitizer_syscall_post_setfsuid32(res, ...)
2004 | #define __sanitizer_syscall_post_setgid32(res, ...)
2005 | #define __sanitizer_syscall_post_setgroups32(res, ...)
2006 | #define __sanitizer_syscall_post_setregid32(res, ...)
2007 | #define __sanitizer_syscall_post_setresgid32(res, ...)
2008 | #define __sanitizer_syscall_post_setresuid32(res, ...)
2009 | #define __sanitizer_syscall_post_setreuid32(res, ...)
2010 | #define __sanitizer_syscall_post_set_thread_area(res, ...)
```
- **Line 2001 / 第 2001 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2002 / 第 2002 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2003 / 第 2003 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2004 / 第 2004 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2005 / 第 2005 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2006 / 第 2006 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2007 / 第 2007 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2008 / 第 2008 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2009 / 第 2009 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2010 / 第 2010 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2011-2020 / 第 2011-2020 行
```cpp
2011 | #define __sanitizer_syscall_post_setuid32(res, ...)
2012 | #define __sanitizer_syscall_post_sigreturn(res, ...)
2013 | #define __sanitizer_syscall_post_sigsuspend(res, ...)
2014 | #define __sanitizer_syscall_post_stty(res, ...)
2015 | #define __sanitizer_syscall_post_sync_file_range(res, ...)
2016 | #define __sanitizer_syscall_post__sysctl(res, ...)
2017 | #define __sanitizer_syscall_post_truncate64(res, ...)
2018 | #define __sanitizer_syscall_post_tuxcall(res, ...)
2019 | #define __sanitizer_syscall_post_ugetrlimit(res, ...)
2020 | #define __sanitizer_syscall_post_ulimit(res, ...)
```
- **Line 2011 / 第 2011 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2012 / 第 2012 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2013 / 第 2013 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2014 / 第 2014 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2015 / 第 2015 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2016 / 第 2016 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2017 / 第 2017 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2018 / 第 2018 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2019 / 第 2019 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2020 / 第 2020 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2021-2030 / 第 2021-2030 行
```cpp
2021 | #define __sanitizer_syscall_post_umount2(res, ...)
2022 | #define __sanitizer_syscall_post_vm86old(res, ...)
2023 | #define __sanitizer_syscall_post_vm86(res, ...)
2024 | #define __sanitizer_syscall_post_vserver(res, ...)
2025 | 
2026 | #ifdef __cplusplus
2027 | extern "C" {
2028 | #endif
2029 | 
2030 | // Private declarations. Do not call directly from user code. Use macros above.
```
- **Line 2021 / 第 2021 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2022 / 第 2022 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2023 / 第 2023 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2024 / 第 2024 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2025 / 第 2025 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2026 / 第 2026 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 2027 / 第 2027 行**: EN: Uses C linkage so the declaration keeps a stable ABI. CN: 使用 C 链接约定，以保持稳定 ABI。
- **Line 2028 / 第 2028 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 2029 / 第 2029 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2030 / 第 2030 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 2031-2040 / 第 2031-2040 行
```cpp
2031 | void __sanitizer_syscall_pre_impl_time(long tloc);
2032 | void __sanitizer_syscall_post_impl_time(long res, long tloc);
2033 | void __sanitizer_syscall_pre_impl_stime(long tptr);
2034 | void __sanitizer_syscall_post_impl_stime(long res, long tptr);
2035 | void __sanitizer_syscall_pre_impl_gettimeofday(long tv, long tz);
2036 | void __sanitizer_syscall_post_impl_gettimeofday(long res, long tv, long tz);
2037 | void __sanitizer_syscall_pre_impl_settimeofday(long tv, long tz);
2038 | void __sanitizer_syscall_post_impl_settimeofday(long res, long tv, long tz);
2039 | void __sanitizer_syscall_pre_impl_adjtimex(long txc_p);
2040 | void __sanitizer_syscall_post_impl_adjtimex(long res, long txc_p);
```
- **Line 2031 / 第 2031 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_time`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_time`。
- **Line 2032 / 第 2032 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_time`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_time`。
- **Line 2033 / 第 2033 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_stime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_stime`。
- **Line 2034 / 第 2034 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_stime`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_stime`。
- **Line 2035 / 第 2035 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_gettimeofday`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_gettimeofday`。
- **Line 2036 / 第 2036 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_gettimeofday`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_gettimeofday`。
- **Line 2037 / 第 2037 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_settimeofday`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_settimeofday`。
- **Line 2038 / 第 2038 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_settimeofday`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_settimeofday`。
- **Line 2039 / 第 2039 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_adjtimex`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_adjtimex`。
- **Line 2040 / 第 2040 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_adjtimex`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_adjtimex`。

### Lines 2041-2050 / 第 2041-2050 行
```cpp
2041 | void __sanitizer_syscall_pre_impl_times(long tbuf);
2042 | void __sanitizer_syscall_post_impl_times(long res, long tbuf);
2043 | void __sanitizer_syscall_pre_impl_gettid();
2044 | void __sanitizer_syscall_post_impl_gettid(long res);
2045 | void __sanitizer_syscall_pre_impl_nanosleep(long rqtp, long rmtp);
2046 | void __sanitizer_syscall_post_impl_nanosleep(long res, long rqtp, long rmtp);
2047 | void __sanitizer_syscall_pre_impl_alarm(long seconds);
2048 | void __sanitizer_syscall_post_impl_alarm(long res, long seconds);
2049 | void __sanitizer_syscall_pre_impl_getpid();
2050 | void __sanitizer_syscall_post_impl_getpid(long res);
```
- **Line 2041 / 第 2041 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_times`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_times`。
- **Line 2042 / 第 2042 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_times`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_times`。
- **Line 2043 / 第 2043 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_gettid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_gettid`。
- **Line 2044 / 第 2044 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_gettid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_gettid`。
- **Line 2045 / 第 2045 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_nanosleep`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_nanosleep`。
- **Line 2046 / 第 2046 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_nanosleep`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_nanosleep`。
- **Line 2047 / 第 2047 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_alarm`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_alarm`。
- **Line 2048 / 第 2048 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_alarm`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_alarm`。
- **Line 2049 / 第 2049 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpid`。
- **Line 2050 / 第 2050 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getpid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getpid`。

### Lines 2051-2060 / 第 2051-2060 行
```cpp
2051 | void __sanitizer_syscall_pre_impl_getppid();
2052 | void __sanitizer_syscall_post_impl_getppid(long res);
2053 | void __sanitizer_syscall_pre_impl_getuid();
2054 | void __sanitizer_syscall_post_impl_getuid(long res);
2055 | void __sanitizer_syscall_pre_impl_geteuid();
2056 | void __sanitizer_syscall_post_impl_geteuid(long res);
2057 | void __sanitizer_syscall_pre_impl_getgid();
2058 | void __sanitizer_syscall_post_impl_getgid(long res);
2059 | void __sanitizer_syscall_pre_impl_getegid();
2060 | void __sanitizer_syscall_post_impl_getegid(long res);
```
- **Line 2051 / 第 2051 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getppid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getppid`。
- **Line 2052 / 第 2052 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getppid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getppid`。
- **Line 2053 / 第 2053 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getuid`。
- **Line 2054 / 第 2054 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getuid`。
- **Line 2055 / 第 2055 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_geteuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_geteuid`。
- **Line 2056 / 第 2056 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_geteuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_geteuid`。
- **Line 2057 / 第 2057 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getgid`。
- **Line 2058 / 第 2058 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getgid`。
- **Line 2059 / 第 2059 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getegid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getegid`。
- **Line 2060 / 第 2060 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getegid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getegid`。

### Lines 2061-2070 / 第 2061-2070 行
```cpp
2061 | void __sanitizer_syscall_pre_impl_getresuid(long ruid, long euid, long suid);
2062 | void __sanitizer_syscall_post_impl_getresuid(long res, long ruid, long euid,
2063 |                                              long suid);
2064 | void __sanitizer_syscall_pre_impl_getresgid(long rgid, long egid, long sgid);
2065 | void __sanitizer_syscall_post_impl_getresgid(long res, long rgid, long egid,
2066 |                                              long sgid);
2067 | void __sanitizer_syscall_pre_impl_getpgid(long pid);
2068 | void __sanitizer_syscall_post_impl_getpgid(long res, long pid);
2069 | void __sanitizer_syscall_pre_impl_getpgrp();
2070 | void __sanitizer_syscall_post_impl_getpgrp(long res);
```
- **Line 2061 / 第 2061 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getresuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getresuid`。
- **Line 2062 / 第 2062 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2063 / 第 2063 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2064 / 第 2064 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getresgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getresgid`。
- **Line 2065 / 第 2065 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2066 / 第 2066 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2067 / 第 2067 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpgid`。
- **Line 2068 / 第 2068 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getpgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getpgid`。
- **Line 2069 / 第 2069 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpgrp`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpgrp`。
- **Line 2070 / 第 2070 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getpgrp`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getpgrp`。

### Lines 2071-2080 / 第 2071-2080 行
```cpp
2071 | void __sanitizer_syscall_pre_impl_getsid(long pid);
2072 | void __sanitizer_syscall_post_impl_getsid(long res, long pid);
2073 | void __sanitizer_syscall_pre_impl_getgroups(long gidsetsize, long grouplist);
2074 | void __sanitizer_syscall_post_impl_getgroups(long res, long gidsetsize,
2075 |                                              long grouplist);
2076 | void __sanitizer_syscall_pre_impl_setregid(long rgid, long egid);
2077 | void __sanitizer_syscall_post_impl_setregid(long res, long rgid, long egid);
2078 | void __sanitizer_syscall_pre_impl_setgid(long gid);
2079 | void __sanitizer_syscall_post_impl_setgid(long res, long gid);
2080 | void __sanitizer_syscall_pre_impl_setreuid(long ruid, long euid);
```
- **Line 2071 / 第 2071 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getsid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getsid`。
- **Line 2072 / 第 2072 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getsid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getsid`。
- **Line 2073 / 第 2073 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getgroups`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getgroups`。
- **Line 2074 / 第 2074 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2075 / 第 2075 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2076 / 第 2076 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setregid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setregid`。
- **Line 2077 / 第 2077 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setregid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setregid`。
- **Line 2078 / 第 2078 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setgid`。
- **Line 2079 / 第 2079 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setgid`。
- **Line 2080 / 第 2080 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setreuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setreuid`。

### Lines 2081-2090 / 第 2081-2090 行
```cpp
2081 | void __sanitizer_syscall_post_impl_setreuid(long res, long ruid, long euid);
2082 | void __sanitizer_syscall_pre_impl_setuid(long uid);
2083 | void __sanitizer_syscall_post_impl_setuid(long res, long uid);
2084 | void __sanitizer_syscall_pre_impl_setresuid(long ruid, long euid, long suid);
2085 | void __sanitizer_syscall_post_impl_setresuid(long res, long ruid, long euid,
2086 |                                              long suid);
2087 | void __sanitizer_syscall_pre_impl_setresgid(long rgid, long egid, long sgid);
2088 | void __sanitizer_syscall_post_impl_setresgid(long res, long rgid, long egid,
2089 |                                              long sgid);
2090 | void __sanitizer_syscall_pre_impl_setfsuid(long uid);
```
- **Line 2081 / 第 2081 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setreuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setreuid`。
- **Line 2082 / 第 2082 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setuid`。
- **Line 2083 / 第 2083 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setuid`。
- **Line 2084 / 第 2084 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setresuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setresuid`。
- **Line 2085 / 第 2085 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2086 / 第 2086 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2087 / 第 2087 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setresgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setresgid`。
- **Line 2088 / 第 2088 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2089 / 第 2089 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2090 / 第 2090 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setfsuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setfsuid`。

### Lines 2091-2100 / 第 2091-2100 行
```cpp
2091 | void __sanitizer_syscall_post_impl_setfsuid(long res, long uid);
2092 | void __sanitizer_syscall_pre_impl_setfsgid(long gid);
2093 | void __sanitizer_syscall_post_impl_setfsgid(long res, long gid);
2094 | void __sanitizer_syscall_pre_impl_setpgid(long pid, long pgid);
2095 | void __sanitizer_syscall_post_impl_setpgid(long res, long pid, long pgid);
2096 | void __sanitizer_syscall_pre_impl_setsid();
2097 | void __sanitizer_syscall_post_impl_setsid(long res);
2098 | void __sanitizer_syscall_pre_impl_setgroups(long gidsetsize, long grouplist);
2099 | void __sanitizer_syscall_post_impl_setgroups(long res, long gidsetsize,
2100 |                                              long grouplist);
```
- **Line 2091 / 第 2091 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setfsuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setfsuid`。
- **Line 2092 / 第 2092 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setfsgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setfsgid`。
- **Line 2093 / 第 2093 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setfsgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setfsgid`。
- **Line 2094 / 第 2094 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setpgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setpgid`。
- **Line 2095 / 第 2095 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setpgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setpgid`。
- **Line 2096 / 第 2096 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setsid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setsid`。
- **Line 2097 / 第 2097 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setsid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setsid`。
- **Line 2098 / 第 2098 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setgroups`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setgroups`。
- **Line 2099 / 第 2099 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2100 / 第 2100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2101-2110 / 第 2101-2110 行
```cpp
2101 | void __sanitizer_syscall_pre_impl_acct(long name);
2102 | void __sanitizer_syscall_post_impl_acct(long res, long name);
2103 | void __sanitizer_syscall_pre_impl_capget(long header, long dataptr);
2104 | void __sanitizer_syscall_post_impl_capget(long res, long header, long dataptr);
2105 | void __sanitizer_syscall_pre_impl_capset(long header, long data);
2106 | void __sanitizer_syscall_post_impl_capset(long res, long header, long data);
2107 | void __sanitizer_syscall_pre_impl_personality(long personality);
2108 | void __sanitizer_syscall_post_impl_personality(long res, long personality);
2109 | void __sanitizer_syscall_pre_impl_sigpending(long set);
2110 | void __sanitizer_syscall_post_impl_sigpending(long res, long set);
```
- **Line 2101 / 第 2101 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_acct`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_acct`。
- **Line 2102 / 第 2102 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_acct`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_acct`。
- **Line 2103 / 第 2103 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_capget`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_capget`。
- **Line 2104 / 第 2104 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_capget`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_capget`。
- **Line 2105 / 第 2105 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_capset`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_capset`。
- **Line 2106 / 第 2106 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_capset`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_capset`。
- **Line 2107 / 第 2107 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_personality`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_personality`。
- **Line 2108 / 第 2108 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_personality`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_personality`。
- **Line 2109 / 第 2109 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sigpending`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sigpending`。
- **Line 2110 / 第 2110 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sigpending`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sigpending`。

### Lines 2111-2120 / 第 2111-2120 行
```cpp
2111 | void __sanitizer_syscall_pre_impl_sigprocmask(long how, long set, long oset);
2112 | void __sanitizer_syscall_post_impl_sigprocmask(long res, long how, long set,
2113 |                                                long oset);
2114 | void __sanitizer_syscall_pre_impl_getitimer(long which, long value);
2115 | void __sanitizer_syscall_post_impl_getitimer(long res, long which, long value);
2116 | void __sanitizer_syscall_pre_impl_setitimer(long which, long value,
2117 |                                             long ovalue);
2118 | void __sanitizer_syscall_post_impl_setitimer(long res, long which, long value,
2119 |                                              long ovalue);
2120 | void __sanitizer_syscall_pre_impl_timer_create(long which_clock,
```
- **Line 2111 / 第 2111 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sigprocmask`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sigprocmask`。
- **Line 2112 / 第 2112 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2113 / 第 2113 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2114 / 第 2114 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getitimer`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getitimer`。
- **Line 2115 / 第 2115 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getitimer`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getitimer`。
- **Line 2116 / 第 2116 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2117 / 第 2117 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2118 / 第 2118 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2119 / 第 2119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2120 / 第 2120 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2121-2130 / 第 2121-2130 行
```cpp
2121 |                                                long timer_event_spec,
2122 |                                                long created_timer_id);
2123 | void __sanitizer_syscall_post_impl_timer_create(long res, long which_clock,
2124 |                                                 long timer_event_spec,
2125 |                                                 long created_timer_id);
2126 | void __sanitizer_syscall_pre_impl_timer_gettime(long timer_id, long setting);
2127 | void __sanitizer_syscall_post_impl_timer_gettime(long res, long timer_id,
2128 |                                                  long setting);
2129 | void __sanitizer_syscall_pre_impl_timer_getoverrun(long timer_id);
2130 | void __sanitizer_syscall_post_impl_timer_getoverrun(long res, long timer_id);
```
- **Line 2121 / 第 2121 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2122 / 第 2122 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2123 / 第 2123 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2124 / 第 2124 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2125 / 第 2125 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2126 / 第 2126 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_timer_gettime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_timer_gettime`。
- **Line 2127 / 第 2127 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2128 / 第 2128 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2129 / 第 2129 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_timer_getoverrun`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_timer_getoverrun`。
- **Line 2130 / 第 2130 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_timer_getoverrun`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_timer_getoverrun`。

### Lines 2131-2140 / 第 2131-2140 行
```cpp
2131 | void __sanitizer_syscall_pre_impl_timer_settime(long timer_id, long flags,
2132 |                                                 long new_setting,
2133 |                                                 long old_setting);
2134 | void __sanitizer_syscall_post_impl_timer_settime(long res, long timer_id,
2135 |                                                  long flags, long new_setting,
2136 |                                                  long old_setting);
2137 | void __sanitizer_syscall_pre_impl_timer_delete(long timer_id);
2138 | void __sanitizer_syscall_post_impl_timer_delete(long res, long timer_id);
2139 | void __sanitizer_syscall_pre_impl_clock_settime(long which_clock, long tp);
2140 | void __sanitizer_syscall_post_impl_clock_settime(long res, long which_clock,
```
- **Line 2131 / 第 2131 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2132 / 第 2132 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2133 / 第 2133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2134 / 第 2134 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2135 / 第 2135 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2136 / 第 2136 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2137 / 第 2137 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_timer_delete`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_timer_delete`。
- **Line 2138 / 第 2138 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_timer_delete`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_timer_delete`。
- **Line 2139 / 第 2139 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_clock_settime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_clock_settime`。
- **Line 2140 / 第 2140 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2141-2150 / 第 2141-2150 行
```cpp
2141 |                                                  long tp);
2142 | void __sanitizer_syscall_pre_impl_clock_gettime(long which_clock, long tp);
2143 | void __sanitizer_syscall_post_impl_clock_gettime(long res, long which_clock,
2144 |                                                  long tp);
2145 | void __sanitizer_syscall_pre_impl_clock_adjtime(long which_clock, long tx);
2146 | void __sanitizer_syscall_post_impl_clock_adjtime(long res, long which_clock,
2147 |                                                  long tx);
2148 | void __sanitizer_syscall_pre_impl_clock_getres(long which_clock, long tp);
2149 | void __sanitizer_syscall_post_impl_clock_getres(long res, long which_clock,
2150 |                                                 long tp);
```
- **Line 2141 / 第 2141 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2142 / 第 2142 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_clock_gettime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_clock_gettime`。
- **Line 2143 / 第 2143 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2144 / 第 2144 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2145 / 第 2145 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_clock_adjtime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_clock_adjtime`。
- **Line 2146 / 第 2146 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2147 / 第 2147 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2148 / 第 2148 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_clock_getres`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_clock_getres`。
- **Line 2149 / 第 2149 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2150 / 第 2150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2151-2160 / 第 2151-2160 行
```cpp
2151 | void __sanitizer_syscall_pre_impl_clock_nanosleep(long which_clock, long flags,
2152 |                                                   long rqtp, long rmtp);
2153 | void __sanitizer_syscall_post_impl_clock_nanosleep(long res, long which_clock,
2154 |                                                    long flags, long rqtp,
2155 |                                                    long rmtp);
2156 | void __sanitizer_syscall_pre_impl_nice(long increment);
2157 | void __sanitizer_syscall_post_impl_nice(long res, long increment);
2158 | void __sanitizer_syscall_pre_impl_sched_setscheduler(long pid, long policy,
2159 |                                                      long param);
2160 | void __sanitizer_syscall_post_impl_sched_setscheduler(long res, long pid,
```
- **Line 2151 / 第 2151 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2152 / 第 2152 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2153 / 第 2153 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2154 / 第 2154 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2155 / 第 2155 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2156 / 第 2156 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_nice`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_nice`。
- **Line 2157 / 第 2157 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_nice`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_nice`。
- **Line 2158 / 第 2158 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2159 / 第 2159 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2160 / 第 2160 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2161-2170 / 第 2161-2170 行
```cpp
2161 |                                                       long policy, long param);
2162 | void __sanitizer_syscall_pre_impl_sched_setparam(long pid, long param);
2163 | void __sanitizer_syscall_post_impl_sched_setparam(long res, long pid,
2164 |                                                   long param);
2165 | void __sanitizer_syscall_pre_impl_sched_getscheduler(long pid);
2166 | void __sanitizer_syscall_post_impl_sched_getscheduler(long res, long pid);
2167 | void __sanitizer_syscall_pre_impl_sched_getparam(long pid, long param);
2168 | void __sanitizer_syscall_post_impl_sched_getparam(long res, long pid,
2169 |                                                   long param);
2170 | void __sanitizer_syscall_pre_impl_sched_setaffinity(long pid, long len,
```
- **Line 2161 / 第 2161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2162 / 第 2162 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sched_setparam`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sched_setparam`。
- **Line 2163 / 第 2163 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2164 / 第 2164 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2165 / 第 2165 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sched_getscheduler`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sched_getscheduler`。
- **Line 2166 / 第 2166 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sched_getscheduler`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sched_getscheduler`。
- **Line 2167 / 第 2167 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sched_getparam`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sched_getparam`。
- **Line 2168 / 第 2168 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2169 / 第 2169 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2170 / 第 2170 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2171-2180 / 第 2171-2180 行
```cpp
2171 |                                                     long user_mask_ptr);
2172 | void __sanitizer_syscall_post_impl_sched_setaffinity(long res, long pid,
2173 |                                                      long len,
2174 |                                                      long user_mask_ptr);
2175 | void __sanitizer_syscall_pre_impl_sched_getaffinity(long pid, long len,
2176 |                                                     long user_mask_ptr);
2177 | void __sanitizer_syscall_post_impl_sched_getaffinity(long res, long pid,
2178 |                                                      long len,
2179 |                                                      long user_mask_ptr);
2180 | void __sanitizer_syscall_pre_impl_sched_yield();
```
- **Line 2171 / 第 2171 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2172 / 第 2172 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2173 / 第 2173 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2174 / 第 2174 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2175 / 第 2175 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2176 / 第 2176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2177 / 第 2177 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2178 / 第 2178 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2179 / 第 2179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2180 / 第 2180 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sched_yield`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sched_yield`。

### Lines 2181-2190 / 第 2181-2190 行
```cpp
2181 | void __sanitizer_syscall_post_impl_sched_yield(long res);
2182 | void __sanitizer_syscall_pre_impl_sched_get_priority_max(long policy);
2183 | void __sanitizer_syscall_post_impl_sched_get_priority_max(long res,
2184 |                                                           long policy);
2185 | void __sanitizer_syscall_pre_impl_sched_get_priority_min(long policy);
2186 | void __sanitizer_syscall_post_impl_sched_get_priority_min(long res,
2187 |                                                           long policy);
2188 | void __sanitizer_syscall_pre_impl_sched_rr_get_interval(long pid,
2189 |                                                         long interval);
2190 | void __sanitizer_syscall_post_impl_sched_rr_get_interval(long res, long pid,
```
- **Line 2181 / 第 2181 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sched_yield`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sched_yield`。
- **Line 2182 / 第 2182 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sched_get_priority_max`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sched_get_priority_max`。
- **Line 2183 / 第 2183 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2184 / 第 2184 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2185 / 第 2185 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sched_get_priority_min`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sched_get_priority_min`。
- **Line 2186 / 第 2186 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2187 / 第 2187 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2188 / 第 2188 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2189 / 第 2189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2190 / 第 2190 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2191-2200 / 第 2191-2200 行
```cpp
2191 |                                                          long interval);
2192 | void __sanitizer_syscall_pre_impl_setpriority(long which, long who,
2193 |                                               long niceval);
2194 | void __sanitizer_syscall_post_impl_setpriority(long res, long which, long who,
2195 |                                                long niceval);
2196 | void __sanitizer_syscall_pre_impl_getpriority(long which, long who);
2197 | void __sanitizer_syscall_post_impl_getpriority(long res, long which, long who);
2198 | void __sanitizer_syscall_pre_impl_shutdown(long arg0, long arg1);
2199 | void __sanitizer_syscall_post_impl_shutdown(long res, long arg0, long arg1);
2200 | void __sanitizer_syscall_pre_impl_reboot(long magic1, long magic2, long cmd,
```
- **Line 2191 / 第 2191 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2192 / 第 2192 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2193 / 第 2193 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2194 / 第 2194 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2195 / 第 2195 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2196 / 第 2196 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpriority`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpriority`。
- **Line 2197 / 第 2197 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getpriority`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getpriority`。
- **Line 2198 / 第 2198 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_shutdown`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_shutdown`。
- **Line 2199 / 第 2199 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_shutdown`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_shutdown`。
- **Line 2200 / 第 2200 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2201-2210 / 第 2201-2210 行
```cpp
2201 |                                          long arg);
2202 | void __sanitizer_syscall_post_impl_reboot(long res, long magic1, long magic2,
2203 |                                           long cmd, long arg);
2204 | void __sanitizer_syscall_pre_impl_restart_syscall();
2205 | void __sanitizer_syscall_post_impl_restart_syscall(long res);
2206 | void __sanitizer_syscall_pre_impl_kexec_load(long entry, long nr_segments,
2207 |                                              long segments, long flags);
2208 | void __sanitizer_syscall_post_impl_kexec_load(long res, long entry,
2209 |                                               long nr_segments, long segments,
2210 |                                               long flags);
```
- **Line 2201 / 第 2201 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2202 / 第 2202 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2203 / 第 2203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2204 / 第 2204 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_restart_syscall`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_restart_syscall`。
- **Line 2205 / 第 2205 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_restart_syscall`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_restart_syscall`。
- **Line 2206 / 第 2206 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2207 / 第 2207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2208 / 第 2208 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2209 / 第 2209 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2210 / 第 2210 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2211-2220 / 第 2211-2220 行
```cpp
2211 | void __sanitizer_syscall_pre_impl_exit(long error_code);
2212 | void __sanitizer_syscall_post_impl_exit(long res, long error_code);
2213 | void __sanitizer_syscall_pre_impl_exit_group(long error_code);
2214 | void __sanitizer_syscall_post_impl_exit_group(long res, long error_code);
2215 | void __sanitizer_syscall_pre_impl_wait4(long pid, long stat_addr, long options,
2216 |                                         long ru);
2217 | void __sanitizer_syscall_post_impl_wait4(long res, long pid, long stat_addr,
2218 |                                          long options, long ru);
2219 | void __sanitizer_syscall_pre_impl_waitid(long which, long pid, long infop,
2220 |                                          long options, long ru);
```
- **Line 2211 / 第 2211 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_exit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_exit`。
- **Line 2212 / 第 2212 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_exit`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_exit`。
- **Line 2213 / 第 2213 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_exit_group`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_exit_group`。
- **Line 2214 / 第 2214 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_exit_group`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_exit_group`。
- **Line 2215 / 第 2215 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2216 / 第 2216 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2217 / 第 2217 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2218 / 第 2218 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2219 / 第 2219 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2220 / 第 2220 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2221-2230 / 第 2221-2230 行
```cpp
2221 | void __sanitizer_syscall_post_impl_waitid(long res, long which, long pid,
2222 |                                           long infop, long options, long ru);
2223 | void __sanitizer_syscall_pre_impl_waitpid(long pid, long stat_addr,
2224 |                                           long options);
2225 | void __sanitizer_syscall_post_impl_waitpid(long res, long pid, long stat_addr,
2226 |                                            long options);
2227 | void __sanitizer_syscall_pre_impl_set_tid_address(long tidptr);
2228 | void __sanitizer_syscall_post_impl_set_tid_address(long res, long tidptr);
2229 | void __sanitizer_syscall_pre_impl_init_module(long umod, long len, long uargs);
2230 | void __sanitizer_syscall_post_impl_init_module(long res, long umod, long len,
```
- **Line 2221 / 第 2221 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2222 / 第 2222 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2223 / 第 2223 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2224 / 第 2224 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2225 / 第 2225 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2226 / 第 2226 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2227 / 第 2227 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_set_tid_address`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_set_tid_address`。
- **Line 2228 / 第 2228 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_set_tid_address`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_set_tid_address`。
- **Line 2229 / 第 2229 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_init_module`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_init_module`。
- **Line 2230 / 第 2230 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2231-2240 / 第 2231-2240 行
```cpp
2231 |                                                long uargs);
2232 | void __sanitizer_syscall_pre_impl_delete_module(long name_user, long flags);
2233 | void __sanitizer_syscall_post_impl_delete_module(long res, long name_user,
2234 |                                                  long flags);
2235 | void __sanitizer_syscall_pre_impl_rt_sigprocmask(long how, long set, long oset,
2236 |                                                  long sigsetsize);
2237 | void __sanitizer_syscall_post_impl_rt_sigprocmask(long res, long how, long set,
2238 |                                                   long oset, long sigsetsize);
2239 | void __sanitizer_syscall_pre_impl_rt_sigpending(long set, long sigsetsize);
2240 | void __sanitizer_syscall_post_impl_rt_sigpending(long res, long set,
```
- **Line 2231 / 第 2231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2232 / 第 2232 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_delete_module`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_delete_module`。
- **Line 2233 / 第 2233 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2234 / 第 2234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2235 / 第 2235 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2236 / 第 2236 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2237 / 第 2237 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2238 / 第 2238 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2239 / 第 2239 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_rt_sigpending`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_rt_sigpending`。
- **Line 2240 / 第 2240 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2241-2250 / 第 2241-2250 行
```cpp
2241 |                                                  long sigsetsize);
2242 | void __sanitizer_syscall_pre_impl_rt_sigtimedwait(long uthese, long uinfo,
2243 |                                                   long uts, long sigsetsize);
2244 | void __sanitizer_syscall_post_impl_rt_sigtimedwait(long res, long uthese,
2245 |                                                    long uinfo, long uts,
2246 |                                                    long sigsetsize);
2247 | void __sanitizer_syscall_pre_impl_rt_tgsigqueueinfo(long tgid, long pid,
2248 |                                                     long sig, long uinfo);
2249 | void __sanitizer_syscall_post_impl_rt_tgsigqueueinfo(long res, long tgid,
2250 |                                                      long pid, long sig,
```
- **Line 2241 / 第 2241 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2242 / 第 2242 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2243 / 第 2243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2244 / 第 2244 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2245 / 第 2245 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2246 / 第 2246 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2247 / 第 2247 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2248 / 第 2248 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2249 / 第 2249 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2250 / 第 2250 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2251-2260 / 第 2251-2260 行
```cpp
2251 |                                                      long uinfo);
2252 | void __sanitizer_syscall_pre_impl_kill(long pid, long sig);
2253 | void __sanitizer_syscall_post_impl_kill(long res, long pid, long sig);
2254 | void __sanitizer_syscall_pre_impl_tgkill(long tgid, long pid, long sig);
2255 | void __sanitizer_syscall_post_impl_tgkill(long res, long tgid, long pid,
2256 |                                           long sig);
2257 | void __sanitizer_syscall_pre_impl_tkill(long pid, long sig);
2258 | void __sanitizer_syscall_post_impl_tkill(long res, long pid, long sig);
2259 | void __sanitizer_syscall_pre_impl_rt_sigqueueinfo(long pid, long sig,
2260 |                                                   long uinfo);
```
- **Line 2251 / 第 2251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2252 / 第 2252 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_kill`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_kill`。
- **Line 2253 / 第 2253 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_kill`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_kill`。
- **Line 2254 / 第 2254 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_tgkill`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_tgkill`。
- **Line 2255 / 第 2255 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2256 / 第 2256 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2257 / 第 2257 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_tkill`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_tkill`。
- **Line 2258 / 第 2258 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_tkill`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_tkill`。
- **Line 2259 / 第 2259 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2260 / 第 2260 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2261-2270 / 第 2261-2270 行
```cpp
2261 | void __sanitizer_syscall_post_impl_rt_sigqueueinfo(long res, long pid, long sig,
2262 |                                                    long uinfo);
2263 | void __sanitizer_syscall_pre_impl_sgetmask();
2264 | void __sanitizer_syscall_post_impl_sgetmask(long res);
2265 | void __sanitizer_syscall_pre_impl_ssetmask(long newmask);
2266 | void __sanitizer_syscall_post_impl_ssetmask(long res, long newmask);
2267 | void __sanitizer_syscall_pre_impl_signal(long sig, long handler);
2268 | void __sanitizer_syscall_post_impl_signal(long res, long sig, long handler);
2269 | void __sanitizer_syscall_pre_impl_pause();
2270 | void __sanitizer_syscall_post_impl_pause(long res);
```
- **Line 2261 / 第 2261 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2262 / 第 2262 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2263 / 第 2263 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sgetmask`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sgetmask`。
- **Line 2264 / 第 2264 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sgetmask`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sgetmask`。
- **Line 2265 / 第 2265 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ssetmask`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ssetmask`。
- **Line 2266 / 第 2266 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_ssetmask`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_ssetmask`。
- **Line 2267 / 第 2267 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_signal`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_signal`。
- **Line 2268 / 第 2268 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_signal`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_signal`。
- **Line 2269 / 第 2269 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pause`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pause`。
- **Line 2270 / 第 2270 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_pause`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_pause`。

### Lines 2271-2280 / 第 2271-2280 行
```cpp
2271 | void __sanitizer_syscall_pre_impl_sync();
2272 | void __sanitizer_syscall_post_impl_sync(long res);
2273 | void __sanitizer_syscall_pre_impl_fsync(long fd);
2274 | void __sanitizer_syscall_post_impl_fsync(long res, long fd);
2275 | void __sanitizer_syscall_pre_impl_fdatasync(long fd);
2276 | void __sanitizer_syscall_post_impl_fdatasync(long res, long fd);
2277 | void __sanitizer_syscall_pre_impl_bdflush(long func, long data);
2278 | void __sanitizer_syscall_post_impl_bdflush(long res, long func, long data);
2279 | void __sanitizer_syscall_pre_impl_mount(long dev_name, long dir_name, long type,
2280 |                                         long flags, long data);
```
- **Line 2271 / 第 2271 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sync`。
- **Line 2272 / 第 2272 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sync`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sync`。
- **Line 2273 / 第 2273 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fsync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fsync`。
- **Line 2274 / 第 2274 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fsync`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fsync`。
- **Line 2275 / 第 2275 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fdatasync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fdatasync`。
- **Line 2276 / 第 2276 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fdatasync`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fdatasync`。
- **Line 2277 / 第 2277 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_bdflush`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_bdflush`。
- **Line 2278 / 第 2278 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_bdflush`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_bdflush`。
- **Line 2279 / 第 2279 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2280 / 第 2280 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2281-2290 / 第 2281-2290 行
```cpp
2281 | void __sanitizer_syscall_post_impl_mount(long res, long dev_name, long dir_name,
2282 |                                          long type, long flags, long data);
2283 | void __sanitizer_syscall_pre_impl_umount(long name, long flags);
2284 | void __sanitizer_syscall_post_impl_umount(long res, long name, long flags);
2285 | void __sanitizer_syscall_pre_impl_oldumount(long name);
2286 | void __sanitizer_syscall_post_impl_oldumount(long res, long name);
2287 | void __sanitizer_syscall_pre_impl_truncate(long path, long length);
2288 | void __sanitizer_syscall_post_impl_truncate(long res, long path, long length);
2289 | void __sanitizer_syscall_pre_impl_ftruncate(long fd, long length);
2290 | void __sanitizer_syscall_post_impl_ftruncate(long res, long fd, long length);
```
- **Line 2281 / 第 2281 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2282 / 第 2282 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2283 / 第 2283 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_umount`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_umount`。
- **Line 2284 / 第 2284 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_umount`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_umount`。
- **Line 2285 / 第 2285 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_oldumount`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_oldumount`。
- **Line 2286 / 第 2286 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_oldumount`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_oldumount`。
- **Line 2287 / 第 2287 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_truncate`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_truncate`。
- **Line 2288 / 第 2288 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_truncate`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_truncate`。
- **Line 2289 / 第 2289 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ftruncate`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ftruncate`。
- **Line 2290 / 第 2290 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_ftruncate`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_ftruncate`。

### Lines 2291-2300 / 第 2291-2300 行
```cpp
2291 | void __sanitizer_syscall_pre_impl_stat(long filename, long statbuf);
2292 | void __sanitizer_syscall_post_impl_stat(long res, long filename, long statbuf);
2293 | void __sanitizer_syscall_pre_impl_statfs(long path, long buf);
2294 | void __sanitizer_syscall_post_impl_statfs(long res, long path, long buf);
2295 | void __sanitizer_syscall_pre_impl_statfs64(long path, long sz, long buf);
2296 | void __sanitizer_syscall_post_impl_statfs64(long res, long path, long sz,
2297 |                                             long buf);
2298 | void __sanitizer_syscall_pre_impl_fstatfs(long fd, long buf);
2299 | void __sanitizer_syscall_post_impl_fstatfs(long res, long fd, long buf);
2300 | void __sanitizer_syscall_pre_impl_fstatfs64(long fd, long sz, long buf);
```
- **Line 2291 / 第 2291 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_stat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_stat`。
- **Line 2292 / 第 2292 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_stat`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_stat`。
- **Line 2293 / 第 2293 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_statfs`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_statfs`。
- **Line 2294 / 第 2294 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_statfs`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_statfs`。
- **Line 2295 / 第 2295 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_statfs64`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_statfs64`。
- **Line 2296 / 第 2296 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2297 / 第 2297 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2298 / 第 2298 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fstatfs`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fstatfs`。
- **Line 2299 / 第 2299 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fstatfs`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fstatfs`。
- **Line 2300 / 第 2300 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fstatfs64`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fstatfs64`。

### Lines 2301-2310 / 第 2301-2310 行
```cpp
2301 | void __sanitizer_syscall_post_impl_fstatfs64(long res, long fd, long sz,
2302 |                                              long buf);
2303 | void __sanitizer_syscall_pre_impl_lstat(long filename, long statbuf);
2304 | void __sanitizer_syscall_post_impl_lstat(long res, long filename, long statbuf);
2305 | void __sanitizer_syscall_pre_impl_fstat(long fd, long statbuf);
2306 | void __sanitizer_syscall_post_impl_fstat(long res, long fd, long statbuf);
2307 | void __sanitizer_syscall_pre_impl_newstat(long filename, long statbuf);
2308 | void __sanitizer_syscall_post_impl_newstat(long res, long filename,
2309 |                                            long statbuf);
2310 | void __sanitizer_syscall_pre_impl_newlstat(long filename, long statbuf);
```
- **Line 2301 / 第 2301 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2302 / 第 2302 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2303 / 第 2303 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lstat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lstat`。
- **Line 2304 / 第 2304 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_lstat`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_lstat`。
- **Line 2305 / 第 2305 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fstat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fstat`。
- **Line 2306 / 第 2306 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fstat`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fstat`。
- **Line 2307 / 第 2307 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_newstat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_newstat`。
- **Line 2308 / 第 2308 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2309 / 第 2309 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2310 / 第 2310 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_newlstat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_newlstat`。

### Lines 2311-2320 / 第 2311-2320 行
```cpp
2311 | void __sanitizer_syscall_post_impl_newlstat(long res, long filename,
2312 |                                             long statbuf);
2313 | void __sanitizer_syscall_pre_impl_newfstat(long fd, long statbuf);
2314 | void __sanitizer_syscall_post_impl_newfstat(long res, long fd, long statbuf);
2315 | void __sanitizer_syscall_pre_impl_ustat(long dev, long ubuf);
2316 | void __sanitizer_syscall_post_impl_ustat(long res, long dev, long ubuf);
2317 | void __sanitizer_syscall_pre_impl_stat64(long filename, long statbuf);
2318 | void __sanitizer_syscall_post_impl_stat64(long res, long filename,
2319 |                                           long statbuf);
2320 | void __sanitizer_syscall_pre_impl_fstat64(long fd, long statbuf);
```
- **Line 2311 / 第 2311 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2312 / 第 2312 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2313 / 第 2313 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_newfstat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_newfstat`。
- **Line 2314 / 第 2314 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_newfstat`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_newfstat`。
- **Line 2315 / 第 2315 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ustat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ustat`。
- **Line 2316 / 第 2316 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_ustat`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_ustat`。
- **Line 2317 / 第 2317 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_stat64`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_stat64`。
- **Line 2318 / 第 2318 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2319 / 第 2319 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2320 / 第 2320 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fstat64`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fstat64`。

### Lines 2321-2330 / 第 2321-2330 行
```cpp
2321 | void __sanitizer_syscall_post_impl_fstat64(long res, long fd, long statbuf);
2322 | void __sanitizer_syscall_pre_impl_lstat64(long filename, long statbuf);
2323 | void __sanitizer_syscall_post_impl_lstat64(long res, long filename,
2324 |                                            long statbuf);
2325 | void __sanitizer_syscall_pre_impl_setxattr(long path, long name, long value,
2326 |                                            long size, long flags);
2327 | void __sanitizer_syscall_post_impl_setxattr(long res, long path, long name,
2328 |                                             long value, long size, long flags);
2329 | void __sanitizer_syscall_pre_impl_lsetxattr(long path, long name, long value,
2330 |                                             long size, long flags);
```
- **Line 2321 / 第 2321 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fstat64`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fstat64`。
- **Line 2322 / 第 2322 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lstat64`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lstat64`。
- **Line 2323 / 第 2323 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2324 / 第 2324 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2325 / 第 2325 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2326 / 第 2326 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2327 / 第 2327 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2328 / 第 2328 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2329 / 第 2329 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2330 / 第 2330 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2331-2340 / 第 2331-2340 行
```cpp
2331 | void __sanitizer_syscall_post_impl_lsetxattr(long res, long path, long name,
2332 |                                              long value, long size, long flags);
2333 | void __sanitizer_syscall_pre_impl_fsetxattr(long fd, long name, long value,
2334 |                                             long size, long flags);
2335 | void __sanitizer_syscall_post_impl_fsetxattr(long res, long fd, long name,
2336 |                                              long value, long size, long flags);
2337 | void __sanitizer_syscall_pre_impl_getxattr(long path, long name, long value,
2338 |                                            long size);
2339 | void __sanitizer_syscall_post_impl_getxattr(long res, long path, long name,
2340 |                                             long value, long size);
```
- **Line 2331 / 第 2331 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2332 / 第 2332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2333 / 第 2333 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2334 / 第 2334 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2335 / 第 2335 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2336 / 第 2336 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2337 / 第 2337 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2338 / 第 2338 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2339 / 第 2339 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2340 / 第 2340 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2341-2350 / 第 2341-2350 行
```cpp
2341 | void __sanitizer_syscall_pre_impl_lgetxattr(long path, long name, long value,
2342 |                                             long size);
2343 | void __sanitizer_syscall_post_impl_lgetxattr(long res, long path, long name,
2344 |                                              long value, long size);
2345 | void __sanitizer_syscall_pre_impl_fgetxattr(long fd, long name, long value,
2346 |                                             long size);
2347 | void __sanitizer_syscall_post_impl_fgetxattr(long res, long fd, long name,
2348 |                                              long value, long size);
2349 | void __sanitizer_syscall_pre_impl_listxattr(long path, long list, long size);
2350 | void __sanitizer_syscall_post_impl_listxattr(long res, long path, long list,
```
- **Line 2341 / 第 2341 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2342 / 第 2342 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2343 / 第 2343 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2344 / 第 2344 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2345 / 第 2345 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2346 / 第 2346 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2347 / 第 2347 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2348 / 第 2348 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2349 / 第 2349 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_listxattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_listxattr`。
- **Line 2350 / 第 2350 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2351-2360 / 第 2351-2360 行
```cpp
2351 |                                              long size);
2352 | void __sanitizer_syscall_pre_impl_llistxattr(long path, long list, long size);
2353 | void __sanitizer_syscall_post_impl_llistxattr(long res, long path, long list,
2354 |                                               long size);
2355 | void __sanitizer_syscall_pre_impl_flistxattr(long fd, long list, long size);
2356 | void __sanitizer_syscall_post_impl_flistxattr(long res, long fd, long list,
2357 |                                               long size);
2358 | void __sanitizer_syscall_pre_impl_removexattr(long path, long name);
2359 | void __sanitizer_syscall_post_impl_removexattr(long res, long path, long name);
2360 | void __sanitizer_syscall_pre_impl_lremovexattr(long path, long name);
```
- **Line 2351 / 第 2351 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2352 / 第 2352 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_llistxattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_llistxattr`。
- **Line 2353 / 第 2353 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2354 / 第 2354 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2355 / 第 2355 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_flistxattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_flistxattr`。
- **Line 2356 / 第 2356 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2357 / 第 2357 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2358 / 第 2358 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_removexattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_removexattr`。
- **Line 2359 / 第 2359 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_removexattr`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_removexattr`。
- **Line 2360 / 第 2360 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lremovexattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lremovexattr`。

### Lines 2361-2370 / 第 2361-2370 行
```cpp
2361 | void __sanitizer_syscall_post_impl_lremovexattr(long res, long path, long name);
2362 | void __sanitizer_syscall_pre_impl_fremovexattr(long fd, long name);
2363 | void __sanitizer_syscall_post_impl_fremovexattr(long res, long fd, long name);
2364 | void __sanitizer_syscall_pre_impl_brk(long brk);
2365 | void __sanitizer_syscall_post_impl_brk(long res, long brk);
2366 | void __sanitizer_syscall_pre_impl_mprotect(long start, long len, long prot);
2367 | void __sanitizer_syscall_post_impl_mprotect(long res, long start, long len,
2368 |                                             long prot);
2369 | void __sanitizer_syscall_pre_impl_mremap(long addr, long old_len, long new_len,
2370 |                                          long flags, long new_addr);
```
- **Line 2361 / 第 2361 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_lremovexattr`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_lremovexattr`。
- **Line 2362 / 第 2362 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fremovexattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fremovexattr`。
- **Line 2363 / 第 2363 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fremovexattr`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fremovexattr`。
- **Line 2364 / 第 2364 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_brk`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_brk`。
- **Line 2365 / 第 2365 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_brk`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_brk`。
- **Line 2366 / 第 2366 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mprotect`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mprotect`。
- **Line 2367 / 第 2367 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2368 / 第 2368 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2369 / 第 2369 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2370 / 第 2370 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2371-2380 / 第 2371-2380 行
```cpp
2371 | void __sanitizer_syscall_post_impl_mremap(long res, long addr, long old_len,
2372 |                                           long new_len, long flags,
2373 |                                           long new_addr);
2374 | void __sanitizer_syscall_pre_impl_remap_file_pages(long start, long size,
2375 |                                                    long prot, long pgoff,
2376 |                                                    long flags);
2377 | void __sanitizer_syscall_post_impl_remap_file_pages(long res, long start,
2378 |                                                     long size, long prot,
2379 |                                                     long pgoff, long flags);
2380 | void __sanitizer_syscall_pre_impl_msync(long start, long len, long flags);
```
- **Line 2371 / 第 2371 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2372 / 第 2372 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2373 / 第 2373 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2374 / 第 2374 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2375 / 第 2375 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2376 / 第 2376 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2377 / 第 2377 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2378 / 第 2378 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2379 / 第 2379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2380 / 第 2380 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_msync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_msync`。

### Lines 2381-2390 / 第 2381-2390 行
```cpp
2381 | void __sanitizer_syscall_post_impl_msync(long res, long start, long len,
2382 |                                          long flags);
2383 | void __sanitizer_syscall_pre_impl_munmap(long addr, long len);
2384 | void __sanitizer_syscall_post_impl_munmap(long res, long addr, long len);
2385 | void __sanitizer_syscall_pre_impl_mlock(long start, long len);
2386 | void __sanitizer_syscall_post_impl_mlock(long res, long start, long len);
2387 | void __sanitizer_syscall_pre_impl_munlock(long start, long len);
2388 | void __sanitizer_syscall_post_impl_munlock(long res, long start, long len);
2389 | void __sanitizer_syscall_pre_impl_mlockall(long flags);
2390 | void __sanitizer_syscall_post_impl_mlockall(long res, long flags);
```
- **Line 2381 / 第 2381 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2382 / 第 2382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2383 / 第 2383 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_munmap`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_munmap`。
- **Line 2384 / 第 2384 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_munmap`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_munmap`。
- **Line 2385 / 第 2385 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mlock`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mlock`。
- **Line 2386 / 第 2386 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_mlock`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_mlock`。
- **Line 2387 / 第 2387 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_munlock`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_munlock`。
- **Line 2388 / 第 2388 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_munlock`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_munlock`。
- **Line 2389 / 第 2389 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mlockall`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mlockall`。
- **Line 2390 / 第 2390 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_mlockall`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_mlockall`。

### Lines 2391-2400 / 第 2391-2400 行
```cpp
2391 | void __sanitizer_syscall_pre_impl_munlockall();
2392 | void __sanitizer_syscall_post_impl_munlockall(long res);
2393 | void __sanitizer_syscall_pre_impl_madvise(long start, long len, long behavior);
2394 | void __sanitizer_syscall_post_impl_madvise(long res, long start, long len,
2395 |                                            long behavior);
2396 | void __sanitizer_syscall_pre_impl_mincore(long start, long len, long vec);
2397 | void __sanitizer_syscall_post_impl_mincore(long res, long start, long len,
2398 |                                            long vec);
2399 | void __sanitizer_syscall_pre_impl_pivot_root(long new_root, long put_old);
2400 | void __sanitizer_syscall_post_impl_pivot_root(long res, long new_root,
```
- **Line 2391 / 第 2391 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_munlockall`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_munlockall`。
- **Line 2392 / 第 2392 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_munlockall`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_munlockall`。
- **Line 2393 / 第 2393 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_madvise`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_madvise`。
- **Line 2394 / 第 2394 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2395 / 第 2395 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2396 / 第 2396 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mincore`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mincore`。
- **Line 2397 / 第 2397 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2398 / 第 2398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2399 / 第 2399 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pivot_root`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pivot_root`。
- **Line 2400 / 第 2400 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2401-2410 / 第 2401-2410 行
```cpp
2401 |                                               long put_old);
2402 | void __sanitizer_syscall_pre_impl_chroot(long filename);
2403 | void __sanitizer_syscall_post_impl_chroot(long res, long filename);
2404 | void __sanitizer_syscall_pre_impl_mknod(long filename, long mode, long dev);
2405 | void __sanitizer_syscall_post_impl_mknod(long res, long filename, long mode,
2406 |                                          long dev);
2407 | void __sanitizer_syscall_pre_impl_link(long oldname, long newname);
2408 | void __sanitizer_syscall_post_impl_link(long res, long oldname, long newname);
2409 | void __sanitizer_syscall_pre_impl_symlink(long old, long new_);
2410 | void __sanitizer_syscall_post_impl_symlink(long res, long old, long new_);
```
- **Line 2401 / 第 2401 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2402 / 第 2402 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chroot`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chroot`。
- **Line 2403 / 第 2403 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_chroot`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_chroot`。
- **Line 2404 / 第 2404 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mknod`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mknod`。
- **Line 2405 / 第 2405 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2406 / 第 2406 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2407 / 第 2407 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_link`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_link`。
- **Line 2408 / 第 2408 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_link`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_link`。
- **Line 2409 / 第 2409 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_symlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_symlink`。
- **Line 2410 / 第 2410 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_symlink`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_symlink`。

### Lines 2411-2420 / 第 2411-2420 行
```cpp
2411 | void __sanitizer_syscall_pre_impl_unlink(long pathname);
2412 | void __sanitizer_syscall_post_impl_unlink(long res, long pathname);
2413 | void __sanitizer_syscall_pre_impl_rename(long oldname, long newname);
2414 | void __sanitizer_syscall_post_impl_rename(long res, long oldname, long newname);
2415 | void __sanitizer_syscall_pre_impl_chmod(long filename, long mode);
2416 | void __sanitizer_syscall_post_impl_chmod(long res, long filename, long mode);
2417 | void __sanitizer_syscall_pre_impl_fchmod(long fd, long mode);
2418 | void __sanitizer_syscall_post_impl_fchmod(long res, long fd, long mode);
2419 | void __sanitizer_syscall_pre_impl_fcntl(long fd, long cmd, long arg);
2420 | void __sanitizer_syscall_post_impl_fcntl(long res, long fd, long cmd, long arg);
```
- **Line 2411 / 第 2411 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_unlink`。
- **Line 2412 / 第 2412 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_unlink`。
- **Line 2413 / 第 2413 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_rename`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_rename`。
- **Line 2414 / 第 2414 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_rename`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_rename`。
- **Line 2415 / 第 2415 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chmod`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chmod`。
- **Line 2416 / 第 2416 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_chmod`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_chmod`。
- **Line 2417 / 第 2417 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchmod`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchmod`。
- **Line 2418 / 第 2418 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fchmod`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fchmod`。
- **Line 2419 / 第 2419 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fcntl`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fcntl`。
- **Line 2420 / 第 2420 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fcntl`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fcntl`。

### Lines 2421-2430 / 第 2421-2430 行
```cpp
2421 | void __sanitizer_syscall_pre_impl_fcntl64(long fd, long cmd, long arg);
2422 | void __sanitizer_syscall_post_impl_fcntl64(long res, long fd, long cmd,
2423 |                                            long arg);
2424 | void __sanitizer_syscall_pre_impl_pipe(long fildes);
2425 | void __sanitizer_syscall_post_impl_pipe(long res, long fildes);
2426 | void __sanitizer_syscall_pre_impl_pipe2(long fildes, long flags);
2427 | void __sanitizer_syscall_post_impl_pipe2(long res, long fildes, long flags);
2428 | void __sanitizer_syscall_pre_impl_dup(long fildes);
2429 | void __sanitizer_syscall_post_impl_dup(long res, long fildes);
2430 | void __sanitizer_syscall_pre_impl_dup2(long oldfd, long newfd);
```
- **Line 2421 / 第 2421 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fcntl64`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fcntl64`。
- **Line 2422 / 第 2422 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2423 / 第 2423 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2424 / 第 2424 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pipe`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pipe`。
- **Line 2425 / 第 2425 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_pipe`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_pipe`。
- **Line 2426 / 第 2426 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pipe2`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pipe2`。
- **Line 2427 / 第 2427 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_pipe2`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_pipe2`。
- **Line 2428 / 第 2428 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_dup`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_dup`。
- **Line 2429 / 第 2429 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_dup`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_dup`。
- **Line 2430 / 第 2430 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_dup2`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_dup2`。

### Lines 2431-2440 / 第 2431-2440 行
```cpp
2431 | void __sanitizer_syscall_post_impl_dup2(long res, long oldfd, long newfd);
2432 | void __sanitizer_syscall_pre_impl_dup3(long oldfd, long newfd, long flags);
2433 | void __sanitizer_syscall_post_impl_dup3(long res, long oldfd, long newfd,
2434 |                                         long flags);
2435 | void __sanitizer_syscall_pre_impl_ioperm(long from, long num, long on);
2436 | void __sanitizer_syscall_post_impl_ioperm(long res, long from, long num,
2437 |                                           long on);
2438 | void __sanitizer_syscall_pre_impl_ioctl(long fd, long cmd, long arg);
2439 | void __sanitizer_syscall_post_impl_ioctl(long res, long fd, long cmd, long arg);
2440 | void __sanitizer_syscall_pre_impl_flock(long fd, long cmd);
```
- **Line 2431 / 第 2431 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_dup2`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_dup2`。
- **Line 2432 / 第 2432 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_dup3`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_dup3`。
- **Line 2433 / 第 2433 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2434 / 第 2434 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2435 / 第 2435 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ioperm`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ioperm`。
- **Line 2436 / 第 2436 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2437 / 第 2437 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2438 / 第 2438 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ioctl`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ioctl`。
- **Line 2439 / 第 2439 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_ioctl`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_ioctl`。
- **Line 2440 / 第 2440 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_flock`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_flock`。

### Lines 2441-2450 / 第 2441-2450 行
```cpp
2441 | void __sanitizer_syscall_post_impl_flock(long res, long fd, long cmd);
2442 | void __sanitizer_syscall_pre_impl_io_setup(long nr_reqs, long ctx);
2443 | void __sanitizer_syscall_post_impl_io_setup(long res, long nr_reqs, long ctx);
2444 | void __sanitizer_syscall_pre_impl_io_destroy(long ctx);
2445 | void __sanitizer_syscall_post_impl_io_destroy(long res, long ctx);
2446 | void __sanitizer_syscall_pre_impl_io_getevents(long ctx_id, long min_nr,
2447 |                                                long nr, long events,
2448 |                                                long timeout);
2449 | void __sanitizer_syscall_post_impl_io_getevents(long res, long ctx_id,
2450 |                                                 long min_nr, long nr,
```
- **Line 2441 / 第 2441 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_flock`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_flock`。
- **Line 2442 / 第 2442 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_io_setup`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_io_setup`。
- **Line 2443 / 第 2443 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_io_setup`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_io_setup`。
- **Line 2444 / 第 2444 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_io_destroy`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_io_destroy`。
- **Line 2445 / 第 2445 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_io_destroy`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_io_destroy`。
- **Line 2446 / 第 2446 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2447 / 第 2447 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2448 / 第 2448 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2449 / 第 2449 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2450 / 第 2450 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2451-2460 / 第 2451-2460 行
```cpp
2451 |                                                 long events, long timeout);
2452 | void __sanitizer_syscall_pre_impl_io_submit(long ctx_id, long arg1, long arg2);
2453 | void __sanitizer_syscall_post_impl_io_submit(long res, long ctx_id, long arg1,
2454 |                                              long arg2);
2455 | void __sanitizer_syscall_pre_impl_io_cancel(long ctx_id, long iocb,
2456 |                                             long result);
2457 | void __sanitizer_syscall_post_impl_io_cancel(long res, long ctx_id, long iocb,
2458 |                                              long result);
2459 | void __sanitizer_syscall_pre_impl_sendfile(long out_fd, long in_fd, long offset,
2460 |                                            long count);
```
- **Line 2451 / 第 2451 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2452 / 第 2452 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_io_submit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_io_submit`。
- **Line 2453 / 第 2453 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2454 / 第 2454 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2455 / 第 2455 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2456 / 第 2456 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2457 / 第 2457 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2458 / 第 2458 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2459 / 第 2459 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2460 / 第 2460 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2461-2470 / 第 2461-2470 行
```cpp
2461 | void __sanitizer_syscall_post_impl_sendfile(long res, long out_fd, long in_fd,
2462 |                                             long offset, long count);
2463 | void __sanitizer_syscall_pre_impl_sendfile64(long out_fd, long in_fd,
2464 |                                              long offset, long count);
2465 | void __sanitizer_syscall_post_impl_sendfile64(long res, long out_fd, long in_fd,
2466 |                                               long offset, long count);
2467 | void __sanitizer_syscall_pre_impl_readlink(long path, long buf, long bufsiz);
2468 | void __sanitizer_syscall_post_impl_readlink(long res, long path, long buf,
2469 |                                             long bufsiz);
2470 | void __sanitizer_syscall_pre_impl_creat(long pathname, long mode);
```
- **Line 2461 / 第 2461 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2462 / 第 2462 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2463 / 第 2463 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2464 / 第 2464 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2465 / 第 2465 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2466 / 第 2466 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2467 / 第 2467 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_readlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_readlink`。
- **Line 2468 / 第 2468 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2469 / 第 2469 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2470 / 第 2470 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_creat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_creat`。

### Lines 2471-2480 / 第 2471-2480 行
```cpp
2471 | void __sanitizer_syscall_post_impl_creat(long res, long pathname, long mode);
2472 | void __sanitizer_syscall_pre_impl_open(long filename, long flags, long mode);
2473 | void __sanitizer_syscall_post_impl_open(long res, long filename, long flags,
2474 |                                         long mode);
2475 | void __sanitizer_syscall_pre_impl_close(long fd);
2476 | void __sanitizer_syscall_post_impl_close(long res, long fd);
2477 | void __sanitizer_syscall_pre_impl_access(long filename, long mode);
2478 | void __sanitizer_syscall_post_impl_access(long res, long filename, long mode);
2479 | void __sanitizer_syscall_pre_impl_vhangup();
2480 | void __sanitizer_syscall_post_impl_vhangup(long res);
```
- **Line 2471 / 第 2471 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_creat`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_creat`。
- **Line 2472 / 第 2472 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_open`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_open`。
- **Line 2473 / 第 2473 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2474 / 第 2474 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2475 / 第 2475 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_close`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_close`。
- **Line 2476 / 第 2476 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_close`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_close`。
- **Line 2477 / 第 2477 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_access`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_access`。
- **Line 2478 / 第 2478 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_access`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_access`。
- **Line 2479 / 第 2479 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_vhangup`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_vhangup`。
- **Line 2480 / 第 2480 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_vhangup`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_vhangup`。

### Lines 2481-2490 / 第 2481-2490 行
```cpp
2481 | void __sanitizer_syscall_pre_impl_chown(long filename, long user, long group);
2482 | void __sanitizer_syscall_post_impl_chown(long res, long filename, long user,
2483 |                                          long group);
2484 | void __sanitizer_syscall_pre_impl_lchown(long filename, long user, long group);
2485 | void __sanitizer_syscall_post_impl_lchown(long res, long filename, long user,
2486 |                                           long group);
2487 | void __sanitizer_syscall_pre_impl_fchown(long fd, long user, long group);
2488 | void __sanitizer_syscall_post_impl_fchown(long res, long fd, long user,
2489 |                                           long group);
2490 | void __sanitizer_syscall_pre_impl_chown16(long filename, long user, long group);
```
- **Line 2481 / 第 2481 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chown`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chown`。
- **Line 2482 / 第 2482 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2483 / 第 2483 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2484 / 第 2484 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lchown`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lchown`。
- **Line 2485 / 第 2485 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2486 / 第 2486 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2487 / 第 2487 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchown`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchown`。
- **Line 2488 / 第 2488 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2489 / 第 2489 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2490 / 第 2490 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chown16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chown16`。

### Lines 2491-2500 / 第 2491-2500 行
```cpp
2491 | void __sanitizer_syscall_post_impl_chown16(long res, long filename, long user,
2492 |                                            long group);
2493 | void __sanitizer_syscall_pre_impl_lchown16(long filename, long user,
2494 |                                            long group);
2495 | void __sanitizer_syscall_post_impl_lchown16(long res, long filename, long user,
2496 |                                             long group);
2497 | void __sanitizer_syscall_pre_impl_fchown16(long fd, long user, long group);
2498 | void __sanitizer_syscall_post_impl_fchown16(long res, long fd, long user,
2499 |                                             long group);
2500 | void __sanitizer_syscall_pre_impl_setregid16(long rgid, long egid);
```
- **Line 2491 / 第 2491 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2492 / 第 2492 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2493 / 第 2493 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2494 / 第 2494 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2495 / 第 2495 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2496 / 第 2496 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2497 / 第 2497 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchown16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchown16`。
- **Line 2498 / 第 2498 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2499 / 第 2499 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2500 / 第 2500 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setregid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setregid16`。

### Lines 2501-2510 / 第 2501-2510 行
```cpp
2501 | void __sanitizer_syscall_post_impl_setregid16(long res, long rgid, long egid);
2502 | void __sanitizer_syscall_pre_impl_setgid16(long gid);
2503 | void __sanitizer_syscall_post_impl_setgid16(long res, long gid);
2504 | void __sanitizer_syscall_pre_impl_setreuid16(long ruid, long euid);
2505 | void __sanitizer_syscall_post_impl_setreuid16(long res, long ruid, long euid);
2506 | void __sanitizer_syscall_pre_impl_setuid16(long uid);
2507 | void __sanitizer_syscall_post_impl_setuid16(long res, long uid);
2508 | void __sanitizer_syscall_pre_impl_setresuid16(long ruid, long euid, long suid);
2509 | void __sanitizer_syscall_post_impl_setresuid16(long res, long ruid, long euid,
2510 |                                                long suid);
```
- **Line 2501 / 第 2501 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setregid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setregid16`。
- **Line 2502 / 第 2502 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setgid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setgid16`。
- **Line 2503 / 第 2503 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setgid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setgid16`。
- **Line 2504 / 第 2504 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setreuid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setreuid16`。
- **Line 2505 / 第 2505 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setreuid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setreuid16`。
- **Line 2506 / 第 2506 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setuid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setuid16`。
- **Line 2507 / 第 2507 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setuid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setuid16`。
- **Line 2508 / 第 2508 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setresuid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setresuid16`。
- **Line 2509 / 第 2509 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2510 / 第 2510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2511-2520 / 第 2511-2520 行
```cpp
2511 | void __sanitizer_syscall_pre_impl_getresuid16(long ruid, long euid, long suid);
2512 | void __sanitizer_syscall_post_impl_getresuid16(long res, long ruid, long euid,
2513 |                                                long suid);
2514 | void __sanitizer_syscall_pre_impl_setresgid16(long rgid, long egid, long sgid);
2515 | void __sanitizer_syscall_post_impl_setresgid16(long res, long rgid, long egid,
2516 |                                                long sgid);
2517 | void __sanitizer_syscall_pre_impl_getresgid16(long rgid, long egid, long sgid);
2518 | void __sanitizer_syscall_post_impl_getresgid16(long res, long rgid, long egid,
2519 |                                                long sgid);
2520 | void __sanitizer_syscall_pre_impl_setfsuid16(long uid);
```
- **Line 2511 / 第 2511 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getresuid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getresuid16`。
- **Line 2512 / 第 2512 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2513 / 第 2513 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2514 / 第 2514 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setresgid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setresgid16`。
- **Line 2515 / 第 2515 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2516 / 第 2516 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2517 / 第 2517 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getresgid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getresgid16`。
- **Line 2518 / 第 2518 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2519 / 第 2519 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2520 / 第 2520 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setfsuid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setfsuid16`。

### Lines 2521-2530 / 第 2521-2530 行
```cpp
2521 | void __sanitizer_syscall_post_impl_setfsuid16(long res, long uid);
2522 | void __sanitizer_syscall_pre_impl_setfsgid16(long gid);
2523 | void __sanitizer_syscall_post_impl_setfsgid16(long res, long gid);
2524 | void __sanitizer_syscall_pre_impl_getgroups16(long gidsetsize, long grouplist);
2525 | void __sanitizer_syscall_post_impl_getgroups16(long res, long gidsetsize,
2526 |                                                long grouplist);
2527 | void __sanitizer_syscall_pre_impl_setgroups16(long gidsetsize, long grouplist);
2528 | void __sanitizer_syscall_post_impl_setgroups16(long res, long gidsetsize,
2529 |                                                long grouplist);
2530 | void __sanitizer_syscall_pre_impl_getuid16();
```
- **Line 2521 / 第 2521 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setfsuid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setfsuid16`。
- **Line 2522 / 第 2522 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setfsgid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setfsgid16`。
- **Line 2523 / 第 2523 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setfsgid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setfsgid16`。
- **Line 2524 / 第 2524 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getgroups16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getgroups16`。
- **Line 2525 / 第 2525 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2526 / 第 2526 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2527 / 第 2527 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setgroups16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setgroups16`。
- **Line 2528 / 第 2528 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2529 / 第 2529 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2530 / 第 2530 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getuid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getuid16`。

### Lines 2531-2540 / 第 2531-2540 行
```cpp
2531 | void __sanitizer_syscall_post_impl_getuid16(long res);
2532 | void __sanitizer_syscall_pre_impl_geteuid16();
2533 | void __sanitizer_syscall_post_impl_geteuid16(long res);
2534 | void __sanitizer_syscall_pre_impl_getgid16();
2535 | void __sanitizer_syscall_post_impl_getgid16(long res);
2536 | void __sanitizer_syscall_pre_impl_getegid16();
2537 | void __sanitizer_syscall_post_impl_getegid16(long res);
2538 | void __sanitizer_syscall_pre_impl_utime(long filename, long times);
2539 | void __sanitizer_syscall_post_impl_utime(long res, long filename, long times);
2540 | void __sanitizer_syscall_pre_impl_utimes(long filename, long utimes);
```
- **Line 2531 / 第 2531 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getuid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getuid16`。
- **Line 2532 / 第 2532 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_geteuid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_geteuid16`。
- **Line 2533 / 第 2533 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_geteuid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_geteuid16`。
- **Line 2534 / 第 2534 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getgid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getgid16`。
- **Line 2535 / 第 2535 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getgid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getgid16`。
- **Line 2536 / 第 2536 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getegid16`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getegid16`。
- **Line 2537 / 第 2537 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getegid16`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getegid16`。
- **Line 2538 / 第 2538 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_utime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_utime`。
- **Line 2539 / 第 2539 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_utime`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_utime`。
- **Line 2540 / 第 2540 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_utimes`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_utimes`。

### Lines 2541-2550 / 第 2541-2550 行
```cpp
2541 | void __sanitizer_syscall_post_impl_utimes(long res, long filename, long utimes);
2542 | void __sanitizer_syscall_pre_impl_lseek(long fd, long offset, long origin);
2543 | void __sanitizer_syscall_post_impl_lseek(long res, long fd, long offset,
2544 |                                          long origin);
2545 | void __sanitizer_syscall_pre_impl_llseek(long fd, long offset_high,
2546 |                                          long offset_low, long result,
2547 |                                          long origin);
2548 | void __sanitizer_syscall_post_impl_llseek(long res, long fd, long offset_high,
2549 |                                           long offset_low, long result,
2550 |                                           long origin);
```
- **Line 2541 / 第 2541 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_utimes`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_utimes`。
- **Line 2542 / 第 2542 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lseek`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lseek`。
- **Line 2543 / 第 2543 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2544 / 第 2544 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2545 / 第 2545 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2546 / 第 2546 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2547 / 第 2547 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2548 / 第 2548 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2549 / 第 2549 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2550 / 第 2550 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2551-2560 / 第 2551-2560 行
```cpp
2551 | void __sanitizer_syscall_pre_impl_read(long fd, long buf, long count);
2552 | void __sanitizer_syscall_post_impl_read(long res, long fd, long buf,
2553 |                                         long count);
2554 | void __sanitizer_syscall_pre_impl_readv(long fd, long vec, long vlen);
2555 | void __sanitizer_syscall_post_impl_readv(long res, long fd, long vec,
2556 |                                          long vlen);
2557 | void __sanitizer_syscall_pre_impl_write(long fd, long buf, long count);
2558 | void __sanitizer_syscall_post_impl_write(long res, long fd, long buf,
2559 |                                          long count);
2560 | void __sanitizer_syscall_pre_impl_writev(long fd, long vec, long vlen);
```
- **Line 2551 / 第 2551 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_read`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_read`。
- **Line 2552 / 第 2552 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2553 / 第 2553 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2554 / 第 2554 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_readv`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_readv`。
- **Line 2555 / 第 2555 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2556 / 第 2556 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2557 / 第 2557 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_write`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_write`。
- **Line 2558 / 第 2558 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2559 / 第 2559 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2560 / 第 2560 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_writev`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_writev`。

### Lines 2561-2570 / 第 2561-2570 行
```cpp
2561 | void __sanitizer_syscall_post_impl_writev(long res, long fd, long vec,
2562 |                                           long vlen);
2563 | 
2564 | #ifdef _LP64
2565 | void __sanitizer_syscall_pre_impl_pread64(long fd, long buf, long count,
2566 |                                           long pos);
2567 | void __sanitizer_syscall_post_impl_pread64(long res, long fd, long buf,
2568 |                                            long count, long pos);
2569 | void __sanitizer_syscall_pre_impl_pwrite64(long fd, long buf, long count,
2570 |                                            long pos);
```
- **Line 2561 / 第 2561 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2562 / 第 2562 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2563 / 第 2563 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2564 / 第 2564 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 2565 / 第 2565 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2566 / 第 2566 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2567 / 第 2567 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2568 / 第 2568 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2569 / 第 2569 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2570 / 第 2570 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2571-2580 / 第 2571-2580 行
```cpp
2571 | void __sanitizer_syscall_post_impl_pwrite64(long res, long fd, long buf,
2572 |                                             long count, long pos);
2573 | #else
2574 | void __sanitizer_syscall_pre_impl_pread64(long fd, long buf, long count,
2575 |                                           long pos0, long pos1);
2576 | void __sanitizer_syscall_post_impl_pread64(long res, long fd, long buf,
2577 |                                            long count, long pos0, long pos1);
2578 | void __sanitizer_syscall_pre_impl_pwrite64(long fd, long buf, long count,
2579 |                                            long pos0, long pos1);
2580 | void __sanitizer_syscall_post_impl_pwrite64(long res, long fd, long buf,
```
- **Line 2571 / 第 2571 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2572 / 第 2572 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2573 / 第 2573 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 2574 / 第 2574 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2575 / 第 2575 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2576 / 第 2576 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2577 / 第 2577 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2578 / 第 2578 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2579 / 第 2579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2580 / 第 2580 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2581-2590 / 第 2581-2590 行
```cpp
2581 |                                             long count, long pos0, long pos1);
2582 | #endif
2583 | 
2584 | void __sanitizer_syscall_pre_impl_preadv(long fd, long vec, long vlen,
2585 |                                          long pos_l, long pos_h);
2586 | void __sanitizer_syscall_post_impl_preadv(long res, long fd, long vec,
2587 |                                           long vlen, long pos_l, long pos_h);
2588 | void __sanitizer_syscall_pre_impl_pwritev(long fd, long vec, long vlen,
2589 |                                           long pos_l, long pos_h);
2590 | void __sanitizer_syscall_post_impl_pwritev(long res, long fd, long vec,
```
- **Line 2581 / 第 2581 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2582 / 第 2582 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 2583 / 第 2583 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2584 / 第 2584 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2585 / 第 2585 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2586 / 第 2586 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2587 / 第 2587 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2588 / 第 2588 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2589 / 第 2589 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2590 / 第 2590 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2591-2600 / 第 2591-2600 行
```cpp
2591 |                                            long vlen, long pos_l, long pos_h);
2592 | void __sanitizer_syscall_pre_impl_getcwd(long buf, long size);
2593 | void __sanitizer_syscall_post_impl_getcwd(long res, long buf, long size);
2594 | void __sanitizer_syscall_pre_impl_mkdir(long pathname, long mode);
2595 | void __sanitizer_syscall_post_impl_mkdir(long res, long pathname, long mode);
2596 | void __sanitizer_syscall_pre_impl_chdir(long filename);
2597 | void __sanitizer_syscall_post_impl_chdir(long res, long filename);
2598 | void __sanitizer_syscall_pre_impl_fchdir(long fd);
2599 | void __sanitizer_syscall_post_impl_fchdir(long res, long fd);
2600 | void __sanitizer_syscall_pre_impl_rmdir(long pathname);
```
- **Line 2591 / 第 2591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2592 / 第 2592 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getcwd`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getcwd`。
- **Line 2593 / 第 2593 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getcwd`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getcwd`。
- **Line 2594 / 第 2594 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mkdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mkdir`。
- **Line 2595 / 第 2595 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_mkdir`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_mkdir`。
- **Line 2596 / 第 2596 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chdir`。
- **Line 2597 / 第 2597 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_chdir`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_chdir`。
- **Line 2598 / 第 2598 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchdir`。
- **Line 2599 / 第 2599 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fchdir`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fchdir`。
- **Line 2600 / 第 2600 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_rmdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_rmdir`。

### Lines 2601-2610 / 第 2601-2610 行
```cpp
2601 | void __sanitizer_syscall_post_impl_rmdir(long res, long pathname);
2602 | void __sanitizer_syscall_pre_impl_lookup_dcookie(long cookie64, long buf,
2603 |                                                  long len);
2604 | void __sanitizer_syscall_post_impl_lookup_dcookie(long res, long cookie64,
2605 |                                                   long buf, long len);
2606 | void __sanitizer_syscall_pre_impl_quotactl(long cmd, long special, long id,
2607 |                                            long addr);
2608 | void __sanitizer_syscall_post_impl_quotactl(long res, long cmd, long special,
2609 |                                             long id, long addr);
2610 | void __sanitizer_syscall_pre_impl_getdents(long fd, long dirent, long count);
```
- **Line 2601 / 第 2601 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_rmdir`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_rmdir`。
- **Line 2602 / 第 2602 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2603 / 第 2603 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2604 / 第 2604 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2605 / 第 2605 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2606 / 第 2606 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2607 / 第 2607 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2608 / 第 2608 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2609 / 第 2609 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2610 / 第 2610 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getdents`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getdents`。

### Lines 2611-2620 / 第 2611-2620 行
```cpp
2611 | void __sanitizer_syscall_post_impl_getdents(long res, long fd, long dirent,
2612 |                                             long count);
2613 | void __sanitizer_syscall_pre_impl_getdents64(long fd, long dirent, long count);
2614 | void __sanitizer_syscall_post_impl_getdents64(long res, long fd, long dirent,
2615 |                                               long count);
2616 | void __sanitizer_syscall_pre_impl_setsockopt(long fd, long level, long optname,
2617 |                                              long optval, long optlen);
2618 | void __sanitizer_syscall_post_impl_setsockopt(long res, long fd, long level,
2619 |                                               long optname, long optval,
2620 |                                               long optlen);
```
- **Line 2611 / 第 2611 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2612 / 第 2612 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2613 / 第 2613 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getdents64`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getdents64`。
- **Line 2614 / 第 2614 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2615 / 第 2615 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2616 / 第 2616 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2617 / 第 2617 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2618 / 第 2618 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2619 / 第 2619 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2620 / 第 2620 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2621-2630 / 第 2621-2630 行
```cpp
2621 | void __sanitizer_syscall_pre_impl_getsockopt(long fd, long level, long optname,
2622 |                                              long optval, long optlen);
2623 | void __sanitizer_syscall_post_impl_getsockopt(long res, long fd, long level,
2624 |                                               long optname, long optval,
2625 |                                               long optlen);
2626 | void __sanitizer_syscall_pre_impl_bind(long arg0, long arg1, long arg2);
2627 | void __sanitizer_syscall_post_impl_bind(long res, long arg0, long arg1,
2628 |                                         long arg2);
2629 | void __sanitizer_syscall_pre_impl_connect(long arg0, long arg1, long arg2);
2630 | void __sanitizer_syscall_post_impl_connect(long res, long arg0, long arg1,
```
- **Line 2621 / 第 2621 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2622 / 第 2622 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2623 / 第 2623 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2624 / 第 2624 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2625 / 第 2625 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2626 / 第 2626 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_bind`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_bind`。
- **Line 2627 / 第 2627 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2628 / 第 2628 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2629 / 第 2629 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_connect`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_connect`。
- **Line 2630 / 第 2630 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2631-2640 / 第 2631-2640 行
```cpp
2631 |                                            long arg2);
2632 | void __sanitizer_syscall_pre_impl_accept(long arg0, long arg1, long arg2);
2633 | void __sanitizer_syscall_post_impl_accept(long res, long arg0, long arg1,
2634 |                                           long arg2);
2635 | void __sanitizer_syscall_pre_impl_accept4(long arg0, long arg1, long arg2,
2636 |                                           long arg3);
2637 | void __sanitizer_syscall_post_impl_accept4(long res, long arg0, long arg1,
2638 |                                            long arg2, long arg3);
2639 | void __sanitizer_syscall_pre_impl_getsockname(long arg0, long arg1, long arg2);
2640 | void __sanitizer_syscall_post_impl_getsockname(long res, long arg0, long arg1,
```
- **Line 2631 / 第 2631 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2632 / 第 2632 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_accept`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_accept`。
- **Line 2633 / 第 2633 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2634 / 第 2634 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2635 / 第 2635 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2636 / 第 2636 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2637 / 第 2637 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2638 / 第 2638 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2639 / 第 2639 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getsockname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getsockname`。
- **Line 2640 / 第 2640 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2641-2650 / 第 2641-2650 行
```cpp
2641 |                                                long arg2);
2642 | void __sanitizer_syscall_pre_impl_getpeername(long arg0, long arg1, long arg2);
2643 | void __sanitizer_syscall_post_impl_getpeername(long res, long arg0, long arg1,
2644 |                                                long arg2);
2645 | void __sanitizer_syscall_pre_impl_send(long arg0, long arg1, long arg2,
2646 |                                        long arg3);
2647 | void __sanitizer_syscall_post_impl_send(long res, long arg0, long arg1,
2648 |                                         long arg2, long arg3);
2649 | void __sanitizer_syscall_pre_impl_sendto(long arg0, long arg1, long arg2,
2650 |                                          long arg3, long arg4, long arg5);
```
- **Line 2641 / 第 2641 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2642 / 第 2642 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpeername`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpeername`。
- **Line 2643 / 第 2643 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2644 / 第 2644 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2645 / 第 2645 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2646 / 第 2646 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2647 / 第 2647 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2648 / 第 2648 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2649 / 第 2649 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2650 / 第 2650 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2651-2660 / 第 2651-2660 行
```cpp
2651 | void __sanitizer_syscall_post_impl_sendto(long res, long arg0, long arg1,
2652 |                                           long arg2, long arg3, long arg4,
2653 |                                           long arg5);
2654 | void __sanitizer_syscall_pre_impl_sendmsg(long fd, long msg, long flags);
2655 | void __sanitizer_syscall_post_impl_sendmsg(long res, long fd, long msg,
2656 |                                            long flags);
2657 | void __sanitizer_syscall_pre_impl_sendmmsg(long fd, long msg, long vlen,
2658 |                                            long flags);
2659 | void __sanitizer_syscall_post_impl_sendmmsg(long res, long fd, long msg,
2660 |                                             long vlen, long flags);
```
- **Line 2651 / 第 2651 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2652 / 第 2652 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2653 / 第 2653 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2654 / 第 2654 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sendmsg`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sendmsg`。
- **Line 2655 / 第 2655 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2656 / 第 2656 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2657 / 第 2657 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2658 / 第 2658 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2659 / 第 2659 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2660 / 第 2660 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2661-2670 / 第 2661-2670 行
```cpp
2661 | void __sanitizer_syscall_pre_impl_recv(long arg0, long arg1, long arg2,
2662 |                                        long arg3);
2663 | void __sanitizer_syscall_post_impl_recv(long res, long arg0, long arg1,
2664 |                                         long arg2, long arg3);
2665 | void __sanitizer_syscall_pre_impl_recvfrom(long arg0, long arg1, long arg2,
2666 |                                            long arg3, long arg4, long arg5);
2667 | void __sanitizer_syscall_post_impl_recvfrom(long res, long arg0, long arg1,
2668 |                                             long arg2, long arg3, long arg4,
2669 |                                             long arg5);
2670 | void __sanitizer_syscall_pre_impl_recvmsg(long fd, long msg, long flags);
```
- **Line 2661 / 第 2661 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2662 / 第 2662 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2663 / 第 2663 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2664 / 第 2664 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2665 / 第 2665 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2666 / 第 2666 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2667 / 第 2667 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2668 / 第 2668 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2669 / 第 2669 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2670 / 第 2670 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_recvmsg`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_recvmsg`。

### Lines 2671-2680 / 第 2671-2680 行
```cpp
2671 | void __sanitizer_syscall_post_impl_recvmsg(long res, long fd, long msg,
2672 |                                            long flags);
2673 | void __sanitizer_syscall_pre_impl_recvmmsg(long fd, long msg, long vlen,
2674 |                                            long flags, long timeout);
2675 | void __sanitizer_syscall_post_impl_recvmmsg(long res, long fd, long msg,
2676 |                                             long vlen, long flags,
2677 |                                             long timeout);
2678 | void __sanitizer_syscall_pre_impl_socket(long arg0, long arg1, long arg2);
2679 | void __sanitizer_syscall_post_impl_socket(long res, long arg0, long arg1,
2680 |                                           long arg2);
```
- **Line 2671 / 第 2671 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2672 / 第 2672 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2673 / 第 2673 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2674 / 第 2674 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2675 / 第 2675 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2676 / 第 2676 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2677 / 第 2677 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2678 / 第 2678 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_socket`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_socket`。
- **Line 2679 / 第 2679 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2680 / 第 2680 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2681-2690 / 第 2681-2690 行
```cpp
2681 | void __sanitizer_syscall_pre_impl_socketpair(long arg0, long arg1, long arg2,
2682 |                                              long arg3);
2683 | void __sanitizer_syscall_post_impl_socketpair(long res, long arg0, long arg1,
2684 |                                               long arg2, long arg3);
2685 | void __sanitizer_syscall_pre_impl_socketcall(long call, long args);
2686 | void __sanitizer_syscall_post_impl_socketcall(long res, long call, long args);
2687 | void __sanitizer_syscall_pre_impl_listen(long arg0, long arg1);
2688 | void __sanitizer_syscall_post_impl_listen(long res, long arg0, long arg1);
2689 | void __sanitizer_syscall_pre_impl_poll(long ufds, long nfds, long timeout);
2690 | void __sanitizer_syscall_post_impl_poll(long res, long ufds, long nfds,
```
- **Line 2681 / 第 2681 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2682 / 第 2682 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2683 / 第 2683 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2684 / 第 2684 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2685 / 第 2685 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_socketcall`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_socketcall`。
- **Line 2686 / 第 2686 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_socketcall`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_socketcall`。
- **Line 2687 / 第 2687 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_listen`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_listen`。
- **Line 2688 / 第 2688 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_listen`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_listen`。
- **Line 2689 / 第 2689 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_poll`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_poll`。
- **Line 2690 / 第 2690 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2691-2700 / 第 2691-2700 行
```cpp
2691 |                                         long timeout);
2692 | void __sanitizer_syscall_pre_impl_select(long n, long inp, long outp, long exp,
2693 |                                          long tvp);
2694 | void __sanitizer_syscall_post_impl_select(long res, long n, long inp, long outp,
2695 |                                           long exp, long tvp);
2696 | void __sanitizer_syscall_pre_impl_old_select(long arg);
2697 | void __sanitizer_syscall_post_impl_old_select(long res, long arg);
2698 | void __sanitizer_syscall_pre_impl_epoll_create(long size);
2699 | void __sanitizer_syscall_post_impl_epoll_create(long res, long size);
2700 | void __sanitizer_syscall_pre_impl_epoll_create1(long flags);
```
- **Line 2691 / 第 2691 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2692 / 第 2692 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2693 / 第 2693 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2694 / 第 2694 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2695 / 第 2695 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2696 / 第 2696 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_old_select`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_old_select`。
- **Line 2697 / 第 2697 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_old_select`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_old_select`。
- **Line 2698 / 第 2698 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_epoll_create`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_epoll_create`。
- **Line 2699 / 第 2699 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_epoll_create`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_epoll_create`。
- **Line 2700 / 第 2700 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_epoll_create1`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_epoll_create1`。

### Lines 2701-2710 / 第 2701-2710 行
```cpp
2701 | void __sanitizer_syscall_post_impl_epoll_create1(long res, long flags);
2702 | void __sanitizer_syscall_pre_impl_epoll_ctl(long epfd, long op, long fd,
2703 |                                             long event);
2704 | void __sanitizer_syscall_post_impl_epoll_ctl(long res, long epfd, long op,
2705 |                                              long fd, long event);
2706 | void __sanitizer_syscall_pre_impl_epoll_wait(long epfd, long events,
2707 |                                              long maxevents, long timeout);
2708 | void __sanitizer_syscall_post_impl_epoll_wait(long res, long epfd, long events,
2709 |                                               long maxevents, long timeout);
2710 | void __sanitizer_syscall_pre_impl_epoll_pwait(long epfd, long events,
```
- **Line 2701 / 第 2701 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_epoll_create1`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_epoll_create1`。
- **Line 2702 / 第 2702 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2703 / 第 2703 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2704 / 第 2704 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2705 / 第 2705 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2706 / 第 2706 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2707 / 第 2707 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2708 / 第 2708 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2709 / 第 2709 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2710 / 第 2710 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2711-2720 / 第 2711-2720 行
```cpp
2711 |                                               long maxevents, long timeout,
2712 |                                               long sigmask, long sigsetsize);
2713 | void __sanitizer_syscall_post_impl_epoll_pwait(long res, long epfd, long events,
2714 |                                                long maxevents, long timeout,
2715 |                                                long sigmask, long sigsetsize);
2716 | void __sanitizer_syscall_pre_impl_epoll_pwait2(long epfd, long events,
2717 |                                                long maxevents, long timeout,
2718 |                                                long sigmask, long sigsetsize);
2719 | void __sanitizer_syscall_post_impl_epoll_pwait2(long res, long epfd,
2720 |                                                 long events, long maxevents,
```
- **Line 2711 / 第 2711 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2712 / 第 2712 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2713 / 第 2713 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2714 / 第 2714 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2715 / 第 2715 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2716 / 第 2716 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2717 / 第 2717 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2718 / 第 2718 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2719 / 第 2719 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2720 / 第 2720 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2721-2730 / 第 2721-2730 行
```cpp
2721 |                                                 long timeout, long sigmask,
2722 |                                                 long sigsetsize);
2723 | void __sanitizer_syscall_pre_impl_gethostname(long name, long len);
2724 | void __sanitizer_syscall_post_impl_gethostname(long res, long name, long len);
2725 | void __sanitizer_syscall_pre_impl_sethostname(long name, long len);
2726 | void __sanitizer_syscall_post_impl_sethostname(long res, long name, long len);
2727 | void __sanitizer_syscall_pre_impl_setdomainname(long name, long len);
2728 | void __sanitizer_syscall_post_impl_setdomainname(long res, long name, long len);
2729 | void __sanitizer_syscall_pre_impl_newuname(long name);
2730 | void __sanitizer_syscall_post_impl_newuname(long res, long name);
```
- **Line 2721 / 第 2721 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2722 / 第 2722 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2723 / 第 2723 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_gethostname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_gethostname`。
- **Line 2724 / 第 2724 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_gethostname`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_gethostname`。
- **Line 2725 / 第 2725 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sethostname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sethostname`。
- **Line 2726 / 第 2726 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sethostname`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sethostname`。
- **Line 2727 / 第 2727 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setdomainname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setdomainname`。
- **Line 2728 / 第 2728 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setdomainname`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setdomainname`。
- **Line 2729 / 第 2729 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_newuname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_newuname`。
- **Line 2730 / 第 2730 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_newuname`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_newuname`。

### Lines 2731-2740 / 第 2731-2740 行
```cpp
2731 | void __sanitizer_syscall_pre_impl_uname(long arg0);
2732 | void __sanitizer_syscall_post_impl_uname(long res, long arg0);
2733 | void __sanitizer_syscall_pre_impl_olduname(long arg0);
2734 | void __sanitizer_syscall_post_impl_olduname(long res, long arg0);
2735 | void __sanitizer_syscall_pre_impl_getrlimit(long resource, long rlim);
2736 | void __sanitizer_syscall_post_impl_getrlimit(long res, long resource,
2737 |                                              long rlim);
2738 | void __sanitizer_syscall_pre_impl_old_getrlimit(long resource, long rlim);
2739 | void __sanitizer_syscall_post_impl_old_getrlimit(long res, long resource,
2740 |                                                  long rlim);
```
- **Line 2731 / 第 2731 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_uname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_uname`。
- **Line 2732 / 第 2732 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_uname`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_uname`。
- **Line 2733 / 第 2733 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_olduname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_olduname`。
- **Line 2734 / 第 2734 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_olduname`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_olduname`。
- **Line 2735 / 第 2735 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getrlimit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getrlimit`。
- **Line 2736 / 第 2736 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2737 / 第 2737 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2738 / 第 2738 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_old_getrlimit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_old_getrlimit`。
- **Line 2739 / 第 2739 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2740 / 第 2740 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2741-2750 / 第 2741-2750 行
```cpp
2741 | void __sanitizer_syscall_pre_impl_setrlimit(long resource, long rlim);
2742 | void __sanitizer_syscall_post_impl_setrlimit(long res, long resource,
2743 |                                              long rlim);
2744 | void __sanitizer_syscall_pre_impl_prlimit64(long pid, long resource,
2745 |                                             long new_rlim, long old_rlim);
2746 | void __sanitizer_syscall_post_impl_prlimit64(long res, long pid, long resource,
2747 |                                              long new_rlim, long old_rlim);
2748 | void __sanitizer_syscall_pre_impl_getrusage(long who, long ru);
2749 | void __sanitizer_syscall_post_impl_getrusage(long res, long who, long ru);
2750 | void __sanitizer_syscall_pre_impl_umask(long mask);
```
- **Line 2741 / 第 2741 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setrlimit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setrlimit`。
- **Line 2742 / 第 2742 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2743 / 第 2743 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2744 / 第 2744 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2745 / 第 2745 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2746 / 第 2746 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2747 / 第 2747 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2748 / 第 2748 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getrusage`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getrusage`。
- **Line 2749 / 第 2749 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getrusage`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getrusage`。
- **Line 2750 / 第 2750 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_umask`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_umask`。

### Lines 2751-2760 / 第 2751-2760 行
```cpp
2751 | void __sanitizer_syscall_post_impl_umask(long res, long mask);
2752 | void __sanitizer_syscall_pre_impl_msgget(long key, long msgflg);
2753 | void __sanitizer_syscall_post_impl_msgget(long res, long key, long msgflg);
2754 | void __sanitizer_syscall_pre_impl_msgsnd(long msqid, long msgp, long msgsz,
2755 |                                          long msgflg);
2756 | void __sanitizer_syscall_post_impl_msgsnd(long res, long msqid, long msgp,
2757 |                                           long msgsz, long msgflg);
2758 | void __sanitizer_syscall_pre_impl_msgrcv(long msqid, long msgp, long msgsz,
2759 |                                          long msgtyp, long msgflg);
2760 | void __sanitizer_syscall_post_impl_msgrcv(long res, long msqid, long msgp,
```
- **Line 2751 / 第 2751 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_umask`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_umask`。
- **Line 2752 / 第 2752 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_msgget`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_msgget`。
- **Line 2753 / 第 2753 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_msgget`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_msgget`。
- **Line 2754 / 第 2754 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2755 / 第 2755 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2756 / 第 2756 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2757 / 第 2757 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2758 / 第 2758 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2759 / 第 2759 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2760 / 第 2760 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2761-2770 / 第 2761-2770 行
```cpp
2761 |                                           long msgsz, long msgtyp, long msgflg);
2762 | void __sanitizer_syscall_pre_impl_msgctl(long msqid, long cmd, long buf);
2763 | void __sanitizer_syscall_post_impl_msgctl(long res, long msqid, long cmd,
2764 |                                           long buf);
2765 | void __sanitizer_syscall_pre_impl_semget(long key, long nsems, long semflg);
2766 | void __sanitizer_syscall_post_impl_semget(long res, long key, long nsems,
2767 |                                           long semflg);
2768 | void __sanitizer_syscall_pre_impl_semop(long semid, long sops, long nsops);
2769 | void __sanitizer_syscall_post_impl_semop(long res, long semid, long sops,
2770 |                                          long nsops);
```
- **Line 2761 / 第 2761 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2762 / 第 2762 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_msgctl`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_msgctl`。
- **Line 2763 / 第 2763 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2764 / 第 2764 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2765 / 第 2765 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_semget`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_semget`。
- **Line 2766 / 第 2766 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2767 / 第 2767 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2768 / 第 2768 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_semop`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_semop`。
- **Line 2769 / 第 2769 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2770 / 第 2770 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2771-2780 / 第 2771-2780 行
```cpp
2771 | void __sanitizer_syscall_pre_impl_semctl(long semid, long semnum, long cmd,
2772 |                                          long arg);
2773 | void __sanitizer_syscall_post_impl_semctl(long res, long semid, long semnum,
2774 |                                           long cmd, long arg);
2775 | void __sanitizer_syscall_pre_impl_semtimedop(long semid, long sops, long nsops,
2776 |                                              long timeout);
2777 | void __sanitizer_syscall_post_impl_semtimedop(long res, long semid, long sops,
2778 |                                               long nsops, long timeout);
2779 | void __sanitizer_syscall_pre_impl_shmat(long shmid, long shmaddr, long shmflg);
2780 | void __sanitizer_syscall_post_impl_shmat(long res, long shmid, long shmaddr,
```
- **Line 2771 / 第 2771 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2772 / 第 2772 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2773 / 第 2773 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2774 / 第 2774 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2775 / 第 2775 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2776 / 第 2776 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2777 / 第 2777 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2778 / 第 2778 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2779 / 第 2779 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_shmat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_shmat`。
- **Line 2780 / 第 2780 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2781-2790 / 第 2781-2790 行
```cpp
2781 |                                          long shmflg);
2782 | void __sanitizer_syscall_pre_impl_shmget(long key, long size, long flag);
2783 | void __sanitizer_syscall_post_impl_shmget(long res, long key, long size,
2784 |                                           long flag);
2785 | void __sanitizer_syscall_pre_impl_shmdt(long shmaddr);
2786 | void __sanitizer_syscall_post_impl_shmdt(long res, long shmaddr);
2787 | void __sanitizer_syscall_pre_impl_shmctl(long shmid, long cmd, long buf);
2788 | void __sanitizer_syscall_post_impl_shmctl(long res, long shmid, long cmd,
2789 |                                           long buf);
2790 | void __sanitizer_syscall_pre_impl_ipc(long call, long first, long second,
```
- **Line 2781 / 第 2781 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2782 / 第 2782 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_shmget`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_shmget`。
- **Line 2783 / 第 2783 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2784 / 第 2784 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2785 / 第 2785 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_shmdt`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_shmdt`。
- **Line 2786 / 第 2786 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_shmdt`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_shmdt`。
- **Line 2787 / 第 2787 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_shmctl`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_shmctl`。
- **Line 2788 / 第 2788 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2789 / 第 2789 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2790 / 第 2790 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2791-2800 / 第 2791-2800 行
```cpp
2791 |                                       long third, long ptr, long fifth);
2792 | void __sanitizer_syscall_post_impl_ipc(long res, long call, long first,
2793 |                                        long second, long third, long ptr,
2794 |                                        long fifth);
2795 | void __sanitizer_syscall_pre_impl_mq_open(long name, long oflag, long mode,
2796 |                                           long attr);
2797 | void __sanitizer_syscall_post_impl_mq_open(long res, long name, long oflag,
2798 |                                            long mode, long attr);
2799 | void __sanitizer_syscall_pre_impl_mq_unlink(long name);
2800 | void __sanitizer_syscall_post_impl_mq_unlink(long res, long name);
```
- **Line 2791 / 第 2791 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2792 / 第 2792 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2793 / 第 2793 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2794 / 第 2794 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2795 / 第 2795 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2796 / 第 2796 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2797 / 第 2797 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2798 / 第 2798 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2799 / 第 2799 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mq_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mq_unlink`。
- **Line 2800 / 第 2800 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_mq_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_mq_unlink`。

### Lines 2801-2810 / 第 2801-2810 行
```cpp
2801 | void __sanitizer_syscall_pre_impl_mq_timedsend(long mqdes, long msg_ptr,
2802 |                                                long msg_len, long msg_prio,
2803 |                                                long abs_timeout);
2804 | void __sanitizer_syscall_post_impl_mq_timedsend(long res, long mqdes,
2805 |                                                 long msg_ptr, long msg_len,
2806 |                                                 long msg_prio,
2807 |                                                 long abs_timeout);
2808 | void __sanitizer_syscall_pre_impl_mq_timedreceive(long mqdes, long msg_ptr,
2809 |                                                   long msg_len, long msg_prio,
2810 |                                                   long abs_timeout);
```
- **Line 2801 / 第 2801 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2802 / 第 2802 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2803 / 第 2803 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2804 / 第 2804 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2805 / 第 2805 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2806 / 第 2806 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2807 / 第 2807 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2808 / 第 2808 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2809 / 第 2809 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2810 / 第 2810 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2811-2820 / 第 2811-2820 行
```cpp
2811 | void __sanitizer_syscall_post_impl_mq_timedreceive(long res, long mqdes,
2812 |                                                    long msg_ptr, long msg_len,
2813 |                                                    long msg_prio,
2814 |                                                    long abs_timeout);
2815 | void __sanitizer_syscall_pre_impl_mq_notify(long mqdes, long notification);
2816 | void __sanitizer_syscall_post_impl_mq_notify(long res, long mqdes,
2817 |                                              long notification);
2818 | void __sanitizer_syscall_pre_impl_mq_getsetattr(long mqdes, long mqstat,
2819 |                                                 long omqstat);
2820 | void __sanitizer_syscall_post_impl_mq_getsetattr(long res, long mqdes,
```
- **Line 2811 / 第 2811 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2812 / 第 2812 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2813 / 第 2813 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2814 / 第 2814 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2815 / 第 2815 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mq_notify`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mq_notify`。
- **Line 2816 / 第 2816 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2817 / 第 2817 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2818 / 第 2818 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2819 / 第 2819 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2820 / 第 2820 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2821-2830 / 第 2821-2830 行
```cpp
2821 |                                                  long mqstat, long omqstat);
2822 | void __sanitizer_syscall_pre_impl_pciconfig_iobase(long which, long bus,
2823 |                                                    long devfn);
2824 | void __sanitizer_syscall_post_impl_pciconfig_iobase(long res, long which,
2825 |                                                     long bus, long devfn);
2826 | void __sanitizer_syscall_pre_impl_pciconfig_read(long bus, long dfn, long off,
2827 |                                                  long len, long buf);
2828 | void __sanitizer_syscall_post_impl_pciconfig_read(long res, long bus, long dfn,
2829 |                                                   long off, long len, long buf);
2830 | void __sanitizer_syscall_pre_impl_pciconfig_write(long bus, long dfn, long off,
```
- **Line 2821 / 第 2821 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2822 / 第 2822 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2823 / 第 2823 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2824 / 第 2824 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2825 / 第 2825 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2826 / 第 2826 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2827 / 第 2827 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2828 / 第 2828 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2829 / 第 2829 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2830 / 第 2830 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2831-2840 / 第 2831-2840 行
```cpp
2831 |                                                   long len, long buf);
2832 | void __sanitizer_syscall_post_impl_pciconfig_write(long res, long bus, long dfn,
2833 |                                                    long off, long len,
2834 |                                                    long buf);
2835 | void __sanitizer_syscall_pre_impl_swapon(long specialfile, long swap_flags);
2836 | void __sanitizer_syscall_post_impl_swapon(long res, long specialfile,
2837 |                                           long swap_flags);
2838 | void __sanitizer_syscall_pre_impl_swapoff(long specialfile);
2839 | void __sanitizer_syscall_post_impl_swapoff(long res, long specialfile);
2840 | void __sanitizer_syscall_pre_impl_sysctl(long args);
```
- **Line 2831 / 第 2831 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2832 / 第 2832 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2833 / 第 2833 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2834 / 第 2834 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2835 / 第 2835 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_swapon`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_swapon`。
- **Line 2836 / 第 2836 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2837 / 第 2837 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2838 / 第 2838 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_swapoff`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_swapoff`。
- **Line 2839 / 第 2839 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_swapoff`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_swapoff`。
- **Line 2840 / 第 2840 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sysctl`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sysctl`。

### Lines 2841-2850 / 第 2841-2850 行
```cpp
2841 | void __sanitizer_syscall_post_impl_sysctl(long res, long args);
2842 | void __sanitizer_syscall_pre_impl_sysinfo(long info);
2843 | void __sanitizer_syscall_post_impl_sysinfo(long res, long info);
2844 | void __sanitizer_syscall_pre_impl_sysfs(long option, long arg1, long arg2);
2845 | void __sanitizer_syscall_post_impl_sysfs(long res, long option, long arg1,
2846 |                                          long arg2);
2847 | void __sanitizer_syscall_pre_impl_syslog(long type, long buf, long len);
2848 | void __sanitizer_syscall_post_impl_syslog(long res, long type, long buf,
2849 |                                           long len);
2850 | void __sanitizer_syscall_pre_impl_uselib(long library);
```
- **Line 2841 / 第 2841 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sysctl`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sysctl`。
- **Line 2842 / 第 2842 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sysinfo`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sysinfo`。
- **Line 2843 / 第 2843 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sysinfo`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sysinfo`。
- **Line 2844 / 第 2844 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sysfs`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sysfs`。
- **Line 2845 / 第 2845 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2846 / 第 2846 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2847 / 第 2847 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_syslog`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_syslog`。
- **Line 2848 / 第 2848 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2849 / 第 2849 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2850 / 第 2850 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_uselib`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_uselib`。

### Lines 2851-2860 / 第 2851-2860 行
```cpp
2851 | void __sanitizer_syscall_post_impl_uselib(long res, long library);
2852 | void __sanitizer_syscall_pre_impl_ni_syscall();
2853 | void __sanitizer_syscall_post_impl_ni_syscall(long res);
2854 | void __sanitizer_syscall_pre_impl_ptrace(long request, long pid, long addr,
2855 |                                          long data);
2856 | void __sanitizer_syscall_post_impl_ptrace(long res, long request, long pid,
2857 |                                           long addr, long data);
2858 | void __sanitizer_syscall_pre_impl_add_key(long _type, long _description,
2859 |                                           long _payload, long plen,
2860 |                                           long destringid);
```
- **Line 2851 / 第 2851 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_uselib`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_uselib`。
- **Line 2852 / 第 2852 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ni_syscall`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ni_syscall`。
- **Line 2853 / 第 2853 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_ni_syscall`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_ni_syscall`。
- **Line 2854 / 第 2854 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2855 / 第 2855 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2856 / 第 2856 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2857 / 第 2857 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2858 / 第 2858 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2859 / 第 2859 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2860 / 第 2860 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2861-2870 / 第 2861-2870 行
```cpp
2861 | void __sanitizer_syscall_post_impl_add_key(long res, long _type,
2862 |                                            long _description, long _payload,
2863 |                                            long plen, long destringid);
2864 | void __sanitizer_syscall_pre_impl_request_key(long _type, long _description,
2865 |                                               long _callout_info,
2866 |                                               long destringid);
2867 | void __sanitizer_syscall_post_impl_request_key(long res, long _type,
2868 |                                                long _description,
2869 |                                                long _callout_info,
2870 |                                                long destringid);
```
- **Line 2861 / 第 2861 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2862 / 第 2862 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2863 / 第 2863 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2864 / 第 2864 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2865 / 第 2865 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2866 / 第 2866 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2867 / 第 2867 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2868 / 第 2868 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2869 / 第 2869 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2870 / 第 2870 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2871-2880 / 第 2871-2880 行
```cpp
2871 | void __sanitizer_syscall_pre_impl_keyctl(long cmd, long arg2, long arg3,
2872 |                                          long arg4, long arg5);
2873 | void __sanitizer_syscall_post_impl_keyctl(long res, long cmd, long arg2,
2874 |                                           long arg3, long arg4, long arg5);
2875 | void __sanitizer_syscall_pre_impl_ioprio_set(long which, long who, long ioprio);
2876 | void __sanitizer_syscall_post_impl_ioprio_set(long res, long which, long who,
2877 |                                               long ioprio);
2878 | void __sanitizer_syscall_pre_impl_ioprio_get(long which, long who);
2879 | void __sanitizer_syscall_post_impl_ioprio_get(long res, long which, long who);
2880 | void __sanitizer_syscall_pre_impl_set_mempolicy(long mode, long nmask,
```
- **Line 2871 / 第 2871 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2872 / 第 2872 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2873 / 第 2873 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2874 / 第 2874 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2875 / 第 2875 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ioprio_set`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ioprio_set`。
- **Line 2876 / 第 2876 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2877 / 第 2877 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2878 / 第 2878 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ioprio_get`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ioprio_get`。
- **Line 2879 / 第 2879 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_ioprio_get`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_ioprio_get`。
- **Line 2880 / 第 2880 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2881-2890 / 第 2881-2890 行
```cpp
2881 |                                                 long maxnode);
2882 | void __sanitizer_syscall_post_impl_set_mempolicy(long res, long mode,
2883 |                                                  long nmask, long maxnode);
2884 | void __sanitizer_syscall_pre_impl_migrate_pages(long pid, long maxnode,
2885 |                                                 long from, long to);
2886 | void __sanitizer_syscall_post_impl_migrate_pages(long res, long pid,
2887 |                                                  long maxnode, long from,
2888 |                                                  long to);
2889 | void __sanitizer_syscall_pre_impl_move_pages(long pid, long nr_pages,
2890 |                                              long pages, long nodes,
```
- **Line 2881 / 第 2881 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2882 / 第 2882 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2883 / 第 2883 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2884 / 第 2884 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2885 / 第 2885 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2886 / 第 2886 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2887 / 第 2887 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2888 / 第 2888 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2889 / 第 2889 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2890 / 第 2890 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2891-2900 / 第 2891-2900 行
```cpp
2891 |                                              long status, long flags);
2892 | void __sanitizer_syscall_post_impl_move_pages(long res, long pid, long nr_pages,
2893 |                                               long pages, long nodes,
2894 |                                               long status, long flags);
2895 | void __sanitizer_syscall_pre_impl_mbind(long start, long len, long mode,
2896 |                                         long nmask, long maxnode, long flags);
2897 | void __sanitizer_syscall_post_impl_mbind(long res, long start, long len,
2898 |                                          long mode, long nmask, long maxnode,
2899 |                                          long flags);
2900 | void __sanitizer_syscall_pre_impl_get_mempolicy(long policy, long nmask,
```
- **Line 2891 / 第 2891 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2892 / 第 2892 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2893 / 第 2893 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2894 / 第 2894 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2895 / 第 2895 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2896 / 第 2896 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2897 / 第 2897 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2898 / 第 2898 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2899 / 第 2899 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2900 / 第 2900 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2901-2910 / 第 2901-2910 行
```cpp
2901 |                                                 long maxnode, long addr,
2902 |                                                 long flags);
2903 | void __sanitizer_syscall_post_impl_get_mempolicy(long res, long policy,
2904 |                                                  long nmask, long maxnode,
2905 |                                                  long addr, long flags);
2906 | void __sanitizer_syscall_pre_impl_inotify_init();
2907 | void __sanitizer_syscall_post_impl_inotify_init(long res);
2908 | void __sanitizer_syscall_pre_impl_inotify_init1(long flags);
2909 | void __sanitizer_syscall_post_impl_inotify_init1(long res, long flags);
2910 | void __sanitizer_syscall_pre_impl_inotify_add_watch(long fd, long path,
```
- **Line 2901 / 第 2901 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2902 / 第 2902 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2903 / 第 2903 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2904 / 第 2904 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2905 / 第 2905 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2906 / 第 2906 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_inotify_init`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_inotify_init`。
- **Line 2907 / 第 2907 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_inotify_init`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_inotify_init`。
- **Line 2908 / 第 2908 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_inotify_init1`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_inotify_init1`。
- **Line 2909 / 第 2909 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_inotify_init1`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_inotify_init1`。
- **Line 2910 / 第 2910 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2911-2920 / 第 2911-2920 行
```cpp
2911 |                                                     long mask);
2912 | void __sanitizer_syscall_post_impl_inotify_add_watch(long res, long fd,
2913 |                                                      long path, long mask);
2914 | void __sanitizer_syscall_pre_impl_inotify_rm_watch(long fd, long wd);
2915 | void __sanitizer_syscall_post_impl_inotify_rm_watch(long res, long fd, long wd);
2916 | void __sanitizer_syscall_pre_impl_spu_run(long fd, long unpc, long ustatus);
2917 | void __sanitizer_syscall_post_impl_spu_run(long res, long fd, long unpc,
2918 |                                            long ustatus);
2919 | void __sanitizer_syscall_pre_impl_spu_create(long name, long flags, long mode,
2920 |                                              long fd);
```
- **Line 2911 / 第 2911 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2912 / 第 2912 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2913 / 第 2913 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2914 / 第 2914 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_inotify_rm_watch`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_inotify_rm_watch`。
- **Line 2915 / 第 2915 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_inotify_rm_watch`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_inotify_rm_watch`。
- **Line 2916 / 第 2916 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_spu_run`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_spu_run`。
- **Line 2917 / 第 2917 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2918 / 第 2918 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2919 / 第 2919 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2920 / 第 2920 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2921-2930 / 第 2921-2930 行
```cpp
2921 | void __sanitizer_syscall_post_impl_spu_create(long res, long name, long flags,
2922 |                                               long mode, long fd);
2923 | void __sanitizer_syscall_pre_impl_mknodat(long dfd, long filename, long mode,
2924 |                                           long dev);
2925 | void __sanitizer_syscall_post_impl_mknodat(long res, long dfd, long filename,
2926 |                                            long mode, long dev);
2927 | void __sanitizer_syscall_pre_impl_mkdirat(long dfd, long pathname, long mode);
2928 | void __sanitizer_syscall_post_impl_mkdirat(long res, long dfd, long pathname,
2929 |                                            long mode);
2930 | void __sanitizer_syscall_pre_impl_unlinkat(long dfd, long pathname, long flag);
```
- **Line 2921 / 第 2921 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2922 / 第 2922 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2923 / 第 2923 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2924 / 第 2924 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2925 / 第 2925 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2926 / 第 2926 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2927 / 第 2927 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mkdirat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mkdirat`。
- **Line 2928 / 第 2928 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2929 / 第 2929 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2930 / 第 2930 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_unlinkat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_unlinkat`。

### Lines 2931-2940 / 第 2931-2940 行
```cpp
2931 | void __sanitizer_syscall_post_impl_unlinkat(long res, long dfd, long pathname,
2932 |                                             long flag);
2933 | void __sanitizer_syscall_pre_impl_symlinkat(long oldname, long newdfd,
2934 |                                             long newname);
2935 | void __sanitizer_syscall_post_impl_symlinkat(long res, long oldname,
2936 |                                              long newdfd, long newname);
2937 | void __sanitizer_syscall_pre_impl_linkat(long olddfd, long oldname, long newdfd,
2938 |                                          long newname, long flags);
2939 | void __sanitizer_syscall_post_impl_linkat(long res, long olddfd, long oldname,
2940 |                                           long newdfd, long newname,
```
- **Line 2931 / 第 2931 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2932 / 第 2932 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2933 / 第 2933 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2934 / 第 2934 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2935 / 第 2935 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2936 / 第 2936 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2937 / 第 2937 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2938 / 第 2938 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2939 / 第 2939 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2940 / 第 2940 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2941-2950 / 第 2941-2950 行
```cpp
2941 |                                           long flags);
2942 | void __sanitizer_syscall_pre_impl_renameat(long olddfd, long oldname,
2943 |                                            long newdfd, long newname);
2944 | void __sanitizer_syscall_post_impl_renameat(long res, long olddfd, long oldname,
2945 |                                             long newdfd, long newname);
2946 | void __sanitizer_syscall_pre_impl_futimesat(long dfd, long filename,
2947 |                                             long utimes);
2948 | void __sanitizer_syscall_post_impl_futimesat(long res, long dfd, long filename,
2949 |                                              long utimes);
2950 | void __sanitizer_syscall_pre_impl_faccessat(long dfd, long filename, long mode);
```
- **Line 2941 / 第 2941 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2942 / 第 2942 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2943 / 第 2943 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2944 / 第 2944 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2945 / 第 2945 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2946 / 第 2946 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2947 / 第 2947 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2948 / 第 2948 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2949 / 第 2949 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2950 / 第 2950 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_faccessat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_faccessat`。

### Lines 2951-2960 / 第 2951-2960 行
```cpp
2951 | void __sanitizer_syscall_post_impl_faccessat(long res, long dfd, long filename,
2952 |                                              long mode);
2953 | void __sanitizer_syscall_pre_impl_fchmodat(long dfd, long filename, long mode);
2954 | void __sanitizer_syscall_post_impl_fchmodat(long res, long dfd, long filename,
2955 |                                             long mode);
2956 | void __sanitizer_syscall_pre_impl_fchownat(long dfd, long filename, long user,
2957 |                                            long group, long flag);
2958 | void __sanitizer_syscall_post_impl_fchownat(long res, long dfd, long filename,
2959 |                                             long user, long group, long flag);
2960 | void __sanitizer_syscall_pre_impl_openat(long dfd, long filename, long flags,
```
- **Line 2951 / 第 2951 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2952 / 第 2952 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2953 / 第 2953 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchmodat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchmodat`。
- **Line 2954 / 第 2954 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2955 / 第 2955 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2956 / 第 2956 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2957 / 第 2957 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2958 / 第 2958 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2959 / 第 2959 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2960 / 第 2960 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2961-2970 / 第 2961-2970 行
```cpp
2961 |                                          long mode);
2962 | void __sanitizer_syscall_post_impl_openat(long res, long dfd, long filename,
2963 |                                           long flags, long mode);
2964 | void __sanitizer_syscall_pre_impl_newfstatat(long dfd, long filename,
2965 |                                              long statbuf, long flag);
2966 | void __sanitizer_syscall_post_impl_newfstatat(long res, long dfd, long filename,
2967 |                                               long statbuf, long flag);
2968 | void __sanitizer_syscall_pre_impl_fstatat64(long dfd, long filename,
2969 |                                             long statbuf, long flag);
2970 | void __sanitizer_syscall_post_impl_fstatat64(long res, long dfd, long filename,
```
- **Line 2961 / 第 2961 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2962 / 第 2962 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2963 / 第 2963 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2964 / 第 2964 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2965 / 第 2965 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2966 / 第 2966 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2967 / 第 2967 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2968 / 第 2968 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2969 / 第 2969 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2970 / 第 2970 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2971-2980 / 第 2971-2980 行
```cpp
2971 |                                              long statbuf, long flag);
2972 | void __sanitizer_syscall_pre_impl_readlinkat(long dfd, long path, long buf,
2973 |                                              long bufsiz);
2974 | void __sanitizer_syscall_post_impl_readlinkat(long res, long dfd, long path,
2975 |                                               long buf, long bufsiz);
2976 | void __sanitizer_syscall_pre_impl_utimensat(long dfd, long filename,
2977 |                                             long utimes, long flags);
2978 | void __sanitizer_syscall_post_impl_utimensat(long res, long dfd, long filename,
2979 |                                              long utimes, long flags);
2980 | void __sanitizer_syscall_pre_impl_unshare(long unshare_flags);
```
- **Line 2971 / 第 2971 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2972 / 第 2972 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2973 / 第 2973 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2974 / 第 2974 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2975 / 第 2975 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2976 / 第 2976 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2977 / 第 2977 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2978 / 第 2978 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2979 / 第 2979 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2980 / 第 2980 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_unshare`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_unshare`。

### Lines 2981-2990 / 第 2981-2990 行
```cpp
2981 | void __sanitizer_syscall_post_impl_unshare(long res, long unshare_flags);
2982 | void __sanitizer_syscall_pre_impl_splice(long fd_in, long off_in, long fd_out,
2983 |                                          long off_out, long len, long flags);
2984 | void __sanitizer_syscall_post_impl_splice(long res, long fd_in, long off_in,
2985 |                                           long fd_out, long off_out, long len,
2986 |                                           long flags);
2987 | void __sanitizer_syscall_pre_impl_vmsplice(long fd, long iov, long nr_segs,
2988 |                                            long flags);
2989 | void __sanitizer_syscall_post_impl_vmsplice(long res, long fd, long iov,
2990 |                                             long nr_segs, long flags);
```
- **Line 2981 / 第 2981 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_unshare`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_unshare`。
- **Line 2982 / 第 2982 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2983 / 第 2983 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2984 / 第 2984 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2985 / 第 2985 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2986 / 第 2986 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2987 / 第 2987 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2988 / 第 2988 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2989 / 第 2989 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2990 / 第 2990 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2991-3000 / 第 2991-3000 行
```cpp
2991 | void __sanitizer_syscall_pre_impl_tee(long fdin, long fdout, long len,
2992 |                                       long flags);
2993 | void __sanitizer_syscall_post_impl_tee(long res, long fdin, long fdout,
2994 |                                        long len, long flags);
2995 | void __sanitizer_syscall_pre_impl_get_robust_list(long pid, long head_ptr,
2996 |                                                   long len_ptr);
2997 | void __sanitizer_syscall_post_impl_get_robust_list(long res, long pid,
2998 |                                                    long head_ptr, long len_ptr);
2999 | void __sanitizer_syscall_pre_impl_set_robust_list(long head, long len);
3000 | void __sanitizer_syscall_post_impl_set_robust_list(long res, long head,
```
- **Line 2991 / 第 2991 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2992 / 第 2992 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2993 / 第 2993 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2994 / 第 2994 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2995 / 第 2995 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2996 / 第 2996 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2997 / 第 2997 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2998 / 第 2998 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2999 / 第 2999 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_set_robust_list`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_set_robust_list`。
- **Line 3000 / 第 3000 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3001-3010 / 第 3001-3010 行
```cpp
3001 |                                                    long len);
3002 | void __sanitizer_syscall_pre_impl_getcpu(long cpu, long node, long cache);
3003 | void __sanitizer_syscall_post_impl_getcpu(long res, long cpu, long node,
3004 |                                           long cache);
3005 | void __sanitizer_syscall_pre_impl_signalfd(long ufd, long user_mask,
3006 |                                            long sizemask);
3007 | void __sanitizer_syscall_post_impl_signalfd(long res, long ufd, long user_mask,
3008 |                                             long sizemask);
3009 | void __sanitizer_syscall_pre_impl_signalfd4(long ufd, long user_mask,
3010 |                                             long sizemask, long flags);
```
- **Line 3001 / 第 3001 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3002 / 第 3002 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getcpu`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getcpu`。
- **Line 3003 / 第 3003 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3004 / 第 3004 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3005 / 第 3005 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3006 / 第 3006 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3007 / 第 3007 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3008 / 第 3008 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3009 / 第 3009 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3010 / 第 3010 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3011-3020 / 第 3011-3020 行
```cpp
3011 | void __sanitizer_syscall_post_impl_signalfd4(long res, long ufd, long user_mask,
3012 |                                              long sizemask, long flags);
3013 | void __sanitizer_syscall_pre_impl_timerfd_create(long clockid, long flags);
3014 | void __sanitizer_syscall_post_impl_timerfd_create(long res, long clockid,
3015 |                                                   long flags);
3016 | void __sanitizer_syscall_pre_impl_timerfd_settime(long ufd, long flags,
3017 |                                                   long utmr, long otmr);
3018 | void __sanitizer_syscall_post_impl_timerfd_settime(long res, long ufd,
3019 |                                                    long flags, long utmr,
3020 |                                                    long otmr);
```
- **Line 3011 / 第 3011 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3012 / 第 3012 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3013 / 第 3013 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_timerfd_create`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_timerfd_create`。
- **Line 3014 / 第 3014 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3015 / 第 3015 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3016 / 第 3016 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3017 / 第 3017 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3018 / 第 3018 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3019 / 第 3019 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3020 / 第 3020 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3021-3030 / 第 3021-3030 行
```cpp
3021 | void __sanitizer_syscall_pre_impl_timerfd_gettime(long ufd, long otmr);
3022 | void __sanitizer_syscall_post_impl_timerfd_gettime(long res, long ufd,
3023 |                                                    long otmr);
3024 | void __sanitizer_syscall_pre_impl_eventfd(long count);
3025 | void __sanitizer_syscall_post_impl_eventfd(long res, long count);
3026 | void __sanitizer_syscall_pre_impl_eventfd2(long count, long flags);
3027 | void __sanitizer_syscall_post_impl_eventfd2(long res, long count, long flags);
3028 | void __sanitizer_syscall_pre_impl_old_readdir(long arg0, long arg1, long arg2);
3029 | void __sanitizer_syscall_post_impl_old_readdir(long res, long arg0, long arg1,
3030 |                                                long arg2);
```
- **Line 3021 / 第 3021 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_timerfd_gettime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_timerfd_gettime`。
- **Line 3022 / 第 3022 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3023 / 第 3023 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3024 / 第 3024 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_eventfd`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_eventfd`。
- **Line 3025 / 第 3025 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_eventfd`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_eventfd`。
- **Line 3026 / 第 3026 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_eventfd2`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_eventfd2`。
- **Line 3027 / 第 3027 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_eventfd2`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_eventfd2`。
- **Line 3028 / 第 3028 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_old_readdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_old_readdir`。
- **Line 3029 / 第 3029 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3030 / 第 3030 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3031-3040 / 第 3031-3040 行
```cpp
3031 | void __sanitizer_syscall_pre_impl_pselect6(long arg0, long arg1, long arg2,
3032 |                                            long arg3, long arg4, long arg5);
3033 | void __sanitizer_syscall_post_impl_pselect6(long res, long arg0, long arg1,
3034 |                                             long arg2, long arg3, long arg4,
3035 |                                             long arg5);
3036 | void __sanitizer_syscall_pre_impl_ppoll(long arg0, long arg1, long arg2,
3037 |                                         long arg3, long arg4);
3038 | void __sanitizer_syscall_post_impl_ppoll(long res, long arg0, long arg1,
3039 |                                          long arg2, long arg3, long arg4);
3040 | void __sanitizer_syscall_pre_impl_fanotify_init(long flags, long event_f_flags);
```
- **Line 3031 / 第 3031 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3032 / 第 3032 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3033 / 第 3033 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3034 / 第 3034 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3035 / 第 3035 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3036 / 第 3036 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3037 / 第 3037 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3038 / 第 3038 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3039 / 第 3039 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3040 / 第 3040 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fanotify_init`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fanotify_init`。

### Lines 3041-3050 / 第 3041-3050 行
```cpp
3041 | void __sanitizer_syscall_post_impl_fanotify_init(long res, long flags,
3042 |                                                  long event_f_flags);
3043 | void __sanitizer_syscall_pre_impl_fanotify_mark(long fanotify_fd, long flags,
3044 |                                                 long mask, long fd,
3045 |                                                 long pathname);
3046 | void __sanitizer_syscall_post_impl_fanotify_mark(long res, long fanotify_fd,
3047 |                                                  long flags, long mask, long fd,
3048 |                                                  long pathname);
3049 | void __sanitizer_syscall_pre_impl_syncfs(long fd);
3050 | void __sanitizer_syscall_post_impl_syncfs(long res, long fd);
```
- **Line 3041 / 第 3041 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3042 / 第 3042 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3043 / 第 3043 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3044 / 第 3044 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3045 / 第 3045 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3046 / 第 3046 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3047 / 第 3047 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3048 / 第 3048 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3049 / 第 3049 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_syncfs`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_syncfs`。
- **Line 3050 / 第 3050 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_syncfs`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_syncfs`。

### Lines 3051-3060 / 第 3051-3060 行
```cpp
3051 | void __sanitizer_syscall_pre_impl_perf_event_open(long attr_uptr, long pid,
3052 |                                                   long cpu, long group_fd,
3053 |                                                   long flags);
3054 | void __sanitizer_syscall_post_impl_perf_event_open(long res, long attr_uptr,
3055 |                                                    long pid, long cpu,
3056 |                                                    long group_fd, long flags);
3057 | void __sanitizer_syscall_pre_impl_mmap_pgoff(long addr, long len, long prot,
3058 |                                              long flags, long fd, long pgoff);
3059 | void __sanitizer_syscall_post_impl_mmap_pgoff(long res, long addr, long len,
3060 |                                               long prot, long flags, long fd,
```
- **Line 3051 / 第 3051 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3052 / 第 3052 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3053 / 第 3053 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3054 / 第 3054 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3055 / 第 3055 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3056 / 第 3056 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3057 / 第 3057 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3058 / 第 3058 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3059 / 第 3059 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3060 / 第 3060 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3061-3070 / 第 3061-3070 行
```cpp
3061 |                                               long pgoff);
3062 | void __sanitizer_syscall_pre_impl_old_mmap(long arg);
3063 | void __sanitizer_syscall_post_impl_old_mmap(long res, long arg);
3064 | void __sanitizer_syscall_pre_impl_name_to_handle_at(long dfd, long name,
3065 |                                                     long handle, long mnt_id,
3066 |                                                     long flag);
3067 | void __sanitizer_syscall_post_impl_name_to_handle_at(long res, long dfd,
3068 |                                                      long name, long handle,
3069 |                                                      long mnt_id, long flag);
3070 | void __sanitizer_syscall_pre_impl_open_by_handle_at(long mountdirfd,
```
- **Line 3061 / 第 3061 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3062 / 第 3062 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_old_mmap`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_old_mmap`。
- **Line 3063 / 第 3063 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_old_mmap`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_old_mmap`。
- **Line 3064 / 第 3064 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3065 / 第 3065 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3066 / 第 3066 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3067 / 第 3067 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3068 / 第 3068 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3069 / 第 3069 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3070 / 第 3070 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3071-3080 / 第 3071-3080 行
```cpp
3071 |                                                     long handle, long flags);
3072 | void __sanitizer_syscall_post_impl_open_by_handle_at(long res, long mountdirfd,
3073 |                                                      long handle, long flags);
3074 | void __sanitizer_syscall_pre_impl_setns(long fd, long nstype);
3075 | void __sanitizer_syscall_post_impl_setns(long res, long fd, long nstype);
3076 | void __sanitizer_syscall_pre_impl_process_vm_readv(long pid, long lvec,
3077 |                                                    long liovcnt, long rvec,
3078 |                                                    long riovcnt, long flags);
3079 | void __sanitizer_syscall_post_impl_process_vm_readv(long res, long pid,
3080 |                                                     long lvec, long liovcnt,
```
- **Line 3071 / 第 3071 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3072 / 第 3072 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3073 / 第 3073 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3074 / 第 3074 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setns`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setns`。
- **Line 3075 / 第 3075 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setns`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setns`。
- **Line 3076 / 第 3076 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3077 / 第 3077 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3078 / 第 3078 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3079 / 第 3079 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3080 / 第 3080 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3081-3090 / 第 3081-3090 行
```cpp
3081 |                                                     long rvec, long riovcnt,
3082 |                                                     long flags);
3083 | void __sanitizer_syscall_pre_impl_process_vm_writev(long pid, long lvec,
3084 |                                                     long liovcnt, long rvec,
3085 |                                                     long riovcnt, long flags);
3086 | void __sanitizer_syscall_post_impl_process_vm_writev(long res, long pid,
3087 |                                                      long lvec, long liovcnt,
3088 |                                                      long rvec, long riovcnt,
3089 |                                                      long flags);
3090 | void __sanitizer_syscall_pre_impl_fork();
```
- **Line 3081 / 第 3081 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3082 / 第 3082 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3083 / 第 3083 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3084 / 第 3084 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3085 / 第 3085 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3086 / 第 3086 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3087 / 第 3087 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3088 / 第 3088 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3089 / 第 3089 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3090 / 第 3090 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fork`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fork`。

### Lines 3091-3100 / 第 3091-3100 行
```cpp
3091 | void __sanitizer_syscall_post_impl_fork(long res);
3092 | void __sanitizer_syscall_pre_impl_vfork();
3093 | void __sanitizer_syscall_post_impl_vfork(long res);
3094 | void __sanitizer_syscall_pre_impl_sigaction(long signum, long act, long oldact);
3095 | void __sanitizer_syscall_post_impl_sigaction(long res, long signum, long act,
3096 |                                              long oldact);
3097 | void __sanitizer_syscall_pre_impl_rt_sigaction(long signum, long act,
3098 |                                                long oldact, long sz);
3099 | void __sanitizer_syscall_post_impl_rt_sigaction(long res, long signum, long act,
3100 |                                                 long oldact, long sz);
```
- **Line 3091 / 第 3091 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fork`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fork`。
- **Line 3092 / 第 3092 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_vfork`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_vfork`。
- **Line 3093 / 第 3093 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_vfork`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_vfork`。
- **Line 3094 / 第 3094 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sigaction`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sigaction`。
- **Line 3095 / 第 3095 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3096 / 第 3096 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3097 / 第 3097 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3098 / 第 3098 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3099 / 第 3099 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3100 / 第 3100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3101-3110 / 第 3101-3110 行
```cpp
3101 | void __sanitizer_syscall_pre_impl_sigaltstack(long ss, long oss);
3102 | void __sanitizer_syscall_post_impl_sigaltstack(long res, long ss, long oss);
3103 | void __sanitizer_syscall_pre_impl_futex(long uaddr, long futex_op, long val,
3104 |                                         long timeout, long uaddr2, long val3);
3105 | void __sanitizer_syscall_post_impl_futex(long res, long uaddr, long futex_op,
3106 |                                          long val, long timeout, long uaddr2,
3107 |                                          long val3);
3108 | #ifdef __cplusplus
3109 | } // extern "C"
3110 | #endif
```
- **Line 3101 / 第 3101 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sigaltstack`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sigaltstack`。
- **Line 3102 / 第 3102 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sigaltstack`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sigaltstack`。
- **Line 3103 / 第 3103 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3104 / 第 3104 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3105 / 第 3105 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3106 / 第 3106 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3107 / 第 3107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3108 / 第 3108 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 3109 / 第 3109 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3110 / 第 3110 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

### Lines 3111-3112 / 第 3111-3112 行
```cpp
3111 | 
3112 | #endif // SANITIZER_LINUX_SYSCALL_HOOKS_H
```
- **Line 3111 / 第 3111 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 3112 / 第 3112 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

## Key Concepts / 关键概念

- **EN**: sanitizer common runtime infrastructure
  - **CN**: sanitizer 通用运行时基础设施
- **EN**: system call wrapping hooks
  - **CN**: 系统调用包装钩子
- **EN**: C ABI compatibility
  - **CN**: C ABI 兼容性
- **EN**: public sanitizer C interfaces
  - **CN**: 公开的 sanitizer C 接口

## Dependencies / 依赖关系

- No direct `#include` dependencies appear in this file. / 该文件中没有直接出现 `#include` 依赖。
