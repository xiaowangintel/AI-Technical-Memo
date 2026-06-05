# CGProfile.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Transforms/Instrumentation/CGProfile.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: Implements the LLVM transform logic, helper routines, and pass plumbing for CGProfile. / 该文件位于 `Transforms/Instrumentation`，主要实现 `CGProfile` 相关的 LLVM 变换逻辑、辅助流程以及 pass 接线代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===-- CGProfile.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Instrumentation/CGProfile.h"

#include "llvm/ADT/MapVector.h"
#include "llvm/Analysis/BlockFrequencyInfo.h"
#include "llvm/Analysis/LazyBlockFrequencyInfo.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PassManager.h"
#include "llvm/ProfileData/InstrProf.h"
#include "llvm/Transforms/Utils/Instrumentation.h"
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Includes "llvm/Transforms/Instrumentation/CGProfile.h" to access transform-specific declarations. / 引入 "llvm/Transforms/Instrumentation/CGProfile.h" 以使用变换相关声明。
- **L10**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L11**: Includes "llvm/ADT/MapVector.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/MapVector.h" 以使用LLVM ADT 数据结构/工具。
- **L12**: Includes "llvm/Analysis/BlockFrequencyInfo.h" to access analysis interfaces and cached results. / 引入 "llvm/Analysis/BlockFrequencyInfo.h" 以使用分析接口与缓存结果。
- **L13**: Includes "llvm/Analysis/LazyBlockFrequencyInfo.h" to access analysis interfaces and cached results. / 引入 "llvm/Analysis/LazyBlockFrequencyInfo.h" 以使用分析接口与缓存结果。
- **L14**: Includes "llvm/Analysis/TargetTransformInfo.h" to access analysis interfaces and cached results. / 引入 "llvm/Analysis/TargetTransformInfo.h" 以使用分析接口与缓存结果。
- **L15**: Includes "llvm/IR/Constants.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Constants.h" 以使用LLVM IR 核心类型与构造工具。
- **L16**: Includes "llvm/IR/MDBuilder.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/MDBuilder.h" 以使用LLVM IR 核心类型与构造工具。
- **L17**: Includes "llvm/IR/Module.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Module.h" 以使用LLVM IR 核心类型与构造工具。
- **L18**: Includes "llvm/IR/PassManager.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/PassManager.h" 以使用LLVM IR 核心类型与构造工具。
- **L19**: Includes "llvm/ProfileData/InstrProf.h" to access local declarations used by this file. / 引入 "llvm/ProfileData/InstrProf.h" 以使用本文件使用的本地声明。
- **L20**: Includes "llvm/Transforms/Utils/Instrumentation.h" to access shared transform utilities. / 引入 "llvm/Transforms/Utils/Instrumentation.h" 以使用共享的变换辅助工具。

### Lines 21-40

```cpp
#include <optional>

using namespace llvm;

static bool
addModuleFlags(Module &M,
               MapVector<std::pair<Function *, Function *>, uint64_t> &Counts) {
  if (Counts.empty())
    return false;

  LLVMContext &Context = M.getContext();
  MDBuilder MDB(Context);
  std::vector<Metadata *> Nodes;

  for (auto E : Counts) {
    Metadata *Vals[] = {ValueAsMetadata::get(E.first.first),
                        ValueAsMetadata::get(E.first.second),
                        MDB.createConstant(ConstantInt::get(
                            Type::getInt64Ty(Context), E.second))};
    Nodes.push_back(MDNode::get(Context, Vals));
```

- **L21**: Includes <optional> to access supporting declarations. / 引入 <optional> 以使用所需的辅助声明。
- **L22**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L23**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L24**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L25**: Continues the surrounding expression or declaration: `static bool`. / 继续构造周围的表达式或声明：`static bool`。
- **L26**: Continues a multi-line argument list or initializer: `addModuleFlags(Module &M,`. / 继续一个多行参数列表或初始化器：`addModuleFlags(Module &M,`。
- **L27**: Continues the surrounding expression or declaration: `MapVector<std::pair<Function *, Function *>, uint64_t> &Counts) {`. / 继续构造周围的表达式或声明：`MapVector<std::pair<Function *, Function *>, uint64_t> &Counts) {`。
- **L28**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L29**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L30**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L31**: Executes call or statement centered on `M.getContext`. / 执行以 `M.getContext` 为核心的调用或语句。
- **L32**: Executes call or statement centered on `MDB`. / 执行以 `MDB` 为核心的调用或语句。
- **L33**: Executes a standalone statement or declaration: `std::vector<Metadata *> Nodes;`. / 执行一条独立语句或声明：`std::vector<Metadata *> Nodes;`。
- **L34**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L35**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L36**: Continues a multi-line argument list or initializer: `Metadata *Vals[] = {ValueAsMetadata::get(E.first.first),`. / 继续一个多行参数列表或初始化器：`Metadata *Vals[] = {ValueAsMetadata::get(E.first.first),`。
- **L37**: Continues a multi-line argument list or initializer: `ValueAsMetadata::get(E.first.second),`. / 继续一个多行参数列表或初始化器：`ValueAsMetadata::get(E.first.second),`。
- **L38**: Continues the surrounding expression or declaration: `MDB.createConstant(ConstantInt::get(`. / 继续构造周围的表达式或声明：`MDB.createConstant(ConstantInt::get(`。
- **L39**: Executes call or statement centered on `Type::getInt64Ty`. / 执行以 `Type::getInt64Ty` 为核心的调用或语句。
- **L40**: Executes call or statement centered on `Nodes.push_back`. / 执行以 `Nodes.push_back` 为核心的调用或语句。

### Lines 41-60

```cpp
  }

  M.addModuleFlag(Module::Append, "CG Profile",
                  MDTuple::getDistinct(Context, Nodes));
  return true;
}

static bool runCGProfilePass(Module &M, FunctionAnalysisManager &FAM,
                             bool InLTO) {
  MapVector<std::pair<Function *, Function *>, uint64_t> Counts;
  InstrProfSymtab Symtab;
  auto UpdateCounts = [&](TargetTransformInfo &TTI, Function *F,
                          Function *CalledF, uint64_t NewCount) {
    if (NewCount == 0)
      return;
    if (!CalledF || !TTI.isLoweredToCall(CalledF) ||
        CalledF->hasDLLImportStorageClass())
      return;
    uint64_t &Count = Counts[std::make_pair(F, CalledF)];
    Count = SaturatingAdd(Count, NewCount);
```

- **L41**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L42**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L43**: Continues a multi-line argument list or initializer: `M.addModuleFlag(Module::Append, "CG Profile",`. / 继续一个多行参数列表或初始化器：`M.addModuleFlag(Module::Append, "CG Profile",`。
- **L44**: Executes call or statement centered on `MDTuple::getDistinct`. / 执行以 `MDTuple::getDistinct` 为核心的调用或语句。
- **L45**: Returns from the current function with `true`. / 以 `true` 从当前函数返回。
- **L46**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L47**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L48**: Continues a multi-line argument list or initializer: `static bool runCGProfilePass(Module &M, FunctionAnalysisManager &FAM,`. / 继续一个多行参数列表或初始化器：`static bool runCGProfilePass(Module &M, FunctionAnalysisManager &FAM,`。
- **L49**: Continues the surrounding expression or declaration: `bool InLTO) {`. / 继续构造周围的表达式或声明：`bool InLTO) {`。
- **L50**: Executes a standalone statement or declaration: `MapVector<std::pair<Function *, Function *>, uint64_t> Counts;`. / 执行一条独立语句或声明：`MapVector<std::pair<Function *, Function *>, uint64_t> Counts;`。
- **L51**: Executes a standalone statement or declaration: `InstrProfSymtab Symtab;`. / 执行一条独立语句或声明：`InstrProfSymtab Symtab;`。
- **L52**: Continues a multi-line argument list or initializer: `auto UpdateCounts = [&](TargetTransformInfo &TTI, Function *F,`. / 继续一个多行参数列表或初始化器：`auto UpdateCounts = [&](TargetTransformInfo &TTI, Function *F,`。
- **L53**: Continues the surrounding expression or declaration: `Function *CalledF, uint64_t NewCount) {`. / 继续构造周围的表达式或声明：`Function *CalledF, uint64_t NewCount) {`。
- **L54**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L55**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L56**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L57**: Continues the surrounding expression or declaration: `CalledF->hasDLLImportStorageClass())`. / 继续构造周围的表达式或声明：`CalledF->hasDLLImportStorageClass())`。
- **L58**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L59**: Executes call or statement centered on `Counts[std::make_pair`. / 执行以 `Counts[std::make_pair` 为核心的调用或语句。
- **L60**: Executes call or statement centered on `SaturatingAdd`. / 执行以 `SaturatingAdd` 为核心的调用或语句。

### Lines 61-80

```cpp
  };
  // Ignore error here.  Indirect calls are ignored if this fails.
  (void)(bool)Symtab.create(M, InLTO);
  for (auto &F : M) {
    // Avoid extra cost of running passes for BFI when the function doesn't have
    // entry count.
    if (F.isDeclaration() || !F.getEntryCount())
      continue;
    auto &BFI = FAM.getResult<BlockFrequencyAnalysis>(F);
    if (BFI.getEntryFreq() == BlockFrequency(0))
      continue;
    TargetTransformInfo &TTI = FAM.getResult<TargetIRAnalysis>(F);
    for (auto &BB : F) {
      std::optional<uint64_t> BBCount = BFI.getBlockProfileCount(&BB);
      if (!BBCount)
        continue;
      for (auto &I : BB) {
        CallBase *CB = dyn_cast<CallBase>(&I);
        if (!CB)
          continue;
```

- **L61**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L62**: Comment documents the nearby logic or transformation intent: `Ignore error here.  Indirect calls are ignored if this fails.`. / 注释说明了附近代码的逻辑或变换意图：`Ignore error here.  Indirect calls are ignored if this fails.`。
- **L63**: Executes call or statement centered on `statement`. / 执行以 `statement` 为核心的调用或语句。
- **L64**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L65**: Comment documents the nearby logic or transformation intent: `Avoid extra cost of running passes for BFI when the function doesn't have`. / 注释说明了附近代码的逻辑或变换意图：`Avoid extra cost of running passes for BFI when the function doesn't have`。
- **L66**: Comment documents the nearby logic or transformation intent: `entry count.`. / 注释说明了附近代码的逻辑或变换意图：`entry count.`。
- **L67**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L68**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L69**: Executes call or statement centered on `FAM.getResult<BlockFrequencyAnalysis>`. / 执行以 `FAM.getResult<BlockFrequencyAnalysis>` 为核心的调用或语句。
- **L70**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L71**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L72**: Executes call or statement centered on `FAM.getResult<TargetIRAnalysis>`. / 执行以 `FAM.getResult<TargetIRAnalysis>` 为核心的调用或语句。
- **L73**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L74**: Initializes variable `BBCount` from the right-hand expression. / 使用右侧表达式初始化变量 `BBCount`。
- **L75**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L76**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L77**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L78**: Executes call or statement centered on `dyn_cast<CallBase>`. / 执行以 `dyn_cast<CallBase>` 为核心的调用或语句。
- **L79**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L80**: Skips to the next loop iteration. / 跳到下一次循环迭代。

### Lines 81-100

```cpp
        if (CB->isIndirectCall()) {
          uint64_t TotalC;
          auto ValueData =
              getValueProfDataFromInst(*CB, IPVK_IndirectCallTarget, 8, TotalC);
          for (const auto &VD : ValueData)
            UpdateCounts(TTI, &F, Symtab.getFunction(VD.Value), VD.Count);
          continue;
        }
        UpdateCounts(TTI, &F, CB->getCalledFunction(), *BBCount);
      }
    }
  }

  return addModuleFlags(M, Counts);
}

PreservedAnalyses CGProfilePass::run(Module &M, ModuleAnalysisManager &MAM) {
  FunctionAnalysisManager &FAM =
      MAM.getResult<FunctionAnalysisManagerModuleProxy>(M).getManager();
  runCGProfilePass(M, FAM, InLTO);
```

- **L81**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L82**: Executes a standalone statement or declaration: `uint64_t TotalC;`. / 执行一条独立语句或声明：`uint64_t TotalC;`。
- **L83**: Continues the surrounding expression or declaration: `auto ValueData =`. / 继续构造周围的表达式或声明：`auto ValueData =`。
- **L84**: Executes call or statement centered on `getValueProfDataFromInst`. / 执行以 `getValueProfDataFromInst` 为核心的调用或语句。
- **L85**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L86**: Executes call or statement centered on `UpdateCounts`. / 执行以 `UpdateCounts` 为核心的调用或语句。
- **L87**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L88**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L89**: Executes call or statement centered on `UpdateCounts`. / 执行以 `UpdateCounts` 为核心的调用或语句。
- **L90**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L91**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L92**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L93**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L94**: Returns from the current function with `addModuleFlags(M, Counts)`. / 以 `addModuleFlags(M, Counts)` 从当前函数返回。
- **L95**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L96**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L97**: Starts a function, method, or lambda body: `PreservedAnalyses CGProfilePass::run(Module &M, ModuleAnalysisManager &MAM) {`. / 开始一个函数、方法或 lambda 的主体：`PreservedAnalyses CGProfilePass::run(Module &M, ModuleAnalysisManager &MAM) {`。
- **L98**: Continues the surrounding expression or declaration: `FunctionAnalysisManager &FAM =`. / 继续构造周围的表达式或声明：`FunctionAnalysisManager &FAM =`。
- **L99**: Executes call or statement centered on `MAM.getResult<FunctionAnalysisManagerModuleProxy>`. / 执行以 `MAM.getResult<FunctionAnalysisManagerModuleProxy>` 为核心的调用或语句。
- **L100**: Executes call or statement centered on `runCGProfilePass`. / 执行以 `runCGProfilePass` 为核心的调用或语句。

### Lines 101-103

```cpp

  return PreservedAnalyses::all();
}
```

- **L101**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L102**: Returns from the current function with `PreservedAnalyses::all()`. / 以 `PreservedAnalyses::all()` 从当前函数返回。
- **L103**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Instrumentation transform pipeline / Instrumentation 变换流水线**
- **Function-level traversal and updates / 函数级遍历与更新**
- **Module-wide coordination / 模块范围的协调**
- **Target-aware profitability decisions / 面向目标平台的收益判断**
- **Analysis preservation reporting / 分析保持情况报告**

## Dependencies / 依赖关系

- `llvm/Transforms/Instrumentation/CGProfile.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `llvm/ADT/MapVector.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/Analysis/BlockFrequencyInfo.h`: Provides analysis interfaces and cached results. / 提供分析接口与缓存结果。
- `llvm/Analysis/LazyBlockFrequencyInfo.h`: Provides analysis interfaces and cached results. / 提供分析接口与缓存结果。
- `llvm/Analysis/TargetTransformInfo.h`: Provides analysis interfaces and cached results. / 提供分析接口与缓存结果。
- `llvm/IR/Constants.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/MDBuilder.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Module.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/PassManager.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/ProfileData/InstrProf.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `llvm/Transforms/Utils/Instrumentation.h`: Provides shared transform utilities. / 提供共享的变换辅助工具。
- `optional`: Provides supporting declarations. / 提供所需的辅助声明。
