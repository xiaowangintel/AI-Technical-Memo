# PartiallyInlineLibCalls.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/PartiallyInlineLibCalls.cpp` | `llvm/lib/Transforms/Scalar/PartiallyInlineLibCalls.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements partially inline libcalls within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 PartiallyInlineLibCalls 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

```cpp
//===--- PartiallyInlineLibCalls.cpp - Partially inline libcalls ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass tries to partially inline the fast path of well-known library
// functions, such as using square-root instructions for cases where sqrt()
// does not need to set errno.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/PartiallyInlineLibCalls.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/DebugCounter.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include <optional>

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 29-51

```cpp
using namespace llvm;

namespace llvm {
extern cl::opt<bool> ProfcheckDisableMetadataFixes;
} // namespace llvm

#define DEBUG_TYPE "partially-inline-libcalls"

DEBUG_COUNTER(PILCounter, "partially-inline-libcalls-transform",
              "Controls transformations in partially-inline-libcalls");

static bool optimizeSQRT(CallInst *Call, Function *CalledFunc,
                         BasicBlock &CurrBB, Function::iterator &BB,
                         const TargetTransformInfo *TTI, DomTreeUpdater *DTU,
                         OptimizationRemarkEmitter *ORE) {
  // There is no need to change the IR, since backend will emit sqrt
  // instruction if the call has already been marked read-only.
  if (Call->onlyReadsMemory())
    return false;

  if (!DebugCounter::shouldExecute(PILCounter))
    return false;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, OptimizationRemarkEmitter, TargetTransformInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, OptimizationRemarkEmitter, TargetTransformInfo 等分析结果。

### Lines 52-76

```cpp
  // Do the following transformation:
  //
  // (before)
  // dst = sqrt(src)
  //
  // (after)
  // v0 = sqrt_noreadmem(src) # native sqrt instruction.
  // [if (v0 is a NaN) || if (src < 0)]
  //   v1 = sqrt(src)         # library call.
  // dst = phi(v0, v1)
  //

  Type *Ty = Call->getType();
  IRBuilder<> Builder(Call->getNextNode());

  // Split CurrBB right after the call, create a 'then' block (that branches
  // back to split-off tail of CurrBB) into which we'll insert a libcall.
  Instruction *LibCallTerm = SplitBlockAndInsertIfThen(
      Builder.getTrue(), Call->getNextNode(), /*Unreachable=*/false,
      /*BranchWeights*/ nullptr, DTU);

  auto *CurrBBTerm = cast<CondBrInst>(CurrBB.getTerminator());
  // We want an 'else' block though, not a 'then' block.
  CurrBBTerm->swapSuccessors();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 77-100

```cpp
  // Create phi that will merge results of either sqrt and replace all uses.
  BasicBlock *JoinBB = LibCallTerm->getSuccessor(0);
  JoinBB->setName(CurrBB.getName() + ".split");
  Builder.SetInsertPoint(JoinBB, JoinBB->begin());
  PHINode *Phi = Builder.CreatePHI(Ty, 2);
  Call->replaceAllUsesWith(Phi);

  // Finally, insert the libcall into 'else' block.
  BasicBlock *LibCallBB = LibCallTerm->getParent();
  LibCallBB->setName("call.sqrt");
  Builder.SetInsertPoint(LibCallTerm);
  Instruction *LibCall = Call->clone();
  Builder.Insert(LibCall);

  // Add memory(none) attribute, so that the backend can use a native sqrt
  // instruction for this call.
  Call->setDoesNotAccessMemory();

  // Insert a FP compare instruction and use it as the CurrBB branch condition.
  Builder.SetInsertPoint(CurrBBTerm);
  Value *FCmp = TTI->isFCmpOrdCheaperThanFCmpZero(Ty)
                    ? Builder.CreateFCmpORD(Call, Call)
                    : Builder.CreateFCmpOGE(Call->getOperand(0),
                                            ConstantFP::get(Ty, 0.0));
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 101-125

```cpp
  CurrBBTerm->setCondition(FCmp);
  if (!ProfcheckDisableMetadataFixes &&
      CurrBBTerm->getFunction()->getEntryCount()) {
    // Presume the quick path - where we don't call the library call - is the
    // frequent one
    MDBuilder MDB(CurrBBTerm->getContext());
    CurrBBTerm->setMetadata(LLVMContext::MD_prof,
                            MDB.createLikelyBranchWeights());
  }
  // Add phi operands.
  Phi->addIncoming(Call, &CurrBB);
  Phi->addIncoming(LibCall, LibCallBB);

  BB = JoinBB->getIterator();
  return true;
}

static bool runPartiallyInlineLibCalls(Function &F, TargetLibraryInfo *TLI,
                                       const TargetTransformInfo *TTI,
                                       DominatorTree *DT,
                                       OptimizationRemarkEmitter *ORE) {
  std::optional<DomTreeUpdater> DTU;
  if (DT)
    DTU.emplace(DT, DomTreeUpdater::UpdateStrategy::Lazy);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, OptimizationRemarkEmitter, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, OptimizationRemarkEmitter, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 126-152

```cpp
  bool Changed = false;

  Function::iterator CurrBB;
  for (Function::iterator BB = F.begin(), BE = F.end(); BB != BE;) {
    CurrBB = BB++;

    for (BasicBlock::iterator II = CurrBB->begin(), IE = CurrBB->end();
         II != IE; ++II) {
      CallInst *Call = dyn_cast<CallInst>(&*II);
      Function *CalledFunc;

      if (!Call || !(CalledFunc = Call->getCalledFunction()))
        continue;

      if (Call->isNoBuiltin() || Call->isStrictFP())
        continue;

      if (Call->isMustTailCall())
        continue;

      // Skip if function either has local linkage or is not a known library
      // function.
      LibFunc LF;
      if (CalledFunc->hasLocalLinkage() ||
          !TLI->getLibFunc(*CalledFunc, LF) || !TLI->has(LF))
        continue;

```
- EN: This region continues the PartiallyInlineLibCalls implementation with local helper logic centered on Changed, Function, CurrBB, BasicBlock.
- CN: 这一段延续了 PartiallyInlineLibCalls 的主体实现，围绕 Changed, Function, CurrBB, BasicBlock 等局部辅助逻辑展开。

### Lines 153-176

```cpp
      switch (LF) {
      case LibFunc_sqrtf:
      case LibFunc_sqrt:
        if (TTI->haveFastSqrt(Call->getType()) &&
            optimizeSQRT(Call, CalledFunc, *CurrBB, BB, TTI,
                         DTU ? &*DTU : nullptr, ORE))
          break;
        continue;
      default:
        continue;
      }

      Changed = true;
      break;
    }
  }

  return Changed;
}

PreservedAnalyses
PartiallyInlineLibCallsPass::run(Function &F, FunctionAnalysisManager &AM) {
  auto &TLI = AM.getResult<TargetLibraryAnalysis>(F);
  auto &TTI = AM.getResult<TargetIRAnalysis>(F);
```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 177-203

```cpp
  auto *DT = AM.getCachedResult<DominatorTreeAnalysis>(F);
  auto &ORE = AM.getResult<OptimizationRemarkEmitterAnalysis>(F);
  if (!runPartiallyInlineLibCalls(F, &TLI, &TTI, DT, &ORE))
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}

namespace {
class PartiallyInlineLibCallsLegacyPass : public FunctionPass {
public:
  static char ID;

  PartiallyInlineLibCallsLegacyPass() : FunctionPass(ID) {
    initializePartiallyInlineLibCallsLegacyPassPass(
        *PassRegistry::getPassRegistry());
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<TargetLibraryInfoWrapperPass>();
    AU.addRequired<TargetTransformInfoWrapperPass>();
    AU.addPreserved<DominatorTreeWrapperPass>();
    AU.addRequired<OptimizationRemarkEmitterWrapperPass>();
    FunctionPass::getAnalysisUsage(AU);
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include PartiallyInlineLibCallsLegacyPass, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 PartiallyInlineLibCallsLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo 等分析结果。

### Lines 204-227

```cpp
  bool runOnFunction(Function &F) override {
    if (skipFunction(F))
      return false;

    TargetLibraryInfo *TLI =
        &getAnalysis<TargetLibraryInfoWrapperPass>().getTLI(F);
    const TargetTransformInfo *TTI =
        &getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);
    DominatorTree *DT = nullptr;
    if (auto *DTWP = getAnalysisIfAvailable<DominatorTreeWrapperPass>())
      DT = &DTWP->getDomTree();
    auto *ORE = &getAnalysis<OptimizationRemarkEmitterWrapperPass>().getORE();
    return runPartiallyInlineLibCalls(F, TLI, TTI, DT, ORE);
  }
};
}

char PartiallyInlineLibCallsLegacyPass::ID = 0;
INITIALIZE_PASS_BEGIN(PartiallyInlineLibCallsLegacyPass,
                      "partially-inline-libcalls",
                      "Partially inline calls to library functions", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(TargetLibraryInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo 等分析结果。

### Lines 228-236

```cpp
INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(OptimizationRemarkEmitterWrapperPass)
INITIALIZE_PASS_END(PartiallyInlineLibCallsLegacyPass,
                    "partially-inline-libcalls",
                    "Partially inline calls to library functions", false, false)

FunctionPass *llvm::createPartiallyInlineLibCallsPass() {
  return new PartiallyInlineLibCallsLegacyPass();
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter, TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter, TargetTransformInfo 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `PartiallyInlineLibCallsLegacyPass, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PartiallyInlineLibCallsLegacyPass, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, DomTreeUpdater, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, DomTreeUpdater, OptimizationRemarkEmitter, TargetLibraryInfo, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/MDBuilder.h`, `llvm/Transforms/Scalar/PartiallyInlineLibCalls.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/MDBuilder.h`, `llvm/Transforms/Scalar/PartiallyInlineLibCalls.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/DebugCounter.h`, `llvm/InitializePasses.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/DebugCounter.h`, `llvm/InitializePasses.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DominatorTree`, `DomTreeUpdater`, `OptimizationRemarkEmitter`, `TargetLibraryInfo`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `DomTreeUpdater`, `OptimizationRemarkEmitter`, `TargetLibraryInfo`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
