# SwitchLoweringUtils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/SwitchLoweringUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Switch Lowering` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Switch Lowering”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- SwitchLoweringUtils.cpp - Switch Lowering --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains switch inst lowering optimizations and utilities for
// codegen, so that it can be used for both SelectionDAG and GlobalISel.
//
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/SwitchLoweringUtils.h"
#include "llvm/CodeGen/FunctionLoweringInfo.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/Target/TargetMachine.h"

using namespace llvm;
````
- **L1 EN**: Comment documents: `===- SwitchLoweringUtils.cpp - Switch Lowering -------------------------…`.
  **L1 CN**: 注释说明：`===- SwitchLoweringUtils.cpp - Switch Lowering -------------------------…`。
- **L2 EN**: Continues the surrounding comment block.
  **L2 CN**: 延续周围的注释块。
- **L3 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`.
  **L3 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`。
- **L4 EN**: Comment documents: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Continues the surrounding comment block.
  **L6 CN**: 延续周围的注释块。
- **L7 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L7 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L8 EN**: Continues the surrounding comment block.
  **L8 CN**: 延续周围的注释块。
- **L9 EN**: Comment documents: `This file contains switch inst lowering optimizations and utilities for`.
  **L9 CN**: 注释说明：`This file contains switch inst lowering optimizations and utilities for`。
- **L10 EN**: Comment documents: `codegen, so that it can be used for both SelectionDAG and GlobalISel.`.
  **L10 CN**: 注释说明：`codegen, so that it can be used for both SelectionDAG and GlobalISel.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/SwitchLoweringUtils.h` for SwitchLoweringUtils support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/SwitchLoweringUtils.h`，用于 SwitchLoweringUtils 相关支持。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/FunctionLoweringInfo.h` for FunctionLoweringInfo support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/FunctionLoweringInfo.h`，用于 FunctionLoweringInfo 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/MachineJumpTableInfo.h` for MachineJumpTableInfo support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineJumpTableInfo.h`，用于 MachineJumpTableInfo 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/TargetLowering.h` for TargetLowering support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetLowering.h`，用于 TargetLowering 相关支持。
- **L18 EN**: Includes LLVM header `llvm/Target/TargetMachine.h` for TargetMachine support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/Target/TargetMachine.h`，用于 TargetMachine 相关支持。
- **L19 EN**: Separates nearby statements for readability.
  **L19 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L20 EN**: Imports namespace `llvm` into this translation unit.
  **L20 CN**: 将命名空间 `llvm` 引入当前编译单元。

### Lines 21-40

````cpp
using namespace SwitchCG;

uint64_t SwitchCG::getJumpTableRange(const CaseClusterVector &Clusters,
                                     unsigned First, unsigned Last) {
  assert(Last >= First);
  const APInt &LowCase = Clusters[First].Low->getValue();
  const APInt &HighCase = Clusters[Last].High->getValue();
  assert(LowCase.getBitWidth() == HighCase.getBitWidth());

  // FIXME: A range of consecutive cases has 100% density, but only requires one
  // comparison to lower. We should discriminate against such consecutive ranges
  // in jump tables.
  return (HighCase - LowCase).getLimitedValue((UINT64_MAX - 1) / 100) + 1;
}

uint64_t
SwitchCG::getJumpTableNumCases(const SmallVectorImpl<unsigned> &TotalCases,
                               unsigned First, unsigned Last) {
  assert(Last >= First);
  assert(TotalCases[Last] >= TotalCases[First]);
````
- **L21 EN**: Imports namespace `SwitchCG` into this translation unit.
  **L21 CN**: 将命名空间 `SwitchCG` 引入当前编译单元。
- **L22 EN**: Separates nearby statements for readability.
  **L22 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L23 EN**: Provides part of the signature for `getJumpTableRange`.
  **L23 CN**: 给出 `getJumpTableRange` 的一部分签名。
- **L24 EN**: Starts block `unsigned First, unsigned Last)`.
  **L24 CN**: 开始代码块 `unsigned First, unsigned Last)`。
- **L25 EN**: Checks an invariant in debug builds.
  **L25 CN**: 在调试构建中检查一个不变量。
- **L26 EN**: Assigns or initializes `const APInt &LowCase`.
  **L26 CN**: 对 `const APInt &LowCase` 进行赋值或初始化。
- **L27 EN**: Assigns or initializes `const APInt &HighCase`.
  **L27 CN**: 对 `const APInt &HighCase` 进行赋值或初始化。
- **L28 EN**: Checks an invariant in debug builds.
  **L28 CN**: 在调试构建中检查一个不变量。
- **L29 EN**: Separates nearby statements for readability.
  **L29 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L30 EN**: Comment documents: `FIXME: A range of consecutive cases has 100% density, but only requires …`.
  **L30 CN**: 注释说明：`FIXME: A range of consecutive cases has 100% density, but only requires …`。
- **L31 EN**: Comment documents: `comparison to lower. We should discriminate against such consecutive ran…`.
  **L31 CN**: 注释说明：`comparison to lower. We should discriminate against such consecutive ran…`。
- **L32 EN**: Comment documents: `in jump tables.`.
  **L32 CN**: 注释说明：`in jump tables.`。
- **L33 EN**: Returns `(HighCase - LowCase).getLimitedValue((UINT64_MAX - 1) / 100) + 1` to the caller.
  **L33 CN**: 向调用者返回 `(HighCase - LowCase).getLimitedValue((UINT64_MAX - 1) / 100) + 1`。
- **L34 EN**: Closes the current scope.
  **L34 CN**: 关闭当前作用域。
- **L35 EN**: Separates nearby statements for readability.
  **L35 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L36 EN**: Continues logic with `uint64_t`.
  **L36 CN**: 继续处理逻辑：`uint64_t`。
- **L37 EN**: Provides part of the signature for `getJumpTableNumCases`.
  **L37 CN**: 给出 `getJumpTableNumCases` 的一部分签名。
- **L38 EN**: Starts block `unsigned First, unsigned Last)`.
  **L38 CN**: 开始代码块 `unsigned First, unsigned Last)`。
- **L39 EN**: Checks an invariant in debug builds.
  **L39 CN**: 在调试构建中检查一个不变量。
- **L40 EN**: Checks an invariant in debug builds.
  **L40 CN**: 在调试构建中检查一个不变量。

### Lines 41-60

````cpp
  uint64_t NumCases =
      TotalCases[Last] - (First == 0 ? 0 : TotalCases[First - 1]);
  return NumCases;
}

void SwitchCG::SwitchLowering::findJumpTables(CaseClusterVector &Clusters,
                                              const SwitchInst *SI,
                                              std::optional<SDLoc> SL,
                                              MachineBasicBlock *DefaultMBB,
                                              ProfileSummaryInfo *PSI,
                                              BlockFrequencyInfo *BFI) {
#ifndef NDEBUG
  // Clusters must be non-empty, sorted, and only contain Range clusters.
  assert(!Clusters.empty());
  for (CaseCluster &C : Clusters)
    assert(C.Kind == CC_Range);
  for (unsigned i = 1, e = Clusters.size(); i < e; ++i)
    assert(Clusters[i - 1].High->getValue().slt(Clusters[i].Low->getValue()));
#endif

````
- **L41 EN**: Continues logic with `uint64_t NumCases =`.
  **L41 CN**: 继续处理逻辑：`uint64_t NumCases =`。
- **L42 EN**: Assigns or initializes `TotalCases[Last] - (First`.
  **L42 CN**: 对 `TotalCases[Last] - (First` 进行赋值或初始化。
- **L43 EN**: Returns `NumCases` to the caller.
  **L43 CN**: 向调用者返回 `NumCases`。
- **L44 EN**: Closes the current scope.
  **L44 CN**: 关闭当前作用域。
- **L45 EN**: Separates nearby statements for readability.
  **L45 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L46 EN**: Provides part of the signature for `findJumpTables`.
  **L46 CN**: 给出 `findJumpTables` 的一部分签名。
- **L47 EN**: Continues logic with `const SwitchInst *SI,`.
  **L47 CN**: 继续处理逻辑：`const SwitchInst *SI,`。
- **L48 EN**: Continues logic with `std::optional<SDLoc> SL,`.
  **L48 CN**: 继续处理逻辑：`std::optional<SDLoc> SL,`。
- **L49 EN**: Continues logic with `MachineBasicBlock *DefaultMBB,`.
  **L49 CN**: 继续处理逻辑：`MachineBasicBlock *DefaultMBB,`。
- **L50 EN**: Continues logic with `ProfileSummaryInfo *PSI,`.
  **L50 CN**: 继续处理逻辑：`ProfileSummaryInfo *PSI,`。
- **L51 EN**: Starts block `BlockFrequencyInfo *BFI)`.
  **L51 CN**: 开始代码块 `BlockFrequencyInfo *BFI)`。
- **L52 EN**: Starts a preprocessor conditional block.
  **L52 CN**: 开始一个预处理条件块。
- **L53 EN**: Comment documents: `Clusters must be non-empty, sorted, and only contain Range clusters.`.
  **L53 CN**: 注释说明：`Clusters must be non-empty, sorted, and only contain Range clusters.`。
- **L54 EN**: Checks an invariant in debug builds.
  **L54 CN**: 在调试构建中检查一个不变量。
- **L55 EN**: Starts a loop over a sequence or range.
  **L55 CN**: 开始遍历序列或范围的循环。
- **L56 EN**: Checks an invariant in debug builds.
  **L56 CN**: 在调试构建中检查一个不变量。
- **L57 EN**: Starts a loop over a sequence or range.
  **L57 CN**: 开始遍历序列或范围的循环。
- **L58 EN**: Checks an invariant in debug builds.
  **L58 CN**: 在调试构建中检查一个不变量。
- **L59 EN**: Ends the current preprocessor conditional block.
  **L59 CN**: 结束当前的预处理条件块。
- **L60 EN**: Separates nearby statements for readability.
  **L60 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 61-80

````cpp
  assert(TLI && "TLI not set!");
  if (!TLI->areJTsAllowed(SI->getParent()->getParent()))
    return;

  const unsigned MinJumpTableEntries = TLI->getMinimumJumpTableEntries();
  const unsigned SmallNumberOfEntries = MinJumpTableEntries / 2;

  // Bail if not enough cases.
  const int64_t N = Clusters.size();
  if (N < 2 || N < MinJumpTableEntries)
    return;

  // Accumulated number of cases in each cluster and those prior to it.
  SmallVector<unsigned, 8> TotalCases(N);
  for (unsigned i = 0; i < N; ++i) {
    const APInt &Hi = Clusters[i].High->getValue();
    const APInt &Lo = Clusters[i].Low->getValue();
    TotalCases[i] = (Hi - Lo).getLimitedValue() + 1;
    if (i != 0)
      TotalCases[i] += TotalCases[i - 1];
````
- **L61 EN**: Checks an invariant in debug builds.
  **L61 CN**: 在调试构建中检查一个不变量。
- **L62 EN**: Begins a conditional branch.
  **L62 CN**: 开始一个条件分支。
- **L63 EN**: Returns control to the caller.
  **L63 CN**: 将控制流返回给调用者。
- **L64 EN**: Separates nearby statements for readability.
  **L64 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L65 EN**: Assigns or initializes `const unsigned MinJumpTableEntries`.
  **L65 CN**: 对 `const unsigned MinJumpTableEntries` 进行赋值或初始化。
- **L66 EN**: Assigns or initializes `const unsigned SmallNumberOfEntries`.
  **L66 CN**: 对 `const unsigned SmallNumberOfEntries` 进行赋值或初始化。
- **L67 EN**: Separates nearby statements for readability.
  **L67 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L68 EN**: Comment documents: `Bail if not enough cases.`.
  **L68 CN**: 注释说明：`Bail if not enough cases.`。
- **L69 EN**: Assigns or initializes `const int64_t N`.
  **L69 CN**: 对 `const int64_t N` 进行赋值或初始化。
- **L70 EN**: Begins a conditional branch.
  **L70 CN**: 开始一个条件分支。
- **L71 EN**: Returns control to the caller.
  **L71 CN**: 将控制流返回给调用者。
- **L72 EN**: Separates nearby statements for readability.
  **L72 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L73 EN**: Comment documents: `Accumulated number of cases in each cluster and those prior to it.`.
  **L73 CN**: 注释说明：`Accumulated number of cases in each cluster and those prior to it.`。
- **L74 EN**: Declares function or method `TotalCases`.
  **L74 CN**: 声明函数或方法 `TotalCases`。
- **L75 EN**: Starts a loop over a sequence or range.
  **L75 CN**: 开始遍历序列或范围的循环。
- **L76 EN**: Assigns or initializes `const APInt &Hi`.
  **L76 CN**: 对 `const APInt &Hi` 进行赋值或初始化。
- **L77 EN**: Assigns or initializes `const APInt &Lo`.
  **L77 CN**: 对 `const APInt &Lo` 进行赋值或初始化。
- **L78 EN**: Assigns or initializes `TotalCases[i]`.
  **L78 CN**: 对 `TotalCases[i]` 进行赋值或初始化。
- **L79 EN**: Begins a conditional branch.
  **L79 CN**: 开始一个条件分支。
- **L80 EN**: Assigns or initializes `TotalCases[i] +`.
  **L80 CN**: 对 `TotalCases[i] +` 进行赋值或初始化。

### Lines 81-100

````cpp
  }

  uint64_t Range = getJumpTableRange(Clusters,0, N - 1);
  uint64_t NumCases = getJumpTableNumCases(TotalCases, 0, N - 1);
  assert(NumCases < UINT64_MAX / 100);
  assert(Range >= NumCases);

  // Cheap case: the whole range may be suitable for jump table.
  if (TLI->isSuitableForJumpTable(SI, NumCases, Range, PSI, BFI)) {
    CaseCluster JTCluster;
    if (buildJumpTable(Clusters, 0, N - 1, SI, SL, DefaultMBB, JTCluster)) {
      Clusters[0] = JTCluster;
      Clusters.resize(1);
      return;
    }
  }

  // The algorithm below is not suitable for -O0.
  if (TM->getOptLevel() == CodeGenOptLevel::None)
    return;
````
- **L81 EN**: Closes the current scope.
  **L81 CN**: 关闭当前作用域。
- **L82 EN**: Separates nearby statements for readability.
  **L82 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L83 EN**: Assigns or initializes `uint64_t Range`.
  **L83 CN**: 对 `uint64_t Range` 进行赋值或初始化。
- **L84 EN**: Assigns or initializes `uint64_t NumCases`.
  **L84 CN**: 对 `uint64_t NumCases` 进行赋值或初始化。
- **L85 EN**: Checks an invariant in debug builds.
  **L85 CN**: 在调试构建中检查一个不变量。
- **L86 EN**: Checks an invariant in debug builds.
  **L86 CN**: 在调试构建中检查一个不变量。
- **L87 EN**: Separates nearby statements for readability.
  **L87 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L88 EN**: Comment documents: `Cheap case: the whole range may be suitable for jump table.`.
  **L88 CN**: 注释说明：`Cheap case: the whole range may be suitable for jump table.`。
- **L89 EN**: Begins a conditional branch.
  **L89 CN**: 开始一个条件分支。
- **L90 EN**: Executes statement `CaseCluster JTCluster;`.
  **L90 CN**: 执行语句 `CaseCluster JTCluster;`。
- **L91 EN**: Begins a conditional branch.
  **L91 CN**: 开始一个条件分支。
- **L92 EN**: Assigns or initializes `Clusters[0]`.
  **L92 CN**: 对 `Clusters[0]` 进行赋值或初始化。
- **L93 EN**: Executes statement `Clusters.resize(1);`.
  **L93 CN**: 执行语句 `Clusters.resize(1);`。
- **L94 EN**: Returns control to the caller.
  **L94 CN**: 将控制流返回给调用者。
- **L95 EN**: Closes the current scope.
  **L95 CN**: 关闭当前作用域。
- **L96 EN**: Closes the current scope.
  **L96 CN**: 关闭当前作用域。
- **L97 EN**: Separates nearby statements for readability.
  **L97 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L98 EN**: Comment documents: `The algorithm below is not suitable for -O0.`.
  **L98 CN**: 注释说明：`The algorithm below is not suitable for -O0.`。
- **L99 EN**: Begins a conditional branch.
  **L99 CN**: 开始一个条件分支。
- **L100 EN**: Returns control to the caller.
  **L100 CN**: 将控制流返回给调用者。

### Lines 101-120

````cpp

  // Split Clusters into minimum number of dense partitions. The algorithm uses
  // the same idea as Kannan & Proebsting "Correction to 'Producing Good Code
  // for the Case Statement'" (1994), but builds the MinPartitions array in
  // reverse order to make it easier to reconstruct the partitions in ascending
  // order. In the choice between two optimal partitionings, it picks the one
  // which yields more jump tables. The algorithm is described in
  // https://arxiv.org/pdf/1910.02351v2

  // MinPartitions[i] is the minimum nbr of partitions of Clusters[i..N-1].
  SmallVector<unsigned, 8> MinPartitions(N);
  // LastElement[i] is the last element of the partition starting at i.
  SmallVector<unsigned, 8> LastElement(N);
  // PartitionsScore[i] is used to break ties when choosing between two
  // partitionings resulting in the same number of partitions.
  SmallVector<unsigned, 8> PartitionsScore(N);
  // For PartitionsScore, a small number of comparisons is considered as good as
  // a jump table and a single comparison is considered better than a jump
  // table.
  enum PartitionScores : unsigned {
````
- **L101 EN**: Separates nearby statements for readability.
  **L101 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L102 EN**: Comment documents: `Split Clusters into minimum number of dense partitions. The algorithm us…`.
  **L102 CN**: 注释说明：`Split Clusters into minimum number of dense partitions. The algorithm us…`。
- **L103 EN**: Comment documents: `the same idea as Kannan & Proebsting "Correction to 'Producing Good Code`.
  **L103 CN**: 注释说明：`the same idea as Kannan & Proebsting "Correction to 'Producing Good Code`。
- **L104 EN**: Comment documents: `for the Case Statement'" (1994), but builds the MinPartitions array in`.
  **L104 CN**: 注释说明：`for the Case Statement'" (1994), but builds the MinPartitions array in`。
- **L105 EN**: Comment documents: `reverse order to make it easier to reconstruct the partitions in ascendi…`.
  **L105 CN**: 注释说明：`reverse order to make it easier to reconstruct the partitions in ascendi…`。
- **L106 EN**: Comment documents: `order. In the choice between two optimal partitionings, it picks the one`.
  **L106 CN**: 注释说明：`order. In the choice between two optimal partitionings, it picks the one`。
- **L107 EN**: Comment documents: `which yields more jump tables. The algorithm is described in`.
  **L107 CN**: 注释说明：`which yields more jump tables. The algorithm is described in`。
- **L108 EN**: Comment documents: `https://arxiv.org/pdf/1910.02351v2`.
  **L108 CN**: 注释说明：`https://arxiv.org/pdf/1910.02351v2`。
- **L109 EN**: Separates nearby statements for readability.
  **L109 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L110 EN**: Comment documents: `MinPartitions[i] is the minimum nbr of partitions of Clusters[i..N-1].`.
  **L110 CN**: 注释说明：`MinPartitions[i] is the minimum nbr of partitions of Clusters[i..N-1].`。
- **L111 EN**: Declares function or method `MinPartitions`.
  **L111 CN**: 声明函数或方法 `MinPartitions`。
- **L112 EN**: Comment documents: `LastElement[i] is the last element of the partition starting at i.`.
  **L112 CN**: 注释说明：`LastElement[i] is the last element of the partition starting at i.`。
- **L113 EN**: Declares function or method `LastElement`.
  **L113 CN**: 声明函数或方法 `LastElement`。
- **L114 EN**: Comment documents: `PartitionsScore[i] is used to break ties when choosing between two`.
  **L114 CN**: 注释说明：`PartitionsScore[i] is used to break ties when choosing between two`。
- **L115 EN**: Comment documents: `partitionings resulting in the same number of partitions.`.
  **L115 CN**: 注释说明：`partitionings resulting in the same number of partitions.`。
- **L116 EN**: Declares function or method `PartitionsScore`.
  **L116 CN**: 声明函数或方法 `PartitionsScore`。
- **L117 EN**: Comment documents: `For PartitionsScore, a small number of comparisons is considered as good…`.
  **L117 CN**: 注释说明：`For PartitionsScore, a small number of comparisons is considered as good…`。
- **L118 EN**: Comment documents: `a jump table and a single comparison is considered better than a jump`.
  **L118 CN**: 注释说明：`a jump table and a single comparison is considered better than a jump`。
- **L119 EN**: Comment documents: `table.`.
  **L119 CN**: 注释说明：`table.`。
- **L120 EN**: Starts an enumeration declaration `enum PartitionScores : unsigned {`.
  **L120 CN**: 开始枚举声明 `enum PartitionScores : unsigned {`。

### Lines 121-140

````cpp
    NoTable = 0,
    Table = 1,
    FewCases = 1,
    SingleCase = 2
  };

  // Base case: There is only one way to partition Clusters[N-1].
  MinPartitions[N - 1] = 1;
  LastElement[N - 1] = N - 1;
  PartitionsScore[N - 1] = PartitionScores::SingleCase;

  // Note: loop indexes are signed to avoid underflow.
  for (int64_t i = N - 2; i >= 0; i--) {
    // Find optimal partitioning of Clusters[i..N-1].
    // Baseline: Put Clusters[i] into a partition on its own.
    MinPartitions[i] = MinPartitions[i + 1] + 1;
    LastElement[i] = i;
    PartitionsScore[i] = PartitionsScore[i + 1] + PartitionScores::SingleCase;

    // Search for a solution that results in fewer partitions.
````
- **L121 EN**: Continues logic with `NoTable = 0,`.
  **L121 CN**: 继续处理逻辑：`NoTable = 0,`。
- **L122 EN**: Continues logic with `Table = 1,`.
  **L122 CN**: 继续处理逻辑：`Table = 1,`。
- **L123 EN**: Continues logic with `FewCases = 1,`.
  **L123 CN**: 继续处理逻辑：`FewCases = 1,`。
- **L124 EN**: Continues logic with `SingleCase = 2`.
  **L124 CN**: 继续处理逻辑：`SingleCase = 2`。
- **L125 EN**: Closes the current scope.
  **L125 CN**: 关闭当前作用域。
- **L126 EN**: Separates nearby statements for readability.
  **L126 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L127 EN**: Comment documents: `Base case: There is only one way to partition Clusters[N-1].`.
  **L127 CN**: 注释说明：`Base case: There is only one way to partition Clusters[N-1].`。
- **L128 EN**: Assigns or initializes `MinPartitions[N - 1]`.
  **L128 CN**: 对 `MinPartitions[N - 1]` 进行赋值或初始化。
- **L129 EN**: Assigns or initializes `LastElement[N - 1]`.
  **L129 CN**: 对 `LastElement[N - 1]` 进行赋值或初始化。
- **L130 EN**: Assigns or initializes `PartitionsScore[N - 1]`.
  **L130 CN**: 对 `PartitionsScore[N - 1]` 进行赋值或初始化。
- **L131 EN**: Separates nearby statements for readability.
  **L131 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L132 EN**: Comment documents: `Note: loop indexes are signed to avoid underflow.`.
  **L132 CN**: 注释说明：`Note: loop indexes are signed to avoid underflow.`。
- **L133 EN**: Starts a loop over a sequence or range.
  **L133 CN**: 开始遍历序列或范围的循环。
- **L134 EN**: Comment documents: `Find optimal partitioning of Clusters[i..N-1].`.
  **L134 CN**: 注释说明：`Find optimal partitioning of Clusters[i..N-1].`。
- **L135 EN**: Comment documents: `Baseline: Put Clusters[i] into a partition on its own.`.
  **L135 CN**: 注释说明：`Baseline: Put Clusters[i] into a partition on its own.`。
- **L136 EN**: Assigns or initializes `MinPartitions[i]`.
  **L136 CN**: 对 `MinPartitions[i]` 进行赋值或初始化。
- **L137 EN**: Assigns or initializes `LastElement[i]`.
  **L137 CN**: 对 `LastElement[i]` 进行赋值或初始化。
- **L138 EN**: Assigns or initializes `PartitionsScore[i]`.
  **L138 CN**: 对 `PartitionsScore[i]` 进行赋值或初始化。
- **L139 EN**: Separates nearby statements for readability.
  **L139 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L140 EN**: Comment documents: `Search for a solution that results in fewer partitions.`.
  **L140 CN**: 注释说明：`Search for a solution that results in fewer partitions.`。

### Lines 141-160

````cpp
    for (int64_t j = N - 1; j > i; j--) {
      // Try building a partition from Clusters[i..j].
      Range = getJumpTableRange(Clusters, i, j);
      NumCases = getJumpTableNumCases(TotalCases, i, j);
      assert(NumCases < UINT64_MAX / 100);
      assert(Range >= NumCases);

      if (TLI->isSuitableForJumpTable(SI, NumCases, Range, PSI, BFI)) {
        unsigned NumPartitions = 1 + (j == N - 1 ? 0 : MinPartitions[j + 1]);
        unsigned Score = j == N - 1 ? 0 : PartitionsScore[j + 1];
        int64_t NumEntries = j - i + 1;

        if (NumEntries == 1)
          Score += PartitionScores::SingleCase;
        else if (NumEntries <= SmallNumberOfEntries)
          Score += PartitionScores::FewCases;
        else if (NumEntries >= MinJumpTableEntries)
          Score += PartitionScores::Table;

        // If this leads to fewer partitions, or to the same number of
````
- **L141 EN**: Starts a loop over a sequence or range.
  **L141 CN**: 开始遍历序列或范围的循环。
- **L142 EN**: Comment documents: `Try building a partition from Clusters[i..j].`.
  **L142 CN**: 注释说明：`Try building a partition from Clusters[i..j].`。
- **L143 EN**: Assigns or initializes `Range`.
  **L143 CN**: 对 `Range` 进行赋值或初始化。
- **L144 EN**: Assigns or initializes `NumCases`.
  **L144 CN**: 对 `NumCases` 进行赋值或初始化。
- **L145 EN**: Checks an invariant in debug builds.
  **L145 CN**: 在调试构建中检查一个不变量。
- **L146 EN**: Checks an invariant in debug builds.
  **L146 CN**: 在调试构建中检查一个不变量。
- **L147 EN**: Separates nearby statements for readability.
  **L147 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L148 EN**: Begins a conditional branch.
  **L148 CN**: 开始一个条件分支。
- **L149 EN**: Assigns or initializes `unsigned NumPartitions`.
  **L149 CN**: 对 `unsigned NumPartitions` 进行赋值或初始化。
- **L150 EN**: Assigns or initializes `unsigned Score`.
  **L150 CN**: 对 `unsigned Score` 进行赋值或初始化。
- **L151 EN**: Assigns or initializes `int64_t NumEntries`.
  **L151 CN**: 对 `int64_t NumEntries` 进行赋值或初始化。
- **L152 EN**: Separates nearby statements for readability.
  **L152 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L153 EN**: Begins a conditional branch.
  **L153 CN**: 开始一个条件分支。
- **L154 EN**: Assigns or initializes `Score +`.
  **L154 CN**: 对 `Score +` 进行赋值或初始化。
- **L155 EN**: Checks an alternate conditional path.
  **L155 CN**: 检查一个备用条件分支。
- **L156 EN**: Assigns or initializes `Score +`.
  **L156 CN**: 对 `Score +` 进行赋值或初始化。
- **L157 EN**: Checks an alternate conditional path.
  **L157 CN**: 检查一个备用条件分支。
- **L158 EN**: Assigns or initializes `Score +`.
  **L158 CN**: 对 `Score +` 进行赋值或初始化。
- **L159 EN**: Separates nearby statements for readability.
  **L159 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L160 EN**: Comment documents: `If this leads to fewer partitions, or to the same number of`.
  **L160 CN**: 注释说明：`If this leads to fewer partitions, or to the same number of`。

### Lines 161-180

````cpp
        // partitions with better score, it is a better partitioning.
        if (NumPartitions < MinPartitions[i] ||
            (NumPartitions == MinPartitions[i] && Score > PartitionsScore[i])) {
          MinPartitions[i] = NumPartitions;
          LastElement[i] = j;
          PartitionsScore[i] = Score;
        }
      }
    }
  }

  // Iterate over the partitions, replacing some with jump tables in-place.
  unsigned DstIndex = 0;
  for (unsigned First = 0, Last; First < N; First = Last + 1) {
    Last = LastElement[First];
    assert(Last >= First);
    assert(DstIndex <= First);
    unsigned NumClusters = Last - First + 1;

    CaseCluster JTCluster;
````
- **L161 EN**: Comment documents: `partitions with better score, it is a better partitioning.`.
  **L161 CN**: 注释说明：`partitions with better score, it is a better partitioning.`。
- **L162 EN**: Begins a conditional branch.
  **L162 CN**: 开始一个条件分支。
- **L163 EN**: Starts block `(NumPartitions == MinPartitions[i] && Score > PartitionsScore[i]))`.
  **L163 CN**: 开始代码块 `(NumPartitions == MinPartitions[i] && Score > PartitionsScore[i]))`。
- **L164 EN**: Assigns or initializes `MinPartitions[i]`.
  **L164 CN**: 对 `MinPartitions[i]` 进行赋值或初始化。
- **L165 EN**: Assigns or initializes `LastElement[i]`.
  **L165 CN**: 对 `LastElement[i]` 进行赋值或初始化。
- **L166 EN**: Assigns or initializes `PartitionsScore[i]`.
  **L166 CN**: 对 `PartitionsScore[i]` 进行赋值或初始化。
- **L167 EN**: Closes the current scope.
  **L167 CN**: 关闭当前作用域。
- **L168 EN**: Closes the current scope.
  **L168 CN**: 关闭当前作用域。
- **L169 EN**: Closes the current scope.
  **L169 CN**: 关闭当前作用域。
- **L170 EN**: Closes the current scope.
  **L170 CN**: 关闭当前作用域。
- **L171 EN**: Separates nearby statements for readability.
  **L171 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L172 EN**: Comment documents: `Iterate over the partitions, replacing some with jump tables in-place.`.
  **L172 CN**: 注释说明：`Iterate over the partitions, replacing some with jump tables in-place.`。
- **L173 EN**: Assigns or initializes `unsigned DstIndex`.
  **L173 CN**: 对 `unsigned DstIndex` 进行赋值或初始化。
- **L174 EN**: Starts a loop over a sequence or range.
  **L174 CN**: 开始遍历序列或范围的循环。
- **L175 EN**: Assigns or initializes `Last`.
  **L175 CN**: 对 `Last` 进行赋值或初始化。
- **L176 EN**: Checks an invariant in debug builds.
  **L176 CN**: 在调试构建中检查一个不变量。
- **L177 EN**: Checks an invariant in debug builds.
  **L177 CN**: 在调试构建中检查一个不变量。
- **L178 EN**: Assigns or initializes `unsigned NumClusters`.
  **L178 CN**: 对 `unsigned NumClusters` 进行赋值或初始化。
- **L179 EN**: Separates nearby statements for readability.
  **L179 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L180 EN**: Executes statement `CaseCluster JTCluster;`.
  **L180 CN**: 执行语句 `CaseCluster JTCluster;`。

### Lines 181-200

````cpp
    if (NumClusters >= MinJumpTableEntries &&
        buildJumpTable(Clusters, First, Last, SI, SL, DefaultMBB, JTCluster)) {
      Clusters[DstIndex++] = JTCluster;
    } else {
      for (unsigned I = First; I <= Last; ++I)
        std::memmove(&Clusters[DstIndex++], &Clusters[I], sizeof(Clusters[I]));
    }
  }
  Clusters.resize(DstIndex);
}

bool SwitchCG::SwitchLowering::buildJumpTable(const CaseClusterVector &Clusters,
                                              unsigned First, unsigned Last,
                                              const SwitchInst *SI,
                                              const std::optional<SDLoc> &SL,
                                              MachineBasicBlock *DefaultMBB,
                                              CaseCluster &JTCluster) {
  assert(First <= Last);

  auto Prob = BranchProbability::getZero();
````
- **L181 EN**: Begins a conditional branch.
  **L181 CN**: 开始一个条件分支。
- **L182 EN**: Starts block `buildJumpTable(Clusters, First, Last, SI, SL, DefaultMBB, JTCluster))`.
  **L182 CN**: 开始代码块 `buildJumpTable(Clusters, First, Last, SI, SL, DefaultMBB, JTCluster))`。
- **L183 EN**: Assigns or initializes `Clusters[DstIndex++]`.
  **L183 CN**: 对 `Clusters[DstIndex++]` 进行赋值或初始化。
- **L184 EN**: Starts block `} else`.
  **L184 CN**: 开始代码块 `} else`。
- **L185 EN**: Starts a loop over a sequence or range.
  **L185 CN**: 开始遍历序列或范围的循环。
- **L186 EN**: Declares function or method `memmove`.
  **L186 CN**: 声明函数或方法 `memmove`。
- **L187 EN**: Closes the current scope.
  **L187 CN**: 关闭当前作用域。
- **L188 EN**: Closes the current scope.
  **L188 CN**: 关闭当前作用域。
- **L189 EN**: Executes statement `Clusters.resize(DstIndex);`.
  **L189 CN**: 执行语句 `Clusters.resize(DstIndex);`。
- **L190 EN**: Closes the current scope.
  **L190 CN**: 关闭当前作用域。
- **L191 EN**: Separates nearby statements for readability.
  **L191 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L192 EN**: Provides part of the signature for `buildJumpTable`.
  **L192 CN**: 给出 `buildJumpTable` 的一部分签名。
- **L193 EN**: Continues logic with `unsigned First, unsigned Last,`.
  **L193 CN**: 继续处理逻辑：`unsigned First, unsigned Last,`。
- **L194 EN**: Continues logic with `const SwitchInst *SI,`.
  **L194 CN**: 继续处理逻辑：`const SwitchInst *SI,`。
- **L195 EN**: Continues logic with `const std::optional<SDLoc> &SL,`.
  **L195 CN**: 继续处理逻辑：`const std::optional<SDLoc> &SL,`。
- **L196 EN**: Continues logic with `MachineBasicBlock *DefaultMBB,`.
  **L196 CN**: 继续处理逻辑：`MachineBasicBlock *DefaultMBB,`。
- **L197 EN**: Starts block `CaseCluster &JTCluster)`.
  **L197 CN**: 开始代码块 `CaseCluster &JTCluster)`。
- **L198 EN**: Checks an invariant in debug builds.
  **L198 CN**: 在调试构建中检查一个不变量。
- **L199 EN**: Separates nearby statements for readability.
  **L199 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L200 EN**: Declares function or method `getZero`.
  **L200 CN**: 声明函数或方法 `getZero`。

### Lines 201-220

````cpp
  std::vector<MachineBasicBlock*> Table;
  DenseMap<MachineBasicBlock*, BranchProbability> JTProbs;

  // Initialize probabilities in JTProbs.
  for (unsigned I = First; I <= Last; ++I)
    JTProbs[Clusters[I].MBB] = BranchProbability::getZero();

  DenseMap<const BasicBlock *, unsigned int> DestMap;
  for (unsigned I = First; I <= Last; ++I) {
    assert(Clusters[I].Kind == CC_Range);
    Prob += Clusters[I].Prob;
    const APInt &Low = Clusters[I].Low->getValue();
    const APInt &High = Clusters[I].High->getValue();
    unsigned int NumCmp = (Low == High) ? 1 : 2;
    const BasicBlock *BB = Clusters[I].MBB->getBasicBlock();
    DestMap[BB] += NumCmp;

    if (I != First) {
      // Fill the gap between this and the previous cluster.
      const APInt &PreviousHigh = Clusters[I - 1].High->getValue();
````
- **L201 EN**: Executes statement `std::vector<MachineBasicBlock*> Table;`.
  **L201 CN**: 执行语句 `std::vector<MachineBasicBlock*> Table;`。
- **L202 EN**: Executes statement `DenseMap<MachineBasicBlock*, BranchProbability> JTProbs;`.
  **L202 CN**: 执行语句 `DenseMap<MachineBasicBlock*, BranchProbability> JTProbs;`。
- **L203 EN**: Separates nearby statements for readability.
  **L203 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L204 EN**: Comment documents: `Initialize probabilities in JTProbs.`.
  **L204 CN**: 注释说明：`Initialize probabilities in JTProbs.`。
- **L205 EN**: Starts a loop over a sequence or range.
  **L205 CN**: 开始遍历序列或范围的循环。
- **L206 EN**: Declares function or method `getZero`.
  **L206 CN**: 声明函数或方法 `getZero`。
- **L207 EN**: Separates nearby statements for readability.
  **L207 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L208 EN**: Executes statement `DenseMap<const BasicBlock *, unsigned int> DestMap;`.
  **L208 CN**: 执行语句 `DenseMap<const BasicBlock *, unsigned int> DestMap;`。
- **L209 EN**: Starts a loop over a sequence or range.
  **L209 CN**: 开始遍历序列或范围的循环。
- **L210 EN**: Checks an invariant in debug builds.
  **L210 CN**: 在调试构建中检查一个不变量。
- **L211 EN**: Assigns or initializes `Prob +`.
  **L211 CN**: 对 `Prob +` 进行赋值或初始化。
- **L212 EN**: Assigns or initializes `const APInt &Low`.
  **L212 CN**: 对 `const APInt &Low` 进行赋值或初始化。
- **L213 EN**: Assigns or initializes `const APInt &High`.
  **L213 CN**: 对 `const APInt &High` 进行赋值或初始化。
- **L214 EN**: Assigns or initializes `unsigned int NumCmp`.
  **L214 CN**: 对 `unsigned int NumCmp` 进行赋值或初始化。
- **L215 EN**: Assigns or initializes `const BasicBlock *BB`.
  **L215 CN**: 对 `const BasicBlock *BB` 进行赋值或初始化。
- **L216 EN**: Assigns or initializes `DestMap[BB] +`.
  **L216 CN**: 对 `DestMap[BB] +` 进行赋值或初始化。
- **L217 EN**: Separates nearby statements for readability.
  **L217 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L218 EN**: Begins a conditional branch.
  **L218 CN**: 开始一个条件分支。
- **L219 EN**: Comment documents: `Fill the gap between this and the previous cluster.`.
  **L219 CN**: 注释说明：`Fill the gap between this and the previous cluster.`。
- **L220 EN**: Assigns or initializes `const APInt &PreviousHigh`.
  **L220 CN**: 对 `const APInt &PreviousHigh` 进行赋值或初始化。

### Lines 221-240

````cpp
      assert(PreviousHigh.slt(Low));
      uint64_t Gap = (Low - PreviousHigh).getLimitedValue() - 1;
      for (uint64_t J = 0; J < Gap; J++)
        Table.push_back(DefaultMBB);
    }
    uint64_t ClusterSize = (High - Low).getLimitedValue() + 1;
    for (uint64_t J = 0; J < ClusterSize; ++J)
      Table.push_back(Clusters[I].MBB);
    JTProbs[Clusters[I].MBB] += Clusters[I].Prob;
  }

  if (TLI->isSuitableForBitTests(DestMap, Clusters[First].Low->getValue(),
                                 Clusters[Last].High->getValue(), *DL)) {
    // Clusters[First..Last] should be lowered as bit tests instead.
    return false;
  }

  // Create the MBB that will load from and jump through the table.
  // Note: We create it here, but it's not inserted into the function yet.
  MachineFunction *CurMF = FuncInfo.MF;
````
- **L221 EN**: Checks an invariant in debug builds.
  **L221 CN**: 在调试构建中检查一个不变量。
- **L222 EN**: Assigns or initializes `uint64_t Gap`.
  **L222 CN**: 对 `uint64_t Gap` 进行赋值或初始化。
- **L223 EN**: Starts a loop over a sequence or range.
  **L223 CN**: 开始遍历序列或范围的循环。
- **L224 EN**: Executes statement `Table.push_back(DefaultMBB);`.
  **L224 CN**: 执行语句 `Table.push_back(DefaultMBB);`。
- **L225 EN**: Closes the current scope.
  **L225 CN**: 关闭当前作用域。
- **L226 EN**: Assigns or initializes `uint64_t ClusterSize`.
  **L226 CN**: 对 `uint64_t ClusterSize` 进行赋值或初始化。
- **L227 EN**: Starts a loop over a sequence or range.
  **L227 CN**: 开始遍历序列或范围的循环。
- **L228 EN**: Executes statement `Table.push_back(Clusters[I].MBB);`.
  **L228 CN**: 执行语句 `Table.push_back(Clusters[I].MBB);`。
- **L229 EN**: Assigns or initializes `JTProbs[Clusters[I].MBB] +`.
  **L229 CN**: 对 `JTProbs[Clusters[I].MBB] +` 进行赋值或初始化。
- **L230 EN**: Closes the current scope.
  **L230 CN**: 关闭当前作用域。
- **L231 EN**: Separates nearby statements for readability.
  **L231 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L232 EN**: Begins a conditional branch.
  **L232 CN**: 开始一个条件分支。
- **L233 EN**: Starts block `Clusters[Last].High->getValue(), *DL))`.
  **L233 CN**: 开始代码块 `Clusters[Last].High->getValue(), *DL))`。
- **L234 EN**: Comment documents: `Clusters[First..Last] should be lowered as bit tests instead.`.
  **L234 CN**: 注释说明：`Clusters[First..Last] should be lowered as bit tests instead.`。
- **L235 EN**: Returns `false` to the caller.
  **L235 CN**: 向调用者返回 `false`。
- **L236 EN**: Closes the current scope.
  **L236 CN**: 关闭当前作用域。
- **L237 EN**: Separates nearby statements for readability.
  **L237 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L238 EN**: Comment documents: `Create the MBB that will load from and jump through the table.`.
  **L238 CN**: 注释说明：`Create the MBB that will load from and jump through the table.`。
- **L239 EN**: Comment documents: `Note: We create it here, but it's not inserted into the function yet.`.
  **L239 CN**: 注释说明：`Note: We create it here, but it's not inserted into the function yet.`。
- **L240 EN**: Assigns or initializes `MachineFunction *CurMF`.
  **L240 CN**: 对 `MachineFunction *CurMF` 进行赋值或初始化。

### Lines 241-260

````cpp
  MachineBasicBlock *JumpTableMBB =
      CurMF->CreateMachineBasicBlock(SI->getParent());

  // Add successors. Note: use table order for determinism.
  SmallPtrSet<MachineBasicBlock *, 8> Done;
  for (MachineBasicBlock *Succ : Table) {
    if (Done.count(Succ))
      continue;
    addSuccessorWithProb(JumpTableMBB, Succ, JTProbs[Succ]);
    Done.insert(Succ);
  }
  JumpTableMBB->normalizeSuccProbs();

  unsigned JTI = CurMF->getOrCreateJumpTableInfo(TLI->getJumpTableEncoding())
                     ->createJumpTableIndex(Table);

  // Set up the jump table info.
  JumpTable JT(Register(), JTI, JumpTableMBB, nullptr, SL);
  JumpTableHeader JTH(Clusters[First].Low->getValue(),
                      Clusters[Last].High->getValue(), SI->getCondition(),
````
- **L241 EN**: Continues logic with `MachineBasicBlock *JumpTableMBB =`.
  **L241 CN**: 继续处理逻辑：`MachineBasicBlock *JumpTableMBB =`。
- **L242 EN**: Executes statement `CurMF->CreateMachineBasicBlock(SI->getParent());`.
  **L242 CN**: 执行语句 `CurMF->CreateMachineBasicBlock(SI->getParent());`。
- **L243 EN**: Separates nearby statements for readability.
  **L243 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L244 EN**: Comment documents: `Add successors. Note: use table order for determinism.`.
  **L244 CN**: 注释说明：`Add successors. Note: use table order for determinism.`。
- **L245 EN**: Executes statement `SmallPtrSet<MachineBasicBlock *, 8> Done;`.
  **L245 CN**: 执行语句 `SmallPtrSet<MachineBasicBlock *, 8> Done;`。
- **L246 EN**: Starts a loop over a sequence or range.
  **L246 CN**: 开始遍历序列或范围的循环。
- **L247 EN**: Begins a conditional branch.
  **L247 CN**: 开始一个条件分支。
- **L248 EN**: Skips to the next loop iteration.
  **L248 CN**: 跳到下一次循环迭代。
- **L249 EN**: Executes statement `addSuccessorWithProb(JumpTableMBB, Succ, JTProbs[Succ]);`.
  **L249 CN**: 执行语句 `addSuccessorWithProb(JumpTableMBB, Succ, JTProbs[Succ]);`。
- **L250 EN**: Executes statement `Done.insert(Succ);`.
  **L250 CN**: 执行语句 `Done.insert(Succ);`。
- **L251 EN**: Closes the current scope.
  **L251 CN**: 关闭当前作用域。
- **L252 EN**: Executes statement `JumpTableMBB->normalizeSuccProbs();`.
  **L252 CN**: 执行语句 `JumpTableMBB->normalizeSuccProbs();`。
- **L253 EN**: Separates nearby statements for readability.
  **L253 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L254 EN**: Continues logic with `unsigned JTI = CurMF->getOrCreateJumpTableInfo(TLI->getJumpTableEncoding…`.
  **L254 CN**: 继续处理逻辑：`unsigned JTI = CurMF->getOrCreateJumpTableInfo(TLI->getJumpTableEncoding…`。
- **L255 EN**: Executes statement `->createJumpTableIndex(Table);`.
  **L255 CN**: 执行语句 `->createJumpTableIndex(Table);`。
- **L256 EN**: Separates nearby statements for readability.
  **L256 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L257 EN**: Comment documents: `Set up the jump table info.`.
  **L257 CN**: 注释说明：`Set up the jump table info.`。
- **L258 EN**: Declares function or method `JT`.
  **L258 CN**: 声明函数或方法 `JT`。
- **L259 EN**: Provides part of the signature for `JTH`.
  **L259 CN**: 给出 `JTH` 的一部分签名。
- **L260 EN**: Continues logic with `Clusters[Last].High->getValue(), SI->getCondition(),`.
  **L260 CN**: 继续处理逻辑：`Clusters[Last].High->getValue(), SI->getCondition(),`。

### Lines 261-280

````cpp
                      nullptr, false);
  JTCases.emplace_back(std::move(JTH), std::move(JT));

  JTCluster = CaseCluster::jumpTable(Clusters[First].Low, Clusters[Last].High,
                                     JTCases.size() - 1, Prob);
  return true;
}

void SwitchCG::SwitchLowering::findBitTestClusters(CaseClusterVector &Clusters,
                                                   const SwitchInst *SI) {
  // Partition Clusters into as few subsets as possible, where each subset has a
  // range that fits in a machine word and has <= 3 unique destinations.

#ifndef NDEBUG
  // Clusters must be sorted and contain Range or JumpTable clusters.
  assert(!Clusters.empty());
  assert(Clusters[0].Kind == CC_Range || Clusters[0].Kind == CC_JumpTable);
  for (const CaseCluster &C : Clusters)
    assert(C.Kind == CC_Range || C.Kind == CC_JumpTable);
  for (unsigned i = 1; i < Clusters.size(); ++i)
````
- **L261 EN**: Executes statement `nullptr, false);`.
  **L261 CN**: 执行语句 `nullptr, false);`。
- **L262 EN**: Declares function or method `emplace_back`.
  **L262 CN**: 声明函数或方法 `emplace_back`。
- **L263 EN**: Separates nearby statements for readability.
  **L263 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L264 EN**: Provides part of the signature for `jumpTable`.
  **L264 CN**: 给出 `jumpTable` 的一部分签名。
- **L265 EN**: Executes statement `JTCases.size() - 1, Prob);`.
  **L265 CN**: 执行语句 `JTCases.size() - 1, Prob);`。
- **L266 EN**: Returns `true` to the caller.
  **L266 CN**: 向调用者返回 `true`。
- **L267 EN**: Closes the current scope.
  **L267 CN**: 关闭当前作用域。
- **L268 EN**: Separates nearby statements for readability.
  **L268 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L269 EN**: Provides part of the signature for `findBitTestClusters`.
  **L269 CN**: 给出 `findBitTestClusters` 的一部分签名。
- **L270 EN**: Starts block `const SwitchInst *SI)`.
  **L270 CN**: 开始代码块 `const SwitchInst *SI)`。
- **L271 EN**: Comment documents: `Partition Clusters into as few subsets as possible, where each subset ha…`.
  **L271 CN**: 注释说明：`Partition Clusters into as few subsets as possible, where each subset ha…`。
- **L272 EN**: Comment documents: `range that fits in a machine word and has <= 3 unique destinations.`.
  **L272 CN**: 注释说明：`range that fits in a machine word and has <= 3 unique destinations.`。
- **L273 EN**: Separates nearby statements for readability.
  **L273 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L274 EN**: Starts a preprocessor conditional block.
  **L274 CN**: 开始一个预处理条件块。
- **L275 EN**: Comment documents: `Clusters must be sorted and contain Range or JumpTable clusters.`.
  **L275 CN**: 注释说明：`Clusters must be sorted and contain Range or JumpTable clusters.`。
- **L276 EN**: Checks an invariant in debug builds.
  **L276 CN**: 在调试构建中检查一个不变量。
- **L277 EN**: Checks an invariant in debug builds.
  **L277 CN**: 在调试构建中检查一个不变量。
- **L278 EN**: Starts a loop over a sequence or range.
  **L278 CN**: 开始遍历序列或范围的循环。
- **L279 EN**: Checks an invariant in debug builds.
  **L279 CN**: 在调试构建中检查一个不变量。
- **L280 EN**: Starts a loop over a sequence or range.
  **L280 CN**: 开始遍历序列或范围的循环。

### Lines 281-300

````cpp
    assert(Clusters[i-1].High->getValue().slt(Clusters[i].Low->getValue()));
#endif

  // The algorithm below is not suitable for -O0.
  if (TM->getOptLevel() == CodeGenOptLevel::None)
    return;

  // If target does not have legal shift left, do not emit bit tests at all.
  EVT PTy = TLI->getPointerTy(*DL);
  if (!TLI->isOperationLegal(ISD::SHL, PTy))
    return;

  int BitWidth = PTy.getSizeInBits();
  const int64_t N = Clusters.size();

  // MinPartitions[i] is the minimum nbr of partitions of Clusters[i..N-1].
  SmallVector<unsigned, 8> MinPartitions(N);
  // LastElement[i] is the last element of the partition starting at i.
  SmallVector<unsigned, 8> LastElement(N);

````
- **L281 EN**: Checks an invariant in debug builds.
  **L281 CN**: 在调试构建中检查一个不变量。
- **L282 EN**: Ends the current preprocessor conditional block.
  **L282 CN**: 结束当前的预处理条件块。
- **L283 EN**: Separates nearby statements for readability.
  **L283 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L284 EN**: Comment documents: `The algorithm below is not suitable for -O0.`.
  **L284 CN**: 注释说明：`The algorithm below is not suitable for -O0.`。
- **L285 EN**: Begins a conditional branch.
  **L285 CN**: 开始一个条件分支。
- **L286 EN**: Returns control to the caller.
  **L286 CN**: 将控制流返回给调用者。
- **L287 EN**: Separates nearby statements for readability.
  **L287 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L288 EN**: Comment documents: `If target does not have legal shift left, do not emit bit tests at all.`.
  **L288 CN**: 注释说明：`If target does not have legal shift left, do not emit bit tests at all.`。
- **L289 EN**: Assigns or initializes `EVT PTy`.
  **L289 CN**: 对 `EVT PTy` 进行赋值或初始化。
- **L290 EN**: Begins a conditional branch.
  **L290 CN**: 开始一个条件分支。
- **L291 EN**: Returns control to the caller.
  **L291 CN**: 将控制流返回给调用者。
- **L292 EN**: Separates nearby statements for readability.
  **L292 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L293 EN**: Assigns or initializes `int BitWidth`.
  **L293 CN**: 对 `int BitWidth` 进行赋值或初始化。
- **L294 EN**: Assigns or initializes `const int64_t N`.
  **L294 CN**: 对 `const int64_t N` 进行赋值或初始化。
- **L295 EN**: Separates nearby statements for readability.
  **L295 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L296 EN**: Comment documents: `MinPartitions[i] is the minimum nbr of partitions of Clusters[i..N-1].`.
  **L296 CN**: 注释说明：`MinPartitions[i] is the minimum nbr of partitions of Clusters[i..N-1].`。
- **L297 EN**: Declares function or method `MinPartitions`.
  **L297 CN**: 声明函数或方法 `MinPartitions`。
- **L298 EN**: Comment documents: `LastElement[i] is the last element of the partition starting at i.`.
  **L298 CN**: 注释说明：`LastElement[i] is the last element of the partition starting at i.`。
- **L299 EN**: Declares function or method `LastElement`.
  **L299 CN**: 声明函数或方法 `LastElement`。
- **L300 EN**: Separates nearby statements for readability.
  **L300 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 301-320

````cpp
  // FIXME: This might not be the best algorithm for finding bit test clusters.

  // Base case: There is only one way to partition Clusters[N-1].
  MinPartitions[N - 1] = 1;
  LastElement[N - 1] = N - 1;

  // Note: loop indexes are signed to avoid underflow.
  for (int64_t i = N - 2; i >= 0; --i) {
    // Find optimal partitioning of Clusters[i..N-1].
    // Baseline: Put Clusters[i] into a partition on its own.
    MinPartitions[i] = MinPartitions[i + 1] + 1;
    LastElement[i] = i;

    // Search for a solution that results in fewer partitions.
    // Note: the search is limited by BitWidth, reducing time complexity.
    for (int64_t j = std::min(N - 1, i + BitWidth - 1); j > i; --j) {
      // Try building a partition from Clusters[i..j].

      // Check the range.
      if (!TLI->rangeFitsInWord(Clusters[i].Low->getValue(),
````
- **L301 EN**: Comment documents: `FIXME: This might not be the best algorithm for finding bit test cluster…`.
  **L301 CN**: 注释说明：`FIXME: This might not be the best algorithm for finding bit test cluster…`。
- **L302 EN**: Separates nearby statements for readability.
  **L302 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L303 EN**: Comment documents: `Base case: There is only one way to partition Clusters[N-1].`.
  **L303 CN**: 注释说明：`Base case: There is only one way to partition Clusters[N-1].`。
- **L304 EN**: Assigns or initializes `MinPartitions[N - 1]`.
  **L304 CN**: 对 `MinPartitions[N - 1]` 进行赋值或初始化。
- **L305 EN**: Assigns or initializes `LastElement[N - 1]`.
  **L305 CN**: 对 `LastElement[N - 1]` 进行赋值或初始化。
- **L306 EN**: Separates nearby statements for readability.
  **L306 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L307 EN**: Comment documents: `Note: loop indexes are signed to avoid underflow.`.
  **L307 CN**: 注释说明：`Note: loop indexes are signed to avoid underflow.`。
- **L308 EN**: Starts a loop over a sequence or range.
  **L308 CN**: 开始遍历序列或范围的循环。
- **L309 EN**: Comment documents: `Find optimal partitioning of Clusters[i..N-1].`.
  **L309 CN**: 注释说明：`Find optimal partitioning of Clusters[i..N-1].`。
- **L310 EN**: Comment documents: `Baseline: Put Clusters[i] into a partition on its own.`.
  **L310 CN**: 注释说明：`Baseline: Put Clusters[i] into a partition on its own.`。
- **L311 EN**: Assigns or initializes `MinPartitions[i]`.
  **L311 CN**: 对 `MinPartitions[i]` 进行赋值或初始化。
- **L312 EN**: Assigns or initializes `LastElement[i]`.
  **L312 CN**: 对 `LastElement[i]` 进行赋值或初始化。
- **L313 EN**: Separates nearby statements for readability.
  **L313 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L314 EN**: Comment documents: `Search for a solution that results in fewer partitions.`.
  **L314 CN**: 注释说明：`Search for a solution that results in fewer partitions.`。
- **L315 EN**: Comment documents: `Note: the search is limited by BitWidth, reducing time complexity.`.
  **L315 CN**: 注释说明：`Note: the search is limited by BitWidth, reducing time complexity.`。
- **L316 EN**: Starts a loop over a sequence or range.
  **L316 CN**: 开始遍历序列或范围的循环。
- **L317 EN**: Comment documents: `Try building a partition from Clusters[i..j].`.
  **L317 CN**: 注释说明：`Try building a partition from Clusters[i..j].`。
- **L318 EN**: Separates nearby statements for readability.
  **L318 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L319 EN**: Comment documents: `Check the range.`.
  **L319 CN**: 注释说明：`Check the range.`。
- **L320 EN**: Begins a conditional branch.
  **L320 CN**: 开始一个条件分支。

### Lines 321-340

````cpp
                                Clusters[j].High->getValue(), *DL))
        continue;

      // Check nbr of destinations and cluster types.
      // FIXME: This works, but doesn't seem very efficient.
      bool RangesOnly = true;
      BitVector Dests(FuncInfo.MF->getNumBlockIDs());
      for (int64_t k = i; k <= j; k++) {
        if (Clusters[k].Kind != CC_Range) {
          RangesOnly = false;
          break;
        }
        Dests.set(Clusters[k].MBB->getNumber());
      }
      if (!RangesOnly || Dests.count() > 3)
        break;

      // Check if it's a better partition.
      unsigned NumPartitions = 1 + (j == N - 1 ? 0 : MinPartitions[j + 1]);
      if (NumPartitions < MinPartitions[i]) {
````
- **L321 EN**: Continues logic with `Clusters[j].High->getValue(), *DL))`.
  **L321 CN**: 继续处理逻辑：`Clusters[j].High->getValue(), *DL))`。
- **L322 EN**: Skips to the next loop iteration.
  **L322 CN**: 跳到下一次循环迭代。
- **L323 EN**: Separates nearby statements for readability.
  **L323 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L324 EN**: Comment documents: `Check nbr of destinations and cluster types.`.
  **L324 CN**: 注释说明：`Check nbr of destinations and cluster types.`。
- **L325 EN**: Comment documents: `FIXME: This works, but doesn't seem very efficient.`.
  **L325 CN**: 注释说明：`FIXME: This works, but doesn't seem very efficient.`。
- **L326 EN**: Assigns or initializes `bool RangesOnly`.
  **L326 CN**: 对 `bool RangesOnly` 进行赋值或初始化。
- **L327 EN**: Declares function or method `Dests`.
  **L327 CN**: 声明函数或方法 `Dests`。
- **L328 EN**: Starts a loop over a sequence or range.
  **L328 CN**: 开始遍历序列或范围的循环。
- **L329 EN**: Begins a conditional branch.
  **L329 CN**: 开始一个条件分支。
- **L330 EN**: Assigns or initializes `RangesOnly`.
  **L330 CN**: 对 `RangesOnly` 进行赋值或初始化。
- **L331 EN**: Breaks out of the current control-flow construct.
  **L331 CN**: 跳出当前控制流结构。
- **L332 EN**: Closes the current scope.
  **L332 CN**: 关闭当前作用域。
- **L333 EN**: Executes statement `Dests.set(Clusters[k].MBB->getNumber());`.
  **L333 CN**: 执行语句 `Dests.set(Clusters[k].MBB->getNumber());`。
- **L334 EN**: Closes the current scope.
  **L334 CN**: 关闭当前作用域。
- **L335 EN**: Begins a conditional branch.
  **L335 CN**: 开始一个条件分支。
- **L336 EN**: Breaks out of the current control-flow construct.
  **L336 CN**: 跳出当前控制流结构。
- **L337 EN**: Separates nearby statements for readability.
  **L337 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L338 EN**: Comment documents: `Check if it's a better partition.`.
  **L338 CN**: 注释说明：`Check if it's a better partition.`。
- **L339 EN**: Assigns or initializes `unsigned NumPartitions`.
  **L339 CN**: 对 `unsigned NumPartitions` 进行赋值或初始化。
- **L340 EN**: Begins a conditional branch.
  **L340 CN**: 开始一个条件分支。

### Lines 341-360

````cpp
        // Found a better partition.
        MinPartitions[i] = NumPartitions;
        LastElement[i] = j;
      }
    }
  }

  // Iterate over the partitions, replacing with bit-test clusters in-place.
  unsigned DstIndex = 0;
  for (unsigned First = 0, Last; First < N; First = Last + 1) {
    Last = LastElement[First];
    assert(First <= Last);
    assert(DstIndex <= First);

    CaseCluster BitTestCluster;
    if (buildBitTests(Clusters, First, Last, SI, BitTestCluster)) {
      Clusters[DstIndex++] = BitTestCluster;
    } else {
      size_t NumClusters = Last - First + 1;
      std::memmove(&Clusters[DstIndex], &Clusters[First],
````
- **L341 EN**: Comment documents: `Found a better partition.`.
  **L341 CN**: 注释说明：`Found a better partition.`。
- **L342 EN**: Assigns or initializes `MinPartitions[i]`.
  **L342 CN**: 对 `MinPartitions[i]` 进行赋值或初始化。
- **L343 EN**: Assigns or initializes `LastElement[i]`.
  **L343 CN**: 对 `LastElement[i]` 进行赋值或初始化。
- **L344 EN**: Closes the current scope.
  **L344 CN**: 关闭当前作用域。
- **L345 EN**: Closes the current scope.
  **L345 CN**: 关闭当前作用域。
- **L346 EN**: Closes the current scope.
  **L346 CN**: 关闭当前作用域。
- **L347 EN**: Separates nearby statements for readability.
  **L347 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L348 EN**: Comment documents: `Iterate over the partitions, replacing with bit-test clusters in-place.`.
  **L348 CN**: 注释说明：`Iterate over the partitions, replacing with bit-test clusters in-place.`。
- **L349 EN**: Assigns or initializes `unsigned DstIndex`.
  **L349 CN**: 对 `unsigned DstIndex` 进行赋值或初始化。
- **L350 EN**: Starts a loop over a sequence or range.
  **L350 CN**: 开始遍历序列或范围的循环。
- **L351 EN**: Assigns or initializes `Last`.
  **L351 CN**: 对 `Last` 进行赋值或初始化。
- **L352 EN**: Checks an invariant in debug builds.
  **L352 CN**: 在调试构建中检查一个不变量。
- **L353 EN**: Checks an invariant in debug builds.
  **L353 CN**: 在调试构建中检查一个不变量。
- **L354 EN**: Separates nearby statements for readability.
  **L354 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L355 EN**: Executes statement `CaseCluster BitTestCluster;`.
  **L355 CN**: 执行语句 `CaseCluster BitTestCluster;`。
- **L356 EN**: Begins a conditional branch.
  **L356 CN**: 开始一个条件分支。
- **L357 EN**: Assigns or initializes `Clusters[DstIndex++]`.
  **L357 CN**: 对 `Clusters[DstIndex++]` 进行赋值或初始化。
- **L358 EN**: Starts block `} else`.
  **L358 CN**: 开始代码块 `} else`。
- **L359 EN**: Assigns or initializes `size_t NumClusters`.
  **L359 CN**: 对 `size_t NumClusters` 进行赋值或初始化。
- **L360 EN**: Provides part of the signature for `memmove`.
  **L360 CN**: 给出 `memmove` 的一部分签名。

### Lines 361-380

````cpp
                   sizeof(Clusters[0]) * NumClusters);
      DstIndex += NumClusters;
    }
  }
  Clusters.resize(DstIndex);
}

bool SwitchCG::SwitchLowering::buildBitTests(CaseClusterVector &Clusters,
                                             unsigned First, unsigned Last,
                                             const SwitchInst *SI,
                                             CaseCluster &BTCluster) {
  assert(First <= Last);
  if (First == Last)
    return false;

  DenseMap<const BasicBlock *, unsigned int> DestMap;
  for (int64_t I = First; I <= Last; ++I) {
    assert(Clusters[I].Kind == CC_Range);
    unsigned NumCmp = (Clusters[I].Low == Clusters[I].High) ? 1 : 2;
    const BasicBlock *BB = Clusters[I].MBB->getBasicBlock();
````
- **L361 EN**: Executes statement `sizeof(Clusters[0]) * NumClusters);`.
  **L361 CN**: 执行语句 `sizeof(Clusters[0]) * NumClusters);`。
- **L362 EN**: Assigns or initializes `DstIndex +`.
  **L362 CN**: 对 `DstIndex +` 进行赋值或初始化。
- **L363 EN**: Closes the current scope.
  **L363 CN**: 关闭当前作用域。
- **L364 EN**: Closes the current scope.
  **L364 CN**: 关闭当前作用域。
- **L365 EN**: Executes statement `Clusters.resize(DstIndex);`.
  **L365 CN**: 执行语句 `Clusters.resize(DstIndex);`。
- **L366 EN**: Closes the current scope.
  **L366 CN**: 关闭当前作用域。
- **L367 EN**: Separates nearby statements for readability.
  **L367 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L368 EN**: Provides part of the signature for `buildBitTests`.
  **L368 CN**: 给出 `buildBitTests` 的一部分签名。
- **L369 EN**: Continues logic with `unsigned First, unsigned Last,`.
  **L369 CN**: 继续处理逻辑：`unsigned First, unsigned Last,`。
- **L370 EN**: Continues logic with `const SwitchInst *SI,`.
  **L370 CN**: 继续处理逻辑：`const SwitchInst *SI,`。
- **L371 EN**: Starts block `CaseCluster &BTCluster)`.
  **L371 CN**: 开始代码块 `CaseCluster &BTCluster)`。
- **L372 EN**: Checks an invariant in debug builds.
  **L372 CN**: 在调试构建中检查一个不变量。
- **L373 EN**: Begins a conditional branch.
  **L373 CN**: 开始一个条件分支。
- **L374 EN**: Returns `false` to the caller.
  **L374 CN**: 向调用者返回 `false`。
- **L375 EN**: Separates nearby statements for readability.
  **L375 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L376 EN**: Executes statement `DenseMap<const BasicBlock *, unsigned int> DestMap;`.
  **L376 CN**: 执行语句 `DenseMap<const BasicBlock *, unsigned int> DestMap;`。
- **L377 EN**: Starts a loop over a sequence or range.
  **L377 CN**: 开始遍历序列或范围的循环。
- **L378 EN**: Checks an invariant in debug builds.
  **L378 CN**: 在调试构建中检查一个不变量。
- **L379 EN**: Assigns or initializes `unsigned NumCmp`.
  **L379 CN**: 对 `unsigned NumCmp` 进行赋值或初始化。
- **L380 EN**: Assigns or initializes `const BasicBlock *BB`.
  **L380 CN**: 对 `const BasicBlock *BB` 进行赋值或初始化。

### Lines 381-400

````cpp
    DestMap[BB] += NumCmp;
  }

  APInt Low = Clusters[First].Low->getValue();
  APInt High = Clusters[Last].High->getValue();
  assert(Low.slt(High));

  if (!TLI->isSuitableForBitTests(DestMap, Low, High, *DL))
    return false;

  APInt LowBound;
  APInt CmpRange;

  const int BitWidth = TLI->getPointerTy(*DL).getSizeInBits();
  assert(TLI->rangeFitsInWord(Low, High, *DL) &&
         "Case range must fit in bit mask!");

  // Check if the clusters cover a contiguous range such that no value in the
  // range will jump to the default statement.
  bool ContiguousRange = true;
````
- **L381 EN**: Assigns or initializes `DestMap[BB] +`.
  **L381 CN**: 对 `DestMap[BB] +` 进行赋值或初始化。
- **L382 EN**: Closes the current scope.
  **L382 CN**: 关闭当前作用域。
- **L383 EN**: Separates nearby statements for readability.
  **L383 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L384 EN**: Assigns or initializes `APInt Low`.
  **L384 CN**: 对 `APInt Low` 进行赋值或初始化。
- **L385 EN**: Assigns or initializes `APInt High`.
  **L385 CN**: 对 `APInt High` 进行赋值或初始化。
- **L386 EN**: Checks an invariant in debug builds.
  **L386 CN**: 在调试构建中检查一个不变量。
- **L387 EN**: Separates nearby statements for readability.
  **L387 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L388 EN**: Begins a conditional branch.
  **L388 CN**: 开始一个条件分支。
- **L389 EN**: Returns `false` to the caller.
  **L389 CN**: 向调用者返回 `false`。
- **L390 EN**: Separates nearby statements for readability.
  **L390 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L391 EN**: Executes statement `APInt LowBound;`.
  **L391 CN**: 执行语句 `APInt LowBound;`。
- **L392 EN**: Executes statement `APInt CmpRange;`.
  **L392 CN**: 执行语句 `APInt CmpRange;`。
- **L393 EN**: Separates nearby statements for readability.
  **L393 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L394 EN**: Assigns or initializes `const int BitWidth`.
  **L394 CN**: 对 `const int BitWidth` 进行赋值或初始化。
- **L395 EN**: Checks an invariant in debug builds.
  **L395 CN**: 在调试构建中检查一个不变量。
- **L396 EN**: Executes statement `"Case range must fit in bit mask!");`.
  **L396 CN**: 执行语句 `"Case range must fit in bit mask!");`。
- **L397 EN**: Separates nearby statements for readability.
  **L397 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L398 EN**: Comment documents: `Check if the clusters cover a contiguous range such that no value in the`.
  **L398 CN**: 注释说明：`Check if the clusters cover a contiguous range such that no value in the`。
- **L399 EN**: Comment documents: `range will jump to the default statement.`.
  **L399 CN**: 注释说明：`range will jump to the default statement.`。
- **L400 EN**: Assigns or initializes `bool ContiguousRange`.
  **L400 CN**: 对 `bool ContiguousRange` 进行赋值或初始化。

### Lines 401-420

````cpp
  for (int64_t I = First + 1; I <= Last; ++I) {
    if (Clusters[I].Low->getValue() != Clusters[I - 1].High->getValue() + 1) {
      ContiguousRange = false;
      break;
    }
  }

  if (Low.isStrictlyPositive() && High.slt(BitWidth)) {
    // Optimize the case where all the case values fit in a word without having
    // to subtract minValue. In this case, we can optimize away the subtraction.
    LowBound = APInt::getZero(Low.getBitWidth());
    CmpRange = High;
    ContiguousRange = false;
  } else {
    LowBound = Low;
    CmpRange = High - Low;
  }

  CaseBitsVector CBV;
  auto TotalProb = BranchProbability::getZero();
````
- **L401 EN**: Starts a loop over a sequence or range.
  **L401 CN**: 开始遍历序列或范围的循环。
- **L402 EN**: Begins a conditional branch.
  **L402 CN**: 开始一个条件分支。
- **L403 EN**: Assigns or initializes `ContiguousRange`.
  **L403 CN**: 对 `ContiguousRange` 进行赋值或初始化。
- **L404 EN**: Breaks out of the current control-flow construct.
  **L404 CN**: 跳出当前控制流结构。
- **L405 EN**: Closes the current scope.
  **L405 CN**: 关闭当前作用域。
- **L406 EN**: Closes the current scope.
  **L406 CN**: 关闭当前作用域。
- **L407 EN**: Separates nearby statements for readability.
  **L407 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L408 EN**: Begins a conditional branch.
  **L408 CN**: 开始一个条件分支。
- **L409 EN**: Comment documents: `Optimize the case where all the case values fit in a word without having`.
  **L409 CN**: 注释说明：`Optimize the case where all the case values fit in a word without having`。
- **L410 EN**: Comment documents: `to subtract minValue. In this case, we can optimize away the subtraction…`.
  **L410 CN**: 注释说明：`to subtract minValue. In this case, we can optimize away the subtraction…`。
- **L411 EN**: Declares function or method `getZero`.
  **L411 CN**: 声明函数或方法 `getZero`。
- **L412 EN**: Assigns or initializes `CmpRange`.
  **L412 CN**: 对 `CmpRange` 进行赋值或初始化。
- **L413 EN**: Assigns or initializes `ContiguousRange`.
  **L413 CN**: 对 `ContiguousRange` 进行赋值或初始化。
- **L414 EN**: Starts block `} else`.
  **L414 CN**: 开始代码块 `} else`。
- **L415 EN**: Assigns or initializes `LowBound`.
  **L415 CN**: 对 `LowBound` 进行赋值或初始化。
- **L416 EN**: Assigns or initializes `CmpRange`.
  **L416 CN**: 对 `CmpRange` 进行赋值或初始化。
- **L417 EN**: Closes the current scope.
  **L417 CN**: 关闭当前作用域。
- **L418 EN**: Separates nearby statements for readability.
  **L418 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L419 EN**: Executes statement `CaseBitsVector CBV;`.
  **L419 CN**: 执行语句 `CaseBitsVector CBV;`。
- **L420 EN**: Declares function or method `getZero`.
  **L420 CN**: 声明函数或方法 `getZero`。

### Lines 421-440

````cpp
  for (unsigned i = First; i <= Last; ++i) {
    // Find the CaseBits for this destination.
    unsigned j;
    for (j = 0; j < CBV.size(); ++j)
      if (CBV[j].BB == Clusters[i].MBB)
        break;
    if (j == CBV.size())
      CBV.push_back(
          CaseBits(0, Clusters[i].MBB, 0, BranchProbability::getZero()));
    CaseBits *CB = &CBV[j];

    // Update Mask, Bits and ExtraProb.
    uint64_t Lo = (Clusters[i].Low->getValue() - LowBound).getZExtValue();
    uint64_t Hi = (Clusters[i].High->getValue() - LowBound).getZExtValue();
    assert(Hi >= Lo && Hi < 64 && "Invalid bit case!");
    CB->Mask |= (-1ULL >> (63 - (Hi - Lo))) << Lo;
    CB->Bits += Hi - Lo + 1;
    CB->ExtraProb += Clusters[i].Prob;
    TotalProb += Clusters[i].Prob;
  }
````
- **L421 EN**: Starts a loop over a sequence or range.
  **L421 CN**: 开始遍历序列或范围的循环。
- **L422 EN**: Comment documents: `Find the CaseBits for this destination.`.
  **L422 CN**: 注释说明：`Find the CaseBits for this destination.`。
- **L423 EN**: Executes statement `unsigned j;`.
  **L423 CN**: 执行语句 `unsigned j;`。
- **L424 EN**: Starts a loop over a sequence or range.
  **L424 CN**: 开始遍历序列或范围的循环。
- **L425 EN**: Begins a conditional branch.
  **L425 CN**: 开始一个条件分支。
- **L426 EN**: Breaks out of the current control-flow construct.
  **L426 CN**: 跳出当前控制流结构。
- **L427 EN**: Begins a conditional branch.
  **L427 CN**: 开始一个条件分支。
- **L428 EN**: Continues logic with `CBV.push_back(`.
  **L428 CN**: 继续处理逻辑：`CBV.push_back(`。
- **L429 EN**: Declares function or method `CaseBits`.
  **L429 CN**: 声明函数或方法 `CaseBits`。
- **L430 EN**: Assigns or initializes `CaseBits *CB`.
  **L430 CN**: 对 `CaseBits *CB` 进行赋值或初始化。
- **L431 EN**: Separates nearby statements for readability.
  **L431 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L432 EN**: Comment documents: `Update Mask, Bits and ExtraProb.`.
  **L432 CN**: 注释说明：`Update Mask, Bits and ExtraProb.`。
- **L433 EN**: Assigns or initializes `uint64_t Lo`.
  **L433 CN**: 对 `uint64_t Lo` 进行赋值或初始化。
- **L434 EN**: Assigns or initializes `uint64_t Hi`.
  **L434 CN**: 对 `uint64_t Hi` 进行赋值或初始化。
- **L435 EN**: Checks an invariant in debug builds.
  **L435 CN**: 在调试构建中检查一个不变量。
- **L436 EN**: Assigns or initializes `CB->Mask |`.
  **L436 CN**: 对 `CB->Mask |` 进行赋值或初始化。
- **L437 EN**: Assigns or initializes `CB->Bits +`.
  **L437 CN**: 对 `CB->Bits +` 进行赋值或初始化。
- **L438 EN**: Assigns or initializes `CB->ExtraProb +`.
  **L438 CN**: 对 `CB->ExtraProb +` 进行赋值或初始化。
- **L439 EN**: Assigns or initializes `TotalProb +`.
  **L439 CN**: 对 `TotalProb +` 进行赋值或初始化。
- **L440 EN**: Closes the current scope.
  **L440 CN**: 关闭当前作用域。

### Lines 441-460

````cpp

  BitTestInfo BTI;
  llvm::sort(CBV, [](const CaseBits &a, const CaseBits &b) {
    // Sort by probability first, number of bits second, bit mask third.
    if (a.ExtraProb != b.ExtraProb)
      return a.ExtraProb > b.ExtraProb;
    if (a.Bits != b.Bits)
      return a.Bits > b.Bits;
    return a.Mask < b.Mask;
  });

  for (auto &CB : CBV) {
    MachineBasicBlock *BitTestBB =
        FuncInfo.MF->CreateMachineBasicBlock(SI->getParent());
    BTI.push_back(BitTestCase(CB.Mask, BitTestBB, CB.BB, CB.ExtraProb));
  }
  BitTestCases.emplace_back(std::move(LowBound), std::move(CmpRange),
                            SI->getCondition(), Register(), MVT::Other, false,
                            ContiguousRange, nullptr, nullptr, std::move(BTI),
                            TotalProb);
````
- **L441 EN**: Separates nearby statements for readability.
  **L441 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L442 EN**: Executes statement `BitTestInfo BTI;`.
  **L442 CN**: 执行语句 `BitTestInfo BTI;`。
- **L443 EN**: Begins the definition of `sort`.
  **L443 CN**: 开始定义 `sort`。
- **L444 EN**: Comment documents: `Sort by probability first, number of bits second, bit mask third.`.
  **L444 CN**: 注释说明：`Sort by probability first, number of bits second, bit mask third.`。
- **L445 EN**: Begins a conditional branch.
  **L445 CN**: 开始一个条件分支。
- **L446 EN**: Returns `a.ExtraProb > b.ExtraProb` to the caller.
  **L446 CN**: 向调用者返回 `a.ExtraProb > b.ExtraProb`。
- **L447 EN**: Begins a conditional branch.
  **L447 CN**: 开始一个条件分支。
- **L448 EN**: Returns `a.Bits > b.Bits` to the caller.
  **L448 CN**: 向调用者返回 `a.Bits > b.Bits`。
- **L449 EN**: Returns `a.Mask < b.Mask` to the caller.
  **L449 CN**: 向调用者返回 `a.Mask < b.Mask`。
- **L450 EN**: Executes statement `});`.
  **L450 CN**: 执行语句 `});`。
- **L451 EN**: Separates nearby statements for readability.
  **L451 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L452 EN**: Starts a loop over a sequence or range.
  **L452 CN**: 开始遍历序列或范围的循环。
- **L453 EN**: Continues logic with `MachineBasicBlock *BitTestBB =`.
  **L453 CN**: 继续处理逻辑：`MachineBasicBlock *BitTestBB =`。
- **L454 EN**: Executes statement `FuncInfo.MF->CreateMachineBasicBlock(SI->getParent());`.
  **L454 CN**: 执行语句 `FuncInfo.MF->CreateMachineBasicBlock(SI->getParent());`。
- **L455 EN**: Executes statement `BTI.push_back(BitTestCase(CB.Mask, BitTestBB, CB.BB, CB.ExtraProb));`.
  **L455 CN**: 执行语句 `BTI.push_back(BitTestCase(CB.Mask, BitTestBB, CB.BB, CB.ExtraProb));`。
- **L456 EN**: Closes the current scope.
  **L456 CN**: 关闭当前作用域。
- **L457 EN**: Provides part of the signature for `emplace_back`.
  **L457 CN**: 给出 `emplace_back` 的一部分签名。
- **L458 EN**: Continues logic with `SI->getCondition(), Register(), MVT::Other, false,`.
  **L458 CN**: 继续处理逻辑：`SI->getCondition(), Register(), MVT::Other, false,`。
- **L459 EN**: Provides part of the signature for `move`.
  **L459 CN**: 给出 `move` 的一部分签名。
- **L460 EN**: Executes statement `TotalProb);`.
  **L460 CN**: 执行语句 `TotalProb);`。

### Lines 461-480

````cpp

  BTCluster = CaseCluster::bitTests(Clusters[First].Low, Clusters[Last].High,
                                    BitTestCases.size() - 1, TotalProb);
  return true;
}

void SwitchCG::sortAndRangeify(CaseClusterVector &Clusters) {
#ifndef NDEBUG
  for (const CaseCluster &CC : Clusters)
    assert(CC.Low == CC.High && "Input clusters must be single-case");
#endif

  llvm::sort(Clusters, [](const CaseCluster &a, const CaseCluster &b) {
    return a.Low->getValue().slt(b.Low->getValue());
  });

  // Merge adjacent clusters with the same destination.
  const unsigned N = Clusters.size();
  unsigned DstIndex = 0;
  for (unsigned SrcIndex = 0; SrcIndex < N; ++SrcIndex) {
````
- **L461 EN**: Separates nearby statements for readability.
  **L461 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L462 EN**: Provides part of the signature for `bitTests`.
  **L462 CN**: 给出 `bitTests` 的一部分签名。
- **L463 EN**: Executes statement `BitTestCases.size() - 1, TotalProb);`.
  **L463 CN**: 执行语句 `BitTestCases.size() - 1, TotalProb);`。
- **L464 EN**: Returns `true` to the caller.
  **L464 CN**: 向调用者返回 `true`。
- **L465 EN**: Closes the current scope.
  **L465 CN**: 关闭当前作用域。
- **L466 EN**: Separates nearby statements for readability.
  **L466 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L467 EN**: Begins the definition of `sortAndRangeify`.
  **L467 CN**: 开始定义 `sortAndRangeify`。
- **L468 EN**: Starts a preprocessor conditional block.
  **L468 CN**: 开始一个预处理条件块。
- **L469 EN**: Starts a loop over a sequence or range.
  **L469 CN**: 开始遍历序列或范围的循环。
- **L470 EN**: Checks an invariant in debug builds.
  **L470 CN**: 在调试构建中检查一个不变量。
- **L471 EN**: Ends the current preprocessor conditional block.
  **L471 CN**: 结束当前的预处理条件块。
- **L472 EN**: Separates nearby statements for readability.
  **L472 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L473 EN**: Begins the definition of `sort`.
  **L473 CN**: 开始定义 `sort`。
- **L474 EN**: Returns `a.Low->getValue().slt(b.Low->getValue())` to the caller.
  **L474 CN**: 向调用者返回 `a.Low->getValue().slt(b.Low->getValue())`。
- **L475 EN**: Executes statement `});`.
  **L475 CN**: 执行语句 `});`。
- **L476 EN**: Separates nearby statements for readability.
  **L476 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L477 EN**: Comment documents: `Merge adjacent clusters with the same destination.`.
  **L477 CN**: 注释说明：`Merge adjacent clusters with the same destination.`。
- **L478 EN**: Assigns or initializes `const unsigned N`.
  **L478 CN**: 对 `const unsigned N` 进行赋值或初始化。
- **L479 EN**: Assigns or initializes `unsigned DstIndex`.
  **L479 CN**: 对 `unsigned DstIndex` 进行赋值或初始化。
- **L480 EN**: Starts a loop over a sequence or range.
  **L480 CN**: 开始遍历序列或范围的循环。

### Lines 481-500

````cpp
    CaseCluster &CC = Clusters[SrcIndex];
    const ConstantInt *CaseVal = CC.Low;
    MachineBasicBlock *Succ = CC.MBB;

    if (DstIndex != 0 && Clusters[DstIndex - 1].MBB == Succ &&
        (CaseVal->getValue() - Clusters[DstIndex - 1].High->getValue()) == 1) {
      // If this case has the same successor and is a neighbour, merge it into
      // the previous cluster.
      Clusters[DstIndex - 1].High = CaseVal;
      Clusters[DstIndex - 1].Prob += CC.Prob;
    } else {
      std::memmove(&Clusters[DstIndex++], &Clusters[SrcIndex],
                   sizeof(Clusters[SrcIndex]));
    }
  }
  Clusters.resize(DstIndex);
}

unsigned SwitchCG::SwitchLowering::caseClusterRank(const CaseCluster &CC,
                                                   CaseClusterIt First,
````
- **L481 EN**: Assigns or initializes `CaseCluster &CC`.
  **L481 CN**: 对 `CaseCluster &CC` 进行赋值或初始化。
- **L482 EN**: Assigns or initializes `const ConstantInt *CaseVal`.
  **L482 CN**: 对 `const ConstantInt *CaseVal` 进行赋值或初始化。
- **L483 EN**: Assigns or initializes `MachineBasicBlock *Succ`.
  **L483 CN**: 对 `MachineBasicBlock *Succ` 进行赋值或初始化。
- **L484 EN**: Separates nearby statements for readability.
  **L484 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L485 EN**: Begins a conditional branch.
  **L485 CN**: 开始一个条件分支。
- **L486 EN**: Starts block `(CaseVal->getValue() - Clusters[DstIndex - 1].High->getValue()) == 1)`.
  **L486 CN**: 开始代码块 `(CaseVal->getValue() - Clusters[DstIndex - 1].High->getValue()) == 1)`。
- **L487 EN**: Comment documents: `If this case has the same successor and is a neighbour, merge it into`.
  **L487 CN**: 注释说明：`If this case has the same successor and is a neighbour, merge it into`。
- **L488 EN**: Comment documents: `the previous cluster.`.
  **L488 CN**: 注释说明：`the previous cluster.`。
- **L489 EN**: Assigns or initializes `Clusters[DstIndex - 1].High`.
  **L489 CN**: 对 `Clusters[DstIndex - 1].High` 进行赋值或初始化。
- **L490 EN**: Assigns or initializes `Clusters[DstIndex - 1].Prob +`.
  **L490 CN**: 对 `Clusters[DstIndex - 1].Prob +` 进行赋值或初始化。
- **L491 EN**: Starts block `} else`.
  **L491 CN**: 开始代码块 `} else`。
- **L492 EN**: Provides part of the signature for `memmove`.
  **L492 CN**: 给出 `memmove` 的一部分签名。
- **L493 EN**: Executes statement `sizeof(Clusters[SrcIndex]));`.
  **L493 CN**: 执行语句 `sizeof(Clusters[SrcIndex]));`。
- **L494 EN**: Closes the current scope.
  **L494 CN**: 关闭当前作用域。
- **L495 EN**: Closes the current scope.
  **L495 CN**: 关闭当前作用域。
- **L496 EN**: Executes statement `Clusters.resize(DstIndex);`.
  **L496 CN**: 执行语句 `Clusters.resize(DstIndex);`。
- **L497 EN**: Closes the current scope.
  **L497 CN**: 关闭当前作用域。
- **L498 EN**: Separates nearby statements for readability.
  **L498 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L499 EN**: Provides part of the signature for `caseClusterRank`.
  **L499 CN**: 给出 `caseClusterRank` 的一部分签名。
- **L500 EN**: Continues logic with `CaseClusterIt First,`.
  **L500 CN**: 继续处理逻辑：`CaseClusterIt First,`。

### Lines 501-520

````cpp
                                                   CaseClusterIt Last) {
  return std::count_if(First, Last + 1, [&](const CaseCluster &X) {
    if (X.Prob != CC.Prob)
      return X.Prob > CC.Prob;

    // Ties are broken by comparing the case value.
    return X.Low->getValue().slt(CC.Low->getValue());
  });
}

llvm::SwitchCG::SwitchLowering::SplitWorkItemInfo
SwitchCG::SwitchLowering::computeSplitWorkItemInfo(
    const SwitchWorkListItem &W) {
  CaseClusterIt LastLeft = W.FirstCluster;
  CaseClusterIt FirstRight = W.LastCluster;
  auto LeftProb = LastLeft->Prob + W.DefaultProb / 2;
  auto RightProb = FirstRight->Prob + W.DefaultProb / 2;

  // Move LastLeft and FirstRight towards each other from opposite directions to
  // find a partitioning of the clusters which balances the probability on both
````
- **L501 EN**: Starts block `CaseClusterIt Last)`.
  **L501 CN**: 开始代码块 `CaseClusterIt Last)`。
- **L502 EN**: Returns `std::count_if(First, Last + 1, [&](const CaseCluster &X) {` to the caller.
  **L502 CN**: 向调用者返回 `std::count_if(First, Last + 1, [&](const CaseCluster &X) {`。
- **L503 EN**: Begins a conditional branch.
  **L503 CN**: 开始一个条件分支。
- **L504 EN**: Returns `X.Prob > CC.Prob` to the caller.
  **L504 CN**: 向调用者返回 `X.Prob > CC.Prob`。
- **L505 EN**: Separates nearby statements for readability.
  **L505 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L506 EN**: Comment documents: `Ties are broken by comparing the case value.`.
  **L506 CN**: 注释说明：`Ties are broken by comparing the case value.`。
- **L507 EN**: Returns `X.Low->getValue().slt(CC.Low->getValue())` to the caller.
  **L507 CN**: 向调用者返回 `X.Low->getValue().slt(CC.Low->getValue())`。
- **L508 EN**: Executes statement `});`.
  **L508 CN**: 执行语句 `});`。
- **L509 EN**: Closes the current scope.
  **L509 CN**: 关闭当前作用域。
- **L510 EN**: Separates nearby statements for readability.
  **L510 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L511 EN**: Continues logic with `llvm::SwitchCG::SwitchLowering::SplitWorkItemInfo`.
  **L511 CN**: 继续处理逻辑：`llvm::SwitchCG::SwitchLowering::SplitWorkItemInfo`。
- **L512 EN**: Provides part of the signature for `computeSplitWorkItemInfo`.
  **L512 CN**: 给出 `computeSplitWorkItemInfo` 的一部分签名。
- **L513 EN**: Starts block `const SwitchWorkListItem &W)`.
  **L513 CN**: 开始代码块 `const SwitchWorkListItem &W)`。
- **L514 EN**: Assigns or initializes `CaseClusterIt LastLeft`.
  **L514 CN**: 对 `CaseClusterIt LastLeft` 进行赋值或初始化。
- **L515 EN**: Assigns or initializes `CaseClusterIt FirstRight`.
  **L515 CN**: 对 `CaseClusterIt FirstRight` 进行赋值或初始化。
- **L516 EN**: Assigns or initializes `auto LeftProb`.
  **L516 CN**: 对 `auto LeftProb` 进行赋值或初始化。
- **L517 EN**: Assigns or initializes `auto RightProb`.
  **L517 CN**: 对 `auto RightProb` 进行赋值或初始化。
- **L518 EN**: Separates nearby statements for readability.
  **L518 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L519 EN**: Comment documents: `Move LastLeft and FirstRight towards each other from opposite directions…`.
  **L519 CN**: 注释说明：`Move LastLeft and FirstRight towards each other from opposite directions…`。
- **L520 EN**: Comment documents: `find a partitioning of the clusters which balances the probability on bo…`.
  **L520 CN**: 注释说明：`find a partitioning of the clusters which balances the probability on bo…`。

### Lines 521-540

````cpp
  // sides. If LeftProb and RightProb are equal, alternate which side is
  // taken to ensure 0-probability nodes are distributed evenly.
  unsigned I = 0;
  while (LastLeft + 1 < FirstRight) {
    if (LeftProb < RightProb || (LeftProb == RightProb && (I & 1)))
      LeftProb += (++LastLeft)->Prob;
    else
      RightProb += (--FirstRight)->Prob;
    I++;
  }

  while (true) {
    // Our binary search tree differs from a typical BST in that ours can have
    // up to three values in each leaf. The pivot selection above doesn't take
    // that into account, which means the tree might require more nodes and be
    // less efficient. We compensate for this here.

    unsigned NumLeft = LastLeft - W.FirstCluster + 1;
    unsigned NumRight = W.LastCluster - FirstRight + 1;

````
- **L521 EN**: Comment documents: `sides. If LeftProb and RightProb are equal, alternate which side is`.
  **L521 CN**: 注释说明：`sides. If LeftProb and RightProb are equal, alternate which side is`。
- **L522 EN**: Comment documents: `taken to ensure 0-probability nodes are distributed evenly.`.
  **L522 CN**: 注释说明：`taken to ensure 0-probability nodes are distributed evenly.`。
- **L523 EN**: Assigns or initializes `unsigned I`.
  **L523 CN**: 对 `unsigned I` 进行赋值或初始化。
- **L524 EN**: Starts a while loop controlled by a condition.
  **L524 CN**: 开始一个由条件控制的 while 循环。
- **L525 EN**: Begins a conditional branch.
  **L525 CN**: 开始一个条件分支。
- **L526 EN**: Assigns or initializes `LeftProb +`.
  **L526 CN**: 对 `LeftProb +` 进行赋值或初始化。
- **L527 EN**: Handles the fallback branch.
  **L527 CN**: 处理兜底分支。
- **L528 EN**: Assigns or initializes `RightProb +`.
  **L528 CN**: 对 `RightProb +` 进行赋值或初始化。
- **L529 EN**: Executes statement `I++;`.
  **L529 CN**: 执行语句 `I++;`。
- **L530 EN**: Closes the current scope.
  **L530 CN**: 关闭当前作用域。
- **L531 EN**: Separates nearby statements for readability.
  **L531 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L532 EN**: Starts a while loop controlled by a condition.
  **L532 CN**: 开始一个由条件控制的 while 循环。
- **L533 EN**: Comment documents: `Our binary search tree differs from a typical BST in that ours can have`.
  **L533 CN**: 注释说明：`Our binary search tree differs from a typical BST in that ours can have`。
- **L534 EN**: Comment documents: `up to three values in each leaf. The pivot selection above doesn't take`.
  **L534 CN**: 注释说明：`up to three values in each leaf. The pivot selection above doesn't take`。
- **L535 EN**: Comment documents: `that into account, which means the tree might require more nodes and be`.
  **L535 CN**: 注释说明：`that into account, which means the tree might require more nodes and be`。
- **L536 EN**: Comment documents: `less efficient. We compensate for this here.`.
  **L536 CN**: 注释说明：`less efficient. We compensate for this here.`。
- **L537 EN**: Separates nearby statements for readability.
  **L537 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L538 EN**: Assigns or initializes `unsigned NumLeft`.
  **L538 CN**: 对 `unsigned NumLeft` 进行赋值或初始化。
- **L539 EN**: Assigns or initializes `unsigned NumRight`.
  **L539 CN**: 对 `unsigned NumRight` 进行赋值或初始化。
- **L540 EN**: Separates nearby statements for readability.
  **L540 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 541-560

````cpp
    if (std::min(NumLeft, NumRight) < 3 && std::max(NumLeft, NumRight) > 3) {
      // If one side has less than 3 clusters, and the other has more than 3,
      // consider taking a cluster from the other side.

      if (NumLeft < NumRight) {
        // Consider moving the first cluster on the right to the left side.
        CaseCluster &CC = *FirstRight;
        unsigned RightSideRank = caseClusterRank(CC, FirstRight, W.LastCluster);
        unsigned LeftSideRank = caseClusterRank(CC, W.FirstCluster, LastLeft);
        if (LeftSideRank <= RightSideRank) {
          // Moving the cluster to the left does not demote it.
          ++LastLeft;
          ++FirstRight;
          continue;
        }
      } else {
        assert(NumRight < NumLeft);
        // Consider moving the last element on the left to the right side.
        CaseCluster &CC = *LastLeft;
        unsigned LeftSideRank = caseClusterRank(CC, W.FirstCluster, LastLeft);
````
- **L541 EN**: Begins a conditional branch.
  **L541 CN**: 开始一个条件分支。
- **L542 EN**: Comment documents: `If one side has less than 3 clusters, and the other has more than 3,`.
  **L542 CN**: 注释说明：`If one side has less than 3 clusters, and the other has more than 3,`。
- **L543 EN**: Comment documents: `consider taking a cluster from the other side.`.
  **L543 CN**: 注释说明：`consider taking a cluster from the other side.`。
- **L544 EN**: Separates nearby statements for readability.
  **L544 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L545 EN**: Begins a conditional branch.
  **L545 CN**: 开始一个条件分支。
- **L546 EN**: Comment documents: `Consider moving the first cluster on the right to the left side.`.
  **L546 CN**: 注释说明：`Consider moving the first cluster on the right to the left side.`。
- **L547 EN**: Assigns or initializes `CaseCluster &CC`.
  **L547 CN**: 对 `CaseCluster &CC` 进行赋值或初始化。
- **L548 EN**: Assigns or initializes `unsigned RightSideRank`.
  **L548 CN**: 对 `unsigned RightSideRank` 进行赋值或初始化。
- **L549 EN**: Assigns or initializes `unsigned LeftSideRank`.
  **L549 CN**: 对 `unsigned LeftSideRank` 进行赋值或初始化。
- **L550 EN**: Begins a conditional branch.
  **L550 CN**: 开始一个条件分支。
- **L551 EN**: Comment documents: `Moving the cluster to the left does not demote it.`.
  **L551 CN**: 注释说明：`Moving the cluster to the left does not demote it.`。
- **L552 EN**: Executes statement `++LastLeft;`.
  **L552 CN**: 执行语句 `++LastLeft;`。
- **L553 EN**: Executes statement `++FirstRight;`.
  **L553 CN**: 执行语句 `++FirstRight;`。
- **L554 EN**: Skips to the next loop iteration.
  **L554 CN**: 跳到下一次循环迭代。
- **L555 EN**: Closes the current scope.
  **L555 CN**: 关闭当前作用域。
- **L556 EN**: Starts block `} else`.
  **L556 CN**: 开始代码块 `} else`。
- **L557 EN**: Checks an invariant in debug builds.
  **L557 CN**: 在调试构建中检查一个不变量。
- **L558 EN**: Comment documents: `Consider moving the last element on the left to the right side.`.
  **L558 CN**: 注释说明：`Consider moving the last element on the left to the right side.`。
- **L559 EN**: Assigns or initializes `CaseCluster &CC`.
  **L559 CN**: 对 `CaseCluster &CC` 进行赋值或初始化。
- **L560 EN**: Assigns or initializes `unsigned LeftSideRank`.
  **L560 CN**: 对 `unsigned LeftSideRank` 进行赋值或初始化。

### Lines 561-578

````cpp
        unsigned RightSideRank = caseClusterRank(CC, FirstRight, W.LastCluster);
        if (RightSideRank <= LeftSideRank) {
          // Moving the cluster to the right does not demot it.
          --LastLeft;
          --FirstRight;
          continue;
        }
      }
    }
    break;
  }

  assert(LastLeft + 1 == FirstRight);
  assert(LastLeft >= W.FirstCluster);
  assert(FirstRight <= W.LastCluster);

  return SplitWorkItemInfo{LastLeft, FirstRight, LeftProb, RightProb};
}
````
- **L561 EN**: Assigns or initializes `unsigned RightSideRank`.
  **L561 CN**: 对 `unsigned RightSideRank` 进行赋值或初始化。
- **L562 EN**: Begins a conditional branch.
  **L562 CN**: 开始一个条件分支。
- **L563 EN**: Comment documents: `Moving the cluster to the right does not demot it.`.
  **L563 CN**: 注释说明：`Moving the cluster to the right does not demot it.`。
- **L564 EN**: Executes statement `--LastLeft;`.
  **L564 CN**: 执行语句 `--LastLeft;`。
- **L565 EN**: Executes statement `--FirstRight;`.
  **L565 CN**: 执行语句 `--FirstRight;`。
- **L566 EN**: Skips to the next loop iteration.
  **L566 CN**: 跳到下一次循环迭代。
- **L567 EN**: Closes the current scope.
  **L567 CN**: 关闭当前作用域。
- **L568 EN**: Closes the current scope.
  **L568 CN**: 关闭当前作用域。
- **L569 EN**: Closes the current scope.
  **L569 CN**: 关闭当前作用域。
- **L570 EN**: Breaks out of the current control-flow construct.
  **L570 CN**: 跳出当前控制流结构。
- **L571 EN**: Closes the current scope.
  **L571 CN**: 关闭当前作用域。
- **L572 EN**: Separates nearby statements for readability.
  **L572 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L573 EN**: Checks an invariant in debug builds.
  **L573 CN**: 在调试构建中检查一个不变量。
- **L574 EN**: Checks an invariant in debug builds.
  **L574 CN**: 在调试构建中检查一个不变量。
- **L575 EN**: Checks an invariant in debug builds.
  **L575 CN**: 在调试构建中检查一个不变量。
- **L576 EN**: Separates nearby statements for readability.
  **L576 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L577 EN**: Returns `SplitWorkItemInfo{LastLeft, FirstRight, LeftProb, RightProb}` to the caller.
  **L577 CN**: 向调用者返回 `SplitWorkItemInfo{LastLeft, FirstRight, LeftProb, RightProb}`。
- **L578 EN**: Closes the current scope.
  **L578 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Register management** / **寄存器管理**
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **SelectionDAG lowering** / **SelectionDAG 降低**
- **GlobalISel pipeline** / **GlobalISel 流水线**
- **Control-flow updates** / **控制流更新**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/SwitchLoweringUtils.h`, `llvm/CodeGen/FunctionLoweringInfo.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/Target/TargetMachine.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
