# common.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/gwp_asan/common.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Backtrace() returns the number of available frames, which may be greater than the number of frames in the buffer. In this case, we need to only pack the number of frames that are in the buffer.
  - **CN**: 实现 GWP-ASan 守护分配运行时中与 `common` 相关的部件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行
```cpp
 1 | //===-- common.cpp ----------------------------------------------*- C++ -*-===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | 
 9 | #include "gwp_asan/common.h"
10 | #include "gwp_asan/stack_trace_compressor.h"
11 | 
12 | #include <assert.h>
```
- **Line 1 / 第 1 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2 / 第 2 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3 / 第 3 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4 / 第 4 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 5 / 第 5 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 6 / 第 6 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 7 / 第 7 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 8 / 第 8 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 9 / 第 9 行**: EN: Includes `gwp_asan/common.h` so this file can use its declarations. CN: 包含 `gwp_asan/common.h`，以便当前文件使用其中的声明。
- **Line 10 / 第 10 行**: EN: Includes `gwp_asan/stack_trace_compressor.h` so this file can use its declarations. CN: 包含 `gwp_asan/stack_trace_compressor.h`，以便当前文件使用其中的声明。
- **Line 11 / 第 11 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 12 / 第 12 行**: EN: Includes `assert.h` so this file can use its declarations. CN: 包含 `assert.h`，以便当前文件使用其中的声明。

### Lines 13-24 / 第 13-24 行
```cpp
13 | 
14 | using AllocationMetadata = gwp_asan::AllocationMetadata;
15 | using Error = gwp_asan::Error;
16 | 
17 | namespace gwp_asan {
18 | 
19 | const char *ErrorToString(const Error &E) {
20 |   switch (E) {
21 |   case Error::UNKNOWN:
22 |     return "Unknown";
23 |   case Error::USE_AFTER_FREE:
24 |     return "Use After Free";
```
- **Line 13 / 第 13 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 14 / 第 14 行**: EN: Adds a using declaration or alias for a type, symbol, or namespace. CN: 为类型、符号或命名空间添加 using 声明或别名。
- **Line 15 / 第 15 行**: EN: Adds a using declaration or alias for a type, symbol, or namespace. CN: 为类型、符号或命名空间添加 using 声明或别名。
- **Line 16 / 第 16 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 17 / 第 17 行**: EN: Opens namespace `gwp_asan` to scope related declarations. CN: 打开命名空间 `gwp_asan`，为相关声明建立作用域。
- **Line 18 / 第 18 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 19 / 第 19 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 20 / 第 20 行**: EN: Dispatches control flow based on a selector expression. CN: 根据选择表达式分发控制流。
- **Line 21 / 第 21 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 22 / 第 22 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 23 / 第 23 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 24 / 第 24 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 25-36 / 第 25-36 行
```cpp
25 |   case Error::DOUBLE_FREE:
26 |     return "Double Free";
27 |   case Error::INVALID_FREE:
28 |     return "Invalid (Wild) Free";
29 |   case Error::BUFFER_OVERFLOW:
30 |     return "Buffer Overflow";
31 |   case Error::BUFFER_UNDERFLOW:
32 |     return "Buffer Underflow";
33 |   }
34 |   __builtin_trap();
35 | }
36 | 
```
- **Line 25 / 第 25 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 26 / 第 26 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 27 / 第 27 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 28 / 第 28 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 29 / 第 29 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 30 / 第 30 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 31 / 第 31 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 32 / 第 32 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 33 / 第 33 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 34 / 第 34 行**: EN: Declares function or method `__builtin_trap`. CN: 声明函数或方法 `__builtin_trap`。
- **Line 35 / 第 35 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 36 / 第 36 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 37-48 / 第 37-48 行
```cpp
37 | constexpr size_t AllocationMetadata::kStackFrameStorageBytes;
38 | constexpr size_t AllocationMetadata::kMaxTraceLengthToCollect;
39 | 
40 | void AllocationMetadata::RecordAllocation(uintptr_t AllocAddr,
41 |                                           size_t AllocSize) {
42 |   Addr = AllocAddr;
43 |   RequestedSize = AllocSize;
44 |   IsDeallocated = false;
45 | 
46 |   AllocationTrace.ThreadID = getThreadID();
47 |   DeallocationTrace.TraceSize = 0;
48 |   DeallocationTrace.ThreadID = kInvalidThreadID;
```
- **Line 37 / 第 37 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 38 / 第 38 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 39 / 第 39 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 40 / 第 40 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 41 / 第 41 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 42 / 第 42 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 43 / 第 43 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 44 / 第 44 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 45 / 第 45 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 46 / 第 46 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 47 / 第 47 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 48 / 第 48 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 49-60 / 第 49-60 行
```cpp
49 | }
50 | 
51 | void AllocationMetadata::RecordDeallocation() {
52 |   IsDeallocated = true;
53 |   DeallocationTrace.ThreadID = getThreadID();
54 | }
55 | 
56 | void AllocationMetadata::CallSiteInfo::RecordBacktrace(
57 |     options::Backtrace_t Backtrace) {
58 |   TraceSize = 0;
59 |   if (!Backtrace)
60 |     return;
```
- **Line 49 / 第 49 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 50 / 第 50 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 51 / 第 51 行**: EN: Starts the definition of function or method `AllocationMetadata::RecordDeallocation`. CN: 开始定义函数或方法 `AllocationMetadata::RecordDeallocation`。
- **Line 52 / 第 52 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 53 / 第 53 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 54 / 第 54 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 55 / 第 55 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 56 / 第 56 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 57 / 第 57 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 58 / 第 58 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 59 / 第 59 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 60 / 第 60 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 61-72 / 第 61-72 行
```cpp
61 | 
62 |   uintptr_t UncompressedBuffer[kMaxTraceLengthToCollect];
63 |   size_t BacktraceLength =
64 |       Backtrace(UncompressedBuffer, kMaxTraceLengthToCollect);
65 |   // Backtrace() returns the number of available frames, which may be greater
66 |   // than the number of frames in the buffer. In this case, we need to only pack
67 |   // the number of frames that are in the buffer.
68 |   if (BacktraceLength > kMaxTraceLengthToCollect)
69 |     BacktraceLength = kMaxTraceLengthToCollect;
70 |   TraceSize =
71 |       compression::pack(UncompressedBuffer, BacktraceLength, CompressedTrace,
72 |                         AllocationMetadata::kStackFrameStorageBytes);
```
- **Line 61 / 第 61 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 62 / 第 62 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 63 / 第 63 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 64 / 第 64 行**: EN: Declares function or method `Backtrace`. CN: 声明函数或方法 `Backtrace`。
- **Line 65 / 第 65 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 66 / 第 66 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 67 / 第 67 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 68 / 第 68 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 69 / 第 69 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 70 / 第 70 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 71 / 第 71 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 72 / 第 72 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 73-84 / 第 73-84 行
```cpp
73 | }
74 | 
75 | size_t AllocatorState::maximumAllocationSize() const { return PageSize; }
76 | 
77 | uintptr_t AllocatorState::slotToAddr(size_t N) const {
78 |   return GuardedPagePool + (PageSize * (1 + N)) + (maximumAllocationSize() * N);
79 | }
80 | 
81 | bool AllocatorState::isGuardPage(uintptr_t Ptr) const {
82 |   assert(pointerIsMine(reinterpret_cast<void *>(Ptr)));
83 |   size_t PageOffsetFromPoolStart = (Ptr - GuardedPagePool) / PageSize;
84 |   size_t PagesPerSlot = maximumAllocationSize() / PageSize;
```
- **Line 73 / 第 73 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 74 / 第 74 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 75 / 第 75 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 76 / 第 76 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 77 / 第 77 行**: EN: Starts the definition of function or method `AllocatorState::slotToAddr`. CN: 开始定义函数或方法 `AllocatorState::slotToAddr`。
- **Line 78 / 第 78 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 79 / 第 79 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 80 / 第 80 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 81 / 第 81 行**: EN: Starts the definition of function or method `AllocatorState::isGuardPage`. CN: 开始定义函数或方法 `AllocatorState::isGuardPage`。
- **Line 82 / 第 82 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 83 / 第 83 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 84 / 第 84 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 85-96 / 第 85-96 行
```cpp
85 |   return (PageOffsetFromPoolStart % (PagesPerSlot + 1)) == 0;
86 | }
87 | 
88 | static size_t addrToSlot(const AllocatorState *State, uintptr_t Ptr) {
89 |   size_t ByteOffsetFromPoolStart = Ptr - State->GuardedPagePool;
90 |   return ByteOffsetFromPoolStart /
91 |          (State->maximumAllocationSize() + State->PageSize);
92 | }
93 | 
94 | size_t AllocatorState::getNearestSlot(uintptr_t Ptr) const {
95 |   if (Ptr <= GuardedPagePool + PageSize)
96 |     return 0;
```
- **Line 85 / 第 85 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 86 / 第 86 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 87 / 第 87 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 88 / 第 88 行**: EN: Starts the definition of function or method `addrToSlot`. CN: 开始定义函数或方法 `addrToSlot`。
- **Line 89 / 第 89 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 90 / 第 90 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 91 / 第 91 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 92 / 第 92 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 93 / 第 93 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 94 / 第 94 行**: EN: Starts the definition of function or method `AllocatorState::getNearestSlot`. CN: 开始定义函数或方法 `AllocatorState::getNearestSlot`。
- **Line 95 / 第 95 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 96 / 第 96 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 97-108 / 第 97-108 行
```cpp
 97 |   if (Ptr > GuardedPagePoolEnd - PageSize)
 98 |     return MaxSimultaneousAllocations - 1;
 99 | 
100 |   if (!isGuardPage(Ptr))
101 |     return addrToSlot(this, Ptr);
102 | 
103 |   if (Ptr % PageSize <= PageSize / 2)
104 |     return addrToSlot(this, Ptr - PageSize); // Round down.
105 |   return addrToSlot(this, Ptr + PageSize);   // Round up.
106 | }
107 | 
108 | uintptr_t AllocatorState::internallyDetectedErrorFaultAddress() const {
```
- **Line 97 / 第 97 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 98 / 第 98 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 99 / 第 99 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 100 / 第 100 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 101 / 第 101 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 102 / 第 102 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 103 / 第 103 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 104 / 第 104 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 105 / 第 105 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 106 / 第 106 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 107 / 第 107 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 108 / 第 108 行**: EN: Starts the definition of function or method `AllocatorState::internallyDetectedErrorFaultAddress`. CN: 开始定义函数或方法 `AllocatorState::internallyDetectedErrorFaultAddress`。

### Lines 109-112 / 第 109-112 行
```cpp
109 |   return GuardedPagePoolEnd - 0x10;
110 | }
111 | 
112 | } // namespace gwp_asan
```
- **Line 109 / 第 109 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 110 / 第 110 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 111 / 第 111 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 112 / 第 112 行**: EN: Closes a named namespace scope. CN: 关闭一个具名命名空间作用域。

## Key Concepts / 关键概念

- **EN**: sampled heap hardening
  - **CN**: 采样式堆加固
- **EN**: guarded allocation metadata
  - **CN**: 守护分配元数据
- **EN**: allocation crash diagnosis
  - **CN**: 分配错误诊断
- **EN**: namespace scoping and organization
  - **CN**: 命名空间作用域与组织

## Dependencies / 依赖关系

- `gwp_asan/common.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `gwp_asan/stack_trace_compressor.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `assert.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
