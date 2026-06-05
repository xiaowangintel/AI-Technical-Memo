# SpeculativeExecution.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/SpeculativeExecution.cpp` | `llvm/lib/Transforms/Scalar/SpeculativeExecution.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Speculative Execution within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 SpeculativeExecution 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-37

```cpp
//===- SpeculativeExecution.cpp ---------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass hoists instructions to enable speculative execution on
// targets where branches are expensive. This is aimed at GPUs. It
// currently works on simple if-then and if-then-else
// patterns.
//
// Removing branches is not the only motivation for this
// pass. E.g. consider this code and assume that there is no
// addressing mode for multiplying by sizeof(*a):
//
//   if (b > 0)
//     c = a[i + 1]
//   if (d > 0)
//     e = a[i + 2]
//
// turns into
//
//   p = &a[i + 1];
//   if (b > 0)
//     c = *p;
//   q = &a[i + 2];
//   if (d > 0)
//     e = *q;
//
// which could later be optimized to
//
//   r = &a[i];
//   if (b > 0)
//     c = r[1];
//   if (d > 0)
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 38-76

```cpp
//     e = r[2];
//
// Later passes sink back much of the speculated code that did not enable
// further optimization.
//
// This pass is more aggressive than the function SpeculativeyExecuteBB in
// SimplifyCFG. SimplifyCFG will not speculate if no selects are introduced and
// it will speculate at most one instruction. It also will not speculate if
// there is a value defined in the if-block that is only used in the then-block.
// These restrictions make sense since the speculation in SimplifyCFG seems
// aimed at introducing cheap selects, while this pass is intended to do more
// aggressive speculation while counting on later passes to either capitalize on
// that or clean it up.
//
// If the pass was created by calling
// createSpeculativeExecutionIfHasBranchDivergencePass or the
// -spec-exec-only-if-divergent-target option is present, this pass only has an
// effect on targets where TargetTransformInfo::hasBranchDivergence() is true;
// on other targets, it is a nop.
//
// This lets you include this pass unconditionally in the IR pass pipeline, but
// only enable it for relevant targets.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/SpeculativeExecution.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Operator.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Scalar.h"

using namespace llvm;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo, ValueTracking.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo, ValueTracking 等分析结果。

### Lines 77-115

```cpp
#define DEBUG_TYPE "speculative-execution"

// The risk that speculation will not pay off increases with the
// number of instructions speculated, so we put a limit on that.
static cl::opt<unsigned> SpecExecMaxSpeculationCost(
    "spec-exec-max-speculation-cost", cl::init(7), cl::Hidden,
    cl::desc("Speculative execution is not applied to basic blocks where "
             "the cost of the instructions to speculatively execute "
             "exceeds this limit."));

// Speculating just a few instructions from a larger block tends not
// to be profitable and this limit prevents that. A reason for that is
// that small basic blocks are more likely to be candidates for
// further optimization.
static cl::opt<unsigned> SpecExecMaxNotHoisted(
    "spec-exec-max-not-hoisted", cl::init(5), cl::Hidden,
    cl::desc("Speculative execution is not applied to basic blocks where the "
             "number of instructions that would not be speculatively executed "
             "exceeds this limit."));

static cl::opt<bool> SpecExecOnlyIfDivergentTarget(
    "spec-exec-only-if-divergent-target", cl::init(false), cl::Hidden,
    cl::desc("Speculative execution is applied only to targets with divergent "
             "branches, even if the pass was configured to apply only to all "
             "targets."));

namespace {

class SpeculativeExecutionLegacyPass : public FunctionPass {
public:
  static char ID;
  explicit SpeculativeExecutionLegacyPass(bool OnlyIfDivergentTarget = false)
      : FunctionPass(ID), OnlyIfDivergentTarget(OnlyIfDivergentTarget ||
                                                SpecExecOnlyIfDivergentTarget),
        Impl(OnlyIfDivergentTarget) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override;
  bool runOnFunction(Function &F) override;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include SpeculativeExecutionLegacyPass, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 SpeculativeExecutionLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 116-158

```cpp
  StringRef getPassName() const override {
    if (OnlyIfDivergentTarget)
      return "Speculatively execute instructions if target has divergent "
             "branches";
    return "Speculatively execute instructions";
  }

private:
  // Variable preserved purely for correct name printing.
  const bool OnlyIfDivergentTarget;

  SpeculativeExecutionPass Impl;
};
} // namespace

char SpeculativeExecutionLegacyPass::ID = 0;
INITIALIZE_PASS_BEGIN(SpeculativeExecutionLegacyPass, "speculative-execution",
                      "Speculatively execute instructions", false, false)
INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)
INITIALIZE_PASS_END(SpeculativeExecutionLegacyPass, "speculative-execution",
                    "Speculatively execute instructions", false, false)

void SpeculativeExecutionLegacyPass::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<TargetTransformInfoWrapperPass>();
  AU.addPreserved<GlobalsAAWrapperPass>();
  AU.setPreservesCFG();
}

bool SpeculativeExecutionLegacyPass::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;

  auto *TTI = &getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);
  return Impl.runImpl(F, TTI);
}

bool SpeculativeExecutionPass::runImpl(Function &F, TargetTransformInfo *TTI) {
  if (OnlyIfDivergentTarget && !TTI->hasBranchDivergence(&F)) {
    LLVM_DEBUG(dbgs() << "Not running SpeculativeExecution because "
                         "TTI->hasBranchDivergence() is false.\n");
    return false;
  }

```
- EN: Core entities appearing here include getAnalysisUsage, runOnFunction, runImpl, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo.
- CN: 此处出现的核心实体包括 getAnalysisUsage, runOnFunction, runImpl，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。

### Lines 159-195

```cpp
  this->TTI = TTI;
  bool Changed = false;
  for (auto& B : F) {
    Changed |= runOnBasicBlock(B);
  }
  return Changed;
}

bool SpeculativeExecutionPass::runOnBasicBlock(BasicBlock &B) {
  CondBrInst *BI = dyn_cast<CondBrInst>(B.getTerminator());
  if (!BI)
    return false;

  BasicBlock &Succ0 = *BI->getSuccessor(0);
  BasicBlock &Succ1 = *BI->getSuccessor(1);

  if (&B == &Succ0 || &B == &Succ1 || &Succ0 == &Succ1) {
    return false;
  }

  // Hoist from if-then (triangle).
  if (Succ0.getSinglePredecessor() != nullptr &&
      Succ0.getSingleSuccessor() == &Succ1) {
    return considerHoistingFromTo(Succ0, B);
  }

  // Hoist from if-else (triangle).
  if (Succ1.getSinglePredecessor() != nullptr &&
      Succ1.getSingleSuccessor() == &Succ0) {
    return considerHoistingFromTo(Succ1, B);
  }

  // Hoist from if-then-else (diamond), but only if it is equivalent to
  // an if-else or if-then due to one of the branches doing nothing.
  if (Succ0.getSinglePredecessor() != nullptr &&
      Succ1.getSinglePredecessor() != nullptr &&
      Succ1.getSingleSuccessor() != nullptr &&
```
- EN: Core entities appearing here include runOnBasicBlock, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 runOnBasicBlock，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 196-232

```cpp
      Succ1.getSingleSuccessor() != &B &&
      Succ1.getSingleSuccessor() == Succ0.getSingleSuccessor()) {
    // If a block has only one instruction, then that is a terminator
    // instruction so that the block does nothing. This does happen.
    if (Succ1.size() == 1) // equivalent to if-then
      return considerHoistingFromTo(Succ0, B);
    if (Succ0.size() == 1) // equivalent to if-else
      return considerHoistingFromTo(Succ1, B);
  }

  return false;
}

static InstructionCost ComputeSpeculationCost(const Instruction *I,
                                              const TargetTransformInfo &TTI) {
  switch (Operator::getOpcode(I)) {
    case Instruction::GetElementPtr:
    case Instruction::Add:
    case Instruction::Mul:
    case Instruction::And:
    case Instruction::Or:
    case Instruction::Select:
    case Instruction::Shl:
    case Instruction::Sub:
    case Instruction::LShr:
    case Instruction::AShr:
    case Instruction::Xor:
    case Instruction::ZExt:
    case Instruction::SExt:
    case Instruction::Call:
    case Instruction::BitCast:
    case Instruction::PtrToInt:
    case Instruction::PtrToAddr:
    case Instruction::IntToPtr:
    case Instruction::AddrSpaceCast:
    case Instruction::FPToUI:
    case Instruction::FPToSI:
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。

### Lines 233-269

```cpp
    case Instruction::UIToFP:
    case Instruction::SIToFP:
    case Instruction::FPExt:
    case Instruction::FPTrunc:
    case Instruction::FAdd:
    case Instruction::FSub:
    case Instruction::FMul:
    case Instruction::FDiv:
    case Instruction::FRem:
    case Instruction::FNeg:
    case Instruction::ICmp:
    case Instruction::FCmp:
    case Instruction::Trunc:
    case Instruction::Freeze:
    case Instruction::ExtractElement:
    case Instruction::InsertElement:
    case Instruction::ShuffleVector:
    case Instruction::ExtractValue:
    case Instruction::InsertValue:
      return TTI.getInstructionCost(I, TargetTransformInfo::TCK_SizeAndLatency);

    default:
      return InstructionCost::getInvalid(); // Disallow anything not explicitly
                                            // listed.
  }
}

// Do not hoist any debug info intrinsics.
// ...
// if (cond) {
//   x = y * z;
//   foo();
// }
// ...
// -------- Which then becomes:
// ...
// if.then:
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 270-306

```cpp
//   %x = mul i32 %y, %z
//   call void @llvm.dbg.value(%x, !"x", !DIExpression())
//   call void foo()
//
// SpeculativeExecution might decide to hoist the 'y * z' calculation
// out of the 'if' block, because it is more efficient that way, so the
// '%x = mul i32 %y, %z' moves to the block above. But it might also
// decide to hoist the 'llvm.dbg.value' call.
// This is incorrect, because even if we've moved the calculation of
// 'y * z', we should not see the value of 'x' change unless we
// actually go inside the 'if' block.

bool SpeculativeExecutionPass::considerHoistingFromTo(
    BasicBlock &FromBlock, BasicBlock &ToBlock) {
  SmallPtrSet<const Instruction *, 8> NotHoisted;
  auto HasNoUnhoistedInstr = [&NotHoisted](auto Values) {
    for (const Value *V : Values) {
      if (const auto *I = dyn_cast_or_null<Instruction>(V))
        if (NotHoisted.contains(I))
          return false;
    }
    return true;
  };
  auto AllPrecedingUsesFromBlockHoisted =
      [&HasNoUnhoistedInstr](const User *U) {
        return HasNoUnhoistedInstr(U->operand_values());
      };

  InstructionCost TotalSpeculationCost = 0;
  unsigned NotHoistedInstCount = 0;
  for (const auto &I : FromBlock) {
    const InstructionCost Cost = ComputeSpeculationCost(&I, *TTI);
    if (Cost.isValid() && isSafeToSpeculativelyExecute(&I) &&
        AllPrecedingUsesFromBlockHoisted(&I)) {
      TotalSpeculationCost += Cost;
      if (TotalSpeculationCost > SpecExecMaxSpeculationCost)
        return false;  // too much to hoist
```
- EN: Core entities appearing here include AllPrecedingUsesFromBlockHoisted, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 AllPrecedingUsesFromBlockHoisted，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 307-345

```cpp
    } else {
      NotHoistedInstCount++;
      if (NotHoistedInstCount > SpecExecMaxNotHoisted)
        return false; // too much left behind
      NotHoisted.insert(&I);
    }
  }

  for (auto I = FromBlock.begin(); I != FromBlock.end();) {
    // We have to increment I before moving Current as moving Current
    // changes the list that I is iterating through.
    auto Current = I;
    ++I;
    if (!NotHoisted.count(&*Current)) {
      Current->moveBefore(ToBlock.getTerminator()->getIterator());
      Current->dropLocation();
    }
  }
  return true;
}

FunctionPass *llvm::createSpeculativeExecutionPass() {
  return new SpeculativeExecutionLegacyPass();
}

FunctionPass *llvm::createSpeculativeExecutionIfHasBranchDivergencePass() {
  return new SpeculativeExecutionLegacyPass(/* OnlyIfDivergentTarget = */ true);
}

SpeculativeExecutionPass::SpeculativeExecutionPass(bool OnlyIfDivergentTarget)
    : OnlyIfDivergentTarget(OnlyIfDivergentTarget ||
                            SpecExecOnlyIfDivergentTarget) {}

PreservedAnalyses SpeculativeExecutionPass::run(Function &F,
                                                FunctionAnalysisManager &AM) {
  auto *TTI = &AM.getResult<TargetIRAnalysis>(F);

  bool Changed = runImpl(F, TTI);

```
- EN: This region continues the SpeculativeExecution implementation with local helper logic centered on NotHoistedInstCount, SpecExecMaxNotHoisted, NotHoisted, FromBlock.
- CN: 这一段延续了 SpeculativeExecution 的主体实现，围绕 NotHoistedInstCount, SpecExecMaxNotHoisted, NotHoisted, FromBlock 等局部辅助逻辑展开。

### Lines 346-361

```cpp
  if (!Changed)
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

void SpeculativeExecutionPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<SpeculativeExecutionPass> *>(this)->printPipeline(
      OS, MapClassName2PassName);
  OS << '<';
  if (OnlyIfDivergentTarget)
    OS << "only-if-divergent-target";
  OS << '>';
}
```
- EN: This region continues the SpeculativeExecution implementation with local helper logic centered on Changed, PreservedAnalyses, CFGAnalyses, SpeculativeExecutionPass.
- CN: 这一段延续了 SpeculativeExecution 的主体实现，围绕 Changed, PreservedAnalyses, CFGAnalyses, SpeculativeExecutionPass 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `SpeculativeExecutionLegacyPass, getAnalysisUsage, runOnFunction, runImpl, runOnBasicBlock, AllPrecedingUsesFromBlockHoisted` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`SpeculativeExecutionLegacyPass, getAnalysisUsage, runOnFunction, runImpl, runOnBasicBlock, AllPrecedingUsesFromBlockHoisted` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `TargetTransformInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `TargetTransformInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `SpecExecMaxSpeculationCost, SpecExecMaxNotHoisted, SpecExecOnlyIfDivergentTarget` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `SpecExecMaxSpeculationCost, SpecExecMaxNotHoisted, SpecExecOnlyIfDivergentTarget` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Instructions.h`, `llvm/IR/Operator.h`, `llvm/Transforms/Scalar/SpeculativeExecution.h`, `llvm/Transforms/Scalar.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Instructions.h`, `llvm/IR/Operator.h`, `llvm/Transforms/Scalar/SpeculativeExecution.h`, `llvm/Transforms/Scalar.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/InitializePasses.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/InitializePasses.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `TargetTransformInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`TargetTransformInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
