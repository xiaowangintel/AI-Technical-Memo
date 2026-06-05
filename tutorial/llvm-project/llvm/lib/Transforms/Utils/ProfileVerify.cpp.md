# ProfileVerify.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/ProfileVerify.cpp` | `llvm/lib/Transforms/Utils/ProfileVerify.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements verify profile info for testing within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 ProfileVerify 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

```cpp
//===- ProfileVerify.cpp - Verify profile info for testing ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/ProfileVerify.h"
#include "llvm/ADT/DynamicAPInt.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/Analysis/BranchProbabilityInfo.h"
#include "llvm/IR/Analysis.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PassManager.h"
#include "llvm/IR/ProfDataUtils.h"
#include "llvm/Support/BranchProbability.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as BranchProbabilityInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 BranchProbabilityInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 30-54

```cpp
using namespace llvm;
static cl::opt<int64_t>
    DefaultFunctionEntryCount("profcheck-default-function-entry-count",
                              cl::init(1000));
static cl::opt<bool>
    AnnotateSelect("profcheck-annotate-select", cl::init(true),
                   cl::desc("Also inject (if missing) and verify MD_prof for "
                            "`select` instructions"));
static cl::opt<bool>
    WeightsForTest("profcheck-weights-for-test", cl::init(false),
                   cl::desc("Generate weights with small values for tests."));

static cl::opt<uint32_t> SelectTrueWeight(
    "profcheck-default-select-true-weight", cl::init(2U),
    cl::desc("When annotating `select` instructions, this value will be used "
             "for the first ('true') case."));
static cl::opt<uint32_t> SelectFalseWeight(
    "profcheck-default-select-false-weight", cl::init(3U),
    cl::desc("When annotating `select` instructions, this value will be used "
             "for the second ('false') case."));
namespace {
class ProfileInjector {
  Function &F;
  FunctionAnalysisManager &FAM;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include ProfileInjector, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 ProfileInjector，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 55-82

```cpp
public:
  static const Instruction *
  getTerminatorBenefitingFromMDProf(const BasicBlock &BB) {
    if (succ_size(&BB) < 2)
      return nullptr;
    auto *Term = BB.getTerminator();
    return (isa<CondBrInst>(Term) || isa<SwitchInst>(Term) ||
            isa<IndirectBrInst>(Term) || isa<CallBrInst>(Term))
               ? Term
               : nullptr;
  }

  static Instruction *getTerminatorBenefitingFromMDProf(BasicBlock &BB) {
    return const_cast<Instruction *>(
        getTerminatorBenefitingFromMDProf(const_cast<const BasicBlock &>(BB)));
  }

  ProfileInjector(Function &F, FunctionAnalysisManager &FAM) : F(F), FAM(FAM) {}
  bool inject();
};

bool isAsmOnly(const Function &F) {
  if (!F.hasFnAttribute(Attribute::AttrKind::Naked))
    return false;
  for (const auto &BB : F)
    for (const auto &I : drop_end(BB)) {
      const auto *CB = dyn_cast<CallBase>(&I);
      if (!CB || !CB->isInlineAsm())
```
- EN: Core entities appearing here include getTerminatorBenefitingFromMDProf, isAsmOnly, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getTerminatorBenefitingFromMDProf, isAsmOnly，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 83-111

```cpp
        return false;
    }
  return true;
}

void emitProfileError(StringRef Msg, Function &F) {
  F.getContext().emitError("Profile verification failed for function '" +
                           F.getName() + "': " + Msg);
}

} // namespace

// FIXME: currently this injects only for terminators. Select isn't yet
// supported.
bool ProfileInjector::inject() {
  // skip purely asm functions
  if (isAsmOnly(F))
    return false;
  // Get whatever branch probability info can be derived from the given IR -
  // whether it has or not metadata. The main intention for this pass is to
  // ensure that other passes don't drop or "forget" to update MD_prof. We do
  // this as a mode in which lit tests would run. We want to avoid changing the
  // behavior of those tests. A pass may use BPI (or BFI, which is computed from
  // BPI). If no metadata is present, BPI is guesstimated by
  // BranchProbabilityAnalysis. The injector (this pass) only persists whatever
  // information the analysis provides, in other words, the pass being tested
  // will get the same BPI it does if the injector wasn't running.
  auto &BPI = FAM.getResult<BranchProbabilityAnalysis>(F);

```
- EN: Core entities appearing here include emitProfileError, inject, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 emitProfileError, inject，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 112-139

```cpp
  // Inject a function count if there's none. It's reasonable for a pass to
  // want to clear the MD_prof of a function with zero entry count. If the
  // original profile (iFDO or AFDO) is empty for a function, it's simpler to
  // require assigning it the 0-entry count explicitly than to mark every branch
  // as cold (we do want some explicit information in the spirit of what this
  // verifier wants to achieve - make dropping / corrupting MD_prof
  // unit-testable)
  if (!F.getEntryCount(/*AllowSynthetic=*/true))
    F.setEntryCount(DefaultFunctionEntryCount);
  // If there is an entry count that's 0, then don't bother injecting. We won't
  // verify these either.
  if (F.getEntryCount(/*AllowSynthetic=*/true)->getCount() == 0)
    return false;
  bool Changed = false;
  // Cycle through the weights list. If we didn't, tests with more than (say)
  // one conditional branch would have the same !prof metadata on all of them,
  // and numerically that may make for a poor unit test.
  uint32_t WeightsForTestOffset = 0;
  for (auto &BB : F) {
    if (AnnotateSelect) {
      for (auto &I : BB) {
        if (auto *SI = dyn_cast<SelectInst>(&I)) {
          if (SI->getCondition()->getType()->isVectorTy())
            continue;
          if (I.getMetadata(LLVMContext::MD_prof))
            continue;
          setBranchWeights(I, {SelectTrueWeight, SelectFalseWeight},
                           /*IsExpected=*/false);
```
- EN: This region continues the ProfileVerify implementation with local helper logic centered on Inject, MD_prof, AFDO, AllowSynthetic.
- CN: 这一段延续了 ProfileVerify 的主体实现，围绕 Inject, MD_prof, AFDO, AllowSynthetic 等局部辅助逻辑展开。

### Lines 140-167

```cpp
        }
      }
    }
    auto *Term = getTerminatorBenefitingFromMDProf(BB);
    if (!Term || Term->getMetadata(LLVMContext::MD_prof))
      continue;
    SmallVector<BranchProbability> Probs;

    SmallVector<uint32_t> Weights;
    Weights.reserve(Term->getNumSuccessors());
    if (WeightsForTest) {
      static const std::array Primes{3,  5,  7,  11, 13, 17, 19, 23, 29, 31,
                                     37, 41, 43, 47, 53, 59, 61, 67, 71};
      for (uint32_t I = 0, E = Term->getNumSuccessors(); I < E; ++I)
        Weights.emplace_back(
            Primes[(WeightsForTestOffset + I) % Primes.size()]);
      ++WeightsForTestOffset;
    } else {
      Probs.reserve(Term->getNumSuccessors());
      for (auto I = 0U, E = Term->getNumSuccessors(); I < E; ++I)
        Probs.emplace_back(BPI.getEdgeProbability(&BB, Term->getSuccessor(I)));

      assert(llvm::find_if(Probs,
                           [](const BranchProbability &P) {
                             return P.isUnknown();
                           }) == Probs.end() &&
             "All branch probabilities should be valid");
      const auto *FirstZeroDenominator =
```
- EN: This region continues the ProfileVerify implementation with local helper logic centered on Term, LLVMContext, MD_prof, SmallVector.
- CN: 这一段延续了 ProfileVerify 的主体实现，围绕 Term, LLVMContext, MD_prof, SmallVector 等局部辅助逻辑展开。

### Lines 168-195

```cpp
          find_if(Probs, [](const BranchProbability &P) {
            return P.getDenominator() == 0;
          });
      (void)FirstZeroDenominator;
      assert(FirstZeroDenominator == Probs.end());
      const auto *FirstNonZeroNumerator = find_if(
          Probs, [](const BranchProbability &P) { return !P.isZero(); });
      assert(FirstNonZeroNumerator != Probs.end());
      DynamicAPInt LCM(Probs[0].getDenominator());
      DynamicAPInt GCD(FirstNonZeroNumerator->getNumerator());
      for (const auto &Prob : drop_begin(Probs)) {
        if (!Prob.getNumerator())
          continue;
        LCM = llvm::lcm(LCM, DynamicAPInt(Prob.getDenominator()));
        GCD = llvm::gcd(GCD, DynamicAPInt(Prob.getNumerator()));
      }
      for (const auto &Prob : Probs) {
        DynamicAPInt W =
            (Prob.getNumerator() * LCM / GCD) / Prob.getDenominator();
        Weights.emplace_back(static_cast<uint32_t>((int64_t)W));
      }
    }
    setBranchWeights(*Term, Weights, /*IsExpected=*/false);
    Changed = true;
  }
  return Changed;
}

```
- EN: Core entities appearing here include find_if, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 find_if，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 196-221

```cpp
PreservedAnalyses ProfileInjectorPass::run(Function &F,
                                           FunctionAnalysisManager &FAM) {
  ProfileInjector PI(F, FAM);
  if (!PI.inject())
    return PreservedAnalyses::all();

  return PreservedAnalyses::none();
}

PreservedAnalyses ProfileVerifierPass::run(Module &M,
                                           ModuleAnalysisManager &MAM) {
  auto PopulateIgnoreList = [&](StringRef GVName) {
    if (const auto *CT = M.getGlobalVariable(GVName))
      if (CT->hasInitializer())
        if (const auto *CA =
                dyn_cast_if_present<ConstantArray>(CT->getInitializer()))
          for (const auto &Elt : CA->operands())
            if (const auto *CS = dyn_cast<ConstantStruct>(Elt))
              if (CS->getNumOperands() >= 2 && CS->getOperand(1))
                if (const auto *F = dyn_cast<Function>(
                        CS->getOperand(1)->stripPointerCasts()))
                  IgnoreList.insert(F);
  };
  PopulateIgnoreList("llvm.global_ctors");
  PopulateIgnoreList("llvm.global_dtors");

```
- EN: This region continues the ProfileVerify implementation with local helper logic centered on PreservedAnalyses, ProfileInjectorPass, Function, FunctionAnalysisManager.
- CN: 这一段延续了 ProfileVerify 的主体实现，围绕 PreservedAnalyses, ProfileInjectorPass, Function, FunctionAnalysisManager 等局部辅助逻辑展开。

### Lines 222-249

```cpp
  // expose the function-level run as public through a wrapper, so we can use
  // pass manager mechanisms dealing with declarations and with composing the
  // returned PreservedAnalyses values.
  struct Wrapper : OptionalPassInfoMixin<Wrapper> {
    ProfileVerifierPass &PVP;
    PreservedAnalyses run(Function &F, FunctionAnalysisManager &FAM) {
      return PVP.run(F, FAM);
    }
    explicit Wrapper(ProfileVerifierPass &PVP) : PVP(PVP) {}
  };

  return createModuleToFunctionPassAdaptor(Wrapper(*this)).run(M, MAM);
}

PreservedAnalyses ProfileVerifierPass::run(Function &F,
                                           FunctionAnalysisManager &FAM) {
  // skip purely asm functions
  if (isAsmOnly(F))
    return PreservedAnalyses::all();
  if (IgnoreList.contains(&F))
    return PreservedAnalyses::all();

  const auto EntryCount = F.getEntryCount(/*AllowSynthetic=*/true);
  if (!EntryCount) {
    auto *MD = F.getMetadata(LLVMContext::MD_prof);
    if (!MD || !isExplicitlyUnknownProfileMetadata(*MD)) {
      emitProfileError("function entry count missing (set to 0 if cold)", F);
      return PreservedAnalyses::all();
```
- EN: Core entities appearing here include Wrapper, run, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 Wrapper, run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 250-271

```cpp
    }
  } else if (EntryCount->getCount() == 0) {
    return PreservedAnalyses::all();
  }
  for (const auto &BB : F) {
    if (AnnotateSelect) {
      for (const auto &I : BB)
        if (auto *SI = dyn_cast<SelectInst>(&I)) {
          if (SI->getCondition()->getType()->isVectorTy())
            continue;
          if (I.getMetadata(LLVMContext::MD_prof))
            continue;
          emitProfileError("select annotation missing", F);
        }
    }
    if (const auto *Term =
            ProfileInjector::getTerminatorBenefitingFromMDProf(BB))
      if (!Term->getMetadata(LLVMContext::MD_prof))
        emitProfileError("branch annotation missing", F);
  }
  return PreservedAnalyses::all();
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `ProfileInjector, Wrapper, getTerminatorBenefitingFromMDProf, isAsmOnly, emitProfileError, inject, find_if, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`ProfileInjector, Wrapper, getTerminatorBenefitingFromMDProf, isAsmOnly, emitProfileError, inject, find_if, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `BranchProbabilityInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `BranchProbabilityInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `DefaultFunctionEntryCount, AnnotateSelect, WeightsForTest, SelectTrueWeight, SelectFalseWeight` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `DefaultFunctionEntryCount, AnnotateSelect, WeightsForTest, SelectTrueWeight, SelectFalseWeight` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/BranchProbabilityInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/BranchProbabilityInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Analysis.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/PassManager.h`, `llvm/IR/ProfDataUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Analysis.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/PassManager.h`, `llvm/IR/ProfDataUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/BranchProbability.h`, `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/ADT/DynamicAPInt.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/BranchProbability.h`, `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/ADT/DynamicAPInt.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `BranchProbabilityInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`BranchProbabilityInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
