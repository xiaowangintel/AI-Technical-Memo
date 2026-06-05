# MVETailPredication.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MVETailPredication.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MVETailPredication` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `MVETailPredication`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- MVETailPredication.cpp - MVE Tail Predication ------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Armv8.1m introduced MVE, M-Profile Vector Extension, and low-overhead
/// branches to help accelerate DSP applications. These two extensions,
/// combined with a new form of predication called tail-predication, can be used
/// to provide implicit vector predication within a low-overhead loop.
/// This is implicit because the predicate of active/inactive lanes is
/// calculated by hardware, and thus does not need to be explicitly passed
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-30
```cpp
/// to vector instructions. The instructions responsible for this are the
/// DLSTP and WLSTP instructions, which setup a tail-predicated loop and the
/// the total number of data elements processed by the loop. The loop-end
/// LETP instruction is responsible for decrementing and setting the remaining
/// elements to be processed and generating the mask of active lanes.
///
/// The HardwareLoops pass inserts intrinsics identifying loops that the
/// backend will attempt to convert into a low-overhead loop. The vectorizer is
/// responsible for generating a vectorized loop in which the lanes are
/// predicated upon an get.active.lane.mask intrinsic. This pass looks at these
/// get.active.lane.mask intrinsic and attempts to convert them to VCTP
/// instructions. This will be picked up by the ARM Low-overhead loop pass later
/// in the backend, which performs the final transformation to a DLSTP or WLSTP
/// tail-predicated loop.
//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 31-31
```cpp
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 33-47
```cpp
#include "ARM.h"
#include "ARMSubtarget.h"
#include "ARMTargetTransformInfo.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/LoopPass.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/ScalarEvolutionExpressions.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicsARM.h"
#include "llvm/Support/Debug.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 48-51
```cpp
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Local.h"
#include "llvm/Transforms/Utils/LoopUtils.h"
#include "llvm/Transforms/Utils/ScalarEvolutionExpander.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 53-53
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 55-56
```cpp
#define DEBUG_TYPE "mve-tail-predication"
#define DESC "Transform predicated vector loops to use MVE tail predication"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 58-72
```cpp
cl::opt<TailPredication::Mode> EnableTailPredication(
   "tail-predication", cl::desc("MVE tail-predication pass options"),
   cl::init(TailPredication::Enabled),
   cl::values(clEnumValN(TailPredication::Disabled, "disabled",
                         "Don't tail-predicate loops"),
              clEnumValN(TailPredication::EnabledNoReductions,
                         "enabled-no-reductions",
                         "Enable tail-predication, but not for reduction loops"),
              clEnumValN(TailPredication::Enabled,
                         "enabled",
                         "Enable tail-predication, including reduction loops"),
              clEnumValN(TailPredication::ForceEnabledNoReductions,
                         "force-enabled-no-reductions",
                         "Enable tail-predication, but not for reduction loops, "
                         "and force this which might be unsafe"),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 73-76
```cpp
              clEnumValN(TailPredication::ForceEnabled,
                         "force-enabled",
                         "Enable tail-predication, including reduction loops, "
                         "and force this which might be unsafe")));
```
- EN: Declares `clEnumValN`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clEnumValN`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-79
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 81-86
```cpp
class MVETailPredication : public LoopPass {
  SmallVector<IntrinsicInst*, 4> MaskedInsts;
  Loop *L = nullptr;
  ScalarEvolution *SE = nullptr;
  TargetTransformInfo *TTI = nullptr;
  const ARMSubtarget *ST = nullptr;
```
- EN: Declares `MVETailPredication`, packaging target-specific state and APIs around `MVETailPredication`.
- CN: 这里声明 `MVETailPredication`，把与 `MVETailPredication` 相关的目标特定状态和 API 组织在一起。

### Lines 88-89
```cpp
public:
  static char ID;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 91-91
```cpp
  MVETailPredication() : LoopPass(ID) { }
```
- EN: Implements `MVETailPredication`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETailPredication`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-100
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<ScalarEvolutionWrapperPass>();
    AU.addRequired<LoopInfoWrapperPass>();
    AU.addRequired<TargetPassConfig>();
    AU.addRequired<TargetTransformInfoWrapperPass>();
    AU.addPreserved<LoopInfoWrapperPass>();
    AU.setPreservesCFG();
  }
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 102-102
```cpp
  bool runOnLoop(Loop *L, LPPassManager&) override;
```
- EN: Declares `runOnLoop`, a pass-entry routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnLoop`，它是一个围绕目标相关状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 104-107
```cpp
private:
  /// Perform the relevant checks on the loop and convert active lane masks if
  /// possible.
  bool TryConvertActiveLaneMask(Value *TripCount);
```
- EN: Declares `TryConvertActiveLaneMask`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `TryConvertActiveLaneMask`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 109-113
```cpp
  /// Perform several checks on the arguments of @llvm.get.active.lane.mask
  /// intrinsic. E.g., check that the loop induction variable and the element
  /// count are of the form we expect, and also perform overflow checks for
  /// the new expressions that are created.
  const SCEV *IsSafeActiveMask(IntrinsicInst *ActiveLaneMask, Value *TripCount);
```
- EN: Declares `IsSafeActiveMask`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `IsSafeActiveMask`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 115-117
```cpp
  /// Insert the intrinsic to represent the effect of tail predication.
  void InsertVCTPIntrinsic(IntrinsicInst *ActiveLaneMask, Value *Start);
};
```
- EN: Declares `InsertVCTPIntrinsic`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `InsertVCTPIntrinsic`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 119-119
```cpp
} // end namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 121-123
```cpp
bool MVETailPredication::runOnLoop(Loop *L, LPPassManager&) {
  if (skipLoop(L) || !EnableTailPredication)
    return false;
```
- EN: Implements `MVETailPredication::runOnLoop`, a pass-entry routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETailPredication::runOnLoop`，它是一个围绕目标相关状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 125-132
```cpp
  MaskedInsts.clear();
  Function &F = *L->getHeader()->getParent();
  auto &TPC = getAnalysis<TargetPassConfig>();
  auto &TM = TPC.getTM<TargetMachine>();
  ST = &TM.getSubtarget<ARMSubtarget>(F);
  TTI = &getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);
  SE = &getAnalysis<ScalarEvolutionWrapperPass>().getSE();
  this->L = L;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 134-139
```cpp
  // The MVE and LOB extensions are combined to enable tail-predication, but
  // there's nothing preventing us from generating VCTP instructions for v8.1m.
  if (!ST->hasMVEIntegerOps() || !ST->hasV8_1MMainlineOps()) {
    LLVM_DEBUG(dbgs() << "ARM TP: Not a v8.1m.main+mve target.\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 141-143
```cpp
  BasicBlock *Preheader = L->getLoopPreheader();
  if (!Preheader)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 145-149
```cpp
  auto FindLoopIterations = [](BasicBlock *BB) -> IntrinsicInst* {
    for (auto &I : *BB) {
      auto *Call = dyn_cast<IntrinsicInst>(&I);
      if (!Call)
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 151-157
```cpp
      Intrinsic::ID ID = Call->getIntrinsicID();
      if (ID == Intrinsic::start_loop_iterations ||
          ID == Intrinsic::test_start_loop_iterations)
        return cast<IntrinsicInst>(&I);
    }
    return nullptr;
  };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 159-160
```cpp
  // Look for the hardware loop intrinsic that sets the iteration count.
  IntrinsicInst *Setup = FindLoopIterations(Preheader);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 162-169
```cpp
  // The test.set iteration could live in the pre-preheader.
  if (!Setup) {
    if (!Preheader->getSinglePredecessor())
      return false;
    Setup = FindLoopIterations(Preheader->getSinglePredecessor());
    if (!Setup)
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 171-171
```cpp
  LLVM_DEBUG(dbgs() << "ARM TP: Running on Loop: " << *L << *Setup << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 173-173
```cpp
  bool Changed = TryConvertActiveLaneMask(Setup->getArgOperand(0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 175-176
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 178-192
```cpp
// The active lane intrinsic has this form:
//
//    @llvm.get.active.lane.mask(IV, TC)
//
// Here we perform checks that this intrinsic behaves as expected,
// which means:
//
// 1) Check that the TripCount (TC) belongs to this loop (originally).
// 2) The element count (TC) needs to be sufficiently large that the decrement
//    of element counter doesn't overflow, which means that we need to prove:
//        ceil(ElementCount / VectorWidth) >= TripCount
//    by rounding up ElementCount up:
//        ((ElementCount + (VectorWidth - 1)) / VectorWidth
//    and evaluate if expression isKnownNonNegative:
//        (((ElementCount + (VectorWidth - 1)) / VectorWidth) - TripCount
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 193-199
```cpp
// 3) The IV must be an induction phi with an increment equal to the
//    vector width.
const SCEV *MVETailPredication::IsSafeActiveMask(IntrinsicInst *ActiveLaneMask,
                                                 Value *TripCount) {
  bool ForceTailPredication =
    EnableTailPredication == TailPredication::ForceEnabledNoReductions ||
    EnableTailPredication == TailPredication::ForceEnabled;
```
- EN: Implements `MVETailPredication::IsSafeActiveMask`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETailPredication::IsSafeActiveMask`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 201-204
```cpp
  Value *ElemCount = ActiveLaneMask->getOperand(1);
  bool Changed = false;
  if (!L->makeLoopInvariant(ElemCount, Changed))
    return nullptr;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 206-213
```cpp
  const SCEV *EC = SE->getSCEV(ElemCount);
  const SCEV *TC = SE->getSCEV(TripCount);
  int VectorWidth =
      cast<FixedVectorType>(ActiveLaneMask->getType())->getNumElements();
  if (VectorWidth != 2 && VectorWidth != 4 && VectorWidth != 8 &&
      VectorWidth != 16)
    return nullptr;
  ConstantInt *ConstElemCount = nullptr;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 215-221
```cpp
  // 1) Smoke tests that the original scalar loop TripCount (TC) belongs to
  // this loop.  The scalar tripcount corresponds the number of elements
  // processed by the loop, so we will refer to that from this point on.
  if (!SE->isLoopInvariant(EC, L)) {
    LLVM_DEBUG(dbgs() << "ARM TP: element count must be loop invariant.\n");
    return nullptr;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 223-230
```cpp
  // 2) Find out if IV is an induction phi. Note that we can't use Loop
  // helpers here to get the induction variable, because the hardware loop is
  // no longer in loopsimplify form, and also the hwloop intrinsic uses a
  // different counter. Using SCEV, we check that the induction is of the
  // form i = i + 4, where the increment must be equal to the VectorWidth.
  auto *IV = ActiveLaneMask->getOperand(0);
  const SCEV *IVExpr = SE->getSCEV(IV);
  auto *AddExpr = dyn_cast<SCEVAddRecExpr>(IVExpr);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-246
```cpp
  if (!AddExpr) {
    LLVM_DEBUG(dbgs() << "ARM TP: induction not an add expr: "; IVExpr->dump());
    return nullptr;
  }
  // Check that this AddRec is associated with this loop.
  if (AddExpr->getLoop() != L) {
    LLVM_DEBUG(dbgs() << "ARM TP: phi not part of this loop\n");
    return nullptr;
  }
  auto *Step = dyn_cast<SCEVConstant>(AddExpr->getOperand(1));
  if (!Step) {
    LLVM_DEBUG(dbgs() << "ARM TP: induction step is not a constant: ";
               AddExpr->getOperand(1)->dump());
    return nullptr;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 247-252
```cpp
  auto StepValue = Step->getValue()->getSExtValue();
  if (VectorWidth != StepValue) {
    LLVM_DEBUG(dbgs() << "ARM TP: Step value " << StepValue
                      << " doesn't match vector width " << VectorWidth << "\n");
    return nullptr;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 254-260
```cpp
  if ((ConstElemCount = dyn_cast<ConstantInt>(ElemCount))) {
    ConstantInt *TC = dyn_cast<ConstantInt>(TripCount);
    if (!TC) {
      LLVM_DEBUG(dbgs() << "ARM TP: Constant tripcount expected in "
                           "set.loop.iterations\n");
      return nullptr;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 262-268
```cpp
    // Calculate 2 tripcount values and check that they are consistent with
    // each other. The TripCount for a predicated vector loop body is
    // ceil(ElementCount/Width), or floor((ElementCount+Width-1)/Width) as we
    // work it out here.
    uint64_t TC1 = TC->getZExtValue();
    uint64_t TC2 =
        (ConstElemCount->getZExtValue() + VectorWidth - 1) / VectorWidth;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 270-284
```cpp
    // If the tripcount values are inconsistent, we can't insert the VCTP and
    // trigger tail-predication; keep the intrinsic as a get.active.lane.mask
    // and legalize this.
    if (TC1 != TC2) {
      LLVM_DEBUG(dbgs() << "ARM TP: inconsistent constant tripcount values: "
                 << TC1 << " from set.loop.iterations, and "
                 << TC2 << " from get.active.lane.mask\n");
      return nullptr;
    }
  } else if (!ForceTailPredication) {
    // 3) We need to prove that the sub expression that we create in the
    // tail-predicated loop body, which calculates the remaining elements to be
    // processed, is non-negative, i.e. it doesn't overflow:
    //
    //   ((ElementCount + VectorWidth - 1) / VectorWidth) - TripCount >= 0
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 285-298
```cpp
    //
    // This is true if:
    //
    //    TripCount == (ElementCount + VectorWidth - 1) / VectorWidth
    //
    // which what we will be using here.
    //
    const SCEV *VW =
        SE->getSCEV(ConstantInt::get(TripCount->getType(), VectorWidth));
    // ElementCount + (VW-1):
    const SCEV *Start = AddExpr->getStart();
    const SCEV *ECPlusVWMinus1 = SE->getAddExpr(
        EC,
        SE->getSCEV(ConstantInt::get(TripCount->getType(), VectorWidth - 1)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 300-301
```cpp
    // Ceil = ElementCount + (VW-1) / VW
    const SCEV *Ceil = SE->getUDivExpr(ECPlusVWMinus1, VW);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 303-313
```cpp
    // Prevent unused variable warnings with TC
    (void)TC;
    LLVM_DEBUG({
      dbgs() << "ARM TP: Analysing overflow behaviour for:\n";
      dbgs() << "ARM TP: - TripCount = " << *TC << "\n";
      dbgs() << "ARM TP: - ElemCount = " << *EC << "\n";
      dbgs() << "ARM TP: - Start = " << *Start << "\n";
      dbgs() << "ARM TP: - BETC = " << *SE->getBackedgeTakenCount(L) << "\n";
      dbgs() << "ARM TP: - VecWidth =  " << VectorWidth << "\n";
      dbgs() << "ARM TP: - (ElemCount+VW-1) / VW = " << *Ceil << "\n";
    });
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 315-329
```cpp
    // As an example, almost all the tripcount expressions (produced by the
    // vectoriser) look like this:
    //
    //   TC = ((-4 + (4 * ((3 + %N) /u 4))<nuw> - start) /u 4)
    //
    // and "ElementCount + (VW-1) / VW":
    //
    //   Ceil = ((3 + %N) /u 4)
    //
    // Check for equality of TC and Ceil by calculating SCEV expression
    // TC - Ceil and test it for zero.
    //
    const SCEV *Div = SE->getUDivExpr(
        SE->getAddExpr(SE->getMulExpr(Ceil, VW), SE->getNegativeSCEV(VW),
                       SE->getNegativeSCEV(Start)),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 330-332
```cpp
        VW);
    const SCEV *Sub = SE->getMinusSCEV(SE->getBackedgeTakenCount(L), Div);
    LLVM_DEBUG(dbgs() << "ARM TP: - Sub       = "; Sub->dump());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 334-338
```cpp
    // Use context sensitive facts about the path to the loop to refine.  This
    // comes up as the backedge taken count can incorporate context sensitive
    // reasoning, and our RHS just above doesn't.
    Sub = SE->applyLoopGuards(Sub, L);
    LLVM_DEBUG(dbgs() << "ARM TP: - (Guarded) = "; Sub->dump());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 340-344
```cpp
    if (!Sub->isZero()) {
      LLVM_DEBUG(dbgs() << "ARM TP: possible overflow in sub expression.\n");
      return nullptr;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 346-360
```cpp
  // Check that the start value is a multiple of the VectorWidth.
  // TODO: This could do with a method to check if the scev is a multiple of
  // VectorWidth. For the moment we just check for constants, muls and unknowns
  // (which use MaskedValueIsZero and seems to be the most common).
  if (auto *BaseC = dyn_cast<SCEVConstant>(AddExpr->getStart())) {
    if (BaseC->getAPInt().urem(VectorWidth) == 0)
      return SE->getMinusSCEV(EC, BaseC);
  } else if (auto *BaseV = dyn_cast<SCEVUnknown>(AddExpr->getStart())) {
    Type *Ty = BaseV->getType();
    APInt Mask = APInt::getLowBitsSet(Ty->getPrimitiveSizeInBits(),
                                      Log2_64(VectorWidth));
    if (MaskedValueIsZero(BaseV->getValue(), Mask,
                          L->getHeader()->getDataLayout()))
      return SE->getMinusSCEV(EC, BaseV);
  } else if (auto *BaseMul = dyn_cast<SCEVMulExpr>(AddExpr->getStart())) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 361-367
```cpp
    if (auto *BaseC = dyn_cast<SCEVConstant>(BaseMul->getOperand(0)))
      if (BaseC->getAPInt().urem(VectorWidth) == 0)
        return SE->getMinusSCEV(EC, BaseC);
    if (auto *BaseC = dyn_cast<SCEVConstant>(BaseMul->getOperand(1)))
      if (BaseC->getAPInt().urem(VectorWidth) == 0)
        return SE->getMinusSCEV(EC, BaseC);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 369-373
```cpp
  LLVM_DEBUG(
      dbgs() << "ARM TP: induction base is not know to be a multiple of VF: "
             << *AddExpr->getOperand(0) << "\n");
  return nullptr;
}
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 375-381
```cpp
void MVETailPredication::InsertVCTPIntrinsic(IntrinsicInst *ActiveLaneMask,
                                             Value *Start) {
  IRBuilder<> Builder(L->getLoopPreheader()->getTerminator());
  Module *M = L->getHeader()->getModule();
  Type *Ty = IntegerType::get(M->getContext(), 32);
  unsigned VectorWidth =
      cast<FixedVectorType>(ActiveLaneMask->getType())->getNumElements();
```
- EN: Implements `MVETailPredication::InsertVCTPIntrinsic`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETailPredication::InsertVCTPIntrinsic`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 383-386
```cpp
  // Insert a phi to count the number of elements processed by the loop.
  Builder.SetInsertPoint(L->getHeader(), L->getHeader()->getFirstNonPHIIt());
  PHINode *Processed = Builder.CreatePHI(Ty, 2);
  Processed->addIncoming(Start, L->getLoopPreheader());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 388-391
```cpp
  // Replace @llvm.get.active.mask() with the ARM specific VCTP intrinsic, and
  // thus represent the effect of tail predication.
  Builder.SetInsertPoint(ActiveLaneMask);
  ConstantInt *Factor = ConstantInt::get(cast<IntegerType>(Ty), VectorWidth);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 393-403
```cpp
  Intrinsic::ID VCTPID;
  switch (VectorWidth) {
  default:
    llvm_unreachable("unexpected number of lanes");
  case 2:  VCTPID = Intrinsic::arm_mve_vctp64; break;
  case 4:  VCTPID = Intrinsic::arm_mve_vctp32; break;
  case 8:  VCTPID = Intrinsic::arm_mve_vctp16; break;
  case 16: VCTPID = Intrinsic::arm_mve_vctp8; break;
  }
  Value *VCTPCall = Builder.CreateIntrinsic(VCTPID, Processed);
  ActiveLaneMask->replaceAllUsesWith(VCTPCall);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 405-412
```cpp
  // Add the incoming value to the new phi.
  // TODO: This add likely already exists in the loop.
  Value *Remaining = Builder.CreateSub(Processed, Factor);
  Processed->addIncoming(Remaining, L->getLoopLatch());
  LLVM_DEBUG(dbgs() << "ARM TP: Insert processed elements phi: "
             << *Processed << "\n"
             << "ARM TP: Inserted VCTP: " << *VCTPCall << "\n");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 414-420
```cpp
bool MVETailPredication::TryConvertActiveLaneMask(Value *TripCount) {
  SmallVector<IntrinsicInst *, 4> ActiveLaneMasks;
  for (auto *BB : L->getBlocks())
    for (auto &I : *BB)
      if (auto *Int = dyn_cast<IntrinsicInst>(&I))
        if (Int->getIntrinsicID() == Intrinsic::get_active_lane_mask)
          ActiveLaneMasks.push_back(Int);
```
- EN: Implements `MVETailPredication::TryConvertActiveLaneMask`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVETailPredication::TryConvertActiveLaneMask`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 422-423
```cpp
  if (ActiveLaneMasks.empty())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 425-425
```cpp
  LLVM_DEBUG(dbgs() << "ARM TP: Found predicated vector loop.\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 427-429
```cpp
  for (auto *ActiveLaneMask : ActiveLaneMasks) {
    LLVM_DEBUG(dbgs() << "ARM TP: Found active lane mask: "
                      << *ActiveLaneMask << "\n");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 431-443
```cpp
    const SCEV *StartSCEV = IsSafeActiveMask(ActiveLaneMask, TripCount);
    if (!StartSCEV) {
      LLVM_DEBUG(dbgs() << "ARM TP: Not safe to insert VCTP.\n");
      return false;
    }
    LLVM_DEBUG(dbgs() << "ARM TP: Safe to insert VCTP. Start is " << *StartSCEV
                      << "\n");
    SCEVExpander Expander(*SE, "start");
    Instruction *Ins = L->getLoopPreheader()->getTerminator();
    Value *Start = Expander.expandCodeFor(StartSCEV, StartSCEV->getType(), Ins);
    LLVM_DEBUG(dbgs() << "ARM TP: Created start value " << *Start << "\n");
    InsertVCTPIntrinsic(ActiveLaneMask, Start);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 445-451
```cpp
  // Remove dead instructions and now dead phis.
  for (auto *II : ActiveLaneMasks)
    RecursivelyDeleteTriviallyDeadInstructions(II);
  for (auto *I : L->blocks())
    DeleteDeadPHIs(I);
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 453-455
```cpp
Pass *llvm::createMVETailPredicationPass() {
  return new MVETailPredication();
}
```
- EN: Implements `llvm::createMVETailPredicationPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMVETailPredicationPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 457-457
```cpp
char MVETailPredication::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 459-459
```cpp
INITIALIZE_PASS(MVETailPredication, DEBUG_TYPE, DESC, false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMSubtarget.h`, `ARMTargetTransformInfo.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMSubtarget.h`, `ARMTargetTransformInfo.h`。
- EN: LLVM infrastructure headers: `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/CodeGen/TargetPassConfig.h` ... (+8 more).
  - CN: LLVM 基础设施头文件：`llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/CodeGen/TargetPassConfig.h` ... (+8 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
