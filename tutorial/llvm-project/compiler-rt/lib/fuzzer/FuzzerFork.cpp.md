# FuzzerFork.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/fuzzer/FuzzerFork.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Spawn and orchestrate separate fuzzing processes.
  - **CN**: 实现 libFuzzer 中与 `FuzzerFork` 相关的组件或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
```cpp
 1 | //===- FuzzerFork.cpp - run fuzzing in separate subprocesses --------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | // Spawn and orchestrate separate fuzzing processes.
 9 | //===----------------------------------------------------------------------===//
10 | 
11 | #include "FuzzerCommand.h"
12 | #include "FuzzerFork.h"
13 | #include "FuzzerIO.h"
14 | #include "FuzzerInternal.h"
15 | #include "FuzzerMerge.h"
16 | #include "FuzzerSHA1.h"
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
- **Line 11 / 第 11 行**: EN: Includes `FuzzerCommand.h` so this file can use its declarations. CN: 包含 `FuzzerCommand.h`，以便当前文件使用其中的声明。
- **Line 12 / 第 12 行**: EN: Includes `FuzzerFork.h` so this file can use its declarations. CN: 包含 `FuzzerFork.h`，以便当前文件使用其中的声明。
- **Line 13 / 第 13 行**: EN: Includes `FuzzerIO.h` so this file can use its declarations. CN: 包含 `FuzzerIO.h`，以便当前文件使用其中的声明。
- **Line 14 / 第 14 行**: EN: Includes `FuzzerInternal.h` so this file can use its declarations. CN: 包含 `FuzzerInternal.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Includes `FuzzerMerge.h` so this file can use its declarations. CN: 包含 `FuzzerMerge.h`，以便当前文件使用其中的声明。
- **Line 16 / 第 16 行**: EN: Includes `FuzzerSHA1.h` so this file can use its declarations. CN: 包含 `FuzzerSHA1.h`，以便当前文件使用其中的声明。

### Lines 17-32 / 第 17-32 行
```cpp
17 | #include "FuzzerTracePC.h"
18 | #include "FuzzerUtil.h"
19 | 
20 | #include <atomic>
21 | #include <chrono>
22 | #include <condition_variable>
23 | #include <fstream>
24 | #include <memory>
25 | #include <mutex>
26 | #include <queue>
27 | #include <sstream>
28 | #include <thread>
29 | 
30 | namespace fuzzer {
31 | 
32 | struct Stats {
```
- **Line 17 / 第 17 行**: EN: Includes `FuzzerTracePC.h` so this file can use its declarations. CN: 包含 `FuzzerTracePC.h`，以便当前文件使用其中的声明。
- **Line 18 / 第 18 行**: EN: Includes `FuzzerUtil.h` so this file can use its declarations. CN: 包含 `FuzzerUtil.h`，以便当前文件使用其中的声明。
- **Line 19 / 第 19 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 20 / 第 20 行**: EN: Includes `atomic` so this file can use its declarations. CN: 包含 `atomic`，以便当前文件使用其中的声明。
- **Line 21 / 第 21 行**: EN: Includes `chrono` so this file can use its declarations. CN: 包含 `chrono`，以便当前文件使用其中的声明。
- **Line 22 / 第 22 行**: EN: Includes `condition_variable` so this file can use its declarations. CN: 包含 `condition_variable`，以便当前文件使用其中的声明。
- **Line 23 / 第 23 行**: EN: Includes `fstream` so this file can use its declarations. CN: 包含 `fstream`，以便当前文件使用其中的声明。
- **Line 24 / 第 24 行**: EN: Includes `memory` so this file can use its declarations. CN: 包含 `memory`，以便当前文件使用其中的声明。
- **Line 25 / 第 25 行**: EN: Includes `mutex` so this file can use its declarations. CN: 包含 `mutex`，以便当前文件使用其中的声明。
- **Line 26 / 第 26 行**: EN: Includes `queue` so this file can use its declarations. CN: 包含 `queue`，以便当前文件使用其中的声明。
- **Line 27 / 第 27 行**: EN: Includes `sstream` so this file can use its declarations. CN: 包含 `sstream`，以便当前文件使用其中的声明。
- **Line 28 / 第 28 行**: EN: Includes `thread` so this file can use its declarations. CN: 包含 `thread`，以便当前文件使用其中的声明。
- **Line 29 / 第 29 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 30 / 第 30 行**: EN: Opens namespace `fuzzer` to scope related declarations. CN: 打开命名空间 `fuzzer`，为相关声明建立作用域。
- **Line 31 / 第 31 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 32 / 第 32 行**: EN: Begins the declaration of struct `Stats`. CN: 开始声明 struct `Stats`。

### Lines 33-48 / 第 33-48 行
```cpp
33 |   size_t number_of_executed_units = 0;
34 |   size_t peak_rss_mb = 0;
35 |   size_t average_exec_per_sec = 0;
36 | };
37 | 
38 | static Stats ParseFinalStatsFromLog(const std::string &LogPath) {
39 |   std::ifstream In(LogPath);
40 |   std::string Line;
41 |   Stats Res;
42 |   struct {
43 |     const char *Name;
44 |     size_t *Var;
45 |   } NameVarPairs[] = {
46 |       {"stat::number_of_executed_units:", &Res.number_of_executed_units},
47 |       {"stat::peak_rss_mb:", &Res.peak_rss_mb},
48 |       {"stat::average_exec_per_sec:", &Res.average_exec_per_sec},
```
- **Line 33 / 第 33 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 34 / 第 34 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 35 / 第 35 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 36 / 第 36 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 37 / 第 37 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 38 / 第 38 行**: EN: Starts the definition of function or method `ParseFinalStatsFromLog`. CN: 开始定义函数或方法 `ParseFinalStatsFromLog`。
- **Line 39 / 第 39 行**: EN: Declares function or method `In`. CN: 声明函数或方法 `In`。
- **Line 40 / 第 40 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 41 / 第 41 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 42 / 第 42 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 43 / 第 43 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 44 / 第 44 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 45 / 第 45 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 46 / 第 46 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 47 / 第 47 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 48 / 第 48 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 49-64 / 第 49-64 行
```cpp
49 |       {nullptr, nullptr},
50 |   };
51 |   while (std::getline(In, Line, '\n')) {
52 |     if (Line.find("stat::") != 0) continue;
53 |     std::istringstream ISS(Line);
54 |     std::string Name;
55 |     size_t Val;
56 |     ISS >> Name >> Val;
57 |     for (size_t i = 0; NameVarPairs[i].Name; i++)
58 |       if (Name == NameVarPairs[i].Name)
59 |         *NameVarPairs[i].Var = Val;
60 |   }
61 |   return Res;
62 | }
63 | 
64 | struct FuzzJob {
```
- **Line 49 / 第 49 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 50 / 第 50 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 51 / 第 51 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 52 / 第 52 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 53 / 第 53 行**: EN: Declares function or method `ISS`. CN: 声明函数或方法 `ISS`。
- **Line 54 / 第 54 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 55 / 第 55 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 56 / 第 56 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 57 / 第 57 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 58 / 第 58 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 59 / 第 59 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 60 / 第 60 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 61 / 第 61 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 62 / 第 62 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 63 / 第 63 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 64 / 第 64 行**: EN: Begins the declaration of struct `FuzzJob`. CN: 开始声明 struct `FuzzJob`。

### Lines 65-80 / 第 65-80 行
```cpp
65 |   // Inputs.
66 |   Command Cmd;
67 |   std::string CorpusDir;
68 |   std::string FeaturesDir;
69 |   std::string LogPath;
70 |   std::string SeedListPath;
71 |   std::string CFPath;
72 |   size_t      JobId;
73 | 
74 |   int         DftTimeInSeconds = 0;
75 | 
76 |   // Fuzzing Outputs.
77 |   int ExitCode;
78 | 
79 |   ~FuzzJob() {
80 |     RemoveFile(CFPath);
```
- **Line 65 / 第 65 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 66 / 第 66 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 67 / 第 67 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 68 / 第 68 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 69 / 第 69 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 70 / 第 70 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 71 / 第 71 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 72 / 第 72 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 73 / 第 73 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 74 / 第 74 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 75 / 第 75 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 76 / 第 76 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 77 / 第 77 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 78 / 第 78 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 79 / 第 79 行**: EN: Starts the definition of function or method `~FuzzJob`. CN: 开始定义函数或方法 `~FuzzJob`。
- **Line 80 / 第 80 行**: EN: Declares function or method `RemoveFile`. CN: 声明函数或方法 `RemoveFile`。

### Lines 81-96 / 第 81-96 行
```cpp
81 |     RemoveFile(LogPath);
82 |     RemoveFile(SeedListPath);
83 |     RmDirRecursive(CorpusDir);
84 |     RmDirRecursive(FeaturesDir);
85 |   }
86 | };
87 | 
88 | struct GlobalEnv {
89 |   std::vector<std::string> Args;
90 |   std::vector<std::string> CorpusDirs;
91 |   std::string MainCorpusDir;
92 |   std::string TempDir;
93 |   std::string DFTDir;
94 |   std::string DataFlowBinary;
95 |   std::set<uint32_t> Features, Cov;
96 |   std::set<std::string> FilesWithDFT;
```
- **Line 81 / 第 81 行**: EN: Declares function or method `RemoveFile`. CN: 声明函数或方法 `RemoveFile`。
- **Line 82 / 第 82 行**: EN: Declares function or method `RemoveFile`. CN: 声明函数或方法 `RemoveFile`。
- **Line 83 / 第 83 行**: EN: Declares function or method `RmDirRecursive`. CN: 声明函数或方法 `RmDirRecursive`。
- **Line 84 / 第 84 行**: EN: Declares function or method `RmDirRecursive`. CN: 声明函数或方法 `RmDirRecursive`。
- **Line 85 / 第 85 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 86 / 第 86 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 87 / 第 87 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 88 / 第 88 行**: EN: Begins the declaration of struct `GlobalEnv`. CN: 开始声明 struct `GlobalEnv`。
- **Line 89 / 第 89 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 90 / 第 90 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 91 / 第 91 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 92 / 第 92 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 93 / 第 93 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 94 / 第 94 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 95 / 第 95 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 96 / 第 96 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 97-112 / 第 97-112 行
```cpp
 97 |   std::vector<std::string> Files;
 98 |   std::vector<std::size_t> FilesSizes;
 99 |   Random *Rand;
100 |   std::chrono::system_clock::time_point ProcessStartTime;
101 |   int Verbosity = 0;
102 |   int Group = 0;
103 |   int NumCorpuses = 8;
104 | 
105 |   size_t NumTimeouts = 0;
106 |   size_t NumOOMs = 0;
107 |   size_t NumCrashes = 0;
108 | 
109 | 
110 |   size_t NumRuns = 0;
111 | 
112 |   std::string StopFile() { return DirPlusFile(TempDir, "STOP"); }
```
- **Line 97 / 第 97 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 98 / 第 98 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 99 / 第 99 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 100 / 第 100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 101 / 第 101 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 102 / 第 102 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 103 / 第 103 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 104 / 第 104 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 105 / 第 105 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 106 / 第 106 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 107 / 第 107 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 108 / 第 108 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 109 / 第 109 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 110 / 第 110 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 111 / 第 111 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 112 / 第 112 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 113-128 / 第 113-128 行
```cpp
113 | 
114 |   size_t secondsSinceProcessStartUp() const {
115 |     return std::chrono::duration_cast<std::chrono::seconds>(
116 |                std::chrono::system_clock::now() - ProcessStartTime)
117 |         .count();
118 |   }
119 | 
120 |   FuzzJob *CreateNewJob(size_t JobId) {
121 |     Command Cmd(Args);
122 |     Cmd.removeFlag("fork");
123 |     Cmd.removeFlag("runs");
124 |     Cmd.removeFlag("collect_data_flow");
125 |     for (auto &C : CorpusDirs) // Remove all corpora from the args.
126 |       Cmd.removeArgument(C);
127 |     Cmd.addFlag("reload", "0");  // working in an isolated dir, no reload.
128 |     Cmd.addFlag("print_final_stats", "1");
```
- **Line 113 / 第 113 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 114 / 第 114 行**: EN: Starts the definition of function or method `secondsSinceProcessStartUp`. CN: 开始定义函数或方法 `secondsSinceProcessStartUp`。
- **Line 115 / 第 115 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 116 / 第 116 行**: EN: Starts the definition of function or method `std::chrono::system_clock::now`. CN: 开始定义函数或方法 `std::chrono::system_clock::now`。
- **Line 117 / 第 117 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 118 / 第 118 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 119 / 第 119 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 120 / 第 120 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 121 / 第 121 行**: EN: Declares function or method `Cmd`. CN: 声明函数或方法 `Cmd`。
- **Line 122 / 第 122 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 123 / 第 123 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 124 / 第 124 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 125 / 第 125 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 126 / 第 126 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 127 / 第 127 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 128 / 第 128 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 129-144 / 第 129-144 行
```cpp
129 |     Cmd.addFlag("print_funcs", "0");  // no need to spend time symbolizing.
130 |     Cmd.addFlag("max_total_time", std::to_string(std::min((size_t)300, JobId)));
131 |     Cmd.addFlag("stop_file", StopFile());
132 |     if (!DataFlowBinary.empty()) {
133 |       Cmd.addFlag("data_flow_trace", DFTDir);
134 |       if (!Cmd.hasFlag("focus_function"))
135 |         Cmd.addFlag("focus_function", "auto");
136 |     }
137 |     auto Job = new FuzzJob;
138 |     std::string Seeds;
139 |     if (size_t CorpusSubsetSize =
140 |             std::min(Files.size(), (size_t)sqrt(Files.size() + 2))) {
141 |       auto Time1 = std::chrono::system_clock::now();
142 |       if (Group) { // whether to group the corpus.
143 |         size_t AverageCorpusSize = Files.size() / NumCorpuses + 1;
144 |         size_t StartIndex = ((JobId - 1) % NumCorpuses) * AverageCorpusSize;
```
- **Line 129 / 第 129 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 130 / 第 130 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 131 / 第 131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 132 / 第 132 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 133 / 第 133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 134 / 第 134 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 135 / 第 135 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 136 / 第 136 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 137 / 第 137 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 138 / 第 138 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 139 / 第 139 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 140 / 第 140 行**: EN: Starts the definition of function or method `std::min`. CN: 开始定义函数或方法 `std::min`。
- **Line 141 / 第 141 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 142 / 第 142 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 143 / 第 143 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 144 / 第 144 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 145-160 / 第 145-160 行
```cpp
145 |         for (size_t i = 0; i < CorpusSubsetSize; i++) {
146 |           size_t RandNum = (*Rand)(AverageCorpusSize);
147 |           size_t Index = RandNum + StartIndex;
148 |           Index = Index < Files.size() ? Index
149 |                                        : Rand->SkewTowardsLast(Files.size());
150 |           auto &SF = Files[Index];
151 |           Seeds += (Seeds.empty() ? "" : ",") + SF;
152 |           CollectDFT(SF);
153 |         }
154 |       } else {
155 |         for (size_t i = 0; i < CorpusSubsetSize; i++) {
156 |           auto &SF = Files[Rand->SkewTowardsLast(Files.size())];
157 |           Seeds += (Seeds.empty() ? "" : ",") + SF;
158 |           CollectDFT(SF);
159 |         }
160 |       }
```
- **Line 145 / 第 145 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 146 / 第 146 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 147 / 第 147 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 148 / 第 148 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 149 / 第 149 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 150 / 第 150 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 151 / 第 151 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 152 / 第 152 行**: EN: Declares function or method `CollectDFT`. CN: 声明函数或方法 `CollectDFT`。
- **Line 153 / 第 153 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 154 / 第 154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 155 / 第 155 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 156 / 第 156 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 157 / 第 157 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 158 / 第 158 行**: EN: Declares function or method `CollectDFT`. CN: 声明函数或方法 `CollectDFT`。
- **Line 159 / 第 159 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 160 / 第 160 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 161-176 / 第 161-176 行
```cpp
161 |       auto Time2 = std::chrono::system_clock::now();
162 |       auto DftTimeInSeconds = duration_cast<seconds>(Time2 - Time1).count();
163 |       assert(DftTimeInSeconds < std::numeric_limits<int>::max());
164 |       Job->DftTimeInSeconds = static_cast<int>(DftTimeInSeconds);
165 |     }
166 |     if (!Seeds.empty()) {
167 |       Job->SeedListPath =
168 |           DirPlusFile(TempDir, std::to_string(JobId) + ".seeds");
169 |       WriteToFile(Seeds, Job->SeedListPath);
170 |       Cmd.addFlag("seed_inputs", "@" + Job->SeedListPath);
171 |     }
172 |     Job->LogPath = DirPlusFile(TempDir, std::to_string(JobId) + ".log");
173 |     Job->CorpusDir = DirPlusFile(TempDir, "C" + std::to_string(JobId));
174 |     Job->FeaturesDir = DirPlusFile(TempDir, "F" + std::to_string(JobId));
175 |     Job->CFPath = DirPlusFile(TempDir, std::to_string(JobId) + ".merge");
176 |     Job->JobId = JobId;
```
- **Line 161 / 第 161 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 162 / 第 162 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 163 / 第 163 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 164 / 第 164 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 165 / 第 165 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 166 / 第 166 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 167 / 第 167 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 168 / 第 168 行**: EN: Declares function or method `DirPlusFile`. CN: 声明函数或方法 `DirPlusFile`。
- **Line 169 / 第 169 行**: EN: Declares function or method `WriteToFile`. CN: 声明函数或方法 `WriteToFile`。
- **Line 170 / 第 170 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 171 / 第 171 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 172 / 第 172 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 173 / 第 173 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 174 / 第 174 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 175 / 第 175 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 176 / 第 176 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 177-192 / 第 177-192 行
```cpp
177 | 
178 | 
179 |     Cmd.addArgument(Job->CorpusDir);
180 |     Cmd.addFlag("features_dir", Job->FeaturesDir);
181 | 
182 |     for (auto &D : {Job->CorpusDir, Job->FeaturesDir}) {
183 |       RmDirRecursive(D);
184 |       MkDir(D);
185 |     }
186 | 
187 |     Cmd.setOutputFile(Job->LogPath);
188 |     Cmd.combineOutAndErr();
189 | 
190 |     Job->Cmd = Cmd;
191 | 
192 |     if (Verbosity >= 2)
```
- **Line 177 / 第 177 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 178 / 第 178 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 179 / 第 179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 180 / 第 180 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 181 / 第 181 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 182 / 第 182 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 183 / 第 183 行**: EN: Declares function or method `RmDirRecursive`. CN: 声明函数或方法 `RmDirRecursive`。
- **Line 184 / 第 184 行**: EN: Declares function or method `MkDir`. CN: 声明函数或方法 `MkDir`。
- **Line 185 / 第 185 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 186 / 第 186 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 187 / 第 187 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 188 / 第 188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 189 / 第 189 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 190 / 第 190 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 191 / 第 191 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 192 / 第 192 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 193-208 / 第 193-208 行
```cpp
193 |       Printf("Job %zd/%p Created: %s\n", JobId, Job,
194 |              Job->Cmd.toString().c_str());
195 |     // Start from very short runs and gradually increase them.
196 |     return Job;
197 |   }
198 | 
199 |   void RunOneMergeJob(FuzzJob *Job) {
200 |     auto Stats = ParseFinalStatsFromLog(Job->LogPath);
201 |     NumRuns += Stats.number_of_executed_units;
202 | 
203 |     std::vector<SizedFile> TempFiles, MergeCandidates;
204 |     // Read all newly created inputs and their feature sets.
205 |     // Choose only those inputs that have new features.
206 |     GetSizedFilesFromDir(Job->CorpusDir, &TempFiles);
207 |     std::sort(TempFiles.begin(), TempFiles.end());
208 |     for (auto &F : TempFiles) {
```
- **Line 193 / 第 193 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 194 / 第 194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 195 / 第 195 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 196 / 第 196 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 197 / 第 197 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 198 / 第 198 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 199 / 第 199 行**: EN: Starts the definition of function or method `RunOneMergeJob`. CN: 开始定义函数或方法 `RunOneMergeJob`。
- **Line 200 / 第 200 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 201 / 第 201 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 202 / 第 202 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 203 / 第 203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 204 / 第 204 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 205 / 第 205 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 206 / 第 206 行**: EN: Declares function or method `GetSizedFilesFromDir`. CN: 声明函数或方法 `GetSizedFilesFromDir`。
- **Line 207 / 第 207 行**: EN: Declares function or method `std::sort`. CN: 声明函数或方法 `std::sort`。
- **Line 208 / 第 208 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。

### Lines 209-224 / 第 209-224 行
```cpp
209 |       auto FeatureFile = F.File;
210 |       FeatureFile.replace(0, Job->CorpusDir.size(), Job->FeaturesDir);
211 |       auto FeatureBytes = FileToVector(FeatureFile, 0, false);
212 |       assert((FeatureBytes.size() % sizeof(uint32_t)) == 0);
213 |       std::vector<uint32_t> NewFeatures(FeatureBytes.size() / sizeof(uint32_t));
214 |       memcpy(NewFeatures.data(), FeatureBytes.data(), FeatureBytes.size());
215 |       for (auto Ft : NewFeatures) {
216 |         if (!Features.count(Ft)) {
217 |           MergeCandidates.push_back(F);
218 |           break;
219 |         }
220 |       }
221 |     }
222 |     // if (!FilesToAdd.empty() || Job->ExitCode != 0)
223 |     Printf("#%zd: cov: %zd ft: %zd corp: %zd exec/s: %zd "
224 |            "oom/timeout/crash: %zd/%zd/%zd time: %zds job: %zd dft_time: %d\n",
```
- **Line 209 / 第 209 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 210 / 第 210 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 211 / 第 211 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 212 / 第 212 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 213 / 第 213 行**: EN: Declares function or method `NewFeatures`. CN: 声明函数或方法 `NewFeatures`。
- **Line 214 / 第 214 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 215 / 第 215 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 216 / 第 216 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 217 / 第 217 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 218 / 第 218 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 219 / 第 219 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 220 / 第 220 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 221 / 第 221 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 222 / 第 222 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 223 / 第 223 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 224 / 第 224 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 225-240 / 第 225-240 行
```cpp
225 |            NumRuns, Cov.size(), Features.size(), Files.size(),
226 |            Stats.average_exec_per_sec, NumOOMs, NumTimeouts, NumCrashes,
227 |            secondsSinceProcessStartUp(), Job->JobId, Job->DftTimeInSeconds);
228 | 
229 |     if (MergeCandidates.empty()) return;
230 | 
231 |     std::vector<std::string> FilesToAdd;
232 |     std::set<uint32_t> NewFeatures, NewCov;
233 |     bool IsSetCoverMerge =
234 |         !Job->Cmd.getFlagValue("set_cover_merge").compare("1");
235 |     CrashResistantMerge(Args, {}, MergeCandidates, &FilesToAdd, Features,
236 |                         &NewFeatures, Cov, &NewCov, Job->CFPath, false,
237 |                         IsSetCoverMerge);
238 |     for (auto &Path : FilesToAdd) {
239 |       auto U = FileToVector(Path);
240 |       auto NewPath = DirPlusFile(MainCorpusDir, Hash(U));
```
- **Line 225 / 第 225 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 226 / 第 226 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 227 / 第 227 行**: EN: Declares function or method `secondsSinceProcessStartUp`. CN: 声明函数或方法 `secondsSinceProcessStartUp`。
- **Line 228 / 第 228 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 229 / 第 229 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 230 / 第 230 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 231 / 第 231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 232 / 第 232 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 233 / 第 233 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 234 / 第 234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 235 / 第 235 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 236 / 第 236 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 237 / 第 237 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 238 / 第 238 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 239 / 第 239 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 240 / 第 240 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 241-256 / 第 241-256 行
```cpp
241 |       WriteToFile(U, NewPath);
242 |       if (Group) { // Insert the queue according to the size of the seed.
243 |         size_t UnitSize = U.size();
244 |         auto Idx =
245 |             std::upper_bound(FilesSizes.begin(), FilesSizes.end(), UnitSize) -
246 |             FilesSizes.begin();
247 |         FilesSizes.insert(FilesSizes.begin() + Idx, UnitSize);
248 |         Files.insert(Files.begin() + Idx, NewPath);
249 |       } else {
250 |         Files.push_back(NewPath);
251 |       }
252 |     }
253 |     Features.insert(NewFeatures.begin(), NewFeatures.end());
254 |     Cov.insert(NewCov.begin(), NewCov.end());
255 |     for (auto Idx : NewCov)
256 |       if (auto *TE = TPC.PCTableEntryByIdx(Idx))
```
- **Line 241 / 第 241 行**: EN: Declares function or method `WriteToFile`. CN: 声明函数或方法 `WriteToFile`。
- **Line 242 / 第 242 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 243 / 第 243 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 244 / 第 244 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 245 / 第 245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 246 / 第 246 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 247 / 第 247 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 248 / 第 248 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 249 / 第 249 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 250 / 第 250 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 251 / 第 251 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 252 / 第 252 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 253 / 第 253 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 254 / 第 254 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 255 / 第 255 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 256 / 第 256 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 257-272 / 第 257-272 行
```cpp
257 |         if (TPC.PcIsFuncEntry(TE))
258 |           PrintPC("  NEW_FUNC: %p %F %L\n", "",
259 |                   TPC.GetNextInstructionPc(TE->PC));
260 |   }
261 | 
262 |   void CollectDFT(const std::string &InputPath) {
263 |     if (DataFlowBinary.empty()) return;
264 |     if (!FilesWithDFT.insert(InputPath).second) return;
265 |     Command Cmd(Args);
266 |     Cmd.removeFlag("fork");
267 |     Cmd.removeFlag("runs");
268 |     Cmd.addFlag("data_flow_trace", DFTDir);
269 |     Cmd.addArgument(InputPath);
270 |     for (auto &C : CorpusDirs) // Remove all corpora from the args.
271 |       Cmd.removeArgument(C);
272 |     Cmd.setOutputFile(DirPlusFile(TempDir, "dft.log"));
```
- **Line 257 / 第 257 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 258 / 第 258 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 259 / 第 259 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 260 / 第 260 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 261 / 第 261 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 262 / 第 262 行**: EN: Starts the definition of function or method `CollectDFT`. CN: 开始定义函数或方法 `CollectDFT`。
- **Line 263 / 第 263 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 264 / 第 264 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 265 / 第 265 行**: EN: Declares function or method `Cmd`. CN: 声明函数或方法 `Cmd`。
- **Line 266 / 第 266 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 267 / 第 267 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 268 / 第 268 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 269 / 第 269 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 270 / 第 270 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 271 / 第 271 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 272 / 第 272 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 273-288 / 第 273-288 行
```cpp
273 |     Cmd.combineOutAndErr();
274 |     // Printf("CollectDFT: %s\n", Cmd.toString().c_str());
275 |     ExecuteCommand(Cmd);
276 |   }
277 | 
278 | };
279 | 
280 | struct JobQueue {
281 |   std::queue<FuzzJob *> Qu;
282 |   std::mutex Mu;
283 |   std::condition_variable Cv;
284 | 
285 |   void Push(FuzzJob *Job) {
286 |     {
287 |       std::lock_guard<std::mutex> Lock(Mu);
288 |       Qu.push(Job);
```
- **Line 273 / 第 273 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 274 / 第 274 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 275 / 第 275 行**: EN: Declares function or method `ExecuteCommand`. CN: 声明函数或方法 `ExecuteCommand`。
- **Line 276 / 第 276 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 277 / 第 277 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 278 / 第 278 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 279 / 第 279 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 280 / 第 280 行**: EN: Begins the declaration of struct `JobQueue`. CN: 开始声明 struct `JobQueue`。
- **Line 281 / 第 281 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 282 / 第 282 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 283 / 第 283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 284 / 第 284 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 285 / 第 285 行**: EN: Starts the definition of function or method `Push`. CN: 开始定义函数或方法 `Push`。
- **Line 286 / 第 286 行**: EN: Opens a new scope or block. CN: 打开新的作用域或代码块。
- **Line 287 / 第 287 行**: EN: Declares function or method `Lock`. CN: 声明函数或方法 `Lock`。
- **Line 288 / 第 288 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 289-304 / 第 289-304 行
```cpp
289 |     }
290 |     Cv.notify_one();
291 |   }
292 |   FuzzJob *Pop() {
293 |     std::unique_lock<std::mutex> Lk(Mu);
294 |     // std::lock_guard<std::mutex> Lock(Mu);
295 |     Cv.wait(Lk, [&]{return !Qu.empty();});
296 |     assert(!Qu.empty());
297 |     auto Job = Qu.front();
298 |     Qu.pop();
299 |     return Job;
300 |   }
301 | };
302 | 
303 | void WorkerThread(JobQueue *FuzzQ, JobQueue *MergeQ) {
304 |   while (auto Job = FuzzQ->Pop()) {
```
- **Line 289 / 第 289 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 290 / 第 290 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 291 / 第 291 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 292 / 第 292 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 293 / 第 293 行**: EN: Declares function or method `Lk`. CN: 声明函数或方法 `Lk`。
- **Line 294 / 第 294 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 295 / 第 295 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 296 / 第 296 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 297 / 第 297 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 298 / 第 298 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 299 / 第 299 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 300 / 第 300 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 301 / 第 301 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 302 / 第 302 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 303 / 第 303 行**: EN: Starts the definition of function or method `WorkerThread`. CN: 开始定义函数或方法 `WorkerThread`。
- **Line 304 / 第 304 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。

### Lines 305-320 / 第 305-320 行
```cpp
305 |     // Printf("WorkerThread: job %p\n", Job);
306 |     Job->ExitCode = ExecuteCommand(Job->Cmd);
307 |     MergeQ->Push(Job);
308 |   }
309 | }
310 | 
311 | // This is just a skeleton of an experimental -fork=1 feature.
312 | void FuzzWithFork(Random &Rand, const FuzzingOptions &Options,
313 |                   const std::vector<std::string> &Args,
314 |                   const std::vector<std::string> &CorpusDirs, int NumJobs) {
315 |   Printf("INFO: -fork=%d: fuzzing in separate process(s)\n", NumJobs);
316 | 
317 |   GlobalEnv Env;
318 |   Env.Args = Args;
319 |   Env.CorpusDirs = CorpusDirs;
320 |   Env.Rand = &Rand;
```
- **Line 305 / 第 305 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 306 / 第 306 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 307 / 第 307 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 308 / 第 308 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 309 / 第 309 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 310 / 第 310 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 311 / 第 311 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 312 / 第 312 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 313 / 第 313 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 314 / 第 314 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 315 / 第 315 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 316 / 第 316 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 317 / 第 317 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 318 / 第 318 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 319 / 第 319 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 320 / 第 320 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 321-336 / 第 321-336 行
```cpp
321 |   Env.Verbosity = Options.Verbosity;
322 |   Env.ProcessStartTime = std::chrono::system_clock::now();
323 |   Env.DataFlowBinary = Options.CollectDataFlow;
324 |   Env.Group = Options.ForkCorpusGroups;
325 | 
326 |   std::vector<SizedFile> SeedFiles;
327 |   for (auto &Dir : CorpusDirs)
328 |     GetSizedFilesFromDir(Dir, &SeedFiles);
329 |   std::sort(SeedFiles.begin(), SeedFiles.end());
330 |   Env.TempDir = TempPath("FuzzWithFork", ".dir");
331 |   Env.DFTDir = DirPlusFile(Env.TempDir, "DFT");
332 |   RmDirRecursive(Env.TempDir);  // in case there is a leftover from old runs.
333 |   MkDir(Env.TempDir);
334 |   MkDir(Env.DFTDir);
335 | 
336 | 
```
- **Line 321 / 第 321 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 322 / 第 322 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 323 / 第 323 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 324 / 第 324 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 325 / 第 325 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 326 / 第 326 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 327 / 第 327 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 328 / 第 328 行**: EN: Declares function or method `GetSizedFilesFromDir`. CN: 声明函数或方法 `GetSizedFilesFromDir`。
- **Line 329 / 第 329 行**: EN: Declares function or method `std::sort`. CN: 声明函数或方法 `std::sort`。
- **Line 330 / 第 330 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 331 / 第 331 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 332 / 第 332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 333 / 第 333 行**: EN: Declares function or method `MkDir`. CN: 声明函数或方法 `MkDir`。
- **Line 334 / 第 334 行**: EN: Declares function or method `MkDir`. CN: 声明函数或方法 `MkDir`。
- **Line 335 / 第 335 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 336 / 第 336 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 337-352 / 第 337-352 行
```cpp
337 |   if (CorpusDirs.empty())
338 |     MkDir(Env.MainCorpusDir = DirPlusFile(Env.TempDir, "C"));
339 |   else
340 |     Env.MainCorpusDir = CorpusDirs[0];
341 | 
342 |   if (Options.KeepSeed) {
343 |     for (auto &File : SeedFiles)
344 |       Env.Files.push_back(File.File);
345 |   } else {
346 |     auto CFPath = DirPlusFile(Env.TempDir, "merge.txt");
347 |     std::set<uint32_t> NewFeatures, NewCov;
348 |     CrashResistantMerge(Env.Args, {}, SeedFiles, &Env.Files, Env.Features,
349 |                         &NewFeatures, Env.Cov, &NewCov, CFPath,
350 |                         /*Verbose=*/false, /*IsSetCoverMerge=*/false);
351 |     Env.Features.insert(NewFeatures.begin(), NewFeatures.end());
352 |     Env.Cov.insert(NewCov.begin(), NewCov.end());
```
- **Line 337 / 第 337 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 338 / 第 338 行**: EN: Declares function or method `MkDir`. CN: 声明函数或方法 `MkDir`。
- **Line 339 / 第 339 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 340 / 第 340 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 341 / 第 341 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 342 / 第 342 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 343 / 第 343 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 344 / 第 344 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 345 / 第 345 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 346 / 第 346 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 347 / 第 347 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 348 / 第 348 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 349 / 第 349 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 350 / 第 350 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 351 / 第 351 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 352 / 第 352 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 353-368 / 第 353-368 行
```cpp
353 |     RemoveFile(CFPath);
354 |   }
355 | 
356 |   if (Env.Group) {
357 |     for (auto &path : Env.Files)
358 |       Env.FilesSizes.push_back(FileSize(path));
359 |   }
360 | 
361 |   Printf("INFO: -fork=%d: %zd seed inputs, starting to fuzz in %s\n", NumJobs,
362 |          Env.Files.size(), Env.TempDir.c_str());
363 | 
364 |   int ExitCode = 0;
365 | 
366 |   JobQueue FuzzQ, MergeQ;
367 | 
368 |   auto StopJobs = [&]() {
```
- **Line 353 / 第 353 行**: EN: Declares function or method `RemoveFile`. CN: 声明函数或方法 `RemoveFile`。
- **Line 354 / 第 354 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 355 / 第 355 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 356 / 第 356 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 357 / 第 357 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 358 / 第 358 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 359 / 第 359 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 360 / 第 360 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 361 / 第 361 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 362 / 第 362 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 363 / 第 363 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 364 / 第 364 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 365 / 第 365 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 366 / 第 366 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 367 / 第 367 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 368 / 第 368 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 369-384 / 第 369-384 行
```cpp
369 |     for (int i = 0; i < NumJobs; i++)
370 |       FuzzQ.Push(nullptr);
371 |     MergeQ.Push(nullptr);
372 |     WriteToFile(Unit({1}), Env.StopFile());
373 |   };
374 | 
375 |   size_t MergeCycle = 20;
376 |   size_t JobExecuted = 0;
377 |   size_t JobId = 1;
378 |   std::vector<std::thread> Threads;
379 |   for (int t = 0; t < NumJobs; t++) {
380 |     Threads.push_back(std::thread(WorkerThread, &FuzzQ, &MergeQ));
381 |     FuzzQ.Push(Env.CreateNewJob(JobId++));
382 |   }
383 | 
384 |   while (true) {
```
- **Line 369 / 第 369 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 370 / 第 370 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 371 / 第 371 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 372 / 第 372 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 373 / 第 373 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 374 / 第 374 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 375 / 第 375 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 376 / 第 376 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 377 / 第 377 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 378 / 第 378 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 379 / 第 379 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 380 / 第 380 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 381 / 第 381 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 382 / 第 382 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 383 / 第 383 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 384 / 第 384 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。

### Lines 385-400 / 第 385-400 行
```cpp
385 |     std::unique_ptr<FuzzJob> Job(MergeQ.Pop());
386 |     if (!Job)
387 |       break;
388 |     ExitCode = Job->ExitCode;
389 |     if (ExitCode == Options.InterruptExitCode) {
390 |       Printf("==%lu== libFuzzer: a child was interrupted; exiting\n", GetPid());
391 |       StopJobs();
392 |       break;
393 |     }
394 |     Fuzzer::MaybeExitGracefully();
395 | 
396 |     Env.RunOneMergeJob(Job.get());
397 | 
398 |     // merge the corpus .
399 |     JobExecuted++;
400 |     if (Env.Group && JobExecuted >= MergeCycle) {
```
- **Line 385 / 第 385 行**: EN: Declares function or method `Job`. CN: 声明函数或方法 `Job`。
- **Line 386 / 第 386 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 387 / 第 387 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 388 / 第 388 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 389 / 第 389 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 390 / 第 390 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 391 / 第 391 行**: EN: Declares function or method `StopJobs`. CN: 声明函数或方法 `StopJobs`。
- **Line 392 / 第 392 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 393 / 第 393 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 394 / 第 394 行**: EN: Declares function or method `Fuzzer::MaybeExitGracefully`. CN: 声明函数或方法 `Fuzzer::MaybeExitGracefully`。
- **Line 395 / 第 395 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 396 / 第 396 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 397 / 第 397 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 398 / 第 398 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 399 / 第 399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 400 / 第 400 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 401-416 / 第 401-416 行
```cpp
401 |       std::vector<SizedFile> CurrentSeedFiles;
402 |       for (auto &Dir : CorpusDirs)
403 |         GetSizedFilesFromDir(Dir, &CurrentSeedFiles);
404 |       std::sort(CurrentSeedFiles.begin(), CurrentSeedFiles.end());
405 | 
406 |       auto CFPath = DirPlusFile(Env.TempDir, "merge.txt");
407 |       std::set<uint32_t> TmpNewFeatures, TmpNewCov;
408 |       std::set<uint32_t> TmpFeatures, TmpCov;
409 |       Env.Files.clear();
410 |       Env.FilesSizes.clear();
411 |       CrashResistantMerge(Env.Args, {}, CurrentSeedFiles, &Env.Files,
412 |                           TmpFeatures, &TmpNewFeatures, TmpCov, &TmpNewCov,
413 |                           CFPath, /*Verbose=*/false, /*IsSetCoverMerge=*/false);
414 |       for (auto &path : Env.Files)
415 |         Env.FilesSizes.push_back(FileSize(path));
416 |       RemoveFile(CFPath);
```
- **Line 401 / 第 401 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 402 / 第 402 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 403 / 第 403 行**: EN: Declares function or method `GetSizedFilesFromDir`. CN: 声明函数或方法 `GetSizedFilesFromDir`。
- **Line 404 / 第 404 行**: EN: Declares function or method `std::sort`. CN: 声明函数或方法 `std::sort`。
- **Line 405 / 第 405 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 406 / 第 406 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 407 / 第 407 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 408 / 第 408 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 409 / 第 409 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 410 / 第 410 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 411 / 第 411 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 412 / 第 412 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 413 / 第 413 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 414 / 第 414 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 415 / 第 415 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 416 / 第 416 行**: EN: Declares function or method `RemoveFile`. CN: 声明函数或方法 `RemoveFile`。

### Lines 417-432 / 第 417-432 行
```cpp
417 |       JobExecuted = 0;
418 |       MergeCycle += 5;
419 |     }
420 | 
421 |     // Since the number of corpus seeds will gradually increase, in order to
422 |     // control the number in each group to be about three times the number of
423 |     // seeds selected each time, the number of groups is dynamically adjusted.
424 |     if (Env.Files.size() < 2000)
425 |       Env.NumCorpuses = 12;
426 |     else if (Env.Files.size() < 6000)
427 |       Env.NumCorpuses = 20;
428 |     else if (Env.Files.size() < 12000)
429 |       Env.NumCorpuses = 32;
430 |     else if (Env.Files.size() < 16000)
431 |       Env.NumCorpuses = 40;
432 |     else if (Env.Files.size() < 24000)
```
- **Line 417 / 第 417 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 418 / 第 418 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 419 / 第 419 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 420 / 第 420 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 421 / 第 421 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 422 / 第 422 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 423 / 第 423 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 424 / 第 424 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 425 / 第 425 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 426 / 第 426 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 427 / 第 427 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 428 / 第 428 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 429 / 第 429 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 430 / 第 430 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 431 / 第 431 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 432 / 第 432 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。

### Lines 433-448 / 第 433-448 行
```cpp
433 |       Env.NumCorpuses = 60;
434 |     else
435 |       Env.NumCorpuses = 80;
436 | 
437 |     // Continue if our crash is one of the ignored ones.
438 |     if (Options.IgnoreTimeouts && ExitCode == Options.TimeoutExitCode)
439 |       Env.NumTimeouts++;
440 |     else if (Options.IgnoreOOMs && ExitCode == Options.OOMExitCode)
441 |       Env.NumOOMs++;
442 |     else if (ExitCode != 0) {
443 |       Env.NumCrashes++;
444 |       if (Options.IgnoreCrashes) {
445 |         std::ifstream In(Job->LogPath);
446 |         std::string Line;
447 |         while (std::getline(In, Line, '\n'))
448 |           if (Line.find("ERROR:") != Line.npos ||
```
- **Line 433 / 第 433 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 434 / 第 434 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 435 / 第 435 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 436 / 第 436 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 437 / 第 437 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 438 / 第 438 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 439 / 第 439 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 440 / 第 440 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 441 / 第 441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 442 / 第 442 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 443 / 第 443 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 444 / 第 444 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 445 / 第 445 行**: EN: Declares function or method `In`. CN: 声明函数或方法 `In`。
- **Line 446 / 第 446 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 447 / 第 447 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 448 / 第 448 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 449-464 / 第 449-464 行
```cpp
449 |               Line.find("runtime error:") != Line.npos)
450 |             Printf("%s\n", Line.c_str());
451 |       } else {
452 |         // And exit if we don't ignore this crash.
453 |         Printf("INFO: log from the inner process:\n%s",
454 |                FileToString(Job->LogPath).c_str());
455 |         StopJobs();
456 |         break;
457 |       }
458 |     }
459 | 
460 |     // Stop if we are over the time budget.
461 |     // This is not precise, since other threads are still running
462 |     // and we will wait while joining them.
463 |     // We also don't stop instantly: other jobs need to finish.
464 |     if (Options.MaxTotalTimeSec > 0 &&
```
- **Line 449 / 第 449 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 450 / 第 450 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 451 / 第 451 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 452 / 第 452 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 453 / 第 453 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 454 / 第 454 行**: EN: Declares function or method `FileToString`. CN: 声明函数或方法 `FileToString`。
- **Line 455 / 第 455 行**: EN: Declares function or method `StopJobs`. CN: 声明函数或方法 `StopJobs`。
- **Line 456 / 第 456 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 457 / 第 457 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 458 / 第 458 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 459 / 第 459 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 460 / 第 460 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 461 / 第 461 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 462 / 第 462 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 463 / 第 463 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 464 / 第 464 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 465-480 / 第 465-480 行
```cpp
465 |         Env.secondsSinceProcessStartUp() >= (size_t)Options.MaxTotalTimeSec) {
466 |       Printf("INFO: fuzzed for %zd seconds, wrapping up soon\n",
467 |              Env.secondsSinceProcessStartUp());
468 |       StopJobs();
469 |       break;
470 |     }
471 |     if (Env.NumRuns >= Options.MaxNumberOfRuns) {
472 |       Printf("INFO: fuzzed for %zd iterations, wrapping up soon\n",
473 |              Env.NumRuns);
474 |       StopJobs();
475 |       break;
476 |     }
477 | 
478 |     FuzzQ.Push(Env.CreateNewJob(JobId++));
479 |   }
480 | 
```
- **Line 465 / 第 465 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 466 / 第 466 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 467 / 第 467 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 468 / 第 468 行**: EN: Declares function or method `StopJobs`. CN: 声明函数或方法 `StopJobs`。
- **Line 469 / 第 469 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 470 / 第 470 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 471 / 第 471 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 472 / 第 472 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 473 / 第 473 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 474 / 第 474 行**: EN: Declares function or method `StopJobs`. CN: 声明函数或方法 `StopJobs`。
- **Line 475 / 第 475 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 476 / 第 476 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 477 / 第 477 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 478 / 第 478 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 479 / 第 479 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 480 / 第 480 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 481-494 / 第 481-494 行
```cpp
481 |   for (auto &T : Threads)
482 |     T.join();
483 | 
484 |   // The workers have terminated. Don't try to remove the directory before they
485 |   // terminate to avoid a race condition preventing cleanup on Windows.
486 |   RmDirRecursive(Env.TempDir);
487 | 
488 |   // Use the exit code from the last child process.
489 |   Printf("INFO: exiting: %d time: %zds\n", ExitCode,
490 |          Env.secondsSinceProcessStartUp());
491 |   exit(ExitCode);
492 | }
493 | 
494 | } // namespace fuzzer
```
- **Line 481 / 第 481 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 482 / 第 482 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 483 / 第 483 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 484 / 第 484 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 485 / 第 485 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 486 / 第 486 行**: EN: Declares function or method `RmDirRecursive`. CN: 声明函数或方法 `RmDirRecursive`。
- **Line 487 / 第 487 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 488 / 第 488 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 489 / 第 489 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 490 / 第 490 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 491 / 第 491 行**: EN: Declares function or method `exit`. CN: 声明函数或方法 `exit`。
- **Line 492 / 第 492 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 493 / 第 493 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 494 / 第 494 行**: EN: Closes a named namespace scope. CN: 关闭一个具名命名空间作用域。

## Key Concepts / 关键概念

- **EN**: fuzz input decomposition
  - **CN**: 模糊测试输入拆分
- **EN**: deterministic test input consumption
  - **CN**: 确定性的测试输入消费
- **EN**: coverage-guided fuzzing runtime
  - **CN**: 覆盖率引导的 fuzzing 运行时
- **EN**: namespace scoping and organization
  - **CN**: 命名空间作用域与组织
- **EN**: thread-aware runtime coordination
  - **CN**: 线程感知的运行时协作

## Dependencies / 依赖关系

- `FuzzerCommand.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerFork.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerIO.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerInternal.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerMerge.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerSHA1.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerTracePC.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerUtil.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `atomic` — System or standard library dependency / 系统或标准库依赖
- `chrono` — System or standard library dependency / 系统或标准库依赖
- `condition_variable` — System or standard library dependency / 系统或标准库依赖
- `fstream` — System or standard library dependency / 系统或标准库依赖
