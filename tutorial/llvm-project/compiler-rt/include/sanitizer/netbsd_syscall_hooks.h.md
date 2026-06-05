# netbsd_syscall_hooks.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/include/sanitizer/netbsd_syscall_hooks.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This file is a part of public sanitizer interface.
  - **CN**: 声明 sanitizer 公开接口中与 `netbsd_syscall_hooks` 相关的 API、类型或常量。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行
```cpp
   1 | //===-- netbsd_syscall_hooks.h --------------------------------------------===//
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
  17 | //   long long res = syscall(SYS_getfoo, ...args...);
  18 | //   __sanitizer_syscall_post_getfoo(res, ...args...);
  19 | //
  20 | // DO NOT EDIT! THIS FILE HAS BEEN GENERATED!
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
- **Line 20 / 第 20 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 21-30 / 第 21-30 行
```cpp
  21 | //
  22 | // Generated with: generate_netbsd_syscalls.awk
  23 | // Generated date: 2020-09-10
  24 | // Generated from: syscalls.master,v 1.306 2020/08/14 00:53:16 riastradh Exp
  25 | //
  26 | //===----------------------------------------------------------------------===//
  27 | #ifndef SANITIZER_NETBSD_SYSCALL_HOOKS_H
  28 | #define SANITIZER_NETBSD_SYSCALL_HOOKS_H
  29 | 
  30 | #define __sanitizer_syscall_pre_syscall(code, arg0, arg1, arg2, arg3, arg4,    \
```
- **Line 21 / 第 21 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 22 / 第 22 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 23 / 第 23 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 24 / 第 24 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 25 / 第 25 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 26 / 第 26 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 27 / 第 27 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 28 / 第 28 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 29 / 第 29 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 30 / 第 30 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 31-40 / 第 31-40 行
```cpp
  31 |                                         arg5, arg6, arg7)                      \
  32 |   __sanitizer_syscall_pre_impl_syscall(                                        \
  33 |       (long long)(code), (long long)(arg0), (long long)(arg1),                 \
  34 |       (long long)(arg2), (long long)(arg3), (long long)(arg4),                 \
  35 |       (long long)(arg5), (long long)(arg6), (long long)(arg7))
  36 | #define __sanitizer_syscall_post_syscall(res, code, arg0, arg1, arg2, arg3,    \
  37 |                                          arg4, arg5, arg6, arg7)               \
  38 |   __sanitizer_syscall_post_impl_syscall(                                       \
  39 |       res, (long long)(code), (long long)(arg0), (long long)(arg1),            \
  40 |       (long long)(arg2), (long long)(arg3), (long long)(arg4),                 \
```
- **Line 31 / 第 31 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 32 / 第 32 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 33 / 第 33 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 34 / 第 34 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 35 / 第 35 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 36 / 第 36 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 37 / 第 37 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 38 / 第 38 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 39 / 第 39 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 40 / 第 40 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 41-50 / 第 41-50 行
```cpp
  41 |       (long long)(arg5), (long long)(arg6), (long long)(arg7))
  42 | #define __sanitizer_syscall_pre_exit(rval)                                     \
  43 |   __sanitizer_syscall_pre_impl_exit((long long)(rval))
  44 | #define __sanitizer_syscall_post_exit(res, rval)                               \
  45 |   __sanitizer_syscall_post_impl_exit(res, (long long)(rval))
  46 | #define __sanitizer_syscall_pre_fork() __sanitizer_syscall_pre_impl_fork()
  47 | #define __sanitizer_syscall_post_fork(res)                                     \
  48 |   __sanitizer_syscall_post_impl_fork(res)
  49 | #define __sanitizer_syscall_pre_read(fd, buf, nbyte)                           \
  50 |   __sanitizer_syscall_pre_impl_read((long long)(fd), (long long)(buf),         \
```
- **Line 41 / 第 41 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 42 / 第 42 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 43 / 第 43 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 44 / 第 44 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 45 / 第 45 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 46 / 第 46 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 47 / 第 47 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 48 / 第 48 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 49 / 第 49 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 50 / 第 50 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 51-60 / 第 51-60 行
```cpp
  51 |                                     (long long)(nbyte))
  52 | #define __sanitizer_syscall_post_read(res, fd, buf, nbyte)                     \
  53 |   __sanitizer_syscall_post_impl_read(res, (long long)(fd), (long long)(buf),   \
  54 |                                      (long long)(nbyte))
  55 | #define __sanitizer_syscall_pre_write(fd, buf, nbyte)                          \
  56 |   __sanitizer_syscall_pre_impl_write((long long)(fd), (long long)(buf),        \
  57 |                                      (long long)(nbyte))
  58 | #define __sanitizer_syscall_post_write(res, fd, buf, nbyte)                    \
  59 |   __sanitizer_syscall_post_impl_write(res, (long long)(fd), (long long)(buf),  \
  60 |                                       (long long)(nbyte))
```
- **Line 51 / 第 51 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 52 / 第 52 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 53 / 第 53 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 54 / 第 54 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 55 / 第 55 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 56 / 第 56 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 57 / 第 57 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 58 / 第 58 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 59 / 第 59 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 60 / 第 60 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 61-70 / 第 61-70 行
```cpp
  61 | #define __sanitizer_syscall_pre_open(path, flags, mode)                        \
  62 |   __sanitizer_syscall_pre_impl_open((long long)(path), (long long)(flags),     \
  63 |                                     (long long)(mode))
  64 | #define __sanitizer_syscall_post_open(res, path, flags, mode)                  \
  65 |   __sanitizer_syscall_post_impl_open(res, (long long)(path),                   \
  66 |                                      (long long)(flags), (long long)(mode))
  67 | #define __sanitizer_syscall_pre_close(fd)                                      \
  68 |   __sanitizer_syscall_pre_impl_close((long long)(fd))
  69 | #define __sanitizer_syscall_post_close(res, fd)                                \
  70 |   __sanitizer_syscall_post_impl_close(res, (long long)(fd))
```
- **Line 61 / 第 61 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 62 / 第 62 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 63 / 第 63 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 64 / 第 64 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 65 / 第 65 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 66 / 第 66 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 67 / 第 67 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 68 / 第 68 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 69 / 第 69 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 70 / 第 70 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 71-80 / 第 71-80 行
```cpp
  71 | #define __sanitizer_syscall_pre_compat_50_wait4(pid, status, options, rusage)  \
  72 |   __sanitizer_syscall_pre_impl_compat_50_wait4(                                \
  73 |       (long long)(pid), (long long)(status), (long long)(options),             \
  74 |       (long long)(rusage))
  75 | #define __sanitizer_syscall_post_compat_50_wait4(res, pid, status, options,    \
  76 |                                                  rusage)                       \
  77 |   __sanitizer_syscall_post_impl_compat_50_wait4(                               \
  78 |       res, (long long)(pid), (long long)(status), (long long)(options),        \
  79 |       (long long)(rusage))
  80 | #define __sanitizer_syscall_pre_compat_43_ocreat(path, mode)                   \
```
- **Line 71 / 第 71 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 72 / 第 72 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 73 / 第 73 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 74 / 第 74 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 75 / 第 75 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 76 / 第 76 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 77 / 第 77 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 78 / 第 78 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 79 / 第 79 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 80 / 第 80 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 81-90 / 第 81-90 行
```cpp
  81 |   __sanitizer_syscall_pre_impl_compat_43_ocreat((long long)(path),             \
  82 |                                                 (long long)(mode))
  83 | #define __sanitizer_syscall_post_compat_43_ocreat(res, path, mode)             \
  84 |   __sanitizer_syscall_post_impl_compat_43_ocreat(res, (long long)(path),       \
  85 |                                                  (long long)(mode))
  86 | #define __sanitizer_syscall_pre_link(path, link)                               \
  87 |   __sanitizer_syscall_pre_impl_link((long long)(path), (long long)(link))
  88 | #define __sanitizer_syscall_post_link(res, path, link)                         \
  89 |   __sanitizer_syscall_post_impl_link(res, (long long)(path), (long long)(link))
  90 | #define __sanitizer_syscall_pre_unlink(path)                                   \
```
- **Line 81 / 第 81 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 82 / 第 82 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 83 / 第 83 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 84 / 第 84 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 85 / 第 85 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 86 / 第 86 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 87 / 第 87 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 88 / 第 88 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 89 / 第 89 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 90 / 第 90 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 91-100 / 第 91-100 行
```cpp
  91 |   __sanitizer_syscall_pre_impl_unlink((long long)(path))
  92 | #define __sanitizer_syscall_post_unlink(res, path)                             \
  93 |   __sanitizer_syscall_post_impl_unlink(res, (long long)(path))
  94 | /* syscall 11 has been skipped */
  95 | #define __sanitizer_syscall_pre_chdir(path)                                    \
  96 |   __sanitizer_syscall_pre_impl_chdir((long long)(path))
  97 | #define __sanitizer_syscall_post_chdir(res, path)                              \
  98 |   __sanitizer_syscall_post_impl_chdir(res, (long long)(path))
  99 | #define __sanitizer_syscall_pre_fchdir(fd)                                     \
 100 |   __sanitizer_syscall_pre_impl_fchdir((long long)(fd))
```
- **Line 91 / 第 91 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 92 / 第 92 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 93 / 第 93 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 94 / 第 94 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 95 / 第 95 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 96 / 第 96 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 97 / 第 97 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 98 / 第 98 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 99 / 第 99 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 100 / 第 100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 101-110 / 第 101-110 行
```cpp
 101 | #define __sanitizer_syscall_post_fchdir(res, fd)                               \
 102 |   __sanitizer_syscall_post_impl_fchdir(res, (long long)(fd))
 103 | #define __sanitizer_syscall_pre_compat_50_mknod(path, mode, dev)               \
 104 |   __sanitizer_syscall_pre_impl_compat_50_mknod(                                \
 105 |       (long long)(path), (long long)(mode), (long long)(dev))
 106 | #define __sanitizer_syscall_post_compat_50_mknod(res, path, mode, dev)         \
 107 |   __sanitizer_syscall_post_impl_compat_50_mknod(                               \
 108 |       res, (long long)(path), (long long)(mode), (long long)(dev))
 109 | #define __sanitizer_syscall_pre_chmod(path, mode)                              \
 110 |   __sanitizer_syscall_pre_impl_chmod((long long)(path), (long long)(mode))
```
- **Line 101 / 第 101 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 102 / 第 102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 103 / 第 103 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 104 / 第 104 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 105 / 第 105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 106 / 第 106 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 107 / 第 107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 108 / 第 108 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 109 / 第 109 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 110 / 第 110 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 111-120 / 第 111-120 行
```cpp
 111 | #define __sanitizer_syscall_post_chmod(res, path, mode)                        \
 112 |   __sanitizer_syscall_post_impl_chmod(res, (long long)(path), (long long)(mode))
 113 | #define __sanitizer_syscall_pre_chown(path, uid, gid)                          \
 114 |   __sanitizer_syscall_pre_impl_chown((long long)(path), (long long)(uid),      \
 115 |                                      (long long)(gid))
 116 | #define __sanitizer_syscall_post_chown(res, path, uid, gid)                    \
 117 |   __sanitizer_syscall_post_impl_chown(res, (long long)(path),                  \
 118 |                                       (long long)(uid), (long long)(gid))
 119 | #define __sanitizer_syscall_pre_break(nsize)                                   \
 120 |   __sanitizer_syscall_pre_impl_break((long long)(nsize))
```
- **Line 111 / 第 111 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 112 / 第 112 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 113 / 第 113 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 114 / 第 114 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 115 / 第 115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 116 / 第 116 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 117 / 第 117 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 118 / 第 118 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 119 / 第 119 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 120 / 第 120 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 121-130 / 第 121-130 行
```cpp
 121 | #define __sanitizer_syscall_post_break(res, nsize)                             \
 122 |   __sanitizer_syscall_post_impl_break(res, (long long)(nsize))
 123 | #define __sanitizer_syscall_pre_compat_20_getfsstat(buf, bufsize, flags)       \
 124 |   __sanitizer_syscall_pre_impl_compat_20_getfsstat(                            \
 125 |       (long long)(buf), (long long)(bufsize), (long long)(flags))
 126 | #define __sanitizer_syscall_post_compat_20_getfsstat(res, buf, bufsize, flags) \
 127 |   __sanitizer_syscall_post_impl_compat_20_getfsstat(                           \
 128 |       res, (long long)(buf), (long long)(bufsize), (long long)(flags))
 129 | #define __sanitizer_syscall_pre_compat_43_olseek(fd, offset, whence)           \
 130 |   __sanitizer_syscall_pre_impl_compat_43_olseek(                               \
```
- **Line 121 / 第 121 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
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
 131 |       (long long)(fd), (long long)(offset), (long long)(whence))
 132 | #define __sanitizer_syscall_post_compat_43_olseek(res, fd, offset, whence)     \
 133 |   __sanitizer_syscall_post_impl_compat_43_olseek(                              \
 134 |       res, (long long)(fd), (long long)(offset), (long long)(whence))
 135 | #define __sanitizer_syscall_pre_getpid() __sanitizer_syscall_pre_impl_getpid()
 136 | #define __sanitizer_syscall_post_getpid(res)                                   \
 137 |   __sanitizer_syscall_post_impl_getpid(res)
 138 | #define __sanitizer_syscall_pre_compat_40_mount(type, path, flags, data)       \
 139 |   __sanitizer_syscall_pre_impl_compat_40_mount(                                \
 140 |       (long long)(type), (long long)(path), (long long)(flags),                \
```
- **Line 131 / 第 131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 132 / 第 132 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 133 / 第 133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 134 / 第 134 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 135 / 第 135 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 136 / 第 136 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 137 / 第 137 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 138 / 第 138 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 139 / 第 139 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 140 / 第 140 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 141-150 / 第 141-150 行
```cpp
 141 |       (long long)(data))
 142 | #define __sanitizer_syscall_post_compat_40_mount(res, type, path, flags, data) \
 143 |   __sanitizer_syscall_post_impl_compat_40_mount(                               \
 144 |       res, (long long)(type), (long long)(path), (long long)(flags),           \
 145 |       (long long)(data))
 146 | #define __sanitizer_syscall_pre_unmount(path, flags)                           \
 147 |   __sanitizer_syscall_pre_impl_unmount((long long)(path), (long long)(flags))
 148 | #define __sanitizer_syscall_post_unmount(res, path, flags)                     \
 149 |   __sanitizer_syscall_post_impl_unmount(res, (long long)(path),                \
 150 |                                         (long long)(flags))
```
- **Line 141 / 第 141 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 142 / 第 142 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 143 / 第 143 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 144 / 第 144 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 145 / 第 145 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 146 / 第 146 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 147 / 第 147 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 148 / 第 148 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 149 / 第 149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 150 / 第 150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 151-160 / 第 151-160 行
```cpp
 151 | #define __sanitizer_syscall_pre_setuid(uid)                                    \
 152 |   __sanitizer_syscall_pre_impl_setuid((long long)(uid))
 153 | #define __sanitizer_syscall_post_setuid(res, uid)                              \
 154 |   __sanitizer_syscall_post_impl_setuid(res, (long long)(uid))
 155 | #define __sanitizer_syscall_pre_getuid() __sanitizer_syscall_pre_impl_getuid()
 156 | #define __sanitizer_syscall_post_getuid(res)                                   \
 157 |   __sanitizer_syscall_post_impl_getuid(res)
 158 | #define __sanitizer_syscall_pre_geteuid() __sanitizer_syscall_pre_impl_geteuid()
 159 | #define __sanitizer_syscall_post_geteuid(res)                                  \
 160 |   __sanitizer_syscall_post_impl_geteuid(res)
```
- **Line 151 / 第 151 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 152 / 第 152 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 153 / 第 153 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 154 / 第 154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 155 / 第 155 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 156 / 第 156 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 157 / 第 157 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 158 / 第 158 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 159 / 第 159 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 160 / 第 160 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 161-170 / 第 161-170 行
```cpp
 161 | #define __sanitizer_syscall_pre_ptrace(req, pid, addr, data)                   \
 162 |   __sanitizer_syscall_pre_impl_ptrace((long long)(req), (long long)(pid),      \
 163 |                                       (long long)(addr), (long long)(data))
 164 | #define __sanitizer_syscall_post_ptrace(res, req, pid, addr, data)             \
 165 |   __sanitizer_syscall_post_impl_ptrace(res, (long long)(req),                  \
 166 |                                        (long long)(pid), (long long)(addr),    \
 167 |                                        (long long)(data))
 168 | #define __sanitizer_syscall_pre_recvmsg(s, msg, flags)                         \
 169 |   __sanitizer_syscall_pre_impl_recvmsg((long long)(s), (long long)(msg),       \
 170 |                                        (long long)(flags))
```
- **Line 161 / 第 161 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 162 / 第 162 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 163 / 第 163 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 164 / 第 164 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 165 / 第 165 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 166 / 第 166 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 167 / 第 167 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 168 / 第 168 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 169 / 第 169 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 170 / 第 170 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 171-180 / 第 171-180 行
```cpp
 171 | #define __sanitizer_syscall_post_recvmsg(res, s, msg, flags)                   \
 172 |   __sanitizer_syscall_post_impl_recvmsg(res, (long long)(s), (long long)(msg), \
 173 |                                         (long long)(flags))
 174 | #define __sanitizer_syscall_pre_sendmsg(s, msg, flags)                         \
 175 |   __sanitizer_syscall_pre_impl_sendmsg((long long)(s), (long long)(msg),       \
 176 |                                        (long long)(flags))
 177 | #define __sanitizer_syscall_post_sendmsg(res, s, msg, flags)                   \
 178 |   __sanitizer_syscall_post_impl_sendmsg(res, (long long)(s), (long long)(msg), \
 179 |                                         (long long)(flags))
 180 | #define __sanitizer_syscall_pre_recvfrom(s, buf, len, flags, from,             \
```
- **Line 171 / 第 171 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 172 / 第 172 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 173 / 第 173 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 174 / 第 174 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 175 / 第 175 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 176 / 第 176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 177 / 第 177 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 178 / 第 178 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 179 / 第 179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 180 / 第 180 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 181-190 / 第 181-190 行
```cpp
 181 |                                          fromlenaddr)                          \
 182 |   __sanitizer_syscall_pre_impl_recvfrom(                                       \
 183 |       (long long)(s), (long long)(buf), (long long)(len), (long long)(flags),  \
 184 |       (long long)(from), (long long)(fromlenaddr))
 185 | #define __sanitizer_syscall_post_recvfrom(res, s, buf, len, flags, from,       \
 186 |                                           fromlenaddr)                         \
 187 |   __sanitizer_syscall_post_impl_recvfrom(                                      \
 188 |       res, (long long)(s), (long long)(buf), (long long)(len),                 \
 189 |       (long long)(flags), (long long)(from), (long long)(fromlenaddr))
 190 | #define __sanitizer_syscall_pre_accept(s, name, anamelen)                      \
```
- **Line 181 / 第 181 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 182 / 第 182 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 183 / 第 183 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 184 / 第 184 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 185 / 第 185 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 186 / 第 186 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 187 / 第 187 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 188 / 第 188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 189 / 第 189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 190 / 第 190 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 191-200 / 第 191-200 行
```cpp
 191 |   __sanitizer_syscall_pre_impl_accept((long long)(s), (long long)(name),       \
 192 |                                       (long long)(anamelen))
 193 | #define __sanitizer_syscall_post_accept(res, s, name, anamelen)                \
 194 |   __sanitizer_syscall_post_impl_accept(res, (long long)(s), (long long)(name), \
 195 |                                        (long long)(anamelen))
 196 | #define __sanitizer_syscall_pre_getpeername(fdes, asa, alen)                   \
 197 |   __sanitizer_syscall_pre_impl_getpeername(                                    \
 198 |       (long long)(fdes), (long long)(asa), (long long)(alen))
 199 | #define __sanitizer_syscall_post_getpeername(res, fdes, asa, alen)             \
 200 |   __sanitizer_syscall_post_impl_getpeername(                                   \
```
- **Line 191 / 第 191 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 192 / 第 192 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 193 / 第 193 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 194 / 第 194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 195 / 第 195 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 196 / 第 196 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 197 / 第 197 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 198 / 第 198 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 199 / 第 199 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 200 / 第 200 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 201-210 / 第 201-210 行
```cpp
 201 |       res, (long long)(fdes), (long long)(asa), (long long)(alen))
 202 | #define __sanitizer_syscall_pre_getsockname(fdes, asa, alen)                   \
 203 |   __sanitizer_syscall_pre_impl_getsockname(                                    \
 204 |       (long long)(fdes), (long long)(asa), (long long)(alen))
 205 | #define __sanitizer_syscall_post_getsockname(res, fdes, asa, alen)             \
 206 |   __sanitizer_syscall_post_impl_getsockname(                                   \
 207 |       res, (long long)(fdes), (long long)(asa), (long long)(alen))
 208 | #define __sanitizer_syscall_pre_access(path, flags)                            \
 209 |   __sanitizer_syscall_pre_impl_access((long long)(path), (long long)(flags))
 210 | #define __sanitizer_syscall_post_access(res, path, flags)                      \
```
- **Line 201 / 第 201 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 202 / 第 202 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 203 / 第 203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 204 / 第 204 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 205 / 第 205 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 206 / 第 206 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 207 / 第 207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 208 / 第 208 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 209 / 第 209 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 210 / 第 210 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 211-220 / 第 211-220 行
```cpp
 211 |   __sanitizer_syscall_post_impl_access(res, (long long)(path),                 \
 212 |                                        (long long)(flags))
 213 | #define __sanitizer_syscall_pre_chflags(path, flags)                           \
 214 |   __sanitizer_syscall_pre_impl_chflags((long long)(path), (long long)(flags))
 215 | #define __sanitizer_syscall_post_chflags(res, path, flags)                     \
 216 |   __sanitizer_syscall_post_impl_chflags(res, (long long)(path),                \
 217 |                                         (long long)(flags))
 218 | #define __sanitizer_syscall_pre_fchflags(fd, flags)                            \
 219 |   __sanitizer_syscall_pre_impl_fchflags((long long)(fd), (long long)(flags))
 220 | #define __sanitizer_syscall_post_fchflags(res, fd, flags)                      \
```
- **Line 211 / 第 211 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 212 / 第 212 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 213 / 第 213 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 214 / 第 214 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 215 / 第 215 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 216 / 第 216 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 217 / 第 217 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 218 / 第 218 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 219 / 第 219 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 220 / 第 220 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 221-230 / 第 221-230 行
```cpp
 221 |   __sanitizer_syscall_post_impl_fchflags(res, (long long)(fd),                 \
 222 |                                          (long long)(flags))
 223 | #define __sanitizer_syscall_pre_sync() __sanitizer_syscall_pre_impl_sync()
 224 | #define __sanitizer_syscall_post_sync(res)                                     \
 225 |   __sanitizer_syscall_post_impl_sync(res)
 226 | #define __sanitizer_syscall_pre_kill(pid, signum)                              \
 227 |   __sanitizer_syscall_pre_impl_kill((long long)(pid), (long long)(signum))
 228 | #define __sanitizer_syscall_post_kill(res, pid, signum)                        \
 229 |   __sanitizer_syscall_post_impl_kill(res, (long long)(pid), (long long)(signum))
 230 | #define __sanitizer_syscall_pre_compat_43_stat43(path, ub)                     \
```
- **Line 221 / 第 221 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 222 / 第 222 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 223 / 第 223 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 224 / 第 224 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 225 / 第 225 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 226 / 第 226 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 227 / 第 227 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 228 / 第 228 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 229 / 第 229 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 230 / 第 230 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 231-240 / 第 231-240 行
```cpp
 231 |   __sanitizer_syscall_pre_impl_compat_43_stat43((long long)(path),             \
 232 |                                                 (long long)(ub))
 233 | #define __sanitizer_syscall_post_compat_43_stat43(res, path, ub)               \
 234 |   __sanitizer_syscall_post_impl_compat_43_stat43(res, (long long)(path),       \
 235 |                                                  (long long)(ub))
 236 | #define __sanitizer_syscall_pre_getppid() __sanitizer_syscall_pre_impl_getppid()
 237 | #define __sanitizer_syscall_post_getppid(res)                                  \
 238 |   __sanitizer_syscall_post_impl_getppid(res)
 239 | #define __sanitizer_syscall_pre_compat_43_lstat43(path, ub)                    \
 240 |   __sanitizer_syscall_pre_impl_compat_43_lstat43((long long)(path),            \
```
- **Line 231 / 第 231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 232 / 第 232 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 233 / 第 233 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 234 / 第 234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 235 / 第 235 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 236 / 第 236 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 237 / 第 237 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 238 / 第 238 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 239 / 第 239 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 240 / 第 240 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 241-250 / 第 241-250 行
```cpp
 241 |                                                  (long long)(ub))
 242 | #define __sanitizer_syscall_post_compat_43_lstat43(res, path, ub)              \
 243 |   __sanitizer_syscall_post_impl_compat_43_lstat43(res, (long long)(path),      \
 244 |                                                   (long long)(ub))
 245 | #define __sanitizer_syscall_pre_dup(fd)                                        \
 246 |   __sanitizer_syscall_pre_impl_dup((long long)(fd))
 247 | #define __sanitizer_syscall_post_dup(res, fd)                                  \
 248 |   __sanitizer_syscall_post_impl_dup(res, (long long)(fd))
 249 | #define __sanitizer_syscall_pre_pipe() __sanitizer_syscall_pre_impl_pipe()
 250 | #define __sanitizer_syscall_post_pipe(res)                                     \
```
- **Line 241 / 第 241 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 242 / 第 242 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 243 / 第 243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 244 / 第 244 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 245 / 第 245 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 246 / 第 246 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 247 / 第 247 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 248 / 第 248 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 249 / 第 249 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 250 / 第 250 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 251-260 / 第 251-260 行
```cpp
 251 |   __sanitizer_syscall_post_impl_pipe(res)
 252 | #define __sanitizer_syscall_pre_getegid() __sanitizer_syscall_pre_impl_getegid()
 253 | #define __sanitizer_syscall_post_getegid(res)                                  \
 254 |   __sanitizer_syscall_post_impl_getegid(res)
 255 | #define __sanitizer_syscall_pre_profil(samples, size, offset, scale)           \
 256 |   __sanitizer_syscall_pre_impl_profil((long long)(samples), (long long)(size), \
 257 |                                       (long long)(offset), (long long)(scale))
 258 | #define __sanitizer_syscall_post_profil(res, samples, size, offset, scale)     \
 259 |   __sanitizer_syscall_post_impl_profil(res, (long long)(samples),              \
 260 |                                        (long long)(size), (long long)(offset), \
```
- **Line 251 / 第 251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 252 / 第 252 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 253 / 第 253 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 254 / 第 254 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 255 / 第 255 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 256 / 第 256 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 257 / 第 257 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 258 / 第 258 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 259 / 第 259 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 260 / 第 260 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 261-270 / 第 261-270 行
```cpp
 261 |                                        (long long)(scale))
 262 | #define __sanitizer_syscall_pre_ktrace(fname, ops, facs, pid)                  \
 263 |   __sanitizer_syscall_pre_impl_ktrace((long long)(fname), (long long)(ops),    \
 264 |                                       (long long)(facs), (long long)(pid))
 265 | #define __sanitizer_syscall_post_ktrace(res, fname, ops, facs, pid)            \
 266 |   __sanitizer_syscall_post_impl_ktrace(res, (long long)(fname),                \
 267 |                                        (long long)(ops), (long long)(facs),    \
 268 |                                        (long long)(pid))
 269 | #define __sanitizer_syscall_pre_compat_13_sigaction13(signum, nsa, osa)        \
 270 |   __sanitizer_syscall_pre_impl_compat_13_sigaction13(                          \
```
- **Line 261 / 第 261 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 262 / 第 262 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 263 / 第 263 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 264 / 第 264 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 265 / 第 265 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 266 / 第 266 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 267 / 第 267 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 268 / 第 268 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 269 / 第 269 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 270 / 第 270 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 271-280 / 第 271-280 行
```cpp
 271 |       (long long)(signum), (long long)(nsa), (long long)(osa))
 272 | #define __sanitizer_syscall_post_compat_13_sigaction13(res, signum, nsa, osa)  \
 273 |   __sanitizer_syscall_post_impl_compat_13_sigaction13(                         \
 274 |       res, (long long)(signum), (long long)(nsa), (long long)(osa))
 275 | #define __sanitizer_syscall_pre_getgid() __sanitizer_syscall_pre_impl_getgid()
 276 | #define __sanitizer_syscall_post_getgid(res)                                   \
 277 |   __sanitizer_syscall_post_impl_getgid(res)
 278 | #define __sanitizer_syscall_pre_compat_13_sigprocmask13(how, mask)             \
 279 |   __sanitizer_syscall_pre_impl_compat_13_sigprocmask13((long long)(how),       \
 280 |                                                        (long long)(mask))
```
- **Line 271 / 第 271 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 272 / 第 272 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 273 / 第 273 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 274 / 第 274 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 275 / 第 275 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 276 / 第 276 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 277 / 第 277 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 278 / 第 278 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 279 / 第 279 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 280 / 第 280 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 281-290 / 第 281-290 行
```cpp
 281 | #define __sanitizer_syscall_post_compat_13_sigprocmask13(res, how, mask)       \
 282 |   __sanitizer_syscall_post_impl_compat_13_sigprocmask13(res, (long long)(how), \
 283 |                                                         (long long)(mask))
 284 | #define __sanitizer_syscall_pre___getlogin(namebuf, namelen)                   \
 285 |   __sanitizer_syscall_pre_impl___getlogin((long long)(namebuf),                \
 286 |                                           (long long)(namelen))
 287 | #define __sanitizer_syscall_post___getlogin(res, namebuf, namelen)             \
 288 |   __sanitizer_syscall_post_impl___getlogin(res, (long long)(namebuf),          \
 289 |                                            (long long)(namelen))
 290 | #define __sanitizer_syscall_pre___setlogin(namebuf)                            \
```
- **Line 281 / 第 281 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 282 / 第 282 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 283 / 第 283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 284 / 第 284 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 285 / 第 285 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 286 / 第 286 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 287 / 第 287 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 288 / 第 288 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 289 / 第 289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 290 / 第 290 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 291-300 / 第 291-300 行
```cpp
 291 |   __sanitizer_syscall_pre_impl___setlogin((long long)(namebuf))
 292 | #define __sanitizer_syscall_post___setlogin(res, namebuf)                      \
 293 |   __sanitizer_syscall_post_impl___setlogin(res, (long long)(namebuf))
 294 | #define __sanitizer_syscall_pre_acct(path)                                     \
 295 |   __sanitizer_syscall_pre_impl_acct((long long)(path))
 296 | #define __sanitizer_syscall_post_acct(res, path)                               \
 297 |   __sanitizer_syscall_post_impl_acct(res, (long long)(path))
 298 | #define __sanitizer_syscall_pre_compat_13_sigpending13()                       \
 299 |   __sanitizer_syscall_pre_impl_compat_13_sigpending13()
 300 | #define __sanitizer_syscall_post_compat_13_sigpending13(res)                   \
```
- **Line 291 / 第 291 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 292 / 第 292 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 293 / 第 293 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 294 / 第 294 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 295 / 第 295 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 296 / 第 296 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 297 / 第 297 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 298 / 第 298 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 299 / 第 299 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 300 / 第 300 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 301-310 / 第 301-310 行
```cpp
 301 |   __sanitizer_syscall_post_impl_compat_13_sigpending13(res)
 302 | #define __sanitizer_syscall_pre_compat_13_sigaltstack13(nss, oss)              \
 303 |   __sanitizer_syscall_pre_impl_compat_13_sigaltstack13((long long)(nss),       \
 304 |                                                        (long long)(oss))
 305 | #define __sanitizer_syscall_post_compat_13_sigaltstack13(res, nss, oss)        \
 306 |   __sanitizer_syscall_post_impl_compat_13_sigaltstack13(res, (long long)(nss), \
 307 |                                                         (long long)(oss))
 308 | #define __sanitizer_syscall_pre_ioctl(fd, com, data)                           \
 309 |   __sanitizer_syscall_pre_impl_ioctl((long long)(fd), (long long)(com),        \
 310 |                                      (long long)(data))
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
- **Line 310 / 第 310 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 311-320 / 第 311-320 行
```cpp
 311 | #define __sanitizer_syscall_post_ioctl(res, fd, com, data)                     \
 312 |   __sanitizer_syscall_post_impl_ioctl(res, (long long)(fd), (long long)(com),  \
 313 |                                       (long long)(data))
 314 | #define __sanitizer_syscall_pre_compat_12_oreboot(opt)                         \
 315 |   __sanitizer_syscall_pre_impl_compat_12_oreboot((long long)(opt))
 316 | #define __sanitizer_syscall_post_compat_12_oreboot(res, opt)                   \
 317 |   __sanitizer_syscall_post_impl_compat_12_oreboot(res, (long long)(opt))
 318 | #define __sanitizer_syscall_pre_revoke(path)                                   \
 319 |   __sanitizer_syscall_pre_impl_revoke((long long)(path))
 320 | #define __sanitizer_syscall_post_revoke(res, path)                             \
```
- **Line 311 / 第 311 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 312 / 第 312 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 313 / 第 313 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 314 / 第 314 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 315 / 第 315 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 316 / 第 316 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 317 / 第 317 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 318 / 第 318 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 319 / 第 319 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 320 / 第 320 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 321-330 / 第 321-330 行
```cpp
 321 |   __sanitizer_syscall_post_impl_revoke(res, (long long)(path))
 322 | #define __sanitizer_syscall_pre_symlink(path, link)                            \
 323 |   __sanitizer_syscall_pre_impl_symlink((long long)(path), (long long)(link))
 324 | #define __sanitizer_syscall_post_symlink(res, path, link)                      \
 325 |   __sanitizer_syscall_post_impl_symlink(res, (long long)(path),                \
 326 |                                         (long long)(link))
 327 | #define __sanitizer_syscall_pre_readlink(path, buf, count)                     \
 328 |   __sanitizer_syscall_pre_impl_readlink((long long)(path), (long long)(buf),   \
 329 |                                         (long long)(count))
 330 | #define __sanitizer_syscall_post_readlink(res, path, buf, count)               \
```
- **Line 321 / 第 321 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 322 / 第 322 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 323 / 第 323 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 324 / 第 324 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 325 / 第 325 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 326 / 第 326 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 327 / 第 327 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 328 / 第 328 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 329 / 第 329 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 330 / 第 330 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 331-340 / 第 331-340 行
```cpp
 331 |   __sanitizer_syscall_post_impl_readlink(res, (long long)(path),               \
 332 |                                          (long long)(buf), (long long)(count))
 333 | #define __sanitizer_syscall_pre_execve(path, argp, envp)                       \
 334 |   __sanitizer_syscall_pre_impl_execve((long long)(path), (long long)(argp),    \
 335 |                                       (long long)(envp))
 336 | #define __sanitizer_syscall_post_execve(res, path, argp, envp)                 \
 337 |   __sanitizer_syscall_post_impl_execve(res, (long long)(path),                 \
 338 |                                        (long long)(argp), (long long)(envp))
 339 | #define __sanitizer_syscall_pre_umask(newmask)                                 \
 340 |   __sanitizer_syscall_pre_impl_umask((long long)(newmask))
```
- **Line 331 / 第 331 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 332 / 第 332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 333 / 第 333 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 334 / 第 334 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 335 / 第 335 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 336 / 第 336 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 337 / 第 337 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 338 / 第 338 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 339 / 第 339 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 340 / 第 340 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 341-350 / 第 341-350 行
```cpp
 341 | #define __sanitizer_syscall_post_umask(res, newmask)                           \
 342 |   __sanitizer_syscall_post_impl_umask(res, (long long)(newmask))
 343 | #define __sanitizer_syscall_pre_chroot(path)                                   \
 344 |   __sanitizer_syscall_pre_impl_chroot((long long)(path))
 345 | #define __sanitizer_syscall_post_chroot(res, path)                             \
 346 |   __sanitizer_syscall_post_impl_chroot(res, (long long)(path))
 347 | #define __sanitizer_syscall_pre_compat_43_fstat43(fd, sb)                      \
 348 |   __sanitizer_syscall_pre_impl_compat_43_fstat43((long long)(fd),              \
 349 |                                                  (long long)(sb))
 350 | #define __sanitizer_syscall_post_compat_43_fstat43(res, fd, sb)                \
```
- **Line 341 / 第 341 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 342 / 第 342 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 343 / 第 343 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 344 / 第 344 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 345 / 第 345 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 346 / 第 346 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 347 / 第 347 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 348 / 第 348 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 349 / 第 349 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 350 / 第 350 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 351-360 / 第 351-360 行
```cpp
 351 |   __sanitizer_syscall_post_impl_compat_43_fstat43(res, (long long)(fd),        \
 352 |                                                   (long long)(sb))
 353 | #define __sanitizer_syscall_pre_compat_43_ogetkerninfo(op, where, size, arg)   \
 354 |   __sanitizer_syscall_pre_impl_compat_43_ogetkerninfo(                         \
 355 |       (long long)(op), (long long)(where), (long long)(size),                  \
 356 |       (long long)(arg))
 357 | #define __sanitizer_syscall_post_compat_43_ogetkerninfo(res, op, where, size,  \
 358 |                                                         arg)                   \
 359 |   __sanitizer_syscall_post_impl_compat_43_ogetkerninfo(                        \
 360 |       res, (long long)(op), (long long)(where), (long long)(size),             \
```
- **Line 351 / 第 351 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 352 / 第 352 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 353 / 第 353 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 354 / 第 354 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 355 / 第 355 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 356 / 第 356 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 357 / 第 357 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 358 / 第 358 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 359 / 第 359 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 360 / 第 360 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 361-370 / 第 361-370 行
```cpp
 361 |       (long long)(arg))
 362 | #define __sanitizer_syscall_pre_compat_43_ogetpagesize()                       \
 363 |   __sanitizer_syscall_pre_impl_compat_43_ogetpagesize()
 364 | #define __sanitizer_syscall_post_compat_43_ogetpagesize(res)                   \
 365 |   __sanitizer_syscall_post_impl_compat_43_ogetpagesize(res)
 366 | #define __sanitizer_syscall_pre_compat_12_msync(addr, len)                     \
 367 |   __sanitizer_syscall_pre_impl_compat_12_msync((long long)(addr),              \
 368 |                                                (long long)(len))
 369 | #define __sanitizer_syscall_post_compat_12_msync(res, addr, len)               \
 370 |   __sanitizer_syscall_post_impl_compat_12_msync(res, (long long)(addr),        \
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
 371 |                                                 (long long)(len))
 372 | #define __sanitizer_syscall_pre_vfork() __sanitizer_syscall_pre_impl_vfork()
 373 | #define __sanitizer_syscall_post_vfork(res)                                    \
 374 |   __sanitizer_syscall_post_impl_vfork(res)
 375 | /* syscall 67 has been skipped */
 376 | /* syscall 68 has been skipped */
 377 | /* syscall 69 has been skipped */
 378 | /* syscall 70 has been skipped */
 379 | #define __sanitizer_syscall_pre_compat_43_ommap(addr, len, prot, flags, fd,    \
 380 |                                                 pos)                           \
```
- **Line 371 / 第 371 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 372 / 第 372 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 373 / 第 373 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 374 / 第 374 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 375 / 第 375 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 376 / 第 376 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 377 / 第 377 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 378 / 第 378 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 379 / 第 379 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 380 / 第 380 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 381-390 / 第 381-390 行
```cpp
 381 |   __sanitizer_syscall_pre_impl_compat_43_ommap(                                \
 382 |       (long long)(addr), (long long)(len), (long long)(prot),                  \
 383 |       (long long)(flags), (long long)(fd), (long long)(pos))
 384 | #define __sanitizer_syscall_post_compat_43_ommap(res, addr, len, prot, flags,  \
 385 |                                                  fd, pos)                      \
 386 |   __sanitizer_syscall_post_impl_compat_43_ommap(                               \
 387 |       res, (long long)(addr), (long long)(len), (long long)(prot),             \
 388 |       (long long)(flags), (long long)(fd), (long long)(pos))
 389 | #define __sanitizer_syscall_pre_vadvise(anom)                                  \
 390 |   __sanitizer_syscall_pre_impl_vadvise((long long)(anom))
```
- **Line 381 / 第 381 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 382 / 第 382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 383 / 第 383 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 384 / 第 384 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 385 / 第 385 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 386 / 第 386 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 387 / 第 387 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 388 / 第 388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 389 / 第 389 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 390 / 第 390 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 391-400 / 第 391-400 行
```cpp
 391 | #define __sanitizer_syscall_post_vadvise(res, anom)                            \
 392 |   __sanitizer_syscall_post_impl_vadvise(res, (long long)(anom))
 393 | #define __sanitizer_syscall_pre_munmap(addr, len)                              \
 394 |   __sanitizer_syscall_pre_impl_munmap((long long)(addr), (long long)(len))
 395 | #define __sanitizer_syscall_post_munmap(res, addr, len)                        \
 396 |   __sanitizer_syscall_post_impl_munmap(res, (long long)(addr), (long long)(len))
 397 | #define __sanitizer_syscall_pre_mprotect(addr, len, prot)                      \
 398 |   __sanitizer_syscall_pre_impl_mprotect((long long)(addr), (long long)(len),   \
 399 |                                         (long long)(prot))
 400 | #define __sanitizer_syscall_post_mprotect(res, addr, len, prot)                \
```
- **Line 391 / 第 391 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 392 / 第 392 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 393 / 第 393 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 394 / 第 394 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 395 / 第 395 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 396 / 第 396 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 397 / 第 397 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 398 / 第 398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 399 / 第 399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 400 / 第 400 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 401-410 / 第 401-410 行
```cpp
 401 |   __sanitizer_syscall_post_impl_mprotect(res, (long long)(addr),               \
 402 |                                          (long long)(len), (long long)(prot))
 403 | #define __sanitizer_syscall_pre_madvise(addr, len, behav)                      \
 404 |   __sanitizer_syscall_pre_impl_madvise((long long)(addr), (long long)(len),    \
 405 |                                        (long long)(behav))
 406 | #define __sanitizer_syscall_post_madvise(res, addr, len, behav)                \
 407 |   __sanitizer_syscall_post_impl_madvise(res, (long long)(addr),                \
 408 |                                         (long long)(len), (long long)(behav))
 409 | /* syscall 76 has been skipped */
 410 | /* syscall 77 has been skipped */
```
- **Line 401 / 第 401 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 402 / 第 402 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 403 / 第 403 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 404 / 第 404 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 405 / 第 405 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 406 / 第 406 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 407 / 第 407 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 408 / 第 408 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 409 / 第 409 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 410 / 第 410 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 411-420 / 第 411-420 行
```cpp
 411 | #define __sanitizer_syscall_pre_mincore(addr, len, vec)                        \
 412 |   __sanitizer_syscall_pre_impl_mincore((long long)(addr), (long long)(len),    \
 413 |                                        (long long)(vec))
 414 | #define __sanitizer_syscall_post_mincore(res, addr, len, vec)                  \
 415 |   __sanitizer_syscall_post_impl_mincore(res, (long long)(addr),                \
 416 |                                         (long long)(len), (long long)(vec))
 417 | #define __sanitizer_syscall_pre_getgroups(gidsetsize, gidset)                  \
 418 |   __sanitizer_syscall_pre_impl_getgroups((long long)(gidsetsize),              \
 419 |                                          (long long)(gidset))
 420 | #define __sanitizer_syscall_post_getgroups(res, gidsetsize, gidset)            \
```
- **Line 411 / 第 411 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 412 / 第 412 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 413 / 第 413 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 414 / 第 414 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 415 / 第 415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 416 / 第 416 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 417 / 第 417 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 418 / 第 418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 419 / 第 419 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 420 / 第 420 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 421-430 / 第 421-430 行
```cpp
 421 |   __sanitizer_syscall_post_impl_getgroups(res, (long long)(gidsetsize),        \
 422 |                                           (long long)(gidset))
 423 | #define __sanitizer_syscall_pre_setgroups(gidsetsize, gidset)                  \
 424 |   __sanitizer_syscall_pre_impl_setgroups((long long)(gidsetsize),              \
 425 |                                          (long long)(gidset))
 426 | #define __sanitizer_syscall_post_setgroups(res, gidsetsize, gidset)            \
 427 |   __sanitizer_syscall_post_impl_setgroups(res, (long long)(gidsetsize),        \
 428 |                                           (long long)(gidset))
 429 | #define __sanitizer_syscall_pre_getpgrp() __sanitizer_syscall_pre_impl_getpgrp()
 430 | #define __sanitizer_syscall_post_getpgrp(res)                                  \
```
- **Line 421 / 第 421 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 422 / 第 422 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 423 / 第 423 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 424 / 第 424 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 425 / 第 425 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 426 / 第 426 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 427 / 第 427 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 428 / 第 428 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 429 / 第 429 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 430 / 第 430 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 431-440 / 第 431-440 行
```cpp
 431 |   __sanitizer_syscall_post_impl_getpgrp(res)
 432 | #define __sanitizer_syscall_pre_setpgid(pid, pgid)                             \
 433 |   __sanitizer_syscall_pre_impl_setpgid((long long)(pid), (long long)(pgid))
 434 | #define __sanitizer_syscall_post_setpgid(res, pid, pgid)                       \
 435 |   __sanitizer_syscall_post_impl_setpgid(res, (long long)(pid),                 \
 436 |                                         (long long)(pgid))
 437 | #define __sanitizer_syscall_pre_compat_50_setitimer(which, itv, oitv)          \
 438 |   __sanitizer_syscall_pre_impl_compat_50_setitimer(                            \
 439 |       (long long)(which), (long long)(itv), (long long)(oitv))
 440 | #define __sanitizer_syscall_post_compat_50_setitimer(res, which, itv, oitv)    \
```
- **Line 431 / 第 431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 432 / 第 432 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 433 / 第 433 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 434 / 第 434 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 435 / 第 435 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 436 / 第 436 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 437 / 第 437 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 438 / 第 438 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 439 / 第 439 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 440 / 第 440 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 441-450 / 第 441-450 行
```cpp
 441 |   __sanitizer_syscall_post_impl_compat_50_setitimer(                           \
 442 |       res, (long long)(which), (long long)(itv), (long long)(oitv))
 443 | #define __sanitizer_syscall_pre_compat_43_owait()                              \
 444 |   __sanitizer_syscall_pre_impl_compat_43_owait()
 445 | #define __sanitizer_syscall_post_compat_43_owait(res)                          \
 446 |   __sanitizer_syscall_post_impl_compat_43_owait(res)
 447 | #define __sanitizer_syscall_pre_compat_12_oswapon(name)                        \
 448 |   __sanitizer_syscall_pre_impl_compat_12_oswapon((long long)(name))
 449 | #define __sanitizer_syscall_post_compat_12_oswapon(res, name)                  \
 450 |   __sanitizer_syscall_post_impl_compat_12_oswapon(res, (long long)(name))
```
- **Line 441 / 第 441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 442 / 第 442 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 443 / 第 443 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 444 / 第 444 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 445 / 第 445 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 446 / 第 446 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 447 / 第 447 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 448 / 第 448 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 449 / 第 449 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 450 / 第 450 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 451-460 / 第 451-460 行
```cpp
 451 | #define __sanitizer_syscall_pre_compat_50_getitimer(which, itv)                \
 452 |   __sanitizer_syscall_pre_impl_compat_50_getitimer((long long)(which),         \
 453 |                                                    (long long)(itv))
 454 | #define __sanitizer_syscall_post_compat_50_getitimer(res, which, itv)          \
 455 |   __sanitizer_syscall_post_impl_compat_50_getitimer(res, (long long)(which),   \
 456 |                                                     (long long)(itv))
 457 | #define __sanitizer_syscall_pre_compat_43_ogethostname(hostname, len)          \
 458 |   __sanitizer_syscall_pre_impl_compat_43_ogethostname((long long)(hostname),   \
 459 |                                                       (long long)(len))
 460 | #define __sanitizer_syscall_post_compat_43_ogethostname(res, hostname, len)    \
```
- **Line 451 / 第 451 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 452 / 第 452 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 453 / 第 453 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 454 / 第 454 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 455 / 第 455 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 456 / 第 456 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 457 / 第 457 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 458 / 第 458 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 459 / 第 459 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 460 / 第 460 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 461-470 / 第 461-470 行
```cpp
 461 |   __sanitizer_syscall_post_impl_compat_43_ogethostname(                        \
 462 |       res, (long long)(hostname), (long long)(len))
 463 | #define __sanitizer_syscall_pre_compat_43_osethostname(hostname, len)          \
 464 |   __sanitizer_syscall_pre_impl_compat_43_osethostname((long long)(hostname),   \
 465 |                                                       (long long)(len))
 466 | #define __sanitizer_syscall_post_compat_43_osethostname(res, hostname, len)    \
 467 |   __sanitizer_syscall_post_impl_compat_43_osethostname(                        \
 468 |       res, (long long)(hostname), (long long)(len))
 469 | #define __sanitizer_syscall_pre_compat_43_ogetdtablesize()                     \
 470 |   __sanitizer_syscall_pre_impl_compat_43_ogetdtablesize()
```
- **Line 461 / 第 461 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 462 / 第 462 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 463 / 第 463 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 464 / 第 464 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 465 / 第 465 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 466 / 第 466 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 467 / 第 467 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 468 / 第 468 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 469 / 第 469 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 470 / 第 470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 471-480 / 第 471-480 行
```cpp
 471 | #define __sanitizer_syscall_post_compat_43_ogetdtablesize(res)                 \
 472 |   __sanitizer_syscall_post_impl_compat_43_ogetdtablesize(res)
 473 | #define __sanitizer_syscall_pre_dup2(from, to)                                 \
 474 |   __sanitizer_syscall_pre_impl_dup2((long long)(from), (long long)(to))
 475 | #define __sanitizer_syscall_post_dup2(res, from, to)                           \
 476 |   __sanitizer_syscall_post_impl_dup2(res, (long long)(from), (long long)(to))
 477 | #define __sanitizer_syscall_pre_getrandom(buf, buflen, flags)                  \
 478 |   __sanitizer_syscall_pre_impl_getrandom(                                      \
 479 |       (long long)(buf), (long long)(buflen), (long long)(flags))
 480 | #define __sanitizer_syscall_post_getrandom(res, buf, buflen, flags)            \
```
- **Line 471 / 第 471 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 472 / 第 472 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 473 / 第 473 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 474 / 第 474 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 475 / 第 475 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 476 / 第 476 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 477 / 第 477 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 478 / 第 478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 479 / 第 479 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 480 / 第 480 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 481-490 / 第 481-490 行
```cpp
 481 |   __sanitizer_syscall_post_impl_getrandom(                                     \
 482 |       res, (long long)(buf), (long long)(buflen), (long long)(flags))
 483 | #define __sanitizer_syscall_pre_fcntl(fd, cmd, arg)                            \
 484 |   __sanitizer_syscall_pre_impl_fcntl((long long)(fd), (long long)(cmd),        \
 485 |                                      (long long)(arg))
 486 | #define __sanitizer_syscall_post_fcntl(res, fd, cmd, arg)                      \
 487 |   __sanitizer_syscall_post_impl_fcntl(res, (long long)(fd), (long long)(cmd),  \
 488 |                                       (long long)(arg))
 489 | #define __sanitizer_syscall_pre_compat_50_select(nd, in, ou, ex, tv)           \
 490 |   __sanitizer_syscall_pre_impl_compat_50_select(                               \
```
- **Line 481 / 第 481 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 482 / 第 482 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 483 / 第 483 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 484 / 第 484 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 485 / 第 485 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 486 / 第 486 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 487 / 第 487 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 488 / 第 488 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 489 / 第 489 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 490 / 第 490 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 491-500 / 第 491-500 行
```cpp
 491 |       (long long)(nd), (long long)(in), (long long)(ou), (long long)(ex),      \
 492 |       (long long)(tv))
 493 | #define __sanitizer_syscall_post_compat_50_select(res, nd, in, ou, ex, tv)     \
 494 |   __sanitizer_syscall_post_impl_compat_50_select(                              \
 495 |       res, (long long)(nd), (long long)(in), (long long)(ou), (long long)(ex), \
 496 |       (long long)(tv))
 497 | /* syscall 94 has been skipped */
 498 | #define __sanitizer_syscall_pre_fsync(fd)                                      \
 499 |   __sanitizer_syscall_pre_impl_fsync((long long)(fd))
 500 | #define __sanitizer_syscall_post_fsync(res, fd)                                \
```
- **Line 491 / 第 491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 492 / 第 492 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 493 / 第 493 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 494 / 第 494 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 495 / 第 495 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 496 / 第 496 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 497 / 第 497 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 498 / 第 498 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 499 / 第 499 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 500 / 第 500 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 501-510 / 第 501-510 行
```cpp
 501 |   __sanitizer_syscall_post_impl_fsync(res, (long long)(fd))
 502 | #define __sanitizer_syscall_pre_setpriority(which, who, prio)                  \
 503 |   __sanitizer_syscall_pre_impl_setpriority(                                    \
 504 |       (long long)(which), (long long)(who), (long long)(prio))
 505 | #define __sanitizer_syscall_post_setpriority(res, which, who, prio)            \
 506 |   __sanitizer_syscall_post_impl_setpriority(                                   \
 507 |       res, (long long)(which), (long long)(who), (long long)(prio))
 508 | #define __sanitizer_syscall_pre_compat_30_socket(domain, type, protocol)       \
 509 |   __sanitizer_syscall_pre_impl_compat_30_socket(                               \
 510 |       (long long)(domain), (long long)(type), (long long)(protocol))
```
- **Line 501 / 第 501 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 502 / 第 502 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 503 / 第 503 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 504 / 第 504 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 505 / 第 505 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 506 / 第 506 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 507 / 第 507 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 508 / 第 508 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 509 / 第 509 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 510 / 第 510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 511-520 / 第 511-520 行
```cpp
 511 | #define __sanitizer_syscall_post_compat_30_socket(res, domain, type, protocol) \
 512 |   __sanitizer_syscall_post_impl_compat_30_socket(                              \
 513 |       res, (long long)(domain), (long long)(type), (long long)(protocol))
 514 | #define __sanitizer_syscall_pre_connect(s, name, namelen)                      \
 515 |   __sanitizer_syscall_pre_impl_connect((long long)(s), (long long)(name),      \
 516 |                                        (long long)(namelen))
 517 | #define __sanitizer_syscall_post_connect(res, s, name, namelen)                \
 518 |   __sanitizer_syscall_post_impl_connect(                                       \
 519 |       res, (long long)(s), (long long)(name), (long long)(namelen))
 520 | #define __sanitizer_syscall_pre_compat_43_oaccept(s, name, anamelen)           \
```
- **Line 511 / 第 511 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 512 / 第 512 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 513 / 第 513 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 514 / 第 514 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 515 / 第 515 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 516 / 第 516 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 517 / 第 517 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 518 / 第 518 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 519 / 第 519 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 520 / 第 520 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 521-530 / 第 521-530 行
```cpp
 521 |   __sanitizer_syscall_pre_impl_compat_43_oaccept(                              \
 522 |       (long long)(s), (long long)(name), (long long)(anamelen))
 523 | #define __sanitizer_syscall_post_compat_43_oaccept(res, s, name, anamelen)     \
 524 |   __sanitizer_syscall_post_impl_compat_43_oaccept(                             \
 525 |       res, (long long)(s), (long long)(name), (long long)(anamelen))
 526 | #define __sanitizer_syscall_pre_getpriority(which, who)                        \
 527 |   __sanitizer_syscall_pre_impl_getpriority((long long)(which), (long long)(who))
 528 | #define __sanitizer_syscall_post_getpriority(res, which, who)                  \
 529 |   __sanitizer_syscall_post_impl_getpriority(res, (long long)(which),           \
 530 |                                             (long long)(who))
```
- **Line 521 / 第 521 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 522 / 第 522 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 523 / 第 523 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 524 / 第 524 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 525 / 第 525 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 526 / 第 526 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 527 / 第 527 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 528 / 第 528 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 529 / 第 529 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 530 / 第 530 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 531-540 / 第 531-540 行
```cpp
 531 | #define __sanitizer_syscall_pre_compat_43_osend(s, buf, len, flags)            \
 532 |   __sanitizer_syscall_pre_impl_compat_43_osend(                                \
 533 |       (long long)(s), (long long)(buf), (long long)(len), (long long)(flags))
 534 | #define __sanitizer_syscall_post_compat_43_osend(res, s, buf, len, flags)      \
 535 |   __sanitizer_syscall_post_impl_compat_43_osend(                               \
 536 |       res, (long long)(s), (long long)(buf), (long long)(len),                 \
 537 |       (long long)(flags))
 538 | #define __sanitizer_syscall_pre_compat_43_orecv(s, buf, len, flags)            \
 539 |   __sanitizer_syscall_pre_impl_compat_43_orecv(                                \
 540 |       (long long)(s), (long long)(buf), (long long)(len), (long long)(flags))
```
- **Line 531 / 第 531 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 532 / 第 532 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 533 / 第 533 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 534 / 第 534 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 535 / 第 535 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 536 / 第 536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 537 / 第 537 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 538 / 第 538 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 539 / 第 539 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 540 / 第 540 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 541-550 / 第 541-550 行
```cpp
 541 | #define __sanitizer_syscall_post_compat_43_orecv(res, s, buf, len, flags)      \
 542 |   __sanitizer_syscall_post_impl_compat_43_orecv(                               \
 543 |       res, (long long)(s), (long long)(buf), (long long)(len),                 \
 544 |       (long long)(flags))
 545 | #define __sanitizer_syscall_pre_compat_13_sigreturn13(sigcntxp)                \
 546 |   __sanitizer_syscall_pre_impl_compat_13_sigreturn13((long long)(sigcntxp))
 547 | #define __sanitizer_syscall_post_compat_13_sigreturn13(res, sigcntxp)          \
 548 |   __sanitizer_syscall_post_impl_compat_13_sigreturn13(res,                     \
 549 |                                                       (long long)(sigcntxp))
 550 | #define __sanitizer_syscall_pre_bind(s, name, namelen)                         \
```
- **Line 541 / 第 541 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 542 / 第 542 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 543 / 第 543 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 544 / 第 544 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 545 / 第 545 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 546 / 第 546 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 547 / 第 547 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 548 / 第 548 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 549 / 第 549 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 550 / 第 550 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 551-560 / 第 551-560 行
```cpp
 551 |   __sanitizer_syscall_pre_impl_bind((long long)(s), (long long)(name),         \
 552 |                                     (long long)(namelen))
 553 | #define __sanitizer_syscall_post_bind(res, s, name, namelen)                   \
 554 |   __sanitizer_syscall_post_impl_bind(res, (long long)(s), (long long)(name),   \
 555 |                                      (long long)(namelen))
 556 | #define __sanitizer_syscall_pre_setsockopt(s, level, name, val, valsize)       \
 557 |   __sanitizer_syscall_pre_impl_setsockopt((long long)(s), (long long)(level),  \
 558 |                                           (long long)(name), (long long)(val), \
 559 |                                           (long long)(valsize))
 560 | #define __sanitizer_syscall_post_setsockopt(res, s, level, name, val, valsize) \
```
- **Line 551 / 第 551 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 552 / 第 552 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 553 / 第 553 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 554 / 第 554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 555 / 第 555 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 556 / 第 556 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 557 / 第 557 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 558 / 第 558 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 559 / 第 559 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 560 / 第 560 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 561-570 / 第 561-570 行
```cpp
 561 |   __sanitizer_syscall_post_impl_setsockopt(                                    \
 562 |       res, (long long)(s), (long long)(level), (long long)(name),              \
 563 |       (long long)(val), (long long)(valsize))
 564 | #define __sanitizer_syscall_pre_listen(s, backlog)                             \
 565 |   __sanitizer_syscall_pre_impl_listen((long long)(s), (long long)(backlog))
 566 | #define __sanitizer_syscall_post_listen(res, s, backlog)                       \
 567 |   __sanitizer_syscall_post_impl_listen(res, (long long)(s),                    \
 568 |                                        (long long)(backlog))
 569 | /* syscall 107 has been skipped */
 570 | #define __sanitizer_syscall_pre_compat_43_osigvec(signum, nsv, osv)            \
```
- **Line 561 / 第 561 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 562 / 第 562 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 563 / 第 563 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 564 / 第 564 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 565 / 第 565 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 566 / 第 566 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 567 / 第 567 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 568 / 第 568 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 569 / 第 569 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 570 / 第 570 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 571-580 / 第 571-580 行
```cpp
 571 |   __sanitizer_syscall_pre_impl_compat_43_osigvec(                              \
 572 |       (long long)(signum), (long long)(nsv), (long long)(osv))
 573 | #define __sanitizer_syscall_post_compat_43_osigvec(res, signum, nsv, osv)      \
 574 |   __sanitizer_syscall_post_impl_compat_43_osigvec(                             \
 575 |       res, (long long)(signum), (long long)(nsv), (long long)(osv))
 576 | #define __sanitizer_syscall_pre_compat_43_osigblock(mask)                      \
 577 |   __sanitizer_syscall_pre_impl_compat_43_osigblock((long long)(mask))
 578 | #define __sanitizer_syscall_post_compat_43_osigblock(res, mask)                \
 579 |   __sanitizer_syscall_post_impl_compat_43_osigblock(res, (long long)(mask))
 580 | #define __sanitizer_syscall_pre_compat_43_osigsetmask(mask)                    \
```
- **Line 571 / 第 571 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 572 / 第 572 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 573 / 第 573 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 574 / 第 574 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 575 / 第 575 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 576 / 第 576 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 577 / 第 577 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 578 / 第 578 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 579 / 第 579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 580 / 第 580 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 581-590 / 第 581-590 行
```cpp
 581 |   __sanitizer_syscall_pre_impl_compat_43_osigsetmask((long long)(mask))
 582 | #define __sanitizer_syscall_post_compat_43_osigsetmask(res, mask)              \
 583 |   __sanitizer_syscall_post_impl_compat_43_osigsetmask(res, (long long)(mask))
 584 | #define __sanitizer_syscall_pre_compat_13_sigsuspend13(mask)                   \
 585 |   __sanitizer_syscall_pre_impl_compat_13_sigsuspend13((long long)(mask))
 586 | #define __sanitizer_syscall_post_compat_13_sigsuspend13(res, mask)             \
 587 |   __sanitizer_syscall_post_impl_compat_13_sigsuspend13(res, (long long)(mask))
 588 | #define __sanitizer_syscall_pre_compat_43_osigstack(nss, oss)                  \
 589 |   __sanitizer_syscall_pre_impl_compat_43_osigstack((long long)(nss),           \
 590 |                                                    (long long)(oss))
```
- **Line 581 / 第 581 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 582 / 第 582 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 583 / 第 583 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 584 / 第 584 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 585 / 第 585 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 586 / 第 586 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 587 / 第 587 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 588 / 第 588 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 589 / 第 589 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 590 / 第 590 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 591-600 / 第 591-600 行
```cpp
 591 | #define __sanitizer_syscall_post_compat_43_osigstack(res, nss, oss)            \
 592 |   __sanitizer_syscall_post_impl_compat_43_osigstack(res, (long long)(nss),     \
 593 |                                                     (long long)(oss))
 594 | #define __sanitizer_syscall_pre_compat_43_orecvmsg(s, msg, flags)              \
 595 |   __sanitizer_syscall_pre_impl_compat_43_orecvmsg(                             \
 596 |       (long long)(s), (long long)(msg), (long long)(flags))
 597 | #define __sanitizer_syscall_post_compat_43_orecvmsg(res, s, msg, flags)        \
 598 |   __sanitizer_syscall_post_impl_compat_43_orecvmsg(                            \
 599 |       res, (long long)(s), (long long)(msg), (long long)(flags))
 600 | #define __sanitizer_syscall_pre_compat_43_osendmsg(s, msg, flags)              \
```
- **Line 591 / 第 591 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 592 / 第 592 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 593 / 第 593 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 594 / 第 594 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 595 / 第 595 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 596 / 第 596 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 597 / 第 597 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 598 / 第 598 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 599 / 第 599 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 600 / 第 600 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 601-610 / 第 601-610 行
```cpp
 601 |   __sanitizer_syscall_pre_impl_compat_43_osendmsg(                             \
 602 |       (long long)(s), (long long)(msg), (long long)(flags))
 603 | #define __sanitizer_syscall_post_compat_43_osendmsg(res, s, msg, flags)        \
 604 |   __sanitizer_syscall_post_impl_compat_43_osendmsg(                            \
 605 |       res, (long long)(s), (long long)(msg), (long long)(flags))
 606 | /* syscall 115 has been skipped */
 607 | #define __sanitizer_syscall_pre_compat_50_gettimeofday(tp, tzp)                \
 608 |   __sanitizer_syscall_pre_impl_compat_50_gettimeofday((long long)(tp),         \
 609 |                                                       (long long)(tzp))
 610 | #define __sanitizer_syscall_post_compat_50_gettimeofday(res, tp, tzp)          \
```
- **Line 601 / 第 601 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 602 / 第 602 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 603 / 第 603 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 604 / 第 604 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 605 / 第 605 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 606 / 第 606 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 607 / 第 607 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 608 / 第 608 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 609 / 第 609 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 610 / 第 610 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 611-620 / 第 611-620 行
```cpp
 611 |   __sanitizer_syscall_post_impl_compat_50_gettimeofday(res, (long long)(tp),   \
 612 |                                                        (long long)(tzp))
 613 | #define __sanitizer_syscall_pre_compat_50_getrusage(who, rusage)               \
 614 |   __sanitizer_syscall_pre_impl_compat_50_getrusage((long long)(who),           \
 615 |                                                    (long long)(rusage))
 616 | #define __sanitizer_syscall_post_compat_50_getrusage(res, who, rusage)         \
 617 |   __sanitizer_syscall_post_impl_compat_50_getrusage(res, (long long)(who),     \
 618 |                                                     (long long)(rusage))
 619 | #define __sanitizer_syscall_pre_getsockopt(s, level, name, val, avalsize)      \
 620 |   __sanitizer_syscall_pre_impl_getsockopt((long long)(s), (long long)(level),  \
```
- **Line 611 / 第 611 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 612 / 第 612 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 613 / 第 613 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 614 / 第 614 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 615 / 第 615 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 616 / 第 616 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 617 / 第 617 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 618 / 第 618 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 619 / 第 619 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 620 / 第 620 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 621-630 / 第 621-630 行
```cpp
 621 |                                           (long long)(name), (long long)(val), \
 622 |                                           (long long)(avalsize))
 623 | #define __sanitizer_syscall_post_getsockopt(res, s, level, name, val,          \
 624 |                                             avalsize)                          \
 625 |   __sanitizer_syscall_post_impl_getsockopt(                                    \
 626 |       res, (long long)(s), (long long)(level), (long long)(name),              \
 627 |       (long long)(val), (long long)(avalsize))
 628 | /* syscall 119 has been skipped */
 629 | #define __sanitizer_syscall_pre_readv(fd, iovp, iovcnt)                        \
 630 |   __sanitizer_syscall_pre_impl_readv((long long)(fd), (long long)(iovp),       \
```
- **Line 621 / 第 621 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 622 / 第 622 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 623 / 第 623 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 624 / 第 624 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 625 / 第 625 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 626 / 第 626 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 627 / 第 627 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 628 / 第 628 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 629 / 第 629 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 630 / 第 630 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 631-640 / 第 631-640 行
```cpp
 631 |                                      (long long)(iovcnt))
 632 | #define __sanitizer_syscall_post_readv(res, fd, iovp, iovcnt)                  \
 633 |   __sanitizer_syscall_post_impl_readv(res, (long long)(fd), (long long)(iovp), \
 634 |                                       (long long)(iovcnt))
 635 | #define __sanitizer_syscall_pre_writev(fd, iovp, iovcnt)                       \
 636 |   __sanitizer_syscall_pre_impl_writev((long long)(fd), (long long)(iovp),      \
 637 |                                       (long long)(iovcnt))
 638 | #define __sanitizer_syscall_post_writev(res, fd, iovp, iovcnt)                 \
 639 |   __sanitizer_syscall_post_impl_writev(res, (long long)(fd),                   \
 640 |                                        (long long)(iovp), (long long)(iovcnt))
```
- **Line 631 / 第 631 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 632 / 第 632 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 633 / 第 633 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 634 / 第 634 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 635 / 第 635 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 636 / 第 636 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 637 / 第 637 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 638 / 第 638 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 639 / 第 639 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 640 / 第 640 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 641-650 / 第 641-650 行
```cpp
 641 | #define __sanitizer_syscall_pre_compat_50_settimeofday(tv, tzp)                \
 642 |   __sanitizer_syscall_pre_impl_compat_50_settimeofday((long long)(tv),         \
 643 |                                                       (long long)(tzp))
 644 | #define __sanitizer_syscall_post_compat_50_settimeofday(res, tv, tzp)          \
 645 |   __sanitizer_syscall_post_impl_compat_50_settimeofday(res, (long long)(tv),   \
 646 |                                                        (long long)(tzp))
 647 | #define __sanitizer_syscall_pre_fchown(fd, uid, gid)                           \
 648 |   __sanitizer_syscall_pre_impl_fchown((long long)(fd), (long long)(uid),       \
 649 |                                       (long long)(gid))
 650 | #define __sanitizer_syscall_post_fchown(res, fd, uid, gid)                     \
```
- **Line 641 / 第 641 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 642 / 第 642 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 643 / 第 643 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 644 / 第 644 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 645 / 第 645 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 646 / 第 646 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 647 / 第 647 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 648 / 第 648 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 649 / 第 649 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 650 / 第 650 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 651-660 / 第 651-660 行
```cpp
 651 |   __sanitizer_syscall_post_impl_fchown(res, (long long)(fd), (long long)(uid), \
 652 |                                        (long long)(gid))
 653 | #define __sanitizer_syscall_pre_fchmod(fd, mode)                               \
 654 |   __sanitizer_syscall_pre_impl_fchmod((long long)(fd), (long long)(mode))
 655 | #define __sanitizer_syscall_post_fchmod(res, fd, mode)                         \
 656 |   __sanitizer_syscall_post_impl_fchmod(res, (long long)(fd), (long long)(mode))
 657 | #define __sanitizer_syscall_pre_compat_43_orecvfrom(s, buf, len, flags, from,  \
 658 |                                                     fromlenaddr)               \
 659 |   __sanitizer_syscall_pre_impl_compat_43_orecvfrom(                            \
 660 |       (long long)(s), (long long)(buf), (long long)(len), (long long)(flags),  \
```
- **Line 651 / 第 651 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 652 / 第 652 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 653 / 第 653 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 654 / 第 654 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 655 / 第 655 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 656 / 第 656 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 657 / 第 657 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 658 / 第 658 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 659 / 第 659 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 660 / 第 660 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 661-670 / 第 661-670 行
```cpp
 661 |       (long long)(from), (long long)(fromlenaddr))
 662 | #define __sanitizer_syscall_post_compat_43_orecvfrom(res, s, buf, len, flags,  \
 663 |                                                      from, fromlenaddr)        \
 664 |   __sanitizer_syscall_post_impl_compat_43_orecvfrom(                           \
 665 |       res, (long long)(s), (long long)(buf), (long long)(len),                 \
 666 |       (long long)(flags), (long long)(from), (long long)(fromlenaddr))
 667 | #define __sanitizer_syscall_pre_setreuid(ruid, euid)                           \
 668 |   __sanitizer_syscall_pre_impl_setreuid((long long)(ruid), (long long)(euid))
 669 | #define __sanitizer_syscall_post_setreuid(res, ruid, euid)                     \
 670 |   __sanitizer_syscall_post_impl_setreuid(res, (long long)(ruid),               \
```
- **Line 661 / 第 661 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 662 / 第 662 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 663 / 第 663 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 664 / 第 664 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 665 / 第 665 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 666 / 第 666 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 667 / 第 667 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 668 / 第 668 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 669 / 第 669 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 670 / 第 670 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 671-680 / 第 671-680 行
```cpp
 671 |                                          (long long)(euid))
 672 | #define __sanitizer_syscall_pre_setregid(rgid, egid)                           \
 673 |   __sanitizer_syscall_pre_impl_setregid((long long)(rgid), (long long)(egid))
 674 | #define __sanitizer_syscall_post_setregid(res, rgid, egid)                     \
 675 |   __sanitizer_syscall_post_impl_setregid(res, (long long)(rgid),               \
 676 |                                          (long long)(egid))
 677 | #define __sanitizer_syscall_pre_rename(from, to)                               \
 678 |   __sanitizer_syscall_pre_impl_rename((long long)(from), (long long)(to))
 679 | #define __sanitizer_syscall_post_rename(res, from, to)                         \
 680 |   __sanitizer_syscall_post_impl_rename(res, (long long)(from), (long long)(to))
```
- **Line 671 / 第 671 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 672 / 第 672 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 673 / 第 673 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 674 / 第 674 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 675 / 第 675 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 676 / 第 676 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 677 / 第 677 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 678 / 第 678 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 679 / 第 679 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 680 / 第 680 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 681-690 / 第 681-690 行
```cpp
 681 | #define __sanitizer_syscall_pre_compat_43_otruncate(path, length)              \
 682 |   __sanitizer_syscall_pre_impl_compat_43_otruncate((long long)(path),          \
 683 |                                                    (long long)(length))
 684 | #define __sanitizer_syscall_post_compat_43_otruncate(res, path, length)        \
 685 |   __sanitizer_syscall_post_impl_compat_43_otruncate(res, (long long)(path),    \
 686 |                                                     (long long)(length))
 687 | #define __sanitizer_syscall_pre_compat_43_oftruncate(fd, length)               \
 688 |   __sanitizer_syscall_pre_impl_compat_43_oftruncate((long long)(fd),           \
 689 |                                                     (long long)(length))
 690 | #define __sanitizer_syscall_post_compat_43_oftruncate(res, fd, length)         \
```
- **Line 681 / 第 681 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 682 / 第 682 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 683 / 第 683 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 684 / 第 684 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 685 / 第 685 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 686 / 第 686 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 687 / 第 687 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 688 / 第 688 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 689 / 第 689 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 690 / 第 690 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 691-700 / 第 691-700 行
```cpp
 691 |   __sanitizer_syscall_post_impl_compat_43_oftruncate(res, (long long)(fd),     \
 692 |                                                      (long long)(length))
 693 | #define __sanitizer_syscall_pre_flock(fd, how)                                 \
 694 |   __sanitizer_syscall_pre_impl_flock((long long)(fd), (long long)(how))
 695 | #define __sanitizer_syscall_post_flock(res, fd, how)                           \
 696 |   __sanitizer_syscall_post_impl_flock(res, (long long)(fd), (long long)(how))
 697 | #define __sanitizer_syscall_pre_mkfifo(path, mode)                             \
 698 |   __sanitizer_syscall_pre_impl_mkfifo((long long)(path), (long long)(mode))
 699 | #define __sanitizer_syscall_post_mkfifo(res, path, mode)                       \
 700 |   __sanitizer_syscall_post_impl_mkfifo(res, (long long)(path),                 \
```
- **Line 691 / 第 691 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
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
 701 |                                        (long long)(mode))
 702 | #define __sanitizer_syscall_pre_sendto(s, buf, len, flags, to, tolen)          \
 703 |   __sanitizer_syscall_pre_impl_sendto((long long)(s), (long long)(buf),        \
 704 |                                       (long long)(len), (long long)(flags),    \
 705 |                                       (long long)(to), (long long)(tolen))
 706 | #define __sanitizer_syscall_post_sendto(res, s, buf, len, flags, to, tolen)    \
 707 |   __sanitizer_syscall_post_impl_sendto(res, (long long)(s), (long long)(buf),  \
 708 |                                        (long long)(len), (long long)(flags),   \
 709 |                                        (long long)(to), (long long)(tolen))
 710 | #define __sanitizer_syscall_pre_shutdown(s, how)                               \
```
- **Line 701 / 第 701 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 702 / 第 702 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 703 / 第 703 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 704 / 第 704 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 705 / 第 705 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 706 / 第 706 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 707 / 第 707 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 708 / 第 708 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 709 / 第 709 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 710 / 第 710 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 711-720 / 第 711-720 行
```cpp
 711 |   __sanitizer_syscall_pre_impl_shutdown((long long)(s), (long long)(how))
 712 | #define __sanitizer_syscall_post_shutdown(res, s, how)                         \
 713 |   __sanitizer_syscall_post_impl_shutdown(res, (long long)(s), (long long)(how))
 714 | #define __sanitizer_syscall_pre_socketpair(domain, type, protocol, rsv)        \
 715 |   __sanitizer_syscall_pre_impl_socketpair(                                     \
 716 |       (long long)(domain), (long long)(type), (long long)(protocol),           \
 717 |       (long long)(rsv))
 718 | #define __sanitizer_syscall_post_socketpair(res, domain, type, protocol, rsv)  \
 719 |   __sanitizer_syscall_post_impl_socketpair(                                    \
 720 |       res, (long long)(domain), (long long)(type), (long long)(protocol),      \
```
- **Line 711 / 第 711 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 712 / 第 712 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 713 / 第 713 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 714 / 第 714 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 715 / 第 715 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 716 / 第 716 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 717 / 第 717 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 718 / 第 718 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 719 / 第 719 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 720 / 第 720 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 721-730 / 第 721-730 行
```cpp
 721 |       (long long)(rsv))
 722 | #define __sanitizer_syscall_pre_mkdir(path, mode)                              \
 723 |   __sanitizer_syscall_pre_impl_mkdir((long long)(path), (long long)(mode))
 724 | #define __sanitizer_syscall_post_mkdir(res, path, mode)                        \
 725 |   __sanitizer_syscall_post_impl_mkdir(res, (long long)(path), (long long)(mode))
 726 | #define __sanitizer_syscall_pre_rmdir(path)                                    \
 727 |   __sanitizer_syscall_pre_impl_rmdir((long long)(path))
 728 | #define __sanitizer_syscall_post_rmdir(res, path)                              \
 729 |   __sanitizer_syscall_post_impl_rmdir(res, (long long)(path))
 730 | #define __sanitizer_syscall_pre_compat_50_utimes(path, tptr)                   \
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
 731 |   __sanitizer_syscall_pre_impl_compat_50_utimes((long long)(path),             \
 732 |                                                 (long long)(tptr))
 733 | #define __sanitizer_syscall_post_compat_50_utimes(res, path, tptr)             \
 734 |   __sanitizer_syscall_post_impl_compat_50_utimes(res, (long long)(path),       \
 735 |                                                  (long long)(tptr))
 736 | /* syscall 139 has been skipped */
 737 | #define __sanitizer_syscall_pre_compat_50_adjtime(delta, olddelta)             \
 738 |   __sanitizer_syscall_pre_impl_compat_50_adjtime((long long)(delta),           \
 739 |                                                  (long long)(olddelta))
 740 | #define __sanitizer_syscall_post_compat_50_adjtime(res, delta, olddelta)       \
```
- **Line 731 / 第 731 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 732 / 第 732 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 733 / 第 733 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 734 / 第 734 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 735 / 第 735 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 736 / 第 736 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 737 / 第 737 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 738 / 第 738 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 739 / 第 739 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 740 / 第 740 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 741-750 / 第 741-750 行
```cpp
 741 |   __sanitizer_syscall_post_impl_compat_50_adjtime(res, (long long)(delta),     \
 742 |                                                   (long long)(olddelta))
 743 | #define __sanitizer_syscall_pre_compat_43_ogetpeername(fdes, asa, alen)        \
 744 |   __sanitizer_syscall_pre_impl_compat_43_ogetpeername(                         \
 745 |       (long long)(fdes), (long long)(asa), (long long)(alen))
 746 | #define __sanitizer_syscall_post_compat_43_ogetpeername(res, fdes, asa, alen)  \
 747 |   __sanitizer_syscall_post_impl_compat_43_ogetpeername(                        \
 748 |       res, (long long)(fdes), (long long)(asa), (long long)(alen))
 749 | #define __sanitizer_syscall_pre_compat_43_ogethostid()                         \
 750 |   __sanitizer_syscall_pre_impl_compat_43_ogethostid()
```
- **Line 741 / 第 741 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 742 / 第 742 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 743 / 第 743 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 744 / 第 744 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 745 / 第 745 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 746 / 第 746 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 747 / 第 747 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 748 / 第 748 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 749 / 第 749 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 750 / 第 750 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 751-760 / 第 751-760 行
```cpp
 751 | #define __sanitizer_syscall_post_compat_43_ogethostid(res)                     \
 752 |   __sanitizer_syscall_post_impl_compat_43_ogethostid(res)
 753 | #define __sanitizer_syscall_pre_compat_43_osethostid(hostid)                   \
 754 |   __sanitizer_syscall_pre_impl_compat_43_osethostid((long long)(hostid))
 755 | #define __sanitizer_syscall_post_compat_43_osethostid(res, hostid)             \
 756 |   __sanitizer_syscall_post_impl_compat_43_osethostid(res, (long long)(hostid))
 757 | #define __sanitizer_syscall_pre_compat_43_ogetrlimit(which, rlp)               \
 758 |   __sanitizer_syscall_pre_impl_compat_43_ogetrlimit((long long)(which),        \
 759 |                                                     (long long)(rlp))
 760 | #define __sanitizer_syscall_post_compat_43_ogetrlimit(res, which, rlp)         \
```
- **Line 751 / 第 751 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 752 / 第 752 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 753 / 第 753 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 754 / 第 754 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 755 / 第 755 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 756 / 第 756 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 757 / 第 757 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 758 / 第 758 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 759 / 第 759 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 760 / 第 760 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 761-770 / 第 761-770 行
```cpp
 761 |   __sanitizer_syscall_post_impl_compat_43_ogetrlimit(res, (long long)(which),  \
 762 |                                                      (long long)(rlp))
 763 | #define __sanitizer_syscall_pre_compat_43_osetrlimit(which, rlp)               \
 764 |   __sanitizer_syscall_pre_impl_compat_43_osetrlimit((long long)(which),        \
 765 |                                                     (long long)(rlp))
 766 | #define __sanitizer_syscall_post_compat_43_osetrlimit(res, which, rlp)         \
 767 |   __sanitizer_syscall_post_impl_compat_43_osetrlimit(res, (long long)(which),  \
 768 |                                                      (long long)(rlp))
 769 | #define __sanitizer_syscall_pre_compat_43_okillpg(pgid, signum)                \
 770 |   __sanitizer_syscall_pre_impl_compat_43_okillpg((long long)(pgid),            \
```
- **Line 761 / 第 761 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 762 / 第 762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 763 / 第 763 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 764 / 第 764 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 765 / 第 765 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 766 / 第 766 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 767 / 第 767 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 768 / 第 768 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 769 / 第 769 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 770 / 第 770 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 771-780 / 第 771-780 行
```cpp
 771 |                                                  (long long)(signum))
 772 | #define __sanitizer_syscall_post_compat_43_okillpg(res, pgid, signum)          \
 773 |   __sanitizer_syscall_post_impl_compat_43_okillpg(res, (long long)(pgid),      \
 774 |                                                   (long long)(signum))
 775 | #define __sanitizer_syscall_pre_setsid() __sanitizer_syscall_pre_impl_setsid()
 776 | #define __sanitizer_syscall_post_setsid(res)                                   \
 777 |   __sanitizer_syscall_post_impl_setsid(res)
 778 | #define __sanitizer_syscall_pre_compat_50_quotactl(path, cmd, uid, arg)        \
 779 |   __sanitizer_syscall_pre_impl_compat_50_quotactl(                             \
 780 |       (long long)(path), (long long)(cmd), (long long)(uid), (long long)(arg))
```
- **Line 771 / 第 771 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 772 / 第 772 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 773 / 第 773 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 774 / 第 774 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 775 / 第 775 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 776 / 第 776 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 777 / 第 777 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 778 / 第 778 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 779 / 第 779 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 780 / 第 780 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 781-790 / 第 781-790 行
```cpp
 781 | #define __sanitizer_syscall_post_compat_50_quotactl(res, path, cmd, uid, arg)  \
 782 |   __sanitizer_syscall_post_impl_compat_50_quotactl(                            \
 783 |       res, (long long)(path), (long long)(cmd), (long long)(uid),              \
 784 |       (long long)(arg))
 785 | #define __sanitizer_syscall_pre_compat_43_oquota()                             \
 786 |   __sanitizer_syscall_pre_impl_compat_43_oquota()
 787 | #define __sanitizer_syscall_post_compat_43_oquota(res)                         \
 788 |   __sanitizer_syscall_post_impl_compat_43_oquota(res)
 789 | #define __sanitizer_syscall_pre_compat_43_ogetsockname(fdec, asa, alen)        \
 790 |   __sanitizer_syscall_pre_impl_compat_43_ogetsockname(                         \
```
- **Line 781 / 第 781 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 782 / 第 782 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 783 / 第 783 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 784 / 第 784 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 785 / 第 785 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 786 / 第 786 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 787 / 第 787 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 788 / 第 788 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 789 / 第 789 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 790 / 第 790 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 791-800 / 第 791-800 行
```cpp
 791 |       (long long)(fdec), (long long)(asa), (long long)(alen))
 792 | #define __sanitizer_syscall_post_compat_43_ogetsockname(res, fdec, asa, alen)  \
 793 |   __sanitizer_syscall_post_impl_compat_43_ogetsockname(                        \
 794 |       res, (long long)(fdec), (long long)(asa), (long long)(alen))
 795 | /* syscall 151 has been skipped */
 796 | /* syscall 152 has been skipped */
 797 | /* syscall 153 has been skipped */
 798 | /* syscall 154 has been skipped */
 799 | #define __sanitizer_syscall_pre_nfssvc(flag, argp)                             \
 800 |   __sanitizer_syscall_pre_impl_nfssvc((long long)(flag), (long long)(argp))
```
- **Line 791 / 第 791 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 792 / 第 792 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 793 / 第 793 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 794 / 第 794 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 795 / 第 795 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 796 / 第 796 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 797 / 第 797 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 798 / 第 798 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 799 / 第 799 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 800 / 第 800 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 801-810 / 第 801-810 行
```cpp
 801 | #define __sanitizer_syscall_post_nfssvc(res, flag, argp)                       \
 802 |   __sanitizer_syscall_post_impl_nfssvc(res, (long long)(flag),                 \
 803 |                                        (long long)(argp))
 804 | #define __sanitizer_syscall_pre_compat_43_ogetdirentries(fd, buf, count,       \
 805 |                                                          basep)                \
 806 |   __sanitizer_syscall_pre_impl_compat_43_ogetdirentries(                       \
 807 |       (long long)(fd), (long long)(buf), (long long)(count),                   \
 808 |       (long long)(basep))
 809 | #define __sanitizer_syscall_post_compat_43_ogetdirentries(res, fd, buf, count, \
 810 |                                                           basep)               \
```
- **Line 801 / 第 801 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 802 / 第 802 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 803 / 第 803 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 804 / 第 804 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 805 / 第 805 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 806 / 第 806 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 807 / 第 807 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 808 / 第 808 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 809 / 第 809 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 810 / 第 810 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 811-820 / 第 811-820 行
```cpp
 811 |   __sanitizer_syscall_post_impl_compat_43_ogetdirentries(                      \
 812 |       res, (long long)(fd), (long long)(buf), (long long)(count),              \
 813 |       (long long)(basep))
 814 | #define __sanitizer_syscall_pre_compat_20_statfs(path, buf)                    \
 815 |   __sanitizer_syscall_pre_impl_compat_20_statfs((long long)(path),             \
 816 |                                                 (long long)(buf))
 817 | #define __sanitizer_syscall_post_compat_20_statfs(res, path, buf)              \
 818 |   __sanitizer_syscall_post_impl_compat_20_statfs(res, (long long)(path),       \
 819 |                                                  (long long)(buf))
 820 | #define __sanitizer_syscall_pre_compat_20_fstatfs(fd, buf)                     \
```
- **Line 811 / 第 811 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 812 / 第 812 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 813 / 第 813 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 814 / 第 814 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 815 / 第 815 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 816 / 第 816 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 817 / 第 817 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 818 / 第 818 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 819 / 第 819 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 820 / 第 820 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 821-830 / 第 821-830 行
```cpp
 821 |   __sanitizer_syscall_pre_impl_compat_20_fstatfs((long long)(fd),              \
 822 |                                                  (long long)(buf))
 823 | #define __sanitizer_syscall_post_compat_20_fstatfs(res, fd, buf)               \
 824 |   __sanitizer_syscall_post_impl_compat_20_fstatfs(res, (long long)(fd),        \
 825 |                                                   (long long)(buf))
 826 | /* syscall 159 has been skipped */
 827 | /* syscall 160 has been skipped */
 828 | #define __sanitizer_syscall_pre_compat_30_getfh(fname, fhp)                    \
 829 |   __sanitizer_syscall_pre_impl_compat_30_getfh((long long)(fname),             \
 830 |                                                (long long)(fhp))
```
- **Line 821 / 第 821 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 822 / 第 822 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 823 / 第 823 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 824 / 第 824 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 825 / 第 825 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 826 / 第 826 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 827 / 第 827 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 828 / 第 828 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 829 / 第 829 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 830 / 第 830 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 831-840 / 第 831-840 行
```cpp
 831 | #define __sanitizer_syscall_post_compat_30_getfh(res, fname, fhp)              \
 832 |   __sanitizer_syscall_post_impl_compat_30_getfh(res, (long long)(fname),       \
 833 |                                                 (long long)(fhp))
 834 | #define __sanitizer_syscall_pre_compat_09_ogetdomainname(domainname, len)      \
 835 |   __sanitizer_syscall_pre_impl_compat_09_ogetdomainname(                       \
 836 |       (long long)(domainname), (long long)(len))
 837 | #define __sanitizer_syscall_post_compat_09_ogetdomainname(res, domainname,     \
 838 |                                                           len)                 \
 839 |   __sanitizer_syscall_post_impl_compat_09_ogetdomainname(                      \
 840 |       res, (long long)(domainname), (long long)(len))
```
- **Line 831 / 第 831 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 832 / 第 832 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 833 / 第 833 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 834 / 第 834 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 835 / 第 835 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 836 / 第 836 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 837 / 第 837 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 838 / 第 838 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 839 / 第 839 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 840 / 第 840 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 841-850 / 第 841-850 行
```cpp
 841 | #define __sanitizer_syscall_pre_compat_09_osetdomainname(domainname, len)      \
 842 |   __sanitizer_syscall_pre_impl_compat_09_osetdomainname(                       \
 843 |       (long long)(domainname), (long long)(len))
 844 | #define __sanitizer_syscall_post_compat_09_osetdomainname(res, domainname,     \
 845 |                                                           len)                 \
 846 |   __sanitizer_syscall_post_impl_compat_09_osetdomainname(                      \
 847 |       res, (long long)(domainname), (long long)(len))
 848 | #define __sanitizer_syscall_pre_compat_09_ouname(name)                         \
 849 |   __sanitizer_syscall_pre_impl_compat_09_ouname((long long)(name))
 850 | #define __sanitizer_syscall_post_compat_09_ouname(res, name)                   \
```
- **Line 841 / 第 841 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 842 / 第 842 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 843 / 第 843 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 844 / 第 844 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 845 / 第 845 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 846 / 第 846 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 847 / 第 847 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 848 / 第 848 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 849 / 第 849 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 850 / 第 850 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 851-860 / 第 851-860 行
```cpp
 851 |   __sanitizer_syscall_post_impl_compat_09_ouname(res, (long long)(name))
 852 | #define __sanitizer_syscall_pre_sysarch(op, parms)                             \
 853 |   __sanitizer_syscall_pre_impl_sysarch((long long)(op), (long long)(parms))
 854 | #define __sanitizer_syscall_post_sysarch(res, op, parms)                       \
 855 |   __sanitizer_syscall_post_impl_sysarch(res, (long long)(op),                  \
 856 |                                         (long long)(parms))
 857 | #define __sanitizer_syscall_pre___futex(uaddr, op, val, timeout, uaddr2, val2, \
 858 |                                         val3)                                  \
 859 |   __sanitizer_syscall_pre_impl___futex((long long)(uaddr), (long long)(op),    \
 860 |                                        (long long)(val), (long long)(timeout), \
```
- **Line 851 / 第 851 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 852 / 第 852 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 853 / 第 853 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 854 / 第 854 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 855 / 第 855 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 856 / 第 856 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 857 / 第 857 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 858 / 第 858 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 859 / 第 859 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 860 / 第 860 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 861-870 / 第 861-870 行
```cpp
 861 |                                        (long long)(uaddr2), (long long)(val2), \
 862 |                                        (long long)(val3))
 863 | #define __sanitizer_syscall_post___futex(res, uaddr, op, val, timeout, uaddr2, \
 864 |                                          val2, val3)                           \
 865 |   __sanitizer_syscall_post_impl___futex(                                       \
 866 |       res, (long long)(uaddr), (long long)(op), (long long)(val),              \
 867 |       (long long)(timeout), (long long)(uaddr2), (long long)(val2),            \
 868 |       (long long)(val3))
 869 | #define __sanitizer_syscall_pre___futex_set_robust_list(head, len)             \
 870 |   __sanitizer_syscall_pre_impl___futex_set_robust_list((long long)(head),      \
```
- **Line 861 / 第 861 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 862 / 第 862 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 863 / 第 863 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 864 / 第 864 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 865 / 第 865 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 866 / 第 866 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 867 / 第 867 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 868 / 第 868 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 869 / 第 869 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 870 / 第 870 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 871-880 / 第 871-880 行
```cpp
 871 |                                                        (long long)(len))
 872 | #define __sanitizer_syscall_post___futex_set_robust_list(res, head, len)       \
 873 |   __sanitizer_syscall_post_impl___futex_set_robust_list(                       \
 874 |       res, (long long)(head), (long long)(len))
 875 | #define __sanitizer_syscall_pre___futex_get_robust_list(lwpid, headp, lenp)    \
 876 |   __sanitizer_syscall_pre_impl___futex_get_robust_list(                        \
 877 |       (long long)(lwpid), (long long)(headp), (long long)(lenp))
 878 | #define __sanitizer_syscall_post___futex_get_robust_list(res, lwpid, headp,    \
 879 |                                                          lenp)                 \
 880 |   __sanitizer_syscall_post_impl___futex_get_robust_list(                       \
```
- **Line 871 / 第 871 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 872 / 第 872 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 873 / 第 873 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 874 / 第 874 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 875 / 第 875 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 876 / 第 876 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 877 / 第 877 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 878 / 第 878 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 879 / 第 879 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 880 / 第 880 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 881-890 / 第 881-890 行
```cpp
 881 |       res, (long long)(lwpid), (long long)(headp), (long long)(lenp))
 882 | #if !defined(_LP64)
 883 | #define __sanitizer_syscall_pre_compat_10_osemsys(which, a2, a3, a4, a5)       \
 884 |   __sanitizer_syscall_pre_impl_compat_10_osemsys(                              \
 885 |       (long long)(which), (long long)(a2), (long long)(a3), (long long)(a4),   \
 886 |       (long long)(a5))
 887 | #define __sanitizer_syscall_post_compat_10_osemsys(res, which, a2, a3, a4, a5) \
 888 |   __sanitizer_syscall_post_impl_compat_10_osemsys(                             \
 889 |       res, (long long)(which), (long long)(a2), (long long)(a3),               \
 890 |       (long long)(a4), (long long)(a5))
```
- **Line 881 / 第 881 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 882 / 第 882 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 883 / 第 883 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 884 / 第 884 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 885 / 第 885 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 886 / 第 886 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 887 / 第 887 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 888 / 第 888 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 889 / 第 889 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 890 / 第 890 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 891-900 / 第 891-900 行
```cpp
 891 | #else
 892 | /* syscall 169 has been skipped */
 893 | #endif
 894 | #if !defined(_LP64)
 895 | #define __sanitizer_syscall_pre_compat_10_omsgsys(which, a2, a3, a4, a5, a6)   \
 896 |   __sanitizer_syscall_pre_impl_compat_10_omsgsys(                              \
 897 |       (long long)(which), (long long)(a2), (long long)(a3), (long long)(a4),   \
 898 |       (long long)(a5), (long long)(a6))
 899 | #define __sanitizer_syscall_post_compat_10_omsgsys(res, which, a2, a3, a4, a5, \
 900 |                                                    a6)                         \
```
- **Line 891 / 第 891 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 892 / 第 892 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 893 / 第 893 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 894 / 第 894 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 895 / 第 895 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 896 / 第 896 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 897 / 第 897 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 898 / 第 898 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 899 / 第 899 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 900 / 第 900 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 901-910 / 第 901-910 行
```cpp
 901 |   __sanitizer_syscall_post_impl_compat_10_omsgsys(                             \
 902 |       res, (long long)(which), (long long)(a2), (long long)(a3),               \
 903 |       (long long)(a4), (long long)(a5), (long long)(a6))
 904 | #else
 905 | /* syscall 170 has been skipped */
 906 | #endif
 907 | #if !defined(_LP64)
 908 | #define __sanitizer_syscall_pre_compat_10_oshmsys(which, a2, a3, a4)           \
 909 |   __sanitizer_syscall_pre_impl_compat_10_oshmsys(                              \
 910 |       (long long)(which), (long long)(a2), (long long)(a3), (long long)(a4))
```
- **Line 901 / 第 901 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 902 / 第 902 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 903 / 第 903 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 904 / 第 904 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 905 / 第 905 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 906 / 第 906 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 907 / 第 907 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 908 / 第 908 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 909 / 第 909 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 910 / 第 910 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 911-920 / 第 911-920 行
```cpp
 911 | #define __sanitizer_syscall_post_compat_10_oshmsys(res, which, a2, a3, a4)     \
 912 |   __sanitizer_syscall_post_impl_compat_10_oshmsys(                             \
 913 |       res, (long long)(which), (long long)(a2), (long long)(a3),               \
 914 |       (long long)(a4))
 915 | #else
 916 | /* syscall 171 has been skipped */
 917 | #endif
 918 | /* syscall 172 has been skipped */
 919 | #define __sanitizer_syscall_pre_pread(fd, buf, nbyte, PAD, offset)             \
 920 |   __sanitizer_syscall_pre_impl_pread((long long)(fd), (long long)(buf),        \
```
- **Line 911 / 第 911 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 912 / 第 912 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 913 / 第 913 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 914 / 第 914 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 915 / 第 915 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 916 / 第 916 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 917 / 第 917 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 918 / 第 918 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 919 / 第 919 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 920 / 第 920 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 921-930 / 第 921-930 行
```cpp
 921 |                                      (long long)(nbyte), (long long)(PAD),     \
 922 |                                      (long long)(offset))
 923 | #define __sanitizer_syscall_post_pread(res, fd, buf, nbyte, PAD, offset)       \
 924 |   __sanitizer_syscall_post_impl_pread(res, (long long)(fd), (long long)(buf),  \
 925 |                                       (long long)(nbyte), (long long)(PAD),    \
 926 |                                       (long long)(offset))
 927 | #define __sanitizer_syscall_pre_pwrite(fd, buf, nbyte, PAD, offset)            \
 928 |   __sanitizer_syscall_pre_impl_pwrite((long long)(fd), (long long)(buf),       \
 929 |                                       (long long)(nbyte), (long long)(PAD),    \
 930 |                                       (long long)(offset))
```
- **Line 921 / 第 921 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 922 / 第 922 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 923 / 第 923 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 924 / 第 924 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 925 / 第 925 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 926 / 第 926 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 927 / 第 927 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 928 / 第 928 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 929 / 第 929 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 930 / 第 930 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 931-940 / 第 931-940 行
```cpp
 931 | #define __sanitizer_syscall_post_pwrite(res, fd, buf, nbyte, PAD, offset)      \
 932 |   __sanitizer_syscall_post_impl_pwrite(res, (long long)(fd), (long long)(buf), \
 933 |                                        (long long)(nbyte), (long long)(PAD),   \
 934 |                                        (long long)(offset))
 935 | #define __sanitizer_syscall_pre_compat_30_ntp_gettime(ntvp)                    \
 936 |   __sanitizer_syscall_pre_impl_compat_30_ntp_gettime((long long)(ntvp))
 937 | #define __sanitizer_syscall_post_compat_30_ntp_gettime(res, ntvp)              \
 938 |   __sanitizer_syscall_post_impl_compat_30_ntp_gettime(res, (long long)(ntvp))
 939 | #if defined(NTP) || !defined(_KERNEL_OPT)
 940 | #define __sanitizer_syscall_pre_ntp_adjtime(tp)                                \
```
- **Line 931 / 第 931 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 932 / 第 932 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 933 / 第 933 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 934 / 第 934 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 935 / 第 935 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 936 / 第 936 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 937 / 第 937 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 938 / 第 938 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 939 / 第 939 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 940 / 第 940 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 941-950 / 第 941-950 行
```cpp
 941 |   __sanitizer_syscall_pre_impl_ntp_adjtime((long long)(tp))
 942 | #define __sanitizer_syscall_post_ntp_adjtime(res, tp)                          \
 943 |   __sanitizer_syscall_post_impl_ntp_adjtime(res, (long long)(tp))
 944 | #else
 945 | /* syscall 176 has been skipped */
 946 | #endif
 947 | /* syscall 177 has been skipped */
 948 | /* syscall 178 has been skipped */
 949 | /* syscall 179 has been skipped */
 950 | /* syscall 180 has been skipped */
```
- **Line 941 / 第 941 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 942 / 第 942 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 943 / 第 943 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 944 / 第 944 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 945 / 第 945 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 946 / 第 946 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 947 / 第 947 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 948 / 第 948 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 949 / 第 949 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 950 / 第 950 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 951-960 / 第 951-960 行
```cpp
 951 | #define __sanitizer_syscall_pre_setgid(gid)                                    \
 952 |   __sanitizer_syscall_pre_impl_setgid((long long)(gid))
 953 | #define __sanitizer_syscall_post_setgid(res, gid)                              \
 954 |   __sanitizer_syscall_post_impl_setgid(res, (long long)(gid))
 955 | #define __sanitizer_syscall_pre_setegid(egid)                                  \
 956 |   __sanitizer_syscall_pre_impl_setegid((long long)(egid))
 957 | #define __sanitizer_syscall_post_setegid(res, egid)                            \
 958 |   __sanitizer_syscall_post_impl_setegid(res, (long long)(egid))
 959 | #define __sanitizer_syscall_pre_seteuid(euid)                                  \
 960 |   __sanitizer_syscall_pre_impl_seteuid((long long)(euid))
```
- **Line 951 / 第 951 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 952 / 第 952 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 953 / 第 953 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 954 / 第 954 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 955 / 第 955 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 956 / 第 956 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 957 / 第 957 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 958 / 第 958 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 959 / 第 959 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 960 / 第 960 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 961-970 / 第 961-970 行
```cpp
 961 | #define __sanitizer_syscall_post_seteuid(res, euid)                            \
 962 |   __sanitizer_syscall_post_impl_seteuid(res, (long long)(euid))
 963 | #define __sanitizer_syscall_pre_lfs_bmapv(fsidp, blkiov, blkcnt)               \
 964 |   __sanitizer_syscall_pre_impl_lfs_bmapv(                                      \
 965 |       (long long)(fsidp), (long long)(blkiov), (long long)(blkcnt))
 966 | #define __sanitizer_syscall_post_lfs_bmapv(res, fsidp, blkiov, blkcnt)         \
 967 |   __sanitizer_syscall_post_impl_lfs_bmapv(                                     \
 968 |       res, (long long)(fsidp), (long long)(blkiov), (long long)(blkcnt))
 969 | #define __sanitizer_syscall_pre_lfs_markv(fsidp, blkiov, blkcnt)               \
 970 |   __sanitizer_syscall_pre_impl_lfs_markv(                                      \
```
- **Line 961 / 第 961 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 962 / 第 962 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 963 / 第 963 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 964 / 第 964 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 965 / 第 965 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 966 / 第 966 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 967 / 第 967 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 968 / 第 968 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 969 / 第 969 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 970 / 第 970 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 971-980 / 第 971-980 行
```cpp
 971 |       (long long)(fsidp), (long long)(blkiov), (long long)(blkcnt))
 972 | #define __sanitizer_syscall_post_lfs_markv(res, fsidp, blkiov, blkcnt)         \
 973 |   __sanitizer_syscall_post_impl_lfs_markv(                                     \
 974 |       res, (long long)(fsidp), (long long)(blkiov), (long long)(blkcnt))
 975 | #define __sanitizer_syscall_pre_lfs_segclean(fsidp, segment)                   \
 976 |   __sanitizer_syscall_pre_impl_lfs_segclean((long long)(fsidp),                \
 977 |                                             (long long)(segment))
 978 | #define __sanitizer_syscall_post_lfs_segclean(res, fsidp, segment)             \
 979 |   __sanitizer_syscall_post_impl_lfs_segclean(res, (long long)(fsidp),          \
 980 |                                              (long long)(segment))
```
- **Line 971 / 第 971 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 972 / 第 972 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 973 / 第 973 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 974 / 第 974 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 975 / 第 975 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 976 / 第 976 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 977 / 第 977 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 978 / 第 978 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 979 / 第 979 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 980 / 第 980 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 981-990 / 第 981-990 行
```cpp
 981 | #define __sanitizer_syscall_pre_compat_50_lfs_segwait(fsidp, tv)               \
 982 |   __sanitizer_syscall_pre_impl_compat_50_lfs_segwait((long long)(fsidp),       \
 983 |                                                      (long long)(tv))
 984 | #define __sanitizer_syscall_post_compat_50_lfs_segwait(res, fsidp, tv)         \
 985 |   __sanitizer_syscall_post_impl_compat_50_lfs_segwait(res, (long long)(fsidp), \
 986 |                                                       (long long)(tv))
 987 | #define __sanitizer_syscall_pre_compat_12_stat12(path, ub)                     \
 988 |   __sanitizer_syscall_pre_impl_compat_12_stat12((long long)(path),             \
 989 |                                                 (long long)(ub))
 990 | #define __sanitizer_syscall_post_compat_12_stat12(res, path, ub)               \
```
- **Line 981 / 第 981 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 982 / 第 982 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 983 / 第 983 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 984 / 第 984 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 985 / 第 985 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 986 / 第 986 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 987 / 第 987 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 988 / 第 988 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 989 / 第 989 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 990 / 第 990 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 991-1000 / 第 991-1000 行
```cpp
 991 |   __sanitizer_syscall_post_impl_compat_12_stat12(res, (long long)(path),       \
 992 |                                                  (long long)(ub))
 993 | #define __sanitizer_syscall_pre_compat_12_fstat12(fd, sb)                      \
 994 |   __sanitizer_syscall_pre_impl_compat_12_fstat12((long long)(fd),              \
 995 |                                                  (long long)(sb))
 996 | #define __sanitizer_syscall_post_compat_12_fstat12(res, fd, sb)                \
 997 |   __sanitizer_syscall_post_impl_compat_12_fstat12(res, (long long)(fd),        \
 998 |                                                   (long long)(sb))
 999 | #define __sanitizer_syscall_pre_compat_12_lstat12(path, ub)                    \
1000 |   __sanitizer_syscall_pre_impl_compat_12_lstat12((long long)(path),            \
```
- **Line 991 / 第 991 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 992 / 第 992 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 993 / 第 993 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 994 / 第 994 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 995 / 第 995 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 996 / 第 996 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 997 / 第 997 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 998 / 第 998 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 999 / 第 999 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1000 / 第 1000 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1001-1010 / 第 1001-1010 行
```cpp
1001 |                                                  (long long)(ub))
1002 | #define __sanitizer_syscall_post_compat_12_lstat12(res, path, ub)              \
1003 |   __sanitizer_syscall_post_impl_compat_12_lstat12(res, (long long)(path),      \
1004 |                                                   (long long)(ub))
1005 | #define __sanitizer_syscall_pre_pathconf(path, name)                           \
1006 |   __sanitizer_syscall_pre_impl_pathconf((long long)(path), (long long)(name))
1007 | #define __sanitizer_syscall_post_pathconf(res, path, name)                     \
1008 |   __sanitizer_syscall_post_impl_pathconf(res, (long long)(path),               \
1009 |                                          (long long)(name))
1010 | #define __sanitizer_syscall_pre_fpathconf(fd, name)                            \
```
- **Line 1001 / 第 1001 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1002 / 第 1002 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1003 / 第 1003 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1004 / 第 1004 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1005 / 第 1005 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1006 / 第 1006 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1007 / 第 1007 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1008 / 第 1008 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1009 / 第 1009 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1010 / 第 1010 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1011-1020 / 第 1011-1020 行
```cpp
1011 |   __sanitizer_syscall_pre_impl_fpathconf((long long)(fd), (long long)(name))
1012 | #define __sanitizer_syscall_post_fpathconf(res, fd, name)                      \
1013 |   __sanitizer_syscall_post_impl_fpathconf(res, (long long)(fd),                \
1014 |                                           (long long)(name))
1015 | #define __sanitizer_syscall_pre_getsockopt2(s, level, name, val, avalsize)     \
1016 |   __sanitizer_syscall_pre_impl_getsockopt2(                                    \
1017 |       (long long)(s), (long long)(level), (long long)(name), (long long)(val), \
1018 |       (long long)(avalsize))
1019 | #define __sanitizer_syscall_post_getsockopt2(res, s, level, name, val,         \
1020 |                                              avalsize)                         \
```
- **Line 1011 / 第 1011 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1012 / 第 1012 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1013 / 第 1013 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1014 / 第 1014 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1015 / 第 1015 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1016 / 第 1016 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1017 / 第 1017 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1018 / 第 1018 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1019 / 第 1019 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1020 / 第 1020 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1021-1030 / 第 1021-1030 行
```cpp
1021 |   __sanitizer_syscall_post_impl_getsockopt2(                                   \
1022 |       res, (long long)(s), (long long)(level), (long long)(name),              \
1023 |       (long long)(val), (long long)(avalsize))
1024 | #define __sanitizer_syscall_pre_getrlimit(which, rlp)                          \
1025 |   __sanitizer_syscall_pre_impl_getrlimit((long long)(which), (long long)(rlp))
1026 | #define __sanitizer_syscall_post_getrlimit(res, which, rlp)                    \
1027 |   __sanitizer_syscall_post_impl_getrlimit(res, (long long)(which),             \
1028 |                                           (long long)(rlp))
1029 | #define __sanitizer_syscall_pre_setrlimit(which, rlp)                          \
1030 |   __sanitizer_syscall_pre_impl_setrlimit((long long)(which), (long long)(rlp))
```
- **Line 1021 / 第 1021 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1022 / 第 1022 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1023 / 第 1023 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1024 / 第 1024 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1025 / 第 1025 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1026 / 第 1026 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1027 / 第 1027 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1028 / 第 1028 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1029 / 第 1029 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1030 / 第 1030 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1031-1040 / 第 1031-1040 行
```cpp
1031 | #define __sanitizer_syscall_post_setrlimit(res, which, rlp)                    \
1032 |   __sanitizer_syscall_post_impl_setrlimit(res, (long long)(which),             \
1033 |                                           (long long)(rlp))
1034 | #define __sanitizer_syscall_pre_compat_12_getdirentries(fd, buf, count, basep) \
1035 |   __sanitizer_syscall_pre_impl_compat_12_getdirentries(                        \
1036 |       (long long)(fd), (long long)(buf), (long long)(count),                   \
1037 |       (long long)(basep))
1038 | #define __sanitizer_syscall_post_compat_12_getdirentries(res, fd, buf, count,  \
1039 |                                                          basep)                \
1040 |   __sanitizer_syscall_post_impl_compat_12_getdirentries(                       \
```
- **Line 1031 / 第 1031 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1032 / 第 1032 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1033 / 第 1033 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1034 / 第 1034 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1035 / 第 1035 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1036 / 第 1036 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1037 / 第 1037 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1038 / 第 1038 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1039 / 第 1039 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1040 / 第 1040 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1041-1050 / 第 1041-1050 行
```cpp
1041 |       res, (long long)(fd), (long long)(buf), (long long)(count),              \
1042 |       (long long)(basep))
1043 | #define __sanitizer_syscall_pre_mmap(addr, len, prot, flags, fd, PAD, pos)     \
1044 |   __sanitizer_syscall_pre_impl_mmap(                                           \
1045 |       (long long)(addr), (long long)(len), (long long)(prot),                  \
1046 |       (long long)(flags), (long long)(fd), (long long)(PAD), (long long)(pos))
1047 | #define __sanitizer_syscall_post_mmap(res, addr, len, prot, flags, fd, PAD,    \
1048 |                                       pos)                                     \
1049 |   __sanitizer_syscall_post_impl_mmap(                                          \
1050 |       res, (long long)(addr), (long long)(len), (long long)(prot),             \
```
- **Line 1041 / 第 1041 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1042 / 第 1042 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1043 / 第 1043 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1044 / 第 1044 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1045 / 第 1045 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1046 / 第 1046 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1047 / 第 1047 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1048 / 第 1048 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1049 / 第 1049 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1050 / 第 1050 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1051-1060 / 第 1051-1060 行
```cpp
1051 |       (long long)(flags), (long long)(fd), (long long)(PAD), (long long)(pos))
1052 | #define __sanitizer_syscall_pre___syscall(code, arg0, arg1, arg2, arg3, arg4,  \
1053 |                                           arg5, arg6, arg7)                    \
1054 |   __sanitizer_syscall_pre_impl___syscall(                                      \
1055 |       (long long)(code), (long long)(arg0), (long long)(arg1),                 \
1056 |       (long long)(arg2), (long long)(arg3), (long long)(arg4),                 \
1057 |       (long long)(arg5), (long long)(arg6), (long long)(arg7))
1058 | #define __sanitizer_syscall_post___syscall(res, code, arg0, arg1, arg2, arg3,  \
1059 |                                            arg4, arg5, arg6, arg7)             \
1060 |   __sanitizer_syscall_post_impl___syscall(                                     \
```
- **Line 1051 / 第 1051 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1052 / 第 1052 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1053 / 第 1053 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1054 / 第 1054 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1055 / 第 1055 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1056 / 第 1056 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1057 / 第 1057 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1058 / 第 1058 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1059 / 第 1059 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1060 / 第 1060 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1061-1070 / 第 1061-1070 行
```cpp
1061 |       res, (long long)(code), (long long)(arg0), (long long)(arg1),            \
1062 |       (long long)(arg2), (long long)(arg3), (long long)(arg4),                 \
1063 |       (long long)(arg5), (long long)(arg6), (long long)(arg7))
1064 | #define __sanitizer_syscall_pre_lseek(fd, PAD, offset, whence)                 \
1065 |   __sanitizer_syscall_pre_impl_lseek((long long)(fd), (long long)(PAD),        \
1066 |                                      (long long)(offset), (long long)(whence))
1067 | #define __sanitizer_syscall_post_lseek(res, fd, PAD, offset, whence)           \
1068 |   __sanitizer_syscall_post_impl_lseek(res, (long long)(fd), (long long)(PAD),  \
1069 |                                       (long long)(offset),                     \
1070 |                                       (long long)(whence))
```
- **Line 1061 / 第 1061 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1062 / 第 1062 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1063 / 第 1063 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1064 / 第 1064 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1065 / 第 1065 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1066 / 第 1066 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1067 / 第 1067 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1068 / 第 1068 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1069 / 第 1069 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1070 / 第 1070 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1071-1080 / 第 1071-1080 行
```cpp
1071 | #define __sanitizer_syscall_pre_truncate(path, PAD, length)                    \
1072 |   __sanitizer_syscall_pre_impl_truncate((long long)(path), (long long)(PAD),   \
1073 |                                         (long long)(length))
1074 | #define __sanitizer_syscall_post_truncate(res, path, PAD, length)              \
1075 |   __sanitizer_syscall_post_impl_truncate(                                      \
1076 |       res, (long long)(path), (long long)(PAD), (long long)(length))
1077 | #define __sanitizer_syscall_pre_ftruncate(fd, PAD, length)                     \
1078 |   __sanitizer_syscall_pre_impl_ftruncate((long long)(fd), (long long)(PAD),    \
1079 |                                          (long long)(length))
1080 | #define __sanitizer_syscall_post_ftruncate(res, fd, PAD, length)               \
```
- **Line 1071 / 第 1071 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1072 / 第 1072 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1073 / 第 1073 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1074 / 第 1074 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1075 / 第 1075 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1076 / 第 1076 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1077 / 第 1077 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1078 / 第 1078 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1079 / 第 1079 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1080 / 第 1080 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1081-1090 / 第 1081-1090 行
```cpp
1081 |   __sanitizer_syscall_post_impl_ftruncate(                                     \
1082 |       res, (long long)(fd), (long long)(PAD), (long long)(length))
1083 | #define __sanitizer_syscall_pre___sysctl(name, namelen, oldv, oldlenp, newv,   \
1084 |                                          newlen)                               \
1085 |   __sanitizer_syscall_pre_impl___sysctl(                                       \
1086 |       (long long)(name), (long long)(namelen), (long long)(oldv),              \
1087 |       (long long)(oldlenp), (long long)(newv), (long long)(newlen))
1088 | #define __sanitizer_syscall_post___sysctl(res, name, namelen, oldv, oldlenp,   \
1089 |                                           newv, newlen)                        \
1090 |   __sanitizer_syscall_post_impl___sysctl(                                      \
```
- **Line 1081 / 第 1081 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1082 / 第 1082 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1083 / 第 1083 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1084 / 第 1084 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1085 / 第 1085 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1086 / 第 1086 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1087 / 第 1087 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1088 / 第 1088 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1089 / 第 1089 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1090 / 第 1090 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1091-1100 / 第 1091-1100 行
```cpp
1091 |       res, (long long)(name), (long long)(namelen), (long long)(oldv),         \
1092 |       (long long)(oldlenp), (long long)(newv), (long long)(newlen))
1093 | #define __sanitizer_syscall_pre_mlock(addr, len)                               \
1094 |   __sanitizer_syscall_pre_impl_mlock((long long)(addr), (long long)(len))
1095 | #define __sanitizer_syscall_post_mlock(res, addr, len)                         \
1096 |   __sanitizer_syscall_post_impl_mlock(res, (long long)(addr), (long long)(len))
1097 | #define __sanitizer_syscall_pre_munlock(addr, len)                             \
1098 |   __sanitizer_syscall_pre_impl_munlock((long long)(addr), (long long)(len))
1099 | #define __sanitizer_syscall_post_munlock(res, addr, len)                       \
1100 |   __sanitizer_syscall_post_impl_munlock(res, (long long)(addr),                \
```
- **Line 1091 / 第 1091 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1092 / 第 1092 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1093 / 第 1093 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1094 / 第 1094 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1095 / 第 1095 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1096 / 第 1096 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1097 / 第 1097 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1098 / 第 1098 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1099 / 第 1099 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1100 / 第 1100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1101-1110 / 第 1101-1110 行
```cpp
1101 |                                         (long long)(len))
1102 | #define __sanitizer_syscall_pre_undelete(path)                                 \
1103 |   __sanitizer_syscall_pre_impl_undelete((long long)(path))
1104 | #define __sanitizer_syscall_post_undelete(res, path)                           \
1105 |   __sanitizer_syscall_post_impl_undelete(res, (long long)(path))
1106 | #define __sanitizer_syscall_pre_compat_50_futimes(fd, tptr)                    \
1107 |   __sanitizer_syscall_pre_impl_compat_50_futimes((long long)(fd),              \
1108 |                                                  (long long)(tptr))
1109 | #define __sanitizer_syscall_post_compat_50_futimes(res, fd, tptr)              \
1110 |   __sanitizer_syscall_post_impl_compat_50_futimes(res, (long long)(fd),        \
```
- **Line 1101 / 第 1101 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1102 / 第 1102 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1103 / 第 1103 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1104 / 第 1104 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1105 / 第 1105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1106 / 第 1106 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1107 / 第 1107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1108 / 第 1108 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1109 / 第 1109 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1110 / 第 1110 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1111-1120 / 第 1111-1120 行
```cpp
1111 |                                                   (long long)(tptr))
1112 | #define __sanitizer_syscall_pre_getpgid(pid)                                   \
1113 |   __sanitizer_syscall_pre_impl_getpgid((long long)(pid))
1114 | #define __sanitizer_syscall_post_getpgid(res, pid)                             \
1115 |   __sanitizer_syscall_post_impl_getpgid(res, (long long)(pid))
1116 | #define __sanitizer_syscall_pre_reboot(opt, bootstr)                           \
1117 |   __sanitizer_syscall_pre_impl_reboot((long long)(opt), (long long)(bootstr))
1118 | #define __sanitizer_syscall_post_reboot(res, opt, bootstr)                     \
1119 |   __sanitizer_syscall_post_impl_reboot(res, (long long)(opt),                  \
1120 |                                        (long long)(bootstr))
```
- **Line 1111 / 第 1111 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1112 / 第 1112 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1113 / 第 1113 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1114 / 第 1114 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1115 / 第 1115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1116 / 第 1116 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1117 / 第 1117 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1118 / 第 1118 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1119 / 第 1119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1120 / 第 1120 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1121-1130 / 第 1121-1130 行
```cpp
1121 | #define __sanitizer_syscall_pre_poll(fds, nfds, timeout)                       \
1122 |   __sanitizer_syscall_pre_impl_poll((long long)(fds), (long long)(nfds),       \
1123 |                                     (long long)(timeout))
1124 | #define __sanitizer_syscall_post_poll(res, fds, nfds, timeout)                 \
1125 |   __sanitizer_syscall_post_impl_poll(res, (long long)(fds), (long long)(nfds), \
1126 |                                      (long long)(timeout))
1127 | #define __sanitizer_syscall_pre_afssys(id, a1, a2, a3, a4, a5, a6)             \
1128 |   __sanitizer_syscall_pre_impl_afssys(                                         \
1129 |       (long long)(id), (long long)(a1), (long long)(a2), (long long)(a3),      \
1130 |       (long long)(a4), (long long)(a5), (long long)(a6))
```
- **Line 1121 / 第 1121 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1122 / 第 1122 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1123 / 第 1123 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1124 / 第 1124 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1125 / 第 1125 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1126 / 第 1126 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1127 / 第 1127 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1128 / 第 1128 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1129 / 第 1129 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1130 / 第 1130 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1131-1140 / 第 1131-1140 行
```cpp
1131 | #define __sanitizer_syscall_post_afssys(res, id, a1, a2, a3, a4, a5, a6)       \
1132 |   __sanitizer_syscall_post_impl_afssys(                                        \
1133 |       res, (long long)(id), (long long)(a1), (long long)(a2), (long long)(a3), \
1134 |       (long long)(a4), (long long)(a5), (long long)(a6))
1135 | /* syscall 211 has been skipped */
1136 | /* syscall 212 has been skipped */
1137 | /* syscall 213 has been skipped */
1138 | /* syscall 214 has been skipped */
1139 | /* syscall 215 has been skipped */
1140 | /* syscall 216 has been skipped */
```
- **Line 1131 / 第 1131 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1132 / 第 1132 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1133 / 第 1133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1134 / 第 1134 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1135 / 第 1135 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1136 / 第 1136 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1137 / 第 1137 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1138 / 第 1138 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1139 / 第 1139 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1140 / 第 1140 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 1141-1150 / 第 1141-1150 行
```cpp
1141 | /* syscall 217 has been skipped */
1142 | /* syscall 218 has been skipped */
1143 | /* syscall 219 has been skipped */
1144 | #define __sanitizer_syscall_pre_compat_14___semctl(semid, semnum, cmd, arg)    \
1145 |   __sanitizer_syscall_pre_impl_compat_14___semctl(                             \
1146 |       (long long)(semid), (long long)(semnum), (long long)(cmd),               \
1147 |       (long long)(arg))
1148 | #define __sanitizer_syscall_post_compat_14___semctl(res, semid, semnum, cmd,   \
1149 |                                                     arg)                       \
1150 |   __sanitizer_syscall_post_impl_compat_14___semctl(                            \
```
- **Line 1141 / 第 1141 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1142 / 第 1142 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1143 / 第 1143 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1144 / 第 1144 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1145 / 第 1145 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1146 / 第 1146 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1147 / 第 1147 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1148 / 第 1148 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1149 / 第 1149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1150 / 第 1150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1151-1160 / 第 1151-1160 行
```cpp
1151 |       res, (long long)(semid), (long long)(semnum), (long long)(cmd),          \
1152 |       (long long)(arg))
1153 | #define __sanitizer_syscall_pre_semget(key, nsems, semflg)                     \
1154 |   __sanitizer_syscall_pre_impl_semget((long long)(key), (long long)(nsems),    \
1155 |                                       (long long)(semflg))
1156 | #define __sanitizer_syscall_post_semget(res, key, nsems, semflg)               \
1157 |   __sanitizer_syscall_post_impl_semget(                                        \
1158 |       res, (long long)(key), (long long)(nsems), (long long)(semflg))
1159 | #define __sanitizer_syscall_pre_semop(semid, sops, nsops)                      \
1160 |   __sanitizer_syscall_pre_impl_semop((long long)(semid), (long long)(sops),    \
```
- **Line 1151 / 第 1151 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
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
1161 |                                      (long long)(nsops))
1162 | #define __sanitizer_syscall_post_semop(res, semid, sops, nsops)                \
1163 |   __sanitizer_syscall_post_impl_semop(res, (long long)(semid),                 \
1164 |                                       (long long)(sops), (long long)(nsops))
1165 | #define __sanitizer_syscall_pre_semconfig(flag)                                \
1166 |   __sanitizer_syscall_pre_impl_semconfig((long long)(flag))
1167 | #define __sanitizer_syscall_post_semconfig(res, flag)                          \
1168 |   __sanitizer_syscall_post_impl_semconfig(res, (long long)(flag))
1169 | #define __sanitizer_syscall_pre_compat_14_msgctl(msqid, cmd, buf)              \
1170 |   __sanitizer_syscall_pre_impl_compat_14_msgctl(                               \
```
- **Line 1161 / 第 1161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1162 / 第 1162 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1163 / 第 1163 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1164 / 第 1164 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1165 / 第 1165 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1166 / 第 1166 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1167 / 第 1167 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1168 / 第 1168 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1169 / 第 1169 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1170 / 第 1170 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1171-1180 / 第 1171-1180 行
```cpp
1171 |       (long long)(msqid), (long long)(cmd), (long long)(buf))
1172 | #define __sanitizer_syscall_post_compat_14_msgctl(res, msqid, cmd, buf)        \
1173 |   __sanitizer_syscall_post_impl_compat_14_msgctl(                              \
1174 |       res, (long long)(msqid), (long long)(cmd), (long long)(buf))
1175 | #define __sanitizer_syscall_pre_msgget(key, msgflg)                            \
1176 |   __sanitizer_syscall_pre_impl_msgget((long long)(key), (long long)(msgflg))
1177 | #define __sanitizer_syscall_post_msgget(res, key, msgflg)                      \
1178 |   __sanitizer_syscall_post_impl_msgget(res, (long long)(key),                  \
1179 |                                        (long long)(msgflg))
1180 | #define __sanitizer_syscall_pre_msgsnd(msqid, msgp, msgsz, msgflg)             \
```
- **Line 1171 / 第 1171 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1172 / 第 1172 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1173 / 第 1173 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1174 / 第 1174 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1175 / 第 1175 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1176 / 第 1176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1177 / 第 1177 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1178 / 第 1178 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1179 / 第 1179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1180 / 第 1180 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1181-1190 / 第 1181-1190 行
```cpp
1181 |   __sanitizer_syscall_pre_impl_msgsnd((long long)(msqid), (long long)(msgp),   \
1182 |                                       (long long)(msgsz), (long long)(msgflg))
1183 | #define __sanitizer_syscall_post_msgsnd(res, msqid, msgp, msgsz, msgflg)       \
1184 |   __sanitizer_syscall_post_impl_msgsnd(res, (long long)(msqid),                \
1185 |                                        (long long)(msgp), (long long)(msgsz),  \
1186 |                                        (long long)(msgflg))
1187 | #define __sanitizer_syscall_pre_msgrcv(msqid, msgp, msgsz, msgtyp, msgflg)     \
1188 |   __sanitizer_syscall_pre_impl_msgrcv((long long)(msqid), (long long)(msgp),   \
1189 |                                       (long long)(msgsz), (long long)(msgtyp), \
1190 |                                       (long long)(msgflg))
```
- **Line 1181 / 第 1181 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1182 / 第 1182 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1183 / 第 1183 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1184 / 第 1184 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1185 / 第 1185 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1186 / 第 1186 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1187 / 第 1187 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1188 / 第 1188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1189 / 第 1189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1190 / 第 1190 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1191-1200 / 第 1191-1200 行
```cpp
1191 | #define __sanitizer_syscall_post_msgrcv(res, msqid, msgp, msgsz, msgtyp,       \
1192 |                                         msgflg)                                \
1193 |   __sanitizer_syscall_post_impl_msgrcv(                                        \
1194 |       res, (long long)(msqid), (long long)(msgp), (long long)(msgsz),          \
1195 |       (long long)(msgtyp), (long long)(msgflg))
1196 | #define __sanitizer_syscall_pre_shmat(shmid, shmaddr, shmflg)                  \
1197 |   __sanitizer_syscall_pre_impl_shmat((long long)(shmid), (long long)(shmaddr), \
1198 |                                      (long long)(shmflg))
1199 | #define __sanitizer_syscall_post_shmat(res, shmid, shmaddr, shmflg)            \
1200 |   __sanitizer_syscall_post_impl_shmat(                                         \
```
- **Line 1191 / 第 1191 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1192 / 第 1192 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1193 / 第 1193 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1194 / 第 1194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1195 / 第 1195 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1196 / 第 1196 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1197 / 第 1197 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1198 / 第 1198 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1199 / 第 1199 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1200 / 第 1200 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1201-1210 / 第 1201-1210 行
```cpp
1201 |       res, (long long)(shmid), (long long)(shmaddr), (long long)(shmflg))
1202 | #define __sanitizer_syscall_pre_compat_14_shmctl(shmid, cmd, buf)              \
1203 |   __sanitizer_syscall_pre_impl_compat_14_shmctl(                               \
1204 |       (long long)(shmid), (long long)(cmd), (long long)(buf))
1205 | #define __sanitizer_syscall_post_compat_14_shmctl(res, shmid, cmd, buf)        \
1206 |   __sanitizer_syscall_post_impl_compat_14_shmctl(                              \
1207 |       res, (long long)(shmid), (long long)(cmd), (long long)(buf))
1208 | #define __sanitizer_syscall_pre_shmdt(shmaddr)                                 \
1209 |   __sanitizer_syscall_pre_impl_shmdt((long long)(shmaddr))
1210 | #define __sanitizer_syscall_post_shmdt(res, shmaddr)                           \
```
- **Line 1201 / 第 1201 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1202 / 第 1202 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1203 / 第 1203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1204 / 第 1204 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1205 / 第 1205 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1206 / 第 1206 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1207 / 第 1207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1208 / 第 1208 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1209 / 第 1209 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1210 / 第 1210 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1211-1220 / 第 1211-1220 行
```cpp
1211 |   __sanitizer_syscall_post_impl_shmdt(res, (long long)(shmaddr))
1212 | #define __sanitizer_syscall_pre_shmget(key, size, shmflg)                      \
1213 |   __sanitizer_syscall_pre_impl_shmget((long long)(key), (long long)(size),     \
1214 |                                       (long long)(shmflg))
1215 | #define __sanitizer_syscall_post_shmget(res, key, size, shmflg)                \
1216 |   __sanitizer_syscall_post_impl_shmget(res, (long long)(key),                  \
1217 |                                        (long long)(size), (long long)(shmflg))
1218 | #define __sanitizer_syscall_pre_compat_50_clock_gettime(clock_id, tp)          \
1219 |   __sanitizer_syscall_pre_impl_compat_50_clock_gettime((long long)(clock_id),  \
1220 |                                                        (long long)(tp))
```
- **Line 1211 / 第 1211 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1212 / 第 1212 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1213 / 第 1213 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1214 / 第 1214 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1215 / 第 1215 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1216 / 第 1216 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1217 / 第 1217 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1218 / 第 1218 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1219 / 第 1219 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1220 / 第 1220 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1221-1230 / 第 1221-1230 行
```cpp
1221 | #define __sanitizer_syscall_post_compat_50_clock_gettime(res, clock_id, tp)    \
1222 |   __sanitizer_syscall_post_impl_compat_50_clock_gettime(                       \
1223 |       res, (long long)(clock_id), (long long)(tp))
1224 | #define __sanitizer_syscall_pre_compat_50_clock_settime(clock_id, tp)          \
1225 |   __sanitizer_syscall_pre_impl_compat_50_clock_settime((long long)(clock_id),  \
1226 |                                                        (long long)(tp))
1227 | #define __sanitizer_syscall_post_compat_50_clock_settime(res, clock_id, tp)    \
1228 |   __sanitizer_syscall_post_impl_compat_50_clock_settime(                       \
1229 |       res, (long long)(clock_id), (long long)(tp))
1230 | #define __sanitizer_syscall_pre_compat_50_clock_getres(clock_id, tp)           \
```
- **Line 1221 / 第 1221 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1222 / 第 1222 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1223 / 第 1223 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1224 / 第 1224 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1225 / 第 1225 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1226 / 第 1226 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1227 / 第 1227 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1228 / 第 1228 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1229 / 第 1229 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1230 / 第 1230 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1231-1240 / 第 1231-1240 行
```cpp
1231 |   __sanitizer_syscall_pre_impl_compat_50_clock_getres((long long)(clock_id),   \
1232 |                                                       (long long)(tp))
1233 | #define __sanitizer_syscall_post_compat_50_clock_getres(res, clock_id, tp)     \
1234 |   __sanitizer_syscall_post_impl_compat_50_clock_getres(                        \
1235 |       res, (long long)(clock_id), (long long)(tp))
1236 | #define __sanitizer_syscall_pre_timer_create(clock_id, evp, timerid)           \
1237 |   __sanitizer_syscall_pre_impl_timer_create(                                   \
1238 |       (long long)(clock_id), (long long)(evp), (long long)(timerid))
1239 | #define __sanitizer_syscall_post_timer_create(res, clock_id, evp, timerid)     \
1240 |   __sanitizer_syscall_post_impl_timer_create(                                  \
```
- **Line 1231 / 第 1231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1232 / 第 1232 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1233 / 第 1233 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1234 / 第 1234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1235 / 第 1235 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1236 / 第 1236 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1237 / 第 1237 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1238 / 第 1238 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1239 / 第 1239 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1240 / 第 1240 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1241-1250 / 第 1241-1250 行
```cpp
1241 |       res, (long long)(clock_id), (long long)(evp), (long long)(timerid))
1242 | #define __sanitizer_syscall_pre_timer_delete(timerid)                          \
1243 |   __sanitizer_syscall_pre_impl_timer_delete((long long)(timerid))
1244 | #define __sanitizer_syscall_post_timer_delete(res, timerid)                    \
1245 |   __sanitizer_syscall_post_impl_timer_delete(res, (long long)(timerid))
1246 | #define __sanitizer_syscall_pre_compat_50_timer_settime(timerid, flags, value, \
1247 |                                                         ovalue)                \
1248 |   __sanitizer_syscall_pre_impl_compat_50_timer_settime(                        \
1249 |       (long long)(timerid), (long long)(flags), (long long)(value),            \
1250 |       (long long)(ovalue))
```
- **Line 1241 / 第 1241 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1242 / 第 1242 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1243 / 第 1243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1244 / 第 1244 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1245 / 第 1245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1246 / 第 1246 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1247 / 第 1247 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1248 / 第 1248 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1249 / 第 1249 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1250 / 第 1250 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1251-1260 / 第 1251-1260 行
```cpp
1251 | #define __sanitizer_syscall_post_compat_50_timer_settime(res, timerid, flags,  \
1252 |                                                          value, ovalue)        \
1253 |   __sanitizer_syscall_post_impl_compat_50_timer_settime(                       \
1254 |       res, (long long)(timerid), (long long)(flags), (long long)(value),       \
1255 |       (long long)(ovalue))
1256 | #define __sanitizer_syscall_pre_compat_50_timer_gettime(timerid, value)        \
1257 |   __sanitizer_syscall_pre_impl_compat_50_timer_gettime((long long)(timerid),   \
1258 |                                                        (long long)(value))
1259 | #define __sanitizer_syscall_post_compat_50_timer_gettime(res, timerid, value)  \
1260 |   __sanitizer_syscall_post_impl_compat_50_timer_gettime(                       \
```
- **Line 1251 / 第 1251 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1252 / 第 1252 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1253 / 第 1253 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1254 / 第 1254 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1255 / 第 1255 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1256 / 第 1256 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1257 / 第 1257 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1258 / 第 1258 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1259 / 第 1259 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1260 / 第 1260 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1261-1270 / 第 1261-1270 行
```cpp
1261 |       res, (long long)(timerid), (long long)(value))
1262 | #define __sanitizer_syscall_pre_timer_getoverrun(timerid)                      \
1263 |   __sanitizer_syscall_pre_impl_timer_getoverrun((long long)(timerid))
1264 | #define __sanitizer_syscall_post_timer_getoverrun(res, timerid)                \
1265 |   __sanitizer_syscall_post_impl_timer_getoverrun(res, (long long)(timerid))
1266 | #define __sanitizer_syscall_pre_compat_50_nanosleep(rqtp, rmtp)                \
1267 |   __sanitizer_syscall_pre_impl_compat_50_nanosleep((long long)(rqtp),          \
1268 |                                                    (long long)(rmtp))
1269 | #define __sanitizer_syscall_post_compat_50_nanosleep(res, rqtp, rmtp)          \
1270 |   __sanitizer_syscall_post_impl_compat_50_nanosleep(res, (long long)(rqtp),    \
```
- **Line 1261 / 第 1261 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1262 / 第 1262 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1263 / 第 1263 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1264 / 第 1264 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1265 / 第 1265 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1266 / 第 1266 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1267 / 第 1267 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1268 / 第 1268 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1269 / 第 1269 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1270 / 第 1270 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1271-1280 / 第 1271-1280 行
```cpp
1271 |                                                     (long long)(rmtp))
1272 | #define __sanitizer_syscall_pre_fdatasync(fd)                                  \
1273 |   __sanitizer_syscall_pre_impl_fdatasync((long long)(fd))
1274 | #define __sanitizer_syscall_post_fdatasync(res, fd)                            \
1275 |   __sanitizer_syscall_post_impl_fdatasync(res, (long long)(fd))
1276 | #define __sanitizer_syscall_pre_mlockall(flags)                                \
1277 |   __sanitizer_syscall_pre_impl_mlockall((long long)(flags))
1278 | #define __sanitizer_syscall_post_mlockall(res, flags)                          \
1279 |   __sanitizer_syscall_post_impl_mlockall(res, (long long)(flags))
1280 | #define __sanitizer_syscall_pre_munlockall()                                   \
```
- **Line 1271 / 第 1271 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1272 / 第 1272 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1273 / 第 1273 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1274 / 第 1274 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1275 / 第 1275 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1276 / 第 1276 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1277 / 第 1277 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1278 / 第 1278 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1279 / 第 1279 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1280 / 第 1280 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1281-1290 / 第 1281-1290 行
```cpp
1281 |   __sanitizer_syscall_pre_impl_munlockall()
1282 | #define __sanitizer_syscall_post_munlockall(res)                               \
1283 |   __sanitizer_syscall_post_impl_munlockall(res)
1284 | #define __sanitizer_syscall_pre_compat_50___sigtimedwait(set, info, timeout)   \
1285 |   __sanitizer_syscall_pre_impl_compat_50___sigtimedwait(                       \
1286 |       (long long)(set), (long long)(info), (long long)(timeout))
1287 | #define __sanitizer_syscall_post_compat_50___sigtimedwait(res, set, info,      \
1288 |                                                           timeout)             \
1289 |   __sanitizer_syscall_post_impl_compat_50___sigtimedwait(                      \
1290 |       res, (long long)(set), (long long)(info), (long long)(timeout))
```
- **Line 1281 / 第 1281 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1282 / 第 1282 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1283 / 第 1283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1284 / 第 1284 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1285 / 第 1285 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1286 / 第 1286 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1287 / 第 1287 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1288 / 第 1288 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1289 / 第 1289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1290 / 第 1290 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1291-1300 / 第 1291-1300 行
```cpp
1291 | #define __sanitizer_syscall_pre_sigqueueinfo(pid, info)                        \
1292 |   __sanitizer_syscall_pre_impl_sigqueueinfo((long long)(pid), (long long)(info))
1293 | #define __sanitizer_syscall_post_sigqueueinfo(res, pid, info)                  \
1294 |   __sanitizer_syscall_post_impl_sigqueueinfo(res, (long long)(pid),            \
1295 |                                              (long long)(info))
1296 | #define __sanitizer_syscall_pre_modctl(cmd, arg)                               \
1297 |   __sanitizer_syscall_pre_impl_modctl((long long)(cmd), (long long)(arg))
1298 | #define __sanitizer_syscall_post_modctl(res, cmd, arg)                         \
1299 |   __sanitizer_syscall_post_impl_modctl(res, (long long)(cmd), (long long)(arg))
1300 | #define __sanitizer_syscall_pre__ksem_init(value, idp)                         \
```
- **Line 1291 / 第 1291 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1292 / 第 1292 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1293 / 第 1293 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1294 / 第 1294 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1295 / 第 1295 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1296 / 第 1296 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1297 / 第 1297 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1298 / 第 1298 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1299 / 第 1299 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1300 / 第 1300 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1301-1310 / 第 1301-1310 行
```cpp
1301 |   __sanitizer_syscall_pre_impl__ksem_init((long long)(value), (long long)(idp))
1302 | #define __sanitizer_syscall_post__ksem_init(res, value, idp)                   \
1303 |   __sanitizer_syscall_post_impl__ksem_init(res, (long long)(value),            \
1304 |                                            (long long)(idp))
1305 | #define __sanitizer_syscall_pre__ksem_open(name, oflag, mode, value, idp)      \
1306 |   __sanitizer_syscall_pre_impl__ksem_open(                                     \
1307 |       (long long)(name), (long long)(oflag), (long long)(mode),                \
1308 |       (long long)(value), (long long)(idp))
1309 | #define __sanitizer_syscall_post__ksem_open(res, name, oflag, mode, value,     \
1310 |                                             idp)                               \
```
- **Line 1301 / 第 1301 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1302 / 第 1302 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1303 / 第 1303 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1304 / 第 1304 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1305 / 第 1305 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1306 / 第 1306 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1307 / 第 1307 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1308 / 第 1308 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1309 / 第 1309 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1310 / 第 1310 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1311-1320 / 第 1311-1320 行
```cpp
1311 |   __sanitizer_syscall_post_impl__ksem_open(                                    \
1312 |       res, (long long)(name), (long long)(oflag), (long long)(mode),           \
1313 |       (long long)(value), (long long)(idp))
1314 | #define __sanitizer_syscall_pre__ksem_unlink(name)                             \
1315 |   __sanitizer_syscall_pre_impl__ksem_unlink((long long)(name))
1316 | #define __sanitizer_syscall_post__ksem_unlink(res, name)                       \
1317 |   __sanitizer_syscall_post_impl__ksem_unlink(res, (long long)(name))
1318 | #define __sanitizer_syscall_pre__ksem_close(id)                                \
1319 |   __sanitizer_syscall_pre_impl__ksem_close((long long)(id))
1320 | #define __sanitizer_syscall_post__ksem_close(res, id)                          \
```
- **Line 1311 / 第 1311 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1312 / 第 1312 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1313 / 第 1313 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1314 / 第 1314 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1315 / 第 1315 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1316 / 第 1316 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1317 / 第 1317 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1318 / 第 1318 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1319 / 第 1319 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1320 / 第 1320 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1321-1330 / 第 1321-1330 行
```cpp
1321 |   __sanitizer_syscall_post_impl__ksem_close(res, (long long)(id))
1322 | #define __sanitizer_syscall_pre__ksem_post(id)                                 \
1323 |   __sanitizer_syscall_pre_impl__ksem_post((long long)(id))
1324 | #define __sanitizer_syscall_post__ksem_post(res, id)                           \
1325 |   __sanitizer_syscall_post_impl__ksem_post(res, (long long)(id))
1326 | #define __sanitizer_syscall_pre__ksem_wait(id)                                 \
1327 |   __sanitizer_syscall_pre_impl__ksem_wait((long long)(id))
1328 | #define __sanitizer_syscall_post__ksem_wait(res, id)                           \
1329 |   __sanitizer_syscall_post_impl__ksem_wait(res, (long long)(id))
1330 | #define __sanitizer_syscall_pre__ksem_trywait(id)                              \
```
- **Line 1321 / 第 1321 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1322 / 第 1322 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1323 / 第 1323 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1324 / 第 1324 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1325 / 第 1325 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1326 / 第 1326 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1327 / 第 1327 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1328 / 第 1328 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1329 / 第 1329 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1330 / 第 1330 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1331-1340 / 第 1331-1340 行
```cpp
1331 |   __sanitizer_syscall_pre_impl__ksem_trywait((long long)(id))
1332 | #define __sanitizer_syscall_post__ksem_trywait(res, id)                        \
1333 |   __sanitizer_syscall_post_impl__ksem_trywait(res, (long long)(id))
1334 | #define __sanitizer_syscall_pre__ksem_getvalue(id, value)                      \
1335 |   __sanitizer_syscall_pre_impl__ksem_getvalue((long long)(id),                 \
1336 |                                               (long long)(value))
1337 | #define __sanitizer_syscall_post__ksem_getvalue(res, id, value)                \
1338 |   __sanitizer_syscall_post_impl__ksem_getvalue(res, (long long)(id),           \
1339 |                                                (long long)(value))
1340 | #define __sanitizer_syscall_pre__ksem_destroy(id)                              \
```
- **Line 1331 / 第 1331 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1332 / 第 1332 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
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
1341 |   __sanitizer_syscall_pre_impl__ksem_destroy((long long)(id))
1342 | #define __sanitizer_syscall_post__ksem_destroy(res, id)                        \
1343 |   __sanitizer_syscall_post_impl__ksem_destroy(res, (long long)(id))
1344 | #define __sanitizer_syscall_pre__ksem_timedwait(id, abstime)                   \
1345 |   __sanitizer_syscall_pre_impl__ksem_timedwait((long long)(id),                \
1346 |                                                (long long)(abstime))
1347 | #define __sanitizer_syscall_post__ksem_timedwait(res, id, abstime)             \
1348 |   __sanitizer_syscall_post_impl__ksem_timedwait(res, (long long)(id),          \
1349 |                                                 (long long)(abstime))
1350 | #define __sanitizer_syscall_pre_mq_open(name, oflag, mode, attr)               \
```
- **Line 1341 / 第 1341 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1342 / 第 1342 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1343 / 第 1343 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1344 / 第 1344 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1345 / 第 1345 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1346 / 第 1346 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1347 / 第 1347 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1348 / 第 1348 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1349 / 第 1349 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1350 / 第 1350 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1351-1360 / 第 1351-1360 行
```cpp
1351 |   __sanitizer_syscall_pre_impl_mq_open((long long)(name), (long long)(oflag),  \
1352 |                                        (long long)(mode), (long long)(attr))
1353 | #define __sanitizer_syscall_post_mq_open(res, name, oflag, mode, attr)         \
1354 |   __sanitizer_syscall_post_impl_mq_open(res, (long long)(name),                \
1355 |                                         (long long)(oflag), (long long)(mode), \
1356 |                                         (long long)(attr))
1357 | #define __sanitizer_syscall_pre_mq_close(mqdes)                                \
1358 |   __sanitizer_syscall_pre_impl_mq_close((long long)(mqdes))
1359 | #define __sanitizer_syscall_post_mq_close(res, mqdes)                          \
1360 |   __sanitizer_syscall_post_impl_mq_close(res, (long long)(mqdes))
```
- **Line 1351 / 第 1351 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1352 / 第 1352 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1353 / 第 1353 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1354 / 第 1354 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1355 / 第 1355 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1356 / 第 1356 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1357 / 第 1357 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1358 / 第 1358 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1359 / 第 1359 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1360 / 第 1360 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1361-1370 / 第 1361-1370 行
```cpp
1361 | #define __sanitizer_syscall_pre_mq_unlink(name)                                \
1362 |   __sanitizer_syscall_pre_impl_mq_unlink((long long)(name))
1363 | #define __sanitizer_syscall_post_mq_unlink(res, name)                          \
1364 |   __sanitizer_syscall_post_impl_mq_unlink(res, (long long)(name))
1365 | #define __sanitizer_syscall_pre_mq_getattr(mqdes, mqstat)                      \
1366 |   __sanitizer_syscall_pre_impl_mq_getattr((long long)(mqdes),                  \
1367 |                                           (long long)(mqstat))
1368 | #define __sanitizer_syscall_post_mq_getattr(res, mqdes, mqstat)                \
1369 |   __sanitizer_syscall_post_impl_mq_getattr(res, (long long)(mqdes),            \
1370 |                                            (long long)(mqstat))
```
- **Line 1361 / 第 1361 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1362 / 第 1362 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1363 / 第 1363 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1364 / 第 1364 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1365 / 第 1365 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1366 / 第 1366 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1367 / 第 1367 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1368 / 第 1368 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1369 / 第 1369 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1370 / 第 1370 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1371-1380 / 第 1371-1380 行
```cpp
1371 | #define __sanitizer_syscall_pre_mq_setattr(mqdes, mqstat, omqstat)             \
1372 |   __sanitizer_syscall_pre_impl_mq_setattr(                                     \
1373 |       (long long)(mqdes), (long long)(mqstat), (long long)(omqstat))
1374 | #define __sanitizer_syscall_post_mq_setattr(res, mqdes, mqstat, omqstat)       \
1375 |   __sanitizer_syscall_post_impl_mq_setattr(                                    \
1376 |       res, (long long)(mqdes), (long long)(mqstat), (long long)(omqstat))
1377 | #define __sanitizer_syscall_pre_mq_notify(mqdes, notification)                 \
1378 |   __sanitizer_syscall_pre_impl_mq_notify((long long)(mqdes),                   \
1379 |                                          (long long)(notification))
1380 | #define __sanitizer_syscall_post_mq_notify(res, mqdes, notification)           \
```
- **Line 1371 / 第 1371 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1372 / 第 1372 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1373 / 第 1373 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1374 / 第 1374 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1375 / 第 1375 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1376 / 第 1376 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1377 / 第 1377 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1378 / 第 1378 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1379 / 第 1379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1380 / 第 1380 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1381-1390 / 第 1381-1390 行
```cpp
1381 |   __sanitizer_syscall_post_impl_mq_notify(res, (long long)(mqdes),             \
1382 |                                           (long long)(notification))
1383 | #define __sanitizer_syscall_pre_mq_send(mqdes, msg_ptr, msg_len, msg_prio)     \
1384 |   __sanitizer_syscall_pre_impl_mq_send(                                        \
1385 |       (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),          \
1386 |       (long long)(msg_prio))
1387 | #define __sanitizer_syscall_post_mq_send(res, mqdes, msg_ptr, msg_len,         \
1388 |                                          msg_prio)                             \
1389 |   __sanitizer_syscall_post_impl_mq_send(                                       \
1390 |       res, (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),     \
```
- **Line 1381 / 第 1381 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1382 / 第 1382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1383 / 第 1383 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1384 / 第 1384 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1385 / 第 1385 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1386 / 第 1386 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1387 / 第 1387 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1388 / 第 1388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1389 / 第 1389 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1390 / 第 1390 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1391-1400 / 第 1391-1400 行
```cpp
1391 |       (long long)(msg_prio))
1392 | #define __sanitizer_syscall_pre_mq_receive(mqdes, msg_ptr, msg_len, msg_prio)  \
1393 |   __sanitizer_syscall_pre_impl_mq_receive(                                     \
1394 |       (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),          \
1395 |       (long long)(msg_prio))
1396 | #define __sanitizer_syscall_post_mq_receive(res, mqdes, msg_ptr, msg_len,      \
1397 |                                             msg_prio)                          \
1398 |   __sanitizer_syscall_post_impl_mq_receive(                                    \
1399 |       res, (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),     \
1400 |       (long long)(msg_prio))
```
- **Line 1391 / 第 1391 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1392 / 第 1392 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1393 / 第 1393 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1394 / 第 1394 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1395 / 第 1395 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1396 / 第 1396 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1397 / 第 1397 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1398 / 第 1398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1399 / 第 1399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1400 / 第 1400 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1401-1410 / 第 1401-1410 行
```cpp
1401 | #define __sanitizer_syscall_pre_compat_50_mq_timedsend(                        \
1402 |     mqdes, msg_ptr, msg_len, msg_prio, abs_timeout)                            \
1403 |   __sanitizer_syscall_pre_impl_compat_50_mq_timedsend(                         \
1404 |       (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),          \
1405 |       (long long)(msg_prio), (long long)(abs_timeout))
1406 | #define __sanitizer_syscall_post_compat_50_mq_timedsend(                       \
1407 |     res, mqdes, msg_ptr, msg_len, msg_prio, abs_timeout)                       \
1408 |   __sanitizer_syscall_post_impl_compat_50_mq_timedsend(                        \
1409 |       res, (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),     \
1410 |       (long long)(msg_prio), (long long)(abs_timeout))
```
- **Line 1401 / 第 1401 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1402 / 第 1402 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1403 / 第 1403 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1404 / 第 1404 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1405 / 第 1405 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1406 / 第 1406 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1407 / 第 1407 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1408 / 第 1408 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1409 / 第 1409 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1410 / 第 1410 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1411-1420 / 第 1411-1420 行
```cpp
1411 | #define __sanitizer_syscall_pre_compat_50_mq_timedreceive(                     \
1412 |     mqdes, msg_ptr, msg_len, msg_prio, abs_timeout)                            \
1413 |   __sanitizer_syscall_pre_impl_compat_50_mq_timedreceive(                      \
1414 |       (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),          \
1415 |       (long long)(msg_prio), (long long)(abs_timeout))
1416 | #define __sanitizer_syscall_post_compat_50_mq_timedreceive(                    \
1417 |     res, mqdes, msg_ptr, msg_len, msg_prio, abs_timeout)                       \
1418 |   __sanitizer_syscall_post_impl_compat_50_mq_timedreceive(                     \
1419 |       res, (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),     \
1420 |       (long long)(msg_prio), (long long)(abs_timeout))
```
- **Line 1411 / 第 1411 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1412 / 第 1412 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1413 / 第 1413 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1414 / 第 1414 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1415 / 第 1415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1416 / 第 1416 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1417 / 第 1417 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1418 / 第 1418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1419 / 第 1419 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1420 / 第 1420 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1421-1430 / 第 1421-1430 行
```cpp
1421 | /* syscall 267 has been skipped */
1422 | /* syscall 268 has been skipped */
1423 | /* syscall 269 has been skipped */
1424 | #define __sanitizer_syscall_pre___posix_rename(from, to)                       \
1425 |   __sanitizer_syscall_pre_impl___posix_rename((long long)(from),               \
1426 |                                               (long long)(to))
1427 | #define __sanitizer_syscall_post___posix_rename(res, from, to)                 \
1428 |   __sanitizer_syscall_post_impl___posix_rename(res, (long long)(from),         \
1429 |                                                (long long)(to))
1430 | #define __sanitizer_syscall_pre_swapctl(cmd, arg, misc)                        \
```
- **Line 1421 / 第 1421 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1422 / 第 1422 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1423 / 第 1423 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1424 / 第 1424 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1425 / 第 1425 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1426 / 第 1426 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1427 / 第 1427 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1428 / 第 1428 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1429 / 第 1429 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1430 / 第 1430 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1431-1440 / 第 1431-1440 行
```cpp
1431 |   __sanitizer_syscall_pre_impl_swapctl((long long)(cmd), (long long)(arg),     \
1432 |                                        (long long)(misc))
1433 | #define __sanitizer_syscall_post_swapctl(res, cmd, arg, misc)                  \
1434 |   __sanitizer_syscall_post_impl_swapctl(res, (long long)(cmd),                 \
1435 |                                         (long long)(arg), (long long)(misc))
1436 | #define __sanitizer_syscall_pre_compat_30_getdents(fd, buf, count)             \
1437 |   __sanitizer_syscall_pre_impl_compat_30_getdents(                             \
1438 |       (long long)(fd), (long long)(buf), (long long)(count))
1439 | #define __sanitizer_syscall_post_compat_30_getdents(res, fd, buf, count)       \
1440 |   __sanitizer_syscall_post_impl_compat_30_getdents(                            \
```
- **Line 1431 / 第 1431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1432 / 第 1432 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1433 / 第 1433 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1434 / 第 1434 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1435 / 第 1435 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1436 / 第 1436 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1437 / 第 1437 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1438 / 第 1438 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1439 / 第 1439 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1440 / 第 1440 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1441-1450 / 第 1441-1450 行
```cpp
1441 |       res, (long long)(fd), (long long)(buf), (long long)(count))
1442 | #define __sanitizer_syscall_pre_minherit(addr, len, inherit)                   \
1443 |   __sanitizer_syscall_pre_impl_minherit((long long)(addr), (long long)(len),   \
1444 |                                         (long long)(inherit))
1445 | #define __sanitizer_syscall_post_minherit(res, addr, len, inherit)             \
1446 |   __sanitizer_syscall_post_impl_minherit(                                      \
1447 |       res, (long long)(addr), (long long)(len), (long long)(inherit))
1448 | #define __sanitizer_syscall_pre_lchmod(path, mode)                             \
1449 |   __sanitizer_syscall_pre_impl_lchmod((long long)(path), (long long)(mode))
1450 | #define __sanitizer_syscall_post_lchmod(res, path, mode)                       \
```
- **Line 1441 / 第 1441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1442 / 第 1442 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1443 / 第 1443 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1444 / 第 1444 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1445 / 第 1445 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1446 / 第 1446 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1447 / 第 1447 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1448 / 第 1448 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1449 / 第 1449 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1450 / 第 1450 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1451-1460 / 第 1451-1460 行
```cpp
1451 |   __sanitizer_syscall_post_impl_lchmod(res, (long long)(path),                 \
1452 |                                        (long long)(mode))
1453 | #define __sanitizer_syscall_pre_lchown(path, uid, gid)                         \
1454 |   __sanitizer_syscall_pre_impl_lchown((long long)(path), (long long)(uid),     \
1455 |                                       (long long)(gid))
1456 | #define __sanitizer_syscall_post_lchown(res, path, uid, gid)                   \
1457 |   __sanitizer_syscall_post_impl_lchown(res, (long long)(path),                 \
1458 |                                        (long long)(uid), (long long)(gid))
1459 | #define __sanitizer_syscall_pre_compat_50_lutimes(path, tptr)                  \
1460 |   __sanitizer_syscall_pre_impl_compat_50_lutimes((long long)(path),            \
```
- **Line 1451 / 第 1451 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1452 / 第 1452 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1453 / 第 1453 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1454 / 第 1454 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1455 / 第 1455 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1456 / 第 1456 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1457 / 第 1457 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1458 / 第 1458 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1459 / 第 1459 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1460 / 第 1460 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1461-1470 / 第 1461-1470 行
```cpp
1461 |                                                  (long long)(tptr))
1462 | #define __sanitizer_syscall_post_compat_50_lutimes(res, path, tptr)            \
1463 |   __sanitizer_syscall_post_impl_compat_50_lutimes(res, (long long)(path),      \
1464 |                                                   (long long)(tptr))
1465 | #define __sanitizer_syscall_pre___msync13(addr, len, flags)                    \
1466 |   __sanitizer_syscall_pre_impl___msync13((long long)(addr), (long long)(len),  \
1467 |                                          (long long)(flags))
1468 | #define __sanitizer_syscall_post___msync13(res, addr, len, flags)              \
1469 |   __sanitizer_syscall_post_impl___msync13(                                     \
1470 |       res, (long long)(addr), (long long)(len), (long long)(flags))
```
- **Line 1461 / 第 1461 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1462 / 第 1462 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1463 / 第 1463 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1464 / 第 1464 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1465 / 第 1465 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1466 / 第 1466 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1467 / 第 1467 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1468 / 第 1468 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1469 / 第 1469 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1470 / 第 1470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1471-1480 / 第 1471-1480 行
```cpp
1471 | #define __sanitizer_syscall_pre_compat_30___stat13(path, ub)                   \
1472 |   __sanitizer_syscall_pre_impl_compat_30___stat13((long long)(path),           \
1473 |                                                   (long long)(ub))
1474 | #define __sanitizer_syscall_post_compat_30___stat13(res, path, ub)             \
1475 |   __sanitizer_syscall_post_impl_compat_30___stat13(res, (long long)(path),     \
1476 |                                                    (long long)(ub))
1477 | #define __sanitizer_syscall_pre_compat_30___fstat13(fd, sb)                    \
1478 |   __sanitizer_syscall_pre_impl_compat_30___fstat13((long long)(fd),            \
1479 |                                                    (long long)(sb))
1480 | #define __sanitizer_syscall_post_compat_30___fstat13(res, fd, sb)              \
```
- **Line 1471 / 第 1471 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1472 / 第 1472 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1473 / 第 1473 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1474 / 第 1474 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1475 / 第 1475 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1476 / 第 1476 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1477 / 第 1477 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1478 / 第 1478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1479 / 第 1479 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1480 / 第 1480 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1481-1490 / 第 1481-1490 行
```cpp
1481 |   __sanitizer_syscall_post_impl_compat_30___fstat13(res, (long long)(fd),      \
1482 |                                                     (long long)(sb))
1483 | #define __sanitizer_syscall_pre_compat_30___lstat13(path, ub)                  \
1484 |   __sanitizer_syscall_pre_impl_compat_30___lstat13((long long)(path),          \
1485 |                                                    (long long)(ub))
1486 | #define __sanitizer_syscall_post_compat_30___lstat13(res, path, ub)            \
1487 |   __sanitizer_syscall_post_impl_compat_30___lstat13(res, (long long)(path),    \
1488 |                                                     (long long)(ub))
1489 | #define __sanitizer_syscall_pre___sigaltstack14(nss, oss)                      \
1490 |   __sanitizer_syscall_pre_impl___sigaltstack14((long long)(nss),               \
```
- **Line 1481 / 第 1481 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1482 / 第 1482 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1483 / 第 1483 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1484 / 第 1484 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1485 / 第 1485 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1486 / 第 1486 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1487 / 第 1487 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1488 / 第 1488 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1489 / 第 1489 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1490 / 第 1490 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1491-1500 / 第 1491-1500 行
```cpp
1491 |                                                (long long)(oss))
1492 | #define __sanitizer_syscall_post___sigaltstack14(res, nss, oss)                \
1493 |   __sanitizer_syscall_post_impl___sigaltstack14(res, (long long)(nss),         \
1494 |                                                 (long long)(oss))
1495 | #define __sanitizer_syscall_pre___vfork14()                                    \
1496 |   __sanitizer_syscall_pre_impl___vfork14()
1497 | #define __sanitizer_syscall_post___vfork14(res)                                \
1498 |   __sanitizer_syscall_post_impl___vfork14(res)
1499 | #define __sanitizer_syscall_pre___posix_chown(path, uid, gid)                  \
1500 |   __sanitizer_syscall_pre_impl___posix_chown(                                  \
```
- **Line 1491 / 第 1491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1492 / 第 1492 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1493 / 第 1493 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1494 / 第 1494 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1495 / 第 1495 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1496 / 第 1496 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1497 / 第 1497 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1498 / 第 1498 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1499 / 第 1499 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1500 / 第 1500 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1501-1510 / 第 1501-1510 行
```cpp
1501 |       (long long)(path), (long long)(uid), (long long)(gid))
1502 | #define __sanitizer_syscall_post___posix_chown(res, path, uid, gid)            \
1503 |   __sanitizer_syscall_post_impl___posix_chown(                                 \
1504 |       res, (long long)(path), (long long)(uid), (long long)(gid))
1505 | #define __sanitizer_syscall_pre___posix_fchown(fd, uid, gid)                   \
1506 |   __sanitizer_syscall_pre_impl___posix_fchown(                                 \
1507 |       (long long)(fd), (long long)(uid), (long long)(gid))
1508 | #define __sanitizer_syscall_post___posix_fchown(res, fd, uid, gid)             \
1509 |   __sanitizer_syscall_post_impl___posix_fchown(                                \
1510 |       res, (long long)(fd), (long long)(uid), (long long)(gid))
```
- **Line 1501 / 第 1501 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1502 / 第 1502 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1503 / 第 1503 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1504 / 第 1504 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1505 / 第 1505 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1506 / 第 1506 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1507 / 第 1507 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1508 / 第 1508 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1509 / 第 1509 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1510 / 第 1510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1511-1520 / 第 1511-1520 行
```cpp
1511 | #define __sanitizer_syscall_pre___posix_lchown(path, uid, gid)                 \
1512 |   __sanitizer_syscall_pre_impl___posix_lchown(                                 \
1513 |       (long long)(path), (long long)(uid), (long long)(gid))
1514 | #define __sanitizer_syscall_post___posix_lchown(res, path, uid, gid)           \
1515 |   __sanitizer_syscall_post_impl___posix_lchown(                                \
1516 |       res, (long long)(path), (long long)(uid), (long long)(gid))
1517 | #define __sanitizer_syscall_pre_getsid(pid)                                    \
1518 |   __sanitizer_syscall_pre_impl_getsid((long long)(pid))
1519 | #define __sanitizer_syscall_post_getsid(res, pid)                              \
1520 |   __sanitizer_syscall_post_impl_getsid(res, (long long)(pid))
```
- **Line 1511 / 第 1511 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1512 / 第 1512 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1513 / 第 1513 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1514 / 第 1514 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1515 / 第 1515 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1516 / 第 1516 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1517 / 第 1517 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1518 / 第 1518 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1519 / 第 1519 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1520 / 第 1520 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1521-1530 / 第 1521-1530 行
```cpp
1521 | #define __sanitizer_syscall_pre___clone(flags, stack)                          \
1522 |   __sanitizer_syscall_pre_impl___clone((long long)(flags), (long long)(stack))
1523 | #define __sanitizer_syscall_post___clone(res, flags, stack)                    \
1524 |   __sanitizer_syscall_post_impl___clone(res, (long long)(flags),               \
1525 |                                         (long long)(stack))
1526 | #define __sanitizer_syscall_pre_fktrace(fd, ops, facs, pid)                    \
1527 |   __sanitizer_syscall_pre_impl_fktrace((long long)(fd), (long long)(ops),      \
1528 |                                        (long long)(facs), (long long)(pid))
1529 | #define __sanitizer_syscall_post_fktrace(res, fd, ops, facs, pid)              \
1530 |   __sanitizer_syscall_post_impl_fktrace(res, (long long)(fd),                  \
```
- **Line 1521 / 第 1521 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1522 / 第 1522 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1523 / 第 1523 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1524 / 第 1524 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1525 / 第 1525 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1526 / 第 1526 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1527 / 第 1527 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1528 / 第 1528 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1529 / 第 1529 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1530 / 第 1530 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1531-1540 / 第 1531-1540 行
```cpp
1531 |                                         (long long)(ops), (long long)(facs),   \
1532 |                                         (long long)(pid))
1533 | #define __sanitizer_syscall_pre_preadv(fd, iovp, iovcnt, PAD, offset)          \
1534 |   __sanitizer_syscall_pre_impl_preadv((long long)(fd), (long long)(iovp),      \
1535 |                                       (long long)(iovcnt), (long long)(PAD),   \
1536 |                                       (long long)(offset))
1537 | #define __sanitizer_syscall_post_preadv(res, fd, iovp, iovcnt, PAD, offset)    \
1538 |   __sanitizer_syscall_post_impl_preadv(res, (long long)(fd),                   \
1539 |                                        (long long)(iovp), (long long)(iovcnt), \
1540 |                                        (long long)(PAD), (long long)(offset))
```
- **Line 1531 / 第 1531 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1532 / 第 1532 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1533 / 第 1533 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1534 / 第 1534 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1535 / 第 1535 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1536 / 第 1536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1537 / 第 1537 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1538 / 第 1538 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1539 / 第 1539 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1540 / 第 1540 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1541-1550 / 第 1541-1550 行
```cpp
1541 | #define __sanitizer_syscall_pre_pwritev(fd, iovp, iovcnt, PAD, offset)         \
1542 |   __sanitizer_syscall_pre_impl_pwritev((long long)(fd), (long long)(iovp),     \
1543 |                                        (long long)(iovcnt), (long long)(PAD),  \
1544 |                                        (long long)(offset))
1545 | #define __sanitizer_syscall_post_pwritev(res, fd, iovp, iovcnt, PAD, offset)   \
1546 |   __sanitizer_syscall_post_impl_pwritev(                                       \
1547 |       res, (long long)(fd), (long long)(iovp), (long long)(iovcnt),            \
1548 |       (long long)(PAD), (long long)(offset))
1549 | #define __sanitizer_syscall_pre_compat_16___sigaction14(signum, nsa, osa)      \
1550 |   __sanitizer_syscall_pre_impl_compat_16___sigaction14(                        \
```
- **Line 1541 / 第 1541 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1542 / 第 1542 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1543 / 第 1543 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1544 / 第 1544 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1545 / 第 1545 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1546 / 第 1546 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1547 / 第 1547 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1548 / 第 1548 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1549 / 第 1549 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1550 / 第 1550 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1551-1560 / 第 1551-1560 行
```cpp
1551 |       (long long)(signum), (long long)(nsa), (long long)(osa))
1552 | #define __sanitizer_syscall_post_compat_16___sigaction14(res, signum, nsa,     \
1553 |                                                          osa)                  \
1554 |   __sanitizer_syscall_post_impl_compat_16___sigaction14(                       \
1555 |       res, (long long)(signum), (long long)(nsa), (long long)(osa))
1556 | #define __sanitizer_syscall_pre___sigpending14(set)                            \
1557 |   __sanitizer_syscall_pre_impl___sigpending14((long long)(set))
1558 | #define __sanitizer_syscall_post___sigpending14(res, set)                      \
1559 |   __sanitizer_syscall_post_impl___sigpending14(res, (long long)(set))
1560 | #define __sanitizer_syscall_pre___sigprocmask14(how, set, oset)                \
```
- **Line 1551 / 第 1551 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1552 / 第 1552 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1553 / 第 1553 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1554 / 第 1554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1555 / 第 1555 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1556 / 第 1556 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1557 / 第 1557 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1558 / 第 1558 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1559 / 第 1559 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1560 / 第 1560 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1561-1570 / 第 1561-1570 行
```cpp
1561 |   __sanitizer_syscall_pre_impl___sigprocmask14(                                \
1562 |       (long long)(how), (long long)(set), (long long)(oset))
1563 | #define __sanitizer_syscall_post___sigprocmask14(res, how, set, oset)          \
1564 |   __sanitizer_syscall_post_impl___sigprocmask14(                               \
1565 |       res, (long long)(how), (long long)(set), (long long)(oset))
1566 | #define __sanitizer_syscall_pre___sigsuspend14(set)                            \
1567 |   __sanitizer_syscall_pre_impl___sigsuspend14((long long)(set))
1568 | #define __sanitizer_syscall_post___sigsuspend14(res, set)                      \
1569 |   __sanitizer_syscall_post_impl___sigsuspend14(res, (long long)(set))
1570 | #define __sanitizer_syscall_pre_compat_16___sigreturn14(sigcntxp)              \
```
- **Line 1561 / 第 1561 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1562 / 第 1562 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1563 / 第 1563 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1564 / 第 1564 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1565 / 第 1565 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1566 / 第 1566 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1567 / 第 1567 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1568 / 第 1568 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1569 / 第 1569 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1570 / 第 1570 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1571-1580 / 第 1571-1580 行
```cpp
1571 |   __sanitizer_syscall_pre_impl_compat_16___sigreturn14((long long)(sigcntxp))
1572 | #define __sanitizer_syscall_post_compat_16___sigreturn14(res, sigcntxp)        \
1573 |   __sanitizer_syscall_post_impl_compat_16___sigreturn14(res,                   \
1574 |                                                         (long long)(sigcntxp))
1575 | #define __sanitizer_syscall_pre___getcwd(bufp, length)                         \
1576 |   __sanitizer_syscall_pre_impl___getcwd((long long)(bufp), (long long)(length))
1577 | #define __sanitizer_syscall_post___getcwd(res, bufp, length)                   \
1578 |   __sanitizer_syscall_post_impl___getcwd(res, (long long)(bufp),               \
1579 |                                          (long long)(length))
1580 | #define __sanitizer_syscall_pre_fchroot(fd)                                    \
```
- **Line 1571 / 第 1571 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1572 / 第 1572 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1573 / 第 1573 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1574 / 第 1574 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1575 / 第 1575 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1576 / 第 1576 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1577 / 第 1577 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1578 / 第 1578 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1579 / 第 1579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1580 / 第 1580 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1581-1590 / 第 1581-1590 行
```cpp
1581 |   __sanitizer_syscall_pre_impl_fchroot((long long)(fd))
1582 | #define __sanitizer_syscall_post_fchroot(res, fd)                              \
1583 |   __sanitizer_syscall_post_impl_fchroot(res, (long long)(fd))
1584 | #define __sanitizer_syscall_pre_compat_30_fhopen(fhp, flags)                   \
1585 |   __sanitizer_syscall_pre_impl_compat_30_fhopen((long long)(fhp),              \
1586 |                                                 (long long)(flags))
1587 | #define __sanitizer_syscall_post_compat_30_fhopen(res, fhp, flags)             \
1588 |   __sanitizer_syscall_post_impl_compat_30_fhopen(res, (long long)(fhp),        \
1589 |                                                  (long long)(flags))
1590 | #define __sanitizer_syscall_pre_compat_30_fhstat(fhp, sb)                      \
```
- **Line 1581 / 第 1581 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1582 / 第 1582 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1583 / 第 1583 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1584 / 第 1584 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1585 / 第 1585 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1586 / 第 1586 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1587 / 第 1587 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1588 / 第 1588 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1589 / 第 1589 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1590 / 第 1590 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1591-1600 / 第 1591-1600 行
```cpp
1591 |   __sanitizer_syscall_pre_impl_compat_30_fhstat((long long)(fhp),              \
1592 |                                                 (long long)(sb))
1593 | #define __sanitizer_syscall_post_compat_30_fhstat(res, fhp, sb)                \
1594 |   __sanitizer_syscall_post_impl_compat_30_fhstat(res, (long long)(fhp),        \
1595 |                                                  (long long)(sb))
1596 | #define __sanitizer_syscall_pre_compat_20_fhstatfs(fhp, buf)                   \
1597 |   __sanitizer_syscall_pre_impl_compat_20_fhstatfs((long long)(fhp),            \
1598 |                                                   (long long)(buf))
1599 | #define __sanitizer_syscall_post_compat_20_fhstatfs(res, fhp, buf)             \
1600 |   __sanitizer_syscall_post_impl_compat_20_fhstatfs(res, (long long)(fhp),      \
```
- **Line 1591 / 第 1591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1592 / 第 1592 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1593 / 第 1593 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1594 / 第 1594 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1595 / 第 1595 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1596 / 第 1596 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1597 / 第 1597 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1598 / 第 1598 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1599 / 第 1599 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1600 / 第 1600 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1601-1610 / 第 1601-1610 行
```cpp
1601 |                                                    (long long)(buf))
1602 | #define __sanitizer_syscall_pre_compat_50_____semctl13(semid, semnum, cmd,     \
1603 |                                                        arg)                    \
1604 |   __sanitizer_syscall_pre_impl_compat_50_____semctl13(                         \
1605 |       (long long)(semid), (long long)(semnum), (long long)(cmd),               \
1606 |       (long long)(arg))
1607 | #define __sanitizer_syscall_post_compat_50_____semctl13(res, semid, semnum,    \
1608 |                                                         cmd, arg)              \
1609 |   __sanitizer_syscall_post_impl_compat_50_____semctl13(                        \
1610 |       res, (long long)(semid), (long long)(semnum), (long long)(cmd),          \
```
- **Line 1601 / 第 1601 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1602 / 第 1602 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1603 / 第 1603 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1604 / 第 1604 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1605 / 第 1605 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1606 / 第 1606 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1607 / 第 1607 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1608 / 第 1608 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1609 / 第 1609 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1610 / 第 1610 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1611-1620 / 第 1611-1620 行
```cpp
1611 |       (long long)(arg))
1612 | #define __sanitizer_syscall_pre_compat_50___msgctl13(msqid, cmd, buf)          \
1613 |   __sanitizer_syscall_pre_impl_compat_50___msgctl13(                           \
1614 |       (long long)(msqid), (long long)(cmd), (long long)(buf))
1615 | #define __sanitizer_syscall_post_compat_50___msgctl13(res, msqid, cmd, buf)    \
1616 |   __sanitizer_syscall_post_impl_compat_50___msgctl13(                          \
1617 |       res, (long long)(msqid), (long long)(cmd), (long long)(buf))
1618 | #define __sanitizer_syscall_pre_compat_50___shmctl13(shmid, cmd, buf)          \
1619 |   __sanitizer_syscall_pre_impl_compat_50___shmctl13(                           \
1620 |       (long long)(shmid), (long long)(cmd), (long long)(buf))
```
- **Line 1611 / 第 1611 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1612 / 第 1612 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
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
1621 | #define __sanitizer_syscall_post_compat_50___shmctl13(res, shmid, cmd, buf)    \
1622 |   __sanitizer_syscall_post_impl_compat_50___shmctl13(                          \
1623 |       res, (long long)(shmid), (long long)(cmd), (long long)(buf))
1624 | #define __sanitizer_syscall_pre_lchflags(path, flags)                          \
1625 |   __sanitizer_syscall_pre_impl_lchflags((long long)(path), (long long)(flags))
1626 | #define __sanitizer_syscall_post_lchflags(res, path, flags)                    \
1627 |   __sanitizer_syscall_post_impl_lchflags(res, (long long)(path),               \
1628 |                                          (long long)(flags))
1629 | #define __sanitizer_syscall_pre_issetugid()                                    \
1630 |   __sanitizer_syscall_pre_impl_issetugid()
```
- **Line 1621 / 第 1621 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1622 / 第 1622 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1623 / 第 1623 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1624 / 第 1624 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1625 / 第 1625 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1626 / 第 1626 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1627 / 第 1627 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1628 / 第 1628 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1629 / 第 1629 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1630 / 第 1630 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1631-1640 / 第 1631-1640 行
```cpp
1631 | #define __sanitizer_syscall_post_issetugid(res)                                \
1632 |   __sanitizer_syscall_post_impl_issetugid(res)
1633 | #define __sanitizer_syscall_pre_utrace(label, addr, len)                       \
1634 |   __sanitizer_syscall_pre_impl_utrace((long long)(label), (long long)(addr),   \
1635 |                                       (long long)(len))
1636 | #define __sanitizer_syscall_post_utrace(res, label, addr, len)                 \
1637 |   __sanitizer_syscall_post_impl_utrace(res, (long long)(label),                \
1638 |                                        (long long)(addr), (long long)(len))
1639 | #define __sanitizer_syscall_pre_getcontext(ucp)                                \
1640 |   __sanitizer_syscall_pre_impl_getcontext((long long)(ucp))
```
- **Line 1631 / 第 1631 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1632 / 第 1632 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1633 / 第 1633 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1634 / 第 1634 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1635 / 第 1635 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1636 / 第 1636 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1637 / 第 1637 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1638 / 第 1638 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1639 / 第 1639 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1640 / 第 1640 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1641-1650 / 第 1641-1650 行
```cpp
1641 | #define __sanitizer_syscall_post_getcontext(res, ucp)                          \
1642 |   __sanitizer_syscall_post_impl_getcontext(res, (long long)(ucp))
1643 | #define __sanitizer_syscall_pre_setcontext(ucp)                                \
1644 |   __sanitizer_syscall_pre_impl_setcontext((long long)(ucp))
1645 | #define __sanitizer_syscall_post_setcontext(res, ucp)                          \
1646 |   __sanitizer_syscall_post_impl_setcontext(res, (long long)(ucp))
1647 | #define __sanitizer_syscall_pre__lwp_create(ucp, flags, new_lwp)               \
1648 |   __sanitizer_syscall_pre_impl__lwp_create(                                    \
1649 |       (long long)(ucp), (long long)(flags), (long long)(new_lwp))
1650 | #define __sanitizer_syscall_post__lwp_create(res, ucp, flags, new_lwp)         \
```
- **Line 1641 / 第 1641 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1642 / 第 1642 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1643 / 第 1643 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1644 / 第 1644 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1645 / 第 1645 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1646 / 第 1646 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1647 / 第 1647 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1648 / 第 1648 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1649 / 第 1649 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1650 / 第 1650 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1651-1660 / 第 1651-1660 行
```cpp
1651 |   __sanitizer_syscall_post_impl__lwp_create(                                   \
1652 |       res, (long long)(ucp), (long long)(flags), (long long)(new_lwp))
1653 | #define __sanitizer_syscall_pre__lwp_exit()                                    \
1654 |   __sanitizer_syscall_pre_impl__lwp_exit()
1655 | #define __sanitizer_syscall_post__lwp_exit(res)                                \
1656 |   __sanitizer_syscall_post_impl__lwp_exit(res)
1657 | #define __sanitizer_syscall_pre__lwp_self()                                    \
1658 |   __sanitizer_syscall_pre_impl__lwp_self()
1659 | #define __sanitizer_syscall_post__lwp_self(res)                                \
1660 |   __sanitizer_syscall_post_impl__lwp_self(res)
```
- **Line 1651 / 第 1651 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1652 / 第 1652 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1653 / 第 1653 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1654 / 第 1654 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1655 / 第 1655 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1656 / 第 1656 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1657 / 第 1657 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1658 / 第 1658 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1659 / 第 1659 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1660 / 第 1660 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1661-1670 / 第 1661-1670 行
```cpp
1661 | #define __sanitizer_syscall_pre__lwp_wait(wait_for, departed)                  \
1662 |   __sanitizer_syscall_pre_impl__lwp_wait((long long)(wait_for),                \
1663 |                                          (long long)(departed))
1664 | #define __sanitizer_syscall_post__lwp_wait(res, wait_for, departed)            \
1665 |   __sanitizer_syscall_post_impl__lwp_wait(res, (long long)(wait_for),          \
1666 |                                           (long long)(departed))
1667 | #define __sanitizer_syscall_pre__lwp_suspend(target)                           \
1668 |   __sanitizer_syscall_pre_impl__lwp_suspend((long long)(target))
1669 | #define __sanitizer_syscall_post__lwp_suspend(res, target)                     \
1670 |   __sanitizer_syscall_post_impl__lwp_suspend(res, (long long)(target))
```
- **Line 1661 / 第 1661 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1662 / 第 1662 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1663 / 第 1663 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1664 / 第 1664 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1665 / 第 1665 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1666 / 第 1666 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1667 / 第 1667 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1668 / 第 1668 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1669 / 第 1669 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1670 / 第 1670 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1671-1680 / 第 1671-1680 行
```cpp
1671 | #define __sanitizer_syscall_pre__lwp_continue(target)                          \
1672 |   __sanitizer_syscall_pre_impl__lwp_continue((long long)(target))
1673 | #define __sanitizer_syscall_post__lwp_continue(res, target)                    \
1674 |   __sanitizer_syscall_post_impl__lwp_continue(res, (long long)(target))
1675 | #define __sanitizer_syscall_pre__lwp_wakeup(target)                            \
1676 |   __sanitizer_syscall_pre_impl__lwp_wakeup((long long)(target))
1677 | #define __sanitizer_syscall_post__lwp_wakeup(res, target)                      \
1678 |   __sanitizer_syscall_post_impl__lwp_wakeup(res, (long long)(target))
1679 | #define __sanitizer_syscall_pre__lwp_getprivate()                              \
1680 |   __sanitizer_syscall_pre_impl__lwp_getprivate()
```
- **Line 1671 / 第 1671 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1672 / 第 1672 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1673 / 第 1673 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1674 / 第 1674 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1675 / 第 1675 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1676 / 第 1676 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1677 / 第 1677 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1678 / 第 1678 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1679 / 第 1679 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1680 / 第 1680 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1681-1690 / 第 1681-1690 行
```cpp
1681 | #define __sanitizer_syscall_post__lwp_getprivate(res)                          \
1682 |   __sanitizer_syscall_post_impl__lwp_getprivate(res)
1683 | #define __sanitizer_syscall_pre__lwp_setprivate(ptr)                           \
1684 |   __sanitizer_syscall_pre_impl__lwp_setprivate((long long)(ptr))
1685 | #define __sanitizer_syscall_post__lwp_setprivate(res, ptr)                     \
1686 |   __sanitizer_syscall_post_impl__lwp_setprivate(res, (long long)(ptr))
1687 | #define __sanitizer_syscall_pre__lwp_kill(target, signo)                       \
1688 |   __sanitizer_syscall_pre_impl__lwp_kill((long long)(target),                  \
1689 |                                          (long long)(signo))
1690 | #define __sanitizer_syscall_post__lwp_kill(res, target, signo)                 \
```
- **Line 1681 / 第 1681 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1682 / 第 1682 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1683 / 第 1683 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1684 / 第 1684 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1685 / 第 1685 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1686 / 第 1686 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1687 / 第 1687 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1688 / 第 1688 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1689 / 第 1689 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1690 / 第 1690 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1691-1700 / 第 1691-1700 行
```cpp
1691 |   __sanitizer_syscall_post_impl__lwp_kill(res, (long long)(target),            \
1692 |                                           (long long)(signo))
1693 | #define __sanitizer_syscall_pre__lwp_detach(target)                            \
1694 |   __sanitizer_syscall_pre_impl__lwp_detach((long long)(target))
1695 | #define __sanitizer_syscall_post__lwp_detach(res, target)                      \
1696 |   __sanitizer_syscall_post_impl__lwp_detach(res, (long long)(target))
1697 | #define __sanitizer_syscall_pre_compat_50__lwp_park(ts, unpark, hint,          \
1698 |                                                     unparkhint)                \
1699 |   __sanitizer_syscall_pre_impl_compat_50__lwp_park(                            \
1700 |       (long long)(ts), (long long)(unpark), (long long)(hint),                 \
```
- **Line 1691 / 第 1691 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1692 / 第 1692 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1693 / 第 1693 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1694 / 第 1694 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1695 / 第 1695 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1696 / 第 1696 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1697 / 第 1697 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1698 / 第 1698 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1699 / 第 1699 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1700 / 第 1700 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1701-1710 / 第 1701-1710 行
```cpp
1701 |       (long long)(unparkhint))
1702 | #define __sanitizer_syscall_post_compat_50__lwp_park(res, ts, unpark, hint,    \
1703 |                                                      unparkhint)               \
1704 |   __sanitizer_syscall_post_impl_compat_50__lwp_park(                           \
1705 |       res, (long long)(ts), (long long)(unpark), (long long)(hint),            \
1706 |       (long long)(unparkhint))
1707 | #define __sanitizer_syscall_pre__lwp_unpark(target, hint)                      \
1708 |   __sanitizer_syscall_pre_impl__lwp_unpark((long long)(target),                \
1709 |                                            (long long)(hint))
1710 | #define __sanitizer_syscall_post__lwp_unpark(res, target, hint)                \
```
- **Line 1701 / 第 1701 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1702 / 第 1702 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1703 / 第 1703 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1704 / 第 1704 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1705 / 第 1705 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1706 / 第 1706 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1707 / 第 1707 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1708 / 第 1708 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1709 / 第 1709 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1710 / 第 1710 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1711-1720 / 第 1711-1720 行
```cpp
1711 |   __sanitizer_syscall_post_impl__lwp_unpark(res, (long long)(target),          \
1712 |                                             (long long)(hint))
1713 | #define __sanitizer_syscall_pre__lwp_unpark_all(targets, ntargets, hint)       \
1714 |   __sanitizer_syscall_pre_impl__lwp_unpark_all(                                \
1715 |       (long long)(targets), (long long)(ntargets), (long long)(hint))
1716 | #define __sanitizer_syscall_post__lwp_unpark_all(res, targets, ntargets, hint) \
1717 |   __sanitizer_syscall_post_impl__lwp_unpark_all(                               \
1718 |       res, (long long)(targets), (long long)(ntargets), (long long)(hint))
1719 | #define __sanitizer_syscall_pre__lwp_setname(target, name)                     \
1720 |   __sanitizer_syscall_pre_impl__lwp_setname((long long)(target),               \
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
1721 |                                             (long long)(name))
1722 | #define __sanitizer_syscall_post__lwp_setname(res, target, name)               \
1723 |   __sanitizer_syscall_post_impl__lwp_setname(res, (long long)(target),         \
1724 |                                              (long long)(name))
1725 | #define __sanitizer_syscall_pre__lwp_getname(target, name, len)                \
1726 |   __sanitizer_syscall_pre_impl__lwp_getname(                                   \
1727 |       (long long)(target), (long long)(name), (long long)(len))
1728 | #define __sanitizer_syscall_post__lwp_getname(res, target, name, len)          \
1729 |   __sanitizer_syscall_post_impl__lwp_getname(                                  \
1730 |       res, (long long)(target), (long long)(name), (long long)(len))
```
- **Line 1721 / 第 1721 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1722 / 第 1722 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1723 / 第 1723 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1724 / 第 1724 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1725 / 第 1725 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1726 / 第 1726 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1727 / 第 1727 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1728 / 第 1728 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1729 / 第 1729 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1730 / 第 1730 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1731-1740 / 第 1731-1740 行
```cpp
1731 | #define __sanitizer_syscall_pre__lwp_ctl(features, address)                    \
1732 |   __sanitizer_syscall_pre_impl__lwp_ctl((long long)(features),                 \
1733 |                                         (long long)(address))
1734 | #define __sanitizer_syscall_post__lwp_ctl(res, features, address)              \
1735 |   __sanitizer_syscall_post_impl__lwp_ctl(res, (long long)(features),           \
1736 |                                          (long long)(address))
1737 | /* syscall 326 has been skipped */
1738 | /* syscall 327 has been skipped */
1739 | /* syscall 328 has been skipped */
1740 | /* syscall 329 has been skipped */
```
- **Line 1731 / 第 1731 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1732 / 第 1732 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1733 / 第 1733 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1734 / 第 1734 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1735 / 第 1735 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1736 / 第 1736 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1737 / 第 1737 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1738 / 第 1738 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1739 / 第 1739 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1740 / 第 1740 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 1741-1750 / 第 1741-1750 行
```cpp
1741 | #define __sanitizer_syscall_pre_compat_60_sa_register(newv, oldv, flags,       \
1742 |                                                       stackinfo_offset)        \
1743 |   __sanitizer_syscall_pre_impl_compat_60_sa_register(                          \
1744 |       (long long)(newv), (long long)(oldv), (long long)(flags),                \
1745 |       (long long)(stackinfo_offset))
1746 | #define __sanitizer_syscall_post_compat_60_sa_register(res, newv, oldv, flags, \
1747 |                                                        stackinfo_offset)       \
1748 |   __sanitizer_syscall_post_impl_compat_60_sa_register(                         \
1749 |       res, (long long)(newv), (long long)(oldv), (long long)(flags),           \
1750 |       (long long)(stackinfo_offset))
```
- **Line 1741 / 第 1741 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1742 / 第 1742 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1743 / 第 1743 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1744 / 第 1744 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1745 / 第 1745 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1746 / 第 1746 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1747 / 第 1747 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1748 / 第 1748 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1749 / 第 1749 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1750 / 第 1750 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1751-1760 / 第 1751-1760 行
```cpp
1751 | #define __sanitizer_syscall_pre_compat_60_sa_stacks(num, stacks)               \
1752 |   __sanitizer_syscall_pre_impl_compat_60_sa_stacks((long long)(num),           \
1753 |                                                    (long long)(stacks))
1754 | #define __sanitizer_syscall_post_compat_60_sa_stacks(res, num, stacks)         \
1755 |   __sanitizer_syscall_post_impl_compat_60_sa_stacks(res, (long long)(num),     \
1756 |                                                     (long long)(stacks))
1757 | #define __sanitizer_syscall_pre_compat_60_sa_enable()                          \
1758 |   __sanitizer_syscall_pre_impl_compat_60_sa_enable()
1759 | #define __sanitizer_syscall_post_compat_60_sa_enable(res)                      \
1760 |   __sanitizer_syscall_post_impl_compat_60_sa_enable(res)
```
- **Line 1751 / 第 1751 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1752 / 第 1752 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1753 / 第 1753 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1754 / 第 1754 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1755 / 第 1755 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1756 / 第 1756 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1757 / 第 1757 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1758 / 第 1758 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1759 / 第 1759 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1760 / 第 1760 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1761-1770 / 第 1761-1770 行
```cpp
1761 | #define __sanitizer_syscall_pre_compat_60_sa_setconcurrency(concurrency)       \
1762 |   __sanitizer_syscall_pre_impl_compat_60_sa_setconcurrency(                    \
1763 |       (long long)(concurrency))
1764 | #define __sanitizer_syscall_post_compat_60_sa_setconcurrency(res, concurrency) \
1765 |   __sanitizer_syscall_post_impl_compat_60_sa_setconcurrency(                   \
1766 |       res, (long long)(concurrency))
1767 | #define __sanitizer_syscall_pre_compat_60_sa_yield()                           \
1768 |   __sanitizer_syscall_pre_impl_compat_60_sa_yield()
1769 | #define __sanitizer_syscall_post_compat_60_sa_yield(res)                       \
1770 |   __sanitizer_syscall_post_impl_compat_60_sa_yield(res)
```
- **Line 1761 / 第 1761 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1762 / 第 1762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1763 / 第 1763 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1764 / 第 1764 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1765 / 第 1765 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1766 / 第 1766 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1767 / 第 1767 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1768 / 第 1768 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1769 / 第 1769 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1770 / 第 1770 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1771-1780 / 第 1771-1780 行
```cpp
1771 | #define __sanitizer_syscall_pre_compat_60_sa_preempt(sa_id)                    \
1772 |   __sanitizer_syscall_pre_impl_compat_60_sa_preempt((long long)(sa_id))
1773 | #define __sanitizer_syscall_post_compat_60_sa_preempt(res, sa_id)              \
1774 |   __sanitizer_syscall_post_impl_compat_60_sa_preempt(res, (long long)(sa_id))
1775 | /* syscall 336 has been skipped */
1776 | /* syscall 337 has been skipped */
1777 | /* syscall 338 has been skipped */
1778 | /* syscall 339 has been skipped */
1779 | #define __sanitizer_syscall_pre___sigaction_sigtramp(signum, nsa, osa, tramp,  \
1780 |                                                      vers)                     \
```
- **Line 1771 / 第 1771 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1772 / 第 1772 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1773 / 第 1773 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1774 / 第 1774 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1775 / 第 1775 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1776 / 第 1776 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1777 / 第 1777 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1778 / 第 1778 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1779 / 第 1779 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1780 / 第 1780 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1781-1790 / 第 1781-1790 行
```cpp
1781 |   __sanitizer_syscall_pre_impl___sigaction_sigtramp(                           \
1782 |       (long long)(signum), (long long)(nsa), (long long)(osa),                 \
1783 |       (long long)(tramp), (long long)(vers))
1784 | #define __sanitizer_syscall_post___sigaction_sigtramp(res, signum, nsa, osa,   \
1785 |                                                       tramp, vers)             \
1786 |   __sanitizer_syscall_post_impl___sigaction_sigtramp(                          \
1787 |       res, (long long)(signum), (long long)(nsa), (long long)(osa),            \
1788 |       (long long)(tramp), (long long)(vers))
1789 | /* syscall 341 has been skipped */
1790 | /* syscall 342 has been skipped */
```
- **Line 1781 / 第 1781 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1782 / 第 1782 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1783 / 第 1783 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1784 / 第 1784 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1785 / 第 1785 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1786 / 第 1786 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1787 / 第 1787 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1788 / 第 1788 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1789 / 第 1789 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1790 / 第 1790 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 1791-1800 / 第 1791-1800 行
```cpp
1791 | #define __sanitizer_syscall_pre_rasctl(addr, len, op)                          \
1792 |   __sanitizer_syscall_pre_impl_rasctl((long long)(addr), (long long)(len),     \
1793 |                                       (long long)(op))
1794 | #define __sanitizer_syscall_post_rasctl(res, addr, len, op)                    \
1795 |   __sanitizer_syscall_post_impl_rasctl(res, (long long)(addr),                 \
1796 |                                        (long long)(len), (long long)(op))
1797 | #define __sanitizer_syscall_pre_kqueue() __sanitizer_syscall_pre_impl_kqueue()
1798 | #define __sanitizer_syscall_post_kqueue(res)                                   \
1799 |   __sanitizer_syscall_post_impl_kqueue(res)
1800 | #define __sanitizer_syscall_pre_compat_50_kevent(fd, changelist, nchanges,     \
```
- **Line 1791 / 第 1791 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1792 / 第 1792 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1793 / 第 1793 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1794 / 第 1794 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1795 / 第 1795 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1796 / 第 1796 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1797 / 第 1797 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1798 / 第 1798 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1799 / 第 1799 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1800 / 第 1800 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1801-1810 / 第 1801-1810 行
```cpp
1801 |                                                  eventlist, nevents, timeout)  \
1802 |   __sanitizer_syscall_pre_impl_compat_50_kevent(                               \
1803 |       (long long)(fd), (long long)(changelist), (long long)(nchanges),         \
1804 |       (long long)(eventlist), (long long)(nevents), (long long)(timeout))
1805 | #define __sanitizer_syscall_post_compat_50_kevent(                             \
1806 |     res, fd, changelist, nchanges, eventlist, nevents, timeout)                \
1807 |   __sanitizer_syscall_post_impl_compat_50_kevent(                              \
1808 |       res, (long long)(fd), (long long)(changelist), (long long)(nchanges),    \
1809 |       (long long)(eventlist), (long long)(nevents), (long long)(timeout))
1810 | #define __sanitizer_syscall_pre__sched_setparam(pid, lid, policy, params)      \
```
- **Line 1801 / 第 1801 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1802 / 第 1802 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1803 / 第 1803 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1804 / 第 1804 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1805 / 第 1805 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1806 / 第 1806 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1807 / 第 1807 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1808 / 第 1808 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1809 / 第 1809 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1810 / 第 1810 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1811-1820 / 第 1811-1820 行
```cpp
1811 |   __sanitizer_syscall_pre_impl__sched_setparam(                                \
1812 |       (long long)(pid), (long long)(lid), (long long)(policy),                 \
1813 |       (long long)(params))
1814 | #define __sanitizer_syscall_post__sched_setparam(res, pid, lid, policy,        \
1815 |                                                  params)                       \
1816 |   __sanitizer_syscall_post_impl__sched_setparam(                               \
1817 |       res, (long long)(pid), (long long)(lid), (long long)(policy),            \
1818 |       (long long)(params))
1819 | #define __sanitizer_syscall_pre__sched_getparam(pid, lid, policy, params)      \
1820 |   __sanitizer_syscall_pre_impl__sched_getparam(                                \
```
- **Line 1811 / 第 1811 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1812 / 第 1812 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1813 / 第 1813 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1814 / 第 1814 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1815 / 第 1815 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1816 / 第 1816 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1817 / 第 1817 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1818 / 第 1818 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1819 / 第 1819 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1820 / 第 1820 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1821-1830 / 第 1821-1830 行
```cpp
1821 |       (long long)(pid), (long long)(lid), (long long)(policy),                 \
1822 |       (long long)(params))
1823 | #define __sanitizer_syscall_post__sched_getparam(res, pid, lid, policy,        \
1824 |                                                  params)                       \
1825 |   __sanitizer_syscall_post_impl__sched_getparam(                               \
1826 |       res, (long long)(pid), (long long)(lid), (long long)(policy),            \
1827 |       (long long)(params))
1828 | #define __sanitizer_syscall_pre__sched_setaffinity(pid, lid, size, cpuset)     \
1829 |   __sanitizer_syscall_pre_impl__sched_setaffinity(                             \
1830 |       (long long)(pid), (long long)(lid), (long long)(size),                   \
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
1831 |       (long long)(cpuset))
1832 | #define __sanitizer_syscall_post__sched_setaffinity(res, pid, lid, size,       \
1833 |                                                     cpuset)                    \
1834 |   __sanitizer_syscall_post_impl__sched_setaffinity(                            \
1835 |       res, (long long)(pid), (long long)(lid), (long long)(size),              \
1836 |       (long long)(cpuset))
1837 | #define __sanitizer_syscall_pre__sched_getaffinity(pid, lid, size, cpuset)     \
1838 |   __sanitizer_syscall_pre_impl__sched_getaffinity(                             \
1839 |       (long long)(pid), (long long)(lid), (long long)(size),                   \
1840 |       (long long)(cpuset))
```
- **Line 1831 / 第 1831 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1832 / 第 1832 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1833 / 第 1833 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1834 / 第 1834 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1835 / 第 1835 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1836 / 第 1836 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1837 / 第 1837 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1838 / 第 1838 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1839 / 第 1839 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1840 / 第 1840 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1841-1850 / 第 1841-1850 行
```cpp
1841 | #define __sanitizer_syscall_post__sched_getaffinity(res, pid, lid, size,       \
1842 |                                                     cpuset)                    \
1843 |   __sanitizer_syscall_post_impl__sched_getaffinity(                            \
1844 |       res, (long long)(pid), (long long)(lid), (long long)(size),              \
1845 |       (long long)(cpuset))
1846 | #define __sanitizer_syscall_pre_sched_yield()                                  \
1847 |   __sanitizer_syscall_pre_impl_sched_yield()
1848 | #define __sanitizer_syscall_post_sched_yield(res)                              \
1849 |   __sanitizer_syscall_post_impl_sched_yield(res)
1850 | #define __sanitizer_syscall_pre__sched_protect(priority)                       \
```
- **Line 1841 / 第 1841 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1842 / 第 1842 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1843 / 第 1843 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1844 / 第 1844 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1845 / 第 1845 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1846 / 第 1846 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1847 / 第 1847 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1848 / 第 1848 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1849 / 第 1849 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1850 / 第 1850 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1851-1860 / 第 1851-1860 行
```cpp
1851 |   __sanitizer_syscall_pre_impl__sched_protect((long long)(priority))
1852 | #define __sanitizer_syscall_post__sched_protect(res, priority)                 \
1853 |   __sanitizer_syscall_post_impl__sched_protect(res, (long long)(priority))
1854 | /* syscall 352 has been skipped */
1855 | /* syscall 353 has been skipped */
1856 | #define __sanitizer_syscall_pre_fsync_range(fd, flags, start, length)          \
1857 |   __sanitizer_syscall_pre_impl_fsync_range(                                    \
1858 |       (long long)(fd), (long long)(flags), (long long)(start),                 \
1859 |       (long long)(length))
1860 | #define __sanitizer_syscall_post_fsync_range(res, fd, flags, start, length)    \
```
- **Line 1851 / 第 1851 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1852 / 第 1852 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1853 / 第 1853 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1854 / 第 1854 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1855 / 第 1855 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 1856 / 第 1856 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1857 / 第 1857 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1858 / 第 1858 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1859 / 第 1859 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1860 / 第 1860 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1861-1870 / 第 1861-1870 行
```cpp
1861 |   __sanitizer_syscall_post_impl_fsync_range(                                   \
1862 |       res, (long long)(fd), (long long)(flags), (long long)(start),            \
1863 |       (long long)(length))
1864 | #define __sanitizer_syscall_pre_uuidgen(store, count)                          \
1865 |   __sanitizer_syscall_pre_impl_uuidgen((long long)(store), (long long)(count))
1866 | #define __sanitizer_syscall_post_uuidgen(res, store, count)                    \
1867 |   __sanitizer_syscall_post_impl_uuidgen(res, (long long)(store),               \
1868 |                                         (long long)(count))
1869 | #define __sanitizer_syscall_pre_compat_90_getvfsstat(buf, bufsize, flags)      \
1870 |   __sanitizer_syscall_pre_impl_compat_90_getvfsstat(                           \
```
- **Line 1861 / 第 1861 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1862 / 第 1862 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1863 / 第 1863 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1864 / 第 1864 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1865 / 第 1865 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1866 / 第 1866 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1867 / 第 1867 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1868 / 第 1868 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1869 / 第 1869 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1870 / 第 1870 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1871-1880 / 第 1871-1880 行
```cpp
1871 |       (long long)(buf), (long long)(bufsize), (long long)(flags))
1872 | #define __sanitizer_syscall_post_compat_90_getvfsstat(res, buf, bufsize,       \
1873 |                                                       flags)                   \
1874 |   __sanitizer_syscall_post_impl_compat_90_getvfsstat(                          \
1875 |       res, (long long)(buf), (long long)(bufsize), (long long)(flags))
1876 | #define __sanitizer_syscall_pre_compat_90_statvfs1(path, buf, flags)           \
1877 |   __sanitizer_syscall_pre_impl_compat_90_statvfs1(                             \
1878 |       (long long)(path), (long long)(buf), (long long)(flags))
1879 | #define __sanitizer_syscall_post_compat_90_statvfs1(res, path, buf, flags)     \
1880 |   __sanitizer_syscall_post_impl_compat_90_statvfs1(                            \
```
- **Line 1871 / 第 1871 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1872 / 第 1872 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1873 / 第 1873 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1874 / 第 1874 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1875 / 第 1875 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1876 / 第 1876 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1877 / 第 1877 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1878 / 第 1878 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1879 / 第 1879 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1880 / 第 1880 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1881-1890 / 第 1881-1890 行
```cpp
1881 |       res, (long long)(path), (long long)(buf), (long long)(flags))
1882 | #define __sanitizer_syscall_pre_compat_90_fstatvfs1(fd, buf, flags)            \
1883 |   __sanitizer_syscall_pre_impl_compat_90_fstatvfs1(                            \
1884 |       (long long)(fd), (long long)(buf), (long long)(flags))
1885 | #define __sanitizer_syscall_post_compat_90_fstatvfs1(res, fd, buf, flags)      \
1886 |   __sanitizer_syscall_post_impl_compat_90_fstatvfs1(                           \
1887 |       res, (long long)(fd), (long long)(buf), (long long)(flags))
1888 | #define __sanitizer_syscall_pre_compat_30_fhstatvfs1(fhp, buf, flags)          \
1889 |   __sanitizer_syscall_pre_impl_compat_30_fhstatvfs1(                           \
1890 |       (long long)(fhp), (long long)(buf), (long long)(flags))
```
- **Line 1881 / 第 1881 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1882 / 第 1882 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1883 / 第 1883 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1884 / 第 1884 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1885 / 第 1885 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1886 / 第 1886 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1887 / 第 1887 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1888 / 第 1888 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1889 / 第 1889 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1890 / 第 1890 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1891-1900 / 第 1891-1900 行
```cpp
1891 | #define __sanitizer_syscall_post_compat_30_fhstatvfs1(res, fhp, buf, flags)    \
1892 |   __sanitizer_syscall_post_impl_compat_30_fhstatvfs1(                          \
1893 |       res, (long long)(fhp), (long long)(buf), (long long)(flags))
1894 | #define __sanitizer_syscall_pre_extattrctl(path, cmd, filename, attrnamespace, \
1895 |                                            attrname)                           \
1896 |   __sanitizer_syscall_pre_impl_extattrctl(                                     \
1897 |       (long long)(path), (long long)(cmd), (long long)(filename),              \
1898 |       (long long)(attrnamespace), (long long)(attrname))
1899 | #define __sanitizer_syscall_post_extattrctl(res, path, cmd, filename,          \
1900 |                                             attrnamespace, attrname)           \
```
- **Line 1891 / 第 1891 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1892 / 第 1892 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1893 / 第 1893 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1894 / 第 1894 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1895 / 第 1895 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1896 / 第 1896 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1897 / 第 1897 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1898 / 第 1898 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1899 / 第 1899 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1900 / 第 1900 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1901-1910 / 第 1901-1910 行
```cpp
1901 |   __sanitizer_syscall_post_impl_extattrctl(                                    \
1902 |       res, (long long)(path), (long long)(cmd), (long long)(filename),         \
1903 |       (long long)(attrnamespace), (long long)(attrname))
1904 | #define __sanitizer_syscall_pre_extattr_set_file(path, attrnamespace,          \
1905 |                                                  attrname, data, nbytes)       \
1906 |   __sanitizer_syscall_pre_impl_extattr_set_file(                               \
1907 |       (long long)(path), (long long)(attrnamespace), (long long)(attrname),    \
1908 |       (long long)(data), (long long)(nbytes))
1909 | #define __sanitizer_syscall_post_extattr_set_file(res, path, attrnamespace,    \
1910 |                                                   attrname, data, nbytes)      \
```
- **Line 1901 / 第 1901 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1902 / 第 1902 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1903 / 第 1903 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1904 / 第 1904 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1905 / 第 1905 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1906 / 第 1906 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1907 / 第 1907 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1908 / 第 1908 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1909 / 第 1909 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1910 / 第 1910 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1911-1920 / 第 1911-1920 行
```cpp
1911 |   __sanitizer_syscall_post_impl_extattr_set_file(                              \
1912 |       res, (long long)(path), (long long)(attrnamespace),                      \
1913 |       (long long)(attrname), (long long)(data), (long long)(nbytes))
1914 | #define __sanitizer_syscall_pre_extattr_get_file(path, attrnamespace,          \
1915 |                                                  attrname, data, nbytes)       \
1916 |   __sanitizer_syscall_pre_impl_extattr_get_file(                               \
1917 |       (long long)(path), (long long)(attrnamespace), (long long)(attrname),    \
1918 |       (long long)(data), (long long)(nbytes))
1919 | #define __sanitizer_syscall_post_extattr_get_file(res, path, attrnamespace,    \
1920 |                                                   attrname, data, nbytes)      \
```
- **Line 1911 / 第 1911 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1912 / 第 1912 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1913 / 第 1913 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1914 / 第 1914 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1915 / 第 1915 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1916 / 第 1916 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1917 / 第 1917 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1918 / 第 1918 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1919 / 第 1919 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1920 / 第 1920 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1921-1930 / 第 1921-1930 行
```cpp
1921 |   __sanitizer_syscall_post_impl_extattr_get_file(                              \
1922 |       res, (long long)(path), (long long)(attrnamespace),                      \
1923 |       (long long)(attrname), (long long)(data), (long long)(nbytes))
1924 | #define __sanitizer_syscall_pre_extattr_delete_file(path, attrnamespace,       \
1925 |                                                     attrname)                  \
1926 |   __sanitizer_syscall_pre_impl_extattr_delete_file(                            \
1927 |       (long long)(path), (long long)(attrnamespace), (long long)(attrname))
1928 | #define __sanitizer_syscall_post_extattr_delete_file(res, path, attrnamespace, \
1929 |                                                      attrname)                 \
1930 |   __sanitizer_syscall_post_impl_extattr_delete_file(                           \
```
- **Line 1921 / 第 1921 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1922 / 第 1922 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1923 / 第 1923 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1924 / 第 1924 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1925 / 第 1925 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1926 / 第 1926 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1927 / 第 1927 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1928 / 第 1928 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1929 / 第 1929 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1930 / 第 1930 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1931-1940 / 第 1931-1940 行
```cpp
1931 |       res, (long long)(path), (long long)(attrnamespace),                      \
1932 |       (long long)(attrname))
1933 | #define __sanitizer_syscall_pre_extattr_set_fd(fd, attrnamespace, attrname,    \
1934 |                                                data, nbytes)                   \
1935 |   __sanitizer_syscall_pre_impl_extattr_set_fd(                                 \
1936 |       (long long)(fd), (long long)(attrnamespace), (long long)(attrname),      \
1937 |       (long long)(data), (long long)(nbytes))
1938 | #define __sanitizer_syscall_post_extattr_set_fd(res, fd, attrnamespace,        \
1939 |                                                 attrname, data, nbytes)        \
1940 |   __sanitizer_syscall_post_impl_extattr_set_fd(                                \
```
- **Line 1931 / 第 1931 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1932 / 第 1932 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1933 / 第 1933 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1934 / 第 1934 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1935 / 第 1935 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1936 / 第 1936 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1937 / 第 1937 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1938 / 第 1938 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1939 / 第 1939 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1940 / 第 1940 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1941-1950 / 第 1941-1950 行
```cpp
1941 |       res, (long long)(fd), (long long)(attrnamespace), (long long)(attrname), \
1942 |       (long long)(data), (long long)(nbytes))
1943 | #define __sanitizer_syscall_pre_extattr_get_fd(fd, attrnamespace, attrname,    \
1944 |                                                data, nbytes)                   \
1945 |   __sanitizer_syscall_pre_impl_extattr_get_fd(                                 \
1946 |       (long long)(fd), (long long)(attrnamespace), (long long)(attrname),      \
1947 |       (long long)(data), (long long)(nbytes))
1948 | #define __sanitizer_syscall_post_extattr_get_fd(res, fd, attrnamespace,        \
1949 |                                                 attrname, data, nbytes)        \
1950 |   __sanitizer_syscall_post_impl_extattr_get_fd(                                \
```
- **Line 1941 / 第 1941 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1942 / 第 1942 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1943 / 第 1943 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1944 / 第 1944 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1945 / 第 1945 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1946 / 第 1946 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1947 / 第 1947 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1948 / 第 1948 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1949 / 第 1949 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1950 / 第 1950 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1951-1960 / 第 1951-1960 行
```cpp
1951 |       res, (long long)(fd), (long long)(attrnamespace), (long long)(attrname), \
1952 |       (long long)(data), (long long)(nbytes))
1953 | #define __sanitizer_syscall_pre_extattr_delete_fd(fd, attrnamespace, attrname) \
1954 |   __sanitizer_syscall_pre_impl_extattr_delete_fd(                              \
1955 |       (long long)(fd), (long long)(attrnamespace), (long long)(attrname))
1956 | #define __sanitizer_syscall_post_extattr_delete_fd(res, fd, attrnamespace,     \
1957 |                                                    attrname)                   \
1958 |   __sanitizer_syscall_post_impl_extattr_delete_fd(                             \
1959 |       res, (long long)(fd), (long long)(attrnamespace), (long long)(attrname))
1960 | #define __sanitizer_syscall_pre_extattr_set_link(path, attrnamespace,          \
```
- **Line 1951 / 第 1951 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1952 / 第 1952 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1953 / 第 1953 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1954 / 第 1954 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1955 / 第 1955 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1956 / 第 1956 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1957 / 第 1957 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1958 / 第 1958 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1959 / 第 1959 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1960 / 第 1960 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1961-1970 / 第 1961-1970 行
```cpp
1961 |                                                  attrname, data, nbytes)       \
1962 |   __sanitizer_syscall_pre_impl_extattr_set_link(                               \
1963 |       (long long)(path), (long long)(attrnamespace), (long long)(attrname),    \
1964 |       (long long)(data), (long long)(nbytes))
1965 | #define __sanitizer_syscall_post_extattr_set_link(res, path, attrnamespace,    \
1966 |                                                   attrname, data, nbytes)      \
1967 |   __sanitizer_syscall_post_impl_extattr_set_link(                              \
1968 |       res, (long long)(path), (long long)(attrnamespace),                      \
1969 |       (long long)(attrname), (long long)(data), (long long)(nbytes))
1970 | #define __sanitizer_syscall_pre_extattr_get_link(path, attrnamespace,          \
```
- **Line 1961 / 第 1961 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1962 / 第 1962 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1963 / 第 1963 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1964 / 第 1964 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1965 / 第 1965 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1966 / 第 1966 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1967 / 第 1967 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1968 / 第 1968 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1969 / 第 1969 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1970 / 第 1970 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1971-1980 / 第 1971-1980 行
```cpp
1971 |                                                  attrname, data, nbytes)       \
1972 |   __sanitizer_syscall_pre_impl_extattr_get_link(                               \
1973 |       (long long)(path), (long long)(attrnamespace), (long long)(attrname),    \
1974 |       (long long)(data), (long long)(nbytes))
1975 | #define __sanitizer_syscall_post_extattr_get_link(res, path, attrnamespace,    \
1976 |                                                   attrname, data, nbytes)      \
1977 |   __sanitizer_syscall_post_impl_extattr_get_link(                              \
1978 |       res, (long long)(path), (long long)(attrnamespace),                      \
1979 |       (long long)(attrname), (long long)(data), (long long)(nbytes))
1980 | #define __sanitizer_syscall_pre_extattr_delete_link(path, attrnamespace,       \
```
- **Line 1971 / 第 1971 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1972 / 第 1972 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1973 / 第 1973 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1974 / 第 1974 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1975 / 第 1975 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1976 / 第 1976 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1977 / 第 1977 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1978 / 第 1978 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1979 / 第 1979 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1980 / 第 1980 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 1981-1990 / 第 1981-1990 行
```cpp
1981 |                                                     attrname)                  \
1982 |   __sanitizer_syscall_pre_impl_extattr_delete_link(                            \
1983 |       (long long)(path), (long long)(attrnamespace), (long long)(attrname))
1984 | #define __sanitizer_syscall_post_extattr_delete_link(res, path, attrnamespace, \
1985 |                                                      attrname)                 \
1986 |   __sanitizer_syscall_post_impl_extattr_delete_link(                           \
1987 |       res, (long long)(path), (long long)(attrnamespace),                      \
1988 |       (long long)(attrname))
1989 | #define __sanitizer_syscall_pre_extattr_list_fd(fd, attrnamespace, data,       \
1990 |                                                 nbytes)                        \
```
- **Line 1981 / 第 1981 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1982 / 第 1982 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1983 / 第 1983 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1984 / 第 1984 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1985 / 第 1985 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1986 / 第 1986 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1987 / 第 1987 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1988 / 第 1988 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1989 / 第 1989 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1990 / 第 1990 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 1991-2000 / 第 1991-2000 行
```cpp
1991 |   __sanitizer_syscall_pre_impl_extattr_list_fd(                                \
1992 |       (long long)(fd), (long long)(attrnamespace), (long long)(data),          \
1993 |       (long long)(nbytes))
1994 | #define __sanitizer_syscall_post_extattr_list_fd(res, fd, attrnamespace, data, \
1995 |                                                  nbytes)                       \
1996 |   __sanitizer_syscall_post_impl_extattr_list_fd(                               \
1997 |       res, (long long)(fd), (long long)(attrnamespace), (long long)(data),     \
1998 |       (long long)(nbytes))
1999 | #define __sanitizer_syscall_pre_extattr_list_file(path, attrnamespace, data,   \
2000 |                                                   nbytes)                      \
```
- **Line 1991 / 第 1991 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1992 / 第 1992 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1993 / 第 1993 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1994 / 第 1994 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 1995 / 第 1995 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1996 / 第 1996 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1997 / 第 1997 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1998 / 第 1998 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 1999 / 第 1999 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2000 / 第 2000 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2001-2010 / 第 2001-2010 行
```cpp
2001 |   __sanitizer_syscall_pre_impl_extattr_list_file(                              \
2002 |       (long long)(path), (long long)(attrnamespace), (long long)(data),        \
2003 |       (long long)(nbytes))
2004 | #define __sanitizer_syscall_post_extattr_list_file(res, path, attrnamespace,   \
2005 |                                                    data, nbytes)               \
2006 |   __sanitizer_syscall_post_impl_extattr_list_file(                             \
2007 |       res, (long long)(path), (long long)(attrnamespace), (long long)(data),   \
2008 |       (long long)(nbytes))
2009 | #define __sanitizer_syscall_pre_extattr_list_link(path, attrnamespace, data,   \
2010 |                                                   nbytes)                      \
```
- **Line 2001 / 第 2001 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2002 / 第 2002 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2003 / 第 2003 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2004 / 第 2004 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2005 / 第 2005 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2006 / 第 2006 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2007 / 第 2007 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2008 / 第 2008 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2009 / 第 2009 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2010 / 第 2010 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2011-2020 / 第 2011-2020 行
```cpp
2011 |   __sanitizer_syscall_pre_impl_extattr_list_link(                              \
2012 |       (long long)(path), (long long)(attrnamespace), (long long)(data),        \
2013 |       (long long)(nbytes))
2014 | #define __sanitizer_syscall_post_extattr_list_link(res, path, attrnamespace,   \
2015 |                                                    data, nbytes)               \
2016 |   __sanitizer_syscall_post_impl_extattr_list_link(                             \
2017 |       res, (long long)(path), (long long)(attrnamespace), (long long)(data),   \
2018 |       (long long)(nbytes))
2019 | #define __sanitizer_syscall_pre_compat_50_pselect(nd, in, ou, ex, ts, mask)    \
2020 |   __sanitizer_syscall_pre_impl_compat_50_pselect(                              \
```
- **Line 2011 / 第 2011 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2012 / 第 2012 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2013 / 第 2013 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2014 / 第 2014 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2015 / 第 2015 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2016 / 第 2016 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2017 / 第 2017 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2018 / 第 2018 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2019 / 第 2019 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2020 / 第 2020 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2021-2030 / 第 2021-2030 行
```cpp
2021 |       (long long)(nd), (long long)(in), (long long)(ou), (long long)(ex),      \
2022 |       (long long)(ts), (long long)(mask))
2023 | #define __sanitizer_syscall_post_compat_50_pselect(res, nd, in, ou, ex, ts,    \
2024 |                                                    mask)                       \
2025 |   __sanitizer_syscall_post_impl_compat_50_pselect(                             \
2026 |       res, (long long)(nd), (long long)(in), (long long)(ou), (long long)(ex), \
2027 |       (long long)(ts), (long long)(mask))
2028 | #define __sanitizer_syscall_pre_compat_50_pollts(fds, nfds, ts, mask)          \
2029 |   __sanitizer_syscall_pre_impl_compat_50_pollts(                               \
2030 |       (long long)(fds), (long long)(nfds), (long long)(ts), (long long)(mask))
```
- **Line 2021 / 第 2021 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2022 / 第 2022 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2023 / 第 2023 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2024 / 第 2024 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2025 / 第 2025 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2026 / 第 2026 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2027 / 第 2027 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2028 / 第 2028 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2029 / 第 2029 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2030 / 第 2030 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2031-2040 / 第 2031-2040 行
```cpp
2031 | #define __sanitizer_syscall_post_compat_50_pollts(res, fds, nfds, ts, mask)    \
2032 |   __sanitizer_syscall_post_impl_compat_50_pollts(                              \
2033 |       res, (long long)(fds), (long long)(nfds), (long long)(ts),               \
2034 |       (long long)(mask))
2035 | #define __sanitizer_syscall_pre_setxattr(path, name, value, size, flags)       \
2036 |   __sanitizer_syscall_pre_impl_setxattr((long long)(path), (long long)(name),  \
2037 |                                         (long long)(value), (long long)(size), \
2038 |                                         (long long)(flags))
2039 | #define __sanitizer_syscall_post_setxattr(res, path, name, value, size, flags) \
2040 |   __sanitizer_syscall_post_impl_setxattr(                                      \
```
- **Line 2031 / 第 2031 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2032 / 第 2032 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2033 / 第 2033 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2034 / 第 2034 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2035 / 第 2035 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2036 / 第 2036 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2037 / 第 2037 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2038 / 第 2038 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2039 / 第 2039 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2040 / 第 2040 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2041-2050 / 第 2041-2050 行
```cpp
2041 |       res, (long long)(path), (long long)(name), (long long)(value),           \
2042 |       (long long)(size), (long long)(flags))
2043 | #define __sanitizer_syscall_pre_lsetxattr(path, name, value, size, flags)      \
2044 |   __sanitizer_syscall_pre_impl_lsetxattr(                                      \
2045 |       (long long)(path), (long long)(name), (long long)(value),                \
2046 |       (long long)(size), (long long)(flags))
2047 | #define __sanitizer_syscall_post_lsetxattr(res, path, name, value, size,       \
2048 |                                            flags)                              \
2049 |   __sanitizer_syscall_post_impl_lsetxattr(                                     \
2050 |       res, (long long)(path), (long long)(name), (long long)(value),           \
```
- **Line 2041 / 第 2041 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2042 / 第 2042 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2043 / 第 2043 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2044 / 第 2044 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2045 / 第 2045 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2046 / 第 2046 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2047 / 第 2047 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2048 / 第 2048 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2049 / 第 2049 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2050 / 第 2050 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2051-2060 / 第 2051-2060 行
```cpp
2051 |       (long long)(size), (long long)(flags))
2052 | #define __sanitizer_syscall_pre_fsetxattr(fd, name, value, size, flags)        \
2053 |   __sanitizer_syscall_pre_impl_fsetxattr(                                      \
2054 |       (long long)(fd), (long long)(name), (long long)(value),                  \
2055 |       (long long)(size), (long long)(flags))
2056 | #define __sanitizer_syscall_post_fsetxattr(res, fd, name, value, size, flags)  \
2057 |   __sanitizer_syscall_post_impl_fsetxattr(                                     \
2058 |       res, (long long)(fd), (long long)(name), (long long)(value),             \
2059 |       (long long)(size), (long long)(flags))
2060 | #define __sanitizer_syscall_pre_getxattr(path, name, value, size)              \
```
- **Line 2051 / 第 2051 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2052 / 第 2052 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2053 / 第 2053 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2054 / 第 2054 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2055 / 第 2055 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2056 / 第 2056 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2057 / 第 2057 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2058 / 第 2058 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2059 / 第 2059 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2060 / 第 2060 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2061-2070 / 第 2061-2070 行
```cpp
2061 |   __sanitizer_syscall_pre_impl_getxattr((long long)(path), (long long)(name),  \
2062 |                                         (long long)(value), (long long)(size))
2063 | #define __sanitizer_syscall_post_getxattr(res, path, name, value, size)        \
2064 |   __sanitizer_syscall_post_impl_getxattr(                                      \
2065 |       res, (long long)(path), (long long)(name), (long long)(value),           \
2066 |       (long long)(size))
2067 | #define __sanitizer_syscall_pre_lgetxattr(path, name, value, size)             \
2068 |   __sanitizer_syscall_pre_impl_lgetxattr((long long)(path), (long long)(name), \
2069 |                                          (long long)(value),                   \
2070 |                                          (long long)(size))
```
- **Line 2061 / 第 2061 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2062 / 第 2062 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2063 / 第 2063 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2064 / 第 2064 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2065 / 第 2065 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2066 / 第 2066 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2067 / 第 2067 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2068 / 第 2068 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2069 / 第 2069 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2070 / 第 2070 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2071-2080 / 第 2071-2080 行
```cpp
2071 | #define __sanitizer_syscall_post_lgetxattr(res, path, name, value, size)       \
2072 |   __sanitizer_syscall_post_impl_lgetxattr(                                     \
2073 |       res, (long long)(path), (long long)(name), (long long)(value),           \
2074 |       (long long)(size))
2075 | #define __sanitizer_syscall_pre_fgetxattr(fd, name, value, size)               \
2076 |   __sanitizer_syscall_pre_impl_fgetxattr((long long)(fd), (long long)(name),   \
2077 |                                          (long long)(value),                   \
2078 |                                          (long long)(size))
2079 | #define __sanitizer_syscall_post_fgetxattr(res, fd, name, value, size)         \
2080 |   __sanitizer_syscall_post_impl_fgetxattr(                                     \
```
- **Line 2071 / 第 2071 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2072 / 第 2072 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2073 / 第 2073 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2074 / 第 2074 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2075 / 第 2075 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2076 / 第 2076 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2077 / 第 2077 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2078 / 第 2078 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2079 / 第 2079 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2080 / 第 2080 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2081-2090 / 第 2081-2090 行
```cpp
2081 |       res, (long long)(fd), (long long)(name), (long long)(value),             \
2082 |       (long long)(size))
2083 | #define __sanitizer_syscall_pre_listxattr(path, list, size)                    \
2084 |   __sanitizer_syscall_pre_impl_listxattr((long long)(path), (long long)(list), \
2085 |                                          (long long)(size))
2086 | #define __sanitizer_syscall_post_listxattr(res, path, list, size)              \
2087 |   __sanitizer_syscall_post_impl_listxattr(                                     \
2088 |       res, (long long)(path), (long long)(list), (long long)(size))
2089 | #define __sanitizer_syscall_pre_llistxattr(path, list, size)                   \
2090 |   __sanitizer_syscall_pre_impl_llistxattr(                                     \
```
- **Line 2081 / 第 2081 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2082 / 第 2082 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2083 / 第 2083 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2084 / 第 2084 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2085 / 第 2085 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2086 / 第 2086 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2087 / 第 2087 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2088 / 第 2088 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2089 / 第 2089 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2090 / 第 2090 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2091-2100 / 第 2091-2100 行
```cpp
2091 |       (long long)(path), (long long)(list), (long long)(size))
2092 | #define __sanitizer_syscall_post_llistxattr(res, path, list, size)             \
2093 |   __sanitizer_syscall_post_impl_llistxattr(                                    \
2094 |       res, (long long)(path), (long long)(list), (long long)(size))
2095 | #define __sanitizer_syscall_pre_flistxattr(fd, list, size)                     \
2096 |   __sanitizer_syscall_pre_impl_flistxattr((long long)(fd), (long long)(list),  \
2097 |                                           (long long)(size))
2098 | #define __sanitizer_syscall_post_flistxattr(res, fd, list, size)               \
2099 |   __sanitizer_syscall_post_impl_flistxattr(                                    \
2100 |       res, (long long)(fd), (long long)(list), (long long)(size))
```
- **Line 2091 / 第 2091 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2092 / 第 2092 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2093 / 第 2093 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2094 / 第 2094 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2095 / 第 2095 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2096 / 第 2096 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2097 / 第 2097 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2098 / 第 2098 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2099 / 第 2099 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2100 / 第 2100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2101-2110 / 第 2101-2110 行
```cpp
2101 | #define __sanitizer_syscall_pre_removexattr(path, name)                        \
2102 |   __sanitizer_syscall_pre_impl_removexattr((long long)(path), (long long)(name))
2103 | #define __sanitizer_syscall_post_removexattr(res, path, name)                  \
2104 |   __sanitizer_syscall_post_impl_removexattr(res, (long long)(path),            \
2105 |                                             (long long)(name))
2106 | #define __sanitizer_syscall_pre_lremovexattr(path, name)                       \
2107 |   __sanitizer_syscall_pre_impl_lremovexattr((long long)(path),                 \
2108 |                                             (long long)(name))
2109 | #define __sanitizer_syscall_post_lremovexattr(res, path, name)                 \
2110 |   __sanitizer_syscall_post_impl_lremovexattr(res, (long long)(path),           \
```
- **Line 2101 / 第 2101 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2102 / 第 2102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2103 / 第 2103 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2104 / 第 2104 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2105 / 第 2105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2106 / 第 2106 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2107 / 第 2107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2108 / 第 2108 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2109 / 第 2109 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2110 / 第 2110 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2111-2120 / 第 2111-2120 行
```cpp
2111 |                                              (long long)(name))
2112 | #define __sanitizer_syscall_pre_fremovexattr(fd, name)                         \
2113 |   __sanitizer_syscall_pre_impl_fremovexattr((long long)(fd), (long long)(name))
2114 | #define __sanitizer_syscall_post_fremovexattr(res, fd, name)                   \
2115 |   __sanitizer_syscall_post_impl_fremovexattr(res, (long long)(fd),             \
2116 |                                              (long long)(name))
2117 | #define __sanitizer_syscall_pre_compat_50___stat30(path, ub)                   \
2118 |   __sanitizer_syscall_pre_impl_compat_50___stat30((long long)(path),           \
2119 |                                                   (long long)(ub))
2120 | #define __sanitizer_syscall_post_compat_50___stat30(res, path, ub)             \
```
- **Line 2111 / 第 2111 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2112 / 第 2112 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2113 / 第 2113 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2114 / 第 2114 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2115 / 第 2115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2116 / 第 2116 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2117 / 第 2117 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2118 / 第 2118 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2119 / 第 2119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2120 / 第 2120 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2121-2130 / 第 2121-2130 行
```cpp
2121 |   __sanitizer_syscall_post_impl_compat_50___stat30(res, (long long)(path),     \
2122 |                                                    (long long)(ub))
2123 | #define __sanitizer_syscall_pre_compat_50___fstat30(fd, sb)                    \
2124 |   __sanitizer_syscall_pre_impl_compat_50___fstat30((long long)(fd),            \
2125 |                                                    (long long)(sb))
2126 | #define __sanitizer_syscall_post_compat_50___fstat30(res, fd, sb)              \
2127 |   __sanitizer_syscall_post_impl_compat_50___fstat30(res, (long long)(fd),      \
2128 |                                                     (long long)(sb))
2129 | #define __sanitizer_syscall_pre_compat_50___lstat30(path, ub)                  \
2130 |   __sanitizer_syscall_pre_impl_compat_50___lstat30((long long)(path),          \
```
- **Line 2121 / 第 2121 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2122 / 第 2122 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2123 / 第 2123 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2124 / 第 2124 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2125 / 第 2125 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2126 / 第 2126 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2127 / 第 2127 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2128 / 第 2128 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2129 / 第 2129 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2130 / 第 2130 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2131-2140 / 第 2131-2140 行
```cpp
2131 |                                                    (long long)(ub))
2132 | #define __sanitizer_syscall_post_compat_50___lstat30(res, path, ub)            \
2133 |   __sanitizer_syscall_post_impl_compat_50___lstat30(res, (long long)(path),    \
2134 |                                                     (long long)(ub))
2135 | #define __sanitizer_syscall_pre___getdents30(fd, buf, count)                   \
2136 |   __sanitizer_syscall_pre_impl___getdents30((long long)(fd), (long long)(buf), \
2137 |                                             (long long)(count))
2138 | #define __sanitizer_syscall_post___getdents30(res, fd, buf, count)             \
2139 |   __sanitizer_syscall_post_impl___getdents30(                                  \
2140 |       res, (long long)(fd), (long long)(buf), (long long)(count))
```
- **Line 2131 / 第 2131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2132 / 第 2132 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2133 / 第 2133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2134 / 第 2134 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2135 / 第 2135 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2136 / 第 2136 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2137 / 第 2137 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2138 / 第 2138 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2139 / 第 2139 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2140 / 第 2140 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2141-2150 / 第 2141-2150 行
```cpp
2141 | #define __sanitizer_syscall_pre_posix_fadvise()                                \
2142 |   __sanitizer_syscall_pre_impl_posix_fadvise((long long)())
2143 | #define __sanitizer_syscall_post_posix_fadvise(res)                            \
2144 |   __sanitizer_syscall_post_impl_posix_fadvise(res, (long long)())
2145 | #define __sanitizer_syscall_pre_compat_30___fhstat30(fhp, sb)                  \
2146 |   __sanitizer_syscall_pre_impl_compat_30___fhstat30((long long)(fhp),          \
2147 |                                                     (long long)(sb))
2148 | #define __sanitizer_syscall_post_compat_30___fhstat30(res, fhp, sb)            \
2149 |   __sanitizer_syscall_post_impl_compat_30___fhstat30(res, (long long)(fhp),    \
2150 |                                                      (long long)(sb))
```
- **Line 2141 / 第 2141 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2142 / 第 2142 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2143 / 第 2143 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2144 / 第 2144 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2145 / 第 2145 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2146 / 第 2146 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2147 / 第 2147 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2148 / 第 2148 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2149 / 第 2149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2150 / 第 2150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2151-2160 / 第 2151-2160 行
```cpp
2151 | #define __sanitizer_syscall_pre_compat_50___ntp_gettime30(ntvp)                \
2152 |   __sanitizer_syscall_pre_impl_compat_50___ntp_gettime30((long long)(ntvp))
2153 | #define __sanitizer_syscall_post_compat_50___ntp_gettime30(res, ntvp)          \
2154 |   __sanitizer_syscall_post_impl_compat_50___ntp_gettime30(res,                 \
2155 |                                                           (long long)(ntvp))
2156 | #define __sanitizer_syscall_pre___socket30(domain, type, protocol)             \
2157 |   __sanitizer_syscall_pre_impl___socket30(                                     \
2158 |       (long long)(domain), (long long)(type), (long long)(protocol))
2159 | #define __sanitizer_syscall_post___socket30(res, domain, type, protocol)       \
2160 |   __sanitizer_syscall_post_impl___socket30(                                    \
```
- **Line 2151 / 第 2151 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2152 / 第 2152 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2153 / 第 2153 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2154 / 第 2154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2155 / 第 2155 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2156 / 第 2156 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2157 / 第 2157 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2158 / 第 2158 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2159 / 第 2159 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2160 / 第 2160 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2161-2170 / 第 2161-2170 行
```cpp
2161 |       res, (long long)(domain), (long long)(type), (long long)(protocol))
2162 | #define __sanitizer_syscall_pre___getfh30(fname, fhp, fh_size)                 \
2163 |   __sanitizer_syscall_pre_impl___getfh30((long long)(fname), (long long)(fhp), \
2164 |                                          (long long)(fh_size))
2165 | #define __sanitizer_syscall_post___getfh30(res, fname, fhp, fh_size)           \
2166 |   __sanitizer_syscall_post_impl___getfh30(                                     \
2167 |       res, (long long)(fname), (long long)(fhp), (long long)(fh_size))
2168 | #define __sanitizer_syscall_pre___fhopen40(fhp, fh_size, flags)                \
2169 |   __sanitizer_syscall_pre_impl___fhopen40(                                     \
2170 |       (long long)(fhp), (long long)(fh_size), (long long)(flags))
```
- **Line 2161 / 第 2161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2162 / 第 2162 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2163 / 第 2163 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2164 / 第 2164 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2165 / 第 2165 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2166 / 第 2166 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2167 / 第 2167 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2168 / 第 2168 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2169 / 第 2169 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2170 / 第 2170 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2171-2180 / 第 2171-2180 行
```cpp
2171 | #define __sanitizer_syscall_post___fhopen40(res, fhp, fh_size, flags)          \
2172 |   __sanitizer_syscall_post_impl___fhopen40(                                    \
2173 |       res, (long long)(fhp), (long long)(fh_size), (long long)(flags))
2174 | #define __sanitizer_syscall_pre_compat_90_fhstatvfs1(fhp, fh_size, buf, flags) \
2175 |   __sanitizer_syscall_pre_impl_compat_90_fhstatvfs1(                           \
2176 |       (long long)(fhp), (long long)(fh_size), (long long)(buf),                \
2177 |       (long long)(flags))
2178 | #define __sanitizer_syscall_post_compat_90_fhstatvfs1(res, fhp, fh_size, buf,  \
2179 |                                                       flags)                   \
2180 |   __sanitizer_syscall_post_impl_compat_90_fhstatvfs1(                          \
```
- **Line 2171 / 第 2171 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2172 / 第 2172 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2173 / 第 2173 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2174 / 第 2174 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2175 / 第 2175 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2176 / 第 2176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2177 / 第 2177 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2178 / 第 2178 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2179 / 第 2179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2180 / 第 2180 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2181-2190 / 第 2181-2190 行
```cpp
2181 |       res, (long long)(fhp), (long long)(fh_size), (long long)(buf),           \
2182 |       (long long)(flags))
2183 | #define __sanitizer_syscall_pre_compat_50___fhstat40(fhp, fh_size, sb)         \
2184 |   __sanitizer_syscall_pre_impl_compat_50___fhstat40(                           \
2185 |       (long long)(fhp), (long long)(fh_size), (long long)(sb))
2186 | #define __sanitizer_syscall_post_compat_50___fhstat40(res, fhp, fh_size, sb)   \
2187 |   __sanitizer_syscall_post_impl_compat_50___fhstat40(                          \
2188 |       res, (long long)(fhp), (long long)(fh_size), (long long)(sb))
2189 | #define __sanitizer_syscall_pre_aio_cancel(fildes, aiocbp)                     \
2190 |   __sanitizer_syscall_pre_impl_aio_cancel((long long)(fildes),                 \
```
- **Line 2181 / 第 2181 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2182 / 第 2182 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2183 / 第 2183 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2184 / 第 2184 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2185 / 第 2185 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2186 / 第 2186 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2187 / 第 2187 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2188 / 第 2188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2189 / 第 2189 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2190 / 第 2190 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2191-2200 / 第 2191-2200 行
```cpp
2191 |                                           (long long)(aiocbp))
2192 | #define __sanitizer_syscall_post_aio_cancel(res, fildes, aiocbp)               \
2193 |   __sanitizer_syscall_post_impl_aio_cancel(res, (long long)(fildes),           \
2194 |                                            (long long)(aiocbp))
2195 | #define __sanitizer_syscall_pre_aio_error(aiocbp)                              \
2196 |   __sanitizer_syscall_pre_impl_aio_error((long long)(aiocbp))
2197 | #define __sanitizer_syscall_post_aio_error(res, aiocbp)                        \
2198 |   __sanitizer_syscall_post_impl_aio_error(res, (long long)(aiocbp))
2199 | #define __sanitizer_syscall_pre_aio_fsync(op, aiocbp)                          \
2200 |   __sanitizer_syscall_pre_impl_aio_fsync((long long)(op), (long long)(aiocbp))
```
- **Line 2191 / 第 2191 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2192 / 第 2192 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2193 / 第 2193 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2194 / 第 2194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2195 / 第 2195 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2196 / 第 2196 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2197 / 第 2197 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2198 / 第 2198 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2199 / 第 2199 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2200 / 第 2200 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2201-2210 / 第 2201-2210 行
```cpp
2201 | #define __sanitizer_syscall_post_aio_fsync(res, op, aiocbp)                    \
2202 |   __sanitizer_syscall_post_impl_aio_fsync(res, (long long)(op),                \
2203 |                                           (long long)(aiocbp))
2204 | #define __sanitizer_syscall_pre_aio_read(aiocbp)                               \
2205 |   __sanitizer_syscall_pre_impl_aio_read((long long)(aiocbp))
2206 | #define __sanitizer_syscall_post_aio_read(res, aiocbp)                         \
2207 |   __sanitizer_syscall_post_impl_aio_read(res, (long long)(aiocbp))
2208 | #define __sanitizer_syscall_pre_aio_return(aiocbp)                             \
2209 |   __sanitizer_syscall_pre_impl_aio_return((long long)(aiocbp))
2210 | #define __sanitizer_syscall_post_aio_return(res, aiocbp)                       \
```
- **Line 2201 / 第 2201 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2202 / 第 2202 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2203 / 第 2203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2204 / 第 2204 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2205 / 第 2205 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2206 / 第 2206 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2207 / 第 2207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2208 / 第 2208 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2209 / 第 2209 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2210 / 第 2210 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2211-2220 / 第 2211-2220 行
```cpp
2211 |   __sanitizer_syscall_post_impl_aio_return(res, (long long)(aiocbp))
2212 | #define __sanitizer_syscall_pre_compat_50_aio_suspend(list, nent, timeout)     \
2213 |   __sanitizer_syscall_pre_impl_compat_50_aio_suspend(                          \
2214 |       (long long)(list), (long long)(nent), (long long)(timeout))
2215 | #define __sanitizer_syscall_post_compat_50_aio_suspend(res, list, nent,        \
2216 |                                                        timeout)                \
2217 |   __sanitizer_syscall_post_impl_compat_50_aio_suspend(                         \
2218 |       res, (long long)(list), (long long)(nent), (long long)(timeout))
2219 | #define __sanitizer_syscall_pre_aio_write(aiocbp)                              \
2220 |   __sanitizer_syscall_pre_impl_aio_write((long long)(aiocbp))
```
- **Line 2211 / 第 2211 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2212 / 第 2212 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2213 / 第 2213 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2214 / 第 2214 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2215 / 第 2215 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2216 / 第 2216 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2217 / 第 2217 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2218 / 第 2218 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2219 / 第 2219 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2220 / 第 2220 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2221-2230 / 第 2221-2230 行
```cpp
2221 | #define __sanitizer_syscall_post_aio_write(res, aiocbp)                        \
2222 |   __sanitizer_syscall_post_impl_aio_write(res, (long long)(aiocbp))
2223 | #define __sanitizer_syscall_pre_lio_listio(mode, list, nent, sig)              \
2224 |   __sanitizer_syscall_pre_impl_lio_listio((long long)(mode),                   \
2225 |                                           (long long)(list),                   \
2226 |                                           (long long)(nent), (long long)(sig))
2227 | #define __sanitizer_syscall_post_lio_listio(res, mode, list, nent, sig)        \
2228 |   __sanitizer_syscall_post_impl_lio_listio(                                    \
2229 |       res, (long long)(mode), (long long)(list), (long long)(nent),            \
2230 |       (long long)(sig))
```
- **Line 2221 / 第 2221 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2222 / 第 2222 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2223 / 第 2223 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2224 / 第 2224 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2225 / 第 2225 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2226 / 第 2226 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2227 / 第 2227 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2228 / 第 2228 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2229 / 第 2229 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2230 / 第 2230 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2231-2240 / 第 2231-2240 行
```cpp
2231 | /* syscall 407 has been skipped */
2232 | /* syscall 408 has been skipped */
2233 | /* syscall 409 has been skipped */
2234 | #define __sanitizer_syscall_pre___mount50(type, path, flags, data, data_len)   \
2235 |   __sanitizer_syscall_pre_impl___mount50(                                      \
2236 |       (long long)(type), (long long)(path), (long long)(flags),                \
2237 |       (long long)(data), (long long)(data_len))
2238 | #define __sanitizer_syscall_post___mount50(res, type, path, flags, data,       \
2239 |                                            data_len)                           \
2240 |   __sanitizer_syscall_post_impl___mount50(                                     \
```
- **Line 2231 / 第 2231 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2232 / 第 2232 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2233 / 第 2233 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2234 / 第 2234 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2235 / 第 2235 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2236 / 第 2236 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2237 / 第 2237 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2238 / 第 2238 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2239 / 第 2239 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2240 / 第 2240 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2241-2250 / 第 2241-2250 行
```cpp
2241 |       res, (long long)(type), (long long)(path), (long long)(flags),           \
2242 |       (long long)(data), (long long)(data_len))
2243 | #define __sanitizer_syscall_pre_mremap(old_address, old_size, new_address,     \
2244 |                                        new_size, flags)                        \
2245 |   __sanitizer_syscall_pre_impl_mremap(                                         \
2246 |       (long long)(old_address), (long long)(old_size),                         \
2247 |       (long long)(new_address), (long long)(new_size), (long long)(flags))
2248 | #define __sanitizer_syscall_post_mremap(res, old_address, old_size,            \
2249 |                                         new_address, new_size, flags)          \
2250 |   __sanitizer_syscall_post_impl_mremap(                                        \
```
- **Line 2241 / 第 2241 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2242 / 第 2242 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2243 / 第 2243 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2244 / 第 2244 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2245 / 第 2245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2246 / 第 2246 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2247 / 第 2247 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2248 / 第 2248 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2249 / 第 2249 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2250 / 第 2250 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2251-2260 / 第 2251-2260 行
```cpp
2251 |       res, (long long)(old_address), (long long)(old_size),                    \
2252 |       (long long)(new_address), (long long)(new_size), (long long)(flags))
2253 | #define __sanitizer_syscall_pre_pset_create(psid)                              \
2254 |   __sanitizer_syscall_pre_impl_pset_create((long long)(psid))
2255 | #define __sanitizer_syscall_post_pset_create(res, psid)                        \
2256 |   __sanitizer_syscall_post_impl_pset_create(res, (long long)(psid))
2257 | #define __sanitizer_syscall_pre_pset_destroy(psid)                             \
2258 |   __sanitizer_syscall_pre_impl_pset_destroy((long long)(psid))
2259 | #define __sanitizer_syscall_post_pset_destroy(res, psid)                       \
2260 |   __sanitizer_syscall_post_impl_pset_destroy(res, (long long)(psid))
```
- **Line 2251 / 第 2251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2252 / 第 2252 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2253 / 第 2253 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2254 / 第 2254 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2255 / 第 2255 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2256 / 第 2256 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2257 / 第 2257 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2258 / 第 2258 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2259 / 第 2259 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2260 / 第 2260 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2261-2270 / 第 2261-2270 行
```cpp
2261 | #define __sanitizer_syscall_pre_pset_assign(psid, cpuid, opsid)                \
2262 |   __sanitizer_syscall_pre_impl_pset_assign(                                    \
2263 |       (long long)(psid), (long long)(cpuid), (long long)(opsid))
2264 | #define __sanitizer_syscall_post_pset_assign(res, psid, cpuid, opsid)          \
2265 |   __sanitizer_syscall_post_impl_pset_assign(                                   \
2266 |       res, (long long)(psid), (long long)(cpuid), (long long)(opsid))
2267 | #define __sanitizer_syscall_pre__pset_bind(idtype, first_id, second_id, psid,  \
2268 |                                            opsid)                              \
2269 |   __sanitizer_syscall_pre_impl__pset_bind(                                     \
2270 |       (long long)(idtype), (long long)(first_id), (long long)(second_id),      \
```
- **Line 2261 / 第 2261 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2262 / 第 2262 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2263 / 第 2263 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2264 / 第 2264 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2265 / 第 2265 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2266 / 第 2266 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2267 / 第 2267 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2268 / 第 2268 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2269 / 第 2269 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2270 / 第 2270 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2271-2280 / 第 2271-2280 行
```cpp
2271 |       (long long)(psid), (long long)(opsid))
2272 | #define __sanitizer_syscall_post__pset_bind(res, idtype, first_id, second_id,  \
2273 |                                             psid, opsid)                       \
2274 |   __sanitizer_syscall_post_impl__pset_bind(                                    \
2275 |       res, (long long)(idtype), (long long)(first_id), (long long)(second_id), \
2276 |       (long long)(psid), (long long)(opsid))
2277 | #define __sanitizer_syscall_pre___posix_fadvise50(fd, PAD, offset, len,        \
2278 |                                                   advice)                      \
2279 |   __sanitizer_syscall_pre_impl___posix_fadvise50(                              \
2280 |       (long long)(fd), (long long)(PAD), (long long)(offset),                  \
```
- **Line 2271 / 第 2271 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2272 / 第 2272 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2273 / 第 2273 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2274 / 第 2274 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2275 / 第 2275 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2276 / 第 2276 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2277 / 第 2277 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2278 / 第 2278 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2279 / 第 2279 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2280 / 第 2280 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2281-2290 / 第 2281-2290 行
```cpp
2281 |       (long long)(len), (long long)(advice))
2282 | #define __sanitizer_syscall_post___posix_fadvise50(res, fd, PAD, offset, len,  \
2283 |                                                    advice)                     \
2284 |   __sanitizer_syscall_post_impl___posix_fadvise50(                             \
2285 |       res, (long long)(fd), (long long)(PAD), (long long)(offset),             \
2286 |       (long long)(len), (long long)(advice))
2287 | #define __sanitizer_syscall_pre___select50(nd, in, ou, ex, tv)                 \
2288 |   __sanitizer_syscall_pre_impl___select50((long long)(nd), (long long)(in),    \
2289 |                                           (long long)(ou), (long long)(ex),    \
2290 |                                           (long long)(tv))
```
- **Line 2281 / 第 2281 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2282 / 第 2282 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2283 / 第 2283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2284 / 第 2284 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2285 / 第 2285 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2286 / 第 2286 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2287 / 第 2287 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2288 / 第 2288 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2289 / 第 2289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2290 / 第 2290 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2291-2300 / 第 2291-2300 行
```cpp
2291 | #define __sanitizer_syscall_post___select50(res, nd, in, ou, ex, tv)           \
2292 |   __sanitizer_syscall_post_impl___select50(res, (long long)(nd),               \
2293 |                                            (long long)(in), (long long)(ou),   \
2294 |                                            (long long)(ex), (long long)(tv))
2295 | #define __sanitizer_syscall_pre___gettimeofday50(tp, tzp)                      \
2296 |   __sanitizer_syscall_pre_impl___gettimeofday50((long long)(tp),               \
2297 |                                                 (long long)(tzp))
2298 | #define __sanitizer_syscall_post___gettimeofday50(res, tp, tzp)                \
2299 |   __sanitizer_syscall_post_impl___gettimeofday50(res, (long long)(tp),         \
2300 |                                                  (long long)(tzp))
```
- **Line 2291 / 第 2291 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2292 / 第 2292 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2293 / 第 2293 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2294 / 第 2294 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2295 / 第 2295 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2296 / 第 2296 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2297 / 第 2297 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2298 / 第 2298 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2299 / 第 2299 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2300 / 第 2300 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2301-2310 / 第 2301-2310 行
```cpp
2301 | #define __sanitizer_syscall_pre___settimeofday50(tv, tzp)                      \
2302 |   __sanitizer_syscall_pre_impl___settimeofday50((long long)(tv),               \
2303 |                                                 (long long)(tzp))
2304 | #define __sanitizer_syscall_post___settimeofday50(res, tv, tzp)                \
2305 |   __sanitizer_syscall_post_impl___settimeofday50(res, (long long)(tv),         \
2306 |                                                  (long long)(tzp))
2307 | #define __sanitizer_syscall_pre___utimes50(path, tptr)                         \
2308 |   __sanitizer_syscall_pre_impl___utimes50((long long)(path), (long long)(tptr))
2309 | #define __sanitizer_syscall_post___utimes50(res, path, tptr)                   \
2310 |   __sanitizer_syscall_post_impl___utimes50(res, (long long)(path),             \
```
- **Line 2301 / 第 2301 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2302 / 第 2302 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2303 / 第 2303 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2304 / 第 2304 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2305 / 第 2305 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2306 / 第 2306 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2307 / 第 2307 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2308 / 第 2308 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2309 / 第 2309 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2310 / 第 2310 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2311-2320 / 第 2311-2320 行
```cpp
2311 |                                            (long long)(tptr))
2312 | #define __sanitizer_syscall_pre___adjtime50(delta, olddelta)                   \
2313 |   __sanitizer_syscall_pre_impl___adjtime50((long long)(delta),                 \
2314 |                                            (long long)(olddelta))
2315 | #define __sanitizer_syscall_post___adjtime50(res, delta, olddelta)             \
2316 |   __sanitizer_syscall_post_impl___adjtime50(res, (long long)(delta),           \
2317 |                                             (long long)(olddelta))
2318 | #define __sanitizer_syscall_pre___lfs_segwait50(fsidp, tv)                     \
2319 |   __sanitizer_syscall_pre_impl___lfs_segwait50((long long)(fsidp),             \
2320 |                                                (long long)(tv))
```
- **Line 2311 / 第 2311 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2312 / 第 2312 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2313 / 第 2313 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2314 / 第 2314 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2315 / 第 2315 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2316 / 第 2316 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2317 / 第 2317 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2318 / 第 2318 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2319 / 第 2319 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2320 / 第 2320 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2321-2330 / 第 2321-2330 行
```cpp
2321 | #define __sanitizer_syscall_post___lfs_segwait50(res, fsidp, tv)               \
2322 |   __sanitizer_syscall_post_impl___lfs_segwait50(res, (long long)(fsidp),       \
2323 |                                                 (long long)(tv))
2324 | #define __sanitizer_syscall_pre___futimes50(fd, tptr)                          \
2325 |   __sanitizer_syscall_pre_impl___futimes50((long long)(fd), (long long)(tptr))
2326 | #define __sanitizer_syscall_post___futimes50(res, fd, tptr)                    \
2327 |   __sanitizer_syscall_post_impl___futimes50(res, (long long)(fd),              \
2328 |                                             (long long)(tptr))
2329 | #define __sanitizer_syscall_pre___lutimes50(path, tptr)                        \
2330 |   __sanitizer_syscall_pre_impl___lutimes50((long long)(path), (long long)(tptr))
```
- **Line 2321 / 第 2321 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2322 / 第 2322 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2323 / 第 2323 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2324 / 第 2324 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2325 / 第 2325 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2326 / 第 2326 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2327 / 第 2327 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2328 / 第 2328 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2329 / 第 2329 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2330 / 第 2330 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2331-2340 / 第 2331-2340 行
```cpp
2331 | #define __sanitizer_syscall_post___lutimes50(res, path, tptr)                  \
2332 |   __sanitizer_syscall_post_impl___lutimes50(res, (long long)(path),            \
2333 |                                             (long long)(tptr))
2334 | #define __sanitizer_syscall_pre___setitimer50(which, itv, oitv)                \
2335 |   __sanitizer_syscall_pre_impl___setitimer50(                                  \
2336 |       (long long)(which), (long long)(itv), (long long)(oitv))
2337 | #define __sanitizer_syscall_post___setitimer50(res, which, itv, oitv)          \
2338 |   __sanitizer_syscall_post_impl___setitimer50(                                 \
2339 |       res, (long long)(which), (long long)(itv), (long long)(oitv))
2340 | #define __sanitizer_syscall_pre___getitimer50(which, itv)                      \
```
- **Line 2331 / 第 2331 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2332 / 第 2332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2333 / 第 2333 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2334 / 第 2334 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2335 / 第 2335 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2336 / 第 2336 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2337 / 第 2337 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2338 / 第 2338 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2339 / 第 2339 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2340 / 第 2340 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2341-2350 / 第 2341-2350 行
```cpp
2341 |   __sanitizer_syscall_pre_impl___getitimer50((long long)(which),               \
2342 |                                              (long long)(itv))
2343 | #define __sanitizer_syscall_post___getitimer50(res, which, itv)                \
2344 |   __sanitizer_syscall_post_impl___getitimer50(res, (long long)(which),         \
2345 |                                               (long long)(itv))
2346 | #define __sanitizer_syscall_pre___clock_gettime50(clock_id, tp)                \
2347 |   __sanitizer_syscall_pre_impl___clock_gettime50((long long)(clock_id),        \
2348 |                                                  (long long)(tp))
2349 | #define __sanitizer_syscall_post___clock_gettime50(res, clock_id, tp)          \
2350 |   __sanitizer_syscall_post_impl___clock_gettime50(res, (long long)(clock_id),  \
```
- **Line 2341 / 第 2341 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2342 / 第 2342 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2343 / 第 2343 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2344 / 第 2344 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2345 / 第 2345 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2346 / 第 2346 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2347 / 第 2347 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2348 / 第 2348 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2349 / 第 2349 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2350 / 第 2350 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2351-2360 / 第 2351-2360 行
```cpp
2351 |                                                   (long long)(tp))
2352 | #define __sanitizer_syscall_pre___clock_settime50(clock_id, tp)                \
2353 |   __sanitizer_syscall_pre_impl___clock_settime50((long long)(clock_id),        \
2354 |                                                  (long long)(tp))
2355 | #define __sanitizer_syscall_post___clock_settime50(res, clock_id, tp)          \
2356 |   __sanitizer_syscall_post_impl___clock_settime50(res, (long long)(clock_id),  \
2357 |                                                   (long long)(tp))
2358 | #define __sanitizer_syscall_pre___clock_getres50(clock_id, tp)                 \
2359 |   __sanitizer_syscall_pre_impl___clock_getres50((long long)(clock_id),         \
2360 |                                                 (long long)(tp))
```
- **Line 2351 / 第 2351 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2352 / 第 2352 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2353 / 第 2353 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2354 / 第 2354 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2355 / 第 2355 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2356 / 第 2356 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2357 / 第 2357 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2358 / 第 2358 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2359 / 第 2359 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2360 / 第 2360 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2361-2370 / 第 2361-2370 行
```cpp
2361 | #define __sanitizer_syscall_post___clock_getres50(res, clock_id, tp)           \
2362 |   __sanitizer_syscall_post_impl___clock_getres50(res, (long long)(clock_id),   \
2363 |                                                  (long long)(tp))
2364 | #define __sanitizer_syscall_pre___nanosleep50(rqtp, rmtp)                      \
2365 |   __sanitizer_syscall_pre_impl___nanosleep50((long long)(rqtp),                \
2366 |                                              (long long)(rmtp))
2367 | #define __sanitizer_syscall_post___nanosleep50(res, rqtp, rmtp)                \
2368 |   __sanitizer_syscall_post_impl___nanosleep50(res, (long long)(rqtp),          \
2369 |                                               (long long)(rmtp))
2370 | #define __sanitizer_syscall_pre_____sigtimedwait50(set, info, timeout)         \
```
- **Line 2361 / 第 2361 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2362 / 第 2362 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2363 / 第 2363 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2364 / 第 2364 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2365 / 第 2365 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2366 / 第 2366 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2367 / 第 2367 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2368 / 第 2368 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2369 / 第 2369 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2370 / 第 2370 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2371-2380 / 第 2371-2380 行
```cpp
2371 |   __sanitizer_syscall_pre_impl_____sigtimedwait50(                             \
2372 |       (long long)(set), (long long)(info), (long long)(timeout))
2373 | #define __sanitizer_syscall_post_____sigtimedwait50(res, set, info, timeout)   \
2374 |   __sanitizer_syscall_post_impl_____sigtimedwait50(                            \
2375 |       res, (long long)(set), (long long)(info), (long long)(timeout))
2376 | #define __sanitizer_syscall_pre___mq_timedsend50(mqdes, msg_ptr, msg_len,      \
2377 |                                                  msg_prio, abs_timeout)        \
2378 |   __sanitizer_syscall_pre_impl___mq_timedsend50(                               \
2379 |       (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),          \
2380 |       (long long)(msg_prio), (long long)(abs_timeout))
```
- **Line 2371 / 第 2371 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2372 / 第 2372 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2373 / 第 2373 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2374 / 第 2374 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2375 / 第 2375 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2376 / 第 2376 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2377 / 第 2377 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2378 / 第 2378 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2379 / 第 2379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2380 / 第 2380 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2381-2390 / 第 2381-2390 行
```cpp
2381 | #define __sanitizer_syscall_post___mq_timedsend50(                             \
2382 |     res, mqdes, msg_ptr, msg_len, msg_prio, abs_timeout)                       \
2383 |   __sanitizer_syscall_post_impl___mq_timedsend50(                              \
2384 |       res, (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),     \
2385 |       (long long)(msg_prio), (long long)(abs_timeout))
2386 | #define __sanitizer_syscall_pre___mq_timedreceive50(mqdes, msg_ptr, msg_len,   \
2387 |                                                     msg_prio, abs_timeout)     \
2388 |   __sanitizer_syscall_pre_impl___mq_timedreceive50(                            \
2389 |       (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),          \
2390 |       (long long)(msg_prio), (long long)(abs_timeout))
```
- **Line 2381 / 第 2381 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2382 / 第 2382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2383 / 第 2383 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2384 / 第 2384 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2385 / 第 2385 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2386 / 第 2386 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2387 / 第 2387 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2388 / 第 2388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2389 / 第 2389 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2390 / 第 2390 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2391-2400 / 第 2391-2400 行
```cpp
2391 | #define __sanitizer_syscall_post___mq_timedreceive50(                          \
2392 |     res, mqdes, msg_ptr, msg_len, msg_prio, abs_timeout)                       \
2393 |   __sanitizer_syscall_post_impl___mq_timedreceive50(                           \
2394 |       res, (long long)(mqdes), (long long)(msg_ptr), (long long)(msg_len),     \
2395 |       (long long)(msg_prio), (long long)(abs_timeout))
2396 | #define __sanitizer_syscall_pre_compat_60__lwp_park(ts, unpark, hint,          \
2397 |                                                     unparkhint)                \
2398 |   __sanitizer_syscall_pre_impl_compat_60__lwp_park(                            \
2399 |       (long long)(ts), (long long)(unpark), (long long)(hint),                 \
2400 |       (long long)(unparkhint))
```
- **Line 2391 / 第 2391 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2392 / 第 2392 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2393 / 第 2393 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2394 / 第 2394 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2395 / 第 2395 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2396 / 第 2396 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2397 / 第 2397 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2398 / 第 2398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2399 / 第 2399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2400 / 第 2400 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2401-2410 / 第 2401-2410 行
```cpp
2401 | #define __sanitizer_syscall_post_compat_60__lwp_park(res, ts, unpark, hint,    \
2402 |                                                      unparkhint)               \
2403 |   __sanitizer_syscall_post_impl_compat_60__lwp_park(                           \
2404 |       res, (long long)(ts), (long long)(unpark), (long long)(hint),            \
2405 |       (long long)(unparkhint))
2406 | #define __sanitizer_syscall_pre___kevent50(fd, changelist, nchanges,           \
2407 |                                            eventlist, nevents, timeout)        \
2408 |   __sanitizer_syscall_pre_impl___kevent50(                                     \
2409 |       (long long)(fd), (long long)(changelist), (long long)(nchanges),         \
2410 |       (long long)(eventlist), (long long)(nevents), (long long)(timeout))
```
- **Line 2401 / 第 2401 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2402 / 第 2402 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2403 / 第 2403 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2404 / 第 2404 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2405 / 第 2405 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2406 / 第 2406 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2407 / 第 2407 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2408 / 第 2408 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2409 / 第 2409 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2410 / 第 2410 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2411-2420 / 第 2411-2420 行
```cpp
2411 | #define __sanitizer_syscall_post___kevent50(res, fd, changelist, nchanges,     \
2412 |                                             eventlist, nevents, timeout)       \
2413 |   __sanitizer_syscall_post_impl___kevent50(                                    \
2414 |       res, (long long)(fd), (long long)(changelist), (long long)(nchanges),    \
2415 |       (long long)(eventlist), (long long)(nevents), (long long)(timeout))
2416 | #define __sanitizer_syscall_pre___pselect50(nd, in, ou, ex, ts, mask)          \
2417 |   __sanitizer_syscall_pre_impl___pselect50((long long)(nd), (long long)(in),   \
2418 |                                            (long long)(ou), (long long)(ex),   \
2419 |                                            (long long)(ts), (long long)(mask))
2420 | #define __sanitizer_syscall_post___pselect50(res, nd, in, ou, ex, ts, mask)    \
```
- **Line 2411 / 第 2411 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2412 / 第 2412 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2413 / 第 2413 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2414 / 第 2414 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2415 / 第 2415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2416 / 第 2416 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2417 / 第 2417 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2418 / 第 2418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2419 / 第 2419 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2420 / 第 2420 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2421-2430 / 第 2421-2430 行
```cpp
2421 |   __sanitizer_syscall_post_impl___pselect50(                                   \
2422 |       res, (long long)(nd), (long long)(in), (long long)(ou), (long long)(ex), \
2423 |       (long long)(ts), (long long)(mask))
2424 | #define __sanitizer_syscall_pre___pollts50(fds, nfds, ts, mask)                \
2425 |   __sanitizer_syscall_pre_impl___pollts50((long long)(fds), (long long)(nfds), \
2426 |                                           (long long)(ts), (long long)(mask))
2427 | #define __sanitizer_syscall_post___pollts50(res, fds, nfds, ts, mask)          \
2428 |   __sanitizer_syscall_post_impl___pollts50(res, (long long)(fds),              \
2429 |                                            (long long)(nfds), (long long)(ts), \
2430 |                                            (long long)(mask))
```
- **Line 2421 / 第 2421 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2422 / 第 2422 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2423 / 第 2423 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2424 / 第 2424 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2425 / 第 2425 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2426 / 第 2426 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2427 / 第 2427 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2428 / 第 2428 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2429 / 第 2429 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2430 / 第 2430 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2431-2440 / 第 2431-2440 行
```cpp
2431 | #define __sanitizer_syscall_pre___aio_suspend50(list, nent, timeout)           \
2432 |   __sanitizer_syscall_pre_impl___aio_suspend50(                                \
2433 |       (long long)(list), (long long)(nent), (long long)(timeout))
2434 | #define __sanitizer_syscall_post___aio_suspend50(res, list, nent, timeout)     \
2435 |   __sanitizer_syscall_post_impl___aio_suspend50(                               \
2436 |       res, (long long)(list), (long long)(nent), (long long)(timeout))
2437 | #define __sanitizer_syscall_pre___stat50(path, ub)                             \
2438 |   __sanitizer_syscall_pre_impl___stat50((long long)(path), (long long)(ub))
2439 | #define __sanitizer_syscall_post___stat50(res, path, ub)                       \
2440 |   __sanitizer_syscall_post_impl___stat50(res, (long long)(path),               \
```
- **Line 2431 / 第 2431 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2432 / 第 2432 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2433 / 第 2433 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2434 / 第 2434 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2435 / 第 2435 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2436 / 第 2436 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2437 / 第 2437 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2438 / 第 2438 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2439 / 第 2439 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2440 / 第 2440 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2441-2450 / 第 2441-2450 行
```cpp
2441 |                                          (long long)(ub))
2442 | #define __sanitizer_syscall_pre___fstat50(fd, sb)                              \
2443 |   __sanitizer_syscall_pre_impl___fstat50((long long)(fd), (long long)(sb))
2444 | #define __sanitizer_syscall_post___fstat50(res, fd, sb)                        \
2445 |   __sanitizer_syscall_post_impl___fstat50(res, (long long)(fd), (long long)(sb))
2446 | #define __sanitizer_syscall_pre___lstat50(path, ub)                            \
2447 |   __sanitizer_syscall_pre_impl___lstat50((long long)(path), (long long)(ub))
2448 | #define __sanitizer_syscall_post___lstat50(res, path, ub)                      \
2449 |   __sanitizer_syscall_post_impl___lstat50(res, (long long)(path),              \
2450 |                                           (long long)(ub))
```
- **Line 2441 / 第 2441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2442 / 第 2442 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2443 / 第 2443 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2444 / 第 2444 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2445 / 第 2445 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2446 / 第 2446 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2447 / 第 2447 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2448 / 第 2448 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2449 / 第 2449 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2450 / 第 2450 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2451-2460 / 第 2451-2460 行
```cpp
2451 | #define __sanitizer_syscall_pre_____semctl50(semid, semnum, cmd, arg)          \
2452 |   __sanitizer_syscall_pre_impl_____semctl50(                                   \
2453 |       (long long)(semid), (long long)(semnum), (long long)(cmd),               \
2454 |       (long long)(arg))
2455 | #define __sanitizer_syscall_post_____semctl50(res, semid, semnum, cmd, arg)    \
2456 |   __sanitizer_syscall_post_impl_____semctl50(                                  \
2457 |       res, (long long)(semid), (long long)(semnum), (long long)(cmd),          \
2458 |       (long long)(arg))
2459 | #define __sanitizer_syscall_pre___shmctl50(shmid, cmd, buf)                    \
2460 |   __sanitizer_syscall_pre_impl___shmctl50((long long)(shmid),                  \
```
- **Line 2451 / 第 2451 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2452 / 第 2452 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2453 / 第 2453 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2454 / 第 2454 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2455 / 第 2455 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2456 / 第 2456 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2457 / 第 2457 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2458 / 第 2458 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2459 / 第 2459 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2460 / 第 2460 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2461-2470 / 第 2461-2470 行
```cpp
2461 |                                           (long long)(cmd), (long long)(buf))
2462 | #define __sanitizer_syscall_post___shmctl50(res, shmid, cmd, buf)              \
2463 |   __sanitizer_syscall_post_impl___shmctl50(res, (long long)(shmid),            \
2464 |                                            (long long)(cmd), (long long)(buf))
2465 | #define __sanitizer_syscall_pre___msgctl50(msqid, cmd, buf)                    \
2466 |   __sanitizer_syscall_pre_impl___msgctl50((long long)(msqid),                  \
2467 |                                           (long long)(cmd), (long long)(buf))
2468 | #define __sanitizer_syscall_post___msgctl50(res, msqid, cmd, buf)              \
2469 |   __sanitizer_syscall_post_impl___msgctl50(res, (long long)(msqid),            \
2470 |                                            (long long)(cmd), (long long)(buf))
```
- **Line 2461 / 第 2461 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2462 / 第 2462 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2463 / 第 2463 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2464 / 第 2464 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2465 / 第 2465 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2466 / 第 2466 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2467 / 第 2467 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2468 / 第 2468 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2469 / 第 2469 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2470 / 第 2470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2471-2480 / 第 2471-2480 行
```cpp
2471 | #define __sanitizer_syscall_pre___getrusage50(who, rusage)                     \
2472 |   __sanitizer_syscall_pre_impl___getrusage50((long long)(who),                 \
2473 |                                              (long long)(rusage))
2474 | #define __sanitizer_syscall_post___getrusage50(res, who, rusage)               \
2475 |   __sanitizer_syscall_post_impl___getrusage50(res, (long long)(who),           \
2476 |                                               (long long)(rusage))
2477 | #define __sanitizer_syscall_pre___timer_settime50(timerid, flags, value,       \
2478 |                                                   ovalue)                      \
2479 |   __sanitizer_syscall_pre_impl___timer_settime50(                              \
2480 |       (long long)(timerid), (long long)(flags), (long long)(value),            \
```
- **Line 2471 / 第 2471 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2472 / 第 2472 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2473 / 第 2473 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2474 / 第 2474 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2475 / 第 2475 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2476 / 第 2476 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2477 / 第 2477 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2478 / 第 2478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2479 / 第 2479 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2480 / 第 2480 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2481-2490 / 第 2481-2490 行
```cpp
2481 |       (long long)(ovalue))
2482 | #define __sanitizer_syscall_post___timer_settime50(res, timerid, flags, value, \
2483 |                                                    ovalue)                     \
2484 |   __sanitizer_syscall_post_impl___timer_settime50(                             \
2485 |       res, (long long)(timerid), (long long)(flags), (long long)(value),       \
2486 |       (long long)(ovalue))
2487 | #define __sanitizer_syscall_pre___timer_gettime50(timerid, value)              \
2488 |   __sanitizer_syscall_pre_impl___timer_gettime50((long long)(timerid),         \
2489 |                                                  (long long)(value))
2490 | #define __sanitizer_syscall_post___timer_gettime50(res, timerid, value)        \
```
- **Line 2481 / 第 2481 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2482 / 第 2482 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2483 / 第 2483 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2484 / 第 2484 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2485 / 第 2485 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2486 / 第 2486 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2487 / 第 2487 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2488 / 第 2488 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2489 / 第 2489 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2490 / 第 2490 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2491-2500 / 第 2491-2500 行
```cpp
2491 |   __sanitizer_syscall_post_impl___timer_gettime50(res, (long long)(timerid),   \
2492 |                                                   (long long)(value))
2493 | #if defined(NTP) || !defined(_KERNEL_OPT)
2494 | #define __sanitizer_syscall_pre___ntp_gettime50(ntvp)                          \
2495 |   __sanitizer_syscall_pre_impl___ntp_gettime50((long long)(ntvp))
2496 | #define __sanitizer_syscall_post___ntp_gettime50(res, ntvp)                    \
2497 |   __sanitizer_syscall_post_impl___ntp_gettime50(res, (long long)(ntvp))
2498 | #else
2499 | /* syscall 448 has been skipped */
2500 | #endif
```
- **Line 2491 / 第 2491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2492 / 第 2492 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2493 / 第 2493 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 2494 / 第 2494 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2495 / 第 2495 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2496 / 第 2496 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2497 / 第 2497 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2498 / 第 2498 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 2499 / 第 2499 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2500 / 第 2500 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

### Lines 2501-2510 / 第 2501-2510 行
```cpp
2501 | #define __sanitizer_syscall_pre___wait450(pid, status, options, rusage)        \
2502 |   __sanitizer_syscall_pre_impl___wait450(                                      \
2503 |       (long long)(pid), (long long)(status), (long long)(options),             \
2504 |       (long long)(rusage))
2505 | #define __sanitizer_syscall_post___wait450(res, pid, status, options, rusage)  \
2506 |   __sanitizer_syscall_post_impl___wait450(                                     \
2507 |       res, (long long)(pid), (long long)(status), (long long)(options),        \
2508 |       (long long)(rusage))
2509 | #define __sanitizer_syscall_pre___mknod50(path, mode, dev)                     \
2510 |   __sanitizer_syscall_pre_impl___mknod50((long long)(path), (long long)(mode), \
```
- **Line 2501 / 第 2501 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2502 / 第 2502 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2503 / 第 2503 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2504 / 第 2504 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2505 / 第 2505 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2506 / 第 2506 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2507 / 第 2507 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2508 / 第 2508 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2509 / 第 2509 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2510 / 第 2510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2511-2520 / 第 2511-2520 行
```cpp
2511 |                                          (long long)(dev))
2512 | #define __sanitizer_syscall_post___mknod50(res, path, mode, dev)               \
2513 |   __sanitizer_syscall_post_impl___mknod50(res, (long long)(path),              \
2514 |                                           (long long)(mode), (long long)(dev))
2515 | #define __sanitizer_syscall_pre___fhstat50(fhp, fh_size, sb)                   \
2516 |   __sanitizer_syscall_pre_impl___fhstat50(                                     \
2517 |       (long long)(fhp), (long long)(fh_size), (long long)(sb))
2518 | #define __sanitizer_syscall_post___fhstat50(res, fhp, fh_size, sb)             \
2519 |   __sanitizer_syscall_post_impl___fhstat50(                                    \
2520 |       res, (long long)(fhp), (long long)(fh_size), (long long)(sb))
```
- **Line 2511 / 第 2511 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2512 / 第 2512 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2513 / 第 2513 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2514 / 第 2514 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2515 / 第 2515 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2516 / 第 2516 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2517 / 第 2517 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2518 / 第 2518 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2519 / 第 2519 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2520 / 第 2520 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2521-2530 / 第 2521-2530 行
```cpp
2521 | /* syscall 452 has been skipped */
2522 | #define __sanitizer_syscall_pre_pipe2(fildes, flags)                           \
2523 |   __sanitizer_syscall_pre_impl_pipe2((long long)(fildes), (long long)(flags))
2524 | #define __sanitizer_syscall_post_pipe2(res, fildes, flags)                     \
2525 |   __sanitizer_syscall_post_impl_pipe2(res, (long long)(fildes),                \
2526 |                                       (long long)(flags))
2527 | #define __sanitizer_syscall_pre_dup3(from, to, flags)                          \
2528 |   __sanitizer_syscall_pre_impl_dup3((long long)(from), (long long)(to),        \
2529 |                                     (long long)(flags))
2530 | #define __sanitizer_syscall_post_dup3(res, from, to, flags)                    \
```
- **Line 2521 / 第 2521 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2522 / 第 2522 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2523 / 第 2523 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2524 / 第 2524 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2525 / 第 2525 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2526 / 第 2526 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2527 / 第 2527 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2528 / 第 2528 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2529 / 第 2529 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2530 / 第 2530 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2531-2540 / 第 2531-2540 行
```cpp
2531 |   __sanitizer_syscall_post_impl_dup3(res, (long long)(from), (long long)(to),  \
2532 |                                      (long long)(flags))
2533 | #define __sanitizer_syscall_pre_kqueue1(flags)                                 \
2534 |   __sanitizer_syscall_pre_impl_kqueue1((long long)(flags))
2535 | #define __sanitizer_syscall_post_kqueue1(res, flags)                           \
2536 |   __sanitizer_syscall_post_impl_kqueue1(res, (long long)(flags))
2537 | #define __sanitizer_syscall_pre_paccept(s, name, anamelen, mask, flags)        \
2538 |   __sanitizer_syscall_pre_impl_paccept((long long)(s), (long long)(name),      \
2539 |                                        (long long)(anamelen),                  \
2540 |                                        (long long)(mask), (long long)(flags))
```
- **Line 2531 / 第 2531 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2532 / 第 2532 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2533 / 第 2533 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2534 / 第 2534 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2535 / 第 2535 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2536 / 第 2536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2537 / 第 2537 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2538 / 第 2538 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2539 / 第 2539 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2540 / 第 2540 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2541-2550 / 第 2541-2550 行
```cpp
2541 | #define __sanitizer_syscall_post_paccept(res, s, name, anamelen, mask, flags)  \
2542 |   __sanitizer_syscall_post_impl_paccept(                                       \
2543 |       res, (long long)(s), (long long)(name), (long long)(anamelen),           \
2544 |       (long long)(mask), (long long)(flags))
2545 | #define __sanitizer_syscall_pre_linkat(fd1, name1, fd2, name2, flags)          \
2546 |   __sanitizer_syscall_pre_impl_linkat((long long)(fd1), (long long)(name1),    \
2547 |                                       (long long)(fd2), (long long)(name2),    \
2548 |                                       (long long)(flags))
2549 | #define __sanitizer_syscall_post_linkat(res, fd1, name1, fd2, name2, flags)    \
2550 |   __sanitizer_syscall_post_impl_linkat(res, (long long)(fd1),                  \
```
- **Line 2541 / 第 2541 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2542 / 第 2542 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2543 / 第 2543 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2544 / 第 2544 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2545 / 第 2545 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2546 / 第 2546 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2547 / 第 2547 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2548 / 第 2548 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2549 / 第 2549 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2550 / 第 2550 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2551-2560 / 第 2551-2560 行
```cpp
2551 |                                        (long long)(name1), (long long)(fd2),   \
2552 |                                        (long long)(name2), (long long)(flags))
2553 | #define __sanitizer_syscall_pre_renameat(fromfd, from, tofd, to)               \
2554 |   __sanitizer_syscall_pre_impl_renameat((long long)(fromfd),                   \
2555 |                                         (long long)(from), (long long)(tofd),  \
2556 |                                         (long long)(to))
2557 | #define __sanitizer_syscall_post_renameat(res, fromfd, from, tofd, to)         \
2558 |   __sanitizer_syscall_post_impl_renameat(res, (long long)(fromfd),             \
2559 |                                          (long long)(from), (long long)(tofd), \
2560 |                                          (long long)(to))
```
- **Line 2551 / 第 2551 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2552 / 第 2552 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2553 / 第 2553 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2554 / 第 2554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2555 / 第 2555 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2556 / 第 2556 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2557 / 第 2557 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2558 / 第 2558 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2559 / 第 2559 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2560 / 第 2560 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2561-2570 / 第 2561-2570 行
```cpp
2561 | #define __sanitizer_syscall_pre_mkfifoat(fd, path, mode)                       \
2562 |   __sanitizer_syscall_pre_impl_mkfifoat((long long)(fd), (long long)(path),    \
2563 |                                         (long long)(mode))
2564 | #define __sanitizer_syscall_post_mkfifoat(res, fd, path, mode)                 \
2565 |   __sanitizer_syscall_post_impl_mkfifoat(res, (long long)(fd),                 \
2566 |                                          (long long)(path), (long long)(mode))
2567 | #define __sanitizer_syscall_pre_mknodat(fd, path, mode, PAD, dev)              \
2568 |   __sanitizer_syscall_pre_impl_mknodat((long long)(fd), (long long)(path),     \
2569 |                                        (long long)(mode), (long long)(PAD),    \
2570 |                                        (long long)(dev))
```
- **Line 2561 / 第 2561 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2562 / 第 2562 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2563 / 第 2563 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2564 / 第 2564 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2565 / 第 2565 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2566 / 第 2566 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2567 / 第 2567 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2568 / 第 2568 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2569 / 第 2569 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2570 / 第 2570 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2571-2580 / 第 2571-2580 行
```cpp
2571 | #define __sanitizer_syscall_post_mknodat(res, fd, path, mode, PAD, dev)        \
2572 |   __sanitizer_syscall_post_impl_mknodat(res, (long long)(fd),                  \
2573 |                                         (long long)(path), (long long)(mode),  \
2574 |                                         (long long)(PAD), (long long)(dev))
2575 | #define __sanitizer_syscall_pre_mkdirat(fd, path, mode)                        \
2576 |   __sanitizer_syscall_pre_impl_mkdirat((long long)(fd), (long long)(path),     \
2577 |                                        (long long)(mode))
2578 | #define __sanitizer_syscall_post_mkdirat(res, fd, path, mode)                  \
2579 |   __sanitizer_syscall_post_impl_mkdirat(res, (long long)(fd),                  \
2580 |                                         (long long)(path), (long long)(mode))
```
- **Line 2571 / 第 2571 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2572 / 第 2572 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2573 / 第 2573 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2574 / 第 2574 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2575 / 第 2575 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2576 / 第 2576 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2577 / 第 2577 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2578 / 第 2578 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2579 / 第 2579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2580 / 第 2580 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2581-2590 / 第 2581-2590 行
```cpp
2581 | #define __sanitizer_syscall_pre_faccessat(fd, path, amode, flag)               \
2582 |   __sanitizer_syscall_pre_impl_faccessat((long long)(fd), (long long)(path),   \
2583 |                                          (long long)(amode),                   \
2584 |                                          (long long)(flag))
2585 | #define __sanitizer_syscall_post_faccessat(res, fd, path, amode, flag)         \
2586 |   __sanitizer_syscall_post_impl_faccessat(                                     \
2587 |       res, (long long)(fd), (long long)(path), (long long)(amode),             \
2588 |       (long long)(flag))
2589 | #define __sanitizer_syscall_pre_fchmodat(fd, path, mode, flag)                 \
2590 |   __sanitizer_syscall_pre_impl_fchmodat((long long)(fd), (long long)(path),    \
```
- **Line 2581 / 第 2581 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2582 / 第 2582 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2583 / 第 2583 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2584 / 第 2584 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2585 / 第 2585 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2586 / 第 2586 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2587 / 第 2587 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2588 / 第 2588 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2589 / 第 2589 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2590 / 第 2590 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2591-2600 / 第 2591-2600 行
```cpp
2591 |                                         (long long)(mode), (long long)(flag))
2592 | #define __sanitizer_syscall_post_fchmodat(res, fd, path, mode, flag)           \
2593 |   __sanitizer_syscall_post_impl_fchmodat(res, (long long)(fd),                 \
2594 |                                          (long long)(path), (long long)(mode), \
2595 |                                          (long long)(flag))
2596 | #define __sanitizer_syscall_pre_fchownat(fd, path, owner, group, flag)         \
2597 |   __sanitizer_syscall_pre_impl_fchownat((long long)(fd), (long long)(path),    \
2598 |                                         (long long)(owner),                    \
2599 |                                         (long long)(group), (long long)(flag))
2600 | #define __sanitizer_syscall_post_fchownat(res, fd, path, owner, group, flag)   \
```
- **Line 2591 / 第 2591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2592 / 第 2592 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2593 / 第 2593 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2594 / 第 2594 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2595 / 第 2595 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2596 / 第 2596 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2597 / 第 2597 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2598 / 第 2598 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2599 / 第 2599 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2600 / 第 2600 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2601-2610 / 第 2601-2610 行
```cpp
2601 |   __sanitizer_syscall_post_impl_fchownat(                                      \
2602 |       res, (long long)(fd), (long long)(path), (long long)(owner),             \
2603 |       (long long)(group), (long long)(flag))
2604 | #define __sanitizer_syscall_pre_fexecve(fd, argp, envp)                        \
2605 |   __sanitizer_syscall_pre_impl_fexecve((long long)(fd), (long long)(argp),     \
2606 |                                        (long long)(envp))
2607 | #define __sanitizer_syscall_post_fexecve(res, fd, argp, envp)                  \
2608 |   __sanitizer_syscall_post_impl_fexecve(res, (long long)(fd),                  \
2609 |                                         (long long)(argp), (long long)(envp))
2610 | #define __sanitizer_syscall_pre_fstatat(fd, path, buf, flag)                   \
```
- **Line 2601 / 第 2601 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2602 / 第 2602 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2603 / 第 2603 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2604 / 第 2604 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2605 / 第 2605 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2606 / 第 2606 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2607 / 第 2607 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2608 / 第 2608 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2609 / 第 2609 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2610 / 第 2610 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2611-2620 / 第 2611-2620 行
```cpp
2611 |   __sanitizer_syscall_pre_impl_fstatat((long long)(fd), (long long)(path),     \
2612 |                                        (long long)(buf), (long long)(flag))
2613 | #define __sanitizer_syscall_post_fstatat(res, fd, path, buf, flag)             \
2614 |   __sanitizer_syscall_post_impl_fstatat(res, (long long)(fd),                  \
2615 |                                         (long long)(path), (long long)(buf),   \
2616 |                                         (long long)(flag))
2617 | #define __sanitizer_syscall_pre_utimensat(fd, path, tptr, flag)                \
2618 |   __sanitizer_syscall_pre_impl_utimensat((long long)(fd), (long long)(path),   \
2619 |                                          (long long)(tptr), (long long)(flag))
2620 | #define __sanitizer_syscall_post_utimensat(res, fd, path, tptr, flag)          \
```
- **Line 2611 / 第 2611 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2612 / 第 2612 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2613 / 第 2613 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2614 / 第 2614 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2615 / 第 2615 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2616 / 第 2616 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2617 / 第 2617 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2618 / 第 2618 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2619 / 第 2619 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2620 / 第 2620 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2621-2630 / 第 2621-2630 行
```cpp
2621 |   __sanitizer_syscall_post_impl_utimensat(                                     \
2622 |       res, (long long)(fd), (long long)(path), (long long)(tptr),              \
2623 |       (long long)(flag))
2624 | #define __sanitizer_syscall_pre_openat(fd, path, oflags, mode)                 \
2625 |   __sanitizer_syscall_pre_impl_openat((long long)(fd), (long long)(path),      \
2626 |                                       (long long)(oflags), (long long)(mode))
2627 | #define __sanitizer_syscall_post_openat(res, fd, path, oflags, mode)           \
2628 |   __sanitizer_syscall_post_impl_openat(res, (long long)(fd),                   \
2629 |                                        (long long)(path), (long long)(oflags), \
2630 |                                        (long long)(mode))
```
- **Line 2621 / 第 2621 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2622 / 第 2622 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2623 / 第 2623 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2624 / 第 2624 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2625 / 第 2625 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2626 / 第 2626 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2627 / 第 2627 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2628 / 第 2628 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2629 / 第 2629 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2630 / 第 2630 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2631-2640 / 第 2631-2640 行
```cpp
2631 | #define __sanitizer_syscall_pre_readlinkat(fd, path, buf, bufsize)             \
2632 |   __sanitizer_syscall_pre_impl_readlinkat((long long)(fd), (long long)(path),  \
2633 |                                           (long long)(buf),                    \
2634 |                                           (long long)(bufsize))
2635 | #define __sanitizer_syscall_post_readlinkat(res, fd, path, buf, bufsize)       \
2636 |   __sanitizer_syscall_post_impl_readlinkat(                                    \
2637 |       res, (long long)(fd), (long long)(path), (long long)(buf),               \
2638 |       (long long)(bufsize))
2639 | #define __sanitizer_syscall_pre_symlinkat(path1, fd, path2)                    \
2640 |   __sanitizer_syscall_pre_impl_symlinkat((long long)(path1), (long long)(fd),  \
```
- **Line 2631 / 第 2631 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2632 / 第 2632 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2633 / 第 2633 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2634 / 第 2634 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2635 / 第 2635 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2636 / 第 2636 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2637 / 第 2637 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2638 / 第 2638 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2639 / 第 2639 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2640 / 第 2640 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2641-2650 / 第 2641-2650 行
```cpp
2641 |                                          (long long)(path2))
2642 | #define __sanitizer_syscall_post_symlinkat(res, path1, fd, path2)              \
2643 |   __sanitizer_syscall_post_impl_symlinkat(res, (long long)(path1),             \
2644 |                                           (long long)(fd), (long long)(path2))
2645 | #define __sanitizer_syscall_pre_unlinkat(fd, path, flag)                       \
2646 |   __sanitizer_syscall_pre_impl_unlinkat((long long)(fd), (long long)(path),    \
2647 |                                         (long long)(flag))
2648 | #define __sanitizer_syscall_post_unlinkat(res, fd, path, flag)                 \
2649 |   __sanitizer_syscall_post_impl_unlinkat(res, (long long)(fd),                 \
2650 |                                          (long long)(path), (long long)(flag))
```
- **Line 2641 / 第 2641 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2642 / 第 2642 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2643 / 第 2643 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2644 / 第 2644 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2645 / 第 2645 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2646 / 第 2646 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2647 / 第 2647 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2648 / 第 2648 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2649 / 第 2649 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2650 / 第 2650 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2651-2660 / 第 2651-2660 行
```cpp
2651 | #define __sanitizer_syscall_pre_futimens(fd, tptr)                             \
2652 |   __sanitizer_syscall_pre_impl_futimens((long long)(fd), (long long)(tptr))
2653 | #define __sanitizer_syscall_post_futimens(res, fd, tptr)                       \
2654 |   __sanitizer_syscall_post_impl_futimens(res, (long long)(fd),                 \
2655 |                                          (long long)(tptr))
2656 | #define __sanitizer_syscall_pre___quotactl(path, args)                         \
2657 |   __sanitizer_syscall_pre_impl___quotactl((long long)(path), (long long)(args))
2658 | #define __sanitizer_syscall_post___quotactl(res, path, args)                   \
2659 |   __sanitizer_syscall_post_impl___quotactl(res, (long long)(path),             \
2660 |                                            (long long)(args))
```
- **Line 2651 / 第 2651 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2652 / 第 2652 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2653 / 第 2653 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2654 / 第 2654 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2655 / 第 2655 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2656 / 第 2656 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2657 / 第 2657 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2658 / 第 2658 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2659 / 第 2659 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2660 / 第 2660 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2661-2670 / 第 2661-2670 行
```cpp
2661 | #define __sanitizer_syscall_pre_posix_spawn(pid, path, file_actions, attrp,    \
2662 |                                             argv, envp)                        \
2663 |   __sanitizer_syscall_pre_impl_posix_spawn(                                    \
2664 |       (long long)(pid), (long long)(path), (long long)(file_actions),          \
2665 |       (long long)(attrp), (long long)(argv), (long long)(envp))
2666 | #define __sanitizer_syscall_post_posix_spawn(res, pid, path, file_actions,     \
2667 |                                              attrp, argv, envp)                \
2668 |   __sanitizer_syscall_post_impl_posix_spawn(                                   \
2669 |       res, (long long)(pid), (long long)(path), (long long)(file_actions),     \
2670 |       (long long)(attrp), (long long)(argv), (long long)(envp))
```
- **Line 2661 / 第 2661 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2662 / 第 2662 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2663 / 第 2663 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2664 / 第 2664 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2665 / 第 2665 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2666 / 第 2666 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2667 / 第 2667 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2668 / 第 2668 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2669 / 第 2669 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2670 / 第 2670 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2671-2680 / 第 2671-2680 行
```cpp
2671 | #define __sanitizer_syscall_pre_recvmmsg(s, mmsg, vlen, flags, timeout)        \
2672 |   __sanitizer_syscall_pre_impl_recvmmsg((long long)(s), (long long)(mmsg),     \
2673 |                                         (long long)(vlen), (long long)(flags), \
2674 |                                         (long long)(timeout))
2675 | #define __sanitizer_syscall_post_recvmmsg(res, s, mmsg, vlen, flags, timeout)  \
2676 |   __sanitizer_syscall_post_impl_recvmmsg(                                      \
2677 |       res, (long long)(s), (long long)(mmsg), (long long)(vlen),               \
2678 |       (long long)(flags), (long long)(timeout))
2679 | #define __sanitizer_syscall_pre_sendmmsg(s, mmsg, vlen, flags)                 \
2680 |   __sanitizer_syscall_pre_impl_sendmmsg((long long)(s), (long long)(mmsg),     \
```
- **Line 2671 / 第 2671 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2672 / 第 2672 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2673 / 第 2673 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2674 / 第 2674 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2675 / 第 2675 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2676 / 第 2676 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2677 / 第 2677 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2678 / 第 2678 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2679 / 第 2679 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2680 / 第 2680 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2681-2690 / 第 2681-2690 行
```cpp
2681 |                                         (long long)(vlen), (long long)(flags))
2682 | #define __sanitizer_syscall_post_sendmmsg(res, s, mmsg, vlen, flags)           \
2683 |   __sanitizer_syscall_post_impl_sendmmsg(res, (long long)(s),                  \
2684 |                                          (long long)(mmsg), (long long)(vlen), \
2685 |                                          (long long)(flags))
2686 | #define __sanitizer_syscall_pre_clock_nanosleep(clock_id, flags, rqtp, rmtp)   \
2687 |   __sanitizer_syscall_pre_impl_clock_nanosleep(                                \
2688 |       (long long)(clock_id), (long long)(flags), (long long)(rqtp),            \
2689 |       (long long)(rmtp))
2690 | #define __sanitizer_syscall_post_clock_nanosleep(res, clock_id, flags, rqtp,   \
```
- **Line 2681 / 第 2681 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2682 / 第 2682 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2683 / 第 2683 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2684 / 第 2684 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2685 / 第 2685 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2686 / 第 2686 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2687 / 第 2687 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2688 / 第 2688 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2689 / 第 2689 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2690 / 第 2690 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2691-2700 / 第 2691-2700 行
```cpp
2691 |                                                  rmtp)                         \
2692 |   __sanitizer_syscall_post_impl_clock_nanosleep(                               \
2693 |       res, (long long)(clock_id), (long long)(flags), (long long)(rqtp),       \
2694 |       (long long)(rmtp))
2695 | #define __sanitizer_syscall_pre____lwp_park60(clock_id, flags, ts, unpark,     \
2696 |                                               hint, unparkhint)                \
2697 |   __sanitizer_syscall_pre_impl____lwp_park60(                                  \
2698 |       (long long)(clock_id), (long long)(flags), (long long)(ts),              \
2699 |       (long long)(unpark), (long long)(hint), (long long)(unparkhint))
2700 | #define __sanitizer_syscall_post____lwp_park60(res, clock_id, flags, ts,       \
```
- **Line 2691 / 第 2691 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2692 / 第 2692 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2693 / 第 2693 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2694 / 第 2694 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2695 / 第 2695 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2696 / 第 2696 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2697 / 第 2697 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2698 / 第 2698 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2699 / 第 2699 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2700 / 第 2700 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2701-2710 / 第 2701-2710 行
```cpp
2701 |                                                unpark, hint, unparkhint)       \
2702 |   __sanitizer_syscall_post_impl____lwp_park60(                                 \
2703 |       res, (long long)(clock_id), (long long)(flags), (long long)(ts),         \
2704 |       (long long)(unpark), (long long)(hint), (long long)(unparkhint))
2705 | #define __sanitizer_syscall_pre_posix_fallocate(fd, PAD, pos, len)             \
2706 |   __sanitizer_syscall_pre_impl_posix_fallocate(                                \
2707 |       (long long)(fd), (long long)(PAD), (long long)(pos), (long long)(len))
2708 | #define __sanitizer_syscall_post_posix_fallocate(res, fd, PAD, pos, len)       \
2709 |   __sanitizer_syscall_post_impl_posix_fallocate(                               \
2710 |       res, (long long)(fd), (long long)(PAD), (long long)(pos),                \
```
- **Line 2701 / 第 2701 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2702 / 第 2702 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2703 / 第 2703 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2704 / 第 2704 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2705 / 第 2705 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2706 / 第 2706 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2707 / 第 2707 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2708 / 第 2708 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2709 / 第 2709 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2710 / 第 2710 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2711-2720 / 第 2711-2720 行
```cpp
2711 |       (long long)(len))
2712 | #define __sanitizer_syscall_pre_fdiscard(fd, PAD, pos, len)                    \
2713 |   __sanitizer_syscall_pre_impl_fdiscard((long long)(fd), (long long)(PAD),     \
2714 |                                         (long long)(pos), (long long)(len))
2715 | #define __sanitizer_syscall_post_fdiscard(res, fd, PAD, pos, len)              \
2716 |   __sanitizer_syscall_post_impl_fdiscard(res, (long long)(fd),                 \
2717 |                                          (long long)(PAD), (long long)(pos),   \
2718 |                                          (long long)(len))
2719 | #define __sanitizer_syscall_pre_wait6(idtype, id, status, options, wru, info)  \
2720 |   __sanitizer_syscall_pre_impl_wait6(                                          \
```
- **Line 2711 / 第 2711 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2712 / 第 2712 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2713 / 第 2713 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2714 / 第 2714 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2715 / 第 2715 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2716 / 第 2716 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2717 / 第 2717 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2718 / 第 2718 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2719 / 第 2719 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2720 / 第 2720 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2721-2730 / 第 2721-2730 行
```cpp
2721 |       (long long)(idtype), (long long)(id), (long long)(status),               \
2722 |       (long long)(options), (long long)(wru), (long long)(info))
2723 | #define __sanitizer_syscall_post_wait6(res, idtype, id, status, options, wru,  \
2724 |                                        info)                                   \
2725 |   __sanitizer_syscall_post_impl_wait6(                                         \
2726 |       res, (long long)(idtype), (long long)(id), (long long)(status),          \
2727 |       (long long)(options), (long long)(wru), (long long)(info))
2728 | #define __sanitizer_syscall_pre_clock_getcpuclockid2(idtype, id, clock_id)     \
2729 |   __sanitizer_syscall_pre_impl_clock_getcpuclockid2(                           \
2730 |       (long long)(idtype), (long long)(id), (long long)(clock_id))
```
- **Line 2721 / 第 2721 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2722 / 第 2722 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2723 / 第 2723 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2724 / 第 2724 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2725 / 第 2725 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2726 / 第 2726 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2727 / 第 2727 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2728 / 第 2728 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2729 / 第 2729 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2730 / 第 2730 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2731-2740 / 第 2731-2740 行
```cpp
2731 | #define __sanitizer_syscall_post_clock_getcpuclockid2(res, idtype, id,         \
2732 |                                                       clock_id)                \
2733 |   __sanitizer_syscall_post_impl_clock_getcpuclockid2(                          \
2734 |       res, (long long)(idtype), (long long)(id), (long long)(clock_id))
2735 | #define __sanitizer_syscall_pre___getvfsstat90(buf, bufsize, flags)            \
2736 |   __sanitizer_syscall_pre_impl___getvfsstat90(                                 \
2737 |       (long long)(buf), (long long)(bufsize), (long long)(flags))
2738 | #define __sanitizer_syscall_post___getvfsstat90(res, buf, bufsize, flags)      \
2739 |   __sanitizer_syscall_post_impl___getvfsstat90(                                \
2740 |       res, (long long)(buf), (long long)(bufsize), (long long)(flags))
```
- **Line 2731 / 第 2731 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2732 / 第 2732 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2733 / 第 2733 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2734 / 第 2734 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2735 / 第 2735 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2736 / 第 2736 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2737 / 第 2737 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2738 / 第 2738 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2739 / 第 2739 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2740 / 第 2740 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2741-2750 / 第 2741-2750 行
```cpp
2741 | #define __sanitizer_syscall_pre___statvfs190(path, buf, flags)                 \
2742 |   __sanitizer_syscall_pre_impl___statvfs190(                                   \
2743 |       (long long)(path), (long long)(buf), (long long)(flags))
2744 | #define __sanitizer_syscall_post___statvfs190(res, path, buf, flags)           \
2745 |   __sanitizer_syscall_post_impl___statvfs190(                                  \
2746 |       res, (long long)(path), (long long)(buf), (long long)(flags))
2747 | #define __sanitizer_syscall_pre___fstatvfs190(fd, buf, flags)                  \
2748 |   __sanitizer_syscall_pre_impl___fstatvfs190(                                  \
2749 |       (long long)(fd), (long long)(buf), (long long)(flags))
2750 | #define __sanitizer_syscall_post___fstatvfs190(res, fd, buf, flags)            \
```
- **Line 2741 / 第 2741 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2742 / 第 2742 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2743 / 第 2743 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2744 / 第 2744 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2745 / 第 2745 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2746 / 第 2746 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2747 / 第 2747 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2748 / 第 2748 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2749 / 第 2749 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2750 / 第 2750 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2751-2760 / 第 2751-2760 行
```cpp
2751 |   __sanitizer_syscall_post_impl___fstatvfs190(                                 \
2752 |       res, (long long)(fd), (long long)(buf), (long long)(flags))
2753 | #define __sanitizer_syscall_pre___fhstatvfs190(fhp, fh_size, buf, flags)       \
2754 |   __sanitizer_syscall_pre_impl___fhstatvfs190(                                 \
2755 |       (long long)(fhp), (long long)(fh_size), (long long)(buf),                \
2756 |       (long long)(flags))
2757 | #define __sanitizer_syscall_post___fhstatvfs190(res, fhp, fh_size, buf, flags) \
2758 |   __sanitizer_syscall_post_impl___fhstatvfs190(                                \
2759 |       res, (long long)(fhp), (long long)(fh_size), (long long)(buf),           \
2760 |       (long long)(flags))
```
- **Line 2751 / 第 2751 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2752 / 第 2752 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2753 / 第 2753 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2754 / 第 2754 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2755 / 第 2755 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2756 / 第 2756 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2757 / 第 2757 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2758 / 第 2758 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2759 / 第 2759 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2760 / 第 2760 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2761-2770 / 第 2761-2770 行
```cpp
2761 | #define __sanitizer_syscall_pre___acl_get_link(path, type, aclp)               \
2762 |   __sanitizer_syscall_pre_impl___acl_get_link(                                 \
2763 |       (long long)(path), (long long)(type), (long long)(aclp))
2764 | #define __sanitizer_syscall_post___acl_get_link(res, path, type, aclp)         \
2765 |   __sanitizer_syscall_post_impl___acl_get_link(                                \
2766 |       res, (long long)(path), (long long)(type), (long long)(aclp))
2767 | #define __sanitizer_syscall_pre___acl_set_link(path, type, aclp)               \
2768 |   __sanitizer_syscall_pre_impl___acl_set_link(                                 \
2769 |       (long long)(path), (long long)(type), (long long)(aclp))
2770 | #define __sanitizer_syscall_post___acl_set_link(res, path, type, aclp)         \
```
- **Line 2761 / 第 2761 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2762 / 第 2762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2763 / 第 2763 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2764 / 第 2764 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2765 / 第 2765 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2766 / 第 2766 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2767 / 第 2767 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2768 / 第 2768 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2769 / 第 2769 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2770 / 第 2770 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2771-2780 / 第 2771-2780 行
```cpp
2771 |   __sanitizer_syscall_post_impl___acl_set_link(                                \
2772 |       res, (long long)(path), (long long)(type), (long long)(aclp))
2773 | #define __sanitizer_syscall_pre___acl_delete_link(path, type)                  \
2774 |   __sanitizer_syscall_pre_impl___acl_delete_link((long long)(path),            \
2775 |                                                  (long long)(type))
2776 | #define __sanitizer_syscall_post___acl_delete_link(res, path, type)            \
2777 |   __sanitizer_syscall_post_impl___acl_delete_link(res, (long long)(path),      \
2778 |                                                   (long long)(type))
2779 | #define __sanitizer_syscall_pre___acl_aclcheck_link(path, type, aclp)          \
2780 |   __sanitizer_syscall_pre_impl___acl_aclcheck_link(                            \
```
- **Line 2771 / 第 2771 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2772 / 第 2772 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2773 / 第 2773 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2774 / 第 2774 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2775 / 第 2775 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2776 / 第 2776 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2777 / 第 2777 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2778 / 第 2778 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2779 / 第 2779 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2780 / 第 2780 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2781-2790 / 第 2781-2790 行
```cpp
2781 |       (long long)(path), (long long)(type), (long long)(aclp))
2782 | #define __sanitizer_syscall_post___acl_aclcheck_link(res, path, type, aclp)    \
2783 |   __sanitizer_syscall_post_impl___acl_aclcheck_link(                           \
2784 |       res, (long long)(path), (long long)(type), (long long)(aclp))
2785 | #define __sanitizer_syscall_pre___acl_get_file(path, type, aclp)               \
2786 |   __sanitizer_syscall_pre_impl___acl_get_file(                                 \
2787 |       (long long)(path), (long long)(type), (long long)(aclp))
2788 | #define __sanitizer_syscall_post___acl_get_file(res, path, type, aclp)         \
2789 |   __sanitizer_syscall_post_impl___acl_get_file(                                \
2790 |       res, (long long)(path), (long long)(type), (long long)(aclp))
```
- **Line 2781 / 第 2781 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2782 / 第 2782 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2783 / 第 2783 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2784 / 第 2784 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2785 / 第 2785 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2786 / 第 2786 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2787 / 第 2787 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2788 / 第 2788 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2789 / 第 2789 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2790 / 第 2790 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2791-2800 / 第 2791-2800 行
```cpp
2791 | #define __sanitizer_syscall_pre___acl_set_file(path, type, aclp)               \
2792 |   __sanitizer_syscall_pre_impl___acl_set_file(                                 \
2793 |       (long long)(path), (long long)(type), (long long)(aclp))
2794 | #define __sanitizer_syscall_post___acl_set_file(res, path, type, aclp)         \
2795 |   __sanitizer_syscall_post_impl___acl_set_file(                                \
2796 |       res, (long long)(path), (long long)(type), (long long)(aclp))
2797 | #define __sanitizer_syscall_pre___acl_get_fd(filedes, type, aclp)              \
2798 |   __sanitizer_syscall_pre_impl___acl_get_fd(                                   \
2799 |       (long long)(filedes), (long long)(type), (long long)(aclp))
2800 | #define __sanitizer_syscall_post___acl_get_fd(res, filedes, type, aclp)        \
```
- **Line 2791 / 第 2791 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2792 / 第 2792 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2793 / 第 2793 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2794 / 第 2794 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2795 / 第 2795 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2796 / 第 2796 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2797 / 第 2797 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2798 / 第 2798 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2799 / 第 2799 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2800 / 第 2800 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2801-2810 / 第 2801-2810 行
```cpp
2801 |   __sanitizer_syscall_post_impl___acl_get_fd(                                  \
2802 |       res, (long long)(filedes), (long long)(type), (long long)(aclp))
2803 | #define __sanitizer_syscall_pre___acl_set_fd(filedes, type, aclp)              \
2804 |   __sanitizer_syscall_pre_impl___acl_set_fd(                                   \
2805 |       (long long)(filedes), (long long)(type), (long long)(aclp))
2806 | #define __sanitizer_syscall_post___acl_set_fd(res, filedes, type, aclp)        \
2807 |   __sanitizer_syscall_post_impl___acl_set_fd(                                  \
2808 |       res, (long long)(filedes), (long long)(type), (long long)(aclp))
2809 | #define __sanitizer_syscall_pre___acl_delete_file(path, type)                  \
2810 |   __sanitizer_syscall_pre_impl___acl_delete_file((long long)(path),            \
```
- **Line 2801 / 第 2801 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2802 / 第 2802 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2803 / 第 2803 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2804 / 第 2804 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2805 / 第 2805 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2806 / 第 2806 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2807 / 第 2807 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2808 / 第 2808 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2809 / 第 2809 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2810 / 第 2810 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2811-2820 / 第 2811-2820 行
```cpp
2811 |                                                  (long long)(type))
2812 | #define __sanitizer_syscall_post___acl_delete_file(res, path, type)            \
2813 |   __sanitizer_syscall_post_impl___acl_delete_file(res, (long long)(path),      \
2814 |                                                   (long long)(type))
2815 | #define __sanitizer_syscall_pre___acl_delete_fd(filedes, type)                 \
2816 |   __sanitizer_syscall_pre_impl___acl_delete_fd((long long)(filedes),           \
2817 |                                                (long long)(type))
2818 | #define __sanitizer_syscall_post___acl_delete_fd(res, filedes, type)           \
2819 |   __sanitizer_syscall_post_impl___acl_delete_fd(res, (long long)(filedes),     \
2820 |                                                 (long long)(type))
```
- **Line 2811 / 第 2811 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2812 / 第 2812 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2813 / 第 2813 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2814 / 第 2814 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2815 / 第 2815 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2816 / 第 2816 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2817 / 第 2817 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2818 / 第 2818 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2819 / 第 2819 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2820 / 第 2820 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2821-2830 / 第 2821-2830 行
```cpp
2821 | #define __sanitizer_syscall_pre___acl_aclcheck_file(path, type, aclp)          \
2822 |   __sanitizer_syscall_pre_impl___acl_aclcheck_file(                            \
2823 |       (long long)(path), (long long)(type), (long long)(aclp))
2824 | #define __sanitizer_syscall_post___acl_aclcheck_file(res, path, type, aclp)    \
2825 |   __sanitizer_syscall_post_impl___acl_aclcheck_file(                           \
2826 |       res, (long long)(path), (long long)(type), (long long)(aclp))
2827 | #define __sanitizer_syscall_pre___acl_aclcheck_fd(filedes, type, aclp)         \
2828 |   __sanitizer_syscall_pre_impl___acl_aclcheck_fd(                              \
2829 |       (long long)(filedes), (long long)(type), (long long)(aclp))
2830 | #define __sanitizer_syscall_post___acl_aclcheck_fd(res, filedes, type, aclp)   \
```
- **Line 2821 / 第 2821 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2822 / 第 2822 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2823 / 第 2823 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2824 / 第 2824 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2825 / 第 2825 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2826 / 第 2826 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2827 / 第 2827 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2828 / 第 2828 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2829 / 第 2829 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2830 / 第 2830 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2831-2840 / 第 2831-2840 行
```cpp
2831 |   __sanitizer_syscall_post_impl___acl_aclcheck_fd(                             \
2832 |       res, (long long)(filedes), (long long)(type), (long long)(aclp))
2833 | #define __sanitizer_syscall_pre_lpathconf(path, name)                          \
2834 |   __sanitizer_syscall_pre_impl_lpathconf((long long)(path), (long long)(name))
2835 | #define __sanitizer_syscall_post_lpathconf(res, path, name)                    \
2836 |   __sanitizer_syscall_post_impl_lpathconf(res, (long long)(path),              \
2837 |                                           (long long)(name))
2838 | 
2839 | /* Compat with older releases */
2840 | #define __sanitizer_syscall_pre_getvfsstat                                     \
```
- **Line 2831 / 第 2831 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2832 / 第 2832 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2833 / 第 2833 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2834 / 第 2834 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2835 / 第 2835 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2836 / 第 2836 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2837 / 第 2837 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2838 / 第 2838 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2839 / 第 2839 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2840 / 第 2840 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2841-2850 / 第 2841-2850 行
```cpp
2841 |   __sanitizer_syscall_pre_compat_90_getvfsstat
2842 | #define __sanitizer_syscall_post_getvfsstat                                    \
2843 |   __sanitizer_syscall_post_compat_90_getvfsstat
2844 | 
2845 | #define __sanitizer_syscall_pre_statvfs1                                       \
2846 |   __sanitizer_syscall_pre_compat_90_statvfs1
2847 | #define __sanitizer_syscall_post_statvfs1                                      \
2848 |   __sanitizer_syscall_post_compat_90_statvfs1
2849 | 
2850 | #define __sanitizer_syscall_pre_fstatvfs1                                      \
```
- **Line 2841 / 第 2841 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2842 / 第 2842 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2843 / 第 2843 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2844 / 第 2844 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2845 / 第 2845 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2846 / 第 2846 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2847 / 第 2847 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2848 / 第 2848 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2849 / 第 2849 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2850 / 第 2850 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。

### Lines 2851-2860 / 第 2851-2860 行
```cpp
2851 |   __sanitizer_syscall_pre_compat_90_fstatvfs1
2852 | #define __sanitizer_syscall_post_fstatvfs1                                     \
2853 |   __sanitizer_syscall_post_compat_90_fstatvfs1
2854 | 
2855 | #define __sanitizer_syscall_pre___fhstatvfs140                                 \
2856 |   __sanitizer_syscall_pre_compat_90_fhstatvfs1
2857 | #define __sanitizer_syscall_post___fhstatvfs140                                \
2858 |   __sanitizer_syscall_post_compat_90_fhstatvfs1
2859 | 
2860 | #ifdef __cplusplus
```
- **Line 2851 / 第 2851 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2852 / 第 2852 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2853 / 第 2853 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2854 / 第 2854 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2855 / 第 2855 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2856 / 第 2856 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2857 / 第 2857 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 2858 / 第 2858 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2859 / 第 2859 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2860 / 第 2860 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。

### Lines 2861-2870 / 第 2861-2870 行
```cpp
2861 | extern "C" {
2862 | #endif
2863 | 
2864 | // Private declarations. Do not call directly from user code. Use macros above.
2865 | 
2866 | // DO NOT EDIT! THIS FILE HAS BEEN GENERATED!
2867 | 
2868 | void __sanitizer_syscall_pre_impl_syscall(long long code, long long arg0,
2869 |                                           long long arg1, long long arg2,
2870 |                                           long long arg3, long long arg4,
```
- **Line 2861 / 第 2861 行**: EN: Uses C linkage so the declaration keeps a stable ABI. CN: 使用 C 链接约定，以保持稳定 ABI。
- **Line 2862 / 第 2862 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 2863 / 第 2863 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2864 / 第 2864 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2865 / 第 2865 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2866 / 第 2866 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2867 / 第 2867 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 2868 / 第 2868 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2869 / 第 2869 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2870 / 第 2870 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2871-2880 / 第 2871-2880 行
```cpp
2871 |                                           long long arg5, long long arg6,
2872 |                                           long long arg7);
2873 | void __sanitizer_syscall_post_impl_syscall(long long res, long long code,
2874 |                                            long long arg0, long long arg1,
2875 |                                            long long arg2, long long arg3,
2876 |                                            long long arg4, long long arg5,
2877 |                                            long long arg6, long long arg7);
2878 | void __sanitizer_syscall_pre_impl_exit(long long rval);
2879 | void __sanitizer_syscall_post_impl_exit(long long res, long long rval);
2880 | void __sanitizer_syscall_pre_impl_fork(void);
```
- **Line 2871 / 第 2871 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2872 / 第 2872 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2873 / 第 2873 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2874 / 第 2874 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2875 / 第 2875 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2876 / 第 2876 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2877 / 第 2877 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2878 / 第 2878 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_exit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_exit`。
- **Line 2879 / 第 2879 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_exit`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_exit`。
- **Line 2880 / 第 2880 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fork`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fork`。

### Lines 2881-2890 / 第 2881-2890 行
```cpp
2881 | void __sanitizer_syscall_post_impl_fork(long long res);
2882 | void __sanitizer_syscall_pre_impl_read(long long fd, long long buf,
2883 |                                        long long nbyte);
2884 | void __sanitizer_syscall_post_impl_read(long long res, long long fd,
2885 |                                         long long buf, long long nbyte);
2886 | void __sanitizer_syscall_pre_impl_write(long long fd, long long buf,
2887 |                                         long long nbyte);
2888 | void __sanitizer_syscall_post_impl_write(long long res, long long fd,
2889 |                                          long long buf, long long nbyte);
2890 | void __sanitizer_syscall_pre_impl_open(long long path, long long flags,
```
- **Line 2881 / 第 2881 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fork`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fork`。
- **Line 2882 / 第 2882 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2883 / 第 2883 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2884 / 第 2884 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2885 / 第 2885 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2886 / 第 2886 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2887 / 第 2887 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2888 / 第 2888 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2889 / 第 2889 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2890 / 第 2890 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2891-2900 / 第 2891-2900 行
```cpp
2891 |                                        long long mode);
2892 | void __sanitizer_syscall_post_impl_open(long long res, long long path,
2893 |                                         long long flags, long long mode);
2894 | void __sanitizer_syscall_pre_impl_close(long long fd);
2895 | void __sanitizer_syscall_post_impl_close(long long res, long long fd);
2896 | void __sanitizer_syscall_pre_impl_compat_50_wait4(long long pid,
2897 |                                                   long long status,
2898 |                                                   long long options,
2899 |                                                   long long rusage);
2900 | void __sanitizer_syscall_post_impl_compat_50_wait4(long long res, long long pid,
```
- **Line 2891 / 第 2891 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2892 / 第 2892 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2893 / 第 2893 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2894 / 第 2894 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_close`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_close`。
- **Line 2895 / 第 2895 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_close`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_close`。
- **Line 2896 / 第 2896 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2897 / 第 2897 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2898 / 第 2898 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2899 / 第 2899 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2900 / 第 2900 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2901-2910 / 第 2901-2910 行
```cpp
2901 |                                                    long long status,
2902 |                                                    long long options,
2903 |                                                    long long rusage);
2904 | void __sanitizer_syscall_pre_impl_compat_43_ocreat(long long path,
2905 |                                                    long long mode);
2906 | void __sanitizer_syscall_post_impl_compat_43_ocreat(long long res,
2907 |                                                     long long path,
2908 |                                                     long long mode);
2909 | void __sanitizer_syscall_pre_impl_link(long long path, long long link);
2910 | void __sanitizer_syscall_post_impl_link(long long res, long long path,
```
- **Line 2901 / 第 2901 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2902 / 第 2902 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2903 / 第 2903 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2904 / 第 2904 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2905 / 第 2905 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2906 / 第 2906 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2907 / 第 2907 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2908 / 第 2908 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2909 / 第 2909 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_link`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_link`。
- **Line 2910 / 第 2910 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2911-2920 / 第 2911-2920 行
```cpp
2911 |                                         long long link);
2912 | void __sanitizer_syscall_pre_impl_unlink(long long path);
2913 | void __sanitizer_syscall_post_impl_unlink(long long res, long long path);
2914 | /* syscall 11 has been skipped */
2915 | void __sanitizer_syscall_pre_impl_chdir(long long path);
2916 | void __sanitizer_syscall_post_impl_chdir(long long res, long long path);
2917 | void __sanitizer_syscall_pre_impl_fchdir(long long fd);
2918 | void __sanitizer_syscall_post_impl_fchdir(long long res, long long fd);
2919 | void __sanitizer_syscall_pre_impl_compat_50_mknod(long long path,
2920 |                                                   long long mode,
```
- **Line 2911 / 第 2911 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2912 / 第 2912 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_unlink`。
- **Line 2913 / 第 2913 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_unlink`。
- **Line 2914 / 第 2914 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2915 / 第 2915 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chdir`。
- **Line 2916 / 第 2916 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_chdir`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_chdir`。
- **Line 2917 / 第 2917 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchdir`。
- **Line 2918 / 第 2918 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fchdir`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fchdir`。
- **Line 2919 / 第 2919 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2920 / 第 2920 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2921-2930 / 第 2921-2930 行
```cpp
2921 |                                                   long long dev);
2922 | void __sanitizer_syscall_post_impl_compat_50_mknod(long long res,
2923 |                                                    long long path,
2924 |                                                    long long mode,
2925 |                                                    long long dev);
2926 | void __sanitizer_syscall_pre_impl_chmod(long long path, long long mode);
2927 | void __sanitizer_syscall_post_impl_chmod(long long res, long long path,
2928 |                                          long long mode);
2929 | void __sanitizer_syscall_pre_impl_chown(long long path, long long uid,
2930 |                                         long long gid);
```
- **Line 2921 / 第 2921 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2922 / 第 2922 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2923 / 第 2923 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2924 / 第 2924 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2925 / 第 2925 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2926 / 第 2926 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chmod`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chmod`。
- **Line 2927 / 第 2927 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2928 / 第 2928 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2929 / 第 2929 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2930 / 第 2930 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2931-2940 / 第 2931-2940 行
```cpp
2931 | void __sanitizer_syscall_post_impl_chown(long long res, long long path,
2932 |                                          long long uid, long long gid);
2933 | void __sanitizer_syscall_pre_impl_break(long long nsize);
2934 | void __sanitizer_syscall_post_impl_break(long long res, long long nsize);
2935 | void __sanitizer_syscall_pre_impl_compat_20_getfsstat(long long buf,
2936 |                                                       long long bufsize,
2937 |                                                       long long flags);
2938 | void __sanitizer_syscall_post_impl_compat_20_getfsstat(long long res,
2939 |                                                        long long buf,
2940 |                                                        long long bufsize,
```
- **Line 2931 / 第 2931 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2932 / 第 2932 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2933 / 第 2933 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_break`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_break`。
- **Line 2934 / 第 2934 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_break`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_break`。
- **Line 2935 / 第 2935 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2936 / 第 2936 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2937 / 第 2937 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2938 / 第 2938 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2939 / 第 2939 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2940 / 第 2940 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2941-2950 / 第 2941-2950 行
```cpp
2941 |                                                        long long flags);
2942 | void __sanitizer_syscall_pre_impl_compat_43_olseek(long long fd,
2943 |                                                    long long offset,
2944 |                                                    long long whence);
2945 | void __sanitizer_syscall_post_impl_compat_43_olseek(long long res, long long fd,
2946 |                                                     long long offset,
2947 |                                                     long long whence);
2948 | void __sanitizer_syscall_pre_impl_getpid(void);
2949 | void __sanitizer_syscall_post_impl_getpid(long long res);
2950 | void __sanitizer_syscall_pre_impl_compat_40_mount(long long type,
```
- **Line 2941 / 第 2941 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2942 / 第 2942 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2943 / 第 2943 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2944 / 第 2944 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2945 / 第 2945 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2946 / 第 2946 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2947 / 第 2947 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2948 / 第 2948 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpid`。
- **Line 2949 / 第 2949 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getpid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getpid`。
- **Line 2950 / 第 2950 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2951-2960 / 第 2951-2960 行
```cpp
2951 |                                                   long long path,
2952 |                                                   long long flags,
2953 |                                                   long long data);
2954 | void __sanitizer_syscall_post_impl_compat_40_mount(long long res,
2955 |                                                    long long type,
2956 |                                                    long long path,
2957 |                                                    long long flags,
2958 |                                                    long long data);
2959 | void __sanitizer_syscall_pre_impl_unmount(long long path, long long flags);
2960 | void __sanitizer_syscall_post_impl_unmount(long long res, long long path,
```
- **Line 2951 / 第 2951 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2952 / 第 2952 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2953 / 第 2953 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2954 / 第 2954 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2955 / 第 2955 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2956 / 第 2956 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2957 / 第 2957 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2958 / 第 2958 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2959 / 第 2959 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_unmount`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_unmount`。
- **Line 2960 / 第 2960 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2961-2970 / 第 2961-2970 行
```cpp
2961 |                                            long long flags);
2962 | void __sanitizer_syscall_pre_impl_setuid(long long uid);
2963 | void __sanitizer_syscall_post_impl_setuid(long long res, long long uid);
2964 | void __sanitizer_syscall_pre_impl_getuid(void);
2965 | void __sanitizer_syscall_post_impl_getuid(long long res);
2966 | void __sanitizer_syscall_pre_impl_geteuid(void);
2967 | void __sanitizer_syscall_post_impl_geteuid(long long res);
2968 | void __sanitizer_syscall_pre_impl_ptrace(long long req, long long pid,
2969 |                                          long long addr, long long data);
2970 | void __sanitizer_syscall_post_impl_ptrace(long long res, long long req,
```
- **Line 2961 / 第 2961 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2962 / 第 2962 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setuid`。
- **Line 2963 / 第 2963 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setuid`。
- **Line 2964 / 第 2964 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getuid`。
- **Line 2965 / 第 2965 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getuid`。
- **Line 2966 / 第 2966 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_geteuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_geteuid`。
- **Line 2967 / 第 2967 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_geteuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_geteuid`。
- **Line 2968 / 第 2968 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2969 / 第 2969 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2970 / 第 2970 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 2971-2980 / 第 2971-2980 行
```cpp
2971 |                                           long long pid, long long addr,
2972 |                                           long long data);
2973 | void __sanitizer_syscall_pre_impl_recvmsg(long long s, long long msg,
2974 |                                           long long flags);
2975 | void __sanitizer_syscall_post_impl_recvmsg(long long res, long long s,
2976 |                                            long long msg, long long flags);
2977 | void __sanitizer_syscall_pre_impl_sendmsg(long long s, long long msg,
2978 |                                           long long flags);
2979 | void __sanitizer_syscall_post_impl_sendmsg(long long res, long long s,
2980 |                                            long long msg, long long flags);
```
- **Line 2971 / 第 2971 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2972 / 第 2972 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2973 / 第 2973 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2974 / 第 2974 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2975 / 第 2975 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2976 / 第 2976 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2977 / 第 2977 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2978 / 第 2978 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2979 / 第 2979 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2980 / 第 2980 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2981-2990 / 第 2981-2990 行
```cpp
2981 | void __sanitizer_syscall_pre_impl_recvfrom(long long s, long long buf,
2982 |                                            long long len, long long flags,
2983 |                                            long long from,
2984 |                                            long long fromlenaddr);
2985 | void __sanitizer_syscall_post_impl_recvfrom(long long res, long long s,
2986 |                                             long long buf, long long len,
2987 |                                             long long flags, long long from,
2988 |                                             long long fromlenaddr);
2989 | void __sanitizer_syscall_pre_impl_accept(long long s, long long name,
2990 |                                          long long anamelen);
```
- **Line 2981 / 第 2981 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2982 / 第 2982 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2983 / 第 2983 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2984 / 第 2984 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2985 / 第 2985 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2986 / 第 2986 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2987 / 第 2987 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2988 / 第 2988 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2989 / 第 2989 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2990 / 第 2990 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 2991-3000 / 第 2991-3000 行
```cpp
2991 | void __sanitizer_syscall_post_impl_accept(long long res, long long s,
2992 |                                           long long name, long long anamelen);
2993 | void __sanitizer_syscall_pre_impl_getpeername(long long fdes, long long asa,
2994 |                                               long long alen);
2995 | void __sanitizer_syscall_post_impl_getpeername(long long res, long long fdes,
2996 |                                                long long asa, long long alen);
2997 | void __sanitizer_syscall_pre_impl_getsockname(long long fdes, long long asa,
2998 |                                               long long alen);
2999 | void __sanitizer_syscall_post_impl_getsockname(long long res, long long fdes,
3000 |                                                long long asa, long long alen);
```
- **Line 2991 / 第 2991 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2992 / 第 2992 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2993 / 第 2993 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2994 / 第 2994 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2995 / 第 2995 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2996 / 第 2996 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2997 / 第 2997 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 2998 / 第 2998 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 2999 / 第 2999 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3000 / 第 3000 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3001-3010 / 第 3001-3010 行
```cpp
3001 | void __sanitizer_syscall_pre_impl_access(long long path, long long flags);
3002 | void __sanitizer_syscall_post_impl_access(long long res, long long path,
3003 |                                           long long flags);
3004 | void __sanitizer_syscall_pre_impl_chflags(long long path, long long flags);
3005 | void __sanitizer_syscall_post_impl_chflags(long long res, long long path,
3006 |                                            long long flags);
3007 | void __sanitizer_syscall_pre_impl_fchflags(long long fd, long long flags);
3008 | void __sanitizer_syscall_post_impl_fchflags(long long res, long long fd,
3009 |                                             long long flags);
3010 | void __sanitizer_syscall_pre_impl_sync(void);
```
- **Line 3001 / 第 3001 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_access`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_access`。
- **Line 3002 / 第 3002 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3003 / 第 3003 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3004 / 第 3004 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chflags`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chflags`。
- **Line 3005 / 第 3005 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3006 / 第 3006 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3007 / 第 3007 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchflags`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchflags`。
- **Line 3008 / 第 3008 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3009 / 第 3009 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3010 / 第 3010 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sync`。

### Lines 3011-3020 / 第 3011-3020 行
```cpp
3011 | void __sanitizer_syscall_post_impl_sync(long long res);
3012 | void __sanitizer_syscall_pre_impl_kill(long long pid, long long signum);
3013 | void __sanitizer_syscall_post_impl_kill(long long res, long long pid,
3014 |                                         long long signum);
3015 | void __sanitizer_syscall_pre_impl_compat_43_stat43(long long path,
3016 |                                                    long long ub);
3017 | void __sanitizer_syscall_post_impl_compat_43_stat43(long long res,
3018 |                                                     long long path,
3019 |                                                     long long ub);
3020 | void __sanitizer_syscall_pre_impl_getppid(void);
```
- **Line 3011 / 第 3011 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sync`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sync`。
- **Line 3012 / 第 3012 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_kill`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_kill`。
- **Line 3013 / 第 3013 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3014 / 第 3014 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3015 / 第 3015 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3016 / 第 3016 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3017 / 第 3017 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3018 / 第 3018 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3019 / 第 3019 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3020 / 第 3020 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getppid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getppid`。

### Lines 3021-3030 / 第 3021-3030 行
```cpp
3021 | void __sanitizer_syscall_post_impl_getppid(long long res);
3022 | void __sanitizer_syscall_pre_impl_compat_43_lstat43(long long path,
3023 |                                                     long long ub);
3024 | void __sanitizer_syscall_post_impl_compat_43_lstat43(long long res,
3025 |                                                      long long path,
3026 |                                                      long long ub);
3027 | void __sanitizer_syscall_pre_impl_dup(long long fd);
3028 | void __sanitizer_syscall_post_impl_dup(long long res, long long fd);
3029 | void __sanitizer_syscall_pre_impl_pipe(void);
3030 | void __sanitizer_syscall_post_impl_pipe(long long res);
```
- **Line 3021 / 第 3021 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getppid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getppid`。
- **Line 3022 / 第 3022 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3023 / 第 3023 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3024 / 第 3024 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3025 / 第 3025 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3026 / 第 3026 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3027 / 第 3027 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_dup`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_dup`。
- **Line 3028 / 第 3028 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_dup`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_dup`。
- **Line 3029 / 第 3029 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pipe`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pipe`。
- **Line 3030 / 第 3030 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_pipe`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_pipe`。

### Lines 3031-3040 / 第 3031-3040 行
```cpp
3031 | void __sanitizer_syscall_pre_impl_getegid(void);
3032 | void __sanitizer_syscall_post_impl_getegid(long long res);
3033 | void __sanitizer_syscall_pre_impl_profil(long long samples, long long size,
3034 |                                          long long offset, long long scale);
3035 | void __sanitizer_syscall_post_impl_profil(long long res, long long samples,
3036 |                                           long long size, long long offset,
3037 |                                           long long scale);
3038 | void __sanitizer_syscall_pre_impl_ktrace(long long fname, long long ops,
3039 |                                          long long facs, long long pid);
3040 | void __sanitizer_syscall_post_impl_ktrace(long long res, long long fname,
```
- **Line 3031 / 第 3031 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getegid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getegid`。
- **Line 3032 / 第 3032 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getegid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getegid`。
- **Line 3033 / 第 3033 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3034 / 第 3034 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3035 / 第 3035 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3036 / 第 3036 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3037 / 第 3037 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3038 / 第 3038 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3039 / 第 3039 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3040 / 第 3040 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3041-3050 / 第 3041-3050 行
```cpp
3041 |                                           long long ops, long long facs,
3042 |                                           long long pid);
3043 | void __sanitizer_syscall_pre_impl_compat_13_sigaction13(long long signum,
3044 |                                                         long long nsa,
3045 |                                                         long long osa);
3046 | void __sanitizer_syscall_post_impl_compat_13_sigaction13(long long res,
3047 |                                                          long long signum,
3048 |                                                          long long nsa,
3049 |                                                          long long osa);
3050 | void __sanitizer_syscall_pre_impl_getgid(void);
```
- **Line 3041 / 第 3041 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3042 / 第 3042 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3043 / 第 3043 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3044 / 第 3044 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3045 / 第 3045 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3046 / 第 3046 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3047 / 第 3047 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3048 / 第 3048 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3049 / 第 3049 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3050 / 第 3050 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getgid`。

### Lines 3051-3060 / 第 3051-3060 行
```cpp
3051 | void __sanitizer_syscall_post_impl_getgid(long long res);
3052 | void __sanitizer_syscall_pre_impl_compat_13_sigprocmask13(long long how,
3053 |                                                           long long mask);
3054 | void __sanitizer_syscall_post_impl_compat_13_sigprocmask13(long long res,
3055 |                                                            long long how,
3056 |                                                            long long mask);
3057 | void __sanitizer_syscall_pre_impl___getlogin(long long namebuf,
3058 |                                              long long namelen);
3059 | void __sanitizer_syscall_post_impl___getlogin(long long res, long long namebuf,
3060 |                                               long long namelen);
```
- **Line 3051 / 第 3051 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getgid`。
- **Line 3052 / 第 3052 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3053 / 第 3053 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3054 / 第 3054 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3055 / 第 3055 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3056 / 第 3056 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3057 / 第 3057 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3058 / 第 3058 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3059 / 第 3059 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3060 / 第 3060 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3061-3070 / 第 3061-3070 行
```cpp
3061 | void __sanitizer_syscall_pre_impl___setlogin(long long namebuf);
3062 | void __sanitizer_syscall_post_impl___setlogin(long long res, long long namebuf);
3063 | void __sanitizer_syscall_pre_impl_acct(long long path);
3064 | void __sanitizer_syscall_post_impl_acct(long long res, long long path);
3065 | void __sanitizer_syscall_pre_impl_compat_13_sigpending13(void);
3066 | void __sanitizer_syscall_post_impl_compat_13_sigpending13(long long res);
3067 | void __sanitizer_syscall_pre_impl_compat_13_sigaltstack13(long long nss,
3068 |                                                           long long oss);
3069 | void __sanitizer_syscall_post_impl_compat_13_sigaltstack13(long long res,
3070 |                                                            long long nss,
```
- **Line 3061 / 第 3061 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___setlogin`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___setlogin`。
- **Line 3062 / 第 3062 行**: EN: Declares function or method `__sanitizer_syscall_post_impl___setlogin`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl___setlogin`。
- **Line 3063 / 第 3063 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_acct`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_acct`。
- **Line 3064 / 第 3064 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_acct`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_acct`。
- **Line 3065 / 第 3065 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_13_sigpending13`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_13_sigpending13`。
- **Line 3066 / 第 3066 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_13_sigpending13`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_13_sigpending13`。
- **Line 3067 / 第 3067 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3068 / 第 3068 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3069 / 第 3069 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3070 / 第 3070 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3071-3080 / 第 3071-3080 行
```cpp
3071 |                                                            long long oss);
3072 | void __sanitizer_syscall_pre_impl_ioctl(long long fd, long long com,
3073 |                                         long long data);
3074 | void __sanitizer_syscall_post_impl_ioctl(long long res, long long fd,
3075 |                                          long long com, long long data);
3076 | void __sanitizer_syscall_pre_impl_compat_12_oreboot(long long opt);
3077 | void __sanitizer_syscall_post_impl_compat_12_oreboot(long long res,
3078 |                                                      long long opt);
3079 | void __sanitizer_syscall_pre_impl_revoke(long long path);
3080 | void __sanitizer_syscall_post_impl_revoke(long long res, long long path);
```
- **Line 3071 / 第 3071 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3072 / 第 3072 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3073 / 第 3073 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3074 / 第 3074 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3075 / 第 3075 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3076 / 第 3076 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_12_oreboot`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_12_oreboot`。
- **Line 3077 / 第 3077 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3078 / 第 3078 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3079 / 第 3079 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_revoke`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_revoke`。
- **Line 3080 / 第 3080 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_revoke`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_revoke`。

### Lines 3081-3090 / 第 3081-3090 行
```cpp
3081 | void __sanitizer_syscall_pre_impl_symlink(long long path, long long link);
3082 | void __sanitizer_syscall_post_impl_symlink(long long res, long long path,
3083 |                                            long long link);
3084 | void __sanitizer_syscall_pre_impl_readlink(long long path, long long buf,
3085 |                                            long long count);
3086 | void __sanitizer_syscall_post_impl_readlink(long long res, long long path,
3087 |                                             long long buf, long long count);
3088 | void __sanitizer_syscall_pre_impl_execve(long long path, long long argp,
3089 |                                          long long envp);
3090 | void __sanitizer_syscall_post_impl_execve(long long res, long long path,
```
- **Line 3081 / 第 3081 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_symlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_symlink`。
- **Line 3082 / 第 3082 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3083 / 第 3083 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3084 / 第 3084 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3085 / 第 3085 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3086 / 第 3086 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3087 / 第 3087 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3088 / 第 3088 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3089 / 第 3089 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3090 / 第 3090 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3091-3100 / 第 3091-3100 行
```cpp
3091 |                                           long long argp, long long envp);
3092 | void __sanitizer_syscall_pre_impl_umask(long long newmask);
3093 | void __sanitizer_syscall_post_impl_umask(long long res, long long newmask);
3094 | void __sanitizer_syscall_pre_impl_chroot(long long path);
3095 | void __sanitizer_syscall_post_impl_chroot(long long res, long long path);
3096 | void __sanitizer_syscall_pre_impl_compat_43_fstat43(long long fd, long long sb);
3097 | void __sanitizer_syscall_post_impl_compat_43_fstat43(long long res,
3098 |                                                      long long fd,
3099 |                                                      long long sb);
3100 | void __sanitizer_syscall_pre_impl_compat_43_ogetkerninfo(long long op,
```
- **Line 3091 / 第 3091 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3092 / 第 3092 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_umask`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_umask`。
- **Line 3093 / 第 3093 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_umask`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_umask`。
- **Line 3094 / 第 3094 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_chroot`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_chroot`。
- **Line 3095 / 第 3095 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_chroot`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_chroot`。
- **Line 3096 / 第 3096 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_fstat43`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_fstat43`。
- **Line 3097 / 第 3097 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3098 / 第 3098 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3099 / 第 3099 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3100 / 第 3100 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3101-3110 / 第 3101-3110 行
```cpp
3101 |                                                          long long where,
3102 |                                                          long long size,
3103 |                                                          long long arg);
3104 | void __sanitizer_syscall_post_impl_compat_43_ogetkerninfo(long long res,
3105 |                                                           long long op,
3106 |                                                           long long where,
3107 |                                                           long long size,
3108 |                                                           long long arg);
3109 | void __sanitizer_syscall_pre_impl_compat_43_ogetpagesize(void);
3110 | void __sanitizer_syscall_post_impl_compat_43_ogetpagesize(long long res);
```
- **Line 3101 / 第 3101 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3102 / 第 3102 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3103 / 第 3103 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3104 / 第 3104 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3105 / 第 3105 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3106 / 第 3106 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3107 / 第 3107 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3108 / 第 3108 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3109 / 第 3109 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_ogetpagesize`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_ogetpagesize`。
- **Line 3110 / 第 3110 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_43_ogetpagesize`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_43_ogetpagesize`。

### Lines 3111-3120 / 第 3111-3120 行
```cpp
3111 | void __sanitizer_syscall_pre_impl_compat_12_msync(long long addr,
3112 |                                                   long long len);
3113 | void __sanitizer_syscall_post_impl_compat_12_msync(long long res,
3114 |                                                    long long addr,
3115 |                                                    long long len);
3116 | void __sanitizer_syscall_pre_impl_vfork(void);
3117 | void __sanitizer_syscall_post_impl_vfork(long long res);
3118 | /* syscall 67 has been skipped */
3119 | /* syscall 68 has been skipped */
3120 | /* syscall 69 has been skipped */
```
- **Line 3111 / 第 3111 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3112 / 第 3112 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3113 / 第 3113 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3114 / 第 3114 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3115 / 第 3115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3116 / 第 3116 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_vfork`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_vfork`。
- **Line 3117 / 第 3117 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_vfork`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_vfork`。
- **Line 3118 / 第 3118 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3119 / 第 3119 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3120 / 第 3120 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 3121-3130 / 第 3121-3130 行
```cpp
3121 | /* syscall 70 has been skipped */
3122 | void __sanitizer_syscall_pre_impl_compat_43_ommap(long long addr, long long len,
3123 |                                                   long long prot,
3124 |                                                   long long flags, long long fd,
3125 |                                                   long long pos);
3126 | void __sanitizer_syscall_post_impl_compat_43_ommap(
3127 |     long long res, long long addr, long long len, long long prot,
3128 |     long long flags, long long fd, long long pos);
3129 | void __sanitizer_syscall_pre_impl_vadvise(long long anom);
3130 | void __sanitizer_syscall_post_impl_vadvise(long long res, long long anom);
```
- **Line 3121 / 第 3121 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3122 / 第 3122 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3123 / 第 3123 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3124 / 第 3124 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3125 / 第 3125 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3126 / 第 3126 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3127 / 第 3127 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3128 / 第 3128 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3129 / 第 3129 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_vadvise`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_vadvise`。
- **Line 3130 / 第 3130 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_vadvise`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_vadvise`。

### Lines 3131-3140 / 第 3131-3140 行
```cpp
3131 | void __sanitizer_syscall_pre_impl_munmap(long long addr, long long len);
3132 | void __sanitizer_syscall_post_impl_munmap(long long res, long long addr,
3133 |                                           long long len);
3134 | void __sanitizer_syscall_pre_impl_mprotect(long long addr, long long len,
3135 |                                            long long prot);
3136 | void __sanitizer_syscall_post_impl_mprotect(long long res, long long addr,
3137 |                                             long long len, long long prot);
3138 | void __sanitizer_syscall_pre_impl_madvise(long long addr, long long len,
3139 |                                           long long behav);
3140 | void __sanitizer_syscall_post_impl_madvise(long long res, long long addr,
```
- **Line 3131 / 第 3131 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_munmap`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_munmap`。
- **Line 3132 / 第 3132 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3133 / 第 3133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3134 / 第 3134 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3135 / 第 3135 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3136 / 第 3136 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3137 / 第 3137 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3138 / 第 3138 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3139 / 第 3139 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3140 / 第 3140 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3141-3150 / 第 3141-3150 行
```cpp
3141 |                                            long long len, long long behav);
3142 | /* syscall 76 has been skipped */
3143 | /* syscall 77 has been skipped */
3144 | void __sanitizer_syscall_pre_impl_mincore(long long addr, long long len,
3145 |                                           long long vec);
3146 | void __sanitizer_syscall_post_impl_mincore(long long res, long long addr,
3147 |                                            long long len, long long vec);
3148 | void __sanitizer_syscall_pre_impl_getgroups(long long gidsetsize,
3149 |                                             long long gidset);
3150 | void __sanitizer_syscall_post_impl_getgroups(long long res,
```
- **Line 3141 / 第 3141 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3142 / 第 3142 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3143 / 第 3143 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3144 / 第 3144 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3145 / 第 3145 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3146 / 第 3146 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3147 / 第 3147 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3148 / 第 3148 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3149 / 第 3149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3150 / 第 3150 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3151-3160 / 第 3151-3160 行
```cpp
3151 |                                              long long gidsetsize,
3152 |                                              long long gidset);
3153 | void __sanitizer_syscall_pre_impl_setgroups(long long gidsetsize,
3154 |                                             long long gidset);
3155 | void __sanitizer_syscall_post_impl_setgroups(long long res,
3156 |                                              long long gidsetsize,
3157 |                                              long long gidset);
3158 | void __sanitizer_syscall_pre_impl_getpgrp(void);
3159 | void __sanitizer_syscall_post_impl_getpgrp(long long res);
3160 | void __sanitizer_syscall_pre_impl_setpgid(long long pid, long long pgid);
```
- **Line 3151 / 第 3151 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3152 / 第 3152 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3153 / 第 3153 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3154 / 第 3154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3155 / 第 3155 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3156 / 第 3156 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3157 / 第 3157 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3158 / 第 3158 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpgrp`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpgrp`。
- **Line 3159 / 第 3159 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getpgrp`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getpgrp`。
- **Line 3160 / 第 3160 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setpgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setpgid`。

### Lines 3161-3170 / 第 3161-3170 行
```cpp
3161 | void __sanitizer_syscall_post_impl_setpgid(long long res, long long pid,
3162 |                                            long long pgid);
3163 | void __sanitizer_syscall_pre_impl_compat_50_setitimer(long long which,
3164 |                                                       long long itv,
3165 |                                                       long long oitv);
3166 | void __sanitizer_syscall_post_impl_compat_50_setitimer(long long res,
3167 |                                                        long long which,
3168 |                                                        long long itv,
3169 |                                                        long long oitv);
3170 | void __sanitizer_syscall_pre_impl_compat_43_owait(void);
```
- **Line 3161 / 第 3161 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3162 / 第 3162 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3163 / 第 3163 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3164 / 第 3164 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3165 / 第 3165 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3166 / 第 3166 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3167 / 第 3167 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3168 / 第 3168 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3169 / 第 3169 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3170 / 第 3170 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_owait`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_owait`。

### Lines 3171-3180 / 第 3171-3180 行
```cpp
3171 | void __sanitizer_syscall_post_impl_compat_43_owait(long long res);
3172 | void __sanitizer_syscall_pre_impl_compat_12_oswapon(long long name);
3173 | void __sanitizer_syscall_post_impl_compat_12_oswapon(long long res,
3174 |                                                      long long name);
3175 | void __sanitizer_syscall_pre_impl_compat_50_getitimer(long long which,
3176 |                                                       long long itv);
3177 | void __sanitizer_syscall_post_impl_compat_50_getitimer(long long res,
3178 |                                                        long long which,
3179 |                                                        long long itv);
3180 | void __sanitizer_syscall_pre_impl_compat_43_ogethostname(long long hostname,
```
- **Line 3171 / 第 3171 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_43_owait`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_43_owait`。
- **Line 3172 / 第 3172 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_12_oswapon`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_12_oswapon`。
- **Line 3173 / 第 3173 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3174 / 第 3174 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3175 / 第 3175 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3176 / 第 3176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3177 / 第 3177 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3178 / 第 3178 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3179 / 第 3179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3180 / 第 3180 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3181-3190 / 第 3181-3190 行
```cpp
3181 |                                                          long long len);
3182 | void __sanitizer_syscall_post_impl_compat_43_ogethostname(long long res,
3183 |                                                           long long hostname,
3184 |                                                           long long len);
3185 | void __sanitizer_syscall_pre_impl_compat_43_osethostname(long long hostname,
3186 |                                                          long long len);
3187 | void __sanitizer_syscall_post_impl_compat_43_osethostname(long long res,
3188 |                                                           long long hostname,
3189 |                                                           long long len);
3190 | void __sanitizer_syscall_pre_impl_compat_43_ogetdtablesize(void);
```
- **Line 3181 / 第 3181 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3182 / 第 3182 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3183 / 第 3183 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3184 / 第 3184 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3185 / 第 3185 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3186 / 第 3186 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3187 / 第 3187 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3188 / 第 3188 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3189 / 第 3189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3190 / 第 3190 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_ogetdtablesize`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_ogetdtablesize`。

### Lines 3191-3200 / 第 3191-3200 行
```cpp
3191 | void __sanitizer_syscall_post_impl_compat_43_ogetdtablesize(long long res);
3192 | void __sanitizer_syscall_pre_impl_dup2(long long from, long long to);
3193 | void __sanitizer_syscall_post_impl_dup2(long long res, long long from,
3194 |                                         long long to);
3195 | void __sanitizer_syscall_pre_impl_getrandom(long long buf, long long buflen,
3196 |                                             long long flags);
3197 | void __sanitizer_syscall_post_impl_getrandom(long long res, long long buf,
3198 |                                              long long buflen, long long flags);
3199 | void __sanitizer_syscall_pre_impl_fcntl(long long fd, long long cmd,
3200 |                                         long long arg);
```
- **Line 3191 / 第 3191 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_43_ogetdtablesize`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_43_ogetdtablesize`。
- **Line 3192 / 第 3192 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_dup2`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_dup2`。
- **Line 3193 / 第 3193 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3194 / 第 3194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3195 / 第 3195 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3196 / 第 3196 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3197 / 第 3197 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3198 / 第 3198 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3199 / 第 3199 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3200 / 第 3200 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3201-3210 / 第 3201-3210 行
```cpp
3201 | void __sanitizer_syscall_post_impl_fcntl(long long res, long long fd,
3202 |                                          long long cmd, long long arg);
3203 | void __sanitizer_syscall_pre_impl_compat_50_select(long long nd, long long in,
3204 |                                                    long long ou, long long ex,
3205 |                                                    long long tv);
3206 | void __sanitizer_syscall_post_impl_compat_50_select(long long res, long long nd,
3207 |                                                     long long in, long long ou,
3208 |                                                     long long ex, long long tv);
3209 | /* syscall 94 has been skipped */
3210 | void __sanitizer_syscall_pre_impl_fsync(long long fd);
```
- **Line 3201 / 第 3201 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3202 / 第 3202 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3203 / 第 3203 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3204 / 第 3204 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3205 / 第 3205 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3206 / 第 3206 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3207 / 第 3207 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3208 / 第 3208 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3209 / 第 3209 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3210 / 第 3210 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fsync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fsync`。

### Lines 3211-3220 / 第 3211-3220 行
```cpp
3211 | void __sanitizer_syscall_post_impl_fsync(long long res, long long fd);
3212 | void __sanitizer_syscall_pre_impl_setpriority(long long which, long long who,
3213 |                                               long long prio);
3214 | void __sanitizer_syscall_post_impl_setpriority(long long res, long long which,
3215 |                                                long long who, long long prio);
3216 | void __sanitizer_syscall_pre_impl_compat_30_socket(long long domain,
3217 |                                                    long long type,
3218 |                                                    long long protocol);
3219 | void __sanitizer_syscall_post_impl_compat_30_socket(long long res,
3220 |                                                     long long domain,
```
- **Line 3211 / 第 3211 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fsync`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fsync`。
- **Line 3212 / 第 3212 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3213 / 第 3213 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3214 / 第 3214 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3215 / 第 3215 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3216 / 第 3216 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3217 / 第 3217 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3218 / 第 3218 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3219 / 第 3219 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3220 / 第 3220 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3221-3230 / 第 3221-3230 行
```cpp
3221 |                                                     long long type,
3222 |                                                     long long protocol);
3223 | void __sanitizer_syscall_pre_impl_connect(long long s, long long name,
3224 |                                           long long namelen);
3225 | void __sanitizer_syscall_post_impl_connect(long long res, long long s,
3226 |                                            long long name, long long namelen);
3227 | void __sanitizer_syscall_pre_impl_compat_43_oaccept(long long s, long long name,
3228 |                                                     long long anamelen);
3229 | void __sanitizer_syscall_post_impl_compat_43_oaccept(long long res, long long s,
3230 |                                                      long long name,
```
- **Line 3221 / 第 3221 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3222 / 第 3222 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3223 / 第 3223 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3224 / 第 3224 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3225 / 第 3225 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3226 / 第 3226 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3227 / 第 3227 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3228 / 第 3228 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3229 / 第 3229 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3230 / 第 3230 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3231-3240 / 第 3231-3240 行
```cpp
3231 |                                                      long long anamelen);
3232 | void __sanitizer_syscall_pre_impl_getpriority(long long which, long long who);
3233 | void __sanitizer_syscall_post_impl_getpriority(long long res, long long which,
3234 |                                                long long who);
3235 | void __sanitizer_syscall_pre_impl_compat_43_osend(long long s, long long buf,
3236 |                                                   long long len,
3237 |                                                   long long flags);
3238 | void __sanitizer_syscall_post_impl_compat_43_osend(long long res, long long s,
3239 |                                                    long long buf, long long len,
3240 |                                                    long long flags);
```
- **Line 3231 / 第 3231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3232 / 第 3232 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpriority`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpriority`。
- **Line 3233 / 第 3233 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3234 / 第 3234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3235 / 第 3235 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3236 / 第 3236 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3237 / 第 3237 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3238 / 第 3238 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3239 / 第 3239 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3240 / 第 3240 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3241-3250 / 第 3241-3250 行
```cpp
3241 | void __sanitizer_syscall_pre_impl_compat_43_orecv(long long s, long long buf,
3242 |                                                   long long len,
3243 |                                                   long long flags);
3244 | void __sanitizer_syscall_post_impl_compat_43_orecv(long long res, long long s,
3245 |                                                    long long buf, long long len,
3246 |                                                    long long flags);
3247 | void __sanitizer_syscall_pre_impl_compat_13_sigreturn13(long long sigcntxp);
3248 | void __sanitizer_syscall_post_impl_compat_13_sigreturn13(long long res,
3249 |                                                          long long sigcntxp);
3250 | void __sanitizer_syscall_pre_impl_bind(long long s, long long name,
```
- **Line 3241 / 第 3241 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3242 / 第 3242 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3243 / 第 3243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3244 / 第 3244 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3245 / 第 3245 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3246 / 第 3246 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3247 / 第 3247 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_13_sigreturn13`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_13_sigreturn13`。
- **Line 3248 / 第 3248 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3249 / 第 3249 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3250 / 第 3250 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3251-3260 / 第 3251-3260 行
```cpp
3251 |                                        long long namelen);
3252 | void __sanitizer_syscall_post_impl_bind(long long res, long long s,
3253 |                                         long long name, long long namelen);
3254 | void __sanitizer_syscall_pre_impl_setsockopt(long long s, long long level,
3255 |                                              long long name, long long val,
3256 |                                              long long valsize);
3257 | void __sanitizer_syscall_post_impl_setsockopt(long long res, long long s,
3258 |                                               long long level, long long name,
3259 |                                               long long val, long long valsize);
3260 | void __sanitizer_syscall_pre_impl_listen(long long s, long long backlog);
```
- **Line 3251 / 第 3251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3252 / 第 3252 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3253 / 第 3253 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3254 / 第 3254 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3255 / 第 3255 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3256 / 第 3256 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3257 / 第 3257 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3258 / 第 3258 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3259 / 第 3259 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3260 / 第 3260 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_listen`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_listen`。

### Lines 3261-3270 / 第 3261-3270 行
```cpp
3261 | void __sanitizer_syscall_post_impl_listen(long long res, long long s,
3262 |                                           long long backlog);
3263 | /* syscall 107 has been skipped */
3264 | void __sanitizer_syscall_pre_impl_compat_43_osigvec(long long signum,
3265 |                                                     long long nsv,
3266 |                                                     long long osv);
3267 | void __sanitizer_syscall_post_impl_compat_43_osigvec(long long res,
3268 |                                                      long long signum,
3269 |                                                      long long nsv,
3270 |                                                      long long osv);
```
- **Line 3261 / 第 3261 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3262 / 第 3262 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3263 / 第 3263 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3264 / 第 3264 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3265 / 第 3265 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3266 / 第 3266 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3267 / 第 3267 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3268 / 第 3268 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3269 / 第 3269 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3270 / 第 3270 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3271-3280 / 第 3271-3280 行
```cpp
3271 | void __sanitizer_syscall_pre_impl_compat_43_osigblock(long long mask);
3272 | void __sanitizer_syscall_post_impl_compat_43_osigblock(long long res,
3273 |                                                        long long mask);
3274 | void __sanitizer_syscall_pre_impl_compat_43_osigsetmask(long long mask);
3275 | void __sanitizer_syscall_post_impl_compat_43_osigsetmask(long long res,
3276 |                                                          long long mask);
3277 | void __sanitizer_syscall_pre_impl_compat_13_sigsuspend13(long long mask);
3278 | void __sanitizer_syscall_post_impl_compat_13_sigsuspend13(long long res,
3279 |                                                           long long mask);
3280 | void __sanitizer_syscall_pre_impl_compat_43_osigstack(long long nss,
```
- **Line 3271 / 第 3271 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_osigblock`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_osigblock`。
- **Line 3272 / 第 3272 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3273 / 第 3273 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3274 / 第 3274 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_osigsetmask`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_osigsetmask`。
- **Line 3275 / 第 3275 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3276 / 第 3276 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3277 / 第 3277 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_13_sigsuspend13`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_13_sigsuspend13`。
- **Line 3278 / 第 3278 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3279 / 第 3279 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3280 / 第 3280 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3281-3290 / 第 3281-3290 行
```cpp
3281 |                                                       long long oss);
3282 | void __sanitizer_syscall_post_impl_compat_43_osigstack(long long res,
3283 |                                                        long long nss,
3284 |                                                        long long oss);
3285 | void __sanitizer_syscall_pre_impl_compat_43_orecvmsg(long long s, long long msg,
3286 |                                                      long long flags);
3287 | void __sanitizer_syscall_post_impl_compat_43_orecvmsg(long long res,
3288 |                                                       long long s,
3289 |                                                       long long msg,
3290 |                                                       long long flags);
```
- **Line 3281 / 第 3281 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3282 / 第 3282 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3283 / 第 3283 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3284 / 第 3284 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3285 / 第 3285 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3286 / 第 3286 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3287 / 第 3287 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3288 / 第 3288 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3289 / 第 3289 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3290 / 第 3290 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3291-3300 / 第 3291-3300 行
```cpp
3291 | void __sanitizer_syscall_pre_impl_compat_43_osendmsg(long long s, long long msg,
3292 |                                                      long long flags);
3293 | void __sanitizer_syscall_post_impl_compat_43_osendmsg(long long res,
3294 |                                                       long long s,
3295 |                                                       long long msg,
3296 |                                                       long long flags);
3297 | /* syscall 115 has been skipped */
3298 | void __sanitizer_syscall_pre_impl_compat_50_gettimeofday(long long tp,
3299 |                                                          long long tzp);
3300 | void __sanitizer_syscall_post_impl_compat_50_gettimeofday(long long res,
```
- **Line 3291 / 第 3291 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3292 / 第 3292 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3293 / 第 3293 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3294 / 第 3294 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3295 / 第 3295 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3296 / 第 3296 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3297 / 第 3297 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3298 / 第 3298 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3299 / 第 3299 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3300 / 第 3300 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3301-3310 / 第 3301-3310 行
```cpp
3301 |                                                           long long tp,
3302 |                                                           long long tzp);
3303 | void __sanitizer_syscall_pre_impl_compat_50_getrusage(long long who,
3304 |                                                       long long rusage);
3305 | void __sanitizer_syscall_post_impl_compat_50_getrusage(long long res,
3306 |                                                        long long who,
3307 |                                                        long long rusage);
3308 | void __sanitizer_syscall_pre_impl_getsockopt(long long s, long long level,
3309 |                                              long long name, long long val,
3310 |                                              long long avalsize);
```
- **Line 3301 / 第 3301 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3302 / 第 3302 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3303 / 第 3303 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3304 / 第 3304 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3305 / 第 3305 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3306 / 第 3306 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3307 / 第 3307 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3308 / 第 3308 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3309 / 第 3309 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3310 / 第 3310 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3311-3320 / 第 3311-3320 行
```cpp
3311 | void __sanitizer_syscall_post_impl_getsockopt(long long res, long long s,
3312 |                                               long long level, long long name,
3313 |                                               long long val,
3314 |                                               long long avalsize);
3315 | /* syscall 119 has been skipped */
3316 | void __sanitizer_syscall_pre_impl_readv(long long fd, long long iovp,
3317 |                                         long long iovcnt);
3318 | void __sanitizer_syscall_post_impl_readv(long long res, long long fd,
3319 |                                          long long iovp, long long iovcnt);
3320 | void __sanitizer_syscall_pre_impl_writev(long long fd, long long iovp,
```
- **Line 3311 / 第 3311 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3312 / 第 3312 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3313 / 第 3313 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3314 / 第 3314 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3315 / 第 3315 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3316 / 第 3316 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3317 / 第 3317 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3318 / 第 3318 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3319 / 第 3319 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3320 / 第 3320 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3321-3330 / 第 3321-3330 行
```cpp
3321 |                                          long long iovcnt);
3322 | void __sanitizer_syscall_post_impl_writev(long long res, long long fd,
3323 |                                           long long iovp, long long iovcnt);
3324 | void __sanitizer_syscall_pre_impl_compat_50_settimeofday(long long tv,
3325 |                                                          long long tzp);
3326 | void __sanitizer_syscall_post_impl_compat_50_settimeofday(long long res,
3327 |                                                           long long tv,
3328 |                                                           long long tzp);
3329 | void __sanitizer_syscall_pre_impl_fchown(long long fd, long long uid,
3330 |                                          long long gid);
```
- **Line 3321 / 第 3321 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3322 / 第 3322 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3323 / 第 3323 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3324 / 第 3324 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3325 / 第 3325 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3326 / 第 3326 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3327 / 第 3327 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3328 / 第 3328 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3329 / 第 3329 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3330 / 第 3330 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3331-3340 / 第 3331-3340 行
```cpp
3331 | void __sanitizer_syscall_post_impl_fchown(long long res, long long fd,
3332 |                                           long long uid, long long gid);
3333 | void __sanitizer_syscall_pre_impl_fchmod(long long fd, long long mode);
3334 | void __sanitizer_syscall_post_impl_fchmod(long long res, long long fd,
3335 |                                           long long mode);
3336 | void __sanitizer_syscall_pre_impl_compat_43_orecvfrom(
3337 |     long long s, long long buf, long long len, long long flags, long long from,
3338 |     long long fromlenaddr);
3339 | void __sanitizer_syscall_post_impl_compat_43_orecvfrom(
3340 |     long long res, long long s, long long buf, long long len, long long flags,
```
- **Line 3331 / 第 3331 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3332 / 第 3332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3333 / 第 3333 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchmod`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchmod`。
- **Line 3334 / 第 3334 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3335 / 第 3335 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3336 / 第 3336 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3337 / 第 3337 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3338 / 第 3338 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3339 / 第 3339 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3340 / 第 3340 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3341-3350 / 第 3341-3350 行
```cpp
3341 |     long long from, long long fromlenaddr);
3342 | void __sanitizer_syscall_pre_impl_setreuid(long long ruid, long long euid);
3343 | void __sanitizer_syscall_post_impl_setreuid(long long res, long long ruid,
3344 |                                             long long euid);
3345 | void __sanitizer_syscall_pre_impl_setregid(long long rgid, long long egid);
3346 | void __sanitizer_syscall_post_impl_setregid(long long res, long long rgid,
3347 |                                             long long egid);
3348 | void __sanitizer_syscall_pre_impl_rename(long long from, long long to);
3349 | void __sanitizer_syscall_post_impl_rename(long long res, long long from,
3350 |                                           long long to);
```
- **Line 3341 / 第 3341 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3342 / 第 3342 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setreuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setreuid`。
- **Line 3343 / 第 3343 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3344 / 第 3344 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3345 / 第 3345 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setregid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setregid`。
- **Line 3346 / 第 3346 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3347 / 第 3347 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3348 / 第 3348 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_rename`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_rename`。
- **Line 3349 / 第 3349 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3350 / 第 3350 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3351-3360 / 第 3351-3360 行
```cpp
3351 | void __sanitizer_syscall_pre_impl_compat_43_otruncate(long long path,
3352 |                                                       long long length);
3353 | void __sanitizer_syscall_post_impl_compat_43_otruncate(long long res,
3354 |                                                        long long path,
3355 |                                                        long long length);
3356 | void __sanitizer_syscall_pre_impl_compat_43_oftruncate(long long fd,
3357 |                                                        long long length);
3358 | void __sanitizer_syscall_post_impl_compat_43_oftruncate(long long res,
3359 |                                                         long long fd,
3360 |                                                         long long length);
```
- **Line 3351 / 第 3351 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3352 / 第 3352 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3353 / 第 3353 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3354 / 第 3354 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3355 / 第 3355 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3356 / 第 3356 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3357 / 第 3357 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3358 / 第 3358 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3359 / 第 3359 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3360 / 第 3360 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3361-3370 / 第 3361-3370 行
```cpp
3361 | void __sanitizer_syscall_pre_impl_flock(long long fd, long long how);
3362 | void __sanitizer_syscall_post_impl_flock(long long res, long long fd,
3363 |                                          long long how);
3364 | void __sanitizer_syscall_pre_impl_mkfifo(long long path, long long mode);
3365 | void __sanitizer_syscall_post_impl_mkfifo(long long res, long long path,
3366 |                                           long long mode);
3367 | void __sanitizer_syscall_pre_impl_sendto(long long s, long long buf,
3368 |                                          long long len, long long flags,
3369 |                                          long long to, long long tolen);
3370 | void __sanitizer_syscall_post_impl_sendto(long long res, long long s,
```
- **Line 3361 / 第 3361 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_flock`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_flock`。
- **Line 3362 / 第 3362 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3363 / 第 3363 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3364 / 第 3364 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mkfifo`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mkfifo`。
- **Line 3365 / 第 3365 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3366 / 第 3366 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3367 / 第 3367 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3368 / 第 3368 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3369 / 第 3369 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3370 / 第 3370 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3371-3380 / 第 3371-3380 行
```cpp
3371 |                                           long long buf, long long len,
3372 |                                           long long flags, long long to,
3373 |                                           long long tolen);
3374 | void __sanitizer_syscall_pre_impl_shutdown(long long s, long long how);
3375 | void __sanitizer_syscall_post_impl_shutdown(long long res, long long s,
3376 |                                             long long how);
3377 | void __sanitizer_syscall_pre_impl_socketpair(long long domain, long long type,
3378 |                                              long long protocol, long long rsv);
3379 | void __sanitizer_syscall_post_impl_socketpair(long long res, long long domain,
3380 |                                               long long type,
```
- **Line 3371 / 第 3371 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3372 / 第 3372 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3373 / 第 3373 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3374 / 第 3374 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_shutdown`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_shutdown`。
- **Line 3375 / 第 3375 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3376 / 第 3376 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3377 / 第 3377 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3378 / 第 3378 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3379 / 第 3379 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3380 / 第 3380 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3381-3390 / 第 3381-3390 行
```cpp
3381 |                                               long long protocol,
3382 |                                               long long rsv);
3383 | void __sanitizer_syscall_pre_impl_mkdir(long long path, long long mode);
3384 | void __sanitizer_syscall_post_impl_mkdir(long long res, long long path,
3385 |                                          long long mode);
3386 | void __sanitizer_syscall_pre_impl_rmdir(long long path);
3387 | void __sanitizer_syscall_post_impl_rmdir(long long res, long long path);
3388 | void __sanitizer_syscall_pre_impl_compat_50_utimes(long long path,
3389 |                                                    long long tptr);
3390 | void __sanitizer_syscall_post_impl_compat_50_utimes(long long res,
```
- **Line 3381 / 第 3381 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3382 / 第 3382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3383 / 第 3383 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mkdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mkdir`。
- **Line 3384 / 第 3384 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3385 / 第 3385 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3386 / 第 3386 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_rmdir`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_rmdir`。
- **Line 3387 / 第 3387 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_rmdir`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_rmdir`。
- **Line 3388 / 第 3388 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3389 / 第 3389 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3390 / 第 3390 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3391-3400 / 第 3391-3400 行
```cpp
3391 |                                                     long long path,
3392 |                                                     long long tptr);
3393 | /* syscall 139 has been skipped */
3394 | void __sanitizer_syscall_pre_impl_compat_50_adjtime(long long delta,
3395 |                                                     long long olddelta);
3396 | void __sanitizer_syscall_post_impl_compat_50_adjtime(long long res,
3397 |                                                      long long delta,
3398 |                                                      long long olddelta);
3399 | void __sanitizer_syscall_pre_impl_compat_43_ogetpeername(long long fdes,
3400 |                                                          long long asa,
```
- **Line 3391 / 第 3391 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3392 / 第 3392 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3393 / 第 3393 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3394 / 第 3394 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3395 / 第 3395 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3396 / 第 3396 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3397 / 第 3397 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3398 / 第 3398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3399 / 第 3399 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3400 / 第 3400 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3401-3410 / 第 3401-3410 行
```cpp
3401 |                                                          long long alen);
3402 | void __sanitizer_syscall_post_impl_compat_43_ogetpeername(long long res,
3403 |                                                           long long fdes,
3404 |                                                           long long asa,
3405 |                                                           long long alen);
3406 | void __sanitizer_syscall_pre_impl_compat_43_ogethostid(void);
3407 | void __sanitizer_syscall_post_impl_compat_43_ogethostid(long long res);
3408 | void __sanitizer_syscall_pre_impl_compat_43_osethostid(long long hostid);
3409 | void __sanitizer_syscall_post_impl_compat_43_osethostid(long long res,
3410 |                                                         long long hostid);
```
- **Line 3401 / 第 3401 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3402 / 第 3402 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3403 / 第 3403 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3404 / 第 3404 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3405 / 第 3405 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3406 / 第 3406 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_ogethostid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_ogethostid`。
- **Line 3407 / 第 3407 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_43_ogethostid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_43_ogethostid`。
- **Line 3408 / 第 3408 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_osethostid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_osethostid`。
- **Line 3409 / 第 3409 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3410 / 第 3410 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3411-3420 / 第 3411-3420 行
```cpp
3411 | void __sanitizer_syscall_pre_impl_compat_43_ogetrlimit(long long which,
3412 |                                                        long long rlp);
3413 | void __sanitizer_syscall_post_impl_compat_43_ogetrlimit(long long res,
3414 |                                                         long long which,
3415 |                                                         long long rlp);
3416 | void __sanitizer_syscall_pre_impl_compat_43_osetrlimit(long long which,
3417 |                                                        long long rlp);
3418 | void __sanitizer_syscall_post_impl_compat_43_osetrlimit(long long res,
3419 |                                                         long long which,
3420 |                                                         long long rlp);
```
- **Line 3411 / 第 3411 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3412 / 第 3412 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3413 / 第 3413 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3414 / 第 3414 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3415 / 第 3415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3416 / 第 3416 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3417 / 第 3417 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3418 / 第 3418 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3419 / 第 3419 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3420 / 第 3420 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3421-3430 / 第 3421-3430 行
```cpp
3421 | void __sanitizer_syscall_pre_impl_compat_43_okillpg(long long pgid,
3422 |                                                     long long signum);
3423 | void __sanitizer_syscall_post_impl_compat_43_okillpg(long long res,
3424 |                                                      long long pgid,
3425 |                                                      long long signum);
3426 | void __sanitizer_syscall_pre_impl_setsid(void);
3427 | void __sanitizer_syscall_post_impl_setsid(long long res);
3428 | void __sanitizer_syscall_pre_impl_compat_50_quotactl(long long path,
3429 |                                                      long long cmd,
3430 |                                                      long long uid,
```
- **Line 3421 / 第 3421 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3422 / 第 3422 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3423 / 第 3423 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3424 / 第 3424 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3425 / 第 3425 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3426 / 第 3426 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setsid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setsid`。
- **Line 3427 / 第 3427 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setsid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setsid`。
- **Line 3428 / 第 3428 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3429 / 第 3429 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3430 / 第 3430 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3431-3440 / 第 3431-3440 行
```cpp
3431 |                                                      long long arg);
3432 | void __sanitizer_syscall_post_impl_compat_50_quotactl(
3433 |     long long res, long long path, long long cmd, long long uid, long long arg);
3434 | void __sanitizer_syscall_pre_impl_compat_43_oquota(void);
3435 | void __sanitizer_syscall_post_impl_compat_43_oquota(long long res);
3436 | void __sanitizer_syscall_pre_impl_compat_43_ogetsockname(long long fdec,
3437 |                                                          long long asa,
3438 |                                                          long long alen);
3439 | void __sanitizer_syscall_post_impl_compat_43_ogetsockname(long long res,
3440 |                                                           long long fdec,
```
- **Line 3431 / 第 3431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3432 / 第 3432 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3433 / 第 3433 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3434 / 第 3434 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_43_oquota`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_43_oquota`。
- **Line 3435 / 第 3435 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_43_oquota`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_43_oquota`。
- **Line 3436 / 第 3436 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3437 / 第 3437 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3438 / 第 3438 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3439 / 第 3439 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3440 / 第 3440 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3441-3450 / 第 3441-3450 行
```cpp
3441 |                                                           long long asa,
3442 |                                                           long long alen);
3443 | /* syscall 151 has been skipped */
3444 | /* syscall 152 has been skipped */
3445 | /* syscall 153 has been skipped */
3446 | /* syscall 154 has been skipped */
3447 | void __sanitizer_syscall_pre_impl_nfssvc(long long flag, long long argp);
3448 | void __sanitizer_syscall_post_impl_nfssvc(long long res, long long flag,
3449 |                                           long long argp);
3450 | void __sanitizer_syscall_pre_impl_compat_43_ogetdirentries(long long fd,
```
- **Line 3441 / 第 3441 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3442 / 第 3442 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3443 / 第 3443 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3444 / 第 3444 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3445 / 第 3445 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3446 / 第 3446 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3447 / 第 3447 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_nfssvc`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_nfssvc`。
- **Line 3448 / 第 3448 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3449 / 第 3449 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3450 / 第 3450 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3451-3460 / 第 3451-3460 行
```cpp
3451 |                                                            long long buf,
3452 |                                                            long long count,
3453 |                                                            long long basep);
3454 | void __sanitizer_syscall_post_impl_compat_43_ogetdirentries(long long res,
3455 |                                                             long long fd,
3456 |                                                             long long buf,
3457 |                                                             long long count,
3458 |                                                             long long basep);
3459 | void __sanitizer_syscall_pre_impl_compat_20_statfs(long long path,
3460 |                                                    long long buf);
```
- **Line 3451 / 第 3451 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3452 / 第 3452 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3453 / 第 3453 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3454 / 第 3454 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3455 / 第 3455 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3456 / 第 3456 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3457 / 第 3457 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3458 / 第 3458 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3459 / 第 3459 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3460 / 第 3460 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3461-3470 / 第 3461-3470 行
```cpp
3461 | void __sanitizer_syscall_post_impl_compat_20_statfs(long long res,
3462 |                                                     long long path,
3463 |                                                     long long buf);
3464 | void __sanitizer_syscall_pre_impl_compat_20_fstatfs(long long fd,
3465 |                                                     long long buf);
3466 | void __sanitizer_syscall_post_impl_compat_20_fstatfs(long long res,
3467 |                                                      long long fd,
3468 |                                                      long long buf);
3469 | /* syscall 159 has been skipped */
3470 | /* syscall 160 has been skipped */
```
- **Line 3461 / 第 3461 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3462 / 第 3462 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3463 / 第 3463 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3464 / 第 3464 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3465 / 第 3465 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3466 / 第 3466 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3467 / 第 3467 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3468 / 第 3468 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3469 / 第 3469 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3470 / 第 3470 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 3471-3480 / 第 3471-3480 行
```cpp
3471 | void __sanitizer_syscall_pre_impl_compat_30_getfh(long long fname,
3472 |                                                   long long fhp);
3473 | void __sanitizer_syscall_post_impl_compat_30_getfh(long long res,
3474 |                                                    long long fname,
3475 |                                                    long long fhp);
3476 | void __sanitizer_syscall_pre_impl_compat_09_ogetdomainname(long long domainname,
3477 |                                                            long long len);
3478 | void __sanitizer_syscall_post_impl_compat_09_ogetdomainname(
3479 |     long long res, long long domainname, long long len);
3480 | void __sanitizer_syscall_pre_impl_compat_09_osetdomainname(long long domainname,
```
- **Line 3471 / 第 3471 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3472 / 第 3472 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3473 / 第 3473 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3474 / 第 3474 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3475 / 第 3475 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3476 / 第 3476 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3477 / 第 3477 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3478 / 第 3478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3479 / 第 3479 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3480 / 第 3480 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3481-3490 / 第 3481-3490 行
```cpp
3481 |                                                            long long len);
3482 | void __sanitizer_syscall_post_impl_compat_09_osetdomainname(
3483 |     long long res, long long domainname, long long len);
3484 | void __sanitizer_syscall_pre_impl_compat_09_ouname(long long name);
3485 | void __sanitizer_syscall_post_impl_compat_09_ouname(long long res,
3486 |                                                     long long name);
3487 | void __sanitizer_syscall_pre_impl_sysarch(long long op, long long parms);
3488 | void __sanitizer_syscall_post_impl_sysarch(long long res, long long op,
3489 |                                            long long parms);
3490 | void __sanitizer_syscall_pre_impl___futex(long long uaddr, long long op,
```
- **Line 3481 / 第 3481 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3482 / 第 3482 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3483 / 第 3483 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3484 / 第 3484 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_09_ouname`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_09_ouname`。
- **Line 3485 / 第 3485 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3486 / 第 3486 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3487 / 第 3487 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sysarch`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sysarch`。
- **Line 3488 / 第 3488 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3489 / 第 3489 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3490 / 第 3490 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3491-3500 / 第 3491-3500 行
```cpp
3491 |                                           long long val, long long timeout,
3492 |                                           long long uaddr2, long long val2,
3493 |                                           long long val3);
3494 | void __sanitizer_syscall_post_impl___futex(long long res, long long uaddr,
3495 |                                            long long op, long long val,
3496 |                                            long long timeout, long long uaddr2,
3497 |                                            long long val2, long long val3);
3498 | void __sanitizer_syscall_pre_impl___futex_set_robust_list(long long head,
3499 |                                                           long long len);
3500 | void __sanitizer_syscall_post_impl___futex_set_robust_list(long long res,
```
- **Line 3491 / 第 3491 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3492 / 第 3492 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3493 / 第 3493 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3494 / 第 3494 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3495 / 第 3495 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3496 / 第 3496 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3497 / 第 3497 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3498 / 第 3498 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3499 / 第 3499 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3500 / 第 3500 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3501-3510 / 第 3501-3510 行
```cpp
3501 |                                                            long long head,
3502 |                                                            long long len);
3503 | void __sanitizer_syscall_pre_impl___futex_get_robust_list(long long lwpid,
3504 |                                                           long long headp,
3505 |                                                           long long lenp);
3506 | void __sanitizer_syscall_post_impl___futex_get_robust_list(long long res,
3507 |                                                            long long lwpid,
3508 |                                                            long long headp,
3509 |                                                            long long lenp);
3510 | #if !defined(_LP64)
```
- **Line 3501 / 第 3501 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3502 / 第 3502 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3503 / 第 3503 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3504 / 第 3504 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3505 / 第 3505 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3506 / 第 3506 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3507 / 第 3507 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3508 / 第 3508 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3509 / 第 3509 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3510 / 第 3510 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。

### Lines 3511-3520 / 第 3511-3520 行
```cpp
3511 | void __sanitizer_syscall_pre_impl_compat_10_osemsys(long long which,
3512 |                                                     long long a2, long long a3,
3513 |                                                     long long a4, long long a5);
3514 | void __sanitizer_syscall_post_impl_compat_10_osemsys(long long res,
3515 |                                                      long long which,
3516 |                                                      long long a2, long long a3,
3517 |                                                      long long a4,
3518 |                                                      long long a5);
3519 | #else
3520 | /* syscall 169 has been skipped */
```
- **Line 3511 / 第 3511 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3512 / 第 3512 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3513 / 第 3513 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3514 / 第 3514 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3515 / 第 3515 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3516 / 第 3516 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3517 / 第 3517 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3518 / 第 3518 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3519 / 第 3519 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 3520 / 第 3520 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 3521-3530 / 第 3521-3530 行
```cpp
3521 | #endif
3522 | #if !defined(_LP64)
3523 | void __sanitizer_syscall_pre_impl_compat_10_omsgsys(long long which,
3524 |                                                     long long a2, long long a3,
3525 |                                                     long long a4, long long a5,
3526 |                                                     long long a6);
3527 | void __sanitizer_syscall_post_impl_compat_10_omsgsys(long long res,
3528 |                                                      long long which,
3529 |                                                      long long a2, long long a3,
3530 |                                                      long long a4, long long a5,
```
- **Line 3521 / 第 3521 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 3522 / 第 3522 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 3523 / 第 3523 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3524 / 第 3524 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3525 / 第 3525 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3526 / 第 3526 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3527 / 第 3527 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3528 / 第 3528 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3529 / 第 3529 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3530 / 第 3530 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3531-3540 / 第 3531-3540 行
```cpp
3531 |                                                      long long a6);
3532 | #else
3533 | /* syscall 170 has been skipped */
3534 | #endif
3535 | #if !defined(_LP64)
3536 | void __sanitizer_syscall_pre_impl_compat_10_oshmsys(long long which,
3537 |                                                     long long a2, long long a3,
3538 |                                                     long long a4);
3539 | void __sanitizer_syscall_post_impl_compat_10_oshmsys(long long res,
3540 |                                                      long long which,
```
- **Line 3531 / 第 3531 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3532 / 第 3532 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 3533 / 第 3533 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3534 / 第 3534 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 3535 / 第 3535 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 3536 / 第 3536 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3537 / 第 3537 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3538 / 第 3538 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3539 / 第 3539 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3540 / 第 3540 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3541-3550 / 第 3541-3550 行
```cpp
3541 |                                                      long long a2, long long a3,
3542 |                                                      long long a4);
3543 | #else
3544 | /* syscall 171 has been skipped */
3545 | #endif
3546 | /* syscall 172 has been skipped */
3547 | void __sanitizer_syscall_pre_impl_pread(long long fd, long long buf,
3548 |                                         long long nbyte, long long PAD,
3549 |                                         long long offset);
3550 | void __sanitizer_syscall_post_impl_pread(long long res, long long fd,
```
- **Line 3541 / 第 3541 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3542 / 第 3542 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3543 / 第 3543 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 3544 / 第 3544 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3545 / 第 3545 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 3546 / 第 3546 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3547 / 第 3547 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3548 / 第 3548 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3549 / 第 3549 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3550 / 第 3550 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3551-3560 / 第 3551-3560 行
```cpp
3551 |                                          long long buf, long long nbyte,
3552 |                                          long long PAD, long long offset);
3553 | void __sanitizer_syscall_pre_impl_pwrite(long long fd, long long buf,
3554 |                                          long long nbyte, long long PAD,
3555 |                                          long long offset);
3556 | void __sanitizer_syscall_post_impl_pwrite(long long res, long long fd,
3557 |                                           long long buf, long long nbyte,
3558 |                                           long long PAD, long long offset);
3559 | void __sanitizer_syscall_pre_impl_compat_30_ntp_gettime(long long ntvp);
3560 | void __sanitizer_syscall_post_impl_compat_30_ntp_gettime(long long res,
```
- **Line 3551 / 第 3551 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3552 / 第 3552 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3553 / 第 3553 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3554 / 第 3554 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3555 / 第 3555 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3556 / 第 3556 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3557 / 第 3557 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3558 / 第 3558 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3559 / 第 3559 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_30_ntp_gettime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_30_ntp_gettime`。
- **Line 3560 / 第 3560 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3561-3570 / 第 3561-3570 行
```cpp
3561 |                                                          long long ntvp);
3562 | #if defined(NTP) || !defined(_KERNEL_OPT)
3563 | void __sanitizer_syscall_pre_impl_ntp_adjtime(long long tp);
3564 | void __sanitizer_syscall_post_impl_ntp_adjtime(long long res, long long tp);
3565 | #else
3566 | /* syscall 176 has been skipped */
3567 | #endif
3568 | /* syscall 177 has been skipped */
3569 | /* syscall 178 has been skipped */
3570 | /* syscall 179 has been skipped */
```
- **Line 3561 / 第 3561 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3562 / 第 3562 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 3563 / 第 3563 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_ntp_adjtime`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_ntp_adjtime`。
- **Line 3564 / 第 3564 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_ntp_adjtime`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_ntp_adjtime`。
- **Line 3565 / 第 3565 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 3566 / 第 3566 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3567 / 第 3567 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 3568 / 第 3568 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3569 / 第 3569 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3570 / 第 3570 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 3571-3580 / 第 3571-3580 行
```cpp
3571 | /* syscall 180 has been skipped */
3572 | void __sanitizer_syscall_pre_impl_setgid(long long gid);
3573 | void __sanitizer_syscall_post_impl_setgid(long long res, long long gid);
3574 | void __sanitizer_syscall_pre_impl_setegid(long long egid);
3575 | void __sanitizer_syscall_post_impl_setegid(long long res, long long egid);
3576 | void __sanitizer_syscall_pre_impl_seteuid(long long euid);
3577 | void __sanitizer_syscall_post_impl_seteuid(long long res, long long euid);
3578 | void __sanitizer_syscall_pre_impl_lfs_bmapv(long long fsidp, long long blkiov,
3579 |                                             long long blkcnt);
3580 | void __sanitizer_syscall_post_impl_lfs_bmapv(long long res, long long fsidp,
```
- **Line 3571 / 第 3571 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3572 / 第 3572 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setgid`。
- **Line 3573 / 第 3573 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setgid`。
- **Line 3574 / 第 3574 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setegid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setegid`。
- **Line 3575 / 第 3575 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setegid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setegid`。
- **Line 3576 / 第 3576 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_seteuid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_seteuid`。
- **Line 3577 / 第 3577 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_seteuid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_seteuid`。
- **Line 3578 / 第 3578 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3579 / 第 3579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3580 / 第 3580 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3581-3590 / 第 3581-3590 行
```cpp
3581 |                                              long long blkiov,
3582 |                                              long long blkcnt);
3583 | void __sanitizer_syscall_pre_impl_lfs_markv(long long fsidp, long long blkiov,
3584 |                                             long long blkcnt);
3585 | void __sanitizer_syscall_post_impl_lfs_markv(long long res, long long fsidp,
3586 |                                              long long blkiov,
3587 |                                              long long blkcnt);
3588 | void __sanitizer_syscall_pre_impl_lfs_segclean(long long fsidp,
3589 |                                                long long segment);
3590 | void __sanitizer_syscall_post_impl_lfs_segclean(long long res, long long fsidp,
```
- **Line 3581 / 第 3581 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3582 / 第 3582 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3583 / 第 3583 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3584 / 第 3584 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3585 / 第 3585 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3586 / 第 3586 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3587 / 第 3587 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3588 / 第 3588 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3589 / 第 3589 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3590 / 第 3590 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3591-3600 / 第 3591-3600 行
```cpp
3591 |                                                 long long segment);
3592 | void __sanitizer_syscall_pre_impl_compat_50_lfs_segwait(long long fsidp,
3593 |                                                         long long tv);
3594 | void __sanitizer_syscall_post_impl_compat_50_lfs_segwait(long long res,
3595 |                                                          long long fsidp,
3596 |                                                          long long tv);
3597 | void __sanitizer_syscall_pre_impl_compat_12_stat12(long long path,
3598 |                                                    long long ub);
3599 | void __sanitizer_syscall_post_impl_compat_12_stat12(long long res,
3600 |                                                     long long path,
```
- **Line 3591 / 第 3591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3592 / 第 3592 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3593 / 第 3593 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3594 / 第 3594 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3595 / 第 3595 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3596 / 第 3596 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3597 / 第 3597 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3598 / 第 3598 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3599 / 第 3599 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3600 / 第 3600 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3601-3610 / 第 3601-3610 行
```cpp
3601 |                                                     long long ub);
3602 | void __sanitizer_syscall_pre_impl_compat_12_fstat12(long long fd, long long sb);
3603 | void __sanitizer_syscall_post_impl_compat_12_fstat12(long long res,
3604 |                                                      long long fd,
3605 |                                                      long long sb);
3606 | void __sanitizer_syscall_pre_impl_compat_12_lstat12(long long path,
3607 |                                                     long long ub);
3608 | void __sanitizer_syscall_post_impl_compat_12_lstat12(long long res,
3609 |                                                      long long path,
3610 |                                                      long long ub);
```
- **Line 3601 / 第 3601 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3602 / 第 3602 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_12_fstat12`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_12_fstat12`。
- **Line 3603 / 第 3603 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3604 / 第 3604 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3605 / 第 3605 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3606 / 第 3606 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3607 / 第 3607 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3608 / 第 3608 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3609 / 第 3609 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3610 / 第 3610 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3611-3620 / 第 3611-3620 行
```cpp
3611 | void __sanitizer_syscall_pre_impl_pathconf(long long path, long long name);
3612 | void __sanitizer_syscall_post_impl_pathconf(long long res, long long path,
3613 |                                             long long name);
3614 | void __sanitizer_syscall_pre_impl_fpathconf(long long fd, long long name);
3615 | void __sanitizer_syscall_post_impl_fpathconf(long long res, long long fd,
3616 |                                              long long name);
3617 | void __sanitizer_syscall_pre_impl_getsockopt2(long long s, long long level,
3618 |                                               long long name, long long val,
3619 |                                               long long avalsize);
3620 | void __sanitizer_syscall_post_impl_getsockopt2(long long res, long long s,
```
- **Line 3611 / 第 3611 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pathconf`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pathconf`。
- **Line 3612 / 第 3612 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3613 / 第 3613 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3614 / 第 3614 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fpathconf`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fpathconf`。
- **Line 3615 / 第 3615 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3616 / 第 3616 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3617 / 第 3617 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3618 / 第 3618 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3619 / 第 3619 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3620 / 第 3620 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3621-3630 / 第 3621-3630 行
```cpp
3621 |                                                long long level, long long name,
3622 |                                                long long val,
3623 |                                                long long avalsize);
3624 | void __sanitizer_syscall_pre_impl_getrlimit(long long which, long long rlp);
3625 | void __sanitizer_syscall_post_impl_getrlimit(long long res, long long which,
3626 |                                              long long rlp);
3627 | void __sanitizer_syscall_pre_impl_setrlimit(long long which, long long rlp);
3628 | void __sanitizer_syscall_post_impl_setrlimit(long long res, long long which,
3629 |                                              long long rlp);
3630 | void __sanitizer_syscall_pre_impl_compat_12_getdirentries(long long fd,
```
- **Line 3621 / 第 3621 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3622 / 第 3622 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3623 / 第 3623 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3624 / 第 3624 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getrlimit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getrlimit`。
- **Line 3625 / 第 3625 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3626 / 第 3626 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3627 / 第 3627 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setrlimit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setrlimit`。
- **Line 3628 / 第 3628 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3629 / 第 3629 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3630 / 第 3630 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3631-3640 / 第 3631-3640 行
```cpp
3631 |                                                           long long buf,
3632 |                                                           long long count,
3633 |                                                           long long basep);
3634 | void __sanitizer_syscall_post_impl_compat_12_getdirentries(long long res,
3635 |                                                            long long fd,
3636 |                                                            long long buf,
3637 |                                                            long long count,
3638 |                                                            long long basep);
3639 | void __sanitizer_syscall_pre_impl_mmap(long long addr, long long len,
3640 |                                        long long prot, long long flags,
```
- **Line 3631 / 第 3631 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3632 / 第 3632 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3633 / 第 3633 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3634 / 第 3634 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3635 / 第 3635 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3636 / 第 3636 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3637 / 第 3637 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3638 / 第 3638 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3639 / 第 3639 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3640 / 第 3640 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3641-3650 / 第 3641-3650 行
```cpp
3641 |                                        long long fd, long long PAD,
3642 |                                        long long pos);
3643 | void __sanitizer_syscall_post_impl_mmap(long long res, long long addr,
3644 |                                         long long len, long long prot,
3645 |                                         long long flags, long long fd,
3646 |                                         long long PAD, long long pos);
3647 | void __sanitizer_syscall_pre_impl___syscall(long long code, long long arg0,
3648 |                                             long long arg1, long long arg2,
3649 |                                             long long arg3, long long arg4,
3650 |                                             long long arg5, long long arg6,
```
- **Line 3641 / 第 3641 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3642 / 第 3642 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3643 / 第 3643 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3644 / 第 3644 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3645 / 第 3645 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3646 / 第 3646 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3647 / 第 3647 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3648 / 第 3648 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3649 / 第 3649 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3650 / 第 3650 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3651-3660 / 第 3651-3660 行
```cpp
3651 |                                             long long arg7);
3652 | void __sanitizer_syscall_post_impl___syscall(long long res, long long code,
3653 |                                              long long arg0, long long arg1,
3654 |                                              long long arg2, long long arg3,
3655 |                                              long long arg4, long long arg5,
3656 |                                              long long arg6, long long arg7);
3657 | void __sanitizer_syscall_pre_impl_lseek(long long fd, long long PAD,
3658 |                                         long long offset, long long whence);
3659 | void __sanitizer_syscall_post_impl_lseek(long long res, long long fd,
3660 |                                          long long PAD, long long offset,
```
- **Line 3651 / 第 3651 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3652 / 第 3652 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3653 / 第 3653 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3654 / 第 3654 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3655 / 第 3655 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3656 / 第 3656 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3657 / 第 3657 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3658 / 第 3658 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3659 / 第 3659 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3660 / 第 3660 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3661-3670 / 第 3661-3670 行
```cpp
3661 |                                          long long whence);
3662 | void __sanitizer_syscall_pre_impl_truncate(long long path, long long PAD,
3663 |                                            long long length);
3664 | void __sanitizer_syscall_post_impl_truncate(long long res, long long path,
3665 |                                             long long PAD, long long length);
3666 | void __sanitizer_syscall_pre_impl_ftruncate(long long fd, long long PAD,
3667 |                                             long long length);
3668 | void __sanitizer_syscall_post_impl_ftruncate(long long res, long long fd,
3669 |                                              long long PAD, long long length);
3670 | void __sanitizer_syscall_pre_impl___sysctl(long long name, long long namelen,
```
- **Line 3661 / 第 3661 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3662 / 第 3662 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3663 / 第 3663 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3664 / 第 3664 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3665 / 第 3665 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3666 / 第 3666 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3667 / 第 3667 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3668 / 第 3668 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3669 / 第 3669 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3670 / 第 3670 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3671-3680 / 第 3671-3680 行
```cpp
3671 |                                            long long oldv, long long oldlenp,
3672 |                                            long long newv, long long newlen);
3673 | void __sanitizer_syscall_post_impl___sysctl(long long res, long long name,
3674 |                                             long long namelen, long long oldv,
3675 |                                             long long oldlenp, long long newv,
3676 |                                             long long newlen);
3677 | void __sanitizer_syscall_pre_impl_mlock(long long addr, long long len);
3678 | void __sanitizer_syscall_post_impl_mlock(long long res, long long addr,
3679 |                                          long long len);
3680 | void __sanitizer_syscall_pre_impl_munlock(long long addr, long long len);
```
- **Line 3671 / 第 3671 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3672 / 第 3672 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3673 / 第 3673 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3674 / 第 3674 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3675 / 第 3675 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3676 / 第 3676 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3677 / 第 3677 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mlock`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mlock`。
- **Line 3678 / 第 3678 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3679 / 第 3679 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3680 / 第 3680 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_munlock`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_munlock`。

### Lines 3681-3690 / 第 3681-3690 行
```cpp
3681 | void __sanitizer_syscall_post_impl_munlock(long long res, long long addr,
3682 |                                            long long len);
3683 | void __sanitizer_syscall_pre_impl_undelete(long long path);
3684 | void __sanitizer_syscall_post_impl_undelete(long long res, long long path);
3685 | void __sanitizer_syscall_pre_impl_compat_50_futimes(long long fd,
3686 |                                                     long long tptr);
3687 | void __sanitizer_syscall_post_impl_compat_50_futimes(long long res,
3688 |                                                      long long fd,
3689 |                                                      long long tptr);
3690 | void __sanitizer_syscall_pre_impl_getpgid(long long pid);
```
- **Line 3681 / 第 3681 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3682 / 第 3682 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3683 / 第 3683 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_undelete`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_undelete`。
- **Line 3684 / 第 3684 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_undelete`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_undelete`。
- **Line 3685 / 第 3685 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3686 / 第 3686 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3687 / 第 3687 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3688 / 第 3688 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3689 / 第 3689 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3690 / 第 3690 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getpgid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getpgid`。

### Lines 3691-3700 / 第 3691-3700 行
```cpp
3691 | void __sanitizer_syscall_post_impl_getpgid(long long res, long long pid);
3692 | void __sanitizer_syscall_pre_impl_reboot(long long opt, long long bootstr);
3693 | void __sanitizer_syscall_post_impl_reboot(long long res, long long opt,
3694 |                                           long long bootstr);
3695 | void __sanitizer_syscall_pre_impl_poll(long long fds, long long nfds,
3696 |                                        long long timeout);
3697 | void __sanitizer_syscall_post_impl_poll(long long res, long long fds,
3698 |                                         long long nfds, long long timeout);
3699 | void __sanitizer_syscall_pre_impl_afssys(long long id, long long a1,
3700 |                                          long long a2, long long a3,
```
- **Line 3691 / 第 3691 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getpgid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getpgid`。
- **Line 3692 / 第 3692 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_reboot`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_reboot`。
- **Line 3693 / 第 3693 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3694 / 第 3694 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3695 / 第 3695 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3696 / 第 3696 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3697 / 第 3697 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3698 / 第 3698 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3699 / 第 3699 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3700 / 第 3700 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3701-3710 / 第 3701-3710 行
```cpp
3701 |                                          long long a4, long long a5,
3702 |                                          long long a6);
3703 | void __sanitizer_syscall_post_impl_afssys(long long res, long long id,
3704 |                                           long long a1, long long a2,
3705 |                                           long long a3, long long a4,
3706 |                                           long long a5, long long a6);
3707 | /* syscall 211 has been skipped */
3708 | /* syscall 212 has been skipped */
3709 | /* syscall 213 has been skipped */
3710 | /* syscall 214 has been skipped */
```
- **Line 3701 / 第 3701 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3702 / 第 3702 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3703 / 第 3703 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3704 / 第 3704 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3705 / 第 3705 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3706 / 第 3706 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3707 / 第 3707 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3708 / 第 3708 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3709 / 第 3709 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3710 / 第 3710 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 3711-3720 / 第 3711-3720 行
```cpp
3711 | /* syscall 215 has been skipped */
3712 | /* syscall 216 has been skipped */
3713 | /* syscall 217 has been skipped */
3714 | /* syscall 218 has been skipped */
3715 | /* syscall 219 has been skipped */
3716 | void __sanitizer_syscall_pre_impl_compat_14___semctl(long long semid,
3717 |                                                      long long semnum,
3718 |                                                      long long cmd,
3719 |                                                      long long arg);
3720 | void __sanitizer_syscall_post_impl_compat_14___semctl(long long res,
```
- **Line 3711 / 第 3711 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3712 / 第 3712 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3713 / 第 3713 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3714 / 第 3714 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3715 / 第 3715 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3716 / 第 3716 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3717 / 第 3717 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3718 / 第 3718 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3719 / 第 3719 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3720 / 第 3720 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3721-3730 / 第 3721-3730 行
```cpp
3721 |                                                       long long semid,
3722 |                                                       long long semnum,
3723 |                                                       long long cmd,
3724 |                                                       long long arg);
3725 | void __sanitizer_syscall_pre_impl_semget(long long key, long long nsems,
3726 |                                          long long semflg);
3727 | void __sanitizer_syscall_post_impl_semget(long long res, long long key,
3728 |                                           long long nsems, long long semflg);
3729 | void __sanitizer_syscall_pre_impl_semop(long long semid, long long sops,
3730 |                                         long long nsops);
```
- **Line 3721 / 第 3721 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3722 / 第 3722 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3723 / 第 3723 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3724 / 第 3724 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3725 / 第 3725 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3726 / 第 3726 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3727 / 第 3727 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3728 / 第 3728 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3729 / 第 3729 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3730 / 第 3730 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3731-3740 / 第 3731-3740 行
```cpp
3731 | void __sanitizer_syscall_post_impl_semop(long long res, long long semid,
3732 |                                          long long sops, long long nsops);
3733 | void __sanitizer_syscall_pre_impl_semconfig(long long flag);
3734 | void __sanitizer_syscall_post_impl_semconfig(long long res, long long flag);
3735 | void __sanitizer_syscall_pre_impl_compat_14_msgctl(long long msqid,
3736 |                                                    long long cmd,
3737 |                                                    long long buf);
3738 | void __sanitizer_syscall_post_impl_compat_14_msgctl(long long res,
3739 |                                                     long long msqid,
3740 |                                                     long long cmd,
```
- **Line 3731 / 第 3731 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3732 / 第 3732 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3733 / 第 3733 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_semconfig`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_semconfig`。
- **Line 3734 / 第 3734 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_semconfig`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_semconfig`。
- **Line 3735 / 第 3735 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3736 / 第 3736 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3737 / 第 3737 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3738 / 第 3738 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3739 / 第 3739 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3740 / 第 3740 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3741-3750 / 第 3741-3750 行
```cpp
3741 |                                                     long long buf);
3742 | void __sanitizer_syscall_pre_impl_msgget(long long key, long long msgflg);
3743 | void __sanitizer_syscall_post_impl_msgget(long long res, long long key,
3744 |                                           long long msgflg);
3745 | void __sanitizer_syscall_pre_impl_msgsnd(long long msqid, long long msgp,
3746 |                                          long long msgsz, long long msgflg);
3747 | void __sanitizer_syscall_post_impl_msgsnd(long long res, long long msqid,
3748 |                                           long long msgp, long long msgsz,
3749 |                                           long long msgflg);
3750 | void __sanitizer_syscall_pre_impl_msgrcv(long long msqid, long long msgp,
```
- **Line 3741 / 第 3741 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3742 / 第 3742 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_msgget`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_msgget`。
- **Line 3743 / 第 3743 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3744 / 第 3744 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3745 / 第 3745 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3746 / 第 3746 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3747 / 第 3747 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3748 / 第 3748 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3749 / 第 3749 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3750 / 第 3750 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3751-3760 / 第 3751-3760 行
```cpp
3751 |                                          long long msgsz, long long msgtyp,
3752 |                                          long long msgflg);
3753 | void __sanitizer_syscall_post_impl_msgrcv(long long res, long long msqid,
3754 |                                           long long msgp, long long msgsz,
3755 |                                           long long msgtyp, long long msgflg);
3756 | void __sanitizer_syscall_pre_impl_shmat(long long shmid, long long shmaddr,
3757 |                                         long long shmflg);
3758 | void __sanitizer_syscall_post_impl_shmat(long long res, long long shmid,
3759 |                                          long long shmaddr, long long shmflg);
3760 | void __sanitizer_syscall_pre_impl_compat_14_shmctl(long long shmid,
```
- **Line 3751 / 第 3751 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3752 / 第 3752 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3753 / 第 3753 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3754 / 第 3754 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3755 / 第 3755 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3756 / 第 3756 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3757 / 第 3757 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3758 / 第 3758 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3759 / 第 3759 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3760 / 第 3760 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3761-3770 / 第 3761-3770 行
```cpp
3761 |                                                    long long cmd,
3762 |                                                    long long buf);
3763 | void __sanitizer_syscall_post_impl_compat_14_shmctl(long long res,
3764 |                                                     long long shmid,
3765 |                                                     long long cmd,
3766 |                                                     long long buf);
3767 | void __sanitizer_syscall_pre_impl_shmdt(long long shmaddr);
3768 | void __sanitizer_syscall_post_impl_shmdt(long long res, long long shmaddr);
3769 | void __sanitizer_syscall_pre_impl_shmget(long long key, long long size,
3770 |                                          long long shmflg);
```
- **Line 3761 / 第 3761 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3762 / 第 3762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3763 / 第 3763 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3764 / 第 3764 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3765 / 第 3765 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3766 / 第 3766 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3767 / 第 3767 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_shmdt`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_shmdt`。
- **Line 3768 / 第 3768 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_shmdt`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_shmdt`。
- **Line 3769 / 第 3769 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3770 / 第 3770 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3771-3780 / 第 3771-3780 行
```cpp
3771 | void __sanitizer_syscall_post_impl_shmget(long long res, long long key,
3772 |                                           long long size, long long shmflg);
3773 | void __sanitizer_syscall_pre_impl_compat_50_clock_gettime(long long clock_id,
3774 |                                                           long long tp);
3775 | void __sanitizer_syscall_post_impl_compat_50_clock_gettime(long long res,
3776 |                                                            long long clock_id,
3777 |                                                            long long tp);
3778 | void __sanitizer_syscall_pre_impl_compat_50_clock_settime(long long clock_id,
3779 |                                                           long long tp);
3780 | void __sanitizer_syscall_post_impl_compat_50_clock_settime(long long res,
```
- **Line 3771 / 第 3771 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3772 / 第 3772 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3773 / 第 3773 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3774 / 第 3774 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3775 / 第 3775 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3776 / 第 3776 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3777 / 第 3777 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3778 / 第 3778 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3779 / 第 3779 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3780 / 第 3780 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3781-3790 / 第 3781-3790 行
```cpp
3781 |                                                            long long clock_id,
3782 |                                                            long long tp);
3783 | void __sanitizer_syscall_pre_impl_compat_50_clock_getres(long long clock_id,
3784 |                                                          long long tp);
3785 | void __sanitizer_syscall_post_impl_compat_50_clock_getres(long long res,
3786 |                                                           long long clock_id,
3787 |                                                           long long tp);
3788 | void __sanitizer_syscall_pre_impl_timer_create(long long clock_id,
3789 |                                                long long evp,
3790 |                                                long long timerid);
```
- **Line 3781 / 第 3781 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3782 / 第 3782 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3783 / 第 3783 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3784 / 第 3784 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3785 / 第 3785 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3786 / 第 3786 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3787 / 第 3787 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3788 / 第 3788 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3789 / 第 3789 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3790 / 第 3790 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3791-3800 / 第 3791-3800 行
```cpp
3791 | void __sanitizer_syscall_post_impl_timer_create(long long res,
3792 |                                                 long long clock_id,
3793 |                                                 long long evp,
3794 |                                                 long long timerid);
3795 | void __sanitizer_syscall_pre_impl_timer_delete(long long timerid);
3796 | void __sanitizer_syscall_post_impl_timer_delete(long long res,
3797 |                                                 long long timerid);
3798 | void __sanitizer_syscall_pre_impl_compat_50_timer_settime(long long timerid,
3799 |                                                           long long flags,
3800 |                                                           long long value,
```
- **Line 3791 / 第 3791 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3792 / 第 3792 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3793 / 第 3793 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3794 / 第 3794 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3795 / 第 3795 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_timer_delete`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_timer_delete`。
- **Line 3796 / 第 3796 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3797 / 第 3797 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3798 / 第 3798 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3799 / 第 3799 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3800 / 第 3800 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3801-3810 / 第 3801-3810 行
```cpp
3801 |                                                           long long ovalue);
3802 | void __sanitizer_syscall_post_impl_compat_50_timer_settime(long long res,
3803 |                                                            long long timerid,
3804 |                                                            long long flags,
3805 |                                                            long long value,
3806 |                                                            long long ovalue);
3807 | void __sanitizer_syscall_pre_impl_compat_50_timer_gettime(long long timerid,
3808 |                                                           long long value);
3809 | void __sanitizer_syscall_post_impl_compat_50_timer_gettime(long long res,
3810 |                                                            long long timerid,
```
- **Line 3801 / 第 3801 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3802 / 第 3802 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3803 / 第 3803 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3804 / 第 3804 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3805 / 第 3805 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3806 / 第 3806 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3807 / 第 3807 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3808 / 第 3808 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3809 / 第 3809 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3810 / 第 3810 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3811-3820 / 第 3811-3820 行
```cpp
3811 |                                                            long long value);
3812 | void __sanitizer_syscall_pre_impl_timer_getoverrun(long long timerid);
3813 | void __sanitizer_syscall_post_impl_timer_getoverrun(long long res,
3814 |                                                     long long timerid);
3815 | void __sanitizer_syscall_pre_impl_compat_50_nanosleep(long long rqtp,
3816 |                                                       long long rmtp);
3817 | void __sanitizer_syscall_post_impl_compat_50_nanosleep(long long res,
3818 |                                                        long long rqtp,
3819 |                                                        long long rmtp);
3820 | void __sanitizer_syscall_pre_impl_fdatasync(long long fd);
```
- **Line 3811 / 第 3811 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3812 / 第 3812 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_timer_getoverrun`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_timer_getoverrun`。
- **Line 3813 / 第 3813 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3814 / 第 3814 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3815 / 第 3815 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3816 / 第 3816 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3817 / 第 3817 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3818 / 第 3818 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3819 / 第 3819 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3820 / 第 3820 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fdatasync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fdatasync`。

### Lines 3821-3830 / 第 3821-3830 行
```cpp
3821 | void __sanitizer_syscall_post_impl_fdatasync(long long res, long long fd);
3822 | void __sanitizer_syscall_pre_impl_mlockall(long long flags);
3823 | void __sanitizer_syscall_post_impl_mlockall(long long res, long long flags);
3824 | void __sanitizer_syscall_pre_impl_munlockall(void);
3825 | void __sanitizer_syscall_post_impl_munlockall(long long res);
3826 | void __sanitizer_syscall_pre_impl_compat_50___sigtimedwait(long long set,
3827 |                                                            long long info,
3828 |                                                            long long timeout);
3829 | void __sanitizer_syscall_post_impl_compat_50___sigtimedwait(long long res,
3830 |                                                             long long set,
```
- **Line 3821 / 第 3821 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fdatasync`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fdatasync`。
- **Line 3822 / 第 3822 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mlockall`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mlockall`。
- **Line 3823 / 第 3823 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_mlockall`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_mlockall`。
- **Line 3824 / 第 3824 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_munlockall`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_munlockall`。
- **Line 3825 / 第 3825 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_munlockall`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_munlockall`。
- **Line 3826 / 第 3826 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3827 / 第 3827 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3828 / 第 3828 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3829 / 第 3829 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3830 / 第 3830 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3831-3840 / 第 3831-3840 行
```cpp
3831 |                                                             long long info,
3832 |                                                             long long timeout);
3833 | void __sanitizer_syscall_pre_impl_sigqueueinfo(long long pid, long long info);
3834 | void __sanitizer_syscall_post_impl_sigqueueinfo(long long res, long long pid,
3835 |                                                 long long info);
3836 | void __sanitizer_syscall_pre_impl_modctl(long long cmd, long long arg);
3837 | void __sanitizer_syscall_post_impl_modctl(long long res, long long cmd,
3838 |                                           long long arg);
3839 | void __sanitizer_syscall_pre_impl__ksem_init(long long value, long long idp);
3840 | void __sanitizer_syscall_post_impl__ksem_init(long long res, long long value,
```
- **Line 3831 / 第 3831 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3832 / 第 3832 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3833 / 第 3833 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sigqueueinfo`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sigqueueinfo`。
- **Line 3834 / 第 3834 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3835 / 第 3835 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3836 / 第 3836 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_modctl`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_modctl`。
- **Line 3837 / 第 3837 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3838 / 第 3838 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3839 / 第 3839 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_init`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_init`。
- **Line 3840 / 第 3840 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3841-3850 / 第 3841-3850 行
```cpp
3841 |                                               long long idp);
3842 | void __sanitizer_syscall_pre_impl__ksem_open(long long name, long long oflag,
3843 |                                              long long mode, long long value,
3844 |                                              long long idp);
3845 | void __sanitizer_syscall_post_impl__ksem_open(long long res, long long name,
3846 |                                               long long oflag, long long mode,
3847 |                                               long long value, long long idp);
3848 | void __sanitizer_syscall_pre_impl__ksem_unlink(long long name);
3849 | void __sanitizer_syscall_post_impl__ksem_unlink(long long res, long long name);
3850 | void __sanitizer_syscall_pre_impl__ksem_close(long long id);
```
- **Line 3841 / 第 3841 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3842 / 第 3842 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3843 / 第 3843 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3844 / 第 3844 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3845 / 第 3845 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3846 / 第 3846 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3847 / 第 3847 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3848 / 第 3848 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_unlink`。
- **Line 3849 / 第 3849 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__ksem_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__ksem_unlink`。
- **Line 3850 / 第 3850 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_close`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_close`。

### Lines 3851-3860 / 第 3851-3860 行
```cpp
3851 | void __sanitizer_syscall_post_impl__ksem_close(long long res, long long id);
3852 | void __sanitizer_syscall_pre_impl__ksem_post(long long id);
3853 | void __sanitizer_syscall_post_impl__ksem_post(long long res, long long id);
3854 | void __sanitizer_syscall_pre_impl__ksem_wait(long long id);
3855 | void __sanitizer_syscall_post_impl__ksem_wait(long long res, long long id);
3856 | void __sanitizer_syscall_pre_impl__ksem_trywait(long long id);
3857 | void __sanitizer_syscall_post_impl__ksem_trywait(long long res, long long id);
3858 | void __sanitizer_syscall_pre_impl__ksem_getvalue(long long id, long long value);
3859 | void __sanitizer_syscall_post_impl__ksem_getvalue(long long res, long long id,
3860 |                                                   long long value);
```
- **Line 3851 / 第 3851 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__ksem_close`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__ksem_close`。
- **Line 3852 / 第 3852 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_post`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_post`。
- **Line 3853 / 第 3853 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__ksem_post`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__ksem_post`。
- **Line 3854 / 第 3854 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_wait`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_wait`。
- **Line 3855 / 第 3855 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__ksem_wait`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__ksem_wait`。
- **Line 3856 / 第 3856 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_trywait`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_trywait`。
- **Line 3857 / 第 3857 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__ksem_trywait`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__ksem_trywait`。
- **Line 3858 / 第 3858 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_getvalue`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_getvalue`。
- **Line 3859 / 第 3859 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3860 / 第 3860 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3861-3870 / 第 3861-3870 行
```cpp
3861 | void __sanitizer_syscall_pre_impl__ksem_destroy(long long id);
3862 | void __sanitizer_syscall_post_impl__ksem_destroy(long long res, long long id);
3863 | void __sanitizer_syscall_pre_impl__ksem_timedwait(long long id,
3864 |                                                   long long abstime);
3865 | void __sanitizer_syscall_post_impl__ksem_timedwait(long long res, long long id,
3866 |                                                    long long abstime);
3867 | void __sanitizer_syscall_pre_impl_mq_open(long long name, long long oflag,
3868 |                                           long long mode, long long attr);
3869 | void __sanitizer_syscall_post_impl_mq_open(long long res, long long name,
3870 |                                            long long oflag, long long mode,
```
- **Line 3861 / 第 3861 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__ksem_destroy`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__ksem_destroy`。
- **Line 3862 / 第 3862 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__ksem_destroy`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__ksem_destroy`。
- **Line 3863 / 第 3863 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3864 / 第 3864 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3865 / 第 3865 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3866 / 第 3866 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3867 / 第 3867 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3868 / 第 3868 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3869 / 第 3869 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3870 / 第 3870 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3871-3880 / 第 3871-3880 行
```cpp
3871 |                                            long long attr);
3872 | void __sanitizer_syscall_pre_impl_mq_close(long long mqdes);
3873 | void __sanitizer_syscall_post_impl_mq_close(long long res, long long mqdes);
3874 | void __sanitizer_syscall_pre_impl_mq_unlink(long long name);
3875 | void __sanitizer_syscall_post_impl_mq_unlink(long long res, long long name);
3876 | void __sanitizer_syscall_pre_impl_mq_getattr(long long mqdes, long long mqstat);
3877 | void __sanitizer_syscall_post_impl_mq_getattr(long long res, long long mqdes,
3878 |                                               long long mqstat);
3879 | void __sanitizer_syscall_pre_impl_mq_setattr(long long mqdes, long long mqstat,
3880 |                                              long long omqstat);
```
- **Line 3871 / 第 3871 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3872 / 第 3872 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mq_close`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mq_close`。
- **Line 3873 / 第 3873 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_mq_close`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_mq_close`。
- **Line 3874 / 第 3874 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mq_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mq_unlink`。
- **Line 3875 / 第 3875 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_mq_unlink`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_mq_unlink`。
- **Line 3876 / 第 3876 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_mq_getattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_mq_getattr`。
- **Line 3877 / 第 3877 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3878 / 第 3878 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3879 / 第 3879 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3880 / 第 3880 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3881-3890 / 第 3881-3890 行
```cpp
3881 | void __sanitizer_syscall_post_impl_mq_setattr(long long res, long long mqdes,
3882 |                                               long long mqstat,
3883 |                                               long long omqstat);
3884 | void __sanitizer_syscall_pre_impl_mq_notify(long long mqdes,
3885 |                                             long long notification);
3886 | void __sanitizer_syscall_post_impl_mq_notify(long long res, long long mqdes,
3887 |                                              long long notification);
3888 | void __sanitizer_syscall_pre_impl_mq_send(long long mqdes, long long msg_ptr,
3889 |                                           long long msg_len,
3890 |                                           long long msg_prio);
```
- **Line 3881 / 第 3881 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3882 / 第 3882 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3883 / 第 3883 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3884 / 第 3884 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3885 / 第 3885 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3886 / 第 3886 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3887 / 第 3887 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3888 / 第 3888 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3889 / 第 3889 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3890 / 第 3890 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3891-3900 / 第 3891-3900 行
```cpp
3891 | void __sanitizer_syscall_post_impl_mq_send(long long res, long long mqdes,
3892 |                                            long long msg_ptr, long long msg_len,
3893 |                                            long long msg_prio);
3894 | void __sanitizer_syscall_pre_impl_mq_receive(long long mqdes, long long msg_ptr,
3895 |                                              long long msg_len,
3896 |                                              long long msg_prio);
3897 | void __sanitizer_syscall_post_impl_mq_receive(long long res, long long mqdes,
3898 |                                               long long msg_ptr,
3899 |                                               long long msg_len,
3900 |                                               long long msg_prio);
```
- **Line 3891 / 第 3891 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3892 / 第 3892 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3893 / 第 3893 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3894 / 第 3894 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3895 / 第 3895 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3896 / 第 3896 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3897 / 第 3897 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3898 / 第 3898 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3899 / 第 3899 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3900 / 第 3900 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3901-3910 / 第 3901-3910 行
```cpp
3901 | void __sanitizer_syscall_pre_impl_compat_50_mq_timedsend(long long mqdes,
3902 |                                                          long long msg_ptr,
3903 |                                                          long long msg_len,
3904 |                                                          long long msg_prio,
3905 |                                                          long long abs_timeout);
3906 | void __sanitizer_syscall_post_impl_compat_50_mq_timedsend(
3907 |     long long res, long long mqdes, long long msg_ptr, long long msg_len,
3908 |     long long msg_prio, long long abs_timeout);
3909 | void __sanitizer_syscall_pre_impl_compat_50_mq_timedreceive(
3910 |     long long mqdes, long long msg_ptr, long long msg_len, long long msg_prio,
```
- **Line 3901 / 第 3901 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3902 / 第 3902 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3903 / 第 3903 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3904 / 第 3904 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3905 / 第 3905 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3906 / 第 3906 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3907 / 第 3907 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3908 / 第 3908 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3909 / 第 3909 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3910 / 第 3910 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3911-3920 / 第 3911-3920 行
```cpp
3911 |     long long abs_timeout);
3912 | void __sanitizer_syscall_post_impl_compat_50_mq_timedreceive(
3913 |     long long res, long long mqdes, long long msg_ptr, long long msg_len,
3914 |     long long msg_prio, long long abs_timeout);
3915 | /* syscall 267 has been skipped */
3916 | /* syscall 268 has been skipped */
3917 | /* syscall 269 has been skipped */
3918 | void __sanitizer_syscall_pre_impl___posix_rename(long long from, long long to);
3919 | void __sanitizer_syscall_post_impl___posix_rename(long long res, long long from,
3920 |                                                   long long to);
```
- **Line 3911 / 第 3911 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3912 / 第 3912 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3913 / 第 3913 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3914 / 第 3914 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3915 / 第 3915 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3916 / 第 3916 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3917 / 第 3917 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3918 / 第 3918 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___posix_rename`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___posix_rename`。
- **Line 3919 / 第 3919 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3920 / 第 3920 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3921-3930 / 第 3921-3930 行
```cpp
3921 | void __sanitizer_syscall_pre_impl_swapctl(long long cmd, long long arg,
3922 |                                           long long misc);
3923 | void __sanitizer_syscall_post_impl_swapctl(long long res, long long cmd,
3924 |                                            long long arg, long long misc);
3925 | void __sanitizer_syscall_pre_impl_compat_30_getdents(long long fd,
3926 |                                                      long long buf,
3927 |                                                      long long count);
3928 | void __sanitizer_syscall_post_impl_compat_30_getdents(long long res,
3929 |                                                       long long fd,
3930 |                                                       long long buf,
```
- **Line 3921 / 第 3921 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3922 / 第 3922 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3923 / 第 3923 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3924 / 第 3924 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3925 / 第 3925 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3926 / 第 3926 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3927 / 第 3927 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3928 / 第 3928 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3929 / 第 3929 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3930 / 第 3930 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3931-3940 / 第 3931-3940 行
```cpp
3931 |                                                       long long count);
3932 | void __sanitizer_syscall_pre_impl_minherit(long long addr, long long len,
3933 |                                            long long inherit);
3934 | void __sanitizer_syscall_post_impl_minherit(long long res, long long addr,
3935 |                                             long long len, long long inherit);
3936 | void __sanitizer_syscall_pre_impl_lchmod(long long path, long long mode);
3937 | void __sanitizer_syscall_post_impl_lchmod(long long res, long long path,
3938 |                                           long long mode);
3939 | void __sanitizer_syscall_pre_impl_lchown(long long path, long long uid,
3940 |                                          long long gid);
```
- **Line 3931 / 第 3931 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3932 / 第 3932 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3933 / 第 3933 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3934 / 第 3934 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3935 / 第 3935 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3936 / 第 3936 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lchmod`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lchmod`。
- **Line 3937 / 第 3937 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3938 / 第 3938 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3939 / 第 3939 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3940 / 第 3940 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3941-3950 / 第 3941-3950 行
```cpp
3941 | void __sanitizer_syscall_post_impl_lchown(long long res, long long path,
3942 |                                           long long uid, long long gid);
3943 | void __sanitizer_syscall_pre_impl_compat_50_lutimes(long long path,
3944 |                                                     long long tptr);
3945 | void __sanitizer_syscall_post_impl_compat_50_lutimes(long long res,
3946 |                                                      long long path,
3947 |                                                      long long tptr);
3948 | void __sanitizer_syscall_pre_impl___msync13(long long addr, long long len,
3949 |                                             long long flags);
3950 | void __sanitizer_syscall_post_impl___msync13(long long res, long long addr,
```
- **Line 3941 / 第 3941 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3942 / 第 3942 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3943 / 第 3943 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3944 / 第 3944 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3945 / 第 3945 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3946 / 第 3946 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3947 / 第 3947 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3948 / 第 3948 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3949 / 第 3949 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3950 / 第 3950 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3951-3960 / 第 3951-3960 行
```cpp
3951 |                                              long long len, long long flags);
3952 | void __sanitizer_syscall_pre_impl_compat_30___stat13(long long path,
3953 |                                                      long long ub);
3954 | void __sanitizer_syscall_post_impl_compat_30___stat13(long long res,
3955 |                                                       long long path,
3956 |                                                       long long ub);
3957 | void __sanitizer_syscall_pre_impl_compat_30___fstat13(long long fd,
3958 |                                                       long long sb);
3959 | void __sanitizer_syscall_post_impl_compat_30___fstat13(long long res,
3960 |                                                        long long fd,
```
- **Line 3951 / 第 3951 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3952 / 第 3952 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3953 / 第 3953 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3954 / 第 3954 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3955 / 第 3955 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3956 / 第 3956 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3957 / 第 3957 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3958 / 第 3958 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3959 / 第 3959 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3960 / 第 3960 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3961-3970 / 第 3961-3970 行
```cpp
3961 |                                                        long long sb);
3962 | void __sanitizer_syscall_pre_impl_compat_30___lstat13(long long path,
3963 |                                                       long long ub);
3964 | void __sanitizer_syscall_post_impl_compat_30___lstat13(long long res,
3965 |                                                        long long path,
3966 |                                                        long long ub);
3967 | void __sanitizer_syscall_pre_impl___sigaltstack14(long long nss, long long oss);
3968 | void __sanitizer_syscall_post_impl___sigaltstack14(long long res, long long nss,
3969 |                                                    long long oss);
3970 | void __sanitizer_syscall_pre_impl___vfork14(void);
```
- **Line 3961 / 第 3961 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3962 / 第 3962 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3963 / 第 3963 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3964 / 第 3964 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3965 / 第 3965 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3966 / 第 3966 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3967 / 第 3967 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___sigaltstack14`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___sigaltstack14`。
- **Line 3968 / 第 3968 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3969 / 第 3969 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3970 / 第 3970 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___vfork14`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___vfork14`。

### Lines 3971-3980 / 第 3971-3980 行
```cpp
3971 | void __sanitizer_syscall_post_impl___vfork14(long long res);
3972 | void __sanitizer_syscall_pre_impl___posix_chown(long long path, long long uid,
3973 |                                                 long long gid);
3974 | void __sanitizer_syscall_post_impl___posix_chown(long long res, long long path,
3975 |                                                  long long uid, long long gid);
3976 | void __sanitizer_syscall_pre_impl___posix_fchown(long long fd, long long uid,
3977 |                                                  long long gid);
3978 | void __sanitizer_syscall_post_impl___posix_fchown(long long res, long long fd,
3979 |                                                   long long uid, long long gid);
3980 | void __sanitizer_syscall_pre_impl___posix_lchown(long long path, long long uid,
```
- **Line 3971 / 第 3971 行**: EN: Declares function or method `__sanitizer_syscall_post_impl___vfork14`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl___vfork14`。
- **Line 3972 / 第 3972 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3973 / 第 3973 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3974 / 第 3974 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3975 / 第 3975 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3976 / 第 3976 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3977 / 第 3977 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3978 / 第 3978 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3979 / 第 3979 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3980 / 第 3980 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 3981-3990 / 第 3981-3990 行
```cpp
3981 |                                                  long long gid);
3982 | void __sanitizer_syscall_post_impl___posix_lchown(long long res, long long path,
3983 |                                                   long long uid, long long gid);
3984 | void __sanitizer_syscall_pre_impl_getsid(long long pid);
3985 | void __sanitizer_syscall_post_impl_getsid(long long res, long long pid);
3986 | void __sanitizer_syscall_pre_impl___clone(long long flags, long long stack);
3987 | void __sanitizer_syscall_post_impl___clone(long long res, long long flags,
3988 |                                            long long stack);
3989 | void __sanitizer_syscall_pre_impl_fktrace(long long fd, long long ops,
3990 |                                           long long facs, long long pid);
```
- **Line 3981 / 第 3981 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3982 / 第 3982 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3983 / 第 3983 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3984 / 第 3984 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getsid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getsid`。
- **Line 3985 / 第 3985 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getsid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getsid`。
- **Line 3986 / 第 3986 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___clone`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___clone`。
- **Line 3987 / 第 3987 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3988 / 第 3988 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3989 / 第 3989 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3990 / 第 3990 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 3991-4000 / 第 3991-4000 行
```cpp
3991 | void __sanitizer_syscall_post_impl_fktrace(long long res, long long fd,
3992 |                                            long long ops, long long facs,
3993 |                                            long long pid);
3994 | void __sanitizer_syscall_pre_impl_preadv(long long fd, long long iovp,
3995 |                                          long long iovcnt, long long PAD,
3996 |                                          long long offset);
3997 | void __sanitizer_syscall_post_impl_preadv(long long res, long long fd,
3998 |                                           long long iovp, long long iovcnt,
3999 |                                           long long PAD, long long offset);
4000 | void __sanitizer_syscall_pre_impl_pwritev(long long fd, long long iovp,
```
- **Line 3991 / 第 3991 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3992 / 第 3992 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3993 / 第 3993 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3994 / 第 3994 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3995 / 第 3995 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3996 / 第 3996 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 3997 / 第 3997 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3998 / 第 3998 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 3999 / 第 3999 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4000 / 第 4000 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4001-4010 / 第 4001-4010 行
```cpp
4001 |                                           long long iovcnt, long long PAD,
4002 |                                           long long offset);
4003 | void __sanitizer_syscall_post_impl_pwritev(long long res, long long fd,
4004 |                                            long long iovp, long long iovcnt,
4005 |                                            long long PAD, long long offset);
4006 | void __sanitizer_syscall_pre_impl_compat_16___sigaction14(long long signum,
4007 |                                                           long long nsa,
4008 |                                                           long long osa);
4009 | void __sanitizer_syscall_post_impl_compat_16___sigaction14(long long res,
4010 |                                                            long long signum,
```
- **Line 4001 / 第 4001 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4002 / 第 4002 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4003 / 第 4003 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4004 / 第 4004 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4005 / 第 4005 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4006 / 第 4006 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4007 / 第 4007 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4008 / 第 4008 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4009 / 第 4009 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4010 / 第 4010 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4011-4020 / 第 4011-4020 行
```cpp
4011 |                                                            long long nsa,
4012 |                                                            long long osa);
4013 | void __sanitizer_syscall_pre_impl___sigpending14(long long set);
4014 | void __sanitizer_syscall_post_impl___sigpending14(long long res, long long set);
4015 | void __sanitizer_syscall_pre_impl___sigprocmask14(long long how, long long set,
4016 |                                                   long long oset);
4017 | void __sanitizer_syscall_post_impl___sigprocmask14(long long res, long long how,
4018 |                                                    long long set,
4019 |                                                    long long oset);
4020 | void __sanitizer_syscall_pre_impl___sigsuspend14(long long set);
```
- **Line 4011 / 第 4011 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4012 / 第 4012 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4013 / 第 4013 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___sigpending14`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___sigpending14`。
- **Line 4014 / 第 4014 行**: EN: Declares function or method `__sanitizer_syscall_post_impl___sigpending14`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl___sigpending14`。
- **Line 4015 / 第 4015 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4016 / 第 4016 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4017 / 第 4017 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4018 / 第 4018 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4019 / 第 4019 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4020 / 第 4020 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___sigsuspend14`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___sigsuspend14`。

### Lines 4021-4030 / 第 4021-4030 行
```cpp
4021 | void __sanitizer_syscall_post_impl___sigsuspend14(long long res, long long set);
4022 | void __sanitizer_syscall_pre_impl_compat_16___sigreturn14(long long sigcntxp);
4023 | void __sanitizer_syscall_post_impl_compat_16___sigreturn14(long long res,
4024 |                                                            long long sigcntxp);
4025 | void __sanitizer_syscall_pre_impl___getcwd(long long bufp, long long length);
4026 | void __sanitizer_syscall_post_impl___getcwd(long long res, long long bufp,
4027 |                                             long long length);
4028 | void __sanitizer_syscall_pre_impl_fchroot(long long fd);
4029 | void __sanitizer_syscall_post_impl_fchroot(long long res, long long fd);
4030 | void __sanitizer_syscall_pre_impl_compat_30_fhopen(long long fhp,
```
- **Line 4021 / 第 4021 行**: EN: Declares function or method `__sanitizer_syscall_post_impl___sigsuspend14`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl___sigsuspend14`。
- **Line 4022 / 第 4022 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_16___sigreturn14`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_16___sigreturn14`。
- **Line 4023 / 第 4023 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4024 / 第 4024 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4025 / 第 4025 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___getcwd`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___getcwd`。
- **Line 4026 / 第 4026 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4027 / 第 4027 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4028 / 第 4028 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fchroot`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fchroot`。
- **Line 4029 / 第 4029 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_fchroot`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_fchroot`。
- **Line 4030 / 第 4030 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4031-4040 / 第 4031-4040 行
```cpp
4031 |                                                    long long flags);
4032 | void __sanitizer_syscall_post_impl_compat_30_fhopen(long long res,
4033 |                                                     long long fhp,
4034 |                                                     long long flags);
4035 | void __sanitizer_syscall_pre_impl_compat_30_fhstat(long long fhp, long long sb);
4036 | void __sanitizer_syscall_post_impl_compat_30_fhstat(long long res,
4037 |                                                     long long fhp,
4038 |                                                     long long sb);
4039 | void __sanitizer_syscall_pre_impl_compat_20_fhstatfs(long long fhp,
4040 |                                                      long long buf);
```
- **Line 4031 / 第 4031 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4032 / 第 4032 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4033 / 第 4033 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4034 / 第 4034 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4035 / 第 4035 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_30_fhstat`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_30_fhstat`。
- **Line 4036 / 第 4036 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4037 / 第 4037 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4038 / 第 4038 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4039 / 第 4039 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4040 / 第 4040 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4041-4050 / 第 4041-4050 行
```cpp
4041 | void __sanitizer_syscall_post_impl_compat_20_fhstatfs(long long res,
4042 |                                                       long long fhp,
4043 |                                                       long long buf);
4044 | void __sanitizer_syscall_pre_impl_compat_50_____semctl13(long long semid,
4045 |                                                          long long semnum,
4046 |                                                          long long cmd,
4047 |                                                          long long arg);
4048 | void __sanitizer_syscall_post_impl_compat_50_____semctl13(long long res,
4049 |                                                           long long semid,
4050 |                                                           long long semnum,
```
- **Line 4041 / 第 4041 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4042 / 第 4042 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4043 / 第 4043 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4044 / 第 4044 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4045 / 第 4045 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4046 / 第 4046 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4047 / 第 4047 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4048 / 第 4048 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4049 / 第 4049 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4050 / 第 4050 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4051-4060 / 第 4051-4060 行
```cpp
4051 |                                                           long long cmd,
4052 |                                                           long long arg);
4053 | void __sanitizer_syscall_pre_impl_compat_50___msgctl13(long long msqid,
4054 |                                                        long long cmd,
4055 |                                                        long long buf);
4056 | void __sanitizer_syscall_post_impl_compat_50___msgctl13(long long res,
4057 |                                                         long long msqid,
4058 |                                                         long long cmd,
4059 |                                                         long long buf);
4060 | void __sanitizer_syscall_pre_impl_compat_50___shmctl13(long long shmid,
```
- **Line 4051 / 第 4051 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4052 / 第 4052 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4053 / 第 4053 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4054 / 第 4054 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4055 / 第 4055 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4056 / 第 4056 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4057 / 第 4057 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4058 / 第 4058 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4059 / 第 4059 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4060 / 第 4060 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4061-4070 / 第 4061-4070 行
```cpp
4061 |                                                        long long cmd,
4062 |                                                        long long buf);
4063 | void __sanitizer_syscall_post_impl_compat_50___shmctl13(long long res,
4064 |                                                         long long shmid,
4065 |                                                         long long cmd,
4066 |                                                         long long buf);
4067 | void __sanitizer_syscall_pre_impl_lchflags(long long path, long long flags);
4068 | void __sanitizer_syscall_post_impl_lchflags(long long res, long long path,
4069 |                                             long long flags);
4070 | void __sanitizer_syscall_pre_impl_issetugid(void);
```
- **Line 4061 / 第 4061 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4062 / 第 4062 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4063 / 第 4063 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4064 / 第 4064 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4065 / 第 4065 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4066 / 第 4066 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4067 / 第 4067 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lchflags`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lchflags`。
- **Line 4068 / 第 4068 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4069 / 第 4069 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4070 / 第 4070 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_issetugid`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_issetugid`。

### Lines 4071-4080 / 第 4071-4080 行
```cpp
4071 | void __sanitizer_syscall_post_impl_issetugid(long long res);
4072 | void __sanitizer_syscall_pre_impl_utrace(long long label, long long addr,
4073 |                                          long long len);
4074 | void __sanitizer_syscall_post_impl_utrace(long long res, long long label,
4075 |                                           long long addr, long long len);
4076 | void __sanitizer_syscall_pre_impl_getcontext(long long ucp);
4077 | void __sanitizer_syscall_post_impl_getcontext(long long res, long long ucp);
4078 | void __sanitizer_syscall_pre_impl_setcontext(long long ucp);
4079 | void __sanitizer_syscall_post_impl_setcontext(long long res, long long ucp);
4080 | void __sanitizer_syscall_pre_impl__lwp_create(long long ucp, long long flags,
```
- **Line 4071 / 第 4071 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_issetugid`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_issetugid`。
- **Line 4072 / 第 4072 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4073 / 第 4073 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4074 / 第 4074 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4075 / 第 4075 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4076 / 第 4076 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_getcontext`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_getcontext`。
- **Line 4077 / 第 4077 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_getcontext`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_getcontext`。
- **Line 4078 / 第 4078 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_setcontext`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_setcontext`。
- **Line 4079 / 第 4079 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_setcontext`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_setcontext`。
- **Line 4080 / 第 4080 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4081-4090 / 第 4081-4090 行
```cpp
4081 |                                               long long new_lwp);
4082 | void __sanitizer_syscall_post_impl__lwp_create(long long res, long long ucp,
4083 |                                                long long flags,
4084 |                                                long long new_lwp);
4085 | void __sanitizer_syscall_pre_impl__lwp_exit(void);
4086 | void __sanitizer_syscall_post_impl__lwp_exit(long long res);
4087 | void __sanitizer_syscall_pre_impl__lwp_self(void);
4088 | void __sanitizer_syscall_post_impl__lwp_self(long long res);
4089 | void __sanitizer_syscall_pre_impl__lwp_wait(long long wait_for,
4090 |                                             long long departed);
```
- **Line 4081 / 第 4081 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4082 / 第 4082 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4083 / 第 4083 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4084 / 第 4084 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4085 / 第 4085 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_exit`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_exit`。
- **Line 4086 / 第 4086 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__lwp_exit`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__lwp_exit`。
- **Line 4087 / 第 4087 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_self`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_self`。
- **Line 4088 / 第 4088 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__lwp_self`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__lwp_self`。
- **Line 4089 / 第 4089 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4090 / 第 4090 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4091-4100 / 第 4091-4100 行
```cpp
4091 | void __sanitizer_syscall_post_impl__lwp_wait(long long res, long long wait_for,
4092 |                                              long long departed);
4093 | void __sanitizer_syscall_pre_impl__lwp_suspend(long long target);
4094 | void __sanitizer_syscall_post_impl__lwp_suspend(long long res,
4095 |                                                 long long target);
4096 | void __sanitizer_syscall_pre_impl__lwp_continue(long long target);
4097 | void __sanitizer_syscall_post_impl__lwp_continue(long long res,
4098 |                                                  long long target);
4099 | void __sanitizer_syscall_pre_impl__lwp_wakeup(long long target);
4100 | void __sanitizer_syscall_post_impl__lwp_wakeup(long long res, long long target);
```
- **Line 4091 / 第 4091 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4092 / 第 4092 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4093 / 第 4093 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_suspend`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_suspend`。
- **Line 4094 / 第 4094 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4095 / 第 4095 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4096 / 第 4096 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_continue`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_continue`。
- **Line 4097 / 第 4097 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4098 / 第 4098 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4099 / 第 4099 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_wakeup`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_wakeup`。
- **Line 4100 / 第 4100 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__lwp_wakeup`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__lwp_wakeup`。

### Lines 4101-4110 / 第 4101-4110 行
```cpp
4101 | void __sanitizer_syscall_pre_impl__lwp_getprivate(void);
4102 | void __sanitizer_syscall_post_impl__lwp_getprivate(long long res);
4103 | void __sanitizer_syscall_pre_impl__lwp_setprivate(long long ptr);
4104 | void __sanitizer_syscall_post_impl__lwp_setprivate(long long res,
4105 |                                                    long long ptr);
4106 | void __sanitizer_syscall_pre_impl__lwp_kill(long long target, long long signo);
4107 | void __sanitizer_syscall_post_impl__lwp_kill(long long res, long long target,
4108 |                                              long long signo);
4109 | void __sanitizer_syscall_pre_impl__lwp_detach(long long target);
4110 | void __sanitizer_syscall_post_impl__lwp_detach(long long res, long long target);
```
- **Line 4101 / 第 4101 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_getprivate`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_getprivate`。
- **Line 4102 / 第 4102 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__lwp_getprivate`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__lwp_getprivate`。
- **Line 4103 / 第 4103 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_setprivate`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_setprivate`。
- **Line 4104 / 第 4104 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4105 / 第 4105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4106 / 第 4106 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_kill`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_kill`。
- **Line 4107 / 第 4107 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4108 / 第 4108 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4109 / 第 4109 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_detach`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_detach`。
- **Line 4110 / 第 4110 行**: EN: Declares function or method `__sanitizer_syscall_post_impl__lwp_detach`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl__lwp_detach`。

### Lines 4111-4120 / 第 4111-4120 行
```cpp
4111 | void __sanitizer_syscall_pre_impl_compat_50__lwp_park(long long ts,
4112 |                                                       long long unpark,
4113 |                                                       long long hint,
4114 |                                                       long long unparkhint);
4115 | void __sanitizer_syscall_post_impl_compat_50__lwp_park(long long res,
4116 |                                                        long long ts,
4117 |                                                        long long unpark,
4118 |                                                        long long hint,
4119 |                                                        long long unparkhint);
4120 | void __sanitizer_syscall_pre_impl__lwp_unpark(long long target, long long hint);
```
- **Line 4111 / 第 4111 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4112 / 第 4112 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4113 / 第 4113 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4114 / 第 4114 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4115 / 第 4115 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4116 / 第 4116 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4117 / 第 4117 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4118 / 第 4118 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4119 / 第 4119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4120 / 第 4120 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__lwp_unpark`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__lwp_unpark`。

### Lines 4121-4130 / 第 4121-4130 行
```cpp
4121 | void __sanitizer_syscall_post_impl__lwp_unpark(long long res, long long target,
4122 |                                                long long hint);
4123 | void __sanitizer_syscall_pre_impl__lwp_unpark_all(long long targets,
4124 |                                                   long long ntargets,
4125 |                                                   long long hint);
4126 | void __sanitizer_syscall_post_impl__lwp_unpark_all(long long res,
4127 |                                                    long long targets,
4128 |                                                    long long ntargets,
4129 |                                                    long long hint);
4130 | void __sanitizer_syscall_pre_impl__lwp_setname(long long target,
```
- **Line 4121 / 第 4121 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4122 / 第 4122 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4123 / 第 4123 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4124 / 第 4124 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4125 / 第 4125 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4126 / 第 4126 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4127 / 第 4127 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4128 / 第 4128 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4129 / 第 4129 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4130 / 第 4130 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4131-4140 / 第 4131-4140 行
```cpp
4131 |                                                long long name);
4132 | void __sanitizer_syscall_post_impl__lwp_setname(long long res, long long target,
4133 |                                                 long long name);
4134 | void __sanitizer_syscall_pre_impl__lwp_getname(long long target, long long name,
4135 |                                                long long len);
4136 | void __sanitizer_syscall_post_impl__lwp_getname(long long res, long long target,
4137 |                                                 long long name, long long len);
4138 | void __sanitizer_syscall_pre_impl__lwp_ctl(long long features,
4139 |                                            long long address);
4140 | void __sanitizer_syscall_post_impl__lwp_ctl(long long res, long long features,
```
- **Line 4131 / 第 4131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4132 / 第 4132 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4133 / 第 4133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4134 / 第 4134 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4135 / 第 4135 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4136 / 第 4136 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4137 / 第 4137 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4138 / 第 4138 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4139 / 第 4139 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4140 / 第 4140 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4141-4150 / 第 4141-4150 行
```cpp
4141 |                                             long long address);
4142 | /* syscall 326 has been skipped */
4143 | /* syscall 327 has been skipped */
4144 | /* syscall 328 has been skipped */
4145 | /* syscall 329 has been skipped */
4146 | void __sanitizer_syscall_pre_impl_compat_60_sa_register(
4147 |     long long newv, long long oldv, long long flags,
4148 |     long long stackinfo_offset);
4149 | void __sanitizer_syscall_post_impl_compat_60_sa_register(
4150 |     long long res, long long newv, long long oldv, long long flags,
```
- **Line 4141 / 第 4141 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4142 / 第 4142 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4143 / 第 4143 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4144 / 第 4144 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4145 / 第 4145 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4146 / 第 4146 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4147 / 第 4147 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4148 / 第 4148 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4149 / 第 4149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4150 / 第 4150 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4151-4160 / 第 4151-4160 行
```cpp
4151 |     long long stackinfo_offset);
4152 | void __sanitizer_syscall_pre_impl_compat_60_sa_stacks(long long num,
4153 |                                                       long long stacks);
4154 | void __sanitizer_syscall_post_impl_compat_60_sa_stacks(long long res,
4155 |                                                        long long num,
4156 |                                                        long long stacks);
4157 | void __sanitizer_syscall_pre_impl_compat_60_sa_enable(void);
4158 | void __sanitizer_syscall_post_impl_compat_60_sa_enable(long long res);
4159 | void __sanitizer_syscall_pre_impl_compat_60_sa_setconcurrency(
4160 |     long long concurrency);
```
- **Line 4151 / 第 4151 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4152 / 第 4152 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4153 / 第 4153 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4154 / 第 4154 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4155 / 第 4155 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4156 / 第 4156 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4157 / 第 4157 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_60_sa_enable`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_60_sa_enable`。
- **Line 4158 / 第 4158 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_60_sa_enable`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_60_sa_enable`。
- **Line 4159 / 第 4159 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4160 / 第 4160 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4161-4170 / 第 4161-4170 行
```cpp
4161 | void __sanitizer_syscall_post_impl_compat_60_sa_setconcurrency(
4162 |     long long res, long long concurrency);
4163 | void __sanitizer_syscall_pre_impl_compat_60_sa_yield(void);
4164 | void __sanitizer_syscall_post_impl_compat_60_sa_yield(long long res);
4165 | void __sanitizer_syscall_pre_impl_compat_60_sa_preempt(long long sa_id);
4166 | void __sanitizer_syscall_post_impl_compat_60_sa_preempt(long long res,
4167 |                                                         long long sa_id);
4168 | /* syscall 336 has been skipped */
4169 | /* syscall 337 has been skipped */
4170 | /* syscall 338 has been skipped */
```
- **Line 4161 / 第 4161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4162 / 第 4162 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4163 / 第 4163 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_60_sa_yield`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_60_sa_yield`。
- **Line 4164 / 第 4164 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_compat_60_sa_yield`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_compat_60_sa_yield`。
- **Line 4165 / 第 4165 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_60_sa_preempt`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_60_sa_preempt`。
- **Line 4166 / 第 4166 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4167 / 第 4167 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4168 / 第 4168 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4169 / 第 4169 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4170 / 第 4170 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 4171-4180 / 第 4171-4180 行
```cpp
4171 | /* syscall 339 has been skipped */
4172 | void __sanitizer_syscall_pre_impl___sigaction_sigtramp(long long signum,
4173 |                                                        long long nsa,
4174 |                                                        long long osa,
4175 |                                                        long long tramp,
4176 |                                                        long long vers);
4177 | void __sanitizer_syscall_post_impl___sigaction_sigtramp(
4178 |     long long res, long long signum, long long nsa, long long osa,
4179 |     long long tramp, long long vers);
4180 | /* syscall 341 has been skipped */
```
- **Line 4171 / 第 4171 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4172 / 第 4172 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4173 / 第 4173 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4174 / 第 4174 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4175 / 第 4175 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4176 / 第 4176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4177 / 第 4177 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4178 / 第 4178 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4179 / 第 4179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4180 / 第 4180 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 4181-4190 / 第 4181-4190 行
```cpp
4181 | /* syscall 342 has been skipped */
4182 | void __sanitizer_syscall_pre_impl_rasctl(long long addr, long long len,
4183 |                                          long long op);
4184 | void __sanitizer_syscall_post_impl_rasctl(long long res, long long addr,
4185 |                                           long long len, long long op);
4186 | void __sanitizer_syscall_pre_impl_kqueue(void);
4187 | void __sanitizer_syscall_post_impl_kqueue(long long res);
4188 | void __sanitizer_syscall_pre_impl_compat_50_kevent(
4189 |     long long fd, long long changelist, long long nchanges, long long eventlist,
4190 |     long long nevents, long long timeout);
```
- **Line 4181 / 第 4181 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4182 / 第 4182 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4183 / 第 4183 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4184 / 第 4184 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4185 / 第 4185 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4186 / 第 4186 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_kqueue`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_kqueue`。
- **Line 4187 / 第 4187 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_kqueue`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_kqueue`。
- **Line 4188 / 第 4188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4189 / 第 4189 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4190 / 第 4190 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4191-4200 / 第 4191-4200 行
```cpp
4191 | void __sanitizer_syscall_post_impl_compat_50_kevent(
4192 |     long long res, long long fd, long long changelist, long long nchanges,
4193 |     long long eventlist, long long nevents, long long timeout);
4194 | void __sanitizer_syscall_pre_impl__sched_setparam(long long pid, long long lid,
4195 |                                                   long long policy,
4196 |                                                   long long params);
4197 | void __sanitizer_syscall_post_impl__sched_setparam(long long res, long long pid,
4198 |                                                    long long lid,
4199 |                                                    long long policy,
4200 |                                                    long long params);
```
- **Line 4191 / 第 4191 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4192 / 第 4192 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4193 / 第 4193 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4194 / 第 4194 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4195 / 第 4195 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4196 / 第 4196 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4197 / 第 4197 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4198 / 第 4198 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4199 / 第 4199 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4200 / 第 4200 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4201-4210 / 第 4201-4210 行
```cpp
4201 | void __sanitizer_syscall_pre_impl__sched_getparam(long long pid, long long lid,
4202 |                                                   long long policy,
4203 |                                                   long long params);
4204 | void __sanitizer_syscall_post_impl__sched_getparam(long long res, long long pid,
4205 |                                                    long long lid,
4206 |                                                    long long policy,
4207 |                                                    long long params);
4208 | void __sanitizer_syscall_pre_impl__sched_setaffinity(long long pid,
4209 |                                                      long long lid,
4210 |                                                      long long size,
```
- **Line 4201 / 第 4201 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4202 / 第 4202 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4203 / 第 4203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4204 / 第 4204 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4205 / 第 4205 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4206 / 第 4206 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4207 / 第 4207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4208 / 第 4208 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4209 / 第 4209 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4210 / 第 4210 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4211-4220 / 第 4211-4220 行
```cpp
4211 |                                                      long long cpuset);
4212 | void __sanitizer_syscall_post_impl__sched_setaffinity(long long res,
4213 |                                                       long long pid,
4214 |                                                       long long lid,
4215 |                                                       long long size,
4216 |                                                       long long cpuset);
4217 | void __sanitizer_syscall_pre_impl__sched_getaffinity(long long pid,
4218 |                                                      long long lid,
4219 |                                                      long long size,
4220 |                                                      long long cpuset);
```
- **Line 4211 / 第 4211 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4212 / 第 4212 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4213 / 第 4213 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4214 / 第 4214 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4215 / 第 4215 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4216 / 第 4216 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4217 / 第 4217 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4218 / 第 4218 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4219 / 第 4219 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4220 / 第 4220 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4221-4230 / 第 4221-4230 行
```cpp
4221 | void __sanitizer_syscall_post_impl__sched_getaffinity(long long res,
4222 |                                                       long long pid,
4223 |                                                       long long lid,
4224 |                                                       long long size,
4225 |                                                       long long cpuset);
4226 | void __sanitizer_syscall_pre_impl_sched_yield(void);
4227 | void __sanitizer_syscall_post_impl_sched_yield(long long res);
4228 | void __sanitizer_syscall_pre_impl__sched_protect(long long priority);
4229 | void __sanitizer_syscall_post_impl__sched_protect(long long res,
4230 |                                                   long long priority);
```
- **Line 4221 / 第 4221 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4222 / 第 4222 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4223 / 第 4223 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4224 / 第 4224 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4225 / 第 4225 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4226 / 第 4226 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_sched_yield`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_sched_yield`。
- **Line 4227 / 第 4227 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_sched_yield`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_sched_yield`。
- **Line 4228 / 第 4228 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl__sched_protect`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl__sched_protect`。
- **Line 4229 / 第 4229 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4230 / 第 4230 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4231-4240 / 第 4231-4240 行
```cpp
4231 | /* syscall 352 has been skipped */
4232 | /* syscall 353 has been skipped */
4233 | void __sanitizer_syscall_pre_impl_fsync_range(long long fd, long long flags,
4234 |                                               long long start,
4235 |                                               long long length);
4236 | void __sanitizer_syscall_post_impl_fsync_range(long long res, long long fd,
4237 |                                                long long flags, long long start,
4238 |                                                long long length);
4239 | void __sanitizer_syscall_pre_impl_uuidgen(long long store, long long count);
4240 | void __sanitizer_syscall_post_impl_uuidgen(long long res, long long store,
```
- **Line 4231 / 第 4231 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4232 / 第 4232 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4233 / 第 4233 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4234 / 第 4234 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4235 / 第 4235 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4236 / 第 4236 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4237 / 第 4237 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4238 / 第 4238 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4239 / 第 4239 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_uuidgen`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_uuidgen`。
- **Line 4240 / 第 4240 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4241-4250 / 第 4241-4250 行
```cpp
4241 |                                            long long count);
4242 | void __sanitizer_syscall_pre_impl_compat_90_getvfsstat(long long buf,
4243 |                                                        long long bufsize,
4244 |                                                        long long flags);
4245 | void __sanitizer_syscall_post_impl_compat_90_getvfsstat(long long res,
4246 |                                                         long long buf,
4247 |                                                         long long bufsize,
4248 |                                                         long long flags);
4249 | void __sanitizer_syscall_pre_impl_compat_90_statvfs1(long long path,
4250 |                                                      long long buf,
```
- **Line 4241 / 第 4241 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4242 / 第 4242 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4243 / 第 4243 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4244 / 第 4244 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4245 / 第 4245 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4246 / 第 4246 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4247 / 第 4247 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4248 / 第 4248 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4249 / 第 4249 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4250 / 第 4250 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4251-4260 / 第 4251-4260 行
```cpp
4251 |                                                      long long flags);
4252 | void __sanitizer_syscall_post_impl_compat_90_statvfs1(long long res,
4253 |                                                       long long path,
4254 |                                                       long long buf,
4255 |                                                       long long flags);
4256 | void __sanitizer_syscall_pre_impl_compat_90_fstatvfs1(long long fd,
4257 |                                                       long long buf,
4258 |                                                       long long flags);
4259 | void __sanitizer_syscall_post_impl_compat_90_fstatvfs1(long long res,
4260 |                                                        long long fd,
```
- **Line 4251 / 第 4251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4252 / 第 4252 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4253 / 第 4253 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4254 / 第 4254 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4255 / 第 4255 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4256 / 第 4256 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4257 / 第 4257 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4258 / 第 4258 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4259 / 第 4259 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4260 / 第 4260 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4261-4270 / 第 4261-4270 行
```cpp
4261 |                                                        long long buf,
4262 |                                                        long long flags);
4263 | void __sanitizer_syscall_pre_impl_compat_30_fhstatvfs1(long long fhp,
4264 |                                                        long long buf,
4265 |                                                        long long flags);
4266 | void __sanitizer_syscall_post_impl_compat_30_fhstatvfs1(long long res,
4267 |                                                         long long fhp,
4268 |                                                         long long buf,
4269 |                                                         long long flags);
4270 | void __sanitizer_syscall_pre_impl_extattrctl(long long path, long long cmd,
```
- **Line 4261 / 第 4261 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4262 / 第 4262 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4263 / 第 4263 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4264 / 第 4264 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4265 / 第 4265 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4266 / 第 4266 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4267 / 第 4267 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4268 / 第 4268 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4269 / 第 4269 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4270 / 第 4270 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4271-4280 / 第 4271-4280 行
```cpp
4271 |                                              long long filename,
4272 |                                              long long attrnamespace,
4273 |                                              long long attrname);
4274 | void __sanitizer_syscall_post_impl_extattrctl(long long res, long long path,
4275 |                                               long long cmd, long long filename,
4276 |                                               long long attrnamespace,
4277 |                                               long long attrname);
4278 | void __sanitizer_syscall_pre_impl_extattr_set_file(long long path,
4279 |                                                    long long attrnamespace,
4280 |                                                    long long attrname,
```
- **Line 4271 / 第 4271 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4272 / 第 4272 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4273 / 第 4273 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4274 / 第 4274 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4275 / 第 4275 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4276 / 第 4276 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4277 / 第 4277 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4278 / 第 4278 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4279 / 第 4279 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4280 / 第 4280 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4281-4290 / 第 4281-4290 行
```cpp
4281 |                                                    long long data,
4282 |                                                    long long nbytes);
4283 | void __sanitizer_syscall_post_impl_extattr_set_file(
4284 |     long long res, long long path, long long attrnamespace, long long attrname,
4285 |     long long data, long long nbytes);
4286 | void __sanitizer_syscall_pre_impl_extattr_get_file(long long path,
4287 |                                                    long long attrnamespace,
4288 |                                                    long long attrname,
4289 |                                                    long long data,
4290 |                                                    long long nbytes);
```
- **Line 4281 / 第 4281 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4282 / 第 4282 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4283 / 第 4283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4284 / 第 4284 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4285 / 第 4285 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4286 / 第 4286 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4287 / 第 4287 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4288 / 第 4288 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4289 / 第 4289 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4290 / 第 4290 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4291-4300 / 第 4291-4300 行
```cpp
4291 | void __sanitizer_syscall_post_impl_extattr_get_file(
4292 |     long long res, long long path, long long attrnamespace, long long attrname,
4293 |     long long data, long long nbytes);
4294 | void __sanitizer_syscall_pre_impl_extattr_delete_file(long long path,
4295 |                                                       long long attrnamespace,
4296 |                                                       long long attrname);
4297 | void __sanitizer_syscall_post_impl_extattr_delete_file(long long res,
4298 |                                                        long long path,
4299 |                                                        long long attrnamespace,
4300 |                                                        long long attrname);
```
- **Line 4291 / 第 4291 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4292 / 第 4292 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4293 / 第 4293 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4294 / 第 4294 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4295 / 第 4295 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4296 / 第 4296 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4297 / 第 4297 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4298 / 第 4298 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4299 / 第 4299 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4300 / 第 4300 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4301-4310 / 第 4301-4310 行
```cpp
4301 | void __sanitizer_syscall_pre_impl_extattr_set_fd(long long fd,
4302 |                                                  long long attrnamespace,
4303 |                                                  long long attrname,
4304 |                                                  long long data,
4305 |                                                  long long nbytes);
4306 | void __sanitizer_syscall_post_impl_extattr_set_fd(long long res, long long fd,
4307 |                                                   long long attrnamespace,
4308 |                                                   long long attrname,
4309 |                                                   long long data,
4310 |                                                   long long nbytes);
```
- **Line 4301 / 第 4301 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4302 / 第 4302 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4303 / 第 4303 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4304 / 第 4304 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4305 / 第 4305 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4306 / 第 4306 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4307 / 第 4307 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4308 / 第 4308 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4309 / 第 4309 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4310 / 第 4310 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4311-4320 / 第 4311-4320 行
```cpp
4311 | void __sanitizer_syscall_pre_impl_extattr_get_fd(long long fd,
4312 |                                                  long long attrnamespace,
4313 |                                                  long long attrname,
4314 |                                                  long long data,
4315 |                                                  long long nbytes);
4316 | void __sanitizer_syscall_post_impl_extattr_get_fd(long long res, long long fd,
4317 |                                                   long long attrnamespace,
4318 |                                                   long long attrname,
4319 |                                                   long long data,
4320 |                                                   long long nbytes);
```
- **Line 4311 / 第 4311 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4312 / 第 4312 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4313 / 第 4313 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4314 / 第 4314 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4315 / 第 4315 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4316 / 第 4316 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4317 / 第 4317 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4318 / 第 4318 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4319 / 第 4319 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4320 / 第 4320 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4321-4330 / 第 4321-4330 行
```cpp
4321 | void __sanitizer_syscall_pre_impl_extattr_delete_fd(long long fd,
4322 |                                                     long long attrnamespace,
4323 |                                                     long long attrname);
4324 | void __sanitizer_syscall_post_impl_extattr_delete_fd(long long res,
4325 |                                                      long long fd,
4326 |                                                      long long attrnamespace,
4327 |                                                      long long attrname);
4328 | void __sanitizer_syscall_pre_impl_extattr_set_link(long long path,
4329 |                                                    long long attrnamespace,
4330 |                                                    long long attrname,
```
- **Line 4321 / 第 4321 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4322 / 第 4322 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4323 / 第 4323 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4324 / 第 4324 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4325 / 第 4325 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4326 / 第 4326 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4327 / 第 4327 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4328 / 第 4328 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4329 / 第 4329 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4330 / 第 4330 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4331-4340 / 第 4331-4340 行
```cpp
4331 |                                                    long long data,
4332 |                                                    long long nbytes);
4333 | void __sanitizer_syscall_post_impl_extattr_set_link(
4334 |     long long res, long long path, long long attrnamespace, long long attrname,
4335 |     long long data, long long nbytes);
4336 | void __sanitizer_syscall_pre_impl_extattr_get_link(long long path,
4337 |                                                    long long attrnamespace,
4338 |                                                    long long attrname,
4339 |                                                    long long data,
4340 |                                                    long long nbytes);
```
- **Line 4331 / 第 4331 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4332 / 第 4332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4333 / 第 4333 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4334 / 第 4334 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4335 / 第 4335 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4336 / 第 4336 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4337 / 第 4337 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4338 / 第 4338 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4339 / 第 4339 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4340 / 第 4340 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4341-4350 / 第 4341-4350 行
```cpp
4341 | void __sanitizer_syscall_post_impl_extattr_get_link(
4342 |     long long res, long long path, long long attrnamespace, long long attrname,
4343 |     long long data, long long nbytes);
4344 | void __sanitizer_syscall_pre_impl_extattr_delete_link(long long path,
4345 |                                                       long long attrnamespace,
4346 |                                                       long long attrname);
4347 | void __sanitizer_syscall_post_impl_extattr_delete_link(long long res,
4348 |                                                        long long path,
4349 |                                                        long long attrnamespace,
4350 |                                                        long long attrname);
```
- **Line 4341 / 第 4341 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4342 / 第 4342 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4343 / 第 4343 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4344 / 第 4344 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4345 / 第 4345 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4346 / 第 4346 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4347 / 第 4347 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4348 / 第 4348 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4349 / 第 4349 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4350 / 第 4350 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4351-4360 / 第 4351-4360 行
```cpp
4351 | void __sanitizer_syscall_pre_impl_extattr_list_fd(long long fd,
4352 |                                                   long long attrnamespace,
4353 |                                                   long long data,
4354 |                                                   long long nbytes);
4355 | void __sanitizer_syscall_post_impl_extattr_list_fd(long long res, long long fd,
4356 |                                                    long long attrnamespace,
4357 |                                                    long long data,
4358 |                                                    long long nbytes);
4359 | void __sanitizer_syscall_pre_impl_extattr_list_file(long long path,
4360 |                                                     long long attrnamespace,
```
- **Line 4351 / 第 4351 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4352 / 第 4352 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4353 / 第 4353 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4354 / 第 4354 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4355 / 第 4355 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4356 / 第 4356 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4357 / 第 4357 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4358 / 第 4358 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4359 / 第 4359 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4360 / 第 4360 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4361-4370 / 第 4361-4370 行
```cpp
4361 |                                                     long long data,
4362 |                                                     long long nbytes);
4363 | void __sanitizer_syscall_post_impl_extattr_list_file(long long res,
4364 |                                                      long long path,
4365 |                                                      long long attrnamespace,
4366 |                                                      long long data,
4367 |                                                      long long nbytes);
4368 | void __sanitizer_syscall_pre_impl_extattr_list_link(long long path,
4369 |                                                     long long attrnamespace,
4370 |                                                     long long data,
```
- **Line 4361 / 第 4361 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4362 / 第 4362 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4363 / 第 4363 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4364 / 第 4364 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4365 / 第 4365 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4366 / 第 4366 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4367 / 第 4367 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4368 / 第 4368 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4369 / 第 4369 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4370 / 第 4370 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4371-4380 / 第 4371-4380 行
```cpp
4371 |                                                     long long nbytes);
4372 | void __sanitizer_syscall_post_impl_extattr_list_link(long long res,
4373 |                                                      long long path,
4374 |                                                      long long attrnamespace,
4375 |                                                      long long data,
4376 |                                                      long long nbytes);
4377 | void __sanitizer_syscall_pre_impl_compat_50_pselect(long long nd, long long in,
4378 |                                                     long long ou, long long ex,
4379 |                                                     long long ts,
4380 |                                                     long long mask);
```
- **Line 4371 / 第 4371 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4372 / 第 4372 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4373 / 第 4373 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4374 / 第 4374 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4375 / 第 4375 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4376 / 第 4376 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4377 / 第 4377 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4378 / 第 4378 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4379 / 第 4379 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4380 / 第 4380 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4381-4390 / 第 4381-4390 行
```cpp
4381 | void __sanitizer_syscall_post_impl_compat_50_pselect(long long res,
4382 |                                                      long long nd, long long in,
4383 |                                                      long long ou, long long ex,
4384 |                                                      long long ts,
4385 |                                                      long long mask);
4386 | void __sanitizer_syscall_pre_impl_compat_50_pollts(long long fds,
4387 |                                                    long long nfds, long long ts,
4388 |                                                    long long mask);
4389 | void __sanitizer_syscall_post_impl_compat_50_pollts(
4390 |     long long res, long long fds, long long nfds, long long ts, long long mask);
```
- **Line 4381 / 第 4381 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4382 / 第 4382 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4383 / 第 4383 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4384 / 第 4384 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4385 / 第 4385 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4386 / 第 4386 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4387 / 第 4387 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4388 / 第 4388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4389 / 第 4389 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4390 / 第 4390 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4391-4400 / 第 4391-4400 行
```cpp
4391 | void __sanitizer_syscall_pre_impl_setxattr(long long path, long long name,
4392 |                                            long long value, long long size,
4393 |                                            long long flags);
4394 | void __sanitizer_syscall_post_impl_setxattr(long long res, long long path,
4395 |                                             long long name, long long value,
4396 |                                             long long size, long long flags);
4397 | void __sanitizer_syscall_pre_impl_lsetxattr(long long path, long long name,
4398 |                                             long long value, long long size,
4399 |                                             long long flags);
4400 | void __sanitizer_syscall_post_impl_lsetxattr(long long res, long long path,
```
- **Line 4391 / 第 4391 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4392 / 第 4392 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4393 / 第 4393 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4394 / 第 4394 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4395 / 第 4395 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4396 / 第 4396 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4397 / 第 4397 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4398 / 第 4398 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4399 / 第 4399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4400 / 第 4400 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4401-4410 / 第 4401-4410 行
```cpp
4401 |                                              long long name, long long value,
4402 |                                              long long size, long long flags);
4403 | void __sanitizer_syscall_pre_impl_fsetxattr(long long fd, long long name,
4404 |                                             long long value, long long size,
4405 |                                             long long flags);
4406 | void __sanitizer_syscall_post_impl_fsetxattr(long long res, long long fd,
4407 |                                              long long name, long long value,
4408 |                                              long long size, long long flags);
4409 | void __sanitizer_syscall_pre_impl_getxattr(long long path, long long name,
4410 |                                            long long value, long long size);
```
- **Line 4401 / 第 4401 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4402 / 第 4402 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4403 / 第 4403 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4404 / 第 4404 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4405 / 第 4405 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4406 / 第 4406 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4407 / 第 4407 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4408 / 第 4408 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4409 / 第 4409 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4410 / 第 4410 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4411-4420 / 第 4411-4420 行
```cpp
4411 | void __sanitizer_syscall_post_impl_getxattr(long long res, long long path,
4412 |                                             long long name, long long value,
4413 |                                             long long size);
4414 | void __sanitizer_syscall_pre_impl_lgetxattr(long long path, long long name,
4415 |                                             long long value, long long size);
4416 | void __sanitizer_syscall_post_impl_lgetxattr(long long res, long long path,
4417 |                                              long long name, long long value,
4418 |                                              long long size);
4419 | void __sanitizer_syscall_pre_impl_fgetxattr(long long fd, long long name,
4420 |                                             long long value, long long size);
```
- **Line 4411 / 第 4411 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4412 / 第 4412 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4413 / 第 4413 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4414 / 第 4414 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4415 / 第 4415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4416 / 第 4416 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4417 / 第 4417 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4418 / 第 4418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4419 / 第 4419 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4420 / 第 4420 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4421-4430 / 第 4421-4430 行
```cpp
4421 | void __sanitizer_syscall_post_impl_fgetxattr(long long res, long long fd,
4422 |                                              long long name, long long value,
4423 |                                              long long size);
4424 | void __sanitizer_syscall_pre_impl_listxattr(long long path, long long list,
4425 |                                             long long size);
4426 | void __sanitizer_syscall_post_impl_listxattr(long long res, long long path,
4427 |                                              long long list, long long size);
4428 | void __sanitizer_syscall_pre_impl_llistxattr(long long path, long long list,
4429 |                                              long long size);
4430 | void __sanitizer_syscall_post_impl_llistxattr(long long res, long long path,
```
- **Line 4421 / 第 4421 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4422 / 第 4422 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4423 / 第 4423 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4424 / 第 4424 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4425 / 第 4425 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4426 / 第 4426 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4427 / 第 4427 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4428 / 第 4428 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4429 / 第 4429 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4430 / 第 4430 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4431-4440 / 第 4431-4440 行
```cpp
4431 |                                               long long list, long long size);
4432 | void __sanitizer_syscall_pre_impl_flistxattr(long long fd, long long list,
4433 |                                              long long size);
4434 | void __sanitizer_syscall_post_impl_flistxattr(long long res, long long fd,
4435 |                                               long long list, long long size);
4436 | void __sanitizer_syscall_pre_impl_removexattr(long long path, long long name);
4437 | void __sanitizer_syscall_post_impl_removexattr(long long res, long long path,
4438 |                                                long long name);
4439 | void __sanitizer_syscall_pre_impl_lremovexattr(long long path, long long name);
4440 | void __sanitizer_syscall_post_impl_lremovexattr(long long res, long long path,
```
- **Line 4431 / 第 4431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4432 / 第 4432 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4433 / 第 4433 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4434 / 第 4434 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4435 / 第 4435 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4436 / 第 4436 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_removexattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_removexattr`。
- **Line 4437 / 第 4437 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4438 / 第 4438 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4439 / 第 4439 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lremovexattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lremovexattr`。
- **Line 4440 / 第 4440 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4441-4450 / 第 4441-4450 行
```cpp
4441 |                                                 long long name);
4442 | void __sanitizer_syscall_pre_impl_fremovexattr(long long fd, long long name);
4443 | void __sanitizer_syscall_post_impl_fremovexattr(long long res, long long fd,
4444 |                                                 long long name);
4445 | void __sanitizer_syscall_pre_impl_compat_50___stat30(long long path,
4446 |                                                      long long ub);
4447 | void __sanitizer_syscall_post_impl_compat_50___stat30(long long res,
4448 |                                                       long long path,
4449 |                                                       long long ub);
4450 | void __sanitizer_syscall_pre_impl_compat_50___fstat30(long long fd,
```
- **Line 4441 / 第 4441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4442 / 第 4442 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_fremovexattr`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_fremovexattr`。
- **Line 4443 / 第 4443 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4444 / 第 4444 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4445 / 第 4445 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4446 / 第 4446 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4447 / 第 4447 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4448 / 第 4448 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4449 / 第 4449 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4450 / 第 4450 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4451-4460 / 第 4451-4460 行
```cpp
4451 |                                                       long long sb);
4452 | void __sanitizer_syscall_post_impl_compat_50___fstat30(long long res,
4453 |                                                        long long fd,
4454 |                                                        long long sb);
4455 | void __sanitizer_syscall_pre_impl_compat_50___lstat30(long long path,
4456 |                                                       long long ub);
4457 | void __sanitizer_syscall_post_impl_compat_50___lstat30(long long res,
4458 |                                                        long long path,
4459 |                                                        long long ub);
4460 | void __sanitizer_syscall_pre_impl___getdents30(long long fd, long long buf,
```
- **Line 4451 / 第 4451 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4452 / 第 4452 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4453 / 第 4453 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4454 / 第 4454 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4455 / 第 4455 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4456 / 第 4456 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4457 / 第 4457 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4458 / 第 4458 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4459 / 第 4459 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4460 / 第 4460 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4461-4470 / 第 4461-4470 行
```cpp
4461 |                                                long long count);
4462 | void __sanitizer_syscall_post_impl___getdents30(long long res, long long fd,
4463 |                                                 long long buf, long long count);
4464 | void __sanitizer_syscall_pre_impl_posix_fadvise(long long);
4465 | void __sanitizer_syscall_post_impl_posix_fadvise(long long res, long long);
4466 | void __sanitizer_syscall_pre_impl_compat_30___fhstat30(long long fhp,
4467 |                                                        long long sb);
4468 | void __sanitizer_syscall_post_impl_compat_30___fhstat30(long long res,
4469 |                                                         long long fhp,
4470 |                                                         long long sb);
```
- **Line 4461 / 第 4461 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4462 / 第 4462 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4463 / 第 4463 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4464 / 第 4464 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_posix_fadvise`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_posix_fadvise`。
- **Line 4465 / 第 4465 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_posix_fadvise`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_posix_fadvise`。
- **Line 4466 / 第 4466 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4467 / 第 4467 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4468 / 第 4468 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4469 / 第 4469 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4470 / 第 4470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4471-4480 / 第 4471-4480 行
```cpp
4471 | void __sanitizer_syscall_pre_impl_compat_50___ntp_gettime30(long long ntvp);
4472 | void __sanitizer_syscall_post_impl_compat_50___ntp_gettime30(long long res,
4473 |                                                              long long ntvp);
4474 | void __sanitizer_syscall_pre_impl___socket30(long long domain, long long type,
4475 |                                              long long protocol);
4476 | void __sanitizer_syscall_post_impl___socket30(long long res, long long domain,
4477 |                                               long long type,
4478 |                                               long long protocol);
4479 | void __sanitizer_syscall_pre_impl___getfh30(long long fname, long long fhp,
4480 |                                             long long fh_size);
```
- **Line 4471 / 第 4471 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_compat_50___ntp_gettime30`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_compat_50___ntp_gettime30`。
- **Line 4472 / 第 4472 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4473 / 第 4473 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4474 / 第 4474 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4475 / 第 4475 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4476 / 第 4476 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4477 / 第 4477 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4478 / 第 4478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4479 / 第 4479 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4480 / 第 4480 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4481-4490 / 第 4481-4490 行
```cpp
4481 | void __sanitizer_syscall_post_impl___getfh30(long long res, long long fname,
4482 |                                              long long fhp, long long fh_size);
4483 | void __sanitizer_syscall_pre_impl___fhopen40(long long fhp, long long fh_size,
4484 |                                              long long flags);
4485 | void __sanitizer_syscall_post_impl___fhopen40(long long res, long long fhp,
4486 |                                               long long fh_size,
4487 |                                               long long flags);
4488 | void __sanitizer_syscall_pre_impl_compat_90_fhstatvfs1(long long fhp,
4489 |                                                        long long fh_size,
4490 |                                                        long long buf,
```
- **Line 4481 / 第 4481 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4482 / 第 4482 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4483 / 第 4483 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4484 / 第 4484 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4485 / 第 4485 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4486 / 第 4486 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4487 / 第 4487 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4488 / 第 4488 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4489 / 第 4489 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4490 / 第 4490 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4491-4500 / 第 4491-4500 行
```cpp
4491 |                                                        long long flags);
4492 | void __sanitizer_syscall_post_impl_compat_90_fhstatvfs1(long long res,
4493 |                                                         long long fhp,
4494 |                                                         long long fh_size,
4495 |                                                         long long buf,
4496 |                                                         long long flags);
4497 | void __sanitizer_syscall_pre_impl_compat_50___fhstat40(long long fhp,
4498 |                                                        long long fh_size,
4499 |                                                        long long sb);
4500 | void __sanitizer_syscall_post_impl_compat_50___fhstat40(long long res,
```
- **Line 4491 / 第 4491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4492 / 第 4492 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4493 / 第 4493 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4494 / 第 4494 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4495 / 第 4495 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4496 / 第 4496 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4497 / 第 4497 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4498 / 第 4498 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4499 / 第 4499 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4500 / 第 4500 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4501-4510 / 第 4501-4510 行
```cpp
4501 |                                                         long long fhp,
4502 |                                                         long long fh_size,
4503 |                                                         long long sb);
4504 | void __sanitizer_syscall_pre_impl_aio_cancel(long long fildes,
4505 |                                              long long aiocbp);
4506 | void __sanitizer_syscall_post_impl_aio_cancel(long long res, long long fildes,
4507 |                                               long long aiocbp);
4508 | void __sanitizer_syscall_pre_impl_aio_error(long long aiocbp);
4509 | void __sanitizer_syscall_post_impl_aio_error(long long res, long long aiocbp);
4510 | void __sanitizer_syscall_pre_impl_aio_fsync(long long op, long long aiocbp);
```
- **Line 4501 / 第 4501 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4502 / 第 4502 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4503 / 第 4503 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4504 / 第 4504 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4505 / 第 4505 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4506 / 第 4506 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4507 / 第 4507 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4508 / 第 4508 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_aio_error`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_aio_error`。
- **Line 4509 / 第 4509 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_aio_error`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_aio_error`。
- **Line 4510 / 第 4510 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_aio_fsync`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_aio_fsync`。

### Lines 4511-4520 / 第 4511-4520 行
```cpp
4511 | void __sanitizer_syscall_post_impl_aio_fsync(long long res, long long op,
4512 |                                              long long aiocbp);
4513 | void __sanitizer_syscall_pre_impl_aio_read(long long aiocbp);
4514 | void __sanitizer_syscall_post_impl_aio_read(long long res, long long aiocbp);
4515 | void __sanitizer_syscall_pre_impl_aio_return(long long aiocbp);
4516 | void __sanitizer_syscall_post_impl_aio_return(long long res, long long aiocbp);
4517 | void __sanitizer_syscall_pre_impl_compat_50_aio_suspend(long long list,
4518 |                                                         long long nent,
4519 |                                                         long long timeout);
4520 | void __sanitizer_syscall_post_impl_compat_50_aio_suspend(long long res,
```
- **Line 4511 / 第 4511 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4512 / 第 4512 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4513 / 第 4513 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_aio_read`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_aio_read`。
- **Line 4514 / 第 4514 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_aio_read`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_aio_read`。
- **Line 4515 / 第 4515 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_aio_return`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_aio_return`。
- **Line 4516 / 第 4516 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_aio_return`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_aio_return`。
- **Line 4517 / 第 4517 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4518 / 第 4518 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4519 / 第 4519 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4520 / 第 4520 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4521-4530 / 第 4521-4530 行
```cpp
4521 |                                                          long long list,
4522 |                                                          long long nent,
4523 |                                                          long long timeout);
4524 | void __sanitizer_syscall_pre_impl_aio_write(long long aiocbp);
4525 | void __sanitizer_syscall_post_impl_aio_write(long long res, long long aiocbp);
4526 | void __sanitizer_syscall_pre_impl_lio_listio(long long mode, long long list,
4527 |                                              long long nent, long long sig);
4528 | void __sanitizer_syscall_post_impl_lio_listio(long long res, long long mode,
4529 |                                               long long list, long long nent,
4530 |                                               long long sig);
```
- **Line 4521 / 第 4521 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4522 / 第 4522 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4523 / 第 4523 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4524 / 第 4524 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_aio_write`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_aio_write`。
- **Line 4525 / 第 4525 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_aio_write`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_aio_write`。
- **Line 4526 / 第 4526 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4527 / 第 4527 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4528 / 第 4528 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4529 / 第 4529 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4530 / 第 4530 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4531-4540 / 第 4531-4540 行
```cpp
4531 | /* syscall 407 has been skipped */
4532 | /* syscall 408 has been skipped */
4533 | /* syscall 409 has been skipped */
4534 | void __sanitizer_syscall_pre_impl___mount50(long long type, long long path,
4535 |                                             long long flags, long long data,
4536 |                                             long long data_len);
4537 | void __sanitizer_syscall_post_impl___mount50(long long res, long long type,
4538 |                                              long long path, long long flags,
4539 |                                              long long data,
4540 |                                              long long data_len);
```
- **Line 4531 / 第 4531 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4532 / 第 4532 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4533 / 第 4533 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4534 / 第 4534 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4535 / 第 4535 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4536 / 第 4536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4537 / 第 4537 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4538 / 第 4538 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4539 / 第 4539 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4540 / 第 4540 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4541-4550 / 第 4541-4550 行
```cpp
4541 | void __sanitizer_syscall_pre_impl_mremap(long long old_address,
4542 |                                          long long old_size,
4543 |                                          long long new_address,
4544 |                                          long long new_size, long long flags);
4545 | void __sanitizer_syscall_post_impl_mremap(long long res, long long old_address,
4546 |                                           long long old_size,
4547 |                                           long long new_address,
4548 |                                           long long new_size, long long flags);
4549 | void __sanitizer_syscall_pre_impl_pset_create(long long psid);
4550 | void __sanitizer_syscall_post_impl_pset_create(long long res, long long psid);
```
- **Line 4541 / 第 4541 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4542 / 第 4542 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4543 / 第 4543 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4544 / 第 4544 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4545 / 第 4545 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4546 / 第 4546 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4547 / 第 4547 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4548 / 第 4548 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4549 / 第 4549 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pset_create`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pset_create`。
- **Line 4550 / 第 4550 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_pset_create`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_pset_create`。

### Lines 4551-4560 / 第 4551-4560 行
```cpp
4551 | void __sanitizer_syscall_pre_impl_pset_destroy(long long psid);
4552 | void __sanitizer_syscall_post_impl_pset_destroy(long long res, long long psid);
4553 | void __sanitizer_syscall_pre_impl_pset_assign(long long psid, long long cpuid,
4554 |                                               long long opsid);
4555 | void __sanitizer_syscall_post_impl_pset_assign(long long res, long long psid,
4556 |                                                long long cpuid,
4557 |                                                long long opsid);
4558 | void __sanitizer_syscall_pre_impl__pset_bind(long long idtype,
4559 |                                              long long first_id,
4560 |                                              long long second_id,
```
- **Line 4551 / 第 4551 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pset_destroy`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pset_destroy`。
- **Line 4552 / 第 4552 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_pset_destroy`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_pset_destroy`。
- **Line 4553 / 第 4553 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4554 / 第 4554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4555 / 第 4555 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4556 / 第 4556 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4557 / 第 4557 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4558 / 第 4558 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4559 / 第 4559 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4560 / 第 4560 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4561-4570 / 第 4561-4570 行
```cpp
4561 |                                              long long psid, long long opsid);
4562 | void __sanitizer_syscall_post_impl__pset_bind(long long res, long long idtype,
4563 |                                               long long first_id,
4564 |                                               long long second_id,
4565 |                                               long long psid, long long opsid);
4566 | void __sanitizer_syscall_pre_impl___posix_fadvise50(long long fd, long long PAD,
4567 |                                                     long long offset,
4568 |                                                     long long len,
4569 |                                                     long long advice);
4570 | void __sanitizer_syscall_post_impl___posix_fadvise50(
```
- **Line 4561 / 第 4561 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4562 / 第 4562 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4563 / 第 4563 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4564 / 第 4564 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4565 / 第 4565 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4566 / 第 4566 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4567 / 第 4567 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4568 / 第 4568 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4569 / 第 4569 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4570 / 第 4570 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4571-4580 / 第 4571-4580 行
```cpp
4571 |     long long res, long long fd, long long PAD, long long offset, long long len,
4572 |     long long advice);
4573 | void __sanitizer_syscall_pre_impl___select50(long long nd, long long in,
4574 |                                              long long ou, long long ex,
4575 |                                              long long tv);
4576 | void __sanitizer_syscall_post_impl___select50(long long res, long long nd,
4577 |                                               long long in, long long ou,
4578 |                                               long long ex, long long tv);
4579 | void __sanitizer_syscall_pre_impl___gettimeofday50(long long tp, long long tzp);
4580 | void __sanitizer_syscall_post_impl___gettimeofday50(long long res, long long tp,
```
- **Line 4571 / 第 4571 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4572 / 第 4572 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4573 / 第 4573 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4574 / 第 4574 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4575 / 第 4575 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4576 / 第 4576 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4577 / 第 4577 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4578 / 第 4578 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4579 / 第 4579 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___gettimeofday50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___gettimeofday50`。
- **Line 4580 / 第 4580 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4581-4590 / 第 4581-4590 行
```cpp
4581 |                                                     long long tzp);
4582 | void __sanitizer_syscall_pre_impl___settimeofday50(long long tv, long long tzp);
4583 | void __sanitizer_syscall_post_impl___settimeofday50(long long res, long long tv,
4584 |                                                     long long tzp);
4585 | void __sanitizer_syscall_pre_impl___utimes50(long long path, long long tptr);
4586 | void __sanitizer_syscall_post_impl___utimes50(long long res, long long path,
4587 |                                               long long tptr);
4588 | void __sanitizer_syscall_pre_impl___adjtime50(long long delta,
4589 |                                               long long olddelta);
4590 | void __sanitizer_syscall_post_impl___adjtime50(long long res, long long delta,
```
- **Line 4581 / 第 4581 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4582 / 第 4582 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___settimeofday50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___settimeofday50`。
- **Line 4583 / 第 4583 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4584 / 第 4584 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4585 / 第 4585 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___utimes50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___utimes50`。
- **Line 4586 / 第 4586 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4587 / 第 4587 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4588 / 第 4588 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4589 / 第 4589 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4590 / 第 4590 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4591-4600 / 第 4591-4600 行
```cpp
4591 |                                                long long olddelta);
4592 | void __sanitizer_syscall_pre_impl___lfs_segwait50(long long fsidp,
4593 |                                                   long long tv);
4594 | void __sanitizer_syscall_post_impl___lfs_segwait50(long long res,
4595 |                                                    long long fsidp,
4596 |                                                    long long tv);
4597 | void __sanitizer_syscall_pre_impl___futimes50(long long fd, long long tptr);
4598 | void __sanitizer_syscall_post_impl___futimes50(long long res, long long fd,
4599 |                                                long long tptr);
4600 | void __sanitizer_syscall_pre_impl___lutimes50(long long path, long long tptr);
```
- **Line 4591 / 第 4591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4592 / 第 4592 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4593 / 第 4593 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4594 / 第 4594 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4595 / 第 4595 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4596 / 第 4596 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4597 / 第 4597 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___futimes50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___futimes50`。
- **Line 4598 / 第 4598 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4599 / 第 4599 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4600 / 第 4600 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___lutimes50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___lutimes50`。

### Lines 4601-4610 / 第 4601-4610 行
```cpp
4601 | void __sanitizer_syscall_post_impl___lutimes50(long long res, long long path,
4602 |                                                long long tptr);
4603 | void __sanitizer_syscall_pre_impl___setitimer50(long long which, long long itv,
4604 |                                                 long long oitv);
4605 | void __sanitizer_syscall_post_impl___setitimer50(long long res, long long which,
4606 |                                                  long long itv, long long oitv);
4607 | void __sanitizer_syscall_pre_impl___getitimer50(long long which, long long itv);
4608 | void __sanitizer_syscall_post_impl___getitimer50(long long res, long long which,
4609 |                                                  long long itv);
4610 | void __sanitizer_syscall_pre_impl___clock_gettime50(long long clock_id,
```
- **Line 4601 / 第 4601 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4602 / 第 4602 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4603 / 第 4603 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4604 / 第 4604 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4605 / 第 4605 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4606 / 第 4606 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4607 / 第 4607 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___getitimer50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___getitimer50`。
- **Line 4608 / 第 4608 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4609 / 第 4609 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4610 / 第 4610 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4611-4620 / 第 4611-4620 行
```cpp
4611 |                                                     long long tp);
4612 | void __sanitizer_syscall_post_impl___clock_gettime50(long long res,
4613 |                                                      long long clock_id,
4614 |                                                      long long tp);
4615 | void __sanitizer_syscall_pre_impl___clock_settime50(long long clock_id,
4616 |                                                     long long tp);
4617 | void __sanitizer_syscall_post_impl___clock_settime50(long long res,
4618 |                                                      long long clock_id,
4619 |                                                      long long tp);
4620 | void __sanitizer_syscall_pre_impl___clock_getres50(long long clock_id,
```
- **Line 4611 / 第 4611 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4612 / 第 4612 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4613 / 第 4613 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4614 / 第 4614 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4615 / 第 4615 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4616 / 第 4616 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4617 / 第 4617 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4618 / 第 4618 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4619 / 第 4619 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4620 / 第 4620 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4621-4630 / 第 4621-4630 行
```cpp
4621 |                                                    long long tp);
4622 | void __sanitizer_syscall_post_impl___clock_getres50(long long res,
4623 |                                                     long long clock_id,
4624 |                                                     long long tp);
4625 | void __sanitizer_syscall_pre_impl___nanosleep50(long long rqtp, long long rmtp);
4626 | void __sanitizer_syscall_post_impl___nanosleep50(long long res, long long rqtp,
4627 |                                                  long long rmtp);
4628 | void __sanitizer_syscall_pre_impl_____sigtimedwait50(long long set,
4629 |                                                      long long info,
4630 |                                                      long long timeout);
```
- **Line 4621 / 第 4621 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4622 / 第 4622 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4623 / 第 4623 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4624 / 第 4624 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4625 / 第 4625 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___nanosleep50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___nanosleep50`。
- **Line 4626 / 第 4626 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4627 / 第 4627 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4628 / 第 4628 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4629 / 第 4629 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4630 / 第 4630 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4631-4640 / 第 4631-4640 行
```cpp
4631 | void __sanitizer_syscall_post_impl_____sigtimedwait50(long long res,
4632 |                                                       long long set,
4633 |                                                       long long info,
4634 |                                                       long long timeout);
4635 | void __sanitizer_syscall_pre_impl___mq_timedsend50(long long mqdes,
4636 |                                                    long long msg_ptr,
4637 |                                                    long long msg_len,
4638 |                                                    long long msg_prio,
4639 |                                                    long long abs_timeout);
4640 | void __sanitizer_syscall_post_impl___mq_timedsend50(
```
- **Line 4631 / 第 4631 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4632 / 第 4632 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4633 / 第 4633 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4634 / 第 4634 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4635 / 第 4635 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4636 / 第 4636 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4637 / 第 4637 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4638 / 第 4638 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4639 / 第 4639 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4640 / 第 4640 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4641-4650 / 第 4641-4650 行
```cpp
4641 |     long long res, long long mqdes, long long msg_ptr, long long msg_len,
4642 |     long long msg_prio, long long abs_timeout);
4643 | void __sanitizer_syscall_pre_impl___mq_timedreceive50(long long mqdes,
4644 |                                                       long long msg_ptr,
4645 |                                                       long long msg_len,
4646 |                                                       long long msg_prio,
4647 |                                                       long long abs_timeout);
4648 | void __sanitizer_syscall_post_impl___mq_timedreceive50(
4649 |     long long res, long long mqdes, long long msg_ptr, long long msg_len,
4650 |     long long msg_prio, long long abs_timeout);
```
- **Line 4641 / 第 4641 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4642 / 第 4642 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4643 / 第 4643 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4644 / 第 4644 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4645 / 第 4645 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4646 / 第 4646 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4647 / 第 4647 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4648 / 第 4648 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4649 / 第 4649 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4650 / 第 4650 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4651-4660 / 第 4651-4660 行
```cpp
4651 | void __sanitizer_syscall_pre_impl_compat_60__lwp_park(long long ts,
4652 |                                                       long long unpark,
4653 |                                                       long long hint,
4654 |                                                       long long unparkhint);
4655 | void __sanitizer_syscall_post_impl_compat_60__lwp_park(long long res,
4656 |                                                        long long ts,
4657 |                                                        long long unpark,
4658 |                                                        long long hint,
4659 |                                                        long long unparkhint);
4660 | void __sanitizer_syscall_pre_impl___kevent50(long long fd, long long changelist,
```
- **Line 4651 / 第 4651 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4652 / 第 4652 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4653 / 第 4653 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4654 / 第 4654 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4655 / 第 4655 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4656 / 第 4656 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4657 / 第 4657 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4658 / 第 4658 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4659 / 第 4659 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4660 / 第 4660 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4661-4670 / 第 4661-4670 行
```cpp
4661 |                                              long long nchanges,
4662 |                                              long long eventlist,
4663 |                                              long long nevents,
4664 |                                              long long timeout);
4665 | void __sanitizer_syscall_post_impl___kevent50(
4666 |     long long res, long long fd, long long changelist, long long nchanges,
4667 |     long long eventlist, long long nevents, long long timeout);
4668 | void __sanitizer_syscall_pre_impl___pselect50(long long nd, long long in,
4669 |                                               long long ou, long long ex,
4670 |                                               long long ts, long long mask);
```
- **Line 4661 / 第 4661 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4662 / 第 4662 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4663 / 第 4663 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4664 / 第 4664 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4665 / 第 4665 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4666 / 第 4666 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4667 / 第 4667 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4668 / 第 4668 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4669 / 第 4669 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4670 / 第 4670 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4671-4680 / 第 4671-4680 行
```cpp
4671 | void __sanitizer_syscall_post_impl___pselect50(long long res, long long nd,
4672 |                                                long long in, long long ou,
4673 |                                                long long ex, long long ts,
4674 |                                                long long mask);
4675 | void __sanitizer_syscall_pre_impl___pollts50(long long fds, long long nfds,
4676 |                                              long long ts, long long mask);
4677 | void __sanitizer_syscall_post_impl___pollts50(long long res, long long fds,
4678 |                                               long long nfds, long long ts,
4679 |                                               long long mask);
4680 | void __sanitizer_syscall_pre_impl___aio_suspend50(long long list,
```
- **Line 4671 / 第 4671 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4672 / 第 4672 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4673 / 第 4673 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4674 / 第 4674 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4675 / 第 4675 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4676 / 第 4676 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4677 / 第 4677 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4678 / 第 4678 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4679 / 第 4679 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4680 / 第 4680 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4681-4690 / 第 4681-4690 行
```cpp
4681 |                                                   long long nent,
4682 |                                                   long long timeout);
4683 | void __sanitizer_syscall_post_impl___aio_suspend50(long long res,
4684 |                                                    long long list,
4685 |                                                    long long nent,
4686 |                                                    long long timeout);
4687 | void __sanitizer_syscall_pre_impl___stat50(long long path, long long ub);
4688 | void __sanitizer_syscall_post_impl___stat50(long long res, long long path,
4689 |                                             long long ub);
4690 | void __sanitizer_syscall_pre_impl___fstat50(long long fd, long long sb);
```
- **Line 4681 / 第 4681 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4682 / 第 4682 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4683 / 第 4683 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4684 / 第 4684 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4685 / 第 4685 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4686 / 第 4686 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4687 / 第 4687 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___stat50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___stat50`。
- **Line 4688 / 第 4688 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4689 / 第 4689 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4690 / 第 4690 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___fstat50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___fstat50`。

### Lines 4691-4700 / 第 4691-4700 行
```cpp
4691 | void __sanitizer_syscall_post_impl___fstat50(long long res, long long fd,
4692 |                                              long long sb);
4693 | void __sanitizer_syscall_pre_impl___lstat50(long long path, long long ub);
4694 | void __sanitizer_syscall_post_impl___lstat50(long long res, long long path,
4695 |                                              long long ub);
4696 | void __sanitizer_syscall_pre_impl_____semctl50(long long semid,
4697 |                                                long long semnum, long long cmd,
4698 |                                                long long arg);
4699 | void __sanitizer_syscall_post_impl_____semctl50(long long res, long long semid,
4700 |                                                 long long semnum, long long cmd,
```
- **Line 4691 / 第 4691 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4692 / 第 4692 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4693 / 第 4693 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___lstat50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___lstat50`。
- **Line 4694 / 第 4694 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4695 / 第 4695 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4696 / 第 4696 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4697 / 第 4697 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4698 / 第 4698 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4699 / 第 4699 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4700 / 第 4700 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4701-4710 / 第 4701-4710 行
```cpp
4701 |                                                 long long arg);
4702 | void __sanitizer_syscall_pre_impl___shmctl50(long long shmid, long long cmd,
4703 |                                              long long buf);
4704 | void __sanitizer_syscall_post_impl___shmctl50(long long res, long long shmid,
4705 |                                               long long cmd, long long buf);
4706 | void __sanitizer_syscall_pre_impl___msgctl50(long long msqid, long long cmd,
4707 |                                              long long buf);
4708 | void __sanitizer_syscall_post_impl___msgctl50(long long res, long long msqid,
4709 |                                               long long cmd, long long buf);
4710 | void __sanitizer_syscall_pre_impl___getrusage50(long long who,
```
- **Line 4701 / 第 4701 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4702 / 第 4702 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4703 / 第 4703 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4704 / 第 4704 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4705 / 第 4705 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4706 / 第 4706 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4707 / 第 4707 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4708 / 第 4708 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4709 / 第 4709 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4710 / 第 4710 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4711-4720 / 第 4711-4720 行
```cpp
4711 |                                                 long long rusage);
4712 | void __sanitizer_syscall_post_impl___getrusage50(long long res, long long who,
4713 |                                                  long long rusage);
4714 | void __sanitizer_syscall_pre_impl___timer_settime50(long long timerid,
4715 |                                                     long long flags,
4716 |                                                     long long value,
4717 |                                                     long long ovalue);
4718 | void __sanitizer_syscall_post_impl___timer_settime50(long long res,
4719 |                                                      long long timerid,
4720 |                                                      long long flags,
```
- **Line 4711 / 第 4711 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4712 / 第 4712 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4713 / 第 4713 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4714 / 第 4714 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4715 / 第 4715 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4716 / 第 4716 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4717 / 第 4717 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4718 / 第 4718 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4719 / 第 4719 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4720 / 第 4720 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4721-4730 / 第 4721-4730 行
```cpp
4721 |                                                      long long value,
4722 |                                                      long long ovalue);
4723 | void __sanitizer_syscall_pre_impl___timer_gettime50(long long timerid,
4724 |                                                     long long value);
4725 | void __sanitizer_syscall_post_impl___timer_gettime50(long long res,
4726 |                                                      long long timerid,
4727 |                                                      long long value);
4728 | #if defined(NTP) || !defined(_KERNEL_OPT)
4729 | void __sanitizer_syscall_pre_impl___ntp_gettime50(long long ntvp);
4730 | void __sanitizer_syscall_post_impl___ntp_gettime50(long long res,
```
- **Line 4721 / 第 4721 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4722 / 第 4722 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4723 / 第 4723 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4724 / 第 4724 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4725 / 第 4725 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4726 / 第 4726 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4727 / 第 4727 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4728 / 第 4728 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 4729 / 第 4729 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___ntp_gettime50`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___ntp_gettime50`。
- **Line 4730 / 第 4730 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4731-4740 / 第 4731-4740 行
```cpp
4731 |                                                    long long ntvp);
4732 | #else
4733 | /* syscall 448 has been skipped */
4734 | #endif
4735 | void __sanitizer_syscall_pre_impl___wait450(long long pid, long long status,
4736 |                                             long long options,
4737 |                                             long long rusage);
4738 | void __sanitizer_syscall_post_impl___wait450(long long res, long long pid,
4739 |                                              long long status,
4740 |                                              long long options,
```
- **Line 4731 / 第 4731 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4732 / 第 4732 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 4733 / 第 4733 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4734 / 第 4734 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 4735 / 第 4735 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4736 / 第 4736 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4737 / 第 4737 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4738 / 第 4738 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4739 / 第 4739 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4740 / 第 4740 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4741-4750 / 第 4741-4750 行
```cpp
4741 |                                              long long rusage);
4742 | void __sanitizer_syscall_pre_impl___mknod50(long long path, long long mode,
4743 |                                             long long dev);
4744 | void __sanitizer_syscall_post_impl___mknod50(long long res, long long path,
4745 |                                              long long mode, long long dev);
4746 | void __sanitizer_syscall_pre_impl___fhstat50(long long fhp, long long fh_size,
4747 |                                              long long sb);
4748 | void __sanitizer_syscall_post_impl___fhstat50(long long res, long long fhp,
4749 |                                               long long fh_size, long long sb);
4750 | /* syscall 452 has been skipped */
```
- **Line 4741 / 第 4741 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4742 / 第 4742 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4743 / 第 4743 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4744 / 第 4744 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4745 / 第 4745 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4746 / 第 4746 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4747 / 第 4747 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4748 / 第 4748 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4749 / 第 4749 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4750 / 第 4750 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 4751-4760 / 第 4751-4760 行
```cpp
4751 | void __sanitizer_syscall_pre_impl_pipe2(long long fildes, long long flags);
4752 | void __sanitizer_syscall_post_impl_pipe2(long long res, long long fildes,
4753 |                                          long long flags);
4754 | void __sanitizer_syscall_pre_impl_dup3(long long from, long long to,
4755 |                                        long long flags);
4756 | void __sanitizer_syscall_post_impl_dup3(long long res, long long from,
4757 |                                         long long to, long long flags);
4758 | void __sanitizer_syscall_pre_impl_kqueue1(long long flags);
4759 | void __sanitizer_syscall_post_impl_kqueue1(long long res, long long flags);
4760 | void __sanitizer_syscall_pre_impl_paccept(long long s, long long name,
```
- **Line 4751 / 第 4751 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_pipe2`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_pipe2`。
- **Line 4752 / 第 4752 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4753 / 第 4753 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4754 / 第 4754 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4755 / 第 4755 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4756 / 第 4756 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4757 / 第 4757 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4758 / 第 4758 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_kqueue1`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_kqueue1`。
- **Line 4759 / 第 4759 行**: EN: Declares function or method `__sanitizer_syscall_post_impl_kqueue1`. CN: 声明函数或方法 `__sanitizer_syscall_post_impl_kqueue1`。
- **Line 4760 / 第 4760 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4761-4770 / 第 4761-4770 行
```cpp
4761 |                                           long long anamelen, long long mask,
4762 |                                           long long flags);
4763 | void __sanitizer_syscall_post_impl_paccept(long long res, long long s,
4764 |                                            long long name, long long anamelen,
4765 |                                            long long mask, long long flags);
4766 | void __sanitizer_syscall_pre_impl_linkat(long long fd1, long long name1,
4767 |                                          long long fd2, long long name2,
4768 |                                          long long flags);
4769 | void __sanitizer_syscall_post_impl_linkat(long long res, long long fd1,
4770 |                                           long long name1, long long fd2,
```
- **Line 4761 / 第 4761 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4762 / 第 4762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4763 / 第 4763 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4764 / 第 4764 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4765 / 第 4765 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4766 / 第 4766 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4767 / 第 4767 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4768 / 第 4768 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4769 / 第 4769 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4770 / 第 4770 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4771-4780 / 第 4771-4780 行
```cpp
4771 |                                           long long name2, long long flags);
4772 | void __sanitizer_syscall_pre_impl_renameat(long long fromfd, long long from,
4773 |                                            long long tofd, long long to);
4774 | void __sanitizer_syscall_post_impl_renameat(long long res, long long fromfd,
4775 |                                             long long from, long long tofd,
4776 |                                             long long to);
4777 | void __sanitizer_syscall_pre_impl_mkfifoat(long long fd, long long path,
4778 |                                            long long mode);
4779 | void __sanitizer_syscall_post_impl_mkfifoat(long long res, long long fd,
4780 |                                             long long path, long long mode);
```
- **Line 4771 / 第 4771 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4772 / 第 4772 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4773 / 第 4773 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4774 / 第 4774 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4775 / 第 4775 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4776 / 第 4776 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4777 / 第 4777 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4778 / 第 4778 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4779 / 第 4779 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4780 / 第 4780 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4781-4790 / 第 4781-4790 行
```cpp
4781 | void __sanitizer_syscall_pre_impl_mknodat(long long fd, long long path,
4782 |                                           long long mode, long long PAD,
4783 |                                           long long dev);
4784 | void __sanitizer_syscall_post_impl_mknodat(long long res, long long fd,
4785 |                                            long long path, long long mode,
4786 |                                            long long PAD, long long dev);
4787 | void __sanitizer_syscall_pre_impl_mkdirat(long long fd, long long path,
4788 |                                           long long mode);
4789 | void __sanitizer_syscall_post_impl_mkdirat(long long res, long long fd,
4790 |                                            long long path, long long mode);
```
- **Line 4781 / 第 4781 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4782 / 第 4782 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4783 / 第 4783 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4784 / 第 4784 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4785 / 第 4785 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4786 / 第 4786 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4787 / 第 4787 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4788 / 第 4788 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4789 / 第 4789 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4790 / 第 4790 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4791-4800 / 第 4791-4800 行
```cpp
4791 | void __sanitizer_syscall_pre_impl_faccessat(long long fd, long long path,
4792 |                                             long long amode, long long flag);
4793 | void __sanitizer_syscall_post_impl_faccessat(long long res, long long fd,
4794 |                                              long long path, long long amode,
4795 |                                              long long flag);
4796 | void __sanitizer_syscall_pre_impl_fchmodat(long long fd, long long path,
4797 |                                            long long mode, long long flag);
4798 | void __sanitizer_syscall_post_impl_fchmodat(long long res, long long fd,
4799 |                                             long long path, long long mode,
4800 |                                             long long flag);
```
- **Line 4791 / 第 4791 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4792 / 第 4792 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4793 / 第 4793 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4794 / 第 4794 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4795 / 第 4795 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4796 / 第 4796 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4797 / 第 4797 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4798 / 第 4798 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4799 / 第 4799 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4800 / 第 4800 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4801-4810 / 第 4801-4810 行
```cpp
4801 | void __sanitizer_syscall_pre_impl_fchownat(long long fd, long long path,
4802 |                                            long long owner, long long group,
4803 |                                            long long flag);
4804 | void __sanitizer_syscall_post_impl_fchownat(long long res, long long fd,
4805 |                                             long long path, long long owner,
4806 |                                             long long group, long long flag);
4807 | void __sanitizer_syscall_pre_impl_fexecve(long long fd, long long argp,
4808 |                                           long long envp);
4809 | void __sanitizer_syscall_post_impl_fexecve(long long res, long long fd,
4810 |                                            long long argp, long long envp);
```
- **Line 4801 / 第 4801 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4802 / 第 4802 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4803 / 第 4803 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4804 / 第 4804 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4805 / 第 4805 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4806 / 第 4806 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4807 / 第 4807 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4808 / 第 4808 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4809 / 第 4809 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4810 / 第 4810 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4811-4820 / 第 4811-4820 行
```cpp
4811 | void __sanitizer_syscall_pre_impl_fstatat(long long fd, long long path,
4812 |                                           long long buf, long long flag);
4813 | void __sanitizer_syscall_post_impl_fstatat(long long res, long long fd,
4814 |                                            long long path, long long buf,
4815 |                                            long long flag);
4816 | void __sanitizer_syscall_pre_impl_utimensat(long long fd, long long path,
4817 |                                             long long tptr, long long flag);
4818 | void __sanitizer_syscall_post_impl_utimensat(long long res, long long fd,
4819 |                                              long long path, long long tptr,
4820 |                                              long long flag);
```
- **Line 4811 / 第 4811 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4812 / 第 4812 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4813 / 第 4813 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4814 / 第 4814 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4815 / 第 4815 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4816 / 第 4816 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4817 / 第 4817 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4818 / 第 4818 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4819 / 第 4819 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4820 / 第 4820 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4821-4830 / 第 4821-4830 行
```cpp
4821 | void __sanitizer_syscall_pre_impl_openat(long long fd, long long path,
4822 |                                          long long oflags, long long mode);
4823 | void __sanitizer_syscall_post_impl_openat(long long res, long long fd,
4824 |                                           long long path, long long oflags,
4825 |                                           long long mode);
4826 | void __sanitizer_syscall_pre_impl_readlinkat(long long fd, long long path,
4827 |                                              long long buf, long long bufsize);
4828 | void __sanitizer_syscall_post_impl_readlinkat(long long res, long long fd,
4829 |                                               long long path, long long buf,
4830 |                                               long long bufsize);
```
- **Line 4821 / 第 4821 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4822 / 第 4822 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4823 / 第 4823 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4824 / 第 4824 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4825 / 第 4825 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4826 / 第 4826 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4827 / 第 4827 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4828 / 第 4828 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4829 / 第 4829 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4830 / 第 4830 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4831-4840 / 第 4831-4840 行
```cpp
4831 | void __sanitizer_syscall_pre_impl_symlinkat(long long path1, long long fd,
4832 |                                             long long path2);
4833 | void __sanitizer_syscall_post_impl_symlinkat(long long res, long long path1,
4834 |                                              long long fd, long long path2);
4835 | void __sanitizer_syscall_pre_impl_unlinkat(long long fd, long long path,
4836 |                                            long long flag);
4837 | void __sanitizer_syscall_post_impl_unlinkat(long long res, long long fd,
4838 |                                             long long path, long long flag);
4839 | void __sanitizer_syscall_pre_impl_futimens(long long fd, long long tptr);
4840 | void __sanitizer_syscall_post_impl_futimens(long long res, long long fd,
```
- **Line 4831 / 第 4831 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4832 / 第 4832 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4833 / 第 4833 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4834 / 第 4834 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4835 / 第 4835 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4836 / 第 4836 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4837 / 第 4837 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4838 / 第 4838 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4839 / 第 4839 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_futimens`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_futimens`。
- **Line 4840 / 第 4840 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4841-4850 / 第 4841-4850 行
```cpp
4841 |                                             long long tptr);
4842 | void __sanitizer_syscall_pre_impl___quotactl(long long path, long long args);
4843 | void __sanitizer_syscall_post_impl___quotactl(long long res, long long path,
4844 |                                               long long args);
4845 | void __sanitizer_syscall_pre_impl_posix_spawn(long long pid, long long path,
4846 |                                               long long file_actions,
4847 |                                               long long attrp, long long argv,
4848 |                                               long long envp);
4849 | void __sanitizer_syscall_post_impl_posix_spawn(long long res, long long pid,
4850 |                                                long long path,
```
- **Line 4841 / 第 4841 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4842 / 第 4842 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl___quotactl`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl___quotactl`。
- **Line 4843 / 第 4843 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4844 / 第 4844 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4845 / 第 4845 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4846 / 第 4846 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4847 / 第 4847 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4848 / 第 4848 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4849 / 第 4849 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4850 / 第 4850 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4851-4860 / 第 4851-4860 行
```cpp
4851 |                                                long long file_actions,
4852 |                                                long long attrp, long long argv,
4853 |                                                long long envp);
4854 | void __sanitizer_syscall_pre_impl_recvmmsg(long long s, long long mmsg,
4855 |                                            long long vlen, long long flags,
4856 |                                            long long timeout);
4857 | void __sanitizer_syscall_post_impl_recvmmsg(long long res, long long s,
4858 |                                             long long mmsg, long long vlen,
4859 |                                             long long flags, long long timeout);
4860 | void __sanitizer_syscall_pre_impl_sendmmsg(long long s, long long mmsg,
```
- **Line 4851 / 第 4851 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4852 / 第 4852 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4853 / 第 4853 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4854 / 第 4854 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4855 / 第 4855 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4856 / 第 4856 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4857 / 第 4857 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4858 / 第 4858 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4859 / 第 4859 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4860 / 第 4860 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4861-4870 / 第 4861-4870 行
```cpp
4861 |                                            long long vlen, long long flags);
4862 | void __sanitizer_syscall_post_impl_sendmmsg(long long res, long long s,
4863 |                                             long long mmsg, long long vlen,
4864 |                                             long long flags);
4865 | void __sanitizer_syscall_pre_impl_clock_nanosleep(long long clock_id,
4866 |                                                   long long flags,
4867 |                                                   long long rqtp,
4868 |                                                   long long rmtp);
4869 | void __sanitizer_syscall_post_impl_clock_nanosleep(long long res,
4870 |                                                    long long clock_id,
```
- **Line 4861 / 第 4861 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4862 / 第 4862 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4863 / 第 4863 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4864 / 第 4864 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4865 / 第 4865 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4866 / 第 4866 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4867 / 第 4867 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4868 / 第 4868 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4869 / 第 4869 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4870 / 第 4870 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4871-4880 / 第 4871-4880 行
```cpp
4871 |                                                    long long flags,
4872 |                                                    long long rqtp,
4873 |                                                    long long rmtp);
4874 | void __sanitizer_syscall_pre_impl____lwp_park60(long long clock_id,
4875 |                                                 long long flags, long long ts,
4876 |                                                 long long unpark,
4877 |                                                 long long hint,
4878 |                                                 long long unparkhint);
4879 | void __sanitizer_syscall_post_impl____lwp_park60(
4880 |     long long res, long long clock_id, long long flags, long long ts,
```
- **Line 4871 / 第 4871 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4872 / 第 4872 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4873 / 第 4873 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4874 / 第 4874 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4875 / 第 4875 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4876 / 第 4876 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4877 / 第 4877 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4878 / 第 4878 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4879 / 第 4879 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4880 / 第 4880 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4881-4890 / 第 4881-4890 行
```cpp
4881 |     long long unpark, long long hint, long long unparkhint);
4882 | void __sanitizer_syscall_pre_impl_posix_fallocate(long long fd, long long PAD,
4883 |                                                   long long pos, long long len);
4884 | void __sanitizer_syscall_post_impl_posix_fallocate(long long res, long long fd,
4885 |                                                    long long PAD, long long pos,
4886 |                                                    long long len);
4887 | void __sanitizer_syscall_pre_impl_fdiscard(long long fd, long long PAD,
4888 |                                            long long pos, long long len);
4889 | void __sanitizer_syscall_post_impl_fdiscard(long long res, long long fd,
4890 |                                             long long PAD, long long pos,
```
- **Line 4881 / 第 4881 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4882 / 第 4882 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4883 / 第 4883 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4884 / 第 4884 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4885 / 第 4885 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4886 / 第 4886 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4887 / 第 4887 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4888 / 第 4888 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4889 / 第 4889 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4890 / 第 4890 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4891-4900 / 第 4891-4900 行
```cpp
4891 |                                             long long len);
4892 | void __sanitizer_syscall_pre_impl_wait6(long long idtype, long long id,
4893 |                                         long long status, long long options,
4894 |                                         long long wru, long long info);
4895 | void __sanitizer_syscall_post_impl_wait6(long long res, long long idtype,
4896 |                                          long long id, long long status,
4897 |                                          long long options, long long wru,
4898 |                                          long long info);
4899 | void __sanitizer_syscall_pre_impl_clock_getcpuclockid2(long long idtype,
4900 |                                                        long long id,
```
- **Line 4891 / 第 4891 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4892 / 第 4892 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4893 / 第 4893 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4894 / 第 4894 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4895 / 第 4895 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4896 / 第 4896 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4897 / 第 4897 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4898 / 第 4898 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4899 / 第 4899 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4900 / 第 4900 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4901-4910 / 第 4901-4910 行
```cpp
4901 |                                                        long long clock_id);
4902 | void __sanitizer_syscall_post_impl_clock_getcpuclockid2(long long res,
4903 |                                                         long long idtype,
4904 |                                                         long long id,
4905 |                                                         long long clock_id);
4906 | void __sanitizer_syscall_pre_impl___getvfsstat90(long long buf,
4907 |                                                  long long bufsize,
4908 |                                                  long long flags);
4909 | void __sanitizer_syscall_post_impl___getvfsstat90(long long res, long long buf,
4910 |                                                   long long bufsize,
```
- **Line 4901 / 第 4901 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4902 / 第 4902 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4903 / 第 4903 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4904 / 第 4904 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4905 / 第 4905 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4906 / 第 4906 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4907 / 第 4907 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4908 / 第 4908 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4909 / 第 4909 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4910 / 第 4910 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 4911-4920 / 第 4911-4920 行
```cpp
4911 |                                                   long long flags);
4912 | void __sanitizer_syscall_pre_impl___statvfs190(long long path, long long buf,
4913 |                                                long long flags);
4914 | void __sanitizer_syscall_post_impl___statvfs190(long long res, long long path,
4915 |                                                 long long buf, long long flags);
4916 | void __sanitizer_syscall_pre_impl___fstatvfs190(long long fd, long long buf,
4917 |                                                 long long flags);
4918 | void __sanitizer_syscall_post_impl___fstatvfs190(long long res, long long fd,
4919 |                                                  long long buf,
4920 |                                                  long long flags);
```
- **Line 4911 / 第 4911 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4912 / 第 4912 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4913 / 第 4913 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4914 / 第 4914 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4915 / 第 4915 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4916 / 第 4916 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4917 / 第 4917 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4918 / 第 4918 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4919 / 第 4919 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4920 / 第 4920 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4921-4930 / 第 4921-4930 行
```cpp
4921 | void __sanitizer_syscall_pre_impl___fhstatvfs190(long long fhp,
4922 |                                                  long long fh_size,
4923 |                                                  long long buf,
4924 |                                                  long long flags);
4925 | void __sanitizer_syscall_post_impl___fhstatvfs190(long long res, long long fhp,
4926 |                                                   long long fh_size,
4927 |                                                   long long buf,
4928 |                                                   long long flags);
4929 | void __sanitizer_syscall_pre_impl___acl_get_link(long long path, long long type,
4930 |                                                  long long aclp);
```
- **Line 4921 / 第 4921 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4922 / 第 4922 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4923 / 第 4923 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4924 / 第 4924 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4925 / 第 4925 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4926 / 第 4926 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4927 / 第 4927 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4928 / 第 4928 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4929 / 第 4929 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4930 / 第 4930 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4931-4940 / 第 4931-4940 行
```cpp
4931 | void __sanitizer_syscall_post_impl___acl_get_link(long long res, long long path,
4932 |                                                   long long type,
4933 |                                                   long long aclp);
4934 | void __sanitizer_syscall_pre_impl___acl_set_link(long long path, long long type,
4935 |                                                  long long aclp);
4936 | void __sanitizer_syscall_post_impl___acl_set_link(long long res, long long path,
4937 |                                                   long long type,
4938 |                                                   long long aclp);
4939 | void __sanitizer_syscall_pre_impl___acl_delete_link(long long path,
4940 |                                                     long long type);
```
- **Line 4931 / 第 4931 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4932 / 第 4932 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4933 / 第 4933 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4934 / 第 4934 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4935 / 第 4935 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4936 / 第 4936 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4937 / 第 4937 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4938 / 第 4938 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4939 / 第 4939 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4940 / 第 4940 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4941-4950 / 第 4941-4950 行
```cpp
4941 | void __sanitizer_syscall_post_impl___acl_delete_link(long long res,
4942 |                                                      long long path,
4943 |                                                      long long type);
4944 | void __sanitizer_syscall_pre_impl___acl_aclcheck_link(long long path,
4945 |                                                       long long type,
4946 |                                                       long long aclp);
4947 | void __sanitizer_syscall_post_impl___acl_aclcheck_link(long long res,
4948 |                                                        long long path,
4949 |                                                        long long type,
4950 |                                                        long long aclp);
```
- **Line 4941 / 第 4941 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4942 / 第 4942 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4943 / 第 4943 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4944 / 第 4944 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4945 / 第 4945 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4946 / 第 4946 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4947 / 第 4947 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4948 / 第 4948 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4949 / 第 4949 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4950 / 第 4950 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4951-4960 / 第 4951-4960 行
```cpp
4951 | void __sanitizer_syscall_pre_impl___acl_get_file(long long path, long long type,
4952 |                                                  long long aclp);
4953 | void __sanitizer_syscall_post_impl___acl_get_file(long long res, long long path,
4954 |                                                   long long type,
4955 |                                                   long long aclp);
4956 | void __sanitizer_syscall_pre_impl___acl_set_file(long long path, long long type,
4957 |                                                  long long aclp);
4958 | void __sanitizer_syscall_post_impl___acl_set_file(long long res, long long path,
4959 |                                                   long long type,
4960 |                                                   long long aclp);
```
- **Line 4951 / 第 4951 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4952 / 第 4952 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4953 / 第 4953 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4954 / 第 4954 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4955 / 第 4955 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4956 / 第 4956 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4957 / 第 4957 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4958 / 第 4958 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4959 / 第 4959 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4960 / 第 4960 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4961-4970 / 第 4961-4970 行
```cpp
4961 | void __sanitizer_syscall_pre_impl___acl_get_fd(long long filedes,
4962 |                                                long long type, long long aclp);
4963 | void __sanitizer_syscall_post_impl___acl_get_fd(long long res,
4964 |                                                 long long filedes,
4965 |                                                 long long type, long long aclp);
4966 | void __sanitizer_syscall_pre_impl___acl_set_fd(long long filedes,
4967 |                                                long long type, long long aclp);
4968 | void __sanitizer_syscall_post_impl___acl_set_fd(long long res,
4969 |                                                 long long filedes,
4970 |                                                 long long type, long long aclp);
```
- **Line 4961 / 第 4961 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4962 / 第 4962 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4963 / 第 4963 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4964 / 第 4964 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4965 / 第 4965 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4966 / 第 4966 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4967 / 第 4967 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4968 / 第 4968 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4969 / 第 4969 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4970 / 第 4970 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4971-4980 / 第 4971-4980 行
```cpp
4971 | void __sanitizer_syscall_pre_impl___acl_delete_file(long long path,
4972 |                                                     long long type);
4973 | void __sanitizer_syscall_post_impl___acl_delete_file(long long res,
4974 |                                                      long long path,
4975 |                                                      long long type);
4976 | void __sanitizer_syscall_pre_impl___acl_delete_fd(long long filedes,
4977 |                                                   long long type);
4978 | void __sanitizer_syscall_post_impl___acl_delete_fd(long long res,
4979 |                                                    long long filedes,
4980 |                                                    long long type);
```
- **Line 4971 / 第 4971 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4972 / 第 4972 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4973 / 第 4973 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4974 / 第 4974 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4975 / 第 4975 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4976 / 第 4976 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4977 / 第 4977 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4978 / 第 4978 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4979 / 第 4979 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4980 / 第 4980 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4981-4990 / 第 4981-4990 行
```cpp
4981 | void __sanitizer_syscall_pre_impl___acl_aclcheck_file(long long path,
4982 |                                                       long long type,
4983 |                                                       long long aclp);
4984 | void __sanitizer_syscall_post_impl___acl_aclcheck_file(long long res,
4985 |                                                        long long path,
4986 |                                                        long long type,
4987 |                                                        long long aclp);
4988 | void __sanitizer_syscall_pre_impl___acl_aclcheck_fd(long long filedes,
4989 |                                                     long long type,
4990 |                                                     long long aclp);
```
- **Line 4981 / 第 4981 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4982 / 第 4982 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4983 / 第 4983 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4984 / 第 4984 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4985 / 第 4985 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4986 / 第 4986 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4987 / 第 4987 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4988 / 第 4988 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4989 / 第 4989 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4990 / 第 4990 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 4991-5000 / 第 4991-5000 行
```cpp
4991 | void __sanitizer_syscall_post_impl___acl_aclcheck_fd(long long res,
4992 |                                                      long long filedes,
4993 |                                                      long long type,
4994 |                                                      long long aclp);
4995 | void __sanitizer_syscall_pre_impl_lpathconf(long long path, long long name);
4996 | void __sanitizer_syscall_post_impl_lpathconf(long long res, long long path,
4997 |                                              long long name);
4998 | 
4999 | #ifdef __cplusplus
5000 | } // extern "C"
```
- **Line 4991 / 第 4991 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4992 / 第 4992 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4993 / 第 4993 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4994 / 第 4994 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4995 / 第 4995 行**: EN: Declares function or method `__sanitizer_syscall_pre_impl_lpathconf`. CN: 声明函数或方法 `__sanitizer_syscall_pre_impl_lpathconf`。
- **Line 4996 / 第 4996 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 4997 / 第 4997 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 4998 / 第 4998 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 4999 / 第 4999 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 5000 / 第 5000 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 5001-5005 / 第 5001-5005 行
```cpp
5001 | #endif
5002 | 
5003 | // DO NOT EDIT! THIS FILE HAS BEEN GENERATED!
5004 | 
5005 | #endif // SANITIZER_NETBSD_SYSCALL_HOOKS_H
```
- **Line 5001 / 第 5001 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 5002 / 第 5002 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 5003 / 第 5003 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 5004 / 第 5004 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 5005 / 第 5005 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

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
