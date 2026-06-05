# lsan_posix.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/lsan/lsan_posix.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This file is a part of LeakSanitizer. Standalone LSan RTL code common to POSIX-like systems.
  - **CN**: 实现 LeakSanitizer 运行时中与 `lsan_posix` 相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行
```cpp
 1 | //=-- lsan_posix.cpp -----------------------------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===---------------------------------------------------------------------===//
 8 | //
 9 | // This file is a part of LeakSanitizer.
10 | // Standalone LSan RTL code common to POSIX-like systems.
11 | //
12 | //===---------------------------------------------------------------------===//
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
- **Line 11 / 第 11 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 12 / 第 12 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 13-24 / 第 13-24 行
```cpp
13 | 
14 | #include "sanitizer_common/sanitizer_platform.h"
15 | 
16 | #if SANITIZER_POSIX
17 | #  include <pthread.h>
18 | 
19 | #  include "lsan.h"
20 | #  include "lsan_allocator.h"
21 | #  include "lsan_thread.h"
22 | #  include "sanitizer_common/sanitizer_stacktrace.h"
23 | #  include "sanitizer_common/sanitizer_tls_get_addr.h"
24 | 
```
- **Line 13 / 第 13 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 14 / 第 14 行**: EN: Includes `sanitizer_common/sanitizer_platform.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_platform.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 16 / 第 16 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 17 / 第 17 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 18 / 第 18 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 19 / 第 19 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 20 / 第 20 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 21 / 第 21 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 22 / 第 22 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 23 / 第 23 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 24 / 第 24 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 25-36 / 第 25-36 行
```cpp
25 | namespace __lsan {
26 | 
27 | ThreadContext::ThreadContext(int tid) : ThreadContextLsanBase(tid) {}
28 | 
29 | struct OnStartedArgs {
30 |   uptr stack_begin;
31 |   uptr stack_end;
32 |   uptr cache_begin;
33 |   uptr cache_end;
34 |   uptr tls_begin;
35 |   uptr tls_end;
36 |   DTLS *dtls;
```
- **Line 25 / 第 25 行**: EN: Opens namespace `__lsan` to scope related declarations. CN: 打开命名空间 `__lsan`，为相关声明建立作用域。
- **Line 26 / 第 26 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 27 / 第 27 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 28 / 第 28 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 29 / 第 29 行**: EN: Begins the declaration of struct `OnStartedArgs`. CN: 开始声明 struct `OnStartedArgs`。
- **Line 30 / 第 30 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 31 / 第 31 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 32 / 第 32 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 33 / 第 33 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 34 / 第 34 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 35 / 第 35 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 36 / 第 36 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 37-48 / 第 37-48 行
```cpp
37 | };
38 | 
39 | void ThreadContext::OnStarted(void *arg) {
40 |   ThreadContextLsanBase::OnStarted(arg);
41 |   auto args = reinterpret_cast<const OnStartedArgs *>(arg);
42 |   stack_begin_ = args->stack_begin;
43 |   stack_end_ = args->stack_end;
44 |   tls_begin_ = args->tls_begin;
45 |   tls_end_ = args->tls_end;
46 |   cache_begin_ = args->cache_begin;
47 |   cache_end_ = args->cache_end;
48 |   dtls_ = args->dtls;
```
- **Line 37 / 第 37 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 38 / 第 38 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 39 / 第 39 行**: EN: Starts the definition of function or method `ThreadContext::OnStarted`. CN: 开始定义函数或方法 `ThreadContext::OnStarted`。
- **Line 40 / 第 40 行**: EN: Declares function or method `ThreadContextLsanBase::OnStarted`. CN: 声明函数或方法 `ThreadContextLsanBase::OnStarted`。
- **Line 41 / 第 41 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 42 / 第 42 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 43 / 第 43 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 44 / 第 44 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 45 / 第 45 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 46 / 第 46 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 47 / 第 47 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 48 / 第 48 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 49-60 / 第 49-60 行
```cpp
49 | }
50 | 
51 | void ThreadStart(u32 tid, ThreadID os_id, ThreadType thread_type) {
52 |   OnStartedArgs args;
53 |   GetThreadStackAndTls(tid == kMainTid, &args.stack_begin, &args.stack_end,
54 |                        &args.tls_begin, &args.tls_end);
55 |   GetAllocatorCacheRange(&args.cache_begin, &args.cache_end);
56 |   args.dtls = DTLS_Get();
57 |   ThreadContextLsanBase::ThreadStart(tid, os_id, thread_type, &args);
58 | }
59 | 
60 | bool GetThreadRangesLocked(ThreadID os_id, uptr *stack_begin, uptr *stack_end,
```
- **Line 49 / 第 49 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 50 / 第 50 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 51 / 第 51 行**: EN: Starts the definition of function or method `ThreadStart`. CN: 开始定义函数或方法 `ThreadStart`。
- **Line 52 / 第 52 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 53 / 第 53 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 54 / 第 54 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 55 / 第 55 行**: EN: Declares function or method `GetAllocatorCacheRange`. CN: 声明函数或方法 `GetAllocatorCacheRange`。
- **Line 56 / 第 56 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 57 / 第 57 行**: EN: Declares function or method `ThreadContextLsanBase::ThreadStart`. CN: 声明函数或方法 `ThreadContextLsanBase::ThreadStart`。
- **Line 58 / 第 58 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 59 / 第 59 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 60 / 第 60 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 61-72 / 第 61-72 行
```cpp
61 |                            uptr *tls_begin, uptr *tls_end, uptr *cache_begin,
62 |                            uptr *cache_end, DTLS **dtls) {
63 |   ThreadContext *context = static_cast<ThreadContext *>(
64 |       GetLsanThreadRegistryLocked()->FindThreadContextByOsIDLocked(os_id));
65 |   if (!context)
66 |     return false;
67 |   *stack_begin = context->stack_begin();
68 |   *stack_end = context->stack_end();
69 |   *tls_begin = context->tls_begin();
70 |   *tls_end = context->tls_end();
71 |   *cache_begin = context->cache_begin();
72 |   *cache_end = context->cache_end();
```
- **Line 61 / 第 61 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 62 / 第 62 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 63 / 第 63 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 64 / 第 64 行**: EN: Declares function or method `GetLsanThreadRegistryLocked`. CN: 声明函数或方法 `GetLsanThreadRegistryLocked`。
- **Line 65 / 第 65 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 66 / 第 66 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 67 / 第 67 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 68 / 第 68 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 69 / 第 69 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 70 / 第 70 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 71 / 第 71 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 72 / 第 72 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 73-84 / 第 73-84 行
```cpp
73 |   *dtls = context->dtls();
74 |   return true;
75 | }
76 | 
77 | void InitializeMainThread() {
78 |   u32 tid = ThreadCreate(kMainTid, true);
79 |   CHECK_EQ(tid, kMainTid);
80 |   ThreadStart(tid, GetTid());
81 | }
82 | 
83 | static void OnStackUnwind(const SignalContext &sig, const void *,
84 |                           BufferedStackTrace *stack) {
```
- **Line 73 / 第 73 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 74 / 第 74 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 75 / 第 75 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 76 / 第 76 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 77 / 第 77 行**: EN: Starts the definition of function or method `InitializeMainThread`. CN: 开始定义函数或方法 `InitializeMainThread`。
- **Line 78 / 第 78 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 79 / 第 79 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 80 / 第 80 行**: EN: Declares function or method `ThreadStart`. CN: 声明函数或方法 `ThreadStart`。
- **Line 81 / 第 81 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 82 / 第 82 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 83 / 第 83 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 84 / 第 84 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 85-96 / 第 85-96 行
```cpp
85 |   stack->Unwind(StackTrace::GetNextInstructionPc(sig.pc), sig.bp, sig.context,
86 |                 common_flags()->fast_unwind_on_fatal);
87 | }
88 | 
89 | void LsanOnDeadlySignal(int signo, void *siginfo, void *context) {
90 |   HandleDeadlySignal(siginfo, context, GetCurrentThreadId(), &OnStackUnwind,
91 |                      nullptr);
92 | }
93 | 
94 | void InstallAtExitCheckLeaks() {
95 |   if (common_flags()->detect_leaks && common_flags()->leak_check_at_exit)
96 |     Atexit(DoLeakCheck);
```
- **Line 85 / 第 85 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 86 / 第 86 行**: EN: Declares function or method `common_flags`. CN: 声明函数或方法 `common_flags`。
- **Line 87 / 第 87 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 88 / 第 88 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 89 / 第 89 行**: EN: Starts the definition of function or method `LsanOnDeadlySignal`. CN: 开始定义函数或方法 `LsanOnDeadlySignal`。
- **Line 90 / 第 90 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 91 / 第 91 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 92 / 第 92 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 93 / 第 93 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 94 / 第 94 行**: EN: Starts the definition of function or method `InstallAtExitCheckLeaks`. CN: 开始定义函数或方法 `InstallAtExitCheckLeaks`。
- **Line 95 / 第 95 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 96 / 第 96 行**: EN: Declares function or method `Atexit`. CN: 声明函数或方法 `Atexit`。

### Lines 97-108 / 第 97-108 行
```cpp
 97 | }
 98 | 
 99 | static void BeforeFork() {
100 |   VReport(2, "BeforeFork tid: %llu\n", GetTid());
101 |   LockGlobal();
102 |   LockThreads();
103 |   LockAllocator();
104 |   StackDepotLockBeforeFork();
105 | }
106 | 
107 | static void AfterFork(bool fork_child) {
108 |   StackDepotUnlockAfterFork(fork_child);
```
- **Line 97 / 第 97 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 98 / 第 98 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 99 / 第 99 行**: EN: Starts the definition of function or method `BeforeFork`. CN: 开始定义函数或方法 `BeforeFork`。
- **Line 100 / 第 100 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 101 / 第 101 行**: EN: Declares function or method `LockGlobal`. CN: 声明函数或方法 `LockGlobal`。
- **Line 102 / 第 102 行**: EN: Declares function or method `LockThreads`. CN: 声明函数或方法 `LockThreads`。
- **Line 103 / 第 103 行**: EN: Declares function or method `LockAllocator`. CN: 声明函数或方法 `LockAllocator`。
- **Line 104 / 第 104 行**: EN: Declares function or method `StackDepotLockBeforeFork`. CN: 声明函数或方法 `StackDepotLockBeforeFork`。
- **Line 105 / 第 105 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 106 / 第 106 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 107 / 第 107 行**: EN: Starts the definition of function or method `AfterFork`. CN: 开始定义函数或方法 `AfterFork`。
- **Line 108 / 第 108 行**: EN: Declares function or method `StackDepotUnlockAfterFork`. CN: 声明函数或方法 `StackDepotUnlockAfterFork`。

### Lines 109-120 / 第 109-120 行
```cpp
109 |   UnlockAllocator();
110 |   UnlockThreads();
111 |   UnlockGlobal();
112 |   VReport(2, "AfterFork tid: %llu\n", GetTid());
113 | }
114 | 
115 | void InstallAtForkHandler() {
116 | #  if SANITIZER_SOLARIS || SANITIZER_NETBSD || SANITIZER_APPLE
117 |   return;  // FIXME: Implement FutexWait.
118 | #  endif
119 |   pthread_atfork(
120 |       &BeforeFork, []() { AfterFork(/* fork_child= */ false); },
```
- **Line 109 / 第 109 行**: EN: Declares function or method `UnlockAllocator`. CN: 声明函数或方法 `UnlockAllocator`。
- **Line 110 / 第 110 行**: EN: Declares function or method `UnlockThreads`. CN: 声明函数或方法 `UnlockThreads`。
- **Line 111 / 第 111 行**: EN: Declares function or method `UnlockGlobal`. CN: 声明函数或方法 `UnlockGlobal`。
- **Line 112 / 第 112 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 113 / 第 113 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 114 / 第 114 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 115 / 第 115 行**: EN: Starts the definition of function or method `InstallAtForkHandler`. CN: 开始定义函数或方法 `InstallAtForkHandler`。
- **Line 116 / 第 116 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 117 / 第 117 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 118 / 第 118 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 119 / 第 119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 120 / 第 120 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 121-126 / 第 121-126 行
```cpp
121 |       []() { AfterFork(/* fork_child= */ true); });
122 | }
123 | 
124 | }  // namespace __lsan
125 | 
126 | #endif  // SANITIZER_POSIX
```
- **Line 121 / 第 121 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 122 / 第 122 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 123 / 第 123 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 124 / 第 124 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 125 / 第 125 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 126 / 第 126 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

## Key Concepts / 关键概念

- **EN**: leak root scanning
  - **CN**: 泄漏根扫描
- **EN**: reachability-based leak detection
  - **CN**: 基于可达性的泄漏检测
- **EN**: sanitizer suppression and reporting
  - **CN**: sanitizer 抑制与报告
- **EN**: namespace scoping and organization
  - **CN**: 命名空间作用域与组织
- **EN**: allocator state management
  - **CN**: 分配器状态管理
- **EN**: thread-aware runtime coordination
  - **CN**: 线程感知的运行时协作

## Dependencies / 依赖关系

- `sanitizer_common/sanitizer_platform.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `pthread.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `lsan.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `lsan_allocator.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `lsan_thread.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `sanitizer_common/sanitizer_stacktrace.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `sanitizer_common/sanitizer_tls_get_addr.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
