# FuzzerMutate.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/fuzzer/FuzzerMutate.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Mutate a test input.
  - **CN**: 实现 libFuzzer 中与 `FuzzerMutate` 相关的组件或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
```cpp
 1 | //===- FuzzerMutate.cpp - Mutate a test input -----------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | // Mutate a test input.
 9 | //===----------------------------------------------------------------------===//
10 | 
11 | #include "FuzzerDefs.h"
12 | #include "FuzzerExtFunctions.h"
13 | #include "FuzzerIO.h"
14 | #include "FuzzerMutate.h"
15 | #include "FuzzerOptions.h"
16 | #include "FuzzerTracePC.h"
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
- **Line 11 / 第 11 行**: EN: Includes `FuzzerDefs.h` so this file can use its declarations. CN: 包含 `FuzzerDefs.h`，以便当前文件使用其中的声明。
- **Line 12 / 第 12 行**: EN: Includes `FuzzerExtFunctions.h` so this file can use its declarations. CN: 包含 `FuzzerExtFunctions.h`，以便当前文件使用其中的声明。
- **Line 13 / 第 13 行**: EN: Includes `FuzzerIO.h` so this file can use its declarations. CN: 包含 `FuzzerIO.h`，以便当前文件使用其中的声明。
- **Line 14 / 第 14 行**: EN: Includes `FuzzerMutate.h` so this file can use its declarations. CN: 包含 `FuzzerMutate.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Includes `FuzzerOptions.h` so this file can use its declarations. CN: 包含 `FuzzerOptions.h`，以便当前文件使用其中的声明。
- **Line 16 / 第 16 行**: EN: Includes `FuzzerTracePC.h` so this file can use its declarations. CN: 包含 `FuzzerTracePC.h`，以便当前文件使用其中的声明。

### Lines 17-32 / 第 17-32 行
```cpp
17 | 
18 | namespace fuzzer {
19 | 
20 | const size_t Dictionary::kMaxDictSize;
21 | static const size_t kMaxMutationsToPrint = 10;
22 | 
23 | static void PrintASCII(const Word &W, const char *PrintAfter) {
24 |   PrintASCII(W.data(), W.size(), PrintAfter);
25 | }
26 | 
27 | MutationDispatcher::MutationDispatcher(Random &Rand,
28 |                                        const FuzzingOptions &Options)
29 |     : Rand(Rand), Options(Options) {
30 |   DefaultMutators.insert(
31 |       DefaultMutators.begin(),
32 |       {
```
- **Line 17 / 第 17 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 18 / 第 18 行**: EN: Opens namespace `fuzzer` to scope related declarations. CN: 打开命名空间 `fuzzer`，为相关声明建立作用域。
- **Line 19 / 第 19 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 20 / 第 20 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 21 / 第 21 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 22 / 第 22 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 23 / 第 23 行**: EN: Starts the definition of function or method `PrintASCII`. CN: 开始定义函数或方法 `PrintASCII`。
- **Line 24 / 第 24 行**: EN: Declares function or method `PrintASCII`. CN: 声明函数或方法 `PrintASCII`。
- **Line 25 / 第 25 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 26 / 第 26 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 27 / 第 27 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 28 / 第 28 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 29 / 第 29 行**: EN: Starts the definition of function or method `Rand`. CN: 开始定义函数或方法 `Rand`。
- **Line 30 / 第 30 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 31 / 第 31 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 32 / 第 32 行**: EN: Opens a new scope or block. CN: 打开新的作用域或代码块。

### Lines 33-48 / 第 33-48 行
```cpp
33 |           {&MutationDispatcher::Mutate_EraseBytes, "EraseBytes"},
34 |           {&MutationDispatcher::Mutate_InsertByte, "InsertByte"},
35 |           {&MutationDispatcher::Mutate_InsertRepeatedBytes,
36 |            "InsertRepeatedBytes"},
37 |           {&MutationDispatcher::Mutate_ChangeByte, "ChangeByte"},
38 |           {&MutationDispatcher::Mutate_ChangeBit, "ChangeBit"},
39 |           {&MutationDispatcher::Mutate_ShuffleBytes, "ShuffleBytes"},
40 |           {&MutationDispatcher::Mutate_ChangeASCIIInteger, "ChangeASCIIInt"},
41 |           {&MutationDispatcher::Mutate_ChangeBinaryInteger, "ChangeBinInt"},
42 |           {&MutationDispatcher::Mutate_CopyPart, "CopyPart"},
43 |           {&MutationDispatcher::Mutate_CrossOver, "CrossOver"},
44 |           {&MutationDispatcher::Mutate_AddWordFromManualDictionary,
45 |            "ManualDict"},
46 |           {&MutationDispatcher::Mutate_AddWordFromPersistentAutoDictionary,
47 |            "PersAutoDict"},
48 |       });
```
- **Line 33 / 第 33 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 34 / 第 34 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 35 / 第 35 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 36 / 第 36 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 37 / 第 37 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 38 / 第 38 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 39 / 第 39 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 40 / 第 40 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 41 / 第 41 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 42 / 第 42 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 43 / 第 43 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 44 / 第 44 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 45 / 第 45 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 46 / 第 46 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 47 / 第 47 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 48 / 第 48 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 49-64 / 第 49-64 行
```cpp
49 |   if(Options.UseCmp)
50 |     DefaultMutators.push_back(
51 |         {&MutationDispatcher::Mutate_AddWordFromTORC, "CMP"});
52 | 
53 |   if (EF->LLVMFuzzerCustomMutator)
54 |     Mutators.push_back({&MutationDispatcher::Mutate_Custom, "Custom"});
55 |   else
56 |     Mutators = DefaultMutators;
57 | 
58 |   if (EF->LLVMFuzzerCustomCrossOver)
59 |     Mutators.push_back(
60 |         {&MutationDispatcher::Mutate_CustomCrossOver, "CustomCrossOver"});
61 | }
62 | 
63 | static char RandCh(Random &Rand) {
64 |   if (Rand.RandBool())
```
- **Line 49 / 第 49 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 50 / 第 50 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 51 / 第 51 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 52 / 第 52 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 53 / 第 53 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 54 / 第 54 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 55 / 第 55 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 56 / 第 56 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 57 / 第 57 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 58 / 第 58 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 59 / 第 59 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 60 / 第 60 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 61 / 第 61 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 62 / 第 62 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 63 / 第 63 行**: EN: Starts the definition of function or method `RandCh`. CN: 开始定义函数或方法 `RandCh`。
- **Line 64 / 第 64 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 65-80 / 第 65-80 行
```cpp
65 |     return static_cast<char>(Rand(256));
66 |   const char Special[] = "!*'();:@&=+$,/?%#[]012Az-`~.\xff\x00";
67 |   return Special[Rand(sizeof(Special) - 1)];
68 | }
69 | 
70 | size_t MutationDispatcher::Mutate_Custom(uint8_t *Data, size_t Size,
71 |                                          size_t MaxSize) {
72 |   if (EF->__msan_unpoison)
73 |     EF->__msan_unpoison(Data, Size);
74 |   if (EF->__msan_unpoison_param)
75 |     EF->__msan_unpoison_param(4);
76 |   return EF->LLVMFuzzerCustomMutator(Data, Size, MaxSize,
77 |                                      Rand.Rand<unsigned int>());
78 | }
79 | 
80 | size_t MutationDispatcher::Mutate_CustomCrossOver(uint8_t *Data, size_t Size,
```
- **Line 65 / 第 65 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 66 / 第 66 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 67 / 第 67 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 68 / 第 68 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 69 / 第 69 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 70 / 第 70 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 71 / 第 71 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 72 / 第 72 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 73 / 第 73 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 74 / 第 74 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 75 / 第 75 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 76 / 第 76 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 77 / 第 77 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 78 / 第 78 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 79 / 第 79 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 80 / 第 80 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 81-96 / 第 81-96 行
```cpp
81 |                                                   size_t MaxSize) {
82 |   if (Size == 0)
83 |     return 0;
84 |   if (!CrossOverWith) return 0;
85 |   const Unit &Other = *CrossOverWith;
86 |   if (Other.empty())
87 |     return 0;
88 |   CustomCrossOverInPlaceHere.resize(MaxSize);
89 |   auto &U = CustomCrossOverInPlaceHere;
90 | 
91 |   if (EF->__msan_unpoison) {
92 |     EF->__msan_unpoison(Data, Size);
93 |     EF->__msan_unpoison(Other.data(), Other.size());
94 |     EF->__msan_unpoison(U.data(), U.size());
95 |   }
96 |   if (EF->__msan_unpoison_param)
```
- **Line 81 / 第 81 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 82 / 第 82 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 83 / 第 83 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 84 / 第 84 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 85 / 第 85 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 86 / 第 86 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 87 / 第 87 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 88 / 第 88 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 89 / 第 89 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 90 / 第 90 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 91 / 第 91 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 92 / 第 92 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 93 / 第 93 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 94 / 第 94 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 95 / 第 95 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 96 / 第 96 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 97-112 / 第 97-112 行
```cpp
 97 |     EF->__msan_unpoison_param(7);
 98 |   size_t NewSize = EF->LLVMFuzzerCustomCrossOver(
 99 |       Data, Size, Other.data(), Other.size(), U.data(), U.size(),
100 |       Rand.Rand<unsigned int>());
101 | 
102 |   if (!NewSize)
103 |     return 0;
104 |   assert(NewSize <= MaxSize && "CustomCrossOver returned oversized unit");
105 |   memcpy(Data, U.data(), NewSize);
106 |   return NewSize;
107 | }
108 | 
109 | size_t MutationDispatcher::Mutate_ShuffleBytes(uint8_t *Data, size_t Size,
110 |                                                size_t MaxSize) {
111 |   if (Size > MaxSize || Size == 0) return 0;
112 |   size_t ShuffleAmount =
```
- **Line 97 / 第 97 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 98 / 第 98 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 99 / 第 99 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 100 / 第 100 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 101 / 第 101 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 102 / 第 102 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 103 / 第 103 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 104 / 第 104 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 105 / 第 105 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 106 / 第 106 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 107 / 第 107 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 108 / 第 108 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 109 / 第 109 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 110 / 第 110 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 111 / 第 111 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 112 / 第 112 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 113-128 / 第 113-128 行
```cpp
113 |       Rand(std::min(Size, (size_t)8)) + 1; // [1,8] and <= Size.
114 |   size_t ShuffleStart = Rand(Size - ShuffleAmount);
115 |   assert(ShuffleStart + ShuffleAmount <= Size);
116 |   std::shuffle(Data + ShuffleStart, Data + ShuffleStart + ShuffleAmount, Rand);
117 |   return Size;
118 | }
119 | 
120 | size_t MutationDispatcher::Mutate_EraseBytes(uint8_t *Data, size_t Size,
121 |                                              size_t MaxSize) {
122 |   if (Size <= 1) return 0;
123 |   size_t N = Rand(Size / 2) + 1;
124 |   assert(N < Size);
125 |   size_t Idx = Rand(Size - N + 1);
126 |   // Erase Data[Idx:Idx+N].
127 |   memmove(Data + Idx, Data + Idx + N, Size - Idx - N);
128 |   // Printf("Erase: %zd %zd => %zd; Idx %zd\n", N, Size, Size - N, Idx);
```
- **Line 113 / 第 113 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 114 / 第 114 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 115 / 第 115 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 116 / 第 116 行**: EN: Declares function or method `std::shuffle`. CN: 声明函数或方法 `std::shuffle`。
- **Line 117 / 第 117 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 118 / 第 118 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 119 / 第 119 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 120 / 第 120 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 121 / 第 121 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 122 / 第 122 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 123 / 第 123 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 124 / 第 124 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 125 / 第 125 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 126 / 第 126 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 127 / 第 127 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 128 / 第 128 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 129-144 / 第 129-144 行
```cpp
129 |   return Size - N;
130 | }
131 | 
132 | size_t MutationDispatcher::Mutate_InsertByte(uint8_t *Data, size_t Size,
133 |                                              size_t MaxSize) {
134 |   if (Size >= MaxSize) return 0;
135 |   size_t Idx = Rand(Size + 1);
136 |   // Insert new value at Data[Idx].
137 |   memmove(Data + Idx + 1, Data + Idx, Size - Idx);
138 |   Data[Idx] = RandCh(Rand);
139 |   return Size + 1;
140 | }
141 | 
142 | size_t MutationDispatcher::Mutate_InsertRepeatedBytes(uint8_t *Data,
143 |                                                       size_t Size,
144 |                                                       size_t MaxSize) {
```
- **Line 129 / 第 129 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 130 / 第 130 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 131 / 第 131 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 132 / 第 132 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 133 / 第 133 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 134 / 第 134 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 135 / 第 135 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 136 / 第 136 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 137 / 第 137 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 138 / 第 138 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 139 / 第 139 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 140 / 第 140 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 141 / 第 141 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 142 / 第 142 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 143 / 第 143 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 144 / 第 144 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 145-160 / 第 145-160 行
```cpp
145 |   const size_t kMinBytesToInsert = 3;
146 |   if (Size + kMinBytesToInsert >= MaxSize) return 0;
147 |   size_t MaxBytesToInsert = std::min(MaxSize - Size, (size_t)128);
148 |   size_t N = Rand(MaxBytesToInsert - kMinBytesToInsert + 1) + kMinBytesToInsert;
149 |   assert(Size + N <= MaxSize && N);
150 |   size_t Idx = Rand(Size + 1);
151 |   // Insert new values at Data[Idx].
152 |   memmove(Data + Idx + N, Data + Idx, Size - Idx);
153 |   // Give preference to 0x00 and 0xff.
154 |   uint8_t Byte = static_cast<uint8_t>(
155 |       Rand.RandBool() ? Rand(256) : (Rand.RandBool() ? 0 : 255));
156 |   for (size_t i = 0; i < N; i++)
157 |     Data[Idx + i] = Byte;
158 |   return Size + N;
159 | }
160 | 
```
- **Line 145 / 第 145 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 146 / 第 146 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 147 / 第 147 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 148 / 第 148 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 149 / 第 149 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 150 / 第 150 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 151 / 第 151 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 152 / 第 152 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 153 / 第 153 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 154 / 第 154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 155 / 第 155 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 156 / 第 156 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 157 / 第 157 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 158 / 第 158 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 159 / 第 159 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 160 / 第 160 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 161-176 / 第 161-176 行
```cpp
161 | size_t MutationDispatcher::Mutate_ChangeByte(uint8_t *Data, size_t Size,
162 |                                              size_t MaxSize) {
163 |   if (Size > MaxSize) return 0;
164 |   size_t Idx = Rand(Size);
165 |   Data[Idx] = RandCh(Rand);
166 |   return Size;
167 | }
168 | 
169 | size_t MutationDispatcher::Mutate_ChangeBit(uint8_t *Data, size_t Size,
170 |                                             size_t MaxSize) {
171 |   if (Size > MaxSize) return 0;
172 |   size_t Idx = Rand(Size);
173 |   Data[Idx] ^= 1 << Rand(8);
174 |   return Size;
175 | }
176 | 
```
- **Line 161 / 第 161 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 162 / 第 162 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 163 / 第 163 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 164 / 第 164 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 165 / 第 165 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 166 / 第 166 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 167 / 第 167 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 168 / 第 168 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 169 / 第 169 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 170 / 第 170 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 171 / 第 171 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 172 / 第 172 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 173 / 第 173 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 174 / 第 174 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 175 / 第 175 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 176 / 第 176 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 177-192 / 第 177-192 行
```cpp
177 | size_t MutationDispatcher::Mutate_AddWordFromManualDictionary(uint8_t *Data,
178 |                                                               size_t Size,
179 |                                                               size_t MaxSize) {
180 |   return AddWordFromDictionary(ManualDictionary, Data, Size, MaxSize);
181 | }
182 | 
183 | size_t MutationDispatcher::ApplyDictionaryEntry(uint8_t *Data, size_t Size,
184 |                                                 size_t MaxSize,
185 |                                                 DictionaryEntry &DE) {
186 |   const Word &W = DE.GetW();
187 |   bool UsePositionHint = DE.HasPositionHint() &&
188 |                          DE.GetPositionHint() + W.size() < Size &&
189 |                          Rand.RandBool();
190 |   if (Rand.RandBool()) {  // Insert W.
191 |     if (Size + W.size() > MaxSize) return 0;
192 |     size_t Idx = UsePositionHint ? DE.GetPositionHint() : Rand(Size + 1);
```
- **Line 177 / 第 177 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 178 / 第 178 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 179 / 第 179 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 180 / 第 180 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 181 / 第 181 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 182 / 第 182 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 183 / 第 183 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 184 / 第 184 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 185 / 第 185 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 186 / 第 186 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 187 / 第 187 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 188 / 第 188 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 189 / 第 189 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 190 / 第 190 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 191 / 第 191 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 192 / 第 192 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 193-208 / 第 193-208 行
```cpp
193 |     memmove(Data + Idx + W.size(), Data + Idx, Size - Idx);
194 |     memcpy(Data + Idx, W.data(), W.size());
195 |     Size += W.size();
196 |   } else {  // Overwrite some bytes with W.
197 |     if (W.size() > Size) return 0;
198 |     size_t Idx =
199 |         UsePositionHint ? DE.GetPositionHint() : Rand(Size + 1 - W.size());
200 |     memcpy(Data + Idx, W.data(), W.size());
201 |   }
202 |   return Size;
203 | }
204 | 
205 | // Somewhere in the past we have observed a comparison instructions
206 | // with arguments Arg1 Arg2. This function tries to guess a dictionary
207 | // entry that will satisfy that comparison.
208 | // It first tries to find one of the arguments (possibly swapped) in the
```
- **Line 193 / 第 193 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 194 / 第 194 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 195 / 第 195 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 196 / 第 196 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 197 / 第 197 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 198 / 第 198 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 199 / 第 199 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 200 / 第 200 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 201 / 第 201 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 202 / 第 202 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 203 / 第 203 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 204 / 第 204 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 205 / 第 205 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 206 / 第 206 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 207 / 第 207 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 208 / 第 208 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 209-224 / 第 209-224 行
```cpp
209 | // input and if it succeeds it creates a DE with a position hint.
210 | // Otherwise it creates a DE with one of the arguments w/o a position hint.
211 | DictionaryEntry MutationDispatcher::MakeDictionaryEntryFromCMP(
212 |     const void *Arg1, const void *Arg2,
213 |     const void *Arg1Mutation, const void *Arg2Mutation,
214 |     size_t ArgSize, const uint8_t *Data,
215 |     size_t Size) {
216 |   bool HandleFirst = Rand.RandBool();
217 |   const void *ExistingBytes, *DesiredBytes;
218 |   Word W;
219 |   const uint8_t *End = Data + Size;
220 |   for (int Arg = 0; Arg < 2; Arg++) {
221 |     ExistingBytes = HandleFirst ? Arg1 : Arg2;
222 |     DesiredBytes = HandleFirst ? Arg2Mutation : Arg1Mutation;
223 |     HandleFirst = !HandleFirst;
224 |     W.Set(reinterpret_cast<const uint8_t*>(DesiredBytes), ArgSize);
```
- **Line 209 / 第 209 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 210 / 第 210 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 211 / 第 211 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 212 / 第 212 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 213 / 第 213 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 214 / 第 214 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 215 / 第 215 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 216 / 第 216 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 217 / 第 217 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 218 / 第 218 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 219 / 第 219 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 220 / 第 220 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 221 / 第 221 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 222 / 第 222 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 223 / 第 223 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 224 / 第 224 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 225-240 / 第 225-240 行
```cpp
225 |     const size_t kMaxNumPositions = 8;
226 |     size_t Positions[kMaxNumPositions];
227 |     size_t NumPositions = 0;
228 |     for (const uint8_t *Cur = Data;
229 |          Cur < End && NumPositions < kMaxNumPositions; Cur++) {
230 |       Cur =
231 |           (const uint8_t *)SearchMemory(Cur, End - Cur, ExistingBytes, ArgSize);
232 |       if (!Cur) break;
233 |       Positions[NumPositions++] = Cur - Data;
234 |     }
235 |     if (!NumPositions) continue;
236 |     return DictionaryEntry(W, Positions[Rand(NumPositions)]);
237 |   }
238 |   DictionaryEntry DE(W);
239 |   return DE;
240 | }
```
- **Line 225 / 第 225 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 226 / 第 226 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 227 / 第 227 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 228 / 第 228 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 229 / 第 229 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 230 / 第 230 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 231 / 第 231 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 232 / 第 232 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 233 / 第 233 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 234 / 第 234 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 235 / 第 235 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 236 / 第 236 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 237 / 第 237 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 238 / 第 238 行**: EN: Declares function or method `DE`. CN: 声明函数或方法 `DE`。
- **Line 239 / 第 239 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 240 / 第 240 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 241-256 / 第 241-256 行
```cpp
241 | 
242 | 
243 | template <class T>
244 | DictionaryEntry MutationDispatcher::MakeDictionaryEntryFromCMP(
245 |     T Arg1, T Arg2, const uint8_t *Data, size_t Size) {
246 |   if (Rand.RandBool()) Arg1 = Bswap(Arg1);
247 |   if (Rand.RandBool()) Arg2 = Bswap(Arg2);
248 |   T Arg1Mutation = static_cast<T>(Arg1 + Rand(-1, 1));
249 |   T Arg2Mutation = static_cast<T>(Arg2 + Rand(-1, 1));
250 |   return MakeDictionaryEntryFromCMP(&Arg1, &Arg2, &Arg1Mutation, &Arg2Mutation,
251 |                                     sizeof(Arg1), Data, Size);
252 | }
253 | 
254 | DictionaryEntry MutationDispatcher::MakeDictionaryEntryFromCMP(
255 |     const Word &Arg1, const Word &Arg2, const uint8_t *Data, size_t Size) {
256 |   return MakeDictionaryEntryFromCMP(Arg1.data(), Arg2.data(), Arg1.data(),
```
- **Line 241 / 第 241 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 242 / 第 242 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 243 / 第 243 行**: EN: Introduces a template parameter list for generic code. CN: 为泛型代码引入模板参数列表。
- **Line 244 / 第 244 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 245 / 第 245 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 246 / 第 246 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 247 / 第 247 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 248 / 第 248 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 249 / 第 249 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 250 / 第 250 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 251 / 第 251 行**: EN: Declares function or method `sizeof`. CN: 声明函数或方法 `sizeof`。
- **Line 252 / 第 252 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 253 / 第 253 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 254 / 第 254 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 255 / 第 255 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 256 / 第 256 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 257-272 / 第 257-272 行
```cpp
257 |                                     Arg2.data(), Arg1.size(), Data, Size);
258 | }
259 | 
260 | size_t MutationDispatcher::Mutate_AddWordFromTORC(
261 |     uint8_t *Data, size_t Size, size_t MaxSize) {
262 |   Word W;
263 |   DictionaryEntry DE;
264 |   switch (Rand(4)) {
265 |   case 0: {
266 |     auto X = TPC.TORC8.Get(Rand.Rand<size_t>());
267 |     DE = MakeDictionaryEntryFromCMP(X.A, X.B, Data, Size);
268 |   } break;
269 |   case 1: {
270 |     auto X = TPC.TORC4.Get(Rand.Rand<size_t>());
271 |     if ((X.A >> 16) == 0 && (X.B >> 16) == 0 && Rand.RandBool())
272 |       DE = MakeDictionaryEntryFromCMP((uint16_t)X.A, (uint16_t)X.B, Data, Size);
```
- **Line 257 / 第 257 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 258 / 第 258 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 259 / 第 259 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 260 / 第 260 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 261 / 第 261 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 262 / 第 262 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 263 / 第 263 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 264 / 第 264 行**: EN: Dispatches control flow based on a selector expression. CN: 根据选择表达式分发控制流。
- **Line 265 / 第 265 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 266 / 第 266 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 267 / 第 267 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 268 / 第 268 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 269 / 第 269 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 270 / 第 270 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 271 / 第 271 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 272 / 第 272 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 273-288 / 第 273-288 行
```cpp
273 |     else
274 |       DE = MakeDictionaryEntryFromCMP(X.A, X.B, Data, Size);
275 |   } break;
276 |   case 2: {
277 |     auto X = TPC.TORCW.Get(Rand.Rand<size_t>());
278 |     DE = MakeDictionaryEntryFromCMP(X.A, X.B, Data, Size);
279 |   } break;
280 |   case 3: if (Options.UseMemmem) {
281 |       auto X = TPC.MMT.Get(Rand.Rand<size_t>());
282 |       DE = DictionaryEntry(X);
283 |   } break;
284 |   default:
285 |     assert(0);
286 |   }
287 |   if (!DE.GetW().size()) return 0;
288 |   Size = ApplyDictionaryEntry(Data, Size, MaxSize, DE);
```
- **Line 273 / 第 273 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 274 / 第 274 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 275 / 第 275 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 276 / 第 276 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 277 / 第 277 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 278 / 第 278 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 279 / 第 279 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 280 / 第 280 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 281 / 第 281 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 282 / 第 282 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 283 / 第 283 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 284 / 第 284 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 285 / 第 285 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 286 / 第 286 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 287 / 第 287 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 288 / 第 288 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 289-304 / 第 289-304 行
```cpp
289 |   if (!Size) return 0;
290 |   DictionaryEntry &DERef =
291 |       CmpDictionaryEntriesDeque[CmpDictionaryEntriesDequeIdx++ %
292 |                                 kCmpDictionaryEntriesDequeSize];
293 |   DERef = DE;
294 |   CurrentDictionaryEntrySequence.push_back(&DERef);
295 |   return Size;
296 | }
297 | 
298 | size_t MutationDispatcher::Mutate_AddWordFromPersistentAutoDictionary(
299 |     uint8_t *Data, size_t Size, size_t MaxSize) {
300 |   return AddWordFromDictionary(PersistentAutoDictionary, Data, Size, MaxSize);
301 | }
302 | 
303 | size_t MutationDispatcher::AddWordFromDictionary(Dictionary &D, uint8_t *Data,
304 |                                                  size_t Size, size_t MaxSize) {
```
- **Line 289 / 第 289 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 290 / 第 290 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 291 / 第 291 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 292 / 第 292 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 293 / 第 293 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 294 / 第 294 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 295 / 第 295 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 296 / 第 296 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 297 / 第 297 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 298 / 第 298 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 299 / 第 299 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 300 / 第 300 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 301 / 第 301 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 302 / 第 302 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 303 / 第 303 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 304 / 第 304 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 305-320 / 第 305-320 行
```cpp
305 |   if (Size > MaxSize) return 0;
306 |   if (D.empty()) return 0;
307 |   DictionaryEntry &DE = D[Rand(D.size())];
308 |   Size = ApplyDictionaryEntry(Data, Size, MaxSize, DE);
309 |   if (!Size) return 0;
310 |   DE.IncUseCount();
311 |   CurrentDictionaryEntrySequence.push_back(&DE);
312 |   return Size;
313 | }
314 | 
315 | // Overwrites part of To[0,ToSize) with a part of From[0,FromSize).
316 | // Returns ToSize.
317 | size_t MutationDispatcher::CopyPartOf(const uint8_t *From, size_t FromSize,
318 |                                       uint8_t *To, size_t ToSize) {
319 |   // Copy From[FromBeg, FromBeg + CopySize) into To[ToBeg, ToBeg + CopySize).
320 |   size_t ToBeg = Rand(ToSize);
```
- **Line 305 / 第 305 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 306 / 第 306 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 307 / 第 307 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 308 / 第 308 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 309 / 第 309 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 310 / 第 310 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 311 / 第 311 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 312 / 第 312 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 313 / 第 313 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 314 / 第 314 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 315 / 第 315 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 316 / 第 316 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 317 / 第 317 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 318 / 第 318 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 319 / 第 319 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 320 / 第 320 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 321-336 / 第 321-336 行
```cpp
321 |   size_t CopySize = Rand(ToSize - ToBeg) + 1;
322 |   assert(ToBeg + CopySize <= ToSize);
323 |   CopySize = std::min(CopySize, FromSize);
324 |   size_t FromBeg = Rand(FromSize - CopySize + 1);
325 |   assert(FromBeg + CopySize <= FromSize);
326 |   memmove(To + ToBeg, From + FromBeg, CopySize);
327 |   return ToSize;
328 | }
329 | 
330 | // Inserts part of From[0,ToSize) into To.
331 | // Returns new size of To on success or 0 on failure.
332 | size_t MutationDispatcher::InsertPartOf(const uint8_t *From, size_t FromSize,
333 |                                         uint8_t *To, size_t ToSize,
334 |                                         size_t MaxToSize) {
335 |   if (ToSize >= MaxToSize) return 0;
336 |   size_t AvailableSpace = MaxToSize - ToSize;
```
- **Line 321 / 第 321 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 322 / 第 322 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 323 / 第 323 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 324 / 第 324 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 325 / 第 325 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 326 / 第 326 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 327 / 第 327 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 328 / 第 328 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 329 / 第 329 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 330 / 第 330 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 331 / 第 331 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 332 / 第 332 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 333 / 第 333 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 334 / 第 334 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 335 / 第 335 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 336 / 第 336 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 337-352 / 第 337-352 行
```cpp
337 |   size_t MaxCopySize = std::min(AvailableSpace, FromSize);
338 |   size_t CopySize = Rand(MaxCopySize) + 1;
339 |   size_t FromBeg = Rand(FromSize - CopySize + 1);
340 |   assert(FromBeg + CopySize <= FromSize);
341 |   size_t ToInsertPos = Rand(ToSize + 1);
342 |   assert(ToInsertPos + CopySize <= MaxToSize);
343 |   size_t TailSize = ToSize - ToInsertPos;
344 |   if (To == From) {
345 |     MutateInPlaceHere.resize(MaxToSize);
346 |     memcpy(MutateInPlaceHere.data(), From + FromBeg, CopySize);
347 |     memmove(To + ToInsertPos + CopySize, To + ToInsertPos, TailSize);
348 |     memmove(To + ToInsertPos, MutateInPlaceHere.data(), CopySize);
349 |   } else {
350 |     memmove(To + ToInsertPos + CopySize, To + ToInsertPos, TailSize);
351 |     memmove(To + ToInsertPos, From + FromBeg, CopySize);
352 |   }
```
- **Line 337 / 第 337 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 338 / 第 338 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 339 / 第 339 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 340 / 第 340 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 341 / 第 341 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 342 / 第 342 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 343 / 第 343 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 344 / 第 344 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 345 / 第 345 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 346 / 第 346 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 347 / 第 347 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 348 / 第 348 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 349 / 第 349 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 350 / 第 350 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 351 / 第 351 行**: EN: Declares function or method `memmove`. CN: 声明函数或方法 `memmove`。
- **Line 352 / 第 352 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 353-368 / 第 353-368 行
```cpp
353 |   return ToSize + CopySize;
354 | }
355 | 
356 | size_t MutationDispatcher::Mutate_CopyPart(uint8_t *Data, size_t Size,
357 |                                            size_t MaxSize) {
358 |   if (Size > MaxSize || Size == 0) return 0;
359 |   // If Size == MaxSize, `InsertPartOf(...)` will
360 |   // fail so there's no point using it in this case.
361 |   if (Size == MaxSize || Rand.RandBool())
362 |     return CopyPartOf(Data, Size, Data, Size);
363 |   else
364 |     return InsertPartOf(Data, Size, Data, Size, MaxSize);
365 | }
366 | 
367 | size_t MutationDispatcher::Mutate_ChangeASCIIInteger(uint8_t *Data, size_t Size,
368 |                                                      size_t MaxSize) {
```
- **Line 353 / 第 353 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 354 / 第 354 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 355 / 第 355 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 356 / 第 356 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 357 / 第 357 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 358 / 第 358 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 359 / 第 359 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 360 / 第 360 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 361 / 第 361 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 362 / 第 362 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 363 / 第 363 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 364 / 第 364 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 365 / 第 365 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 366 / 第 366 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 367 / 第 367 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 368 / 第 368 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 369-384 / 第 369-384 行
```cpp
369 |   if (Size > MaxSize) return 0;
370 |   size_t B = Rand(Size);
371 |   while (B < Size && !isdigit(Data[B])) B++;
372 |   if (B == Size) return 0;
373 |   size_t E = B;
374 |   while (E < Size && isdigit(Data[E])) E++;
375 |   assert(B < E);
376 |   // now we have digits in [B, E).
377 |   // strtol and friends don't accept non-zero-teminated data, parse it manually.
378 |   uint64_t Val = Data[B] - '0';
379 |   for (size_t i = B + 1; i < E; i++)
380 |     Val = Val * 10 + Data[i] - '0';
381 | 
382 |   // Mutate the integer value.
383 |   switch(Rand(5)) {
384 |     case 0: Val++; break;
```
- **Line 369 / 第 369 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 370 / 第 370 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 371 / 第 371 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 372 / 第 372 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 373 / 第 373 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 374 / 第 374 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 375 / 第 375 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 376 / 第 376 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 377 / 第 377 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 378 / 第 378 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 379 / 第 379 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 380 / 第 380 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 381 / 第 381 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 382 / 第 382 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 383 / 第 383 行**: EN: Dispatches control flow based on a selector expression. CN: 根据选择表达式分发控制流。
- **Line 384 / 第 384 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。

### Lines 385-400 / 第 385-400 行
```cpp
385 |     case 1: Val--; break;
386 |     case 2: Val /= 2; break;
387 |     case 3: Val *= 2; break;
388 |     case 4: Val = Rand(Val * Val); break;
389 |     default: assert(0);
390 |   }
391 |   // Just replace the bytes with the new ones, don't bother moving bytes.
392 |   for (size_t i = B; i < E; i++) {
393 |     size_t Idx = E + B - i - 1;
394 |     assert(Idx >= B && Idx < E);
395 |     Data[Idx] = (Val % 10) + '0';
396 |     Val /= 10;
397 |   }
398 |   return Size;
399 | }
400 | 
```
- **Line 385 / 第 385 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 386 / 第 386 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 387 / 第 387 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 388 / 第 388 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 389 / 第 389 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 390 / 第 390 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 391 / 第 391 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 392 / 第 392 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 393 / 第 393 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 394 / 第 394 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 395 / 第 395 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 396 / 第 396 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 397 / 第 397 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 398 / 第 398 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 399 / 第 399 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 400 / 第 400 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 401-416 / 第 401-416 行
```cpp
401 | template<class T>
402 | size_t ChangeBinaryInteger(uint8_t *Data, size_t Size, Random &Rand) {
403 |   if (Size < sizeof(T)) return 0;
404 |   size_t Off = Rand(Size - sizeof(T) + 1);
405 |   assert(Off + sizeof(T) <= Size);
406 |   T Val;
407 |   if (Off < 64 && !Rand(4)) {
408 |     Val = static_cast<T>(Size);
409 |     if (Rand.RandBool())
410 |       Val = Bswap(Val);
411 |   } else {
412 |     memcpy(&Val, Data + Off, sizeof(Val));
413 |     T Add = static_cast<T>(Rand(21));
414 |     Add -= 10;
415 |     if (Rand.RandBool())
416 |       Val = Bswap(T(Bswap(Val) + Add)); // Add assuming different endianness.
```
- **Line 401 / 第 401 行**: EN: Introduces a template parameter list for generic code. CN: 为泛型代码引入模板参数列表。
- **Line 402 / 第 402 行**: EN: Starts the definition of function or method `ChangeBinaryInteger`. CN: 开始定义函数或方法 `ChangeBinaryInteger`。
- **Line 403 / 第 403 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 404 / 第 404 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 405 / 第 405 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 406 / 第 406 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 407 / 第 407 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 408 / 第 408 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 409 / 第 409 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 410 / 第 410 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 411 / 第 411 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 412 / 第 412 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 413 / 第 413 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 414 / 第 414 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 415 / 第 415 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 416 / 第 416 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 417-432 / 第 417-432 行
```cpp
417 |     else
418 |       Val = Val + Add;               // Add assuming current endianness.
419 |     if (Add == 0 || Rand.RandBool()) // Maybe negate.
420 |       Val = -Val;
421 |   }
422 |   memcpy(Data + Off, &Val, sizeof(Val));
423 |   return Size;
424 | }
425 | 
426 | size_t MutationDispatcher::Mutate_ChangeBinaryInteger(uint8_t *Data,
427 |                                                       size_t Size,
428 |                                                       size_t MaxSize) {
429 |   if (Size > MaxSize) return 0;
430 |   switch (Rand(4)) {
431 |     case 3: return ChangeBinaryInteger<uint64_t>(Data, Size, Rand);
432 |     case 2: return ChangeBinaryInteger<uint32_t>(Data, Size, Rand);
```
- **Line 417 / 第 417 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 418 / 第 418 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 419 / 第 419 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 420 / 第 420 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 421 / 第 421 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 422 / 第 422 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 423 / 第 423 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 424 / 第 424 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 425 / 第 425 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 426 / 第 426 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 427 / 第 427 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 428 / 第 428 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 429 / 第 429 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 430 / 第 430 行**: EN: Dispatches control flow based on a selector expression. CN: 根据选择表达式分发控制流。
- **Line 431 / 第 431 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 432 / 第 432 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。

### Lines 433-448 / 第 433-448 行
```cpp
433 |     case 1: return ChangeBinaryInteger<uint16_t>(Data, Size, Rand);
434 |     case 0: return ChangeBinaryInteger<uint8_t>(Data, Size, Rand);
435 |     default: assert(0);
436 |   }
437 |   return 0;
438 | }
439 | 
440 | size_t MutationDispatcher::Mutate_CrossOver(uint8_t *Data, size_t Size,
441 |                                             size_t MaxSize) {
442 |   if (Size > MaxSize) return 0;
443 |   if (Size == 0) return 0;
444 |   if (!CrossOverWith) return 0;
445 |   const Unit &O = *CrossOverWith;
446 |   if (O.empty()) return 0;
447 |   size_t NewSize = 0;
448 |   switch(Rand(3)) {
```
- **Line 433 / 第 433 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 434 / 第 434 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 435 / 第 435 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 436 / 第 436 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 437 / 第 437 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 438 / 第 438 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 439 / 第 439 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 440 / 第 440 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 441 / 第 441 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 442 / 第 442 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 443 / 第 443 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 444 / 第 444 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 445 / 第 445 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 446 / 第 446 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 447 / 第 447 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 448 / 第 448 行**: EN: Dispatches control flow based on a selector expression. CN: 根据选择表达式分发控制流。

### Lines 449-464 / 第 449-464 行
```cpp
449 |     case 0:
450 |       MutateInPlaceHere.resize(MaxSize);
451 |       NewSize = CrossOver(Data, Size, O.data(), O.size(),
452 |                           MutateInPlaceHere.data(), MaxSize);
453 |       memcpy(Data, MutateInPlaceHere.data(), NewSize);
454 |       break;
455 |     case 1:
456 |       NewSize = InsertPartOf(O.data(), O.size(), Data, Size, MaxSize);
457 |       if (!NewSize)
458 |         NewSize = CopyPartOf(O.data(), O.size(), Data, Size);
459 |       break;
460 |     case 2:
461 |       NewSize = CopyPartOf(O.data(), O.size(), Data, Size);
462 |       break;
463 |     default: assert(0);
464 |   }
```
- **Line 449 / 第 449 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 450 / 第 450 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 451 / 第 451 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 452 / 第 452 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 453 / 第 453 行**: EN: Declares function or method `memcpy`. CN: 声明函数或方法 `memcpy`。
- **Line 454 / 第 454 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 455 / 第 455 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 456 / 第 456 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 457 / 第 457 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 458 / 第 458 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 459 / 第 459 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 460 / 第 460 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 461 / 第 461 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 462 / 第 462 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 463 / 第 463 行**: EN: Introduces a labeled branch inside a switch statement. CN: 在 switch 语句中引入一个带标签的分支。
- **Line 464 / 第 464 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 465-480 / 第 465-480 行
```cpp
465 |   assert(NewSize > 0 && "CrossOver returned empty unit");
466 |   assert(NewSize <= MaxSize && "CrossOver returned oversized unit");
467 |   return NewSize;
468 | }
469 | 
470 | void MutationDispatcher::StartMutationSequence() {
471 |   CurrentMutatorSequence.clear();
472 |   CurrentDictionaryEntrySequence.clear();
473 | }
474 | 
475 | // Copy successful dictionary entries to PersistentAutoDictionary.
476 | void MutationDispatcher::RecordSuccessfulMutationSequence() {
477 |   for (auto DE : CurrentDictionaryEntrySequence) {
478 |     // PersistentAutoDictionary.AddWithSuccessCountOne(DE);
479 |     DE->IncSuccessCount();
480 |     assert(DE->GetW().size());
```
- **Line 465 / 第 465 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 466 / 第 466 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 467 / 第 467 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 468 / 第 468 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 469 / 第 469 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 470 / 第 470 行**: EN: Starts the definition of function or method `MutationDispatcher::StartMutationSequence`. CN: 开始定义函数或方法 `MutationDispatcher::StartMutationSequence`。
- **Line 471 / 第 471 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 472 / 第 472 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 473 / 第 473 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 474 / 第 474 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 475 / 第 475 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 476 / 第 476 行**: EN: Starts the definition of function or method `MutationDispatcher::RecordSuccessfulMutationSequence`. CN: 开始定义函数或方法 `MutationDispatcher::RecordSuccessfulMutationSequence`。
- **Line 477 / 第 477 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 478 / 第 478 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 479 / 第 479 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 480 / 第 480 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。

### Lines 481-496 / 第 481-496 行
```cpp
481 |     // Linear search is fine here as this happens seldom.
482 |     if (!PersistentAutoDictionary.ContainsWord(DE->GetW()))
483 |       PersistentAutoDictionary.push_back(*DE);
484 |   }
485 | }
486 | 
487 | void MutationDispatcher::PrintRecommendedDictionary() {
488 |   std::vector<DictionaryEntry> V;
489 |   for (auto &DE : PersistentAutoDictionary)
490 |     if (!ManualDictionary.ContainsWord(DE.GetW()))
491 |       V.push_back(DE);
492 |   if (V.empty()) return;
493 |   Printf("###### Recommended dictionary. ######\n");
494 |   for (auto &DE: V) {
495 |     assert(DE.GetW().size());
496 |     Printf("\"");
```
- **Line 481 / 第 481 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 482 / 第 482 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 483 / 第 483 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 484 / 第 484 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 485 / 第 485 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 486 / 第 486 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 487 / 第 487 行**: EN: Starts the definition of function or method `MutationDispatcher::PrintRecommendedDictionary`. CN: 开始定义函数或方法 `MutationDispatcher::PrintRecommendedDictionary`。
- **Line 488 / 第 488 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 489 / 第 489 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 490 / 第 490 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 491 / 第 491 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 492 / 第 492 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 493 / 第 493 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 494 / 第 494 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 495 / 第 495 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 496 / 第 496 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。

### Lines 497-512 / 第 497-512 行
```cpp
497 |     PrintASCII(DE.GetW(), "\"");
498 |     Printf(" # Uses: %zd\n", DE.GetUseCount());
499 |   }
500 |   Printf("###### End of recommended dictionary. ######\n");
501 | }
502 | 
503 | void MutationDispatcher::PrintMutationSequence(bool Verbose) {
504 |   Printf("MS: %zd ", CurrentMutatorSequence.size());
505 |   size_t EntriesToPrint =
506 |       Verbose ? CurrentMutatorSequence.size()
507 |               : std::min(kMaxMutationsToPrint, CurrentMutatorSequence.size());
508 |   for (size_t i = 0; i < EntriesToPrint; i++)
509 |     Printf("%s-", CurrentMutatorSequence[i].Name);
510 |   if (!CurrentDictionaryEntrySequence.empty()) {
511 |     Printf(" DE: ");
512 |     EntriesToPrint = Verbose ? CurrentDictionaryEntrySequence.size()
```
- **Line 497 / 第 497 行**: EN: Declares function or method `PrintASCII`. CN: 声明函数或方法 `PrintASCII`。
- **Line 498 / 第 498 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 499 / 第 499 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 500 / 第 500 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 501 / 第 501 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 502 / 第 502 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 503 / 第 503 行**: EN: Starts the definition of function or method `MutationDispatcher::PrintMutationSequence`. CN: 开始定义函数或方法 `MutationDispatcher::PrintMutationSequence`。
- **Line 504 / 第 504 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 505 / 第 505 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 506 / 第 506 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 507 / 第 507 行**: EN: Declares function or method `std::min`. CN: 声明函数或方法 `std::min`。
- **Line 508 / 第 508 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 509 / 第 509 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 510 / 第 510 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 511 / 第 511 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 512 / 第 512 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 513-528 / 第 513-528 行
```cpp
513 |                              : std::min(kMaxMutationsToPrint,
514 |                                         CurrentDictionaryEntrySequence.size());
515 |     for (size_t i = 0; i < EntriesToPrint; i++) {
516 |       Printf("\"");
517 |       PrintASCII(CurrentDictionaryEntrySequence[i]->GetW(), "\"-");
518 |     }
519 |   }
520 | }
521 | 
522 | std::string MutationDispatcher::MutationSequence() {
523 |   std::string MS;
524 |   for (const auto &M : CurrentMutatorSequence) {
525 |     MS += M.Name;
526 |     MS += "-";
527 |   }
528 |   return MS;
```
- **Line 513 / 第 513 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 514 / 第 514 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 515 / 第 515 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 516 / 第 516 行**: EN: Declares function or method `Printf`. CN: 声明函数或方法 `Printf`。
- **Line 517 / 第 517 行**: EN: Declares function or method `PrintASCII`. CN: 声明函数或方法 `PrintASCII`。
- **Line 518 / 第 518 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 519 / 第 519 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 520 / 第 520 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 521 / 第 521 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 522 / 第 522 行**: EN: Starts the definition of function or method `MutationDispatcher::MutationSequence`. CN: 开始定义函数或方法 `MutationDispatcher::MutationSequence`。
- **Line 523 / 第 523 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 524 / 第 524 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 525 / 第 525 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 526 / 第 526 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 527 / 第 527 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 528 / 第 528 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 529-544 / 第 529-544 行
```cpp
529 | }
530 | 
531 | size_t MutationDispatcher::Mutate(uint8_t *Data, size_t Size, size_t MaxSize) {
532 |   return MutateImpl(Data, Size, MaxSize, Mutators);
533 | }
534 | 
535 | size_t MutationDispatcher::DefaultMutate(uint8_t *Data, size_t Size,
536 |                                          size_t MaxSize) {
537 |   return MutateImpl(Data, Size, MaxSize, DefaultMutators);
538 | }
539 | 
540 | // Mutates Data in place, returns new size.
541 | size_t MutationDispatcher::MutateImpl(uint8_t *Data, size_t Size,
542 |                                       size_t MaxSize,
543 |                                       std::vector<Mutator> &Mutators) {
544 |   assert(MaxSize > 0);
```
- **Line 529 / 第 529 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 530 / 第 530 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 531 / 第 531 行**: EN: Starts the definition of function or method `MutationDispatcher::Mutate`. CN: 开始定义函数或方法 `MutationDispatcher::Mutate`。
- **Line 532 / 第 532 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 533 / 第 533 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 534 / 第 534 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 535 / 第 535 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 536 / 第 536 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 537 / 第 537 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 538 / 第 538 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 539 / 第 539 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 540 / 第 540 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 541 / 第 541 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 542 / 第 542 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 543 / 第 543 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 544 / 第 544 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。

### Lines 545-560 / 第 545-560 行
```cpp
545 |   // Some mutations may fail (e.g. can't insert more bytes if Size == MaxSize),
546 |   // in which case they will return 0.
547 |   // Try several times before returning un-mutated data.
548 |   for (int Iter = 0; Iter < 100; Iter++) {
549 |     auto M = Mutators[Rand(Mutators.size())];
550 |     size_t NewSize = (this->*(M.Fn))(Data, Size, MaxSize);
551 |     if (NewSize && NewSize <= MaxSize) {
552 |       if (Options.OnlyASCII)
553 |         ToASCII(Data, NewSize);
554 |       CurrentMutatorSequence.push_back(M);
555 |       return NewSize;
556 |     }
557 |   }
558 |   *Data = ' ';
559 |   return 1;   // Fallback, should not happen frequently.
560 | }
```
- **Line 545 / 第 545 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 546 / 第 546 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 547 / 第 547 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 548 / 第 548 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 549 / 第 549 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 550 / 第 550 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 551 / 第 551 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 552 / 第 552 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 553 / 第 553 行**: EN: Declares function or method `ToASCII`. CN: 声明函数或方法 `ToASCII`。
- **Line 554 / 第 554 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 555 / 第 555 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 556 / 第 556 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 557 / 第 557 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 558 / 第 558 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 559 / 第 559 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 560 / 第 560 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 561-576 / 第 561-576 行
```cpp
561 | 
562 | // Mask represents the set of Data bytes that are worth mutating.
563 | size_t MutationDispatcher::MutateWithMask(uint8_t *Data, size_t Size,
564 |                                           size_t MaxSize,
565 |                                           const std::vector<uint8_t> &Mask) {
566 |   size_t MaskedSize = std::min(Size, Mask.size());
567 |   // * Copy the worthy bytes into a temporary array T
568 |   // * Mutate T
569 |   // * Copy T back.
570 |   // This is totally unoptimized.
571 |   auto &T = MutateWithMaskTemp;
572 |   if (T.size() < Size)
573 |     T.resize(Size);
574 |   size_t OneBits = 0;
575 |   for (size_t I = 0; I < MaskedSize; I++)
576 |     if (Mask[I])
```
- **Line 561 / 第 561 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 562 / 第 562 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 563 / 第 563 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 564 / 第 564 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 565 / 第 565 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 566 / 第 566 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 567 / 第 567 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 568 / 第 568 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 569 / 第 569 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 570 / 第 570 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 571 / 第 571 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 572 / 第 572 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 573 / 第 573 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 574 / 第 574 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 575 / 第 575 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 576 / 第 576 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 577-592 / 第 577-592 行
```cpp
577 |       T[OneBits++] = Data[I];
578 | 
579 |   if (!OneBits) return 0;
580 |   assert(!T.empty());
581 |   size_t NewSize = Mutate(T.data(), OneBits, OneBits);
582 |   assert(NewSize <= OneBits);
583 |   (void)NewSize;
584 |   // Even if NewSize < OneBits we still use all OneBits bytes.
585 |   for (size_t I = 0, J = 0; I < MaskedSize; I++)
586 |     if (Mask[I])
587 |       Data[I] = T[J++];
588 |   return Size;
589 | }
590 | 
591 | void MutationDispatcher::AddWordToManualDictionary(const Word &W) {
592 |   ManualDictionary.push_back(
```
- **Line 577 / 第 577 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 578 / 第 578 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 579 / 第 579 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 580 / 第 580 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 581 / 第 581 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 582 / 第 582 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 583 / 第 583 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 584 / 第 584 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 585 / 第 585 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 586 / 第 586 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 587 / 第 587 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 588 / 第 588 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 589 / 第 589 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 590 / 第 590 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 591 / 第 591 行**: EN: Starts the definition of function or method `MutationDispatcher::AddWordToManualDictionary`. CN: 开始定义函数或方法 `MutationDispatcher::AddWordToManualDictionary`。
- **Line 592 / 第 592 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 593-596 / 第 593-596 行
```cpp
593 |       {W, std::numeric_limits<size_t>::max()});
594 | }
595 | 
596 | }  // namespace fuzzer
```
- **Line 593 / 第 593 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 594 / 第 594 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 595 / 第 595 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 596 / 第 596 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

## Key Concepts / 关键概念

- **EN**: fuzz input decomposition
  - **CN**: 模糊测试输入拆分
- **EN**: deterministic test input consumption
  - **CN**: 确定性的测试输入消费
- **EN**: coverage-guided fuzzing runtime
  - **CN**: 覆盖率引导的 fuzzing 运行时
- **EN**: namespace scoping and organization
  - **CN**: 命名空间作用域与组织
- **EN**: template-based generic code
  - **CN**: 基于模板的泛型代码

## Dependencies / 依赖关系

- `FuzzerDefs.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerExtFunctions.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerIO.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerMutate.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerOptions.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `FuzzerTracePC.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
