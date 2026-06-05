# AddDiscriminators.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/AddDiscriminators.cpp` | `llvm/lib/Transforms/Utils/AddDiscriminators.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements insert DWARF path discriminators within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 AddDiscriminators 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-25

```cpp
//===- AddDiscriminators.cpp - Insert DWARF path discriminators -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file adds DWARF discriminators to the IR. Path discriminators are
// used to decide what CFG path was taken inside sub-graphs whose instructions
// share the same line and column number information.
//
// The main user of this is the sample profiler. Instruction samples are
// mapped to line number information. Since a single line may be spread
// out over several basic blocks, discriminators add more precise location
// for the samples.
//
// For example,
//
//   1  #define ASSERT(P)
//   2      if (!(P))
//   3        abort()
//   ...
//   100   while (true) {
//   101     ASSERT (sum < 0);
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 26-54

```cpp
//   102     ...
//   130   }
//
// when converted to IR, this snippet looks something like:
//
// while.body:                                       ; preds = %entry, %if.end
//   %0 = load i32* %sum, align 4, !dbg !15
//   %cmp = icmp slt i32 %0, 0, !dbg !15
//   br i1 %cmp, label %if.end, label %if.then, !dbg !15
//
// if.then:                                          ; preds = %while.body
//   call void @abort(), !dbg !15
//   br label %if.end, !dbg !15
//
// Notice that all the instructions in blocks 'while.body' and 'if.then'
// have exactly the same debug information. When this program is sampled
// at runtime, the profiler will assume that all these instructions are
// equally frequent. This, in turn, will consider the edge while.body->if.then
// to be frequently taken (which is incorrect).
//
// By adding a discriminator value to the instructions in block 'if.then',
// we can distinguish instructions at line 101 with discriminator 0 from
// the instructions at line 101 with discriminator 1.
//
// For more details about DWARF discriminators, please visit
// http://wiki.dwarfstd.org/index.php?title=Path_Discriminators
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 55-77

```cpp
#include "llvm/Transforms/Utils/AddDiscriminators.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/DebugInfoMetadata.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/PassManager.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h"
#include <utility>

using namespace llvm;
using namespace sampleprofutil;

#define DEBUG_TYPE "add-discriminators"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 78-102

```cpp
// Command line option to disable discriminator generation even in the
// presence of debug information. This is only needed when debugging
// debug info generation issues.
static cl::opt<bool> NoDiscriminators(
    "no-discriminators", cl::init(false),
    cl::desc("Disable generation of discriminator information."));

static bool shouldHaveDiscriminator(const Instruction *I) {
  return !isa<IntrinsicInst>(I) || isa<MemIntrinsic>(I);
}

/// Assign DWARF discriminators.
///
/// To assign discriminators, we examine the boundaries of every
/// basic block and its successors. Suppose there is a basic block B1
/// with successor B2. The last instruction I1 in B1 and the first
/// instruction I2 in B2 are located at the same file and line number.
/// This situation is illustrated in the following code snippet:
///
///       if (i < 10) x = i;
///
///     entry:
///       br i1 %cmp, label %if.then, label %if.end, !dbg !10
///     if.then:
///       %1 = load i32* %i.addr, align 4, !dbg !10
```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include shouldHaveDiscriminator, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 shouldHaveDiscriminator，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 103-127

```cpp
///       store i32 %1, i32* %x, align 4, !dbg !10
///       br label %if.end, !dbg !10
///     if.end:
///       ret void, !dbg !12
///
/// Notice how the branch instruction in block 'entry' and all the
/// instructions in block 'if.then' have the exact same debug location
/// information (!dbg !10).
///
/// To distinguish instructions in block 'entry' from instructions in
/// block 'if.then', we generate a new lexical block for all the
/// instruction in block 'if.then' that share the same file and line
/// location with the last instruction of block 'entry'.
///
/// This new lexical block will have the same location information as
/// the previous one, but with a new DWARF discriminator value.
///
/// One of the main uses of this discriminator value is in runtime
/// sample profilers. It allows the profiler to distinguish instructions
/// at location !dbg !10 that execute on different basic blocks. This is
/// important because while the predicate 'if (x < 10)' may have been
/// executed millions of times, the assignment 'x = i' may have only
/// executed a handful of times (meaning that the entry->if.then edge is
/// seldom taken).
///
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 128-152

```cpp
/// If we did not have discriminator information, the profiler would
/// assign the same weight to both blocks 'entry' and 'if.then', which
/// in turn will make it conclude that the entry->if.then edge is very
/// hot.
///
/// To decide where to create new discriminator values, this function
/// traverses the CFG and examines instruction at basic block boundaries.
/// If the last instruction I1 of a block B1 is at the same file and line
/// location as instruction I2 of successor B2, then it creates a new
/// lexical block for I2 and all the instruction in B2 that share the same
/// file and line location as I2. This new lexical block will have a
/// different discriminator number than I1.
static bool addDiscriminators(Function &F) {
  // If the function has debug information, but the user has disabled
  // discriminators, do nothing.
  // Simlarly, if the function has no debug info, do nothing.
  if (NoDiscriminators || !F.getSubprogram())
    return false;

  // Create FSDiscriminatorVariable if flow sensitive discriminators are used.
  if (EnableFSDiscriminator)
    createFSDiscriminatorVariable(F.getParent());

  bool Changed = false;

```
- EN: Core entities appearing here include addDiscriminators, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addDiscriminators，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 153-177

```cpp
  using Location = std::pair<StringRef, unsigned>;
  using BBSet = DenseSet<const BasicBlock *>;
  using LocationBBMap = DenseMap<Location, BBSet>;
  using LocationDiscriminatorMap = DenseMap<Location, unsigned>;
  using LocationSet = DenseSet<Location>;

  LocationBBMap LBM;
  LocationDiscriminatorMap LDM;

  // Traverse all instructions in the function. If the source line location
  // of the instruction appears in other basic block, assign a new
  // discriminator for this instruction.
  for (BasicBlock &B : F) {
    for (auto &I : B) {
      // Not all intrinsic calls should have a discriminator.
      // We want to avoid a non-deterministic assignment of discriminators at
      // different debug levels. We still allow discriminators on memory
      // intrinsic calls because those can be early expanded by SROA into
      // pairs of loads and stores, and the expanded load/store instructions
      // should have a valid discriminator.
      if (!shouldHaveDiscriminator(&I))
        continue;
      const DILocation *DIL = I.getDebugLoc();
      if (!DIL)
        continue;
```
- EN: This region continues the AddDiscriminators implementation with local helper logic centered on Location, StringRef, BBSet, DenseSet.
- CN: 这一段延续了 AddDiscriminators 的主体实现，围绕 Location, StringRef, BBSet, DenseSet 等局部辅助逻辑展开。

### Lines 178-203

```cpp
      Location L = std::make_pair(DIL->getFilename(), DIL->getLine());
      auto &BBMap = LBM[L];
      auto R = BBMap.insert(&B);
      if (BBMap.size() == 1)
        continue;
      // If we could insert more than one block with the same line+file, a
      // discriminator is needed to distinguish both instructions.
      // Only the lowest 7 bits are used to represent a discriminator to fit
      // it in 1 byte ULEB128 representation.
      unsigned Discriminator = R.second ? ++LDM[L] : LDM[L];
      auto NewDIL = DIL->cloneWithBaseDiscriminator(Discriminator);
      if (!NewDIL) {
        LLVM_DEBUG(dbgs() << "Could not encode discriminator: "
                          << DIL->getFilename() << ":" << DIL->getLine() << ":"
                          << DIL->getColumn() << ":" << Discriminator << " "
                          << I << "\n");
      } else {
        I.setDebugLoc(*NewDIL);
        LLVM_DEBUG(dbgs() << DIL->getFilename() << ":" << DIL->getLine() << ":"
                   << DIL->getColumn() << ":" << Discriminator << " " << I
                   << "\n");
      }
      Changed = true;
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 204-228

```cpp
  // Traverse all instructions and assign new discriminators to call
  // instructions with the same lineno that are in the same basic block.
  // Sample base profile needs to distinguish different function calls within
  // a same source line for correct profile annotation.
  for (BasicBlock &B : F) {
    LocationSet CallLocations;
    for (auto &I : B) {
      // We bypass intrinsic calls for the following two reasons:
      //  1) We want to avoid a non-deterministic assignment of
      //     discriminators.
      //  2) We want to minimize the number of base discriminators used.
      if (!isa<InvokeInst>(I) && (!isa<CallInst>(I) || isa<IntrinsicInst>(I)))  
        continue;

      DILocation *CurrentDIL = I.getDebugLoc();
      if (!CurrentDIL)
        continue;
      Location L =
          std::make_pair(CurrentDIL->getFilename(), CurrentDIL->getLine());
      if (!CallLocations.insert(L).second) {
        unsigned Discriminator = ++LDM[L];
        auto NewDIL = CurrentDIL->cloneWithBaseDiscriminator(Discriminator);
        if (!NewDIL) {
          LLVM_DEBUG(dbgs()
                     << "Could not encode discriminator: "
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 229-249

```cpp
                     << CurrentDIL->getFilename() << ":"
                     << CurrentDIL->getLine() << ":" << CurrentDIL->getColumn()
                     << ":" << Discriminator << " " << I << "\n");
        } else {
          I.setDebugLoc(*NewDIL);
          Changed = true;
        }
      }
    }
  }
  return Changed;
}

PreservedAnalyses AddDiscriminatorsPass::run(Function &F,
                                             FunctionAnalysisManager &AM) {
  if (!addDiscriminators(F))
    return PreservedAnalyses::all();

  // FIXME: should be all()
  return PreservedAnalyses::none();
}
```
- EN: This region continues the AddDiscriminators implementation with local helper logic centered on CurrentDIL, Discriminator, NewDIL, Changed.
- CN: 这一段延续了 AddDiscriminators 的主体实现，围绕 CurrentDIL, Discriminator, NewDIL, Changed 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `shouldHaveDiscriminator, addDiscriminators` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`shouldHaveDiscriminator, addDiscriminators` 构成该文件对外 API 与主要实现挂钩。
- EN: Tuning surface: command-line knobs such as `NoDiscriminators` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `NoDiscriminators` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Function.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PassManager.h`, `llvm/Transforms/Utils/AddDiscriminators.h`, `llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Function.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PassManager.h`, `llvm/Transforms/Utils/AddDiscriminators.h`, `llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/StringRef.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/StringRef.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
