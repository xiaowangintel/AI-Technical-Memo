# FuzzerMerge.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/fuzzer/FuzzerMerge.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements libFuzzer components related to `FuzzerMerge`.
  - **CN**: 实现 libFuzzer 中与 `FuzzerMerge` 相关的组件或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
```cpp
 1 | //===- FuzzerMerge.cpp - merging corpora ----------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | // Merging corpora.
 9 | //===----------------------------------------------------------------------===//
10 | 
11 | #include "FuzzerCommand.h"
12 | #include "FuzzerMerge.h"
13 | #include "FuzzerIO.h"
14 | #include "FuzzerInternal.h"
15 | #include "FuzzerTracePC.h"
16 | #include "FuzzerUtil.h"
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
- **Line 12 / 第 12 行**: EN: Includes `FuzzerMerge.h` so this file can use its declarations. CN: 包含 `FuzzerMerge.h`，以便当前文件使用其中的声明。
- **Line 13 / 第 13 行**: EN: Includes `FuzzerIO.h` so this file can use its declarations. CN: 包含 `FuzzerIO.h`，以便当前文件使用其中的声明。
- **Line 14 / 第 14 行**: EN: Includes `FuzzerInternal.h` so this file can use its declarations. CN: 包含 `FuzzerInternal.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Includes `FuzzerTracePC.h` so this file can use its declarations. CN: 包含 `FuzzerTracePC.h`，以便当前文件使用其中的声明。
- **Line 16 / 第 16 行**: EN: Includes `FuzzerUtil.h` so this file can use its declarations. CN: 包含 `FuzzerUtil.h`，以便当前文件使用其中的声明。

### Lines 17-32 / 第 17-32 行
```cpp
17 | 
18 | #include <fstream>
19 | #include <iterator>
20 | #include <set>
21 | #include <sstream>
22 | #include <unordered_set>
23 | 
24 | namespace fuzzer {
25 | 
26 | bool Merger::Parse(const std::string &Str, bool ParseCoverage) {
27 |   std::istringstream SS(Str);
28 |   return Parse(SS, ParseCoverage);
29 | }
30 | 
31 | void Merger::ParseOrExit(std::istream &IS, bool ParseCoverage) {
32 |   if (!Parse(IS, ParseCoverage)) {
```
- **Line 17 / 第 17 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 18 / 第 18 行**: EN: Includes `fstream` so this file can use its declarations. CN: 包含 `fstream`，以便当前文件使用其中的声明。
- **Line 19 / 第 19 行**: EN: Includes `iterator` so this file can use its declarations. CN: 包含 `iterator`，以便当前文件使用其中的声明。
- **Line 20 / 第 20 行**: EN: Includes `set` so this file can use its declarations. CN: 包含 `set`，以便当前文件使用其中的声明。
- **Line 21 / 第 21 行**: EN: Includes `sstream` so this file can use its declarations. CN: 包含 `sstream`，以便当前文件使用其中的声明。
- **Line 22 / 第 22 行**: EN: Includes `unordered_set` so this file can use its declarations. CN: 包含 `unordered_set`，以便当前文件使用其中的声明。
- **Line 23 / 第 23 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 24 / 第 24 行**: EN: Opens namespace `fuzzer` to scope related declarations. CN: 打开命名空间 `fuzzer`，为相关声明建立作用域。
- **Line 25 / 第 25 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 26 / 第 26 行**: EN: Starts the definition of function or method `Merger::Parse`. CN: 开始定义函数或方法 `Merger::Parse`。
- **Line 27 / 第 27 行**: EN: Declares function or method `SS`. CN: 声明函数或方法 `SS`。
- **Line 28 / 第 28 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 29 / 第 29 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 30 / 第 30 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 31 / 第 31 行**: EN: Starts the definition of function or method `Merger::ParseOrExit`. CN: 开始定义函数或方法 `Merger::ParseOrExit`。
- **Line 32 / 第 32 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 33-48 / 第 33-48 行
```cpp
33 |     Printf("MERGE: failed to parse the control file (unexpected error)\n");
34 |     exit(1);
35 |   }
36 | }
37 | 
38 | // The control file example:
39 | //
40 | // 3 # The number of inputs
41 | // 1 # The number of inputs in the first corpus, <= the previous number
42 | // file0
43 | // file1
44 | // file2  # One file name per line.
45 | // STARTED 0 123  # FileID, file size
46 | // FT 0 1 4 6 8  # FileID COV1 COV2 ...
47 | // COV 0 7 8 9 # FileID COV1 COV1
48 | // STARTED 1 456  # If FT is missing, the input crashed while processing.
```
- **Line 33 / 第 33 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 34 / 第 34 行**: EN: Declares function or method `exit`. CN: 声明函数或方法 `exit`。
- **Line 35 / 第 35 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 36 / 第 36 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 37 / 第 37 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 38 / 第 38 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 39 / 第 39 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 40 / 第 40 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 41 / 第 41 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 42 / 第 42 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 43 / 第 43 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 44 / 第 44 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 45 / 第 45 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 46 / 第 46 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 47 / 第 47 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 48 / 第 48 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 49-64 / 第 49-64 行
```cpp
49 | // STARTED 2 567
50 | // FT 2 8 9
51 | // COV 2 11 12
52 | bool Merger::Parse(std::istream &IS, bool ParseCoverage) {
53 |   LastFailure.clear();
54 |   std::string Line;
55 | 
56 |   // Parse NumFiles.
57 |   if (!std::getline(IS, Line, '\n')) return false;
58 |   std::istringstream L1(Line);
59 |   size_t NumFiles = 0;
60 |   L1 >> NumFiles;
61 |   if (NumFiles == 0 || NumFiles > 10000000) return false;
62 | 
63 |   // Parse NumFilesInFirstCorpus.
64 |   if (!std::getline(IS, Line, '\n')) return false;
```
- **Line 49 / 第 49 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 50 / 第 50 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 51 / 第 51 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 52 / 第 52 行**: EN: Starts the definition of function or method `Merger::Parse`. CN: 开始定义函数或方法 `Merger::Parse`。
- **Line 53 / 第 53 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 54 / 第 54 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 55 / 第 55 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 56 / 第 56 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 57 / 第 57 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 58 / 第 58 行**: EN: Declares function or method `L1`. CN: 声明函数或方法 `L1`。
- **Line 59 / 第 59 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 60 / 第 60 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 61 / 第 61 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 62 / 第 62 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 63 / 第 63 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 64 / 第 64 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 65-80 / 第 65-80 行
```cpp
65 |   std::istringstream L2(Line);
66 |   NumFilesInFirstCorpus = NumFiles + 1;
67 |   L2 >> NumFilesInFirstCorpus;
68 |   if (NumFilesInFirstCorpus > NumFiles) return false;
69 | 
70 |   // Parse file names.
71 |   Files.resize(NumFiles);
72 |   for (size_t i = 0; i < NumFiles; i++)
73 |     if (!std::getline(IS, Files[i].Name, '\n'))
74 |       return false;
75 | 
76 |   // Parse STARTED, FT, and COV lines.
77 |   size_t ExpectedStartMarker = 0;
78 |   const size_t kInvalidStartMarker = -1;
79 |   size_t LastSeenStartMarker = kInvalidStartMarker;
80 |   bool HaveFtMarker = true;
```
- **Line 65 / 第 65 行**: EN: Declares function or method `L2`. CN: 声明函数或方法 `L2`。
- **Line 66 / 第 66 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 67 / 第 67 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 68 / 第 68 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 69 / 第 69 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 70 / 第 70 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 71 / 第 71 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 72 / 第 72 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 73 / 第 73 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 74 / 第 74 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 75 / 第 75 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 76 / 第 76 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 77 / 第 77 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 78 / 第 78 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 79 / 第 79 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 80 / 第 80 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 81-96 / 第 81-96 行
```cpp
81 |   std::vector<uint32_t> TmpFeatures;
82 |   std::set<uint32_t> PCs;
83 |   while (std::getline(IS, Line, '\n')) {
84 |     std::istringstream ISS1(Line);
85 |     std::string Marker;
86 |     uint32_t N;
87 |     if (!(ISS1 >> Marker) || !(ISS1 >> N))
88 |       return false;
89 |     if (Marker == "STARTED") {
90 |       // STARTED FILE_ID FILE_SIZE
91 |       if (ExpectedStartMarker != N)
92 |         return false;
93 |       ISS1 >> Files[ExpectedStartMarker].Size;
94 |       LastSeenStartMarker = ExpectedStartMarker;
95 |       assert(ExpectedStartMarker < Files.size());
96 |       ExpectedStartMarker++;
```
- **Line 81 / 第 81 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 82 / 第 82 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 83 / 第 83 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 84 / 第 84 行**: EN: Declares function or method `ISS1`. CN: 声明函数或方法 `ISS1`。
- **Line 85 / 第 85 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 86 / 第 86 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 87 / 第 87 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 88 / 第 88 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 89 / 第 89 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 90 / 第 90 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 91 / 第 91 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 92 / 第 92 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 93 / 第 93 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 94 / 第 94 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 95 / 第 95 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 96 / 第 96 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 97-112 / 第 97-112 行
```cpp
 97 |       HaveFtMarker = false;
 98 |     } else if (Marker == "FT") {
 99 |       // FT FILE_ID COV1 COV2 COV3 ...
100 |       size_t CurrentFileIdx = N;
101 |       if (CurrentFileIdx != LastSeenStartMarker)
102 |         return false;
103 |       HaveFtMarker = true;
104 |       if (ParseCoverage) {
105 |         TmpFeatures.clear();  // use a vector from outer scope to avoid resizes.
106 |         while (ISS1 >> N)
107 |           TmpFeatures.push_back(N);
108 |         std::sort(TmpFeatures.begin(), TmpFeatures.end());
109 |         Files[CurrentFileIdx].Features = TmpFeatures;
110 |       }
111 |     } else if (Marker == "COV") {
112 |       size_t CurrentFileIdx = N;
```
- **Line 97 / 第 97 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 98 / 第 98 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 99 / 第 99 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 100 / 第 100 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 101 / 第 101 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 102 / 第 102 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 103 / 第 103 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 104 / 第 104 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 105 / 第 105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 106 / 第 106 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 107 / 第 107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 108 / 第 108 行**: EN: Declares function or method `std::sort`. CN: 声明函数或方法 `std::sort`。
- **Line 109 / 第 109 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 110 / 第 110 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 111 / 第 111 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 112 / 第 112 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 113-128 / 第 113-128 行
```cpp
113 |       if (CurrentFileIdx != LastSeenStartMarker)
114 |         return false;
115 |       if (ParseCoverage)
116 |         while (ISS1 >> N)
117 |           if (PCs.insert(N).second)
118 |             Files[CurrentFileIdx].Cov.push_back(N);
119 |     } else {
120 |       return false;
121 |     }
122 |   }
123 |   if (!HaveFtMarker && LastSeenStartMarker != kInvalidStartMarker)
124 |     LastFailure = Files[LastSeenStartMarker].Name;
125 | 
126 |   FirstNotProcessedFile = ExpectedStartMarker;
127 |   return true;
128 | }
```
- **Line 113 / 第 113 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 114 / 第 114 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 115 / 第 115 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 116 / 第 116 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 117 / 第 117 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 118 / 第 118 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 119 / 第 119 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 120 / 第 120 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 121 / 第 121 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 122 / 第 122 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 123 / 第 123 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 124 / 第 124 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 125 / 第 125 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 126 / 第 126 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 127 / 第 127 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 128 / 第 128 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 129-144 / 第 129-144 行
```cpp
129 | 
130 | size_t Merger::ApproximateMemoryConsumption() const  {
131 |   size_t Res = 0;
132 |   for (const auto &F: Files)
133 |     Res += sizeof(F) + F.Features.size() * sizeof(F.Features[0]);
134 |   return Res;
135 | }
136 | 
137 | // Decides which files need to be merged (add those to NewFiles).
138 | // Returns the number of new features added.
139 | size_t Merger::Merge(const std::set<uint32_t> &InitialFeatures,
140 |                      std::set<uint32_t> *NewFeatures,
141 |                      const std::set<uint32_t> &InitialCov,
142 |                      std::set<uint32_t> *NewCov,
143 |                      std::vector<std::string> *NewFiles) {
144 |   NewFiles->clear();
```
- **Line 129 / 第 129 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 130 / 第 130 行**: EN: Starts the definition of function or method `Merger::ApproximateMemoryConsumption`. CN: 开始定义函数或方法 `Merger::ApproximateMemoryConsumption`。
- **Line 131 / 第 131 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 132 / 第 132 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 133 / 第 133 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 134 / 第 134 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 135 / 第 135 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 136 / 第 136 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 137 / 第 137 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 138 / 第 138 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 139 / 第 139 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 140 / 第 140 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 141 / 第 141 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 142 / 第 142 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 143 / 第 143 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 144 / 第 144 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 145-160 / 第 145-160 行
```cpp
145 |   NewFeatures->clear();
146 |   NewCov->clear();
147 |   assert(NumFilesInFirstCorpus <= Files.size());
148 |   std::set<uint32_t> AllFeatures = InitialFeatures;
149 | 
150 |   // What features are in the initial corpus?
151 |   for (size_t i = 0; i < NumFilesInFirstCorpus; i++) {
152 |     auto &Cur = Files[i].Features;
153 |     AllFeatures.insert(Cur.begin(), Cur.end());
154 |   }
155 |   // Remove all features that we already know from all other inputs.
156 |   for (size_t i = NumFilesInFirstCorpus; i < Files.size(); i++) {
157 |     auto &Cur = Files[i].Features;
158 |     std::vector<uint32_t> Tmp;
159 |     std::set_difference(Cur.begin(), Cur.end(), AllFeatures.begin(),
160 |                         AllFeatures.end(), std::inserter(Tmp, Tmp.begin()));
```
- **Line 145 / 第 145 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 146 / 第 146 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 147 / 第 147 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 148 / 第 148 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 149 / 第 149 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 150 / 第 150 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 151 / 第 151 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 152 / 第 152 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 153 / 第 153 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 154 / 第 154 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 155 / 第 155 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 156 / 第 156 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 157 / 第 157 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 158 / 第 158 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 159 / 第 159 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 160 / 第 160 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 161-176 / 第 161-176 行
```cpp
161 |     Cur.swap(Tmp);
162 |   }
163 | 
164 |   // Sort. Give preference to
165 |   //   * smaller files
166 |   //   * files with more features.
167 |   std::sort(Files.begin() + NumFilesInFirstCorpus, Files.end(),
168 |             [&](const MergeFileInfo &a, const MergeFileInfo &b) -> bool {
169 |               if (a.Size != b.Size)
170 |                 return a.Size < b.Size;
171 |               return a.Features.size() > b.Features.size();
172 |             });
173 | 
174 |   // One greedy pass: add the file's features to AllFeatures.
175 |   // If new features were added, add this file to NewFiles.
176 |   for (size_t i = NumFilesInFirstCorpus; i < Files.size(); i++) {
```
- **Line 161 / 第 161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 162 / 第 162 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 163 / 第 163 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 164 / 第 164 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 165 / 第 165 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 166 / 第 166 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 167 / 第 167 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 168 / 第 168 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 169 / 第 169 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 170 / 第 170 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 171 / 第 171 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 172 / 第 172 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 173 / 第 173 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 174 / 第 174 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 175 / 第 175 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 176 / 第 176 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。

### Lines 177-192 / 第 177-192 行
```cpp
177 |     auto &Cur = Files[i].Features;
178 |     // Printf("%s -> sz %zd ft %zd\n", Files[i].Name.c_str(),
179 |     //       Files[i].Size, Cur.size());
180 |     bool FoundNewFeatures = false;
181 |     for (auto Fe: Cur) {
182 |       if (AllFeatures.insert(Fe).second) {
183 |         FoundNewFeatures = true;
184 |         NewFeatures->insert(Fe);
185 |       }
186 |     }
187 |     if (FoundNewFeatures)
188 |       NewFiles->push_back(Files[i].Name);
189 |     for (auto Cov : Files[i].Cov)
190 |       if (InitialCov.find(Cov) == InitialCov.end())
191 |         NewCov->insert(Cov);
192 |   }
```
- **Line 177 / 第 177 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 178 / 第 178 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 179 / 第 179 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 180 / 第 180 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 181 / 第 181 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 182 / 第 182 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 183 / 第 183 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 184 / 第 184 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 185 / 第 185 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 186 / 第 186 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 187 / 第 187 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 188 / 第 188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 189 / 第 189 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 190 / 第 190 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 191 / 第 191 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 192 / 第 192 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 193-208 / 第 193-208 行
```cpp
193 |   return NewFeatures->size();
194 | }
195 | 
196 | std::set<uint32_t> Merger::AllFeatures() const {
197 |   std::set<uint32_t> S;
198 |   for (auto &File : Files)
199 |     S.insert(File.Features.begin(), File.Features.end());
200 |   return S;
201 | }
202 | 
203 | // Inner process. May crash if the target crashes.
204 | void Fuzzer::CrashResistantMergeInternalStep(const std::string &CFPath,
205 |                                              bool IsSetCoverMerge) {
206 |   Printf("MERGE-INNER: using the control file '%s'\n", CFPath.c_str());
207 |   Merger M;
208 |   std::ifstream IF(CFPath);
```
- **Line 193 / 第 193 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 194 / 第 194 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 195 / 第 195 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 196 / 第 196 行**: EN: Starts the definition of function or method `Merger::AllFeatures`. CN: 开始定义函数或方法 `Merger::AllFeatures`。
- **Line 197 / 第 197 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 198 / 第 198 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 199 / 第 199 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 200 / 第 200 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 201 / 第 201 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 202 / 第 202 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 203 / 第 203 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 204 / 第 204 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 205 / 第 205 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 206 / 第 206 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 207 / 第 207 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 208 / 第 208 行**: EN: Declares function or method `IF`. CN: 声明函数或方法 `IF`。

### Lines 209-224 / 第 209-224 行
```cpp
209 |   M.ParseOrExit(IF, false);
210 |   IF.close();
211 |   if (!M.LastFailure.empty())
212 |     Printf("MERGE-INNER: '%s' caused a failure at the previous merge step\n",
213 |            M.LastFailure.c_str());
214 | 
215 |   Printf("MERGE-INNER: %zd total files;"
216 |          " %zd processed earlier; will process %zd files now\n",
217 |          M.Files.size(), M.FirstNotProcessedFile,
218 |          M.Files.size() - M.FirstNotProcessedFile);
219 | 
220 |   std::ofstream OF(CFPath, std::ofstream::out | std::ofstream::app);
221 |   std::set<size_t> AllFeatures;
222 |   auto PrintStatsWrapper = [this, &AllFeatures](const char* Where) {
223 |     this->PrintStats(Where, "\n", 0, AllFeatures.size());
224 |   };
```
- **Line 209 / 第 209 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 210 / 第 210 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 211 / 第 211 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 212 / 第 212 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 213 / 第 213 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 214 / 第 214 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 215 / 第 215 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 216 / 第 216 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 217 / 第 217 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 218 / 第 218 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 219 / 第 219 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 220 / 第 220 行**: EN: Declares function or method `OF`. CN: 声明函数或方法 `OF`。
- **Line 221 / 第 221 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 222 / 第 222 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 223 / 第 223 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 224 / 第 224 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。

### Lines 225-240 / 第 225-240 行
```cpp
225 |   std::set<const TracePC::PCTableEntry *> AllPCs;
226 |   for (size_t i = M.FirstNotProcessedFile; i < M.Files.size(); i++) {
227 |     Fuzzer::MaybeExitGracefully();
228 |     auto U = FileToVector(M.Files[i].Name);
229 |     if (U.size() > MaxInputLen) {
230 |       U.resize(MaxInputLen);
231 |       U.shrink_to_fit();
232 |     }
233 | 
234 |     // Write the pre-run marker.
235 |     OF << "STARTED " << i << " " << U.size() << "\n";
236 |     OF.flush();  // Flush is important since Command::Execute may crash.
237 |     // Run.
238 |     TPC.ResetMaps();
239 |     ExecuteCallback(U.data(), U.size());
240 |     // Collect coverage. We are iterating over the files in this order:
```
- **Line 225 / 第 225 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 226 / 第 226 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 227 / 第 227 行**: EN: Declares function or method `Fuzzer::MaybeExitGracefully`. CN: 声明函数或方法 `Fuzzer::MaybeExitGracefully`。
- **Line 228 / 第 228 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 229 / 第 229 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 230 / 第 230 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 231 / 第 231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 232 / 第 232 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 233 / 第 233 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 234 / 第 234 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 235 / 第 235 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 236 / 第 236 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 237 / 第 237 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 238 / 第 238 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 239 / 第 239 行**: EN: Declares function or method `ExecuteCallback`. CN: 声明函数或方法 `ExecuteCallback`。
- **Line 240 / 第 240 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 241-256 / 第 241-256 行
```cpp
241 |     // * First, files in the initial corpus ordered by size, smallest first.
242 |     // * Then, all other files, smallest first.
243 |     std::set<size_t> Features;
244 |     if (IsSetCoverMerge)
245 |       TPC.CollectFeatures([&](size_t Feature) { Features.insert(Feature); });
246 |     else
247 |       TPC.CollectFeatures([&](size_t Feature) {
248 |         if (AllFeatures.insert(Feature).second)
249 |           Features.insert(Feature);
250 |       });
251 |     TPC.UpdateObservedPCs();
252 |     // Show stats.
253 |     if (!(TotalNumberOfRuns & (TotalNumberOfRuns - 1)))
254 |       PrintStatsWrapper("pulse ");
255 |     if (TotalNumberOfRuns == M.NumFilesInFirstCorpus)
256 |       PrintStatsWrapper("LOADED");
```
- **Line 241 / 第 241 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 242 / 第 242 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 243 / 第 243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 244 / 第 244 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 245 / 第 245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 246 / 第 246 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 247 / 第 247 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 248 / 第 248 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 249 / 第 249 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 250 / 第 250 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 251 / 第 251 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 252 / 第 252 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 253 / 第 253 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 254 / 第 254 行**: EN: Declares function or method `PrintStatsWrapper`. CN: 声明函数或方法 `PrintStatsWrapper`。
- **Line 255 / 第 255 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 256 / 第 256 行**: EN: Declares function or method `PrintStatsWrapper`. CN: 声明函数或方法 `PrintStatsWrapper`。

### Lines 257-272 / 第 257-272 行
```cpp
257 |     // Write the post-run marker and the coverage.
258 |     OF << "FT " << i;
259 |     for (size_t F : Features)
260 |       OF << " " << F;
261 |     OF << "\n";
262 |     OF << "COV " << i;
263 |     TPC.ForEachObservedPC([&](const TracePC::PCTableEntry *TE) {
264 |       if (AllPCs.insert(TE).second)
265 |         OF << " " << TPC.PCTableEntryIdx(TE);
266 |     });
267 |     OF << "\n";
268 |     OF.flush();
269 |   }
270 |   PrintStatsWrapper("DONE  ");
271 | }
272 | 
```
- **Line 257 / 第 257 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 258 / 第 258 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 259 / 第 259 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 260 / 第 260 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 261 / 第 261 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 262 / 第 262 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 263 / 第 263 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 264 / 第 264 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 265 / 第 265 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 266 / 第 266 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 267 / 第 267 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 268 / 第 268 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 269 / 第 269 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 270 / 第 270 行**: EN: Declares function or method `PrintStatsWrapper`. CN: 声明函数或方法 `PrintStatsWrapper`。
- **Line 271 / 第 271 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 272 / 第 272 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 273-288 / 第 273-288 行
```cpp
273 | // Merges all corpora into the first corpus. A file is added into
274 | // the first corpus only if it adds new features. Unlike `Merger::Merge`,
275 | // this implementation calculates an approximation of the minimum set
276 | // of corpora files, that cover all known features (set cover problem).
277 | // Generally, this means that files with more features are preferred for
278 | // merge into the first corpus. When two files have the same number of
279 | // features, the smaller one is preferred.
280 | size_t Merger::SetCoverMerge(const std::set<uint32_t> &InitialFeatures,
281 |                              std::set<uint32_t> *NewFeatures,
282 |                              const std::set<uint32_t> &InitialCov,
283 |                              std::set<uint32_t> *NewCov,
284 |                              std::vector<std::string> *NewFiles) {
285 |   assert(NumFilesInFirstCorpus <= Files.size());
286 |   NewFiles->clear();
287 |   NewFeatures->clear();
288 |   NewCov->clear();
```
- **Line 273 / 第 273 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 274 / 第 274 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 275 / 第 275 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 276 / 第 276 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 277 / 第 277 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 278 / 第 278 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 279 / 第 279 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 280 / 第 280 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 281 / 第 281 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 282 / 第 282 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 283 / 第 283 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 284 / 第 284 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 285 / 第 285 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 286 / 第 286 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 287 / 第 287 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 288 / 第 288 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 289-304 / 第 289-304 行
```cpp
289 |   std::set<uint32_t> AllFeatures;
290 |   // 1 << 21 - 1 is the maximum feature index.
291 |   // See 'kFeatureSetSize' in 'FuzzerCorpus.h'.
292 |   const uint32_t kFeatureSetSize = 1 << 21;
293 |   std::vector<bool> Covered(kFeatureSetSize, false);
294 |   size_t NumCovered = 0;
295 | 
296 |   std::set<uint32_t> ExistingFeatures = InitialFeatures;
297 |   for (size_t i = 0; i < NumFilesInFirstCorpus; ++i)
298 |     ExistingFeatures.insert(Files[i].Features.begin(), Files[i].Features.end());
299 | 
300 |   // Mark the existing features as covered.
301 |   for (const auto &F : ExistingFeatures) {
302 |     if (!Covered[F % kFeatureSetSize]) {
303 |       ++NumCovered;
304 |       Covered[F % kFeatureSetSize] = true;
```
- **Line 289 / 第 289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 290 / 第 290 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 291 / 第 291 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 292 / 第 292 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 293 / 第 293 行**: EN: Declares function or method `Covered`. CN: 声明函数或方法 `Covered`。
- **Line 294 / 第 294 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 295 / 第 295 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 296 / 第 296 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 297 / 第 297 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 298 / 第 298 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 299 / 第 299 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 300 / 第 300 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 301 / 第 301 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 302 / 第 302 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 303 / 第 303 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 304 / 第 304 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 305-320 / 第 305-320 行
```cpp
305 |     }
306 |     // Calculate an underestimation of the set of covered features
307 |     // since the `Covered` bitvector is smaller than the feature range.
308 |     AllFeatures.insert(F % kFeatureSetSize);
309 |   }
310 | 
311 |   std::set<size_t> RemainingFiles;
312 |   for (size_t i = NumFilesInFirstCorpus; i < Files.size(); ++i) {
313 |     // Construct an incremental sequence which represent the
314 |     // indices to all files (excluding those in the initial corpus).
315 |     // RemainingFiles = range(NumFilesInFirstCorpus..Files.size()).
316 |     RemainingFiles.insert(i);
317 |     // Insert this file's unique features to all features.
318 |     for (const auto &F : Files[i].Features)
319 |       AllFeatures.insert(F % kFeatureSetSize);
320 |   }
```
- **Line 305 / 第 305 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 306 / 第 306 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 307 / 第 307 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 308 / 第 308 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 309 / 第 309 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 310 / 第 310 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 311 / 第 311 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 312 / 第 312 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 313 / 第 313 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 314 / 第 314 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 315 / 第 315 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 316 / 第 316 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 317 / 第 317 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 318 / 第 318 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 319 / 第 319 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 320 / 第 320 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 321-336 / 第 321-336 行
```cpp
321 | 
322 |   // Integrate files into Covered until set is complete.
323 |   while (NumCovered != AllFeatures.size()) {
324 |     // Index to file with largest number of unique features.
325 |     size_t MaxFeaturesIndex = NumFilesInFirstCorpus;
326 |     // Indices to remove from RemainingFiles.
327 |     std::set<size_t> RemoveIndices;
328 |     // Running max unique feature count.
329 |     // Updated upon finding a file with more features.
330 |     size_t MaxNumFeatures = 0;
331 | 
332 |     // Iterate over all files not yet integrated into Covered,
333 |     // to find the file which has the largest number of
334 |     // features that are not already in Covered.
335 |     for (const auto &i : RemainingFiles) {
336 |       const auto &File = Files[i];
```
- **Line 321 / 第 321 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 322 / 第 322 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 323 / 第 323 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 324 / 第 324 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 325 / 第 325 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 326 / 第 326 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 327 / 第 327 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 328 / 第 328 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 329 / 第 329 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 330 / 第 330 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 331 / 第 331 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 332 / 第 332 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 333 / 第 333 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 334 / 第 334 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 335 / 第 335 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 336 / 第 336 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 337-352 / 第 337-352 行
```cpp
337 |       size_t CurrentUnique = 0;
338 |       // Count number of features in this file
339 |       // which are not yet in Covered.
340 |       for (const auto &F : File.Features)
341 |         if (!Covered[F % kFeatureSetSize])
342 |           ++CurrentUnique;
343 | 
344 |       if (CurrentUnique == 0) {
345 |         // All features in this file are already in Covered: skip next time.
346 |         RemoveIndices.insert(i);
347 |       } else if (CurrentUnique > MaxNumFeatures ||
348 |                  (CurrentUnique == MaxNumFeatures &&
349 |                   File.Size < Files[MaxFeaturesIndex].Size)) {
350 |         // Update the max features file based on unique features
351 |         // Break ties by selecting smaller files.
352 |         MaxNumFeatures = CurrentUnique;
```
- **Line 337 / 第 337 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 338 / 第 338 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 339 / 第 339 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 340 / 第 340 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 341 / 第 341 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 342 / 第 342 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 343 / 第 343 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 344 / 第 344 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 345 / 第 345 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 346 / 第 346 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 347 / 第 347 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 348 / 第 348 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 349 / 第 349 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 350 / 第 350 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 351 / 第 351 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 352 / 第 352 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 353-368 / 第 353-368 行
```cpp
353 |         MaxFeaturesIndex = i;
354 |       }
355 |     }
356 |     // Must be a valid index/
357 |     assert(MaxFeaturesIndex < Files.size());
358 |     // Remove any feature-less files found.
359 |     for (const auto &i : RemoveIndices)
360 |       RemainingFiles.erase(i);
361 |     if (MaxNumFeatures == 0) {
362 |       // Did not find a file that adds unique features.
363 |       // This means that we should have no remaining files.
364 |       assert(RemainingFiles.size() == 0);
365 |       assert(NumCovered == AllFeatures.size());
366 |       break;
367 |     }
368 | 
```
- **Line 353 / 第 353 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 354 / 第 354 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 355 / 第 355 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 356 / 第 356 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 357 / 第 357 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 358 / 第 358 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 359 / 第 359 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 360 / 第 360 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 361 / 第 361 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 362 / 第 362 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 363 / 第 363 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 364 / 第 364 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 365 / 第 365 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 366 / 第 366 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 367 / 第 367 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 368 / 第 368 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 369-384 / 第 369-384 行
```cpp
369 |     // MaxFeaturesIndex must be an element of Remaining.
370 |     assert(RemainingFiles.find(MaxFeaturesIndex) != RemainingFiles.end());
371 |     // Remove the file with the most features from Remaining.
372 |     RemainingFiles.erase(MaxFeaturesIndex);
373 |     const auto &MaxFeatureFile = Files[MaxFeaturesIndex];
374 |     // Add the features of the max feature file to Covered.
375 |     for (const auto &F : MaxFeatureFile.Features) {
376 |       if (!Covered[F % kFeatureSetSize]) {
377 |         ++NumCovered;
378 |         Covered[F % kFeatureSetSize] = true;
379 |         NewFeatures->insert(F);
380 |       }
381 |     }
382 |     // Add the index to this file to the result.
383 |     NewFiles->push_back(MaxFeatureFile.Name);
384 |     // Update NewCov with the additional coverage
```
- **Line 369 / 第 369 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 370 / 第 370 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 371 / 第 371 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 372 / 第 372 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 373 / 第 373 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 374 / 第 374 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 375 / 第 375 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 376 / 第 376 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 377 / 第 377 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 378 / 第 378 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 379 / 第 379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 380 / 第 380 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 381 / 第 381 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 382 / 第 382 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 383 / 第 383 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 384 / 第 384 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 385-400 / 第 385-400 行
```cpp
385 |     // that MaxFeatureFile provides.
386 |     for (const auto &C : MaxFeatureFile.Cov)
387 |       if (InitialCov.find(C) == InitialCov.end())
388 |         NewCov->insert(C);
389 |   }
390 | 
391 |   return NewFeatures->size();
392 | }
393 | 
394 | static size_t
395 | WriteNewControlFile(const std::string &CFPath,
396 |                     const std::vector<SizedFile> &OldCorpus,
397 |                     const std::vector<SizedFile> &NewCorpus,
398 |                     const std::vector<MergeFileInfo> &KnownFiles) {
399 |   std::unordered_set<std::string> FilesToSkip;
400 |   for (auto &SF: KnownFiles)
```
- **Line 385 / 第 385 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 386 / 第 386 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 387 / 第 387 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 388 / 第 388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 389 / 第 389 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 390 / 第 390 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 391 / 第 391 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 392 / 第 392 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 393 / 第 393 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 394 / 第 394 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 395 / 第 395 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 396 / 第 396 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 397 / 第 397 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 398 / 第 398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 399 / 第 399 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 400 / 第 400 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。

### Lines 401-416 / 第 401-416 行
```cpp
401 |     FilesToSkip.insert(SF.Name);
402 | 
403 |   std::vector<std::string> FilesToUse;
404 |   auto MaybeUseFile = [=, &FilesToUse](std::string Name) {
405 |     if (FilesToSkip.find(Name) == FilesToSkip.end())
406 |       FilesToUse.push_back(Name);
407 |   };
408 |   for (auto &SF: OldCorpus)
409 |     MaybeUseFile(SF.File);
410 |   auto FilesToUseFromOldCorpus = FilesToUse.size();
411 |   for (auto &SF: NewCorpus)
412 |     MaybeUseFile(SF.File);
413 | 
414 |   RemoveFile(CFPath);
415 |   std::ofstream ControlFile(CFPath);
416 |   ControlFile << FilesToUse.size() << "\n";
```
- **Line 401 / 第 401 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 402 / 第 402 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 403 / 第 403 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 404 / 第 404 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 405 / 第 405 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 406 / 第 406 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 407 / 第 407 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 408 / 第 408 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 409 / 第 409 行**: EN: Declares function or method `MaybeUseFile`. CN: 声明函数或方法 `MaybeUseFile`。
- **Line 410 / 第 410 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 411 / 第 411 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 412 / 第 412 行**: EN: Declares function or method `MaybeUseFile`. CN: 声明函数或方法 `MaybeUseFile`。
- **Line 413 / 第 413 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 414 / 第 414 行**: EN: Declares function or method `RemoveFile`. CN: 声明函数或方法 `RemoveFile`。
- **Line 415 / 第 415 行**: EN: Declares function or method `ControlFile`. CN: 声明函数或方法 `ControlFile`。
- **Line 416 / 第 416 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 417-432 / 第 417-432 行
```cpp
417 |   ControlFile << FilesToUseFromOldCorpus << "\n";
418 |   for (auto &FN: FilesToUse)
419 |     ControlFile << FN << "\n";
420 | 
421 |   if (!ControlFile) {
422 |     Printf("MERGE-OUTER: failed to write to the control file: %s\n",
423 |            CFPath.c_str());
424 |     exit(1);
425 |   }
426 | 
427 |   return FilesToUse.size();
428 | }
429 | 
430 | // Outer process. Does not call the target code and thus should not fail.
431 | void CrashResistantMerge(const std::vector<std::string> &Args,
432 |                          const std::vector<SizedFile> &OldCorpus,
```
- **Line 417 / 第 417 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 418 / 第 418 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 419 / 第 419 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 420 / 第 420 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 421 / 第 421 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 422 / 第 422 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 423 / 第 423 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 424 / 第 424 行**: EN: Declares function or method `exit`. CN: 声明函数或方法 `exit`。
- **Line 425 / 第 425 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 426 / 第 426 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 427 / 第 427 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 428 / 第 428 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 429 / 第 429 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 430 / 第 430 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 431 / 第 431 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 432 / 第 432 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 433-448 / 第 433-448 行
```cpp
433 |                          const std::vector<SizedFile> &NewCorpus,
434 |                          std::vector<std::string> *NewFiles,
435 |                          const std::set<uint32_t> &InitialFeatures,
436 |                          std::set<uint32_t> *NewFeatures,
437 |                          const std::set<uint32_t> &InitialCov,
438 |                          std::set<uint32_t> *NewCov, const std::string &CFPath,
439 |                          bool V, /*Verbose*/
440 |                          bool IsSetCoverMerge) {
441 |   if (NewCorpus.empty() && OldCorpus.empty()) return;  // Nothing to merge.
442 |   size_t NumAttempts = 0;
443 |   std::vector<MergeFileInfo> KnownFiles;
444 |   if (FileSize(CFPath)) {
445 |     VPrintf(V, "MERGE-OUTER: non-empty control file provided: '%s'\n",
446 |            CFPath.c_str());
447 |     Merger M;
448 |     std::ifstream IF(CFPath);
```
- **Line 433 / 第 433 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 434 / 第 434 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 435 / 第 435 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 436 / 第 436 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 437 / 第 437 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 438 / 第 438 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 439 / 第 439 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 440 / 第 440 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 441 / 第 441 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 442 / 第 442 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 443 / 第 443 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 444 / 第 444 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 445 / 第 445 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 446 / 第 446 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 447 / 第 447 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 448 / 第 448 行**: EN: Declares function or method `IF`. CN: 声明函数或方法 `IF`。

### Lines 449-464 / 第 449-464 行
```cpp
449 |     if (M.Parse(IF, /*ParseCoverage=*/true)) {
450 |       VPrintf(V, "MERGE-OUTER: control file ok, %zd files total,"
451 |              " first not processed file %zd\n",
452 |              M.Files.size(), M.FirstNotProcessedFile);
453 |       if (!M.LastFailure.empty())
454 |         VPrintf(V, "MERGE-OUTER: '%s' will be skipped as unlucky "
455 |                "(merge has stumbled on it the last time)\n",
456 |                M.LastFailure.c_str());
457 |       if (M.FirstNotProcessedFile >= M.Files.size()) {
458 |         // Merge has already been completed with the given merge control file.
459 |         if (M.Files.size() == OldCorpus.size() + NewCorpus.size()) {
460 |           VPrintf(
461 |               V,
462 |               "MERGE-OUTER: nothing to do, merge has been completed before\n");
463 |           exit(0);
464 |         }
```
- **Line 449 / 第 449 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 450 / 第 450 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 451 / 第 451 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 452 / 第 452 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 453 / 第 453 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 454 / 第 454 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 455 / 第 455 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 456 / 第 456 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 457 / 第 457 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 458 / 第 458 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 459 / 第 459 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 460 / 第 460 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 461 / 第 461 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 462 / 第 462 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 463 / 第 463 行**: EN: Declares function or method `exit`. CN: 声明函数或方法 `exit`。
- **Line 464 / 第 464 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 465-480 / 第 465-480 行
```cpp
465 | 
466 |         // Number of input files likely changed, start merge from scratch, but
467 |         // reuse coverage information from the given merge control file.
468 |         VPrintf(
469 |             V,
470 |             "MERGE-OUTER: starting merge from scratch, but reusing coverage "
471 |             "information from the given control file\n");
472 |         KnownFiles = M.Files;
473 |       } else {
474 |         // There is a merge in progress, continue.
475 |         NumAttempts = M.Files.size() - M.FirstNotProcessedFile;
476 |       }
477 |     } else {
478 |       VPrintf(V, "MERGE-OUTER: bad control file, will overwrite it\n");
479 |     }
480 |   }
```
- **Line 465 / 第 465 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 466 / 第 466 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 467 / 第 467 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 468 / 第 468 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 469 / 第 469 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 470 / 第 470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 471 / 第 471 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 472 / 第 472 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 473 / 第 473 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 474 / 第 474 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 475 / 第 475 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 476 / 第 476 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 477 / 第 477 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 478 / 第 478 行**: EN: Declares function or method `VPrintf`. CN: 声明函数或方法 `VPrintf`。
- **Line 479 / 第 479 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 480 / 第 480 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 481-496 / 第 481-496 行
```cpp
481 | 
482 |   if (!NumAttempts) {
483 |     // The supplied control file is empty or bad, create a fresh one.
484 |     VPrintf(V, "MERGE-OUTER: "
485 |             "%zd files, %zd in the initial corpus, %zd processed earlier\n",
486 |             OldCorpus.size() + NewCorpus.size(), OldCorpus.size(),
487 |             KnownFiles.size());
488 |     NumAttempts = WriteNewControlFile(CFPath, OldCorpus, NewCorpus, KnownFiles);
489 |   }
490 | 
491 |   // Execute the inner process until it passes.
492 |   // Every inner process should execute at least one input.
493 |   Command BaseCmd(Args);
494 |   BaseCmd.removeFlag("merge");
495 |   BaseCmd.removeFlag("set_cover_merge");
496 |   BaseCmd.removeFlag("fork");
```
- **Line 481 / 第 481 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 482 / 第 482 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 483 / 第 483 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 484 / 第 484 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 485 / 第 485 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 486 / 第 486 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 487 / 第 487 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 488 / 第 488 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 489 / 第 489 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 490 / 第 490 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 491 / 第 491 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 492 / 第 492 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 493 / 第 493 行**: EN: Declares function or method `BaseCmd`. CN: 声明函数或方法 `BaseCmd`。
- **Line 494 / 第 494 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 495 / 第 495 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 496 / 第 496 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 497-512 / 第 497-512 行
```cpp
497 |   BaseCmd.removeFlag("collect_data_flow");
498 |   for (size_t Attempt = 1; Attempt <= NumAttempts; Attempt++) {
499 |     Fuzzer::MaybeExitGracefully();
500 |     VPrintf(V, "MERGE-OUTER: attempt %zd\n", Attempt);
501 |     Command Cmd(BaseCmd);
502 |     Cmd.addFlag("merge_control_file", CFPath);
503 |     // If we are going to use the set cover implementation for
504 |     // minimization add the merge_inner=2 internal flag.
505 |     Cmd.addFlag("merge_inner", IsSetCoverMerge ? "2" : "1");
506 |     if (!V) {
507 |       Cmd.setOutputFile(getDevNull());
508 |       Cmd.combineOutAndErr();
509 |     }
510 |     auto ExitCode = ExecuteCommand(Cmd);
511 |     if (!ExitCode) {
512 |       VPrintf(V, "MERGE-OUTER: successful in %zd attempt(s)\n", Attempt);
```
- **Line 497 / 第 497 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 498 / 第 498 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 499 / 第 499 行**: EN: Declares function or method `Fuzzer::MaybeExitGracefully`. CN: 声明函数或方法 `Fuzzer::MaybeExitGracefully`。
- **Line 500 / 第 500 行**: EN: Declares function or method `VPrintf`. CN: 声明函数或方法 `VPrintf`。
- **Line 501 / 第 501 行**: EN: Declares function or method `Cmd`. CN: 声明函数或方法 `Cmd`。
- **Line 502 / 第 502 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 503 / 第 503 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 504 / 第 504 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 505 / 第 505 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 506 / 第 506 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 507 / 第 507 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 508 / 第 508 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 509 / 第 509 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 510 / 第 510 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 511 / 第 511 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 512 / 第 512 行**: EN: Declares function or method `VPrintf`. CN: 声明函数或方法 `VPrintf`。

### Lines 513-528 / 第 513-528 行
```cpp
513 |       break;
514 |     }
515 |   }
516 |   // Read the control file and do the merge.
517 |   Merger M;
518 |   std::ifstream IF(CFPath);
519 |   IF.seekg(0, IF.end);
520 |   VPrintf(V, "MERGE-OUTER: the control file has %zd bytes\n",
521 |           (size_t)IF.tellg());
522 |   IF.seekg(0, IF.beg);
523 |   M.ParseOrExit(IF, true);
524 |   IF.close();
525 |   VPrintf(V,
526 |           "MERGE-OUTER: consumed %zdMb (%zdMb rss) to parse the control file\n",
527 |           M.ApproximateMemoryConsumption() >> 20, GetPeakRSSMb());
528 | 
```
- **Line 513 / 第 513 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 514 / 第 514 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 515 / 第 515 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 516 / 第 516 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 517 / 第 517 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 518 / 第 518 行**: EN: Declares function or method `IF`. CN: 声明函数或方法 `IF`。
- **Line 519 / 第 519 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 520 / 第 520 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 521 / 第 521 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 522 / 第 522 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 523 / 第 523 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 524 / 第 524 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 525 / 第 525 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 526 / 第 526 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 527 / 第 527 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 528 / 第 528 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 529-539 / 第 529-539 行
```cpp
529 |   M.Files.insert(M.Files.end(), KnownFiles.begin(), KnownFiles.end());
530 |   if (IsSetCoverMerge)
531 |     M.SetCoverMerge(InitialFeatures, NewFeatures, InitialCov, NewCov, NewFiles);
532 |   else
533 |     M.Merge(InitialFeatures, NewFeatures, InitialCov, NewCov, NewFiles);
534 |   VPrintf(V, "MERGE-OUTER: %zd new files with %zd new features added; "
535 |           "%zd new coverage edges\n",
536 |          NewFiles->size(), NewFeatures->size(), NewCov->size());
537 | }
538 | 
539 | } // namespace fuzzer
```
- **Line 529 / 第 529 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 530 / 第 530 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 531 / 第 531 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 532 / 第 532 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 533 / 第 533 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 534 / 第 534 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 535 / 第 535 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 536 / 第 536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 537 / 第 537 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 538 / 第 538 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 539 / 第 539 行**: EN: Closes a named namespace scope. CN: 关闭一个具名命名空间作用域。

## Key Concepts / 关键概念

- **EN**: fuzz input decomposition
  - **CN**: 模糊测试输入拆分
- **EN**: deterministic test input consumption
  - **CN**: 确定性的测试输入消费
- **EN**: coverage-guided fuzzing runtime
  - **CN**: 覆盖率引导的 fuzzing 运行时
- **EN**: namespace scoping and organization
  - **CN**: 命名空间作用域与组织

## Dependencies / 依赖关系

- `FuzzerCommand.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerMerge.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerIO.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerInternal.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerTracePC.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerUtil.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `fstream` — System or standard library dependency / 系统或标准库依赖
- `iterator` — System or standard library dependency / 系统或标准库依赖
- `set` — System or standard library dependency / 系统或标准库依赖
- `sstream` — System or standard library dependency / 系统或标准库依赖
- `unordered_set` — System or standard library dependency / 系统或标准库依赖
