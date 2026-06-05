# FuzzerCorpus.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/fuzzer/FuzzerCorpus.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Declares libFuzzer components related to `FuzzerCorpus`.
  - **CN**: 声明 libFuzzer 中与 `FuzzerCorpus` 相关的组件或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
```cpp
 1 | //===- FuzzerCorpus.h - Internal header for the Fuzzer ----------*- C++ -* ===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | // fuzzer::InputCorpus
 9 | //===----------------------------------------------------------------------===//
10 | 
11 | #ifndef LLVM_FUZZER_CORPUS
12 | #define LLVM_FUZZER_CORPUS
13 | 
14 | #include "FuzzerDataFlowTrace.h"
15 | #include "FuzzerDefs.h"
16 | #include "FuzzerIO.h"
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
- **Line 11 / 第 11 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 12 / 第 12 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 13 / 第 13 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 14 / 第 14 行**: EN: Includes `FuzzerDataFlowTrace.h` so this file can use its declarations. CN: 包含 `FuzzerDataFlowTrace.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Includes `FuzzerDefs.h` so this file can use its declarations. CN: 包含 `FuzzerDefs.h`，以便当前文件使用其中的声明。
- **Line 16 / 第 16 行**: EN: Includes `FuzzerIO.h` so this file can use its declarations. CN: 包含 `FuzzerIO.h`，以便当前文件使用其中的声明。

### Lines 17-32 / 第 17-32 行
```cpp
17 | #include "FuzzerRandom.h"
18 | #include "FuzzerSHA1.h"
19 | #include "FuzzerTracePC.h"
20 | #include <algorithm>
21 | #include <bitset>
22 | #include <chrono>
23 | #include <numeric>
24 | #include <random>
25 | #include <unordered_set>
26 | 
27 | namespace fuzzer {
28 | 
29 | struct InputInfo {
30 |   Unit U;  // The actual input data.
31 |   std::chrono::microseconds TimeOfUnit;
32 |   uint8_t Sha1[kSHA1NumBytes];  // Checksum.
```
- **Line 17 / 第 17 行**: EN: Includes `FuzzerRandom.h` so this file can use its declarations. CN: 包含 `FuzzerRandom.h`，以便当前文件使用其中的声明。
- **Line 18 / 第 18 行**: EN: Includes `FuzzerSHA1.h` so this file can use its declarations. CN: 包含 `FuzzerSHA1.h`，以便当前文件使用其中的声明。
- **Line 19 / 第 19 行**: EN: Includes `FuzzerTracePC.h` so this file can use its declarations. CN: 包含 `FuzzerTracePC.h`，以便当前文件使用其中的声明。
- **Line 20 / 第 20 行**: EN: Includes `algorithm` so this file can use its declarations. CN: 包含 `algorithm`，以便当前文件使用其中的声明。
- **Line 21 / 第 21 行**: EN: Includes `bitset` so this file can use its declarations. CN: 包含 `bitset`，以便当前文件使用其中的声明。
- **Line 22 / 第 22 行**: EN: Includes `chrono` so this file can use its declarations. CN: 包含 `chrono`，以便当前文件使用其中的声明。
- **Line 23 / 第 23 行**: EN: Includes `numeric` so this file can use its declarations. CN: 包含 `numeric`，以便当前文件使用其中的声明。
- **Line 24 / 第 24 行**: EN: Includes `random` so this file can use its declarations. CN: 包含 `random`，以便当前文件使用其中的声明。
- **Line 25 / 第 25 行**: EN: Includes `unordered_set` so this file can use its declarations. CN: 包含 `unordered_set`，以便当前文件使用其中的声明。
- **Line 26 / 第 26 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 27 / 第 27 行**: EN: Opens namespace `fuzzer` to scope related declarations. CN: 打开命名空间 `fuzzer`，为相关声明建立作用域。
- **Line 28 / 第 28 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 29 / 第 29 行**: EN: Begins the declaration of struct `InputInfo`. CN: 开始声明 struct `InputInfo`。
- **Line 30 / 第 30 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 31 / 第 31 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 32 / 第 32 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 33-48 / 第 33-48 行
```cpp
33 |   // Number of features that this input has and no smaller input has.
34 |   size_t NumFeatures = 0;
35 |   size_t Tmp = 0; // Used by ValidateFeatureSet.
36 |   // Stats.
37 |   size_t NumExecutedMutations = 0;
38 |   size_t NumSuccessfulMutations = 0;
39 |   bool NeverReduce = false;
40 |   bool MayDeleteFile = false;
41 |   bool Reduced = false;
42 |   bool HasFocusFunction = false;
43 |   std::vector<uint32_t> UniqFeatureSet;
44 |   std::vector<uint8_t> DataFlowTraceForFocusFunction;
45 |   // Power schedule.
46 |   bool NeedsEnergyUpdate = false;
47 |   double Energy = 0.0;
48 |   double SumIncidence = 0.0;
```
- **Line 33 / 第 33 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 34 / 第 34 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 35 / 第 35 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 36 / 第 36 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 37 / 第 37 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 38 / 第 38 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 39 / 第 39 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 40 / 第 40 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 41 / 第 41 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 42 / 第 42 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 43 / 第 43 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 44 / 第 44 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 45 / 第 45 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 46 / 第 46 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 47 / 第 47 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 48 / 第 48 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 49-64 / 第 49-64 行
```cpp
49 |   std::vector<std::pair<uint32_t, uint16_t>> FeatureFreqs;
50 | 
51 |   // Delete feature Idx and its frequency from FeatureFreqs.
52 |   bool DeleteFeatureFreq(uint32_t Idx) {
53 |     if (FeatureFreqs.empty())
54 |       return false;
55 | 
56 |     // Binary search over local feature frequencies sorted by index.
57 |     auto Lower = std::lower_bound(FeatureFreqs.begin(), FeatureFreqs.end(),
58 |                                   std::pair<uint32_t, uint16_t>(Idx, 0));
59 | 
60 |     if (Lower != FeatureFreqs.end() && Lower->first == Idx) {
61 |       FeatureFreqs.erase(Lower);
62 |       return true;
63 |     }
64 |     return false;
```
- **Line 49 / 第 49 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 50 / 第 50 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 51 / 第 51 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 52 / 第 52 行**: EN: Starts the definition of function or method `DeleteFeatureFreq`. CN: 开始定义函数或方法 `DeleteFeatureFreq`。
- **Line 53 / 第 53 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 54 / 第 54 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 55 / 第 55 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 56 / 第 56 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 57 / 第 57 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 58 / 第 58 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 59 / 第 59 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 60 / 第 60 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 61 / 第 61 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 62 / 第 62 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 63 / 第 63 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 64 / 第 64 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 65-80 / 第 65-80 行
```cpp
65 |   }
66 | 
67 |   // Assign more energy to a high-entropy seed, i.e., that reveals more
68 |   // information about the globally rare features in the neighborhood of the
69 |   // seed. Since we do not know the entropy of a seed that has never been
70 |   // executed we assign fresh seeds maximum entropy and let II->Energy approach
71 |   // the true entropy from above. If ScalePerExecTime is true, the computed
72 |   // entropy is scaled based on how fast this input executes compared to the
73 |   // average execution time of inputs. The faster an input executes, the more
74 |   // energy gets assigned to the input.
75 |   void UpdateEnergy(size_t GlobalNumberOfFeatures, bool ScalePerExecTime,
76 |                     std::chrono::microseconds AverageUnitExecutionTime) {
77 |     Energy = 0.0;
78 |     SumIncidence = 0.0;
79 | 
80 |     // Apply add-one smoothing to locally discovered features.
```
- **Line 65 / 第 65 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 66 / 第 66 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 67 / 第 67 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 68 / 第 68 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 69 / 第 69 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 70 / 第 70 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 71 / 第 71 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 72 / 第 72 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 73 / 第 73 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 74 / 第 74 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 75 / 第 75 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 76 / 第 76 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 77 / 第 77 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 78 / 第 78 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 79 / 第 79 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 80 / 第 80 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 81-96 / 第 81-96 行
```cpp
81 |     for (const auto &F : FeatureFreqs) {
82 |       double LocalIncidence = F.second + 1;
83 |       Energy -= LocalIncidence * log(LocalIncidence);
84 |       SumIncidence += LocalIncidence;
85 |     }
86 | 
87 |     // Apply add-one smoothing to locally undiscovered features.
88 |     //   PreciseEnergy -= 0; // since log(1.0) == 0)
89 |     SumIncidence +=
90 |         static_cast<double>(GlobalNumberOfFeatures - FeatureFreqs.size());
91 | 
92 |     // Add a single locally abundant feature apply add-one smoothing.
93 |     double AbdIncidence = static_cast<double>(NumExecutedMutations + 1);
94 |     Energy -= AbdIncidence * log(AbdIncidence);
95 |     SumIncidence += AbdIncidence;
96 | 
```
- **Line 81 / 第 81 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 82 / 第 82 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 83 / 第 83 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 84 / 第 84 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 85 / 第 85 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 86 / 第 86 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 87 / 第 87 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 88 / 第 88 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 89 / 第 89 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 90 / 第 90 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 91 / 第 91 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 92 / 第 92 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 93 / 第 93 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 94 / 第 94 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 95 / 第 95 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 96 / 第 96 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 97-112 / 第 97-112 行
```cpp
 97 |     // Normalize.
 98 |     if (SumIncidence != 0)
 99 |       Energy = Energy / SumIncidence + log(SumIncidence);
100 | 
101 |     if (ScalePerExecTime) {
102 |       // Scaling to favor inputs with lower execution time.
103 |       uint32_t PerfScore = 100;
104 |       if (TimeOfUnit.count() > AverageUnitExecutionTime.count() * 10)
105 |         PerfScore = 10;
106 |       else if (TimeOfUnit.count() > AverageUnitExecutionTime.count() * 4)
107 |         PerfScore = 25;
108 |       else if (TimeOfUnit.count() > AverageUnitExecutionTime.count() * 2)
109 |         PerfScore = 50;
110 |       else if (TimeOfUnit.count() * 3 > AverageUnitExecutionTime.count() * 4)
111 |         PerfScore = 75;
112 |       else if (TimeOfUnit.count() * 4 < AverageUnitExecutionTime.count())
```
- **Line 97 / 第 97 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 98 / 第 98 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 99 / 第 99 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 100 / 第 100 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 101 / 第 101 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 102 / 第 102 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 103 / 第 103 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 104 / 第 104 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 105 / 第 105 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 106 / 第 106 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 107 / 第 107 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 108 / 第 108 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 109 / 第 109 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 110 / 第 110 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 111 / 第 111 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 112 / 第 112 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。

### Lines 113-128 / 第 113-128 行
```cpp
113 |         PerfScore = 300;
114 |       else if (TimeOfUnit.count() * 3 < AverageUnitExecutionTime.count())
115 |         PerfScore = 200;
116 |       else if (TimeOfUnit.count() * 2 < AverageUnitExecutionTime.count())
117 |         PerfScore = 150;
118 | 
119 |       Energy *= PerfScore;
120 |     }
121 |   }
122 | 
123 |   // Increment the frequency of the feature Idx.
124 |   void UpdateFeatureFrequency(uint32_t Idx) {
125 |     NeedsEnergyUpdate = true;
126 | 
127 |     // The local feature frequencies is an ordered vector of pairs.
128 |     // If there are no local feature frequencies, push_back preserves order.
```
- **Line 113 / 第 113 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 114 / 第 114 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 115 / 第 115 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 116 / 第 116 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 117 / 第 117 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 118 / 第 118 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 119 / 第 119 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 120 / 第 120 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 121 / 第 121 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 122 / 第 122 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 123 / 第 123 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 124 / 第 124 行**: EN: Starts the definition of function or method `UpdateFeatureFrequency`. CN: 开始定义函数或方法 `UpdateFeatureFrequency`。
- **Line 125 / 第 125 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 126 / 第 126 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 127 / 第 127 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 128 / 第 128 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 129-144 / 第 129-144 行
```cpp
129 |     // Set the feature frequency for feature Idx32 to 1.
130 |     if (FeatureFreqs.empty()) {
131 |       FeatureFreqs.push_back(std::pair<uint32_t, uint16_t>(Idx, 1));
132 |       return;
133 |     }
134 | 
135 |     // Binary search over local feature frequencies sorted by index.
136 |     auto Lower = std::lower_bound(FeatureFreqs.begin(), FeatureFreqs.end(),
137 |                                   std::pair<uint32_t, uint16_t>(Idx, 0));
138 | 
139 |     // If feature Idx32 already exists, increment its frequency.
140 |     // Otherwise, insert a new pair right after the next lower index.
141 |     if (Lower != FeatureFreqs.end() && Lower->first == Idx) {
142 |       Lower->second++;
143 |     } else {
144 |       FeatureFreqs.insert(Lower, std::pair<uint32_t, uint16_t>(Idx, 1));
```
- **Line 129 / 第 129 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 130 / 第 130 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 131 / 第 131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 132 / 第 132 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 133 / 第 133 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 134 / 第 134 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 135 / 第 135 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 136 / 第 136 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 137 / 第 137 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 138 / 第 138 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 139 / 第 139 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 140 / 第 140 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 141 / 第 141 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 142 / 第 142 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 143 / 第 143 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 144 / 第 144 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 145-160 / 第 145-160 行
```cpp
145 |     }
146 |   }
147 | };
148 | 
149 | struct EntropicOptions {
150 |   bool Enabled;
151 |   size_t NumberOfRarestFeatures;
152 |   size_t FeatureFrequencyThreshold;
153 |   bool ScalePerExecTime;
154 | };
155 | 
156 | class InputCorpus {
157 |   static const uint32_t kFeatureSetSize = 1 << 21;
158 |   static const uint8_t kMaxMutationFactor = 20;
159 |   static const size_t kSparseEnergyUpdates = 100;
160 | 
```
- **Line 145 / 第 145 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 146 / 第 146 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 147 / 第 147 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 148 / 第 148 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 149 / 第 149 行**: EN: Begins the declaration of struct `EntropicOptions`. CN: 开始声明 struct `EntropicOptions`。
- **Line 150 / 第 150 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 151 / 第 151 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 152 / 第 152 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 153 / 第 153 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 154 / 第 154 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 155 / 第 155 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 156 / 第 156 行**: EN: Begins the declaration of class `InputCorpus`. CN: 开始声明 class `InputCorpus`。
- **Line 157 / 第 157 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 158 / 第 158 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 159 / 第 159 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 160 / 第 160 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 161-176 / 第 161-176 行
```cpp
161 |   size_t NumExecutedMutations = 0;
162 | 
163 |   EntropicOptions Entropic;
164 | 
165 | public:
166 |   InputCorpus(const std::string &OutputCorpus, EntropicOptions Entropic)
167 |       : Entropic(Entropic), OutputCorpus(OutputCorpus) {
168 |     memset(InputSizesPerFeature, 0, sizeof(InputSizesPerFeature));
169 |     memset(SmallestElementPerFeature, 0, sizeof(SmallestElementPerFeature));
170 |   }
171 |   ~InputCorpus() {
172 |     for (auto II : Inputs)
173 |       delete II;
174 |   }
175 |   size_t size() const { return Inputs.size(); }
176 |   size_t SizeInBytes() const {
```
- **Line 161 / 第 161 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 162 / 第 162 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 163 / 第 163 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 164 / 第 164 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 165 / 第 165 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 166 / 第 166 行**: EN: Starts the definition of function or method `InputCorpus`. CN: 开始定义函数或方法 `InputCorpus`。
- **Line 167 / 第 167 行**: EN: Starts the definition of function or method `Entropic`. CN: 开始定义函数或方法 `Entropic`。
- **Line 168 / 第 168 行**: EN: Declares function or method `memset`. CN: 声明函数或方法 `memset`。
- **Line 169 / 第 169 行**: EN: Declares function or method `memset`. CN: 声明函数或方法 `memset`。
- **Line 170 / 第 170 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 171 / 第 171 行**: EN: Starts the definition of function or method `~InputCorpus`. CN: 开始定义函数或方法 `~InputCorpus`。
- **Line 172 / 第 172 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 173 / 第 173 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 174 / 第 174 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 175 / 第 175 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 176 / 第 176 行**: EN: Starts the definition of function or method `SizeInBytes`. CN: 开始定义函数或方法 `SizeInBytes`。

### Lines 177-192 / 第 177-192 行
```cpp
177 |     size_t Res = 0;
178 |     for (auto II : Inputs)
179 |       Res += II->U.size();
180 |     return Res;
181 |   }
182 |   size_t NumActiveUnits() const {
183 |     size_t Res = 0;
184 |     for (auto II : Inputs)
185 |       Res += !II->U.empty();
186 |     return Res;
187 |   }
188 |   size_t MaxInputSize() const {
189 |     size_t Res = 0;
190 |     for (auto II : Inputs)
191 |         Res = std::max(Res, II->U.size());
192 |     return Res;
```
- **Line 177 / 第 177 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 178 / 第 178 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 179 / 第 179 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 180 / 第 180 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 181 / 第 181 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 182 / 第 182 行**: EN: Starts the definition of function or method `NumActiveUnits`. CN: 开始定义函数或方法 `NumActiveUnits`。
- **Line 183 / 第 183 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 184 / 第 184 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 185 / 第 185 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 186 / 第 186 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 187 / 第 187 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 188 / 第 188 行**: EN: Starts the definition of function or method `MaxInputSize`. CN: 开始定义函数或方法 `MaxInputSize`。
- **Line 189 / 第 189 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 190 / 第 190 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 191 / 第 191 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 192 / 第 192 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 193-208 / 第 193-208 行
```cpp
193 |   }
194 |   void IncrementNumExecutedMutations() { NumExecutedMutations++; }
195 | 
196 |   size_t NumInputsThatTouchFocusFunction() {
197 |     return std::count_if(Inputs.begin(), Inputs.end(), [](const InputInfo *II) {
198 |       return II->HasFocusFunction;
199 |     });
200 |   }
201 | 
202 |   size_t NumInputsWithDataFlowTrace() {
203 |     return std::count_if(Inputs.begin(), Inputs.end(), [](const InputInfo *II) {
204 |       return !II->DataFlowTraceForFocusFunction.empty();
205 |     });
206 |   }
207 | 
208 |   bool empty() const { return Inputs.empty(); }
```
- **Line 193 / 第 193 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 194 / 第 194 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 195 / 第 195 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 196 / 第 196 行**: EN: Starts the definition of function or method `NumInputsThatTouchFocusFunction`. CN: 开始定义函数或方法 `NumInputsThatTouchFocusFunction`。
- **Line 197 / 第 197 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 198 / 第 198 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 199 / 第 199 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 200 / 第 200 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 201 / 第 201 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 202 / 第 202 行**: EN: Starts the definition of function or method `NumInputsWithDataFlowTrace`. CN: 开始定义函数或方法 `NumInputsWithDataFlowTrace`。
- **Line 203 / 第 203 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 204 / 第 204 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 205 / 第 205 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 206 / 第 206 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 207 / 第 207 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 208 / 第 208 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 209-224 / 第 209-224 行
```cpp
209 |   const Unit &operator[] (size_t Idx) const { return Inputs[Idx]->U; }
210 |   InputInfo *AddToCorpus(const Unit &U, size_t NumFeatures, bool MayDeleteFile,
211 |                          bool HasFocusFunction, bool NeverReduce,
212 |                          std::chrono::microseconds TimeOfUnit,
213 |                          const std::vector<uint32_t> &FeatureSet,
214 |                          const DataFlowTrace &DFT, const InputInfo *BaseII) {
215 |     assert(!U.empty());
216 |     if (FeatureDebug)
217 |       Printf("ADD_TO_CORPUS %zd NF %zd\n", Inputs.size(), NumFeatures);
218 |     // Inputs.size() is cast to uint32_t below.
219 |     assert(Inputs.size() < std::numeric_limits<uint32_t>::max());
220 |     Inputs.push_back(new InputInfo());
221 |     InputInfo &II = *Inputs.back();
222 |     II.U = U;
223 |     II.NumFeatures = NumFeatures;
224 |     II.NeverReduce = NeverReduce;
```
- **Line 209 / 第 209 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 210 / 第 210 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 211 / 第 211 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 212 / 第 212 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 213 / 第 213 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 214 / 第 214 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 215 / 第 215 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 216 / 第 216 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 217 / 第 217 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 218 / 第 218 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 219 / 第 219 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 220 / 第 220 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 221 / 第 221 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 222 / 第 222 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 223 / 第 223 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 224 / 第 224 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 225-240 / 第 225-240 行
```cpp
225 |     II.TimeOfUnit = TimeOfUnit;
226 |     II.MayDeleteFile = MayDeleteFile;
227 |     II.UniqFeatureSet = FeatureSet;
228 |     II.HasFocusFunction = HasFocusFunction;
229 |     // Assign maximal energy to the new seed.
230 |     II.Energy = RareFeatures.empty() ? 1.0 : log(RareFeatures.size());
231 |     II.SumIncidence = static_cast<double>(RareFeatures.size());
232 |     II.NeedsEnergyUpdate = false;
233 |     std::sort(II.UniqFeatureSet.begin(), II.UniqFeatureSet.end());
234 |     ComputeSHA1(U.data(), U.size(), II.Sha1);
235 |     auto Sha1Str = Sha1ToString(II.Sha1);
236 |     Hashes.insert(Sha1Str);
237 |     if (HasFocusFunction)
238 |       if (auto V = DFT.Get(Sha1Str))
239 |         II.DataFlowTraceForFocusFunction = *V;
240 |     // This is a gross heuristic.
```
- **Line 225 / 第 225 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 226 / 第 226 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 227 / 第 227 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 228 / 第 228 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 229 / 第 229 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 230 / 第 230 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 231 / 第 231 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 232 / 第 232 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 233 / 第 233 行**: EN: Declares function or method `std::sort`. CN: 声明函数或方法 `std::sort`。
- **Line 234 / 第 234 行**: EN: Declares function or method `ComputeSHA1`. CN: 声明函数或方法 `ComputeSHA1`。
- **Line 235 / 第 235 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 236 / 第 236 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 237 / 第 237 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 238 / 第 238 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 239 / 第 239 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 240 / 第 240 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 241-256 / 第 241-256 行
```cpp
241 |     // Ideally, when we add an element to a corpus we need to know its DFT.
242 |     // But if we don't, we'll use the DFT of its base input.
243 |     if (II.DataFlowTraceForFocusFunction.empty() && BaseII)
244 |       II.DataFlowTraceForFocusFunction = BaseII->DataFlowTraceForFocusFunction;
245 |     DistributionNeedsUpdate = true;
246 |     PrintCorpus();
247 |     // ValidateFeatureSet();
248 |     return &II;
249 |   }
250 | 
251 |   // Debug-only
252 |   void PrintUnit(const Unit &U) {
253 |     if (!FeatureDebug) return;
254 |     for (uint8_t C : U) {
255 |       if (C != 'F' && C != 'U' && C != 'Z')
256 |         C = '.';
```
- **Line 241 / 第 241 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 242 / 第 242 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 243 / 第 243 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 244 / 第 244 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 245 / 第 245 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 246 / 第 246 行**: EN: Declares function or method `PrintCorpus`. CN: 声明函数或方法 `PrintCorpus`。
- **Line 247 / 第 247 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 248 / 第 248 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 249 / 第 249 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 250 / 第 250 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 251 / 第 251 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 252 / 第 252 行**: EN: Starts the definition of function or method `PrintUnit`. CN: 开始定义函数或方法 `PrintUnit`。
- **Line 253 / 第 253 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 254 / 第 254 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 255 / 第 255 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 256 / 第 256 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 257-272 / 第 257-272 行
```cpp
257 |       Printf("%c", C);
258 |     }
259 |   }
260 | 
261 |   // Debug-only
262 |   void PrintFeatureSet(const std::vector<uint32_t> &FeatureSet) {
263 |     if (!FeatureDebug) return;
264 |     Printf("{");
265 |     for (uint32_t Feature: FeatureSet)
266 |       Printf("%u,", Feature);
267 |     Printf("}");
268 |   }
269 | 
270 |   // Debug-only
271 |   void PrintCorpus() {
272 |     if (!FeatureDebug) return;
```
- **Line 257 / 第 257 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 258 / 第 258 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 259 / 第 259 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 260 / 第 260 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 261 / 第 261 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 262 / 第 262 行**: EN: Starts the definition of function or method `PrintFeatureSet`. CN: 开始定义函数或方法 `PrintFeatureSet`。
- **Line 263 / 第 263 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 264 / 第 264 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 265 / 第 265 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 266 / 第 266 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 267 / 第 267 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 268 / 第 268 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 269 / 第 269 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 270 / 第 270 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 271 / 第 271 行**: EN: Starts the definition of function or method `PrintCorpus`. CN: 开始定义函数或方法 `PrintCorpus`。
- **Line 272 / 第 272 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 273-288 / 第 273-288 行
```cpp
273 |     Printf("======= CORPUS:\n");
274 |     int i = 0;
275 |     for (auto II : Inputs) {
276 |       if (std::find(II->U.begin(), II->U.end(), 'F') != II->U.end()) {
277 |         Printf("[%2d] ", i);
278 |         Printf("%s sz=%zd ", Sha1ToString(II->Sha1).c_str(), II->U.size());
279 |         PrintUnit(II->U);
280 |         Printf(" ");
281 |         PrintFeatureSet(II->UniqFeatureSet);
282 |         Printf("\n");
283 |       }
284 |       i++;
285 |     }
286 |   }
287 | 
288 |   void Replace(InputInfo *II, const Unit &U,
```
- **Line 273 / 第 273 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 274 / 第 274 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 275 / 第 275 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 276 / 第 276 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 277 / 第 277 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 278 / 第 278 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 279 / 第 279 行**: EN: Declares function or method `PrintUnit`. CN: 声明函数或方法 `PrintUnit`。
- **Line 280 / 第 280 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 281 / 第 281 行**: EN: Declares function or method `PrintFeatureSet`. CN: 声明函数或方法 `PrintFeatureSet`。
- **Line 282 / 第 282 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 283 / 第 283 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 284 / 第 284 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 285 / 第 285 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 286 / 第 286 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 287 / 第 287 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 288 / 第 288 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 289-304 / 第 289-304 行
```cpp
289 |                std::chrono::microseconds TimeOfUnit) {
290 |     assert(II->U.size() > U.size());
291 |     Hashes.erase(Sha1ToString(II->Sha1));
292 |     DeleteFile(*II);
293 |     ComputeSHA1(U.data(), U.size(), II->Sha1);
294 |     Hashes.insert(Sha1ToString(II->Sha1));
295 |     II->U = U;
296 |     II->Reduced = true;
297 |     II->TimeOfUnit = TimeOfUnit;
298 |     DistributionNeedsUpdate = true;
299 |   }
300 | 
301 |   bool HasUnit(const Unit &U) { return Hashes.count(Hash(U)); }
302 |   bool HasUnit(const std::string &H) { return Hashes.count(H); }
303 |   InputInfo &ChooseUnitToMutate(Random &Rand) {
304 |     InputInfo &II = *Inputs[ChooseUnitIdxToMutate(Rand)];
```
- **Line 289 / 第 289 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 290 / 第 290 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 291 / 第 291 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 292 / 第 292 行**: EN: Declares function or method `DeleteFile`. CN: 声明函数或方法 `DeleteFile`。
- **Line 293 / 第 293 行**: EN: Declares function or method `ComputeSHA1`. CN: 声明函数或方法 `ComputeSHA1`。
- **Line 294 / 第 294 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 295 / 第 295 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 296 / 第 296 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 297 / 第 297 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 298 / 第 298 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 299 / 第 299 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 300 / 第 300 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 301 / 第 301 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 302 / 第 302 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 303 / 第 303 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 304 / 第 304 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 305-320 / 第 305-320 行
```cpp
305 |     assert(!II.U.empty());
306 |     return II;
307 |   }
308 | 
309 |   InputInfo &ChooseUnitToCrossOverWith(Random &Rand, bool UniformDist) {
310 |     if (!UniformDist) {
311 |       return ChooseUnitToMutate(Rand);
312 |     }
313 |     InputInfo &II = *Inputs[Rand(Inputs.size())];
314 |     assert(!II.U.empty());
315 |     return II;
316 |   }
317 | 
318 |   // Returns an index of random unit from the corpus to mutate.
319 |   size_t ChooseUnitIdxToMutate(Random &Rand) {
320 |     UpdateCorpusDistribution(Rand);
```
- **Line 305 / 第 305 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 306 / 第 306 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 307 / 第 307 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 308 / 第 308 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 309 / 第 309 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 310 / 第 310 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 311 / 第 311 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 312 / 第 312 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 313 / 第 313 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 314 / 第 314 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 315 / 第 315 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 316 / 第 316 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 317 / 第 317 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 318 / 第 318 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 319 / 第 319 行**: EN: Starts the definition of function or method `ChooseUnitIdxToMutate`. CN: 开始定义函数或方法 `ChooseUnitIdxToMutate`。
- **Line 320 / 第 320 行**: EN: Declares function or method `UpdateCorpusDistribution`. CN: 声明函数或方法 `UpdateCorpusDistribution`。

### Lines 321-336 / 第 321-336 行
```cpp
321 |     size_t Idx = static_cast<size_t>(CorpusDistribution(Rand));
322 |     assert(Idx < Inputs.size());
323 |     return Idx;
324 |   }
325 | 
326 |   void PrintStats() {
327 |     for (size_t i = 0; i < Inputs.size(); i++) {
328 |       const auto &II = *Inputs[i];
329 |       Printf("  [% 3zd %s] sz: % 5zd runs: % 5zd succ: % 5zd focus: %d\n", i,
330 |              Sha1ToString(II.Sha1).c_str(), II.U.size(),
331 |              II.NumExecutedMutations, II.NumSuccessfulMutations,
332 |              II.HasFocusFunction);
333 |     }
334 |   }
335 | 
336 |   void PrintFeatureSet() {
```
- **Line 321 / 第 321 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 322 / 第 322 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 323 / 第 323 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 324 / 第 324 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 325 / 第 325 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 326 / 第 326 行**: EN: Starts the definition of function or method `PrintStats`. CN: 开始定义函数或方法 `PrintStats`。
- **Line 327 / 第 327 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 328 / 第 328 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 329 / 第 329 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 330 / 第 330 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 331 / 第 331 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 332 / 第 332 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 333 / 第 333 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 334 / 第 334 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 335 / 第 335 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 336 / 第 336 行**: EN: Starts the definition of function or method `PrintFeatureSet`. CN: 开始定义函数或方法 `PrintFeatureSet`。

### Lines 337-352 / 第 337-352 行
```cpp
337 |     for (size_t i = 0; i < kFeatureSetSize; i++) {
338 |       if(size_t Sz = GetFeature(i))
339 |         Printf("[%zd: id %zd sz%zd] ", i, (size_t)SmallestElementPerFeature[i],
340 |                Sz);
341 |     }
342 |     Printf("\n\t");
343 |     for (size_t i = 0; i < Inputs.size(); i++)
344 |       if (size_t N = Inputs[i]->NumFeatures)
345 |         Printf(" %zd=>%zd ", i, N);
346 |     Printf("\n");
347 |   }
348 | 
349 |   void DeleteFile(const InputInfo &II) {
350 |     if (!OutputCorpus.empty() && II.MayDeleteFile)
351 |       RemoveFile(DirPlusFile(OutputCorpus, Sha1ToString(II.Sha1)));
352 |   }
```
- **Line 337 / 第 337 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 338 / 第 338 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 339 / 第 339 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 340 / 第 340 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 341 / 第 341 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 342 / 第 342 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 343 / 第 343 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 344 / 第 344 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 345 / 第 345 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 346 / 第 346 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 347 / 第 347 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 348 / 第 348 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 349 / 第 349 行**: EN: Starts the definition of function or method `DeleteFile`. CN: 开始定义函数或方法 `DeleteFile`。
- **Line 350 / 第 350 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 351 / 第 351 行**: EN: Declares function or method `RemoveFile`. CN: 声明函数或方法 `RemoveFile`。
- **Line 352 / 第 352 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 353-368 / 第 353-368 行
```cpp
353 | 
354 |   void DeleteInput(size_t Idx) {
355 |     InputInfo &II = *Inputs[Idx];
356 |     DeleteFile(II);
357 |     Unit().swap(II.U);
358 |     II.Energy = 0.0;
359 |     II.NeedsEnergyUpdate = false;
360 |     DistributionNeedsUpdate = true;
361 |     if (FeatureDebug)
362 |       Printf("EVICTED %zd\n", Idx);
363 |   }
364 | 
365 |   void AddRareFeature(uint32_t Idx) {
366 |     // Maintain *at least* TopXRarestFeatures many rare features
367 |     // and all features with a frequency below ConsideredRare.
368 |     // Remove all other features.
```
- **Line 353 / 第 353 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 354 / 第 354 行**: EN: Starts the definition of function or method `DeleteInput`. CN: 开始定义函数或方法 `DeleteInput`。
- **Line 355 / 第 355 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 356 / 第 356 行**: EN: Declares function or method `DeleteFile`. CN: 声明函数或方法 `DeleteFile`。
- **Line 357 / 第 357 行**: EN: Declares function or method `Unit`. CN: 声明函数或方法 `Unit`。
- **Line 358 / 第 358 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 359 / 第 359 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 360 / 第 360 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 361 / 第 361 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 362 / 第 362 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 363 / 第 363 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 364 / 第 364 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 365 / 第 365 行**: EN: Starts the definition of function or method `AddRareFeature`. CN: 开始定义函数或方法 `AddRareFeature`。
- **Line 366 / 第 366 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 367 / 第 367 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 368 / 第 368 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 369-384 / 第 369-384 行
```cpp
369 |     while (RareFeatures.size() > Entropic.NumberOfRarestFeatures &&
370 |            FreqOfMostAbundantRareFeature > Entropic.FeatureFrequencyThreshold) {
371 | 
372 |       // Find most and second most abbundant feature.
373 |       uint32_t MostAbundantRareFeatureIndices[2] = {RareFeatures[0],
374 |                                                     RareFeatures[0]};
375 |       size_t Delete = 0;
376 |       for (size_t i = 0; i < RareFeatures.size(); i++) {
377 |         uint32_t Idx2 = RareFeatures[i];
378 |         if (GlobalFeatureFreqs[Idx2] >=
379 |             GlobalFeatureFreqs[MostAbundantRareFeatureIndices[0]]) {
380 |           MostAbundantRareFeatureIndices[1] = MostAbundantRareFeatureIndices[0];
381 |           MostAbundantRareFeatureIndices[0] = Idx2;
382 |           Delete = i;
383 |         }
384 |       }
```
- **Line 369 / 第 369 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 370 / 第 370 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 371 / 第 371 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 372 / 第 372 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 373 / 第 373 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 374 / 第 374 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 375 / 第 375 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 376 / 第 376 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 377 / 第 377 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 378 / 第 378 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 379 / 第 379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 380 / 第 380 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 381 / 第 381 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 382 / 第 382 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 383 / 第 383 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 384 / 第 384 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 385-400 / 第 385-400 行
```cpp
385 | 
386 |       // Remove most abundant rare feature.
387 |       IsRareFeature[Delete] = false;
388 |       RareFeatures[Delete] = RareFeatures.back();
389 |       RareFeatures.pop_back();
390 | 
391 |       for (auto II : Inputs) {
392 |         if (II->DeleteFeatureFreq(MostAbundantRareFeatureIndices[0]))
393 |           II->NeedsEnergyUpdate = true;
394 |       }
395 | 
396 |       // Set 2nd most abundant as the new most abundant feature count.
397 |       FreqOfMostAbundantRareFeature =
398 |           GlobalFeatureFreqs[MostAbundantRareFeatureIndices[1]];
399 |     }
400 | 
```
- **Line 385 / 第 385 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 386 / 第 386 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 387 / 第 387 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 388 / 第 388 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 389 / 第 389 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 390 / 第 390 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 391 / 第 391 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 392 / 第 392 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 393 / 第 393 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 394 / 第 394 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 395 / 第 395 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 396 / 第 396 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 397 / 第 397 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 398 / 第 398 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 399 / 第 399 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 400 / 第 400 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 401-416 / 第 401-416 行
```cpp
401 |     // Add rare feature, handle collisions, and update energy.
402 |     RareFeatures.push_back(Idx);
403 |     IsRareFeature[Idx] = true;
404 |     GlobalFeatureFreqs[Idx] = 0;
405 |     for (auto II : Inputs) {
406 |       II->DeleteFeatureFreq(Idx);
407 | 
408 |       // Apply add-one smoothing to this locally undiscovered feature.
409 |       // Zero energy seeds will never be fuzzed and remain zero energy.
410 |       if (II->Energy > 0.0) {
411 |         II->SumIncidence += 1;
412 |         II->Energy += log(II->SumIncidence) / II->SumIncidence;
413 |       }
414 |     }
415 | 
416 |     DistributionNeedsUpdate = true;
```
- **Line 401 / 第 401 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 402 / 第 402 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 403 / 第 403 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 404 / 第 404 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 405 / 第 405 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 406 / 第 406 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 407 / 第 407 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 408 / 第 408 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 409 / 第 409 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 410 / 第 410 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 411 / 第 411 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 412 / 第 412 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 413 / 第 413 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 414 / 第 414 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 415 / 第 415 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 416 / 第 416 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 417-432 / 第 417-432 行
```cpp
417 |   }
418 | 
419 |   bool AddFeature(size_t Idx, uint32_t NewSize, bool Shrink) {
420 |     assert(NewSize);
421 |     Idx = Idx % kFeatureSetSize;
422 |     uint32_t OldSize = GetFeature(Idx);
423 |     if (OldSize == 0 || (Shrink && OldSize > NewSize)) {
424 |       if (OldSize > 0) {
425 |         size_t OldIdx = SmallestElementPerFeature[Idx];
426 |         InputInfo &II = *Inputs[OldIdx];
427 |         assert(II.NumFeatures > 0);
428 |         II.NumFeatures--;
429 |         if (II.NumFeatures == 0)
430 |           DeleteInput(OldIdx);
431 |       } else {
432 |         NumAddedFeatures++;
```
- **Line 417 / 第 417 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 418 / 第 418 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 419 / 第 419 行**: EN: Starts the definition of function or method `AddFeature`. CN: 开始定义函数或方法 `AddFeature`。
- **Line 420 / 第 420 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 421 / 第 421 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 422 / 第 422 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 423 / 第 423 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 424 / 第 424 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 425 / 第 425 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 426 / 第 426 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 427 / 第 427 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 428 / 第 428 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 429 / 第 429 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 430 / 第 430 行**: EN: Declares function or method `DeleteInput`. CN: 声明函数或方法 `DeleteInput`。
- **Line 431 / 第 431 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 432 / 第 432 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 433-448 / 第 433-448 行
```cpp
433 |         if (Entropic.Enabled)
434 |           AddRareFeature((uint32_t)Idx);
435 |       }
436 |       NumUpdatedFeatures++;
437 |       if (FeatureDebug)
438 |         Printf("ADD FEATURE %zd sz %d\n", Idx, NewSize);
439 |       // Inputs.size() is guaranteed to be less than UINT32_MAX by AddToCorpus.
440 |       SmallestElementPerFeature[Idx] = static_cast<uint32_t>(Inputs.size());
441 |       InputSizesPerFeature[Idx] = NewSize;
442 |       return true;
443 |     }
444 |     return false;
445 |   }
446 | 
447 |   // Increment frequency of feature Idx globally and locally.
448 |   void UpdateFeatureFrequency(InputInfo *II, size_t Idx) {
```
- **Line 433 / 第 433 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 434 / 第 434 行**: EN: Declares function or method `AddRareFeature`. CN: 声明函数或方法 `AddRareFeature`。
- **Line 435 / 第 435 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 436 / 第 436 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 437 / 第 437 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 438 / 第 438 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 439 / 第 439 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 440 / 第 440 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 441 / 第 441 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 442 / 第 442 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 443 / 第 443 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 444 / 第 444 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 445 / 第 445 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 446 / 第 446 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 447 / 第 447 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 448 / 第 448 行**: EN: Starts the definition of function or method `UpdateFeatureFrequency`. CN: 开始定义函数或方法 `UpdateFeatureFrequency`。

### Lines 449-464 / 第 449-464 行
```cpp
449 |     uint32_t Idx32 = Idx % kFeatureSetSize;
450 | 
451 |     // Saturated increment.
452 |     if (GlobalFeatureFreqs[Idx32] == 0xFFFF)
453 |       return;
454 |     uint16_t Freq = GlobalFeatureFreqs[Idx32]++;
455 | 
456 |     // Skip if abundant.
457 |     if (Freq > FreqOfMostAbundantRareFeature || !IsRareFeature[Idx32])
458 |       return;
459 | 
460 |     // Update global frequencies.
461 |     if (Freq == FreqOfMostAbundantRareFeature)
462 |       FreqOfMostAbundantRareFeature++;
463 | 
464 |     // Update local frequencies.
```
- **Line 449 / 第 449 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 450 / 第 450 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 451 / 第 451 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 452 / 第 452 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 453 / 第 453 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 454 / 第 454 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 455 / 第 455 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 456 / 第 456 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 457 / 第 457 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 458 / 第 458 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 459 / 第 459 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 460 / 第 460 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 461 / 第 461 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 462 / 第 462 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 463 / 第 463 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 464 / 第 464 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 465-480 / 第 465-480 行
```cpp
465 |     if (II)
466 |       II->UpdateFeatureFrequency(Idx32);
467 |   }
468 | 
469 |   size_t NumFeatures() const { return NumAddedFeatures; }
470 |   size_t NumFeatureUpdates() const { return NumUpdatedFeatures; }
471 | 
472 | private:
473 | 
474 |   static const bool FeatureDebug = false;
475 | 
476 |   uint32_t GetFeature(size_t Idx) const { return InputSizesPerFeature[Idx]; }
477 | 
478 |   void ValidateFeatureSet() {
479 |     if (FeatureDebug)
480 |       PrintFeatureSet();
```
- **Line 465 / 第 465 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 466 / 第 466 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 467 / 第 467 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 468 / 第 468 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 469 / 第 469 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 470 / 第 470 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 471 / 第 471 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 472 / 第 472 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 473 / 第 473 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 474 / 第 474 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 475 / 第 475 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 476 / 第 476 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 477 / 第 477 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 478 / 第 478 行**: EN: Starts the definition of function or method `ValidateFeatureSet`. CN: 开始定义函数或方法 `ValidateFeatureSet`。
- **Line 479 / 第 479 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 480 / 第 480 行**: EN: Declares function or method `PrintFeatureSet`. CN: 声明函数或方法 `PrintFeatureSet`。

### Lines 481-496 / 第 481-496 行
```cpp
481 |     for (size_t Idx = 0; Idx < kFeatureSetSize; Idx++)
482 |       if (GetFeature(Idx))
483 |         Inputs[SmallestElementPerFeature[Idx]]->Tmp++;
484 |     for (auto II: Inputs) {
485 |       if (II->Tmp != II->NumFeatures)
486 |         Printf("ZZZ %zd %zd\n", II->Tmp, II->NumFeatures);
487 |       assert(II->Tmp == II->NumFeatures);
488 |       II->Tmp = 0;
489 |     }
490 |   }
491 | 
492 |   // Updates the probability distribution for the units in the corpus.
493 |   // Must be called whenever the corpus or unit weights are changed.
494 |   //
495 |   // Hypothesis: inputs that maximize information about globally rare features
496 |   // are interesting.
```
- **Line 481 / 第 481 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 482 / 第 482 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 483 / 第 483 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 484 / 第 484 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 485 / 第 485 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 486 / 第 486 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 487 / 第 487 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 488 / 第 488 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 489 / 第 489 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 490 / 第 490 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 491 / 第 491 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 492 / 第 492 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 493 / 第 493 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 494 / 第 494 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 495 / 第 495 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 496 / 第 496 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 497-512 / 第 497-512 行
```cpp
497 |   void UpdateCorpusDistribution(Random &Rand) {
498 |     // Skip update if no seeds or rare features were added/deleted.
499 |     // Sparse updates for local change of feature frequencies,
500 |     // i.e., randomly do not skip.
501 |     if (!DistributionNeedsUpdate &&
502 |         (!Entropic.Enabled || Rand(kSparseEnergyUpdates)))
503 |       return;
504 | 
505 |     DistributionNeedsUpdate = false;
506 | 
507 |     size_t N = Inputs.size();
508 |     assert(N);
509 |     Intervals.resize(N + 1);
510 |     Weights.resize(N);
511 |     std::iota(Intervals.begin(), Intervals.end(), 0);
512 | 
```
- **Line 497 / 第 497 行**: EN: Starts the definition of function or method `UpdateCorpusDistribution`. CN: 开始定义函数或方法 `UpdateCorpusDistribution`。
- **Line 498 / 第 498 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 499 / 第 499 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 500 / 第 500 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 501 / 第 501 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 502 / 第 502 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 503 / 第 503 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 504 / 第 504 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 505 / 第 505 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 506 / 第 506 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 507 / 第 507 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 508 / 第 508 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 509 / 第 509 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 510 / 第 510 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 511 / 第 511 行**: EN: Declares function or method `std::iota`. CN: 声明函数或方法 `std::iota`。
- **Line 512 / 第 512 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 513-528 / 第 513-528 行
```cpp
513 |     std::chrono::microseconds AverageUnitExecutionTime(0);
514 |     for (auto II : Inputs) {
515 |       AverageUnitExecutionTime += II->TimeOfUnit;
516 |     }
517 |     AverageUnitExecutionTime /= N;
518 | 
519 |     bool VanillaSchedule = true;
520 |     if (Entropic.Enabled) {
521 |       for (auto II : Inputs) {
522 |         if (II->NeedsEnergyUpdate && II->Energy != 0.0) {
523 |           II->NeedsEnergyUpdate = false;
524 |           II->UpdateEnergy(RareFeatures.size(), Entropic.ScalePerExecTime,
525 |                            AverageUnitExecutionTime);
526 |         }
527 |       }
528 | 
```
- **Line 513 / 第 513 行**: EN: Declares function or method `AverageUnitExecutionTime`. CN: 声明函数或方法 `AverageUnitExecutionTime`。
- **Line 514 / 第 514 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 515 / 第 515 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 516 / 第 516 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 517 / 第 517 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 518 / 第 518 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 519 / 第 519 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 520 / 第 520 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 521 / 第 521 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 522 / 第 522 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 523 / 第 523 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 524 / 第 524 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 525 / 第 525 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 526 / 第 526 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 527 / 第 527 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 528 / 第 528 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 529-544 / 第 529-544 行
```cpp
529 |       for (size_t i = 0; i < N; i++) {
530 | 
531 |         if (Inputs[i]->NumFeatures == 0) {
532 |           // If the seed doesn't represent any features, assign zero energy.
533 |           Weights[i] = 0.;
534 |         } else if (Inputs[i]->NumExecutedMutations / kMaxMutationFactor >
535 |                    NumExecutedMutations / Inputs.size()) {
536 |           // If the seed was fuzzed a lot more than average, assign zero energy.
537 |           Weights[i] = 0.;
538 |         } else {
539 |           // Otherwise, simply assign the computed energy.
540 |           Weights[i] = Inputs[i]->Energy;
541 |         }
542 | 
543 |         // If energy for all seeds is zero, fall back to vanilla schedule.
544 |         if (Weights[i] > 0.0)
```
- **Line 529 / 第 529 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 530 / 第 530 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 531 / 第 531 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 532 / 第 532 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 533 / 第 533 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 534 / 第 534 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 535 / 第 535 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 536 / 第 536 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 537 / 第 537 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 538 / 第 538 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 539 / 第 539 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 540 / 第 540 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 541 / 第 541 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 542 / 第 542 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 543 / 第 543 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 544 / 第 544 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 545-560 / 第 545-560 行
```cpp
545 |           VanillaSchedule = false;
546 |       }
547 |     }
548 | 
549 |     if (VanillaSchedule) {
550 |       for (size_t i = 0; i < N; i++)
551 |         Weights[i] =
552 |             Inputs[i]->NumFeatures
553 |                 ? static_cast<double>((i + 1) *
554 |                                       (Inputs[i]->HasFocusFunction ? 1000 : 1))
555 |                 : 0.;
556 |     }
557 | 
558 |     if (FeatureDebug) {
559 |       for (size_t i = 0; i < N; i++)
560 |         Printf("%zd ", Inputs[i]->NumFeatures);
```
- **Line 545 / 第 545 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 546 / 第 546 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 547 / 第 547 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 548 / 第 548 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 549 / 第 549 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 550 / 第 550 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 551 / 第 551 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 552 / 第 552 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 553 / 第 553 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 554 / 第 554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 555 / 第 555 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 556 / 第 556 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 557 / 第 557 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 558 / 第 558 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 559 / 第 559 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 560 / 第 560 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。

### Lines 561-576 / 第 561-576 行
```cpp
561 |       Printf("SCORE\n");
562 |       for (size_t i = 0; i < N; i++)
563 |         Printf("%f ", Weights[i]);
564 |       Printf("Weights\n");
565 |     }
566 |     CorpusDistribution = std::piecewise_constant_distribution<double>(
567 |         Intervals.begin(), Intervals.end(), Weights.begin());
568 |   }
569 |   std::piecewise_constant_distribution<double> CorpusDistribution;
570 | 
571 |   std::vector<double> Intervals;
572 |   std::vector<double> Weights;
573 | 
574 |   std::unordered_set<std::string> Hashes;
575 |   std::vector<InputInfo *> Inputs;
576 | 
```
- **Line 561 / 第 561 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 562 / 第 562 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 563 / 第 563 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 564 / 第 564 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 565 / 第 565 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 566 / 第 566 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 567 / 第 567 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 568 / 第 568 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 569 / 第 569 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 570 / 第 570 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 571 / 第 571 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 572 / 第 572 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 573 / 第 573 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 574 / 第 574 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 575 / 第 575 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 576 / 第 576 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 577-592 / 第 577-592 行
```cpp
577 |   size_t NumAddedFeatures = 0;
578 |   size_t NumUpdatedFeatures = 0;
579 |   uint32_t InputSizesPerFeature[kFeatureSetSize];
580 |   uint32_t SmallestElementPerFeature[kFeatureSetSize];
581 | 
582 |   bool DistributionNeedsUpdate = true;
583 |   uint16_t FreqOfMostAbundantRareFeature = 0;
584 |   uint16_t GlobalFeatureFreqs[kFeatureSetSize] = {};
585 |   std::vector<uint32_t> RareFeatures;
586 |   std::bitset<kFeatureSetSize> IsRareFeature;
587 | 
588 |   std::string OutputCorpus;
589 | };
590 | 
591 | }  // namespace fuzzer
592 | 
```
- **Line 577 / 第 577 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 578 / 第 578 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 579 / 第 579 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 580 / 第 580 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 581 / 第 581 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 582 / 第 582 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 583 / 第 583 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 584 / 第 584 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 585 / 第 585 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 586 / 第 586 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 587 / 第 587 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 588 / 第 588 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 589 / 第 589 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 590 / 第 590 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 591 / 第 591 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 592 / 第 592 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 593-593 / 第 593-593 行
```cpp
593 | #endif  // LLVM_FUZZER_CORPUS
```
- **Line 593 / 第 593 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

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

- `FuzzerDataFlowTrace.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerDefs.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerIO.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerRandom.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerSHA1.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerTracePC.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `algorithm` — System or standard library dependency / 系统或标准库依赖
- `bitset` — System or standard library dependency / 系统或标准库依赖
- `chrono` — System or standard library dependency / 系统或标准库依赖
- `numeric` — System or standard library dependency / 系统或标准库依赖
- `random` — System or standard library dependency / 系统或标准库依赖
- `unordered_set` — System or standard library dependency / 系统或标准库依赖
