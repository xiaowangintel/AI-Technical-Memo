# LowerExpectIntrinsic.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/LowerExpectIntrinsic.cpp` | `llvm/lib/Transforms/Scalar/LowerExpectIntrinsic.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements lower expect intrinsic within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 LowerExpectIntrinsic 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-35

```cpp
//===- LowerExpectIntrinsic.cpp - Lower expect intrinsic ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass lowers the 'expect' intrinsic to LLVM metadata.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/LowerExpectIntrinsic.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/ProfDataUtils.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils/MisExpect.h"

#include <cmath>

using namespace llvm;

#define DEBUG_TYPE "lower-expect-intrinsic"

STATISTIC(ExpectIntrinsicsHandled,
          "Number of 'expect' intrinsic instructions handled");

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 36-69

```cpp
// These default values are chosen to represent an extremely skewed outcome for
// a condition, but they leave some room for interpretation by later passes.
//
// If the documentation for __builtin_expect() was made explicit that it should
// only be used in extreme cases, we could make this ratio higher. As it stands,
// programmers may be using __builtin_expect() / llvm.expect to annotate that a
// branch is likely or unlikely to be taken.

// WARNING: these values are internal implementation detail of the pass.
// They should not be exposed to the outside of the pass, front-end codegen
// should emit @llvm.expect intrinsics instead of using these weights directly.
// Transforms should use TargetTransformInfo's getPredictableBranchThreshold().
static cl::opt<uint32_t> LikelyBranchWeight(
    "likely-branch-weight", cl::Hidden, cl::init(2000),
    cl::desc("Weight of the branch likely to be taken (default = 2000)"));
static cl::opt<uint32_t> UnlikelyBranchWeight(
    "unlikely-branch-weight", cl::Hidden, cl::init(1),
    cl::desc("Weight of the branch unlikely to be taken (default = 1)"));

static std::tuple<uint32_t, uint32_t>
getBranchWeight(Intrinsic::ID IntrinsicID, CallInst *CI, int BranchCount) {
  if (IntrinsicID == Intrinsic::expect) {
    // __builtin_expect
    return std::make_tuple(LikelyBranchWeight.getValue(),
                           UnlikelyBranchWeight.getValue());
  } else {
    // __builtin_expect_with_probability
    assert(CI->getNumOperands() >= 3 &&
           "expect with probability must have 3 arguments");
    auto *Confidence = cast<ConstantFP>(CI->getArgOperand(2));
    double TrueProb = Confidence->getValueAPF().convertToDouble();
    assert((TrueProb >= 0.0 && TrueProb <= 1.0) &&
           "probability value must be in the range [0.0, 1.0]");
    double FalseProb = (1.0 - TrueProb) / (BranchCount - 1);
```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include getBranchWeight, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 getBranchWeight，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。

### Lines 70-108

```cpp
    uint32_t LikelyBW = ceil((TrueProb * (double)(INT32_MAX - 1)) + 1.0);
    uint32_t UnlikelyBW = ceil((FalseProb * (double)(INT32_MAX - 1)) + 1.0);
    return std::make_tuple(LikelyBW, UnlikelyBW);
  }
}

static bool handleSwitchExpect(SwitchInst &SI) {
  CallInst *CI = dyn_cast<CallInst>(SI.getCondition());
  if (!CI)
    return false;

  Function *Fn = CI->getCalledFunction();
  if (!Fn || (Fn->getIntrinsicID() != Intrinsic::expect &&
              Fn->getIntrinsicID() != Intrinsic::expect_with_probability))
    return false;

  Value *ArgValue = CI->getArgOperand(0);
  ConstantInt *ExpectedValue = dyn_cast<ConstantInt>(CI->getArgOperand(1));
  if (!ExpectedValue)
    return false;

  SwitchInst::CaseHandle Case = *SI.findCaseValue(ExpectedValue);
  unsigned n = SI.getNumCases(); // +1 for default case.
  uint32_t LikelyBranchWeightVal, UnlikelyBranchWeightVal;
  std::tie(LikelyBranchWeightVal, UnlikelyBranchWeightVal) =
      getBranchWeight(Fn->getIntrinsicID(), CI, n + 1);

  SmallVector<uint32_t, 16> Weights(n + 1, UnlikelyBranchWeightVal);

  uint64_t Index = (Case == *SI.case_default()) ? 0 : Case.getCaseIndex() + 1;
  Weights[Index] = LikelyBranchWeightVal;

  misexpect::checkExpectAnnotations(SI, Weights, /*IsFrontend=*/true);

  SI.setCondition(ArgValue);
  setBranchWeights(SI, Weights, /*IsExpected=*/true);
  return true;
}

```
- EN: Core entities appearing here include handleSwitchExpect, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 handleSwitchExpect，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 109-142

```cpp
/// Handler for PHINodes that define the value argument to an
/// @llvm.expect call.
///
/// If the operand of the phi has a constant value and it 'contradicts'
/// with the expected value of phi def, then the corresponding incoming
/// edge of the phi is unlikely to be taken. Using that information,
/// the branch probability info for the originating branch can be inferred.
static void handlePhiDef(CallInst *Expect) {
  Value &Arg = *Expect->getArgOperand(0);
  ConstantInt *ExpectedValue = dyn_cast<ConstantInt>(Expect->getArgOperand(1));
  if (!ExpectedValue)
    return;
  const APInt &ExpectedPhiValue = ExpectedValue->getValue();
  bool ExpectedValueIsLikely = true;
  Function *Fn = Expect->getCalledFunction();
  // If the function is expect_with_probability, then we need to take the
  // probability into consideration. For example, in
  // expect.with.probability.i64(i64 %a, i64 1, double 0.0), the
  // "ExpectedValue" 1 is unlikely. This affects probability propagation later.
  if (Fn->getIntrinsicID() == Intrinsic::expect_with_probability) {
    auto *Confidence = cast<ConstantFP>(Expect->getArgOperand(2));
    double TrueProb = Confidence->getValueAPF().convertToDouble();
    ExpectedValueIsLikely = (TrueProb > 0.5);
  }

  // Walk up in backward a list of instructions that
  // have 'copy' semantics by 'stripping' the copies
  // until a PHI node or an instruction of unknown kind
  // is reached. Negation via xor is also handled.
  //
  //       C = PHI(...);
  //       B = C;
  //       A = B;
  //       D = __builtin_expect(A, 0);
```
- EN: Core entities appearing here include handlePhiDef, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 handlePhiDef，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 143-176

```cpp
  //
  Value *V = &Arg;
  SmallVector<Instruction *, 4> Operations;
  while (!isa<PHINode>(V)) {
    if (ZExtInst *ZExt = dyn_cast<ZExtInst>(V)) {
      V = ZExt->getOperand(0);
      Operations.push_back(ZExt);
      continue;
    }

    if (SExtInst *SExt = dyn_cast<SExtInst>(V)) {
      V = SExt->getOperand(0);
      Operations.push_back(SExt);
      continue;
    }

    BinaryOperator *BinOp = dyn_cast<BinaryOperator>(V);
    if (!BinOp || BinOp->getOpcode() != Instruction::Xor)
      return;

    ConstantInt *CInt = dyn_cast<ConstantInt>(BinOp->getOperand(1));
    if (!CInt)
      return;

    V = BinOp->getOperand(0);
    Operations.push_back(BinOp);
  }

  // Executes the recorded operations on input 'Value'.
  auto ApplyOperations = [&](const APInt &Value) {
    APInt Result = Value;
    for (auto *Op : llvm::reverse(Operations)) {
      switch (Op->getOpcode()) {
      case Instruction::Xor:
```
- EN: This region continues the LowerExpectIntrinsic implementation with local helper logic centered on Value, Arg, SmallVector, Instruction.
- CN: 这一段延续了 LowerExpectIntrinsic 的主体实现，围绕 Value, Arg, SmallVector, Instruction 等局部辅助逻辑展开。

### Lines 177-215

```cpp
        Result ^= cast<ConstantInt>(Op->getOperand(1))->getValue();
        break;
      case Instruction::ZExt:
        Result = Result.zext(Op->getType()->getIntegerBitWidth());
        break;
      case Instruction::SExt:
        Result = Result.sext(Op->getType()->getIntegerBitWidth());
        break;
      default:
        llvm_unreachable("Unexpected operation");
      }
    }
    return Result;
  };

  auto *PhiDef = cast<PHINode>(V);

  // Get the first dominating conditional branch of the operand
  // i's incoming block.
  auto GetDomConditional = [&](unsigned i) -> CondBrInst * {
    BasicBlock *BB = PhiDef->getIncomingBlock(i);
    if (CondBrInst *BI = dyn_cast<CondBrInst>(BB->getTerminator()))
      return BI;
    BB = BB->getSinglePredecessor();
    if (!BB)
      return nullptr;
    return dyn_cast<CondBrInst>(BB->getTerminator());
  };

  // Now walk through all Phi operands to find phi oprerands with values
  // conflicting with the expected phi output value. Any such operand
  // indicates the incoming edge to that operand is unlikely.
  for (unsigned i = 0, e = PhiDef->getNumIncomingValues(); i != e; ++i) {

    Value *PhiOpnd = PhiDef->getIncomingValue(i);
    ConstantInt *CI = dyn_cast<ConstantInt>(PhiOpnd);
    if (!CI)
      continue;

```
- EN: This region continues the LowerExpectIntrinsic implementation with local helper logic centered on Result, ConstantInt, Instruction, ZExt.
- CN: 这一段延续了 LowerExpectIntrinsic 的主体实现，围绕 Result, ConstantInt, Instruction, ZExt 等局部辅助逻辑展开。

### Lines 216-249

```cpp
    // Not an interesting case when IsUnlikely is false -- we can not infer
    // anything useful when:
    // (1) We expect some phi output and the operand value matches it, or
    // (2) We don't expect some phi output (i.e. the "ExpectedValue" has low
    //     probability) and the operand value doesn't match that.
    const APInt &CurrentPhiValue = ApplyOperations(CI->getValue());
    if (ExpectedValueIsLikely == (ExpectedPhiValue == CurrentPhiValue))
      continue;

    CondBrInst *BI = GetDomConditional(i);
    if (!BI)
      continue;

    MDBuilder MDB(PhiDef->getContext());

    // There are two situations in which an operand of the PhiDef comes
    // from a given successor of a branch instruction BI.
    // 1) When the incoming block of the operand is the successor block;
    // 2) When the incoming block is BI's enclosing block and the
    // successor is the PhiDef's enclosing block.
    //
    // Returns true if the operand which comes from OpndIncomingBB
    // comes from outgoing edge of BI that leads to Succ block.
    auto *OpndIncomingBB = PhiDef->getIncomingBlock(i);
    auto IsOpndComingFromSuccessor = [&](BasicBlock *Succ) {
      if (OpndIncomingBB == Succ)
        // If this successor is the incoming block for this
        // Phi operand, then this successor does lead to the Phi.
        return true;
      if (OpndIncomingBB == BI->getParent() && Succ == PhiDef->getParent())
        // Otherwise, if the edge is directly from the branch
        // to the Phi, this successor is the one feeding this
        // Phi operand.
        return true;
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 250-284

```cpp
      return false;
    };
    uint32_t LikelyBranchWeightVal, UnlikelyBranchWeightVal;
    std::tie(LikelyBranchWeightVal, UnlikelyBranchWeightVal) = getBranchWeight(
        Expect->getCalledFunction()->getIntrinsicID(), Expect, 2);
    if (!ExpectedValueIsLikely)
      std::swap(LikelyBranchWeightVal, UnlikelyBranchWeightVal);

    if (IsOpndComingFromSuccessor(BI->getSuccessor(1)))
      BI->setMetadata(LLVMContext::MD_prof,
                      MDB.createBranchWeights(LikelyBranchWeightVal,
                                              UnlikelyBranchWeightVal,
                                              /*IsExpected=*/true));
    else if (IsOpndComingFromSuccessor(BI->getSuccessor(0)))
      BI->setMetadata(LLVMContext::MD_prof,
                      MDB.createBranchWeights(UnlikelyBranchWeightVal,
                                              LikelyBranchWeightVal,
                                              /*IsExpected=*/true));
  }
}

// Handle both CondBrInst and SelectInst.
template <class BrSelInst> static bool handleBrSelExpect(BrSelInst &BSI) {

  // Handle non-optimized IR code like:
  //   %expval = call i64 @llvm.expect.i64(i64 %conv1, i64 1)
  //   %tobool = icmp ne i64 %expval, 0
  //   br i1 %tobool, label %if.then, label %if.end
  //
  // Or the following simpler case:
  //   %expval = call i1 @llvm.expect.i1(i1 %cmp, i1 1)
  //   br i1 %expval, label %if.then, label %if.end

  CallInst *CI;

```
- EN: Core entities appearing here include t, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 t，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 285-321

```cpp
  ICmpInst *CmpI = dyn_cast<ICmpInst>(BSI.getCondition());
  CmpInst::Predicate Predicate;
  ConstantInt *CmpConstOperand = nullptr;
  if (!CmpI) {
    CI = dyn_cast<CallInst>(BSI.getCondition());
    Predicate = CmpInst::ICMP_NE;
  } else {
    Predicate = CmpI->getPredicate();
    if (Predicate != CmpInst::ICMP_NE && Predicate != CmpInst::ICMP_EQ)
      return false;

    CmpConstOperand = dyn_cast<ConstantInt>(CmpI->getOperand(1));
    if (!CmpConstOperand)
      return false;
    CI = dyn_cast<CallInst>(CmpI->getOperand(0));
  }

  if (!CI)
    return false;

  uint64_t ValueComparedTo = 0;
  if (CmpConstOperand) {
    if (CmpConstOperand->getBitWidth() > 64)
      return false;
    ValueComparedTo = CmpConstOperand->getZExtValue();
  }

  Function *Fn = CI->getCalledFunction();
  if (!Fn || (Fn->getIntrinsicID() != Intrinsic::expect &&
              Fn->getIntrinsicID() != Intrinsic::expect_with_probability))
    return false;

  Value *ArgValue = CI->getArgOperand(0);
  ConstantInt *ExpectedValue = dyn_cast<ConstantInt>(CI->getArgOperand(1));
  if (!ExpectedValue)
    return false;

```
- EN: This region continues the LowerExpectIntrinsic implementation with local helper logic centered on ICmpInst, CmpI, BSI, CmpInst.
- CN: 这一段延续了 LowerExpectIntrinsic 的主体实现，围绕 ICmpInst, CmpI, BSI, CmpInst 等局部辅助逻辑展开。

### Lines 322-355

```cpp
  MDBuilder MDB(CI->getContext());
  MDNode *Node;

  uint32_t LikelyBranchWeightVal, UnlikelyBranchWeightVal;
  std::tie(LikelyBranchWeightVal, UnlikelyBranchWeightVal) =
      getBranchWeight(Fn->getIntrinsicID(), CI, 2);

  SmallVector<uint32_t, 4> ExpectedWeights;
  if ((ExpectedValue->getZExtValue() == ValueComparedTo) ==
      (Predicate == CmpInst::ICMP_EQ)) {
    Node = MDB.createBranchWeights(
        LikelyBranchWeightVal, UnlikelyBranchWeightVal, /*IsExpected=*/true);
    ExpectedWeights = {LikelyBranchWeightVal, UnlikelyBranchWeightVal};
  } else {
    Node = MDB.createBranchWeights(UnlikelyBranchWeightVal,
                                   LikelyBranchWeightVal, /*IsExpected=*/true);
    ExpectedWeights = {UnlikelyBranchWeightVal, LikelyBranchWeightVal};
  }

  if (CmpI)
    CmpI->setOperand(0, ArgValue);
  else
    BSI.setCondition(ArgValue);

  misexpect::checkFrontendInstrumentation(BSI, ExpectedWeights);

  BSI.setMetadata(LLVMContext::MD_prof, Node);

  return true;
}

static bool lowerExpectIntrinsic(Function &F) {
  bool Changed = false;

```
- EN: Core entities appearing here include lowerExpectIntrinsic, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 lowerExpectIntrinsic，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 356-393

```cpp
  for (BasicBlock &BB : F) {
    // Create "block_weights" metadata.
    if (CondBrInst *BI = dyn_cast<CondBrInst>(BB.getTerminator())) {
      if (handleBrSelExpect<CondBrInst>(*BI))
        ExpectIntrinsicsHandled++;
    } else if (SwitchInst *SI = dyn_cast<SwitchInst>(BB.getTerminator())) {
      if (handleSwitchExpect(*SI))
        ExpectIntrinsicsHandled++;
    }

    // Remove llvm.expect intrinsics. Iterate backwards in order
    // to process select instructions before the intrinsic gets
    // removed.
    for (Instruction &Inst : llvm::make_early_inc_range(llvm::reverse(BB))) {
      CallInst *CI = dyn_cast<CallInst>(&Inst);
      if (!CI) {
        if (SelectInst *SI = dyn_cast<SelectInst>(&Inst)) {
          if (handleBrSelExpect(*SI))
            ExpectIntrinsicsHandled++;
        }
        continue;
      }

      Function *Fn = CI->getCalledFunction();
      if (Fn && (Fn->getIntrinsicID() == Intrinsic::expect ||
                 Fn->getIntrinsicID() == Intrinsic::expect_with_probability)) {
        // Before erasing the llvm.expect, walk backward to find
        // phi that define llvm.expect's first arg, and
        // infer branch probability:
        handlePhiDef(CI);
        Value *Exp = CI->getArgOperand(0);
        CI->replaceAllUsesWith(Exp);
        CI->eraseFromParent();
        Changed = true;
      }
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 394-403

```cpp
  return Changed;
}

PreservedAnalyses LowerExpectIntrinsicPass::run(Function &F,
                                                FunctionAnalysisManager &) {
  if (lowerExpectIntrinsic(F))
    return PreservedAnalyses::none();

  return PreservedAnalyses::all();
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `getBranchWeight, handleSwitchExpect, handlePhiDef, t, lowerExpectIntrinsic` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`getBranchWeight, handleSwitchExpect, handlePhiDef, t, lowerExpectIntrinsic` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `LikelyBranchWeight, UnlikelyBranchWeight` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `LikelyBranchWeight, UnlikelyBranchWeight` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Scalar/LowerExpectIntrinsic.h`, `llvm/Transforms/Utils/MisExpect.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Scalar/LowerExpectIntrinsic.h`, `llvm/Transforms/Utils/MisExpect.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cmath` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cmath` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
