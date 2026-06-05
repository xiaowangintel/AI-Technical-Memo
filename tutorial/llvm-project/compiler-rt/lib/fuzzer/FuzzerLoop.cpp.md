# FuzzerLoop.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/fuzzer/FuzzerLoop.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Fuzzer's main loop.
  - **CN**: 实现 libFuzzer 中与 `FuzzerLoop` 相关的组件或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
```cpp
 1 | //===- FuzzerLoop.cpp - Fuzzer's main loop --------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | // Fuzzer's main loop.
 9 | //===----------------------------------------------------------------------===//
10 | 
11 | #include "FuzzerCorpus.h"
12 | #include "FuzzerIO.h"
13 | #include "FuzzerInternal.h"
14 | #include "FuzzerMutate.h"
15 | #include "FuzzerPlatform.h"
16 | #include "FuzzerRandom.h"
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
- **Line 10 / 第 10 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 11 / 第 11 行**: EN: Includes `FuzzerCorpus.h` so this file can use its declarations. CN: 包含 `FuzzerCorpus.h`，以便当前文件使用其中的声明。
- **Line 12 / 第 12 行**: EN: Includes `FuzzerIO.h` so this file can use its declarations. CN: 包含 `FuzzerIO.h`，以便当前文件使用其中的声明。
- **Line 13 / 第 13 行**: EN: Includes `FuzzerInternal.h` so this file can use its declarations. CN: 包含 `FuzzerInternal.h`，以便当前文件使用其中的声明。
- **Line 14 / 第 14 行**: EN: Includes `FuzzerMutate.h` so this file can use its declarations. CN: 包含 `FuzzerMutate.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Includes `FuzzerPlatform.h` so this file can use its declarations. CN: 包含 `FuzzerPlatform.h`，以便当前文件使用其中的声明。
- **Line 16 / 第 16 行**: EN: Includes `FuzzerRandom.h` so this file can use its declarations. CN: 包含 `FuzzerRandom.h`，以便当前文件使用其中的声明。

### Lines 17-32 / 第 17-32 行
```cpp
17 | #include "FuzzerTracePC.h"
18 | #include <algorithm>
19 | #include <cstring>
20 | #include <memory>
21 | #include <mutex>
22 | #include <set>
23 | 
24 | #if defined(__has_include)
25 | #if __has_include(<sanitizer / lsan_interface.h>)
26 | #include <sanitizer/lsan_interface.h>
27 | #endif
28 | #endif
29 | 
30 | #define NO_SANITIZE_MEMORY
31 | #if defined(__has_feature)
32 | #if __has_feature(memory_sanitizer)
```
- **Line 17 / 第 17 行**: EN: Includes `FuzzerTracePC.h` so this file can use its declarations. CN: 包含 `FuzzerTracePC.h`，以便当前文件使用其中的声明。
- **Line 18 / 第 18 行**: EN: Includes `algorithm` so this file can use its declarations. CN: 包含 `algorithm`，以便当前文件使用其中的声明。
- **Line 19 / 第 19 行**: EN: Includes `cstring` so this file can use its declarations. CN: 包含 `cstring`，以便当前文件使用其中的声明。
- **Line 20 / 第 20 行**: EN: Includes `memory` so this file can use its declarations. CN: 包含 `memory`，以便当前文件使用其中的声明。
- **Line 21 / 第 21 行**: EN: Includes `mutex` so this file can use its declarations. CN: 包含 `mutex`，以便当前文件使用其中的声明。
- **Line 22 / 第 22 行**: EN: Includes `set` so this file can use its declarations. CN: 包含 `set`，以便当前文件使用其中的声明。
- **Line 23 / 第 23 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 24 / 第 24 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 25 / 第 25 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 26 / 第 26 行**: EN: Includes `sanitizer/lsan_interface.h` so this file can use its declarations. CN: 包含 `sanitizer/lsan_interface.h`，以便当前文件使用其中的声明。
- **Line 27 / 第 27 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 28 / 第 28 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 29 / 第 29 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 30 / 第 30 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 31 / 第 31 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 32 / 第 32 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。

### Lines 33-48 / 第 33-48 行
```cpp
33 | #undef NO_SANITIZE_MEMORY
34 | #define NO_SANITIZE_MEMORY __attribute__((no_sanitize_memory))
35 | #endif
36 | #endif
37 | 
38 | namespace fuzzer {
39 | static const size_t kMaxUnitSizeToPrint = 256;
40 | 
41 | thread_local bool Fuzzer::IsMyThread;
42 | 
43 | bool RunningUserCallback = false;
44 | 
45 | // Only one Fuzzer per process.
46 | static Fuzzer *F;
47 | 
48 | // Leak detection is expensive, so we first check if there were more mallocs
```
- **Line 33 / 第 33 行**: EN: Undefines a previously introduced macro symbol. CN: 取消定义先前引入的宏符号。
- **Line 34 / 第 34 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 35 / 第 35 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 36 / 第 36 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 37 / 第 37 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 38 / 第 38 行**: EN: Opens namespace `fuzzer` to scope related declarations. CN: 打开命名空间 `fuzzer`，为相关声明建立作用域。
- **Line 39 / 第 39 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 40 / 第 40 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 41 / 第 41 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 42 / 第 42 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 43 / 第 43 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 44 / 第 44 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 45 / 第 45 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 46 / 第 46 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 47 / 第 47 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 48 / 第 48 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 49-64 / 第 49-64 行
```cpp
49 | // than frees (using the sanitizer malloc hooks) and only then try to call lsan.
50 | struct MallocFreeTracer {
51 |   void Start(int TraceLevel) {
52 |     this->TraceLevel = TraceLevel;
53 |     if (TraceLevel)
54 |       Printf("MallocFreeTracer: START\n");
55 |     Mallocs = 0;
56 |     Frees = 0;
57 |   }
58 |   // Returns true if there were more mallocs than frees.
59 |   bool Stop() {
60 |     if (TraceLevel)
61 |       Printf("MallocFreeTracer: STOP %zd %zd (%s)\n", Mallocs.load(),
62 |              Frees.load(), Mallocs == Frees ? "same" : "DIFFERENT");
63 |     bool Result = Mallocs > Frees;
64 |     Mallocs = 0;
```
- **Line 49 / 第 49 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 50 / 第 50 行**: EN: Begins the declaration of struct `MallocFreeTracer`. CN: 开始声明 struct `MallocFreeTracer`。
- **Line 51 / 第 51 行**: EN: Starts the definition of function or method `Start`. CN: 开始定义函数或方法 `Start`。
- **Line 52 / 第 52 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 53 / 第 53 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 54 / 第 54 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 55 / 第 55 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 56 / 第 56 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 57 / 第 57 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 58 / 第 58 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 59 / 第 59 行**: EN: Starts the definition of function or method `Stop`. CN: 开始定义函数或方法 `Stop`。
- **Line 60 / 第 60 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 61 / 第 61 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 62 / 第 62 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 63 / 第 63 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 64 / 第 64 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 65-80 / 第 65-80 行
```cpp
65 |     Frees = 0;
66 |     TraceLevel = 0;
67 |     return Result;
68 |   }
69 |   std::atomic<size_t> Mallocs;
70 |   std::atomic<size_t> Frees;
71 |   int TraceLevel = 0;
72 | 
73 |   std::recursive_mutex TraceMutex;
74 |   bool TraceDisabled = false;
75 | };
76 | 
77 | static MallocFreeTracer AllocTracer;
78 | 
79 | // Locks printing and avoids nested hooks triggered from mallocs/frees in
80 | // sanitizer.
```
- **Line 65 / 第 65 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 66 / 第 66 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 67 / 第 67 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 68 / 第 68 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 69 / 第 69 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 70 / 第 70 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 71 / 第 71 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 72 / 第 72 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 73 / 第 73 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 74 / 第 74 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 75 / 第 75 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 76 / 第 76 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 77 / 第 77 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 78 / 第 78 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 79 / 第 79 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 80 / 第 80 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 81-96 / 第 81-96 行
```cpp
81 | class TraceLock {
82 | public:
83 |   TraceLock() : Lock(AllocTracer.TraceMutex) {
84 |     AllocTracer.TraceDisabled = !AllocTracer.TraceDisabled;
85 |   }
86 |   ~TraceLock() { AllocTracer.TraceDisabled = !AllocTracer.TraceDisabled; }
87 | 
88 |   bool IsDisabled() const {
89 |     // This is already inverted value.
90 |     return !AllocTracer.TraceDisabled;
91 |   }
92 | 
93 | private:
94 |   std::lock_guard<std::recursive_mutex> Lock;
95 | };
96 | 
```
- **Line 81 / 第 81 行**: EN: Begins the declaration of class `TraceLock`. CN: 开始声明 class `TraceLock`。
- **Line 82 / 第 82 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 83 / 第 83 行**: EN: Starts the definition of function or method `TraceLock`. CN: 开始定义函数或方法 `TraceLock`。
- **Line 84 / 第 84 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 85 / 第 85 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 86 / 第 86 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 87 / 第 87 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 88 / 第 88 行**: EN: Starts the definition of function or method `IsDisabled`. CN: 开始定义函数或方法 `IsDisabled`。
- **Line 89 / 第 89 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 90 / 第 90 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 91 / 第 91 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 92 / 第 92 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 93 / 第 93 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 94 / 第 94 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 95 / 第 95 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 96 / 第 96 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 97-112 / 第 97-112 行
```cpp
 97 | ATTRIBUTE_NO_SANITIZE_MEMORY
 98 | void MallocHook(const volatile void *ptr, size_t size) {
 99 |   size_t N = AllocTracer.Mallocs++;
100 |   F->HandleMalloc(size);
101 |   if (int TraceLevel = AllocTracer.TraceLevel) {
102 |     TraceLock Lock;
103 |     if (Lock.IsDisabled())
104 |       return;
105 |     Printf("MALLOC[%zd] %p %zd\n", N, ptr, size);
106 |     if (TraceLevel >= 2 && EF)
107 |       PrintStackTrace();
108 |   }
109 | }
110 | 
111 | ATTRIBUTE_NO_SANITIZE_MEMORY
112 | void FreeHook(const volatile void *ptr) {
```
- **Line 97 / 第 97 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 98 / 第 98 行**: EN: Starts the definition of function or method `MallocHook`. CN: 开始定义函数或方法 `MallocHook`。
- **Line 99 / 第 99 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 100 / 第 100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 101 / 第 101 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 102 / 第 102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 103 / 第 103 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 104 / 第 104 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 105 / 第 105 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 106 / 第 106 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 107 / 第 107 行**: EN: Declares function or method `PrintStackTrace`. CN: 声明函数或方法 `PrintStackTrace`。
- **Line 108 / 第 108 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 109 / 第 109 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 110 / 第 110 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 111 / 第 111 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 112 / 第 112 行**: EN: Starts the definition of function or method `FreeHook`. CN: 开始定义函数或方法 `FreeHook`。

### Lines 113-128 / 第 113-128 行
```cpp
113 |   size_t N = AllocTracer.Frees++;
114 |   if (int TraceLevel = AllocTracer.TraceLevel) {
115 |     TraceLock Lock;
116 |     if (Lock.IsDisabled())
117 |       return;
118 |     Printf("FREE[%zd]   %p\n", N, ptr);
119 |     if (TraceLevel >= 2 && EF)
120 |       PrintStackTrace();
121 |   }
122 | }
123 | 
124 | // Crash on a single malloc that exceeds the rss limit.
125 | void Fuzzer::HandleMalloc(size_t Size) {
126 |   if (!Options.MallocLimitMb || (Size >> 20) < (size_t)Options.MallocLimitMb)
127 |     return;
128 |   Printf("==%d== ERROR: libFuzzer: out-of-memory (malloc(%zd))\n",
```
- **Line 113 / 第 113 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 114 / 第 114 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 115 / 第 115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 116 / 第 116 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 117 / 第 117 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 118 / 第 118 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 119 / 第 119 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 120 / 第 120 行**: EN: Declares function or method `PrintStackTrace`. CN: 声明函数或方法 `PrintStackTrace`。
- **Line 121 / 第 121 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 122 / 第 122 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 123 / 第 123 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 124 / 第 124 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 125 / 第 125 行**: EN: Starts the definition of function or method `Fuzzer::HandleMalloc`. CN: 开始定义函数或方法 `Fuzzer::HandleMalloc`。
- **Line 126 / 第 126 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 127 / 第 127 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 128 / 第 128 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 129-144 / 第 129-144 行
```cpp
129 |          (int)GetPid(), Size);
130 |   Printf("   To change the out-of-memory limit use -rss_limit_mb=<N>\n\n");
131 |   PrintStackTrace();
132 |   DumpCurrentUnit("oom-");
133 |   Printf("SUMMARY: libFuzzer: out-of-memory\n");
134 |   PrintFinalStats();
135 |   _Exit(Options.OOMExitCode); // Stop right now.
136 | }
137 | 
138 | Fuzzer::Fuzzer(UserCallback CB, InputCorpus &Corpus, MutationDispatcher &MD,
139 |                const FuzzingOptions &Options)
140 |     : CB(CB), Corpus(Corpus), MD(MD), Options(Options) {
141 |   if (EF->__sanitizer_set_death_callback)
142 |     EF->__sanitizer_set_death_callback(StaticDeathCallback);
143 |   assert(!F);
144 |   F = this;
```
- **Line 129 / 第 129 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 130 / 第 130 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 131 / 第 131 行**: EN: Declares function or method `PrintStackTrace`. CN: 声明函数或方法 `PrintStackTrace`。
- **Line 132 / 第 132 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 133 / 第 133 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 134 / 第 134 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 135 / 第 135 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 136 / 第 136 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 137 / 第 137 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 138 / 第 138 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 139 / 第 139 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 140 / 第 140 行**: EN: Starts the definition of function or method `CB`. CN: 开始定义函数或方法 `CB`。
- **Line 141 / 第 141 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 142 / 第 142 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 143 / 第 143 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 144 / 第 144 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 145-160 / 第 145-160 行
```cpp
145 |   TPC.ResetMaps();
146 |   IsMyThread = true;
147 |   if (Options.DetectLeaks && EF->__sanitizer_install_malloc_and_free_hooks)
148 |     EF->__sanitizer_install_malloc_and_free_hooks(MallocHook, FreeHook);
149 |   TPC.SetUseCounters(Options.UseCounters);
150 |   TPC.SetUseValueProfileMask(Options.UseValueProfile);
151 | 
152 |   if (Options.Verbosity)
153 |     TPC.PrintModuleInfo();
154 |   if (!Options.OutputCorpus.empty() && Options.ReloadIntervalSec)
155 |     EpochOfLastReadOfOutputCorpus = GetEpoch(Options.OutputCorpus);
156 |   MaxInputLen = MaxMutationLen = Options.MaxLen;
157 |   TmpMaxMutationLen = 0;  // Will be set once we load the corpus.
158 |   AllocateCurrentUnitData();
159 |   CurrentUnitSize = 0;
160 |   memset(BaseSha1, 0, sizeof(BaseSha1));
```
- **Line 145 / 第 145 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 146 / 第 146 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 147 / 第 147 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 148 / 第 148 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 149 / 第 149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 150 / 第 150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 151 / 第 151 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 152 / 第 152 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 153 / 第 153 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 154 / 第 154 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 155 / 第 155 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 156 / 第 156 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 157 / 第 157 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 158 / 第 158 行**: EN: Declares function or method `AllocateCurrentUnitData`. CN: 声明函数或方法 `AllocateCurrentUnitData`。
- **Line 159 / 第 159 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 160 / 第 160 行**: EN: Declares function or method `memset`. CN: 声明函数或方法 `memset`。

### Lines 161-176 / 第 161-176 行
```cpp
161 | }
162 | 
163 | void Fuzzer::AllocateCurrentUnitData() {
164 |   if (CurrentUnitData || MaxInputLen == 0)
165 |     return;
166 |   CurrentUnitData = new uint8_t[MaxInputLen];
167 | }
168 | 
169 | void Fuzzer::StaticDeathCallback() {
170 |   assert(F);
171 |   F->DeathCallback();
172 | }
173 | 
174 | void Fuzzer::DumpCurrentUnit(const char *Prefix) {
175 |   if (!CurrentUnitData)
176 |     return; // Happens when running individual inputs.
```
- **Line 161 / 第 161 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 162 / 第 162 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 163 / 第 163 行**: EN: Starts the definition of function or method `Fuzzer::AllocateCurrentUnitData`. CN: 开始定义函数或方法 `Fuzzer::AllocateCurrentUnitData`。
- **Line 164 / 第 164 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 165 / 第 165 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 166 / 第 166 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 167 / 第 167 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 168 / 第 168 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 169 / 第 169 行**: EN: Starts the definition of function or method `Fuzzer::StaticDeathCallback`. CN: 开始定义函数或方法 `Fuzzer::StaticDeathCallback`。
- **Line 170 / 第 170 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 171 / 第 171 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 172 / 第 172 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 173 / 第 173 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 174 / 第 174 行**: EN: Starts the definition of function or method `Fuzzer::DumpCurrentUnit`. CN: 开始定义函数或方法 `Fuzzer::DumpCurrentUnit`。
- **Line 175 / 第 175 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 176 / 第 176 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 177-192 / 第 177-192 行
```cpp
177 |   ScopedDisableMsanInterceptorChecks S;
178 |   MD.PrintMutationSequence();
179 |   Printf("; base unit: %s\n", Sha1ToString(BaseSha1).c_str());
180 |   size_t UnitSize = CurrentUnitSize;
181 |   if (UnitSize <= kMaxUnitSizeToPrint) {
182 |     PrintHexArray(CurrentUnitData, UnitSize, "\n");
183 |     PrintASCII(CurrentUnitData, UnitSize, "\n");
184 |   }
185 |   WriteUnitToFileWithPrefix({CurrentUnitData, CurrentUnitData + UnitSize},
186 |                             Prefix);
187 | }
188 | 
189 | NO_SANITIZE_MEMORY
190 | void Fuzzer::DeathCallback() {
191 |   DumpCurrentUnit("crash-");
192 |   PrintFinalStats();
```
- **Line 177 / 第 177 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 178 / 第 178 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 179 / 第 179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 180 / 第 180 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 181 / 第 181 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 182 / 第 182 行**: EN: Declares function or method `PrintHexArray`. CN: 声明函数或方法 `PrintHexArray`。
- **Line 183 / 第 183 行**: EN: Declares function or method `PrintASCII`. CN: 声明函数或方法 `PrintASCII`。
- **Line 184 / 第 184 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 185 / 第 185 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 186 / 第 186 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 187 / 第 187 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 188 / 第 188 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 189 / 第 189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 190 / 第 190 行**: EN: Starts the definition of function or method `Fuzzer::DeathCallback`. CN: 开始定义函数或方法 `Fuzzer::DeathCallback`。
- **Line 191 / 第 191 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 192 / 第 192 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。

### Lines 193-208 / 第 193-208 行
```cpp
193 | }
194 | 
195 | void Fuzzer::StaticAlarmCallback() {
196 |   assert(F);
197 |   F->AlarmCallback();
198 | }
199 | 
200 | void Fuzzer::StaticCrashSignalCallback() {
201 |   assert(F);
202 |   F->CrashCallback();
203 | }
204 | 
205 | void Fuzzer::StaticExitCallback() {
206 |   assert(F);
207 |   F->ExitCallback();
208 | }
```
- **Line 193 / 第 193 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 194 / 第 194 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 195 / 第 195 行**: EN: Starts the definition of function or method `Fuzzer::StaticAlarmCallback`. CN: 开始定义函数或方法 `Fuzzer::StaticAlarmCallback`。
- **Line 196 / 第 196 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 197 / 第 197 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 198 / 第 198 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 199 / 第 199 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 200 / 第 200 行**: EN: Starts the definition of function or method `Fuzzer::StaticCrashSignalCallback`. CN: 开始定义函数或方法 `Fuzzer::StaticCrashSignalCallback`。
- **Line 201 / 第 201 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 202 / 第 202 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 203 / 第 203 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 204 / 第 204 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 205 / 第 205 行**: EN: Starts the definition of function or method `Fuzzer::StaticExitCallback`. CN: 开始定义函数或方法 `Fuzzer::StaticExitCallback`。
- **Line 206 / 第 206 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 207 / 第 207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 208 / 第 208 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 209-224 / 第 209-224 行
```cpp
209 | 
210 | void Fuzzer::StaticInterruptCallback() {
211 |   assert(F);
212 |   F->InterruptCallback();
213 | }
214 | 
215 | void Fuzzer::StaticGracefulExitCallback() {
216 |   assert(F);
217 |   F->GracefulExitRequested = true;
218 |   Printf("INFO: signal received, trying to exit gracefully\n");
219 | }
220 | 
221 | void Fuzzer::StaticFileSizeExceedCallback() {
222 |   Printf("==%lu== ERROR: libFuzzer: file size exceeded\n", GetPid());
223 |   exit(1);
224 | }
```
- **Line 209 / 第 209 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 210 / 第 210 行**: EN: Starts the definition of function or method `Fuzzer::StaticInterruptCallback`. CN: 开始定义函数或方法 `Fuzzer::StaticInterruptCallback`。
- **Line 211 / 第 211 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 212 / 第 212 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 213 / 第 213 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 214 / 第 214 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 215 / 第 215 行**: EN: Starts the definition of function or method `Fuzzer::StaticGracefulExitCallback`. CN: 开始定义函数或方法 `Fuzzer::StaticGracefulExitCallback`。
- **Line 216 / 第 216 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 217 / 第 217 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 218 / 第 218 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 219 / 第 219 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 220 / 第 220 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 221 / 第 221 行**: EN: Starts the definition of function or method `Fuzzer::StaticFileSizeExceedCallback`. CN: 开始定义函数或方法 `Fuzzer::StaticFileSizeExceedCallback`。
- **Line 222 / 第 222 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 223 / 第 223 行**: EN: Declares function or method `exit`. CN: 声明函数或方法 `exit`。
- **Line 224 / 第 224 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 225-240 / 第 225-240 行
```cpp
225 | 
226 | void Fuzzer::CrashCallback() {
227 |   if (EF->__sanitizer_acquire_crash_state &&
228 |       !EF->__sanitizer_acquire_crash_state())
229 |     return;
230 |   Printf("==%lu== ERROR: libFuzzer: deadly signal\n", GetPid());
231 |   PrintStackTrace();
232 |   Printf("NOTE: libFuzzer has rudimentary signal handlers.\n"
233 |          "      Combine libFuzzer with AddressSanitizer or similar for better "
234 |          "crash reports.\n");
235 |   Printf("SUMMARY: libFuzzer: deadly signal\n");
236 |   DumpCurrentUnit("crash-");
237 |   PrintFinalStats();
238 |   _Exit(Options.ErrorExitCode); // Stop right now.
239 | }
240 | 
```
- **Line 225 / 第 225 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 226 / 第 226 行**: EN: Starts the definition of function or method `Fuzzer::CrashCallback`. CN: 开始定义函数或方法 `Fuzzer::CrashCallback`。
- **Line 227 / 第 227 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 228 / 第 228 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 229 / 第 229 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 230 / 第 230 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 231 / 第 231 行**: EN: Declares function or method `PrintStackTrace`. CN: 声明函数或方法 `PrintStackTrace`。
- **Line 232 / 第 232 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 233 / 第 233 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 234 / 第 234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 235 / 第 235 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 236 / 第 236 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 237 / 第 237 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 238 / 第 238 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 239 / 第 239 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 240 / 第 240 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 241-256 / 第 241-256 行
```cpp
241 | void Fuzzer::ExitCallback() {
242 |   if (!RunningUserCallback)
243 |     return; // This exit did not come from the user callback
244 |   if (EF->__sanitizer_acquire_crash_state &&
245 |       !EF->__sanitizer_acquire_crash_state())
246 |     return;
247 |   Printf("==%lu== ERROR: libFuzzer: fuzz target exited\n", GetPid());
248 |   PrintStackTrace();
249 |   Printf("SUMMARY: libFuzzer: fuzz target exited\n");
250 |   DumpCurrentUnit("crash-");
251 |   PrintFinalStats();
252 |   _Exit(Options.ErrorExitCode);
253 | }
254 | 
255 | void Fuzzer::MaybeExitGracefully() {
256 |   if (!F->GracefulExitRequested) return;
```
- **Line 241 / 第 241 行**: EN: Starts the definition of function or method `Fuzzer::ExitCallback`. CN: 开始定义函数或方法 `Fuzzer::ExitCallback`。
- **Line 242 / 第 242 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 243 / 第 243 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 244 / 第 244 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 245 / 第 245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 246 / 第 246 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 247 / 第 247 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 248 / 第 248 行**: EN: Declares function or method `PrintStackTrace`. CN: 声明函数或方法 `PrintStackTrace`。
- **Line 249 / 第 249 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 250 / 第 250 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 251 / 第 251 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 252 / 第 252 行**: EN: Declares function or method `_Exit`. CN: 声明函数或方法 `_Exit`。
- **Line 253 / 第 253 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 254 / 第 254 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 255 / 第 255 行**: EN: Starts the definition of function or method `Fuzzer::MaybeExitGracefully`. CN: 开始定义函数或方法 `Fuzzer::MaybeExitGracefully`。
- **Line 256 / 第 256 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 257-272 / 第 257-272 行
```cpp
257 |   Printf("==%lu== INFO: libFuzzer: exiting as requested\n", GetPid());
258 |   RmDirRecursive(TempPath("FuzzWithFork", ".dir"));
259 |   F->PrintFinalStats();
260 |   _Exit(0);
261 | }
262 | 
263 | int Fuzzer::InterruptExitCode() {
264 |   assert(F);
265 |   return F->Options.InterruptExitCode;
266 | }
267 | 
268 | void Fuzzer::InterruptCallback() {
269 |   Printf("==%lu== libFuzzer: run interrupted; exiting\n", GetPid());
270 |   PrintFinalStats();
271 |   ScopedDisableMsanInterceptorChecks S; // RmDirRecursive may call opendir().
272 |   RmDirRecursive(TempPath("FuzzWithFork", ".dir"));
```
- **Line 257 / 第 257 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 258 / 第 258 行**: EN: Declares function or method `RmDirRecursive`. CN: 声明函数或方法 `RmDirRecursive`。
- **Line 259 / 第 259 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 260 / 第 260 行**: EN: Declares function or method `_Exit`. CN: 声明函数或方法 `_Exit`。
- **Line 261 / 第 261 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 262 / 第 262 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 263 / 第 263 行**: EN: Starts the definition of function or method `Fuzzer::InterruptExitCode`. CN: 开始定义函数或方法 `Fuzzer::InterruptExitCode`。
- **Line 264 / 第 264 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 265 / 第 265 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 266 / 第 266 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 267 / 第 267 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 268 / 第 268 行**: EN: Starts the definition of function or method `Fuzzer::InterruptCallback`. CN: 开始定义函数或方法 `Fuzzer::InterruptCallback`。
- **Line 269 / 第 269 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 270 / 第 270 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 271 / 第 271 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 272 / 第 272 行**: EN: Declares function or method `RmDirRecursive`. CN: 声明函数或方法 `RmDirRecursive`。

### Lines 273-288 / 第 273-288 行
```cpp
273 |   // Stop right now, don't perform any at-exit actions.
274 |   _Exit(Options.InterruptExitCode);
275 | }
276 | 
277 | NO_SANITIZE_MEMORY
278 | void Fuzzer::AlarmCallback() {
279 |   assert(Options.UnitTimeoutSec > 0);
280 |   // In Windows and Fuchsia, Alarm callback is executed by a different thread.
281 |   // NetBSD's current behavior needs this change too.
282 | #if !LIBFUZZER_WINDOWS && !LIBFUZZER_NETBSD && !LIBFUZZER_FUCHSIA
283 |   if (!InFuzzingThread())
284 |     return;
285 | #endif
286 |   if (!RunningUserCallback)
287 |     return; // We have not started running units yet.
288 |   size_t Seconds =
```
- **Line 273 / 第 273 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 274 / 第 274 行**: EN: Declares function or method `_Exit`. CN: 声明函数或方法 `_Exit`。
- **Line 275 / 第 275 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 276 / 第 276 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 277 / 第 277 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 278 / 第 278 行**: EN: Starts the definition of function or method `Fuzzer::AlarmCallback`. CN: 开始定义函数或方法 `Fuzzer::AlarmCallback`。
- **Line 279 / 第 279 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 280 / 第 280 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 281 / 第 281 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 282 / 第 282 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 283 / 第 283 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 284 / 第 284 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 285 / 第 285 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 286 / 第 286 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 287 / 第 287 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 288 / 第 288 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 289-304 / 第 289-304 行
```cpp
289 |       duration_cast<seconds>(system_clock::now() - UnitStartTime).count();
290 |   if (Seconds == 0)
291 |     return;
292 |   if (Options.Verbosity >= 2)
293 |     Printf("AlarmCallback %zd\n", Seconds);
294 |   if (Seconds >= (size_t)Options.UnitTimeoutSec) {
295 |     if (EF->__sanitizer_acquire_crash_state &&
296 |         !EF->__sanitizer_acquire_crash_state())
297 |       return;
298 |     Printf("ALARM: working on the last Unit for %zd seconds\n", Seconds);
299 |     Printf("       and the timeout value is %d (use -timeout=N to change)\n",
300 |            Options.UnitTimeoutSec);
301 |     DumpCurrentUnit("timeout-");
302 |     Printf("==%lu== ERROR: libFuzzer: timeout after %zu seconds\n", GetPid(),
303 |            Seconds);
304 |     PrintStackTrace();
```
- **Line 289 / 第 289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 290 / 第 290 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 291 / 第 291 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 292 / 第 292 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 293 / 第 293 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 294 / 第 294 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 295 / 第 295 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 296 / 第 296 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 297 / 第 297 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 298 / 第 298 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 299 / 第 299 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 300 / 第 300 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 301 / 第 301 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 302 / 第 302 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 303 / 第 303 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 304 / 第 304 行**: EN: Declares function or method `PrintStackTrace`. CN: 声明函数或方法 `PrintStackTrace`。

### Lines 305-320 / 第 305-320 行
```cpp
305 |     Printf("SUMMARY: libFuzzer: timeout\n");
306 |     PrintFinalStats();
307 |     _Exit(Options.TimeoutExitCode); // Stop right now.
308 |   }
309 | }
310 | 
311 | void Fuzzer::RssLimitCallback() {
312 |   if (EF->__sanitizer_acquire_crash_state &&
313 |       !EF->__sanitizer_acquire_crash_state())
314 |     return;
315 |   Printf("==%lu== ERROR: libFuzzer: out-of-memory (used: %zdMb; limit: %dMb)\n",
316 |          GetPid(), GetPeakRSSMb(), Options.RssLimitMb);
317 |   Printf("   To change the out-of-memory limit use -rss_limit_mb=<N>\n\n");
318 |   PrintMemoryProfile();
319 |   DumpCurrentUnit("oom-");
320 |   Printf("SUMMARY: libFuzzer: out-of-memory\n");
```
- **Line 305 / 第 305 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 306 / 第 306 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 307 / 第 307 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 308 / 第 308 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 309 / 第 309 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 310 / 第 310 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 311 / 第 311 行**: EN: Starts the definition of function or method `Fuzzer::RssLimitCallback`. CN: 开始定义函数或方法 `Fuzzer::RssLimitCallback`。
- **Line 312 / 第 312 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 313 / 第 313 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 314 / 第 314 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 315 / 第 315 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 316 / 第 316 行**: EN: Declares function or method `GetPid`. CN: 声明函数或方法 `GetPid`。
- **Line 317 / 第 317 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 318 / 第 318 行**: EN: Declares function or method `PrintMemoryProfile`. CN: 声明函数或方法 `PrintMemoryProfile`。
- **Line 319 / 第 319 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 320 / 第 320 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。

### Lines 321-336 / 第 321-336 行
```cpp
321 |   PrintFinalStats();
322 |   _Exit(Options.OOMExitCode); // Stop right now.
323 | }
324 | 
325 | void Fuzzer::PrintStats(const char *Where, const char *End, size_t Units,
326 |                         size_t Features) {
327 |   size_t ExecPerSec = execPerSec();
328 |   if (!Options.Verbosity)
329 |     return;
330 |   Printf("#%zd\t%s", TotalNumberOfRuns, Where);
331 |   if (size_t N = TPC.GetTotalPCCoverage())
332 |     Printf(" cov: %zd", N);
333 |   if (size_t N = Features ? Features : Corpus.NumFeatures())
334 |     Printf(" ft: %zd", N);
335 |   if (!Corpus.empty()) {
336 |     Printf(" corp: %zd", Corpus.NumActiveUnits());
```
- **Line 321 / 第 321 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 322 / 第 322 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 323 / 第 323 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 324 / 第 324 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 325 / 第 325 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 326 / 第 326 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 327 / 第 327 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 328 / 第 328 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 329 / 第 329 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 330 / 第 330 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 331 / 第 331 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 332 / 第 332 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 333 / 第 333 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 334 / 第 334 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 335 / 第 335 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 336 / 第 336 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。

### Lines 337-352 / 第 337-352 行
```cpp
337 |     if (size_t N = Corpus.SizeInBytes()) {
338 |       if (N < (1 << 14))
339 |         Printf("/%zdb", N);
340 |       else if (N < (1 << 24))
341 |         Printf("/%zdKb", N >> 10);
342 |       else
343 |         Printf("/%zdMb", N >> 20);
344 |     }
345 |     if (size_t FF = Corpus.NumInputsThatTouchFocusFunction())
346 |       Printf(" focus: %zd", FF);
347 |   }
348 |   if (TmpMaxMutationLen)
349 |     Printf(" lim: %zd", TmpMaxMutationLen);
350 |   if (Units)
351 |     Printf(" units: %zd", Units);
352 | 
```
- **Line 337 / 第 337 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 338 / 第 338 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 339 / 第 339 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 340 / 第 340 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 341 / 第 341 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 342 / 第 342 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 343 / 第 343 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 344 / 第 344 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 345 / 第 345 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 346 / 第 346 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 347 / 第 347 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 348 / 第 348 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 349 / 第 349 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 350 / 第 350 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 351 / 第 351 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 352 / 第 352 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 353-368 / 第 353-368 行
```cpp
353 |   Printf(" exec/s: %zd", ExecPerSec);
354 |   Printf(" rss: %zdMb", GetPeakRSSMb());
355 |   Printf("%s", End);
356 | }
357 | 
358 | void Fuzzer::PrintFinalStats() {
359 |   if (Options.PrintFullCoverage)
360 |     TPC.PrintCoverage(/*PrintAllCounters=*/true);
361 |   if (Options.PrintCoverage)
362 |     TPC.PrintCoverage(/*PrintAllCounters=*/false);
363 |   if (Options.PrintCorpusStats)
364 |     Corpus.PrintStats();
365 |   if (!Options.PrintFinalStats)
366 |     return;
367 |   size_t ExecPerSec = execPerSec();
368 |   Printf("stat::number_of_executed_units: %zd\n", TotalNumberOfRuns);
```
- **Line 353 / 第 353 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 354 / 第 354 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 355 / 第 355 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 356 / 第 356 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 357 / 第 357 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 358 / 第 358 行**: EN: Starts the definition of function or method `Fuzzer::PrintFinalStats`. CN: 开始定义函数或方法 `Fuzzer::PrintFinalStats`。
- **Line 359 / 第 359 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 360 / 第 360 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 361 / 第 361 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 362 / 第 362 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 363 / 第 363 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 364 / 第 364 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 365 / 第 365 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 366 / 第 366 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 367 / 第 367 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 368 / 第 368 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。

### Lines 369-384 / 第 369-384 行
```cpp
369 |   Printf("stat::average_exec_per_sec:     %zd\n", ExecPerSec);
370 |   Printf("stat::new_units_added:          %zd\n", NumberOfNewUnitsAdded);
371 |   Printf("stat::slowest_unit_time_sec:    %ld\n", TimeOfLongestUnitInSeconds);
372 |   Printf("stat::peak_rss_mb:              %zd\n", GetPeakRSSMb());
373 | }
374 | 
375 | void Fuzzer::SetMaxInputLen(size_t MaxInputLen) {
376 |   assert(this->MaxInputLen == 0); // Can only reset MaxInputLen from 0 to non-0.
377 |   assert(MaxInputLen);
378 |   this->MaxInputLen = MaxInputLen;
379 |   this->MaxMutationLen = MaxInputLen;
380 |   AllocateCurrentUnitData();
381 |   Printf("INFO: -max_len is not provided; "
382 |          "libFuzzer will not generate inputs larger than %zd bytes\n",
383 |          MaxInputLen);
384 | }
```
- **Line 369 / 第 369 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 370 / 第 370 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 371 / 第 371 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 372 / 第 372 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 373 / 第 373 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 374 / 第 374 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 375 / 第 375 行**: EN: Starts the definition of function or method `Fuzzer::SetMaxInputLen`. CN: 开始定义函数或方法 `Fuzzer::SetMaxInputLen`。
- **Line 376 / 第 376 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 377 / 第 377 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 378 / 第 378 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 379 / 第 379 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 380 / 第 380 行**: EN: Declares function or method `AllocateCurrentUnitData`. CN: 声明函数或方法 `AllocateCurrentUnitData`。
- **Line 381 / 第 381 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 382 / 第 382 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 383 / 第 383 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 384 / 第 384 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 385-400 / 第 385-400 行
```cpp
385 | 
386 | void Fuzzer::SetMaxMutationLen(size_t MaxMutationLen) {
387 |   assert(MaxMutationLen && MaxMutationLen <= MaxInputLen);
388 |   this->MaxMutationLen = MaxMutationLen;
389 | }
390 | 
391 | void Fuzzer::CheckExitOnSrcPosOrItem() {
392 |   if (!Options.ExitOnSrcPos.empty()) {
393 |     static auto *PCsSet = new std::set<uintptr_t>;
394 |     auto HandlePC = [&](const TracePC::PCTableEntry *TE) {
395 |       if (!PCsSet->insert(TE->PC).second)
396 |         return;
397 |       std::string Descr = DescribePC("%F %L", TE->PC + 1);
398 |       if (Descr.find(Options.ExitOnSrcPos) != std::string::npos) {
399 |         Printf("INFO: found line matching '%s', exiting.\n",
400 |                Options.ExitOnSrcPos.c_str());
```
- **Line 385 / 第 385 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 386 / 第 386 行**: EN: Starts the definition of function or method `Fuzzer::SetMaxMutationLen`. CN: 开始定义函数或方法 `Fuzzer::SetMaxMutationLen`。
- **Line 387 / 第 387 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 388 / 第 388 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 389 / 第 389 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 390 / 第 390 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 391 / 第 391 行**: EN: Starts the definition of function or method `Fuzzer::CheckExitOnSrcPosOrItem`. CN: 开始定义函数或方法 `Fuzzer::CheckExitOnSrcPosOrItem`。
- **Line 392 / 第 392 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 393 / 第 393 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 394 / 第 394 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 395 / 第 395 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 396 / 第 396 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 397 / 第 397 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 398 / 第 398 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 399 / 第 399 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 400 / 第 400 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 401-416 / 第 401-416 行
```cpp
401 |         _Exit(0);
402 |       }
403 |     };
404 |     TPC.ForEachObservedPC(HandlePC);
405 |   }
406 |   if (!Options.ExitOnItem.empty()) {
407 |     if (Corpus.HasUnit(Options.ExitOnItem)) {
408 |       Printf("INFO: found item with checksum '%s', exiting.\n",
409 |              Options.ExitOnItem.c_str());
410 |       _Exit(0);
411 |     }
412 |   }
413 | }
414 | 
415 | void Fuzzer::RereadOutputCorpus(size_t MaxSize) {
416 |   if (Options.OutputCorpus.empty() || !Options.ReloadIntervalSec)
```
- **Line 401 / 第 401 行**: EN: Declares function or method `_Exit`. CN: 声明函数或方法 `_Exit`。
- **Line 402 / 第 402 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 403 / 第 403 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 404 / 第 404 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 405 / 第 405 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 406 / 第 406 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 407 / 第 407 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 408 / 第 408 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 409 / 第 409 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 410 / 第 410 行**: EN: Declares function or method `_Exit`. CN: 声明函数或方法 `_Exit`。
- **Line 411 / 第 411 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 412 / 第 412 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 413 / 第 413 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 414 / 第 414 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 415 / 第 415 行**: EN: Starts the definition of function or method `Fuzzer::RereadOutputCorpus`. CN: 开始定义函数或方法 `Fuzzer::RereadOutputCorpus`。
- **Line 416 / 第 416 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 417-432 / 第 417-432 行
```cpp
417 |     return;
418 |   std::vector<Unit> AdditionalCorpus;
419 |   std::vector<std::string> AdditionalCorpusPaths;
420 |   ReadDirToVectorOfUnits(
421 |       Options.OutputCorpus.c_str(), &AdditionalCorpus,
422 |       &EpochOfLastReadOfOutputCorpus, MaxSize,
423 |       /*ExitOnError*/ false,
424 |       (Options.Verbosity >= 2 ? &AdditionalCorpusPaths : nullptr));
425 |   if (Options.Verbosity >= 2)
426 |     Printf("Reload: read %zd new units.\n", AdditionalCorpus.size());
427 |   bool Reloaded = false;
428 |   for (size_t i = 0; i != AdditionalCorpus.size(); ++i) {
429 |     auto &U = AdditionalCorpus[i];
430 |     if (U.size() > MaxSize)
431 |       U.resize(MaxSize);
432 |     if (!Corpus.HasUnit(U)) {
```
- **Line 417 / 第 417 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 418 / 第 418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 419 / 第 419 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 420 / 第 420 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 421 / 第 421 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 422 / 第 422 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 423 / 第 423 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 424 / 第 424 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 425 / 第 425 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 426 / 第 426 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 427 / 第 427 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 428 / 第 428 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 429 / 第 429 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 430 / 第 430 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 431 / 第 431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 432 / 第 432 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 433-448 / 第 433-448 行
```cpp
433 |       if (RunOne(U.data(), U.size())) {
434 |         CheckExitOnSrcPosOrItem();
435 |         Reloaded = true;
436 |         if (Options.Verbosity >= 2)
437 |           Printf("Reloaded %s\n", AdditionalCorpusPaths[i].c_str());
438 |       }
439 |     }
440 |   }
441 |   if (Reloaded)
442 |     PrintStats("RELOAD");
443 | }
444 | 
445 | void Fuzzer::PrintPulseAndReportSlowInput(const uint8_t *Data, size_t Size) {
446 |   auto TimeOfUnit =
447 |       duration_cast<seconds>(UnitStopTime - UnitStartTime).count();
448 |   if (!(TotalNumberOfRuns & (TotalNumberOfRuns - 1)) &&
```
- **Line 433 / 第 433 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 434 / 第 434 行**: EN: Declares function or method `CheckExitOnSrcPosOrItem`. CN: 声明函数或方法 `CheckExitOnSrcPosOrItem`。
- **Line 435 / 第 435 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 436 / 第 436 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 437 / 第 437 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 438 / 第 438 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 439 / 第 439 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 440 / 第 440 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 441 / 第 441 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 442 / 第 442 行**: EN: Declares function or method `PrintStats`. CN: 声明函数或方法 `PrintStats`。
- **Line 443 / 第 443 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 444 / 第 444 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 445 / 第 445 行**: EN: Starts the definition of function or method `Fuzzer::PrintPulseAndReportSlowInput`. CN: 开始定义函数或方法 `Fuzzer::PrintPulseAndReportSlowInput`。
- **Line 446 / 第 446 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 447 / 第 447 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 448 / 第 448 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 449-464 / 第 449-464 行
```cpp
449 |       secondsSinceProcessStartUp() >= 2)
450 |     PrintStats("pulse ");
451 |   auto Threshold =
452 |       static_cast<long>(static_cast<double>(TimeOfLongestUnitInSeconds) * 1.1);
453 |   if (TimeOfUnit > Threshold && TimeOfUnit >= Options.ReportSlowUnits) {
454 |     TimeOfLongestUnitInSeconds = TimeOfUnit;
455 |     Printf("Slowest unit: %ld s:\n", TimeOfLongestUnitInSeconds);
456 |     WriteUnitToFileWithPrefix({Data, Data + Size}, "slow-unit-");
457 |   }
458 | }
459 | 
460 | static void WriteFeatureSetToFile(const std::string &FeaturesDir,
461 |                                   const std::string &FileName,
462 |                                   const std::vector<uint32_t> &FeatureSet) {
463 |   if (FeaturesDir.empty() || FeatureSet.empty()) return;
464 |   WriteToFile(reinterpret_cast<const uint8_t *>(FeatureSet.data()),
```
- **Line 449 / 第 449 行**: EN: Starts the definition of function or method `secondsSinceProcessStartUp`. CN: 开始定义函数或方法 `secondsSinceProcessStartUp`。
- **Line 450 / 第 450 行**: EN: Declares function or method `PrintStats`. CN: 声明函数或方法 `PrintStats`。
- **Line 451 / 第 451 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 452 / 第 452 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 453 / 第 453 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 454 / 第 454 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 455 / 第 455 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 456 / 第 456 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 457 / 第 457 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 458 / 第 458 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 459 / 第 459 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 460 / 第 460 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 461 / 第 461 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 462 / 第 462 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 463 / 第 463 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 464 / 第 464 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 465-480 / 第 465-480 行
```cpp
465 |               FeatureSet.size() * sizeof(FeatureSet[0]),
466 |               DirPlusFile(FeaturesDir, FileName));
467 | }
468 | 
469 | static void RenameFeatureSetFile(const std::string &FeaturesDir,
470 |                                  const std::string &OldFile,
471 |                                  const std::string &NewFile) {
472 |   if (FeaturesDir.empty()) return;
473 |   RenameFile(DirPlusFile(FeaturesDir, OldFile),
474 |              DirPlusFile(FeaturesDir, NewFile));
475 | }
476 | 
477 | static void WriteEdgeToMutationGraphFile(const std::string &MutationGraphFile,
478 |                                          const InputInfo *II,
479 |                                          const InputInfo *BaseII,
480 |                                          const std::string &MS) {
```
- **Line 465 / 第 465 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 466 / 第 466 行**: EN: Declares function or method `DirPlusFile`. CN: 声明函数或方法 `DirPlusFile`。
- **Line 467 / 第 467 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 468 / 第 468 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 469 / 第 469 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 470 / 第 470 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 471 / 第 471 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 472 / 第 472 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 473 / 第 473 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 474 / 第 474 行**: EN: Declares function or method `DirPlusFile`. CN: 声明函数或方法 `DirPlusFile`。
- **Line 475 / 第 475 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 476 / 第 476 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 477 / 第 477 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 478 / 第 478 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 479 / 第 479 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 480 / 第 480 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 481-496 / 第 481-496 行
```cpp
481 |   if (MutationGraphFile.empty())
482 |     return;
483 | 
484 |   std::string Sha1 = Sha1ToString(II->Sha1);
485 | 
486 |   std::string OutputString;
487 | 
488 |   // Add a new vertex.
489 |   OutputString.append("\"");
490 |   OutputString.append(Sha1);
491 |   OutputString.append("\"\n");
492 | 
493 |   // Add a new edge if there is base input.
494 |   if (BaseII) {
495 |     std::string BaseSha1 = Sha1ToString(BaseII->Sha1);
496 |     OutputString.append("\"");
```
- **Line 481 / 第 481 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 482 / 第 482 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 483 / 第 483 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 484 / 第 484 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 485 / 第 485 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 486 / 第 486 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 487 / 第 487 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 488 / 第 488 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 489 / 第 489 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 490 / 第 490 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 491 / 第 491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 492 / 第 492 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 493 / 第 493 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 494 / 第 494 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 495 / 第 495 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 496 / 第 496 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 497-512 / 第 497-512 行
```cpp
497 |     OutputString.append(BaseSha1);
498 |     OutputString.append("\" -> \"");
499 |     OutputString.append(Sha1);
500 |     OutputString.append("\" [label=\"");
501 |     OutputString.append(MS);
502 |     OutputString.append("\"];\n");
503 |   }
504 | 
505 |   AppendToFile(OutputString, MutationGraphFile);
506 | }
507 | 
508 | bool Fuzzer::RunOne(const uint8_t *Data, size_t Size, bool MayDeleteFile,
509 |                     InputInfo *II, bool ForceAddToCorpus,
510 |                     bool *FoundUniqFeatures) {
511 |   if (!Size)
512 |     return false;
```
- **Line 497 / 第 497 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 498 / 第 498 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 499 / 第 499 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 500 / 第 500 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 501 / 第 501 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 502 / 第 502 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 503 / 第 503 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 504 / 第 504 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 505 / 第 505 行**: EN: Declares function or method `AppendToFile`. CN: 声明函数或方法 `AppendToFile`。
- **Line 506 / 第 506 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 507 / 第 507 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 508 / 第 508 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 509 / 第 509 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 510 / 第 510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 511 / 第 511 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 512 / 第 512 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 513-528 / 第 513-528 行
```cpp
513 |   // Largest input length should be INT_MAX.
514 |   assert(Size < std::numeric_limits<uint32_t>::max());
515 | 
516 |   if(!ExecuteCallback(Data, Size)) return false;
517 |   auto TimeOfUnit = duration_cast<microseconds>(UnitStopTime - UnitStartTime);
518 | 
519 |   UniqFeatureSetTmp.clear();
520 |   size_t FoundUniqFeaturesOfII = 0;
521 |   size_t NumUpdatesBefore = Corpus.NumFeatureUpdates();
522 |   TPC.CollectFeatures([&](uint32_t Feature) {
523 |     if (Corpus.AddFeature(Feature, static_cast<uint32_t>(Size), Options.Shrink))
524 |       UniqFeatureSetTmp.push_back(Feature);
525 |     if (Options.Entropic)
526 |       Corpus.UpdateFeatureFrequency(II, Feature);
527 |     if (Options.ReduceInputs && II && !II->NeverReduce)
528 |       if (std::binary_search(II->UniqFeatureSet.begin(),
```
- **Line 513 / 第 513 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 514 / 第 514 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 515 / 第 515 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 516 / 第 516 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 517 / 第 517 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 518 / 第 518 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 519 / 第 519 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 520 / 第 520 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 521 / 第 521 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 522 / 第 522 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 523 / 第 523 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 524 / 第 524 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 525 / 第 525 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 526 / 第 526 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 527 / 第 527 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 528 / 第 528 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 529-544 / 第 529-544 行
```cpp
529 |                              II->UniqFeatureSet.end(), Feature))
530 |         FoundUniqFeaturesOfII++;
531 |   });
532 |   if (FoundUniqFeatures)
533 |     *FoundUniqFeatures = FoundUniqFeaturesOfII;
534 |   PrintPulseAndReportSlowInput(Data, Size);
535 |   size_t NumNewFeatures = Corpus.NumFeatureUpdates() - NumUpdatesBefore;
536 |   if (NumNewFeatures || ForceAddToCorpus) {
537 |     TPC.UpdateObservedPCs();
538 |     auto NewII =
539 |         Corpus.AddToCorpus({Data, Data + Size}, NumNewFeatures, MayDeleteFile,
540 |                            TPC.ObservedFocusFunction(), ForceAddToCorpus,
541 |                            TimeOfUnit, UniqFeatureSetTmp, DFT, II);
542 |     WriteFeatureSetToFile(Options.FeaturesDir, Sha1ToString(NewII->Sha1),
543 |                           NewII->UniqFeatureSet);
544 |     WriteEdgeToMutationGraphFile(Options.MutationGraphFile, NewII, II,
```
- **Line 529 / 第 529 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 530 / 第 530 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 531 / 第 531 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 532 / 第 532 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 533 / 第 533 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 534 / 第 534 行**: EN: Declares function or method `PrintPulseAndReportSlowInput`. CN: 声明函数或方法 `PrintPulseAndReportSlowInput`。
- **Line 535 / 第 535 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 536 / 第 536 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 537 / 第 537 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 538 / 第 538 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 539 / 第 539 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 540 / 第 540 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 541 / 第 541 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 542 / 第 542 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 543 / 第 543 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 544 / 第 544 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 545-560 / 第 545-560 行
```cpp
545 |                                  MD.MutationSequence());
546 |     return true;
547 |   }
548 |   if (II && FoundUniqFeaturesOfII &&
549 |       II->DataFlowTraceForFocusFunction.empty() &&
550 |       FoundUniqFeaturesOfII == II->UniqFeatureSet.size() &&
551 |       II->U.size() > Size) {
552 |     auto OldFeaturesFile = Sha1ToString(II->Sha1);
553 |     Corpus.Replace(II, {Data, Data + Size}, TimeOfUnit);
554 |     RenameFeatureSetFile(Options.FeaturesDir, OldFeaturesFile,
555 |                          Sha1ToString(II->Sha1));
556 |     return true;
557 |   }
558 |   return false;
559 | }
560 | 
```
- **Line 545 / 第 545 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 546 / 第 546 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 547 / 第 547 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 548 / 第 548 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 549 / 第 549 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 550 / 第 550 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 551 / 第 551 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 552 / 第 552 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 553 / 第 553 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 554 / 第 554 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 555 / 第 555 行**: EN: Declares function or method `Sha1ToString`. CN: 声明函数或方法 `Sha1ToString`。
- **Line 556 / 第 556 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 557 / 第 557 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 558 / 第 558 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 559 / 第 559 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 560 / 第 560 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 561-576 / 第 561-576 行
```cpp
561 | void Fuzzer::TPCUpdateObservedPCs() { TPC.UpdateObservedPCs(); }
562 | 
563 | size_t Fuzzer::GetCurrentUnitInFuzzingThead(const uint8_t **Data) const {
564 |   assert(InFuzzingThread());
565 |   *Data = CurrentUnitData;
566 |   return CurrentUnitSize;
567 | }
568 | 
569 | void Fuzzer::CrashOnOverwrittenData() {
570 |   Printf("==%d== ERROR: libFuzzer: fuzz target overwrites its const input\n",
571 |          (int)GetPid());
572 |   PrintStackTrace();
573 |   Printf("SUMMARY: libFuzzer: overwrites-const-input\n");
574 |   DumpCurrentUnit("crash-");
575 |   PrintFinalStats();
576 |   _Exit(Options.ErrorExitCode); // Stop right now.
```
- **Line 561 / 第 561 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 562 / 第 562 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 563 / 第 563 行**: EN: Starts the definition of function or method `Fuzzer::GetCurrentUnitInFuzzingThead`. CN: 开始定义函数或方法 `Fuzzer::GetCurrentUnitInFuzzingThead`。
- **Line 564 / 第 564 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 565 / 第 565 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 566 / 第 566 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 567 / 第 567 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 568 / 第 568 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 569 / 第 569 行**: EN: Starts the definition of function or method `Fuzzer::CrashOnOverwrittenData`. CN: 开始定义函数或方法 `Fuzzer::CrashOnOverwrittenData`。
- **Line 570 / 第 570 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 571 / 第 571 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 572 / 第 572 行**: EN: Declares function or method `PrintStackTrace`. CN: 声明函数或方法 `PrintStackTrace`。
- **Line 573 / 第 573 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 574 / 第 574 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 575 / 第 575 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 576 / 第 576 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 577-592 / 第 577-592 行
```cpp
577 | }
578 | 
579 | // Compare two arrays, but not all bytes if the arrays are large.
580 | static bool LooseMemeq(const uint8_t *A, const uint8_t *B, size_t Size) {
581 |   const size_t Limit = 64;
582 |   // memcmp cannot take null pointer arguments even if Size is 0.
583 |   if (!Size)
584 |     return true;
585 |   if (Size <= 64)
586 |     return !memcmp(A, B, Size);
587 |   // Compare first and last Limit/2 bytes.
588 |   return !memcmp(A, B, Limit / 2) &&
589 |          !memcmp(A + Size - Limit / 2, B + Size - Limit / 2, Limit / 2);
590 | }
591 | 
592 | // This method is not inlined because it would cause a test to fail where it
```
- **Line 577 / 第 577 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 578 / 第 578 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 579 / 第 579 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 580 / 第 580 行**: EN: Starts the definition of function or method `LooseMemeq`. CN: 开始定义函数或方法 `LooseMemeq`。
- **Line 581 / 第 581 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 582 / 第 582 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 583 / 第 583 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 584 / 第 584 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 585 / 第 585 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 586 / 第 586 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 587 / 第 587 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 588 / 第 588 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 589 / 第 589 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 590 / 第 590 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 591 / 第 591 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 592 / 第 592 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 593-608 / 第 593-608 行
```cpp
593 | // is part of the stack unwinding. See D97975 for details.
594 | ATTRIBUTE_NOINLINE bool Fuzzer::ExecuteCallback(const uint8_t *Data,
595 |                                                 size_t Size) {
596 |   TPC.RecordInitialStack();
597 |   TotalNumberOfRuns++;
598 |   assert(InFuzzingThread());
599 |   // We copy the contents of Unit into a separate heap buffer
600 |   // so that we reliably find buffer overflows in it.
601 |   uint8_t *DataCopy = new uint8_t[Size];
602 |   // memcpy cannot take null pointer arguments even if Size is 0.
603 |   if (Size)
604 |     memcpy(DataCopy, Data, Size);
605 |   if (EF->__msan_unpoison)
606 |     EF->__msan_unpoison(DataCopy, Size);
607 |   if (EF->__msan_unpoison_param)
608 |     EF->__msan_unpoison_param(2);
```
- **Line 593 / 第 593 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 594 / 第 594 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 595 / 第 595 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 596 / 第 596 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 597 / 第 597 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 598 / 第 598 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 599 / 第 599 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 600 / 第 600 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 601 / 第 601 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 602 / 第 602 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 603 / 第 603 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 604 / 第 604 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 605 / 第 605 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 606 / 第 606 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 607 / 第 607 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 608 / 第 608 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 609-624 / 第 609-624 行
```cpp
609 |   if (CurrentUnitData && CurrentUnitData != Data)
610 |     memcpy(CurrentUnitData, Data, Size);
611 |   CurrentUnitSize = Size;
612 |   int CBRes = 0;
613 |   {
614 |     ScopedEnableMsanInterceptorChecks S;
615 |     AllocTracer.Start(Options.TraceMalloc);
616 |     UnitStartTime = system_clock::now();
617 |     TPC.ResetMaps();
618 |     RunningUserCallback = true;
619 |     CBRes = CB(DataCopy, Size);
620 |     RunningUserCallback = false;
621 |     UnitStopTime = system_clock::now();
622 |     assert(CBRes == 0 || CBRes == -1);
623 |     HasMoreMallocsThanFrees = AllocTracer.Stop();
624 |   }
```
- **Line 609 / 第 609 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 610 / 第 610 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 611 / 第 611 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 612 / 第 612 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 613 / 第 613 行**: EN: Opens a new scope or block. CN: 打开新的作用域或代码块。
- **Line 614 / 第 614 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 615 / 第 615 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 616 / 第 616 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 617 / 第 617 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 618 / 第 618 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 619 / 第 619 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 620 / 第 620 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 621 / 第 621 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 622 / 第 622 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 623 / 第 623 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 624 / 第 624 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 625-640 / 第 625-640 行
```cpp
625 |   if (!LooseMemeq(DataCopy, Data, Size))
626 |     CrashOnOverwrittenData();
627 |   CurrentUnitSize = 0;
628 |   delete[] DataCopy;
629 |   return CBRes == 0;
630 | }
631 | 
632 | std::string Fuzzer::WriteToOutputCorpus(const Unit &U) {
633 |   if (Options.OnlyASCII)
634 |     assert(IsASCII(U));
635 |   if (Options.OutputCorpus.empty())
636 |     return "";
637 |   std::string Path = DirPlusFile(Options.OutputCorpus, Hash(U));
638 |   WriteToFile(U, Path);
639 |   if (Options.Verbosity >= 2)
640 |     Printf("Written %zd bytes to %s\n", U.size(), Path.c_str());
```
- **Line 625 / 第 625 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 626 / 第 626 行**: EN: Declares function or method `CrashOnOverwrittenData`. CN: 声明函数或方法 `CrashOnOverwrittenData`。
- **Line 627 / 第 627 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 628 / 第 628 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 629 / 第 629 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 630 / 第 630 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 631 / 第 631 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 632 / 第 632 行**: EN: Starts the definition of function or method `Fuzzer::WriteToOutputCorpus`. CN: 开始定义函数或方法 `Fuzzer::WriteToOutputCorpus`。
- **Line 633 / 第 633 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 634 / 第 634 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 635 / 第 635 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 636 / 第 636 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 637 / 第 637 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 638 / 第 638 行**: EN: Declares function or method `WriteToFile`. CN: 声明函数或方法 `WriteToFile`。
- **Line 639 / 第 639 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 640 / 第 640 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。

### Lines 641-656 / 第 641-656 行
```cpp
641 |   return Path;
642 | }
643 | 
644 | void Fuzzer::WriteUnitToFileWithPrefix(const Unit &U, const char *Prefix) {
645 |   if (!Options.SaveArtifacts)
646 |     return;
647 |   std::string Path = Options.ArtifactPrefix + Prefix + Hash(U);
648 |   if (!Options.ExactArtifactPath.empty())
649 |     Path = Options.ExactArtifactPath; // Overrides ArtifactPrefix.
650 |   WriteToFile(U, Path);
651 |   Printf("artifact_prefix='%s'; Test unit written to %s\n",
652 |          Options.ArtifactPrefix.c_str(), Path.c_str());
653 |   if (U.size() <= kMaxUnitSizeToPrint)
654 |     Printf("Base64: %s\n", Base64(U).c_str());
655 | }
656 | 
```
- **Line 641 / 第 641 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 642 / 第 642 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 643 / 第 643 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 644 / 第 644 行**: EN: Starts the definition of function or method `Fuzzer::WriteUnitToFileWithPrefix`. CN: 开始定义函数或方法 `Fuzzer::WriteUnitToFileWithPrefix`。
- **Line 645 / 第 645 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 646 / 第 646 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 647 / 第 647 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 648 / 第 648 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 649 / 第 649 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 650 / 第 650 行**: EN: Declares function or method `WriteToFile`. CN: 声明函数或方法 `WriteToFile`。
- **Line 651 / 第 651 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 652 / 第 652 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 653 / 第 653 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 654 / 第 654 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 655 / 第 655 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 656 / 第 656 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 657-672 / 第 657-672 行
```cpp
657 | void Fuzzer::PrintStatusForNewUnit(const Unit &U, const char *Text) {
658 |   if (!Options.PrintNEW)
659 |     return;
660 |   PrintStats(Text, "");
661 |   if (Options.Verbosity) {
662 |     Printf(" L: %zd/%zd ", U.size(), Corpus.MaxInputSize());
663 |     MD.PrintMutationSequence(Options.Verbosity >= 2);
664 |     Printf("\n");
665 |   }
666 | }
667 | 
668 | void Fuzzer::ReportNewCoverage(InputInfo *II, const Unit &U) {
669 |   II->NumSuccessfulMutations++;
670 |   MD.RecordSuccessfulMutationSequence();
671 |   PrintStatusForNewUnit(U, II->Reduced ? "REDUCE" : "NEW   ");
672 |   WriteToOutputCorpus(U);
```
- **Line 657 / 第 657 行**: EN: Starts the definition of function or method `Fuzzer::PrintStatusForNewUnit`. CN: 开始定义函数或方法 `Fuzzer::PrintStatusForNewUnit`。
- **Line 658 / 第 658 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 659 / 第 659 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 660 / 第 660 行**: EN: Declares function or method `PrintStats`. CN: 声明函数或方法 `PrintStats`。
- **Line 661 / 第 661 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 662 / 第 662 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 663 / 第 663 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 664 / 第 664 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 665 / 第 665 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 666 / 第 666 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 667 / 第 667 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 668 / 第 668 行**: EN: Starts the definition of function or method `Fuzzer::ReportNewCoverage`. CN: 开始定义函数或方法 `Fuzzer::ReportNewCoverage`。
- **Line 669 / 第 669 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 670 / 第 670 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 671 / 第 671 行**: EN: Declares function or method `PrintStatusForNewUnit`. CN: 声明函数或方法 `PrintStatusForNewUnit`。
- **Line 672 / 第 672 行**: EN: Declares function or method `WriteToOutputCorpus`. CN: 声明函数或方法 `WriteToOutputCorpus`。

### Lines 673-688 / 第 673-688 行
```cpp
673 |   NumberOfNewUnitsAdded++;
674 |   CheckExitOnSrcPosOrItem(); // Check only after the unit is saved to corpus.
675 |   LastCorpusUpdateRun = TotalNumberOfRuns;
676 | }
677 | 
678 | // Tries detecting a memory leak on the particular input that we have just
679 | // executed before calling this function.
680 | void Fuzzer::TryDetectingAMemoryLeak(const uint8_t *Data, size_t Size,
681 |                                      bool DuringInitialCorpusExecution) {
682 |   if (!HasMoreMallocsThanFrees)
683 |     return; // mallocs==frees, a leak is unlikely.
684 |   if (!Options.DetectLeaks)
685 |     return;
686 |   if (!DuringInitialCorpusExecution &&
687 |       TotalNumberOfRuns >= Options.MaxNumberOfRuns)
688 |     return;
```
- **Line 673 / 第 673 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 674 / 第 674 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 675 / 第 675 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 676 / 第 676 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 677 / 第 677 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 678 / 第 678 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 679 / 第 679 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 680 / 第 680 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 681 / 第 681 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 682 / 第 682 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 683 / 第 683 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 684 / 第 684 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 685 / 第 685 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 686 / 第 686 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 687 / 第 687 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 688 / 第 688 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 689-704 / 第 689-704 行
```cpp
689 |   if (!&(EF->__lsan_enable) || !&(EF->__lsan_disable) ||
690 |       !(EF->__lsan_do_recoverable_leak_check))
691 |     return; // No lsan.
692 |   // Run the target once again, but with lsan disabled so that if there is
693 |   // a real leak we do not report it twice.
694 |   EF->__lsan_disable();
695 |   ExecuteCallback(Data, Size);
696 |   EF->__lsan_enable();
697 |   if (!HasMoreMallocsThanFrees)
698 |     return; // a leak is unlikely.
699 |   if (NumberOfLeakDetectionAttempts++ > 1000) {
700 |     Options.DetectLeaks = false;
701 |     Printf("INFO: libFuzzer disabled leak detection after every mutation.\n"
702 |            "      Most likely the target function accumulates allocated\n"
703 |            "      memory in a global state w/o actually leaking it.\n"
704 |            "      You may try running this binary with -trace_malloc=[12]"
```
- **Line 689 / 第 689 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 690 / 第 690 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 691 / 第 691 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 692 / 第 692 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 693 / 第 693 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 694 / 第 694 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 695 / 第 695 行**: EN: Declares function or method `ExecuteCallback`. CN: 声明函数或方法 `ExecuteCallback`。
- **Line 696 / 第 696 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 697 / 第 697 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 698 / 第 698 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 699 / 第 699 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 700 / 第 700 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 701 / 第 701 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 702 / 第 702 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 703 / 第 703 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 704 / 第 704 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 705-720 / 第 705-720 行
```cpp
705 |            "      to get a trace of mallocs and frees.\n"
706 |            "      If LeakSanitizer is enabled in this process it will still\n"
707 |            "      run on the process shutdown.\n");
708 |     return;
709 |   }
710 |   // Now perform the actual lsan pass. This is expensive and we must ensure
711 |   // we don't call it too often.
712 |   if (EF->__lsan_do_recoverable_leak_check()) { // Leak is found, report it.
713 |     if (DuringInitialCorpusExecution)
714 |       Printf("\nINFO: a leak has been found in the initial corpus.\n\n");
715 |     Printf("INFO: to ignore leaks on libFuzzer side use -detect_leaks=0.\n\n");
716 |     CurrentUnitSize = Size;
717 |     DumpCurrentUnit("leak-");
718 |     PrintFinalStats();
719 |     _Exit(Options.ErrorExitCode); // not exit() to disable lsan further on.
720 |   }
```
- **Line 705 / 第 705 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 706 / 第 706 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 707 / 第 707 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 708 / 第 708 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 709 / 第 709 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 710 / 第 710 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 711 / 第 711 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 712 / 第 712 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 713 / 第 713 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 714 / 第 714 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 715 / 第 715 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 716 / 第 716 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 717 / 第 717 行**: EN: Declares function or method `DumpCurrentUnit`. CN: 声明函数或方法 `DumpCurrentUnit`。
- **Line 718 / 第 718 行**: EN: Declares function or method `PrintFinalStats`. CN: 声明函数或方法 `PrintFinalStats`。
- **Line 719 / 第 719 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 720 / 第 720 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 721-736 / 第 721-736 行
```cpp
721 | }
722 | 
723 | void Fuzzer::MutateAndTestOne() {
724 |   MD.StartMutationSequence();
725 | 
726 |   auto &II = Corpus.ChooseUnitToMutate(MD.GetRand());
727 |   if (Options.DoCrossOver) {
728 |     auto &CrossOverII = Corpus.ChooseUnitToCrossOverWith(
729 |         MD.GetRand(), Options.CrossOverUniformDist);
730 |     MD.SetCrossOverWith(&CrossOverII.U);
731 |   }
732 |   const auto &U = II.U;
733 |   memcpy(BaseSha1, II.Sha1, sizeof(BaseSha1));
734 |   assert(CurrentUnitData);
735 |   size_t Size = U.size();
736 |   assert(Size <= MaxInputLen && "Oversized Unit");
```
- **Line 721 / 第 721 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 722 / 第 722 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 723 / 第 723 行**: EN: Starts the definition of function or method `Fuzzer::MutateAndTestOne`. CN: 开始定义函数或方法 `Fuzzer::MutateAndTestOne`。
- **Line 724 / 第 724 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 725 / 第 725 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 726 / 第 726 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 727 / 第 727 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 728 / 第 728 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 729 / 第 729 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 730 / 第 730 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 731 / 第 731 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 732 / 第 732 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 733 / 第 733 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 734 / 第 734 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 735 / 第 735 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 736 / 第 736 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。

### Lines 737-752 / 第 737-752 行
```cpp
737 |   memcpy(CurrentUnitData, U.data(), Size);
738 | 
739 |   assert(MaxMutationLen > 0);
740 | 
741 |   size_t CurrentMaxMutationLen =
742 |       Min(MaxMutationLen, Max(U.size(), TmpMaxMutationLen));
743 |   assert(CurrentMaxMutationLen > 0);
744 | 
745 |   for (int i = 0; i < Options.MutateDepth; i++) {
746 |     if (TotalNumberOfRuns >= Options.MaxNumberOfRuns)
747 |       break;
748 |     MaybeExitGracefully();
749 |     size_t NewSize = 0;
750 |     if (II.HasFocusFunction && !II.DataFlowTraceForFocusFunction.empty() &&
751 |         Size <= CurrentMaxMutationLen)
752 |       NewSize = MD.MutateWithMask(CurrentUnitData, Size, Size,
```
- **Line 737 / 第 737 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 738 / 第 738 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 739 / 第 739 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 740 / 第 740 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 741 / 第 741 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 742 / 第 742 行**: EN: Declares function or method `Min`. CN: 声明函数或方法 `Min`。
- **Line 743 / 第 743 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 744 / 第 744 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 745 / 第 745 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 746 / 第 746 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 747 / 第 747 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 748 / 第 748 行**: EN: Declares function or method `MaybeExitGracefully`. CN: 声明函数或方法 `MaybeExitGracefully`。
- **Line 749 / 第 749 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 750 / 第 750 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 751 / 第 751 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 752 / 第 752 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 753-768 / 第 753-768 行
```cpp
753 |                                   II.DataFlowTraceForFocusFunction);
754 | 
755 |     // If MutateWithMask either failed or wasn't called, call default Mutate.
756 |     if (!NewSize)
757 |       NewSize = MD.Mutate(CurrentUnitData, Size, CurrentMaxMutationLen);
758 |     assert(NewSize > 0 && "Mutator returned empty unit");
759 |     assert(NewSize <= CurrentMaxMutationLen && "Mutator return oversized unit");
760 |     Size = NewSize;
761 |     II.NumExecutedMutations++;
762 |     Corpus.IncrementNumExecutedMutations();
763 | 
764 |     bool FoundUniqFeatures = false;
765 |     bool NewCov = RunOne(CurrentUnitData, Size, /*MayDeleteFile=*/true, &II,
766 |                          /*ForceAddToCorpus*/ false, &FoundUniqFeatures);
767 |     TryDetectingAMemoryLeak(CurrentUnitData, Size,
768 |                             /*DuringInitialCorpusExecution*/ false);
```
- **Line 753 / 第 753 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 754 / 第 754 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 755 / 第 755 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 756 / 第 756 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 757 / 第 757 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 758 / 第 758 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 759 / 第 759 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 760 / 第 760 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 761 / 第 761 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 762 / 第 762 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 763 / 第 763 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 764 / 第 764 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 765 / 第 765 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 766 / 第 766 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 767 / 第 767 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 768 / 第 768 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 769-784 / 第 769-784 行
```cpp
769 |     if (NewCov) {
770 |       ReportNewCoverage(&II, {CurrentUnitData, CurrentUnitData + Size});
771 |       break;  // We will mutate this input more in the next rounds.
772 |     }
773 |     if (Options.ReduceDepth && !FoundUniqFeatures)
774 |       break;
775 |   }
776 | 
777 |   II.NeedsEnergyUpdate = true;
778 | }
779 | 
780 | void Fuzzer::PurgeAllocator() {
781 |   if (Options.PurgeAllocatorIntervalSec < 0 || !EF->__sanitizer_purge_allocator)
782 |     return;
783 |   if (duration_cast<seconds>(system_clock::now() -
784 |                              LastAllocatorPurgeAttemptTime)
```
- **Line 769 / 第 769 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 770 / 第 770 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 771 / 第 771 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 772 / 第 772 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 773 / 第 773 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 774 / 第 774 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 775 / 第 775 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 776 / 第 776 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 777 / 第 777 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 778 / 第 778 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 779 / 第 779 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 780 / 第 780 行**: EN: Starts the definition of function or method `Fuzzer::PurgeAllocator`. CN: 开始定义函数或方法 `Fuzzer::PurgeAllocator`。
- **Line 781 / 第 781 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 782 / 第 782 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 783 / 第 783 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 784 / 第 784 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 785-800 / 第 785-800 行
```cpp
785 |           .count() < Options.PurgeAllocatorIntervalSec)
786 |     return;
787 | 
788 |   if (Options.RssLimitMb <= 0 ||
789 |       GetPeakRSSMb() > static_cast<size_t>(Options.RssLimitMb) / 2)
790 |     EF->__sanitizer_purge_allocator();
791 | 
792 |   LastAllocatorPurgeAttemptTime = system_clock::now();
793 | }
794 | 
795 | void Fuzzer::ReadAndExecuteSeedCorpora(std::vector<SizedFile> &CorporaFiles) {
796 |   const size_t kMaxSaneLen = 1 << 20;
797 |   const size_t kMinDefaultLen = 4096;
798 |   size_t MaxSize = 0;
799 |   size_t MinSize = -1;
800 |   size_t TotalSize = 0;
```
- **Line 785 / 第 785 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 786 / 第 786 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 787 / 第 787 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 788 / 第 788 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 789 / 第 789 行**: EN: Starts the definition of function or method `GetPeakRSSMb`. CN: 开始定义函数或方法 `GetPeakRSSMb`。
- **Line 790 / 第 790 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 791 / 第 791 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 792 / 第 792 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 793 / 第 793 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 794 / 第 794 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 795 / 第 795 行**: EN: Starts the definition of function or method `Fuzzer::ReadAndExecuteSeedCorpora`. CN: 开始定义函数或方法 `Fuzzer::ReadAndExecuteSeedCorpora`。
- **Line 796 / 第 796 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 797 / 第 797 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 798 / 第 798 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 799 / 第 799 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 800 / 第 800 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 801-816 / 第 801-816 行
```cpp
801 |   for (auto &File : CorporaFiles) {
802 |     MaxSize = Max(File.Size, MaxSize);
803 |     MinSize = Min(File.Size, MinSize);
804 |     TotalSize += File.Size;
805 |   }
806 |   if (Options.MaxLen == 0)
807 |     SetMaxInputLen(std::clamp(MaxSize, kMinDefaultLen, kMaxSaneLen));
808 |   assert(MaxInputLen > 0);
809 | 
810 |   // Test the callback with empty input and never try it again.
811 |   uint8_t dummy = 0;
812 |   ExecuteCallback(&dummy, 0);
813 | 
814 |   if (CorporaFiles.empty()) {
815 |     Printf("INFO: A corpus is not provided, starting from an empty corpus\n");
816 |     Unit U({'\n'}); // Valid ASCII input.
```
- **Line 801 / 第 801 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 802 / 第 802 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 803 / 第 803 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 804 / 第 804 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 805 / 第 805 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 806 / 第 806 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 807 / 第 807 行**: EN: Declares function or method `SetMaxInputLen`. CN: 声明函数或方法 `SetMaxInputLen`。
- **Line 808 / 第 808 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 809 / 第 809 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 810 / 第 810 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 811 / 第 811 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 812 / 第 812 行**: EN: Declares function or method `ExecuteCallback`. CN: 声明函数或方法 `ExecuteCallback`。
- **Line 813 / 第 813 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 814 / 第 814 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 815 / 第 815 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 816 / 第 816 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 817-832 / 第 817-832 行
```cpp
817 |     RunOne(U.data(), U.size());
818 |   } else {
819 |     Printf("INFO: seed corpus: files: %zd min: %zdb max: %zdb total: %zdb"
820 |            " rss: %zdMb\n",
821 |            CorporaFiles.size(), MinSize, MaxSize, TotalSize, GetPeakRSSMb());
822 |     if (Options.ShuffleAtStartUp)
823 |       std::shuffle(CorporaFiles.begin(), CorporaFiles.end(), MD.GetRand());
824 | 
825 |     if (Options.PreferSmall) {
826 |       std::stable_sort(CorporaFiles.begin(), CorporaFiles.end());
827 |       assert(CorporaFiles.front().Size <= CorporaFiles.back().Size);
828 |     }
829 | 
830 |     // Load and execute inputs one by one.
831 |     for (auto &SF : CorporaFiles) {
832 |       auto U = FileToVector(SF.File, MaxInputLen, /*ExitOnError=*/false);
```
- **Line 817 / 第 817 行**: EN: Declares function or method `RunOne`. CN: 声明函数或方法 `RunOne`。
- **Line 818 / 第 818 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 819 / 第 819 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 820 / 第 820 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 821 / 第 821 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 822 / 第 822 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 823 / 第 823 行**: EN: Declares function or method `std::shuffle`. CN: 声明函数或方法 `std::shuffle`。
- **Line 824 / 第 824 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 825 / 第 825 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 826 / 第 826 行**: EN: Declares function or method `std::stable_sort`. CN: 声明函数或方法 `std::stable_sort`。
- **Line 827 / 第 827 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 828 / 第 828 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 829 / 第 829 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 830 / 第 830 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 831 / 第 831 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 832 / 第 832 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 833-848 / 第 833-848 行
```cpp
833 |       assert(U.size() <= MaxInputLen);
834 |       RunOne(U.data(), U.size(), /*MayDeleteFile*/ false, /*II*/ nullptr,
835 |              /*ForceAddToCorpus*/ Options.KeepSeed,
836 |              /*FoundUniqFeatures*/ nullptr);
837 |       CheckExitOnSrcPosOrItem();
838 |       TryDetectingAMemoryLeak(U.data(), U.size(),
839 |                               /*DuringInitialCorpusExecution*/ true);
840 |     }
841 |   }
842 | 
843 |   PrintStats("INITED");
844 |   if (!Options.FocusFunction.empty()) {
845 |     Printf("INFO: %zd/%zd inputs touch the focus function\n",
846 |            Corpus.NumInputsThatTouchFocusFunction(), Corpus.size());
847 |     if (!Options.DataFlowTrace.empty())
848 |       Printf("INFO: %zd/%zd inputs have the Data Flow Trace\n",
```
- **Line 833 / 第 833 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 834 / 第 834 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 835 / 第 835 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 836 / 第 836 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 837 / 第 837 行**: EN: Declares function or method `CheckExitOnSrcPosOrItem`. CN: 声明函数或方法 `CheckExitOnSrcPosOrItem`。
- **Line 838 / 第 838 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 839 / 第 839 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 840 / 第 840 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 841 / 第 841 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 842 / 第 842 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 843 / 第 843 行**: EN: Declares function or method `PrintStats`. CN: 声明函数或方法 `PrintStats`。
- **Line 844 / 第 844 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 845 / 第 845 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 846 / 第 846 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 847 / 第 847 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 848 / 第 848 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 849-864 / 第 849-864 行
```cpp
849 |              Corpus.NumInputsWithDataFlowTrace(),
850 |              Corpus.NumInputsThatTouchFocusFunction());
851 |   }
852 | 
853 |   if (Corpus.empty() && Options.MaxNumberOfRuns) {
854 |     Printf("WARNING: no interesting inputs were found so far. "
855 |            "Is the code instrumented for coverage?\n"
856 |            "This may also happen if the target rejected all inputs we tried so "
857 |            "far\n");
858 |     // The remaining logic requires that the corpus is not empty,
859 |     // so we add one fake input to the in-memory corpus.
860 |     Corpus.AddToCorpus({'\n'}, /*NumFeatures=*/1, /*MayDeleteFile=*/true,
861 |                        /*HasFocusFunction=*/false, /*NeverReduce=*/false,
862 |                        /*TimeOfUnit=*/duration_cast<microseconds>(0s), {0}, DFT,
863 |                        /*BaseII*/ nullptr);
864 |   }
```
- **Line 849 / 第 849 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 850 / 第 850 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 851 / 第 851 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 852 / 第 852 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 853 / 第 853 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 854 / 第 854 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 855 / 第 855 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 856 / 第 856 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 857 / 第 857 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 858 / 第 858 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 859 / 第 859 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 860 / 第 860 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 861 / 第 861 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 862 / 第 862 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 863 / 第 863 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 864 / 第 864 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 865-880 / 第 865-880 行
```cpp
865 | }
866 | 
867 | void Fuzzer::Loop(std::vector<SizedFile> &CorporaFiles) {
868 |   auto FocusFunctionOrAuto = Options.FocusFunction;
869 |   DFT.Init(Options.DataFlowTrace, &FocusFunctionOrAuto, CorporaFiles,
870 |            MD.GetRand());
871 |   TPC.SetFocusFunction(FocusFunctionOrAuto);
872 |   ReadAndExecuteSeedCorpora(CorporaFiles);
873 |   DFT.Clear();  // No need for DFT any more.
874 |   TPC.SetPrintNewPCs(Options.PrintNewCovPcs);
875 |   TPC.SetPrintNewFuncs(Options.PrintNewCovFuncs);
876 |   system_clock::time_point LastCorpusReload = system_clock::now();
877 | 
878 |   TmpMaxMutationLen =
879 |       Min(MaxMutationLen, Max(size_t(4), Corpus.MaxInputSize()));
880 | 
```
- **Line 865 / 第 865 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 866 / 第 866 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 867 / 第 867 行**: EN: Starts the definition of function or method `Fuzzer::Loop`. CN: 开始定义函数或方法 `Fuzzer::Loop`。
- **Line 868 / 第 868 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 869 / 第 869 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 870 / 第 870 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 871 / 第 871 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 872 / 第 872 行**: EN: Declares function or method `ReadAndExecuteSeedCorpora`. CN: 声明函数或方法 `ReadAndExecuteSeedCorpora`。
- **Line 873 / 第 873 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 874 / 第 874 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 875 / 第 875 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 876 / 第 876 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 877 / 第 877 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 878 / 第 878 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 879 / 第 879 行**: EN: Declares function or method `Min`. CN: 声明函数或方法 `Min`。
- **Line 880 / 第 880 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 881-896 / 第 881-896 行
```cpp
881 |   while (true) {
882 |     auto Now = system_clock::now();
883 |     if (!Options.StopFile.empty() &&
884 |         !FileToVector(Options.StopFile, 1, false).empty())
885 |       break;
886 |     if (duration_cast<seconds>(Now - LastCorpusReload).count() >=
887 |         Options.ReloadIntervalSec) {
888 |       RereadOutputCorpus(MaxInputLen);
889 |       LastCorpusReload = system_clock::now();
890 |     }
891 |     if (TotalNumberOfRuns >= Options.MaxNumberOfRuns)
892 |       break;
893 |     if (TimedOut())
894 |       break;
895 | 
896 |     // Update TmpMaxMutationLen
```
- **Line 881 / 第 881 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 882 / 第 882 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 883 / 第 883 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 884 / 第 884 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 885 / 第 885 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 886 / 第 886 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 887 / 第 887 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 888 / 第 888 行**: EN: Declares function or method `RereadOutputCorpus`. CN: 声明函数或方法 `RereadOutputCorpus`。
- **Line 889 / 第 889 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 890 / 第 890 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 891 / 第 891 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 892 / 第 892 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 893 / 第 893 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 894 / 第 894 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 895 / 第 895 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 896 / 第 896 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 897-912 / 第 897-912 行
```cpp
897 |     if (Options.LenControl) {
898 |       if (TmpMaxMutationLen < MaxMutationLen &&
899 |           TotalNumberOfRuns - LastCorpusUpdateRun >
900 |               Options.LenControl * Log(TmpMaxMutationLen)) {
901 |         TmpMaxMutationLen =
902 |             Min(MaxMutationLen, TmpMaxMutationLen + Log(TmpMaxMutationLen));
903 |         LastCorpusUpdateRun = TotalNumberOfRuns;
904 |       }
905 |     } else {
906 |       TmpMaxMutationLen = MaxMutationLen;
907 |     }
908 | 
909 |     // Perform several mutations and runs.
910 |     MutateAndTestOne();
911 | 
912 |     PurgeAllocator();
```
- **Line 897 / 第 897 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 898 / 第 898 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 899 / 第 899 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 900 / 第 900 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 901 / 第 901 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 902 / 第 902 行**: EN: Declares function or method `Min`. CN: 声明函数或方法 `Min`。
- **Line 903 / 第 903 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 904 / 第 904 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 905 / 第 905 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 906 / 第 906 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 907 / 第 907 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 908 / 第 908 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 909 / 第 909 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 910 / 第 910 行**: EN: Declares function or method `MutateAndTestOne`. CN: 声明函数或方法 `MutateAndTestOne`。
- **Line 911 / 第 911 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 912 / 第 912 行**: EN: Declares function or method `PurgeAllocator`. CN: 声明函数或方法 `PurgeAllocator`。

### Lines 913-928 / 第 913-928 行
```cpp
913 |   }
914 | 
915 |   PrintStats("DONE  ", "\n");
916 |   MD.PrintRecommendedDictionary();
917 | }
918 | 
919 | void Fuzzer::MinimizeCrashLoop(const Unit &U) {
920 |   if (U.size() <= 1)
921 |     return;
922 |   while (!TimedOut() && TotalNumberOfRuns < Options.MaxNumberOfRuns) {
923 |     MD.StartMutationSequence();
924 |     memcpy(CurrentUnitData, U.data(), U.size());
925 |     for (int i = 0; i < Options.MutateDepth; i++) {
926 |       size_t NewSize = MD.Mutate(CurrentUnitData, U.size(), MaxMutationLen);
927 |       assert(NewSize > 0 && NewSize <= MaxMutationLen);
928 |       ExecuteCallback(CurrentUnitData, NewSize);
```
- **Line 913 / 第 913 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 914 / 第 914 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 915 / 第 915 行**: EN: Declares function or method `PrintStats`. CN: 声明函数或方法 `PrintStats`。
- **Line 916 / 第 916 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 917 / 第 917 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 918 / 第 918 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 919 / 第 919 行**: EN: Starts the definition of function or method `Fuzzer::MinimizeCrashLoop`. CN: 开始定义函数或方法 `Fuzzer::MinimizeCrashLoop`。
- **Line 920 / 第 920 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 921 / 第 921 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 922 / 第 922 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 923 / 第 923 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 924 / 第 924 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 925 / 第 925 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 926 / 第 926 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 927 / 第 927 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 928 / 第 928 行**: EN: Declares function or method `ExecuteCallback`. CN: 声明函数或方法 `ExecuteCallback`。

### Lines 929-944 / 第 929-944 行
```cpp
929 |       PrintPulseAndReportSlowInput(CurrentUnitData, NewSize);
930 |       TryDetectingAMemoryLeak(CurrentUnitData, NewSize,
931 |                               /*DuringInitialCorpusExecution*/ false);
932 |     }
933 |   }
934 | }
935 | 
936 | } // namespace fuzzer
937 | 
938 | extern "C" {
939 | 
940 | ATTRIBUTE_INTERFACE size_t
941 | LLVMFuzzerMutate(uint8_t *Data, size_t Size, size_t MaxSize) {
942 |   assert(fuzzer::F);
943 |   return fuzzer::F->GetMD().DefaultMutate(Data, Size, MaxSize);
944 | }
```
- **Line 929 / 第 929 行**: EN: Declares function or method `PrintPulseAndReportSlowInput`. CN: 声明函数或方法 `PrintPulseAndReportSlowInput`。
- **Line 930 / 第 930 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 931 / 第 931 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 932 / 第 932 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 933 / 第 933 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 934 / 第 934 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 935 / 第 935 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 936 / 第 936 行**: EN: Closes a named namespace scope. CN: 关闭一个具名命名空间作用域。
- **Line 937 / 第 937 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 938 / 第 938 行**: EN: Uses C linkage so the declaration keeps a stable ABI. CN: 使用 C 链接约定，以保持稳定 ABI。
- **Line 939 / 第 939 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 940 / 第 940 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 941 / 第 941 行**: EN: Starts the definition of function or method `LLVMFuzzerMutate`. CN: 开始定义函数或方法 `LLVMFuzzerMutate`。
- **Line 942 / 第 942 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 943 / 第 943 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 944 / 第 944 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 945-946 / 第 945-946 行
```cpp
945 | 
946 | } // extern "C"
```
- **Line 945 / 第 945 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 946 / 第 946 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

## Key Concepts / 关键概念

- **EN**: fuzz input decomposition
  - **CN**: 模糊测试输入拆分
- **EN**: deterministic test input consumption
  - **CN**: 确定性的测试输入消费
- **EN**: coverage-guided fuzzing runtime
  - **CN**: 覆盖率引导的 fuzzing 运行时
- **EN**: namespace scoping and organization
  - **CN**: 命名空间作用域与组织
- **EN**: allocator state management
  - **CN**: 分配器状态管理
- **EN**: thread-aware runtime coordination
  - **CN**: 线程感知的运行时协作

## Dependencies / 依赖关系

- `FuzzerCorpus.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerIO.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerInternal.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerMutate.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerPlatform.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerRandom.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerTracePC.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `algorithm` — System or standard library dependency / 系统或标准库依赖
- `cstring` — System or standard library dependency / 系统或标准库依赖
- `memory` — System or standard library dependency / 系统或标准库依赖
- `mutex` — System or standard library dependency / 系统或标准库依赖
- `set` — System or standard library dependency / 系统或标准库依赖
