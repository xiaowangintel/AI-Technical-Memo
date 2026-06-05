# lsan_thread.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/lsan/lsan_thread.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This file is a part of LeakSanitizer. See lsan_thread.h for details.
  - **CN**: 实现 LeakSanitizer 运行时中与 `lsan_thread` 相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行
```cpp
 1 | //=-- lsan_thread.cpp -----------------------------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | //
 9 | // This file is a part of LeakSanitizer.
10 | // See lsan_thread.h for details.
11 | //
12 | //===----------------------------------------------------------------------===//
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
14 | #include "lsan_thread.h"
15 | 
16 | #include "lsan.h"
17 | #include "lsan_allocator.h"
18 | #include "lsan_common.h"
19 | #include "sanitizer_common/sanitizer_common.h"
20 | #include "sanitizer_common/sanitizer_placement_new.h"
21 | #include "sanitizer_common/sanitizer_thread_history.h"
22 | #include "sanitizer_common/sanitizer_thread_registry.h"
23 | #include "sanitizer_common/sanitizer_tls_get_addr.h"
24 | 
```
- **Line 13 / 第 13 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 14 / 第 14 行**: EN: Includes `lsan_thread.h` so this file can use its declarations. CN: 包含 `lsan_thread.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 16 / 第 16 行**: EN: Includes `lsan.h` so this file can use its declarations. CN: 包含 `lsan.h`，以便当前文件使用其中的声明。
- **Line 17 / 第 17 行**: EN: Includes `lsan_allocator.h` so this file can use its declarations. CN: 包含 `lsan_allocator.h`，以便当前文件使用其中的声明。
- **Line 18 / 第 18 行**: EN: Includes `lsan_common.h` so this file can use its declarations. CN: 包含 `lsan_common.h`，以便当前文件使用其中的声明。
- **Line 19 / 第 19 行**: EN: Includes `sanitizer_common/sanitizer_common.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_common.h`，以便当前文件使用其中的声明。
- **Line 20 / 第 20 行**: EN: Includes `sanitizer_common/sanitizer_placement_new.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_placement_new.h`，以便当前文件使用其中的声明。
- **Line 21 / 第 21 行**: EN: Includes `sanitizer_common/sanitizer_thread_history.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_thread_history.h`，以便当前文件使用其中的声明。
- **Line 22 / 第 22 行**: EN: Includes `sanitizer_common/sanitizer_thread_registry.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_thread_registry.h`，以便当前文件使用其中的声明。
- **Line 23 / 第 23 行**: EN: Includes `sanitizer_common/sanitizer_tls_get_addr.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_tls_get_addr.h`，以便当前文件使用其中的声明。
- **Line 24 / 第 24 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 25-36 / 第 25-36 行
```cpp
25 | namespace __lsan {
26 | 
27 | static ThreadRegistry *thread_registry;
28 | static ThreadArgRetval *thread_arg_retval;
29 | 
30 | static Mutex mu_for_thread_context;
31 | static LowLevelAllocator allocator_for_thread_context;
32 | 
33 | static ThreadContextBase *CreateThreadContext(u32 tid) {
34 |   Lock lock(&mu_for_thread_context);
35 |   return new (allocator_for_thread_context) ThreadContext(tid);
36 | }
```
- **Line 25 / 第 25 行**: EN: Opens namespace `__lsan` to scope related declarations. CN: 打开命名空间 `__lsan`，为相关声明建立作用域。
- **Line 26 / 第 26 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 27 / 第 27 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 28 / 第 28 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 29 / 第 29 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 30 / 第 30 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 31 / 第 31 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 32 / 第 32 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 33 / 第 33 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 34 / 第 34 行**: EN: Declares function or method `lock`. CN: 声明函数或方法 `lock`。
- **Line 35 / 第 35 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 36 / 第 36 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 37-48 / 第 37-48 行
```cpp
37 | 
38 | void InitializeThreads() {
39 |   alignas(alignof(ThreadRegistry)) static char
40 |       thread_registry_placeholder[sizeof(ThreadRegistry)];
41 |   thread_registry =
42 |       new (thread_registry_placeholder) ThreadRegistry(CreateThreadContext);
43 | 
44 |   alignas(alignof(ThreadArgRetval)) static char
45 |       thread_arg_retval_placeholder[sizeof(ThreadArgRetval)];
46 |   thread_arg_retval = new (thread_arg_retval_placeholder) ThreadArgRetval();
47 | }
48 | 
```
- **Line 37 / 第 37 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 38 / 第 38 行**: EN: Starts the definition of function or method `InitializeThreads`. CN: 开始定义函数或方法 `InitializeThreads`。
- **Line 39 / 第 39 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 40 / 第 40 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 41 / 第 41 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 42 / 第 42 行**: EN: Declares function or method `new`. CN: 声明函数或方法 `new`。
- **Line 43 / 第 43 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 44 / 第 44 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 45 / 第 45 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 46 / 第 46 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 47 / 第 47 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 48 / 第 48 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 49-60 / 第 49-60 行
```cpp
49 | ThreadArgRetval &GetThreadArgRetval() { return *thread_arg_retval; }
50 | 
51 | ThreadContextLsanBase::ThreadContextLsanBase(int tid)
52 |     : ThreadContextBase(tid) {}
53 | 
54 | void ThreadContextLsanBase::OnStarted(void *arg) {
55 |   SetCurrentThread(this);
56 |   AllocatorThreadStart();
57 | }
58 | 
59 | void ThreadContextLsanBase::OnFinished() {
60 |   AllocatorThreadFinish();
```
- **Line 49 / 第 49 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 50 / 第 50 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 51 / 第 51 行**: EN: Starts the definition of function or method `ThreadContextLsanBase::ThreadContextLsanBase`. CN: 开始定义函数或方法 `ThreadContextLsanBase::ThreadContextLsanBase`。
- **Line 52 / 第 52 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 53 / 第 53 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 54 / 第 54 行**: EN: Starts the definition of function or method `ThreadContextLsanBase::OnStarted`. CN: 开始定义函数或方法 `ThreadContextLsanBase::OnStarted`。
- **Line 55 / 第 55 行**: EN: Declares function or method `SetCurrentThread`. CN: 声明函数或方法 `SetCurrentThread`。
- **Line 56 / 第 56 行**: EN: Declares function or method `AllocatorThreadStart`. CN: 声明函数或方法 `AllocatorThreadStart`。
- **Line 57 / 第 57 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 58 / 第 58 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 59 / 第 59 行**: EN: Starts the definition of function or method `ThreadContextLsanBase::OnFinished`. CN: 开始定义函数或方法 `ThreadContextLsanBase::OnFinished`。
- **Line 60 / 第 60 行**: EN: Declares function or method `AllocatorThreadFinish`. CN: 声明函数或方法 `AllocatorThreadFinish`。

### Lines 61-72 / 第 61-72 行
```cpp
61 |   DTLS_Destroy();
62 |   SetCurrentThread(nullptr);
63 | }
64 | 
65 | u32 ThreadCreate(u32 parent_tid, bool detached, void *arg) {
66 |   return thread_registry->CreateThread(0, detached, parent_tid, arg);
67 | }
68 | 
69 | void ThreadContextLsanBase::ThreadStart(u32 tid, ThreadID os_id,
70 |                                         ThreadType thread_type, void *arg) {
71 |   thread_registry->StartThread(tid, os_id, thread_type, arg);
72 | }
```
- **Line 61 / 第 61 行**: EN: Declares function or method `DTLS_Destroy`. CN: 声明函数或方法 `DTLS_Destroy`。
- **Line 62 / 第 62 行**: EN: Declares function or method `SetCurrentThread`. CN: 声明函数或方法 `SetCurrentThread`。
- **Line 63 / 第 63 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 64 / 第 64 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 65 / 第 65 行**: EN: Starts the definition of function or method `ThreadCreate`. CN: 开始定义函数或方法 `ThreadCreate`。
- **Line 66 / 第 66 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 67 / 第 67 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 68 / 第 68 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 69 / 第 69 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 70 / 第 70 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 71 / 第 71 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 72 / 第 72 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 73-84 / 第 73-84 行
```cpp
73 | 
74 | void ThreadFinish() { thread_registry->FinishThread(GetCurrentThreadId()); }
75 | 
76 | void EnsureMainThreadIDIsCorrect() {
77 |   if (GetCurrentThreadId() == kMainTid)
78 |     GetCurrentThread()->os_id = GetTid();
79 | }
80 | 
81 | ///// Interface to the common LSan module. /////
82 | 
83 | void GetThreadExtraStackRangesLocked(ThreadID os_id,
84 |                                      InternalMmapVector<Range> *ranges) {}
```
- **Line 73 / 第 73 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 74 / 第 74 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 75 / 第 75 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 76 / 第 76 行**: EN: Starts the definition of function or method `EnsureMainThreadIDIsCorrect`. CN: 开始定义函数或方法 `EnsureMainThreadIDIsCorrect`。
- **Line 77 / 第 77 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 78 / 第 78 行**: EN: Declares function or method `GetCurrentThread`. CN: 声明函数或方法 `GetCurrentThread`。
- **Line 79 / 第 79 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 80 / 第 80 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 81 / 第 81 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 82 / 第 82 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 83 / 第 83 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 84 / 第 84 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 85-96 / 第 85-96 行
```cpp
85 | void GetThreadExtraStackRangesLocked(InternalMmapVector<Range> *ranges) {}
86 | 
87 | void LockThreads() {
88 |   thread_registry->Lock();
89 |   thread_arg_retval->Lock();
90 | }
91 | 
92 | void UnlockThreads() {
93 |   thread_arg_retval->Unlock();
94 |   thread_registry->Unlock();
95 | }
96 | 
```
- **Line 85 / 第 85 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 86 / 第 86 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 87 / 第 87 行**: EN: Starts the definition of function or method `LockThreads`. CN: 开始定义函数或方法 `LockThreads`。
- **Line 88 / 第 88 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 89 / 第 89 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 90 / 第 90 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 91 / 第 91 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 92 / 第 92 行**: EN: Starts the definition of function or method `UnlockThreads`. CN: 开始定义函数或方法 `UnlockThreads`。
- **Line 93 / 第 93 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 94 / 第 94 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 95 / 第 95 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 96 / 第 96 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 97-108 / 第 97-108 行
```cpp
 97 | ThreadRegistry *GetLsanThreadRegistryLocked() {
 98 |   thread_registry->CheckLocked();
 99 |   return thread_registry;
100 | }
101 | 
102 | void GetRunningThreadsLocked(InternalMmapVector<ThreadID> *threads) {
103 |   GetLsanThreadRegistryLocked()->RunCallbackForEachThreadLocked(
104 |       [](ThreadContextBase *tctx, void *threads) {
105 |         if (tctx->status == ThreadStatusRunning) {
106 |           reinterpret_cast<InternalMmapVector<ThreadID> *>(threads)->push_back(
107 |               tctx->os_id);
108 |         }
```
- **Line 97 / 第 97 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 98 / 第 98 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 99 / 第 99 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 100 / 第 100 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 101 / 第 101 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 102 / 第 102 行**: EN: Starts the definition of function or method `GetRunningThreadsLocked`. CN: 开始定义函数或方法 `GetRunningThreadsLocked`。
- **Line 103 / 第 103 行**: EN: Starts the definition of function or method `GetLsanThreadRegistryLocked`. CN: 开始定义函数或方法 `GetLsanThreadRegistryLocked`。
- **Line 104 / 第 104 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 105 / 第 105 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 106 / 第 106 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 107 / 第 107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 108 / 第 108 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 109-120 / 第 109-120 行
```cpp
109 |       },
110 |       threads);
111 | }
112 | 
113 | void PrintThreads() {
114 |   InternalScopedString out;
115 |   PrintThreadHistory(*thread_registry, out);
116 |   Report("%s\n", out.data());
117 | }
118 | 
119 | void GetAdditionalThreadContextPtrsLocked(InternalMmapVector<uptr> *ptrs) {
120 |   GetThreadArgRetval().GetAllPtrsLocked(ptrs);
```
- **Line 109 / 第 109 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 110 / 第 110 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 111 / 第 111 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 112 / 第 112 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 113 / 第 113 行**: EN: Starts the definition of function or method `PrintThreads`. CN: 开始定义函数或方法 `PrintThreads`。
- **Line 114 / 第 114 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 115 / 第 115 行**: EN: Declares function or method `PrintThreadHistory`. CN: 声明函数或方法 `PrintThreadHistory`。
- **Line 116 / 第 116 行**: EN: Declares function or method `Report`. CN: 声明函数或方法 `Report`。
- **Line 117 / 第 117 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 118 / 第 118 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 119 / 第 119 行**: EN: Starts the definition of function or method `GetAdditionalThreadContextPtrsLocked`. CN: 开始定义函数或方法 `GetAdditionalThreadContextPtrsLocked`。
- **Line 120 / 第 120 行**: EN: Declares function or method `GetThreadArgRetval`. CN: 声明函数或方法 `GetThreadArgRetval`。

### Lines 121-123 / 第 121-123 行
```cpp
121 | }
122 | 
123 | }  // namespace __lsan
```
- **Line 121 / 第 121 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 122 / 第 122 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 123 / 第 123 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

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

- `lsan_thread.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `lsan.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `lsan_allocator.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `lsan_common.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `sanitizer_common/sanitizer_common.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `sanitizer_common/sanitizer_placement_new.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `sanitizer_common/sanitizer_thread_history.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `sanitizer_common/sanitizer_thread_registry.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `sanitizer_common/sanitizer_tls_get_addr.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
